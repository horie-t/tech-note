---
title: "SageMakerでJParaCrawlのコーパスを使って翻訳モデルを作成する"
emoji: "🎃"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["機械学習", "機械翻訳"]
published: true
---

Web上でよく見かける機械翻訳の翻訳モデルを作成した例は、[京都フリー翻訳タスク (KFTT)](http://www.phontron.com/kftt/index-ja.html)の対訳コーパスを使ったものが多いです。このコーパスは京都関連に特化したコーパスで文数も44万文と少ないものです。このデータをGoogle Colabの古いGPUを使って短時間だけ学習させてみて、辛うじて翻訳できているような結果になるものが多いです。

もっと本格的なコーパスを使って、強力なコンピュータを使って学習してみたらどうなるかを、今回試してみました。

まず、コーパスは[JParaCrawl](https://www.kecl.ntt.co.jp/icl/lirg/jparacrawl/)を使用します。JParaCrawlとはNTTによって作成された最大規模(2000万文以上)の日英対訳コーパスです。このコーパスを使って、SageMakerで`ml.g4dn.12xlarge`というGPUが4つもあるマシンで学習させてみました。

## 前準備

[こちらの記事](https://zenn.dev/thorie/articles/548nl_sagemaker_serverless#%E3%83%A2%E3%83%87%E3%83%AB%E3%83%87%E3%83%BC%E3%82%BF%E3%81%AEs3%E3%83%90%E3%82%B1%E3%83%83%E3%83%88%E3%81%B8%E3%81%AE%E8%AA%AD%E3%81%BF%E5%8F%96%E3%82%8A%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B9%E3%81%AErole%E3%82%92%E4%BD%9C%E6%88%90)にあるように、AWS上でIAM Roleを作成しておきます。

## コーパスのダウンロード

[JParaCrawl](https://www.kecl.ntt.co.jp/icl/lirg/jparacrawl/)のサイトからコーパスデータをダウンロードします。

```python
!mkdir data
%cd data
```

```python
!wget https://www.kecl.ntt.co.jp/icl/lirg/jparacrawl/release/3.0/bitext/en-ja.tar.gz
```

## コーパスの前処理

OpenNMT-pyで扱いやすいようにデータを前処理します。
まずは、データを展開します。


```python
!tar xvf en-ja.tar.gz
%cd en-ja/
```

データの中身を確認してみましょう。タブ区切りで5つのフィールドがあり、4、5番目にそれぞれ英文、和文がありました。

文数は25,740,835でした。


```python
!head en-ja.bicleaner05.txt
```

    0001vip.cocolog-nifty.com	0001vip.cocolog-nifty.com	0.535	And everyone will not care that it is not you.	鼻・口のところはあらかじめ少し切っておくといいですね。
    0001vip.cocolog-nifty.com	0001vip.cocolog-nifty.com	0.557	And everyone will not care that it is not you.	アドレス置いとくので、消されないうちにメールくれたら嬉しいです。
    000-lhr.web.wox.cc	000-lhr.web.wox.cc	0.743	Sponsored link This advertisement is displayed when there is no update for a certain period of time.	スポンサードリンク この広告は一定期間更新がない場合に表示されます。
    000-lhr.web.wox.cc	000-lhr.web.wox.cc	0.750	Also, it will always be hidden when becoming a premium user .	また、 プレミアムユーザー になると常に非表示になります。
    000-lhr.web.wox.cc	000-lhr.web.wox.cc	0.751	It will return to non-display when content update is done.	コンテンツの更新が行われると非表示に戻ります。
    000-lhr.web.wox.cc	kapuri21.web.wox.cc	0.743	Sponsored link This advertisement is displayed when there is no update for a certain period of time.	スポンサードリンク この広告は一定期間更新がない場合に表示されます。
    000-lhr.web.wox.cc	kapuri21.web.wox.cc	0.750	Also, it will always be hidden when becoming a premium user .	また、 プレミアムユーザー になると常に非表示になります。
    000-lhr.web.wox.cc	kapuri21.web.wox.cc	0.751	It will return to non-display when content update is done.	コンテンツの更新が行われると非表示に戻ります。
    0017.org	0017.org	0.500	It’s like you can enrich it and save money as a result. I’ve watched a lot of videos, mainly on Youtube, of people who say they’re minimalists, and I agree.	Youtubeを中心にミニマリストと言っている方の動画をたくさんみましたが、納得いくもののも多く、不況と少子化を呼ばれる”今の”日本には合っている考え方なのかな、と感じています。
    0017.org	0017.org	0.502	Go to the original video hierarchy of the conversion source, copy and paste the following is fine. ffmpeg -i sample.mp4 -strict -2 video.webm summary I’ve been using the upload and embed method to Youtube to set up videos on the web.	ffmpeg -i sample.mp4 -strict -2 video.webm まとめ Web上で動画を設置するときは、Youtubeにアップして埋め込む方法ばかり使っていましたが、これで複数の動画形式を作ることができるので、自分のサーバに設定することも可能になりますね。



```python
!wc -l en-ja.bicleaner05.txt
```

    25740835 en-ja.bicleaner05.txt


英文と和文だけを取り出します。


```python
!cat en-ja.bicleaner05.txt | cut -f4 > en.txt
!cat en-ja.bicleaner05.txt | cut -f5 > ja.txt
```

## 文をトークン化

次に学習用に文章をトークン化します。

トークン化には[MeCab](https://taku910.github.io/mecab/)等が用いられることも多いのですが、ここでは、[SentencePiece](https://github.com/google/sentencepiece)を使ってトークン化してみます。

まず、Pythonのパッケージをインストールします。


```python
!pip install sentencepiece
```

全部の文章から単語分割モデルを学習しようとすると、筆者のPCのメモリ(8GB)では足りなかったので、一部の文(100,000文)から学習しました。

PCの環境によっては文数を増やしてもよいかもしれません。

まず最初に、英文、和文の中身の一部をシャッフルして取り出します。


```python
!shuf -n 100000 en.txt -o vocab_train.en
!shuf -n 100000 ja.txt -o vocab_train.ja
```

取り出した文で学習します。

character_coverageは、英語の場合は公式サイトで例示されている値、日本語の場合は一般的によいと言われている値です。


```python
import sentencepiece as spm

spm.SentencePieceTrainer.Train(
   '--input=vocab_train.en --model_prefix=sentencepiece_en --vocab_size=32000 --character_coverage=0.98'
)
spm.SentencePieceTrainer.Train(
   '--input=vocab_train.ja --model_prefix=sentencepiece_ja --vocab_size=32000 --character_coverage=0.9995'
)
```

学習したモデルを使って、試しにトークン化してみます。

英語の場合は空白で区切っただけのように見えますが、日本語の場合は文法通りではなく独自に学習しているようです。


```python
sp_en = spm.SentencePieceProcessor("sentencepiece_en.model")
print(sp_en.encode("It will return to non-display when content update is done.", out_type=str))
```

    ['▁It', '▁will', '▁return', '▁to', '▁non', '-', 'display', '▁when', '▁content', '▁update', '▁is', '▁done', '.']



```python
sp_ja = spm.SentencePieceProcessor("sentencepiece_ja.model")
print(' '.join(sp_ja.encode("スポンサードリンク この広告は一定期間更新がない場合に表示されます。", out_type=str)))
```

    ▁ スポンサー ド リンク ▁この 広告 は 一定 期間 更新 がない場合 に表示されます 。


コーパスのデータとトークン化したファイルを作成します。

このファイルを使って、翻訳モデルを作成します。


```python
with open("./en.txt") as in_f:
    with open("en_tokenized.txt", mode='w') as out_f:
        for line in in_f:
            out_f.write(' '.join(sp_en.encode(line, out_type=str)) + "\n")
```


```python
with open("./ja.txt") as in_f:
    with open("ja_tokenized.txt", mode='w') as out_f:
        for line in in_f:
            out_f.write(' '.join(sp_ja.encode(line, out_type=str)) + "\n")
```

トークン化したら、そのファイルの一部(25730000文)を訓練用、5000文を検証用、5000文をテスト用に分けます。

コーパスファイルの頭から何万文と言うように分けると、取得元のドメインが偏るのでシャッフルしてデータを分けるようにします。

まず最初にコーパスの文数までの数値がランダムにならんだファイルを作成します。


```python
!seq `wc -l en-ja.bicleaner05.txt | cut -f1 -d' '` | shuf -o line_nums.txt -
```


```python
訓練用、検証用、テスト用の文に使うデータの行番号が入ったファイルを作成します。
```


```python
!head --lines=25730000 line_nums.txt | sort --numeric-sort > train_line_nums.txt
!head --lines=25735000 line_nums.txt | tail --lines=5000 | sort --numeric-sort > val_line_nums.txt
!head --lines=25740000 line_nums.txt | tail --lines=5000 | sort --numeric-sort > test_line_nums.txt
```

文のファイル名と、行番号のファイル名から、行番号の文を取り出す関数を定義して、訓練用、検証用、テスト用のファイルを作成します。


```python
# input_file: 取り出し元ファイル
# num_fileの行番号の行だけ取り出します。
# output_file: 取り出した結果ファイル
def extract_lines(input_file, num_file, output_file):
    text_line_num = 1
    with open(num_file) as line_f:
        line_num = int(line_f.readline())
        with open(input_file) as in_f:
            with open(output_file, mode='w') as out_f:
                for line in in_f:
                    if text_line_num == line_num:
                        out_f.write(line)
                        line_num = line_f.readline()
                        if line_num == '':
                            break
                        else:
                            line_num = int(line_num)
                    text_line_num += 1
```


```python
extract_lines("en_tokenized.txt", "train_line_nums.txt", "train.en")
extract_lines("ja_tokenized.txt", "train_line_nums.txt", "train.ja")
extract_lines("en_tokenized.txt", "val_line_nums.txt", "val.en")
extract_lines("ja_tokenized.txt", "val_line_nums.txt", "val.ja")
extract_lines("en_tokenized.txt", "test_line_nums.txt", "test.en")
extract_lines("ja_tokenized.txt", "test_line_nums.txt", "test.ja")
```

試しに日本語の訓練データを見てみましょう。


```python
!head train.ja
```

    ▁ 鼻 ・ 口 の ところ は あらかじめ 少し 切 って お く と いい ですね 。
    ▁ アドレス 置 い と く ので 、 消 されない うち に メール く れた ら 嬉しい です 。
    ▁ スポンサー ド リンク ▁この 広告 は 一定 期間 更新 がない場合 に表示されます 。
    ▁また 、 ▁ プレミアム ユーザー ▁ になると 常に 非表示 になります 。
    ▁ コンテンツ の 更新 が 行われる と 非表示 に戻ります 。
    ▁ スポンサー ド リンク ▁この 広告 は 一定 期間 更新 がない場合 に表示されます 。
    ▁また 、 ▁ プレミアム ユーザー ▁ になると 常に 非表示 になります 。
    ▁ コンテンツ の 更新 が 行われる と 非表示 に戻ります 。
    ▁You tu be を中心に ミニ マ リスト と言って いる 方 の 動画 を たくさん み ました が 、 納 得 いく ものの も 多く 、 不 況 と 少 子 化 を 呼ばれ る ” 今 の ” 日本 には 合 っている 考え方 なのか な 、 と 感じ ています 。
    ▁f f mp e g ▁- i ▁ sa mp le . mp 4 ▁- str ic t ▁ -2 ▁ v ide o . web m ▁ まとめ ▁Web 上で 動画 を 設置 するとき は 、 Y out ub e に アップ して 埋め 込む 方法 ばかり 使 っていました が 、 これ で 複数の 動画 形式 を作る ことができる ので 、 自分の サーバ に 設定 することも 可能 になります ね 。

```python
%cd ../..
```

## OpenNMT-pyで学習

データの準備ができたので、OpenNMT-pyで学習させます。

まずはOpenNMT-pyをインストールします。


```python
!pip install OpenNMT-py
```

以下のようなYamlファイルを作成します。

```yml:vocab_en_ja.yml
save_data: data/en-ja/jparacrawl
## Where the vocab(s) will be written
src_vocab: data/en-ja/jparacrawl.vocab.src
tgt_vocab: data/en-ja/jparacrawl.vocab.tgt
# Prevent overwriting existing files in the folder
overwrite: False

# Corpus opts:
data:
    corpus_1:
        path_src: data/en-ja/train.en
        path_tgt: data/en-ja/train.ja
    valid:
        path_src: data/en-ja/val.en
        path_tgt: data/en-ja/val.ja

```

ボキャブラリー(語彙)ファイルを作成します。

```python
!onmt_build_vocab -config src/bocab_en_ja.yml -n_sample 10000
```

学習に本当に必要なデータだけ別ディレクトリに集めます。

```bash
mkdir -p data_train/en_ja
cp data/en_ja/jparacrawl.vocab.* data_train/en_ja
cp data/en-ja/train.* data_train/en_ja
cp data/en-ja/val.* data_train/en_ja
```

以下のような学習パラメータのyamlファイルを作成します。
```yml
save_data: /opt/ml/input/data/training/en-ja/jparacrawl
## Where the vocab(s) will be written
src_vocab: /opt/ml/input/data/training/en-ja/jparacrawl.vocab.src
tgt_vocab: /opt/ml/input/data/training/en-ja/jparacrawl.vocab.tgt
# Prevent overwriting existing files in the folder
overwrite: False

# Corpus opts:
data:
    corpus_1:
        path_src: /opt/ml/input/data/training/en-ja/train.en
        path_tgt: /opt/ml/input/data/training/en-ja/train.ja
    valid:
        path_src: /opt/ml/input/data/training/en-ja/val.en
        path_tgt: /opt/ml/input/data/training/en-ja/val.ja

#data: /opt/ml/model/dataset.en_ja
save_model: /opt/ml/model/model.en-ja
save_checkpoint_steps: 10000
keep_checkpoint: 10
seed: 3435
train_steps: 40000
valid_steps: 10000
warmup_steps: 8000
report_every: 100

decoder_type: transformer
encoder_type: transformer
word_vec_size: 512
rnn_size: 512
layers: 6
transformer_ff: 2048
heads: 8

accum_count: 2
optim: adam
adam_beta1: 0.9
adam_beta2: 0.998
decay_method: noam
learning_rate: 2.0
max_grad_norm: 0.0

batch_size: 4096
batch_type: tokens
normalization: tokens
dropout: 0.1
label_smoothing: 0.1

max_generator_batches: 2

param_init: 0.0
param_init_glorot: 'true'
position_encoding: 'true'

world_size: 4
gpu_ranks:
- 0
- 1
- 2
- 3
```

以下のような学習コマンドを呼び出すシェルスクリプトを作成します。

```bash
#!/usr/bin/env bash

onmt_train -config train_en_ja.yml
```

以下のようなDockerファイルを作成します。
```docker
FROM nvidia/cuda:11.7.1-runtime-ubuntu20.04

RUN apt-get -y update
RUN apt-get -y install python3 pip

RUN pip --no-cache-dir install OpenNMT-py

ENV PYTHONUNBUFFERED=TRUE
ENV PYTHONDONTWRITEBYTECODE=TRUE
ENV PATH="/opt/program:${PATH}"

COPY src /opt/program
WORKDIR /opt/program
```

以下のようなスクリプトを使ってDockerイメージを作成し、Amazon Elastic Container Registry(ECR)にイメージをpushします。

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

```python
!./build_and_push.sh jparacrawl-train
```

作成しておいたIAM ロールを取得します。

```python
import boto3

role_name = "SageMaker-local"

iam = boto3.client("iam")
role = iam.get_role(RoleName=role_name)["Role"]["Arn"]
```

SageMakerのセッションを開始します。


```python
import sagemaker as sage

sess = sage.Session()
```

学習に使うデータをS3にアップロードします。

```python
prefix = 'jparacrawl/training'
WORK_DIRECTORY = 'data_train'
data_location = sess.upload_data(WORK_DIRECTORY, key_prefix=prefix)
```

データのアップロードが終了したら、SageMakerのコンテナで学習を開始します。

```python
account = sess.boto_session.client('sts').get_caller_identity()['Account']
region = sess.boto_session.region_name
image = '{}.dkr.ecr.{}.amazonaws.com/jparacrawl-train:latest'.format(account, region)

estimator = sage.estimator.Estimator(image,
                       role, 1, 'ml.g4dn.12xlarge',
                       output_path="s3://{}/jparacrawl/output".format(sess.default_bucket()),
                       sagemaker_session=sess)

estimator.fit({"training": data_location})
```

    2022-09-27 12:26:48 Starting - Starting the training job...
    2022-09-27 12:27:11 Starting - Preparing the instances for trainingProfilerReport-1664281607: InProgress
    ......
    2022-09-27 12:28:21 Downloading - Downloading input data.................
    (中略)
    [34m[2022-09-27 21:52:40,044 INFO] Validation accuracy: 55.978[0m
    [34m[2022-09-27 21:52:40,087 INFO] Saving checkpoint /opt/ml/model/model.en-ja_step_40000.pt[0m
    
    2022-09-27 21:52:49 Uploading - Uploading generated training model
    2022-09-27 21:58:06 Completed - Training job completed
    Training seconds: 34186
    Billable seconds: 34186


学習には大体9時間強(33,620 sec)かかりました。その分費用も…

学習結果は、S3に `s3://sagemaker-{リージョン名}-{アカウントID}/jparacrawl/output/jparacrawl-train-{トレーニング日時}/output/model.tar.gz` のURIで出力されます。

今回学習したモデルを使って、[こちらの記事](https://zenn.dev/thorie/articles/548nl_ctranslate2_setup)にあるような形で翻訳する事ができます。ただし、翻訳のスクリプトは以下のようになります。


```python:translate.py
#!/usr/bin/env python3

import ctranslate2
import sentencepiece as spm
import sys

translator = ctranslate2.Translator("ctranslate2_model", device="cpu")
sp_en = spm.SentencePieceProcessor("sentencepiece_en.model")
sp_ja = spm.SentencePieceProcessor("sentencepiece_ja.model")

input_text = sys.argv[1]

input_tokens = sp_en.encode(input_text, out_type=str)

results = translator.translate_batch([input_tokens])

output_tokens = results[0].hypotheses[0]
output_text = sp_ja.decode(output_tokens)

print(output_text)
```

いくつか翻訳してみた結果は以下の通りです。もう少し、まともな訳が出るかと期待していたのですが… 残念。

| 原文 | 訳文 |
| ---- | ---- |
| Hello, world! |  ⁇ 、世界! |
| It is fine today. | 今日は大丈夫です。 |
| The average viewership in the Kanto region was gauged over an hour from 3 p.m. | 九州地方の平均視聴率は ⁇ 3時から1時間にわたって ⁇ されました。 |
| We are in touch with Hilaree’s family and supporting search and rescue efforts in every way we can. | ヒラリーファミリに連絡を取り、あらゆる方法で検索と救 ⁇ 活動をサポートしています。 |

