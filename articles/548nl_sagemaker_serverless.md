---
title: "Amazone SageMakerでサーバレスな機械翻訳サービスを構築する"
emoji: "🐥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["機械学習", "機械翻訳"]
published: true
---

『[CTranslate2を使った機械翻訳をDocker上で実行する](./548nl_ctranslate2_setup.md)』でDockerコンテナ上でCTranslate2を実行できるようになったので、本記事では「Amazone SageMaker Serverless Inference」で実行してサーバレスな機械翻訳サービスを構築します。

SageMaker Serverless Inferenceで独自のアルゴリズムを使用したい場合は以下の2つのリソースが必要になります。

* S3に配置したモデルデータ
* ECRにpushされたDockerイメージ

また、以下の設定が必要になります。

* モデルデータへの読み取りのAWS IAM RoleをSageMakerのモデル設定にアタッチする
* コンテナをデプロイし、S3のモデルデータをコンテナにコピーする

それでは、構築していきましょう。

## S3へのモデルデータを配置

『[CTranslate2を使った機械翻訳をDocker上で実行する](./548nl_ctranslate2_setup.md)』で作成したファイルをtar.gz形式でアーカイブします。

以下のコマンドでアーカイブします。

```bash
cd /tmp/ctranslate2 # 前の記事でモデルデータを変換したディレクトリに移動します。
tar -cvzf model.tar.gz sentencepiece.model ctranslate2_model
```

