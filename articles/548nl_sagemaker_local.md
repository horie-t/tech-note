---
title: "SageMakerをローカルPCで利用する"
emoji: "👌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["機械学習"]
published: true
---

本当は[SageMaker Studio Lab](https://studiolab.sagemaker.aws/)を使用したかったのですが、申し込んでも一向に利用できるようにならないので仕方なくローカルPCからSageMakerを利用してみて、PyTorchでMNISTを実行してみました。

以下のような実行環境になります。

* コーディング等は、ローカルPCで実行するJupyterLabで行います。
* 学習や推論などは、AWSの環境で行う。
* 学習用のデータやモデルは、S3に保存する。

![](https://raw.githubusercontent.com/horie-t/tech-note/master/images/SageMakerLocal.drawio.svg)

# 前準備

## AWS CLIを利用できるようにする

やり方の情報は世の中に溢れているので、例えば[Qiitaの記事](https://qiita.com/SSMU3/items/ce6e291a653f76ddcf79)を参照してください。

## SageMakerを利用するIAMロールを作成する。

1. [IAM](https://console.aws.amazon.com/iam/home)コンソールを開きます。
2. 左のNavigationで`ロール`を選択します。
3. `ロールを作成`ボタンをクリックします。
4. 「信頼されたエンティティタイプ」で`AWS のサービス`を選択し、「ユースケース」で「他の AWS のサービスのユースケース:」に`SageMaker`を入力し`SageMaker - Execution`を選択し、`次へ`ボタンをクリックします。
5. 「許可を追加」画面で`次へ`ボタンをクリックします。
6.  「名前、確認、および作成」画面で、「ロール名」に`SageMaker-local`を入力し、`ロールを作成`ボタンをクリックします。

# JupyterLabの実行環境構築

## Pythonの実行環境をセットアップ

Anacondaのインストールします。

[Anacondaのダウンロードサイト](https://www.anaconda.com/products/distribution#Downloads)からインストーラーをダウンロード。

```bash
cd ダウンロード
chmod +x Anaconda3-2022.05-Linux-x86_64.sh
./Anaconda3-2022.05-Linux-x86_64.sh
```

インストールが始まるので、問い合わせに答えていく。

```text
Welcome to Anaconda3 2022.05

In order to continue the installation process, please review the license
agreement.
Please, press ENTER to continue
>>> 
```
「Enter」キーを押下する。

```text
==================================================
End User License Agreement - Anaconda Distribution
==================================================
(中略)
Last updated February 25, 2022

Do you accept the license terms? [yes|no]
[no] >>> yes
```
ライセンスが表示されるので、スペースキーを押しながら画面を進めて、最後に「yes」を入力。


```text
Anaconda3 will now be installed into this location:
/home/USER_NAME/anaconda3

  - Press ENTER to confirm the location
  - Press CTRL-C to abort the installation
  - Or specify a different location below

[/home/USER_NAME/anaconda3] >>> 
```
デフォルトでインストールするので、そのまま「Enter」を押下。(`USER_NAME`は環境によって変化します。)

```text
  zstd               pkgs/main/linux-64::zstd-1.4.9-haebb681_0


Preparing transaction: done
Executing transaction: - 

    Installed package of scikit-learn can be accelerated using scikit-learn-intelex.
    More details are available here: https://intel.github.io/scikit-learn-intelex

    For example:

        $ conda install scikit-learn-intelex
        $ python -m sklearnex my_application.py

    

done
installation finished.
Do you wish the installer to initialize Anaconda3
by running conda init? [yes|no]
[no] >>> yes
```
インストールが完了したら、condaを初期化するかと聞かれるので、yesを入力。

## SageMaker用のPythonの実行環境をセットアップ

Anacondaをインストールが終わったら、別のターミナルを開き、Anacondaの仮想環境を作成する。

まず、利用できるPythonのバージョンを調べる

```bash
$ conda search python
Loading channels: done
# Name                       Version           Build  Channel             
python                        2.7.13     hac47a24_15  pkgs/main           
python                        2.7.13     heccc3f1_16  pkgs/main
...
python                        3.10.0      h151d27f_3  pkgs/main           
python                        3.10.3      h12debd9_5  pkgs/main           
python                        3.10.4      h12debd9_0  pkgs/main
```

最新版のPythonで環境を構築する。

```bash
conda create -n sagemaker python=3.10.4
```
問い合わせには、`y`を入力する。

インストールした環境を有効化する。

```bash
conda activate sagemaker
```

JupyterLabをインストールする。

```bash
conda install -c conda-forge jupyterlab
```

# SageMakerを利用する

## JupyterLabを起動する

適当なディレクトリで以下を実行します。

```bash
jupyter-lab
```

以降は、ブラウザのJupyterLabの画面で実行します。
以降のNotebookファイルやソースコードは[GitHub](https://github.com/horie-t/experiment/tree/master/sagemaker_mnist)にあります。

## パッケージのインストール
必要なパッケージをインストールします。


```python
!pip install -qU torchvision sagemaker
```

## SageMakerのセッション開始

SageMakerのセッションを開始し、データを保存するS3のプレフィックスと、IAMロールを設定します。
S3のバケット名は、`sagemaker-{リージョン名}-{アカウントID}`になります。

```python
import sagemaker
import boto3

sagemaker_session = sagemaker.Session()

bucket = sagemaker_session.default_bucket()
prefix = "sagemaker/DEMO-pytorch-mnist"

role_name = "SageMaker-local"
iam = boto3.client("iam")
role = iam.get_role(RoleName=role_name)["Role"]["Arn"]
```

## 学習用データの取得
学習用データを取得します。)


```python
from torchvision.datasets import MNIST
from torchvision import transforms

MNIST.mirrors = ["https://sagemaker-sample-files.s3.amazonaws.com/datasets/image/MNIST/"]

MNIST(
    "data",
    download=True,
    transform=transforms.Compose(
        [transforms.ToTensor(), transforms.Normalize((0.1307,), (0.3081,))]
    ),
)
```

## S3にデータをアップロード


```python
inputs = sagemaker_session.upload_data(path="data", bucket=bucket, key_prefix=prefix)
print("input spec (in this case, just an S3 path): {}".format(inputs))
```

## 学習用のスクリプトの作成

以下のようなスクリプトを作成します。スクリプトファイルは[こちら](https://github.com/horie-t/experiment/blob/master/sagemaker_mnist/mnist.py)。

```python
!pygmentize mnist.py
```

## SageMakerで学習を実行する
以下のように学習を実行します。
学習した結果のモデルデータは
`s3://sagemaker-{リージョン名}-{アカウントID}/{トレーニングジョブ名}/model.tar.gz`
に保存されます。(トレーニングジョブ名は実行毎に異なります。)


```python
from sagemaker.pytorch import PyTorch

estimator = PyTorch(
    entry_point="mnist.py",
    role=role,
    py_version="py38",
    framework_version="1.11.0",
    instance_count=2,
    instance_type="ml.c5.2xlarge",
    hyperparameters={"epochs": 1, "backend": "gloo"},
)
```


```python
estimator.fit({"training": inputs})
```

## エンドポイントを作成して推論を実行する
エンドポイントを作成(つまり、AWSの環境にデプロイしてAWS環境で推論を実行できるように)し、推論を実行します。


```python
predictor = estimator.deploy(initial_instance_count=1, instance_type="ml.m4.xlarge")
```


```python
!ls data/MNIST/raw
```


```python
import gzip
import numpy as np
import random
import os

data_dir = "data/MNIST/raw"
with gzip.open(os.path.join(data_dir, "t10k-images-idx3-ubyte.gz"), "rb") as f:
    images = np.frombuffer(f.read(), np.uint8, offset=16).reshape(-1, 28, 28).astype(np.float32)

mask = random.sample(range(len(images)), 16)  # randomly select some of the test images
mask = np.array(mask, dtype=np.int)
data = images[mask]
```


```python
response = predictor.predict(np.expand_dims(data, axis=1))
print("Raw prediction result:")
print(response)
print()

labeled_predictions = list(zip(range(10), response[0]))
print("Labeled predictions: ")
print(labeled_predictions)
print()

labeled_predictions.sort(key=lambda label_and_prob: 1.0 - label_and_prob[1])
print("Most likely answer: {}".format(labeled_predictions[0]))
```

## エンドポイントの削除
デプロイしたエンドポイントを削除します。<font color='red'>削除するまで、コンテナの実行費用がかかります。</font>


```python
sagemaker_session.delete_endpoint(endpoint_name=predictor.endpoint_name)
```

## Serverless Inferenceを使ったエンドポイントの作成
時々しか実行されず、レスポンス速度が重要ではない場合、Serverless Inferenceを使ってデプロイすると、推論の実行時のみ課金されるので費用が抑えられます。

以下のように、Severless環境を実行するDocker ImageのURIと、学習結果のモデルのS3のURIを指定して、モデルを生成しデプロイします。
`{リージョン名}`と`{アカウントID}`は、適宜変更してください。
その他のDocker Imageを使用したい場合、URIは[こちら](https://github.com/aws/deep-learning-containers/blob/master/available_images.md#sagemaker-framework-containers-sm-support-only)を参照してください。

```python
model = sagemaker.pytorch.model.PyTorchModel(
    model_data="s3://sagemaker-{リージョン名}-{アカウントID}/pytorch-training-2022-09-24-07-32-32-301/model.tar.gz",
    entry_point="mnist.py",
    role=role,
    image_uri="763104351884.dkr.ecr.{リージョン名}.amazonaws.com/pytorch-inference:1.12.1-cpu-py38-ubuntu20.04-sagemaker"
)
```


```python
from sagemaker.serverless import ServerlessInferenceConfig

serverless_config = ServerlessInferenceConfig(max_concurrency=1)
```


```python
predictor = model.deploy(serverless_inference_config=serverless_config)
```

先程と同様に、推論を実施します。


```python
response = predictor.predict(np.expand_dims(data, axis=1))
print("Raw prediction result:")
print(response)
print()

labeled_predictions = list(zip(range(10), response[0]))
print("Labeled predictions: ")
print(labeled_predictions)
print()

labeled_predictions.sort(key=lambda label_and_prob: 1.0 - label_and_prob[1])
print("Most likely answer: {}".format(labeled_predictions[0]))
```

## 備考

本ドキュメントは、[Amazon SageMaker Examples](https://github.com/aws/amazon-sagemaker-examples)のコードを含みます。そのライセンスは、[LICENSE.txt](https://github.com/aws/amazon-sagemaker-examples/blob/main/LICENSE.txt)を参照してください。
