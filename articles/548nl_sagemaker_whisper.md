---
title: "WhisperをSageMaker Asynchronous Inferenceで利用する"
emoji: "👻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["機械学習", "音声認識", "sagemaker", "whisper"]
published: true
---

[Whisper](https://github.com/openai/whisper)は[OpenAI](https://openai.com/)がリリースした汎用音声認識モデルです。このWhisperをAWSの[SageMaker](https://aws.amazon.com/jp/sagemaker/)の[非同期推論オプション](https://aws.amazon.com/jp/about-aws/whats-new/2021/08/amazon-sagemaker-asynchronous-new-inference-option/)(Asynchronous Inference)でデプロイして使ってみます。この非同期オプションは推論に長い時間(最大15分)かかる場合に使用します。またこのオプションはリクエストがない時にインスタンスを0にスケールダウンできるので、リアルタイム推論よりもコストダウンが可能です。

SageMakerの非同期推論は以下のような構成になっています。

![](https://docs.aws.amazon.com/sagemaker/latest/dg/images/async-architecture.png)

([Aamazon SageMaker Developer Guide](https://docs.aws.amazon.com/sagemaker/latest/dg/async-inference.html)より)

ユーザからのリクエストは以下のような流れで非同期に処理されます。

1. クライアントは、処理して欲しいデータ一式を1つのファイルにまとめて、S3のバケットにアップロードします。
2. クライアントは、S3のオブジェクトのURLを引数にSageMakerのエンドポイントにリクエストします。
3. SageMakerは、HTTPステータスコード 202(Accepted)で推論処理結果が格納されるS3のURLを含むJSONを返します。
4. SageMakerは、機械学習のインスタンス(ml instance: 実態はコンテナでユーザが自前でコンテナイメージを用意する事も可能)が起動していない場合は起動します。この時、別途S3にアップロードしておいた、モデルデータを所定の位置に展開しておきます。
6. SageMakerは、ml instanceをS3のファイルをbodyにしてHTTPのPOSTメソッドを呼び出します。
7. ml instanceは、処理結果をHTTPレスポンスとして、SageMakerに返します。
8. SageMakerは、受け取ったレスポンスをS3に格納します。
9. クライアントは、SNSで通知を受け取るか、S3に処理結果が格納されていないかポーリングします。
10. クライアントは、結果がS3にあれば処理結果をダウンロードします。

大まかな流れがわかった所で、実際に使ってみます。

## 事前準備

1. [こちらの記事](https://zenn.dev/thorie/articles/548nl_sagemaker_serverless#%E3%83%A2%E3%83%87%E3%83%AB%E3%83%87%E3%83%BC%E3%82%BF%E3%81%AEs3%E3%83%90%E3%82%B1%E3%83%83%E3%83%88%E3%81%B8%E3%81%AE%E8%AA%AD%E3%81%BF%E5%8F%96%E3%82%8A%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B9%E3%81%AErole%E3%82%92%E4%BD%9C%E6%88%90)のように、AWS IAMロールを作成します。
2. [こちらの記事](https://zenn.dev/thorie/articles/548nl_sagemaker_local)のようにSageMakerをローカルPCから利用できるようにセットアップします。

## 手順

この手順はJupyterLab上で実施します。

### 推論用の独自コンテナのイメージを作成

まず、Whisperを稼働させるための以下のDockerfileを作成します。
Cudaのベースイメージに、Python、Webサーバ用パッケージ(Nginx, gunicorn, flask)、FFmpeg、Whisperをインストールし、後述のサーバ・アプリケーションをコピーしています。

```dockerfile
FROM nvidia/cuda:11.7.1-runtime-ubuntu20.04

RUN apt-get -y update
RUN DEBIAN_FRONTEND=noninteractive apt-get -y install \
        python3-pip \
        python3-setuptools \
        ffmpeg \
        git \
         nginx \
         ca-certificates \
    && rm -rf /var/lib/apt/lists/*

RUN pip --no-cache-dir install git+https://github.com/openai/whisper.git setuptools-rust flask gunicorn

ENV PYTHONUNBUFFERED=TRUE
ENV PYTHONDONTWRITEBYTECODE=TRUE
ENV PATH="/opt/program:${PATH}"

COPY src /opt/program
WORKDIR /opt/program
```

このコンテナは以下のような構成になっています。

![](https://raw.githubusercontent.com/aws/amazon-sagemaker-examples/main/advanced_functionality/scikit_bring_your_own/stack.png)

(amazon-sagemaker-examplesの[Building your own algorithm container](https://github.com/aws/amazon-sagemaker-examples/blob/main/advanced_functionality/scikit_bring_your_own/scikit_bring_your_own.ipynb)より)

コンテナの起動処理については、[こちら](https://github.com/horie-t/programming-study/tree/master/aws_sagemaker/sagemaker_whisper/src)の[serve](https://github.com/horie-t/programming-study/blob/master/aws_sagemaker/sagemaker_whisper/src/serve)、[nginx.conf](https://github.com/horie-t/programming-study/blob/master/aws_sagemaker/sagemaker_whisper/src/nginx.conf)、[wsgi.py](https://github.com/horie-t/programming-study/blob/master/aws_sagemaker/sagemaker_whisper/src/wsgi.py)を参照してください。コンテナは、起動時にSageMakerによりserveが呼び出されます。なお、trainは学習処理で使用するので、ここでは何もしないように実装しています。

コンテナで稼働する推論のWebアプリケーションは以下のようになります。

まず、`ping`と`invocations`の2つのエンドポイントを持ちます。
`ping`は、SageMakerが起動完了の確認やヘルスチェックに使用します。

`invocations`は、クライアントからSageMakerのエンドポイントに推論のリクエストがきた時に非同期で呼び出されます。
このときのbodyにはクライアントが指定したS3のファイルの中身が渡されます。

ここでは、

1. `ping`が呼び出された時にモデルを起動するようにし、起動できたら200のレスポンスを返すようにしています。モデルはSageMakerによって`/opt/ml/model`に展開されています。
2. `invocations`が呼び出さた時に、Whisperによって文字起こしが実行されるようにしています。文字起こし結果をレスポンスのbodyにしています。

```python
from __future__ import print_function
import imp

import os, tempfile
from urllib import request

import whisper

import flask

prefix = "/opt/ml/"
model_path = os.path.join(prefix, "model")

class TranslateService(object):
    model = None

    @classmethod
    def get_model(cls):
        if cls.model == None:
            cls.model = whisper.load_model("large", download_root=model_path)

        return cls.model

    @classmethod
    def transcribe(cls, voice_file):
        model = cls.get_model()
        res = model.transcribe(voice_file)

        return res["text"]


app = flask.Flask(__name__)

@app.route("/ping", methods=["GET"])
def ping():
    health = TranslateService.get_model() is not None

    status = 200 if health else 404
    return flask.Response(response="\n", status=status, mimetype="application/json")

@app.route("/invocations", methods=["POST"])
def transcribe():
    res = None
    with tempfile.NamedTemporaryFile() as f:
        f.write(flask.request.get_data())
        res = TranslateService.transcribe(f.name)
        f.close()

    return flask.Response(response=res, status=200, mimetype="text/plain")
```

[こちらのbuild_and_push.sh](https://github.com/horie-t/programming-study/blob/master/aws_sagemaker/sagemaker_whisper/build_and_push.sh)を使って、ECRにコンテナイメージを確認します。

```python
!./build_and_push.sh sagemaker-whisper
```

### WhisperのモデルをS3に配置

Whisperのモデルをダウンロードするディレクトリを作成し、モデルをダウンロードします。
ダウンロードするURLは[こちら](https://github.com/openai/whisper/blob/main/whisper/__init__.py)で確認します。

```python
!mkdir model
%cd model
```

```python
!curl -O "https://openaipublic.azureedge.net/main/whisper/models/e4b87e7e0bf463eb8e6956e646f1e277e901512310def2c24bf0e11bd3c28e9a/large.pt"
```

モデルデータをSageMakerで使うために、`tar.gz`形式でアーカイブします。

```python
!tar -czf model.tar.gz large.pt
```

アーカイブ後にS3にアップロードします。

```python
import boto3

role_name = "SageMaker-local"

iam = boto3.client("iam")
role = iam.get_role(RoleName=role_name)["Role"]["Arn"]
```

```python
import sagemaker as sage

sess = sage.Session()
```

```python
model_location = sess.upload_data("./model.tar.gz", key_prefix="whisper/model")
```

### モデルをデプロイしてエンドポイントを作成

アップロード後にSageMakerに推論用のモデルの定義を作成します。

```python
account = sess.boto_session.client('sts').get_caller_identity()['Account']
region = sess.boto_session.region_name
image = '{}.dkr.ecr.{}.amazonaws.com/sagemaker-whisper:latest'.format(account, region)
```

```python
model_name = 'whisper'

container_params = {
    "Image": image,
    "ModelDataUrl": model_location,
}

model = sess.create_model(model_name, role, container_params)
```

モデルの定義を作成後に、モデルをデプロイするエンドポイントの設定を作成します。

```python
sagemaker_client = sess.boto_session.client('sagemaker', region_name='us-west-2')
```

```python
model = "whisper"
endpoint_config_name = "whisper-config"

create_endpoint_config_response = sagemaker_client.create_endpoint_config(
    EndpointConfigName=endpoint_config_name,
    ProductionVariants=[
        {
            "VariantName": "variant1",
            "ModelName": model, 
            "InstanceType": "ml.p3.2xlarge",
            "InitialInstanceCount": 1
        }
    ],
    AsyncInferenceConfig={
        "OutputConfig": {
            "S3OutputPath": f"s3://{sess.default_bucket()}/whisper/output"
        },
    }
)

```

エンドポイントの設定が完了したら、実際にデプロイします。

```python
endpoint_name = 'whisper'
endpoint_config_name = "whisper-config"

create_endpoint_response = sagemaker_client.create_endpoint(
                                            EndpointName=endpoint_name, 
                                            EndpointConfigName=endpoint_config_name)
```

### 推論(文字起こし)を実行

エンドポイントが`InService`の状態になったら、以下のように適当な音声ファイル(例:`example_voice.mp3`)をS3にアップロードし、SageMakerのエンドポイントを非同期で呼び出します。

```python
model_location = sess.upload_data("./example_voice.mp3", key_prefix="whisper/input")
```

```python
sagemaker_runtime = boto3.client("sagemaker-runtime", region_name='us-west-2')

input_location = f"s3://{sess.default_bucket()}/whisper/input/example_voice.mp3"

response = sagemaker_runtime.invoke_endpoint_async(
                            EndpointName=endpoint_name, 
                            InputLocation=input_location)

print(response)
```

レスポンスのヘッダ(x-amzn-sagemaker-outputlocation)に文字起こし結果が格納されるS3のURLが設定されるので、推論完了後にS3からファイルをダウンロードします。

### オートスケーリングの設定を、リクエストがない時はインスタンス数を0にする

リクエストがない時にインスタンス数を0にするには以下のようにします。

```python
autoscaling_client = sess.boto_session.client('application-autoscaling') 

variant_name = "variant1"
endpoint_name = 'whisper'

resource_id=f'endpoint/{endpoint_name}/variant/{variant_name}' 

response = autoscaling_client.register_scalable_target(
    ServiceNamespace='sagemaker', 
    ResourceId=resource_id,
    ScalableDimension='sagemaker:variant:DesiredInstanceCount',
    MinCapacity=0,  
    MaxCapacity=1
)

response = autoscaling_client.put_scaling_policy(
    PolicyName='Invocations-ScalingPolicy',
    ServiceNamespace='sagemaker', 
    ResourceId=resource_id, 
    ScalableDimension='sagemaker:variant:DesiredInstanceCount',
    PolicyType='TargetTrackingScaling',
    TargetTrackingScalingPolicyConfiguration={
        'TargetValue': 1.0, 
        'CustomizedMetricSpecification': {
            'MetricName': 'ApproximateBacklogSizePerInstance',
            'Namespace': 'AWS/SageMaker',
            'Dimensions': [
                {'Name': 'EndpointName', 'Value': endpoint_name }
            ],
            'Statistic': 'Average',
        },
        'ScaleInCooldown': 120,
        'ScaleOutCooldown': 120
    }
)
```

## 備考

本記事のソースコードは[GitHub](https://github.com/horie-t/programming-study/tree/master/aws_sagemaker/sagemaker_whisper)にて公開しています。

本記事は、[Amazon SageMaker Examples](https://github.com/aws/amazon-sagemaker-examples)にてライセンスされるコードを含んでいます。ライセンスについてはそちらを参照してください。