[S3 コンソール](https://s3.console.aws.amazon.com/s3/) でSageMaker用のS3バケット(例: `sagemeker.example.com` )を作成し、`model.tar.gz`をアップロードします。ここでは、仮にアップロードしたS3 URIを`s3://sagemeker.example.com/models/model.tar.gz`とします。

## モデルデータのS3バケットへの読み取りアクセスのRoleを作成

まず、Roleを作成します。

1. [IAM コンソール](https://console.aws.amazon.com/iam/) を開きます。
2. **ロール**を選択します。
3. **ロールを作成**ボタンをクリックします。
4. **信頼されたエンティティタイプ]**で**AWSのサービス**を選択し、**ユースケース**で**他のAWSのサービスのユースケース**で**SageMaker**を入力し、**SageMaker - Execution**を選択し、**次へ**ボタンをクリックします。
5. **許可を追加**画面で、**次へ**をクリックします。
6. **ロール名** (例: `SageMakerRole`)を入力し、**ロールを作成**ボタンをクリックします。

次に、S3への読み取りのポリシーを作成して、作成したRoleにアタッチします。

1. **ロール**の一覧から、先程作成したロール (`SageMakerRole`)を選択します。
2. **許可を追加**のドロップダウンから、**ポリシーをアタッチ**を選択します。
3. **ポリシーを作成**ボタンをクリックします。
4. **ポリシーの作成**画面で、**JSON**タブをクリックし、以下を入力します。**バケット名**は`sagemeker.example.com`等です。入力後、**次のステップ: タグ**ボタンをクリックします。  
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::バケット名/*"
        }
    ]
}
```
5. **次のステップ: タグ**ボタンをクリックします。
6. **名前** (例: `SageMakerS3BucketAccess`)を入力し、**ポリシーの作成**ボタンをクリックします。
7. **ロール**の**ポリシーにアタッチ**のタブに戻って、**ポリシーを作成**ボタンの左隣の**更新**ボタンをクリックし、前ステップで作成したポリシーを選択し、**ポリシーをアタッチ**ボタンをクリックします。

## 推論用のDockerコンテナを作成しECRへpush

SageMaker はコンテナを起動する際、引数にtrainまたはserve を指定してコンテナを呼び出します。このコンテナは、引数をコンテナが実行するコマンドとして扱うように設定されています。トレーニングする場合はtrainプログラムを実行し、推論する場合はserveプログラムを実行します。通常、serveは推論サーバを開始するラッパープログラムです。

起動された推論サーバは以下のHTTPのAPIをサポートします。

* GET /ping  
  起動の確認や、ヘルスチェックに使用します。
* POST /invocations  
  推論を実行するAPIです。本記事ではBodyに原文をセットして、翻訳をリクエストします。

本記事では、推論サーバで以下のスタックを使用します。

1. nginx: HTTP リクエストの受信を処理し、コンテナ内外の I/O を効率的に管理する軽量なレイヤーです。
2. gunicorn: WSGI のプリフォークワーカーサーバで、アプリケーションの複数のコピーを実行し、それらの間でロードバランシングを行います。
3. flask: 翻訳のアプリケーションで使われるシンプルなWebフレームワークです。これによって、多くのコードを書くことなく、/pingや /invocationsのエンドポイントへの呼び出しに応答することができます。

### flaskで翻訳サーバを実装

以下のように、翻訳サーバを実装します。
/opt/ml/modelには、SageMakerによってコンテナ起動時にS3にアップロードした`model.tar.gz`が

```python:src/translator.py
from __future__ import print_function

import os
import ctranslate2
import sentencepiece as spm

import flask

prefix = "/opt/ml/"
model_path = os.path.join(prefix, "model")

# A singleton for holding the model. This simply loads the model and holds it.
# It has a predict function that does a prediction based on the model and the input data.

class TranslateService(object):
    translator = None
    sp = None

    @classmethod
    def get_model(cls):
        if cls.translator == None:
            cls.translator = ctranslate2.Translator(f"{model_path}/ctranslate2_model", device="cpu")
        if cls.sp == None:
            cls.sp = spm.SentencePieceProcessor(f"{model_path}/sentencepiece.model")
        
        return (cls.translator, cls.sp)

    @classmethod
    def tranlate(cls, original_text):
        (translator, sp) = cls.get_model()
        input_tokens = sp.encode(original_text, out_type=str)

        results = translator.translate_batch([input_tokens])

        output_tokens = results[0].hypotheses[0]
        translated_text = sp.decode(output_tokens)

        return translated_text

# The flask app for serving predictions
app = flask.Flask(__name__)


@app.route("/ping", methods=["GET"])
def ping():
    """Determine if the container is working and healthy. In this sample container, we declare
    it healthy if we can load the model successfully."""
    (translator, sp) = TranslateService.get_model()
    health = translator is not None and sp is not None

    status = 200 if health else 404
    return flask.Response(response="\n", status=status, mimetype="application/json")


@app.route("/invocations", methods=["POST"])
def translate():
    data = None

    data = flask.request.data.decode("utf-8")
    # 翻訳する
    translated_text = TranslateService.tranlate(data)

    return flask.Response(response=translated_text, status=200, mimetype="text/plain")
```

### gunicorn

gunicornはシンプルなラッパーとして構築します。

```python:src/wsgi.py
import translator as myapp

app = myapp.app
```

### nginx

以下のようにnginxを設定します。

```text:src/nginx.conf
worker_processes 1;
daemon off; # Prevent forking


pid /tmp/nginx.pid;
error_log /var/log/nginx/error.log;

events {
  # defaults
}

http {
  include /etc/nginx/mime.types;
  default_type application/octet-stream;
  access_log /var/log/nginx/access.log combined;
  
  upstream gunicorn {
    server unix:/tmp/gunicorn.sock;
  }

  server {
    listen 8080 deferred;
    client_max_body_size 5m;

    keepalive_timeout 5;
    proxy_read_timeout 1200s;

    location ~ ^/(ping|invocations) {
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header Host $http_host;
      proxy_redirect off;
      proxy_pass http://gunicorn;
    }

    location / {
      return 404 "{}";
    }
  }
}
```

### サーバの起動処理

サーバを起動するserveコマンドは以下のようになります。

```python:src/serve
#!/usr/bin/env python3

import multiprocessing
import os
import signal
import subprocess
import sys

cpu_count = multiprocessing.cpu_count()

model_server_timeout = os.environ.get('MODEL_SERVER_TIMEOUT', 60)
model_server_workers = int(os.environ.get('MODEL_SERVER_WORKERS', cpu_count))

def sigterm_handler(nginx_pid, gunicorn_pid):
    try:
        os.kill(nginx_pid, signal.SIGQUIT)
    except OSError:
        pass
    try:
        os.kill(gunicorn_pid, signal.SIGTERM)
    except OSError:
        pass

    sys.exit(0)

def start_server():
    print('Starting the inference server with {} workers.'.format(model_server_workers))

    # link the log streams to stdout/err so they will be logged to the container logs
    subprocess.check_call(['ln', '-sf', '/dev/stdout', '/var/log/nginx/access.log'])
    subprocess.check_call(['ln', '-sf', '/dev/stderr', '/var/log/nginx/error.log'])

    nginx = subprocess.Popen(['nginx', '-c', '/opt/program/nginx.conf'])
    gunicorn = subprocess.Popen(['gunicorn',
                                 '--timeout', str(model_server_timeout),
                                 '-k', 'sync',
                                 '-b', 'unix:/tmp/gunicorn.sock',
                                 '-w', str(model_server_workers),
                                 'wsgi:app'])

    signal.signal(signal.SIGTERM, lambda a, b: sigterm_handler(nginx.pid, gunicorn.pid))

    # If either subprocess exits, so do we.
    pids = set([nginx.pid, gunicorn.pid])
    while True:
        pid, _ = os.wait()
        if pid in pids:
            break

    sigterm_handler(nginx.pid, gunicorn.pid)
    print('Inference server exiting')

# The main routine just invokes the start function.

if __name__ == '__main__':
    start_server()
```

trainコマンドも実装しますが、何もしないようにします。

```python:src/train
#!/usr/bin/env python3

import sys

if __name__ == '__main__':
    # 学習済みモデルを使用するので学習は省略

    # A zero exit code causes the job to be marked a Succeeded.
    sys.exit(0)
```

### Dockerイメージの作成とECRへpush

以下のDockerfileをを作成します。

```docker
FROM ubuntu:22.04

RUN apt-get -y update && apt-get install -y --no-install-recommends \
         wget \
         python3-pip \
         python3-setuptools \
         nginx \
         ca-certificates \
    && rm -rf /var/lib/apt/lists/*

RUN pip --no-cache-dir install ctranslate2 OpenNMT-py sentencepiece flask gunicorn

ENV PYTHONUNBUFFERED=TRUE
ENV PYTHONDONTWRITEBYTECODE=TRUE
ENV PATH="/opt/program:${PATH}"

COPY src /opt/program
WORKDIR /opt/program
```

以下のスクリプトを作成し、DockerイメージをECRにpushできるようにします。ECRのリポジトリが作成されていない場合は、リポジトリが作成されます。

```bash:build_and_push.sh
#!/usr/bin/env bash

# This script shows how to build the Docker image and push it to ECR to be ready for use
# by SageMaker.

# The argument to this script is the image name. This will be used as the image on the local
# machine and combined with the account and region to form the repository name for ECR.
image=$1

if [ "$image" == "" ]
then
    echo "Usage: $0 <image-name>"
    exit 1
fi

chmod +x src/train
chmod +x src/serve

# Get the account number associated with the current IAM credentials
account=$(aws sts get-caller-identity --query Account --output text)

if [ $? -ne 0 ]
then
    exit 255
fi


# Get the region defined in the current configuration (default to us-west-2 if none defined)
region=$(aws configure get region)
region=${region:-us-west-2}


fullname="${account}.dkr.ecr.${region}.amazonaws.com/${image}:latest"

# If the repository doesn't exist in ECR, create it.

aws ecr describe-repositories --repository-names "${image}" > /dev/null 2>&1

if [ $? -ne 0 ]
then
    aws ecr create-repository --repository-name "${image}" > /dev/null
fi

# Get the login command from ECR and execute it directly
aws ecr get-login-password --region "${region}" | docker login --username AWS --password-stdin "${account}".dkr.ecr."${region}".amazonaws.com

# Build the docker image locally with the image name and then push it to ECR
# with the full name.

docker build  -t ${image} .
docker tag ${image} ${fullname}

docker push ${fullname}
```

以下のコマンドで、Dockerイメージを作成しECRへpushします。

```bash
./build_and_push.sh sagemaker-ctranslate2
```

## SageMakerの設定

SageMakerの準備が整ったので、[SageMaker コンソール](https://console.aws.amazon.com/sagemaker/)画面を開きSageMakerの設定をしていきます。

### モデルの作成

まず推論を実行するモデルとコンテナの設定を作成します。

1. **推論** - **モデル** をクリックします。
2. **モデルの作成**をクリックします。
3. **モデルの作成**画面で、
   1. **モデル名**(例: `CTranslate2`)を入力し
   2. **IAMロール**を前述の作成したロールに設定し
   3. **コンテナ入力オプション**は**モデルアーティファクトと推論イメージの場所を指定します。**を選択し
   4. **モデルアーティファクトと推論イメージのオプションを指定します。**は**単一のモデルを使用する**を選択し  
    **推論コードイメージの場所**は[ECR コンソール](https://console.aws.amazon.com/ecr/)で``sagemaker-ctranslate2:latest``のイメージのURI(例: `123456789012.dkr.ecr.us-west-2.amazonaws.com/sagemaker-ctranslate2:latest`)を設定し  
    **アーティファクトの場所**は`model.tar.gz`のURI(例: `s3://sagemeker.example.com/models/model.tar.gz`)を設定し
   5. **モデルの作成**ボタンをクリックします。

### エンドポイント設定の設定

次にモデルをエンドポイントとしてデプロイするための設定を作成します。

1. **推論** - **エンドポイント設定** をクリックします。
2. **エンドポイント設定の作成**ボタンをクリックします。
3. **エンドポイント設定の作成**画面で、  
   1. **エンドポイント設定名**(例: `CTranslate2`)を入力し
   2. **エンドポイントのタイプ**は**サーバーレス**を選択し
   3. **本番稼働用バリアント**の**モデルの追加**をクリックし、前述で作成したモデルを追加し
   4. **エンドポイント設定の作成***ボタンをクリックします。

### エンドポイントの作成

最後に、エンドポイントの作成を実行してモデルをデプロイします。

1. **推論** - **エンドポイント** をクリックします。
2. **エンドポイントの作成**ボタンをクリックします。
3. **エンドポイントの作成と設定**画面で
   1. **エンドポイント名**(例: `CTranslate2`)を入力し
   2. **エンドポイント設定**で前述で作成したエンドポイント設定を選択し**エンドポイント設定の選択**ボタンをクリックします。
   3. **エンドポイントの作成**ボタンをクリックします。
4. 作成されたエンドポイントの**ステータス**が**InServe**になっている事を確認します。

## エンドポイントの呼び出し

エンドポイント呼び出して翻訳を実行するために以下のスクリプトを作成します。

```python:translate.py
#!/usr/bin/env python3

import boto3
import sys

runtime = boto3.client("sagemaker-runtime")

endpoint_name = "CTranslate2"
content_type = "text/plain"
payload = sys.argv[1]

response = runtime.invoke_endpoint(
    EndpointName=endpoint_name,
    ContentType=content_type,
    Body=payload
)

print(response['Body'].read().decode('utf-8'))
```

以下のコマンドを実行して翻訳します。

```bash
$ ./translate.py 'Hello world!'
Hallo Welt!
```

## エンドポイントをAWS Lambdaから呼び出すようにしてHTTP APIとして公開する。

エンドポイントをAWS Lambdaから呼び出せるようにします。Lambdaの構築には、Serverless Frameworkを使用します。
適当なディレクトリに移動して、Serverless Frameworkのプロジェクトを作成します。

```bash
mkdir aws_lambda
cd !$
serverless create --template aws-python3
```

`handler.py`をSlageMakerのエンドポイントを呼び出すように編集します。

```python:aws_lambda/handler.py
import json
import boto3

def translate(event, context):
    runtime = boto3.client("sagemaker-runtime")

    endpoint_name = "CTranslate2"
    content_type = "text/plain"
    original_text = event['body']

    translate_response = runtime.invoke_endpoint(
        EndpointName=endpoint_name,
        ContentType=content_type,
        Body=original_text
    )
    translated_text = translate_response['Body'].read().decode('utf-8')

    response = {
        "statusCode": 200,
        "body": json.dumps(translated_text)
    }
    return response
```

`serverless.yml`を以下のように編集します。。SageMakerのエンドポイントを呼び出せるようにroleを設定しています。

```yml:aws_lambda/serverless.yml
service: aws-lambda

frameworkVersion: '3'

provider:
  name: aws
  runtime: python3.8
  region: us-west-2
  iam:
    role:
      statements:
        - Effect: "Allow"
          Action:
            - "sagemaker:InvokeEndpoint"
          Resource: "arn:aws:sagemaker:*:*:endpoint/ctranslate2"

functions:
  translate:
    handler: handler.translate
    events:
      - httpApi:
          path: /translate
          method: post
```



## 備考

本記事は、[Amazon SageMaker Examples](https://github.com/aws/amazon-sagemaker-examples)にてライセンスされるコードを含んでいます。ライセンスについてはそちらを参照してください。