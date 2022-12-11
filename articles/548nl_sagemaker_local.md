---
title: "SageMakerをローカルPCで利用する"
emoji: "👌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["機械学習", "aws", "sagemaker"]
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
以降のNotebookファイルやソースコードは[GitHub](https://github.com/horie-t/programming-study/tree/master/aws_sagemaker/sagemaker_mnist)にあります。

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

以下のようなスクリプトを作成します。スクリプトファイルは[こちら](https://github.com/horie-t/programming-study/blob/master/aws_sagemaker/sagemaker_mnist/mnist.py)。


```python
!pygmentize mnist.py
```
```python:mnist.py
import argparse
import json
import logging
import os
import sys

#import sagemaker_containers
import torch
import torch.distributed as dist
import torch.nn as nn
import torch.nn.functional as F
import torch.optim as optim
import torch.utils.data
import torch.utils.data.distributed
from torchvision import datasets, transforms

logger = logging.getLogger(__name__)
logger.setLevel(logging.DEBUG)
logger.addHandler(logging.StreamHandler(sys.stdout))


# Based on https://github.com/pytorch/examples/blob/master/mnist/main.py
class Net(nn.Module):
    def __init__(self):
        super(Net, self).__init__()
        self.conv1 = nn.Conv2d(1, 10, kernel_size=5)
        self.conv2 = nn.Conv2d(10, 20, kernel_size=5)
        self.conv2_drop = nn.Dropout2d()
        self.fc1 = nn.Linear(320, 50)
        self.fc2 = nn.Linear(50, 10)

    def forward(self, x):
        x = F.relu(F.max_pool2d(self.conv1(x), 2))
        x = F.relu(F.max_pool2d(self.conv2_drop(self.conv2(x)), 2))
        x = x.view(-1, 320)
        x = F.relu(self.fc1(x))
        x = F.dropout(x, training=self.training)
        x = self.fc2(x)
        return F.log_softmax(x, dim=1)


def _get_train_data_loader(batch_size, training_dir, is_distributed, **kwargs):
    logger.info("Get train data loader")
    dataset = datasets.MNIST(
        training_dir,
        train=True,
        transform=transforms.Compose(
            [transforms.ToTensor(), transforms.Normalize((0.1307,), (0.3081,))]
        ),
    )
    train_sampler = (
        torch.utils.data.distributed.DistributedSampler(dataset) if is_distributed else None
    )
    return torch.utils.data.DataLoader(
        dataset,
        batch_size=batch_size,
        shuffle=train_sampler is None,
        sampler=train_sampler,
        **kwargs
    )


def _get_test_data_loader(test_batch_size, training_dir, **kwargs):
    logger.info("Get test data loader")
    return torch.utils.data.DataLoader(
        datasets.MNIST(
            training_dir,
            train=False,
            transform=transforms.Compose(
                [transforms.ToTensor(), transforms.Normalize((0.1307,), (0.3081,))]
            ),
        ),
        batch_size=test_batch_size,
        shuffle=True,
        **kwargs
    )


def _average_gradients(model):
    # Gradient averaging.
    size = float(dist.get_world_size())
    for param in model.parameters():
        dist.all_reduce(param.grad.data, op=dist.reduce_op.SUM)
        param.grad.data /= size


def train(args):
    is_distributed = len(args.hosts) > 1 and args.backend is not None
    logger.debug("Distributed training - {}".format(is_distributed))
    use_cuda = args.num_gpus > 0
    logger.debug("Number of gpus available - {}".format(args.num_gpus))
    kwargs = {"num_workers": 1, "pin_memory": True} if use_cuda else {}
    device = torch.device("cuda" if use_cuda else "cpu")

    if is_distributed:
        # Initialize the distributed environment.
        world_size = len(args.hosts)
        os.environ["WORLD_SIZE"] = str(world_size)
        host_rank = args.hosts.index(args.current_host)
        os.environ["RANK"] = str(host_rank)
        dist.init_process_group(backend=args.backend, rank=host_rank, world_size=world_size)
        logger.info(
            "Initialized the distributed environment: '{}' backend on {} nodes. ".format(
                args.backend, dist.get_world_size()
            )
            + "Current host rank is {}. Number of gpus: {}".format(dist.get_rank(), args.num_gpus)
        )

    # set the seed for generating random numbers
    torch.manual_seed(args.seed)
    if use_cuda:
        torch.cuda.manual_seed(args.seed)

    train_loader = _get_train_data_loader(args.batch_size, args.data_dir, is_distributed, **kwargs)
    test_loader = _get_test_data_loader(args.test_batch_size, args.data_dir, **kwargs)

    logger.debug(
        "Processes {}/{} ({:.0f}%) of train data".format(
            len(train_loader.sampler),
            len(train_loader.dataset),
            100.0 * len(train_loader.sampler) / len(train_loader.dataset),
        )
    )

    logger.debug(
        "Processes {}/{} ({:.0f}%) of test data".format(
            len(test_loader.sampler),
            len(test_loader.dataset),
            100.0 * len(test_loader.sampler) / len(test_loader.dataset),
        )
    )

    model = Net().to(device)
    if is_distributed and use_cuda:
        # multi-machine multi-gpu case
        model = torch.nn.parallel.DistributedDataParallel(model)
    else:
        # single-machine multi-gpu case or single-machine or multi-machine cpu case
        model = torch.nn.DataParallel(model)

    optimizer = optim.SGD(model.parameters(), lr=args.lr, momentum=args.momentum)

    for epoch in range(1, args.epochs + 1):
        model.train()
        for batch_idx, (data, target) in enumerate(train_loader, 1):
            data, target = data.to(device), target.to(device)
            optimizer.zero_grad()
            output = model(data)
            loss = F.nll_loss(output, target)
            loss.backward()
            if is_distributed and not use_cuda:
                # average gradients manually for multi-machine cpu case only
                _average_gradients(model)
            optimizer.step()
            if batch_idx % args.log_interval == 0:
                logger.info(
                    "Train Epoch: {} [{}/{} ({:.0f}%)] Loss: {:.6f}".format(
                        epoch,
                        batch_idx * len(data),
                        len(train_loader.sampler),
                        100.0 * batch_idx / len(train_loader),
                        loss.item(),
                    )
                )
        test(model, test_loader, device)
    save_model(model, args.model_dir)


def test(model, test_loader, device):
    model.eval()
    test_loss = 0
    correct = 0
    with torch.no_grad():
        for data, target in test_loader:
            data, target = data.to(device), target.to(device)
            output = model(data)
            test_loss += F.nll_loss(output, target, size_average=False).item()  # sum up batch loss
            pred = output.max(1, keepdim=True)[1]  # get the index of the max log-probability
            correct += pred.eq(target.view_as(pred)).sum().item()

    test_loss /= len(test_loader.dataset)
    logger.info(
        "Test set: Average loss: {:.4f}, Accuracy: {}/{} ({:.0f}%)\n".format(
            test_loss, correct, len(test_loader.dataset), 100.0 * correct / len(test_loader.dataset)
        )
    )


def model_fn(model_dir):
    device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
    model = torch.nn.DataParallel(Net())
    with open(os.path.join(model_dir, "model.pth"), "rb") as f:
        model.load_state_dict(torch.load(f))
    return model.to(device)


def save_model(model, model_dir):
    logger.info("Saving the model.")
    path = os.path.join(model_dir, "model.pth")
    # recommended way from http://pytorch.org/docs/master/notes/serialization.html
    torch.save(model.cpu().state_dict(), path)


if __name__ == "__main__":
    parser = argparse.ArgumentParser()

    # Data and model checkpoints directories
    parser.add_argument(
        "--batch-size",
        type=int,
        default=64,
        metavar="N",
        help="input batch size for training (default: 64)",
    )
    parser.add_argument(
        "--test-batch-size",
        type=int,
        default=1000,
        metavar="N",
        help="input batch size for testing (default: 1000)",
    )
    parser.add_argument(
        "--epochs",
        type=int,
        default=10,
        metavar="N",
        help="number of epochs to train (default: 10)",
    )
    parser.add_argument(
        "--lr", type=float, default=0.01, metavar="LR", help="learning rate (default: 0.01)"
    )
    parser.add_argument(
        "--momentum", type=float, default=0.5, metavar="M", help="SGD momentum (default: 0.5)"
    )
    parser.add_argument("--seed", type=int, default=1, metavar="S", help="random seed (default: 1)")
    parser.add_argument(
        "--log-interval",
        type=int,
        default=100,
        metavar="N",
        help="how many batches to wait before logging training status",
    )
    parser.add_argument(
        "--backend",
        type=str,
        default=None,
        help="backend for distributed training (tcp, gloo on cpu and gloo, nccl on gpu)",
    )

    # Container environment
    parser.add_argument("--hosts", type=list, default=json.loads(os.environ["SM_HOSTS"]))
    parser.add_argument("--current-host", type=str, default=os.environ["SM_CURRENT_HOST"])
    parser.add_argument("--model-dir", type=str, default=os.environ["SM_MODEL_DIR"])
    parser.add_argument("--data-dir", type=str, default=os.environ["SM_CHANNEL_TRAINING"])
    parser.add_argument("--num-gpus", type=int, default=os.environ["SM_NUM_GPUS"])

    train(parser.parse_args())
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
