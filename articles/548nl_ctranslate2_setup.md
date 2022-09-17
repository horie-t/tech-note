---
title: "CTranslate2での機械翻訳をDockerで実行する"
emoji: "👏"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["機械学習", "機械翻訳"]
published: true
---

本記事のコードは[GitHub](https://github.com/horie-t/experiment/tree/master/ctranslate2_docker)で公開しています。

## CTranslate2とは

CTranslate2は、Transformerモデルで効率的に推論するためのC++およびPythonライブラリです。このライブラリは、CPUとGPU上で実行するTransformerモデルを高速化およびメモリ使用量を削減するために、重み量子化、レイヤーフュージョン、バッチ並べ替えなどの多くのパフォーマンス最適化手法を適用するカスタムランタイムを実装しています。

Transformerモデルは、深層学習のモデルで主に自然言語処理の分野で使用されます。本記事では、このモデルを使って機械翻訳(英独のみ)を実行してみます。

## Dockerイメージの作成

以下のようなDockerファイルを作成します。
Ubuntuをベースに、Python3をインストールして、CTranslate2で必要なものをpipインストールします。

```docker
FROM ubuntu:22.04

RUN apt-get -y update
RUN apt-get -y install python3 pip
RUN pip install ctranslate2 OpenNMT-py sentencepiece

COPY src/translate.py /usr/bin/
WORKDIR /var/opt/ctranslate2

ENTRYPOINT [ "translate.py" ]
CMD [ "Hello world!" ]
```

翻訳を実行する`translate.py`は以下の内容です。

```python:src/translate.py
#!/usr/bin/env python3

import ctranslate2
import sentencepiece as spm
import sys

translator = ctranslate2.Translator("ctranslate2_model", device="cpu")
sp = spm.SentencePieceProcessor("sentencepiece.model")

input_text = sys.argv[1]
input_tokens = sp.encode(input_text, out_type=str)

results = translator.translate_batch([input_tokens])

output_tokens = results[0].hypotheses[0]
output_text = sp.decode(output_tokens)

print(output_text)
```

翻訳したい原文は、最初の引数で与えます。

モデルを`ctranslate2_model`ディレクトリから読み込んで(データの準備は後述)、CPUで実行するようにしています。
そして、SentencePieceProcessorを使って、原文をエンコードします。エンコードした結果を`ctranslate2.Translator`で翻訳し、結果を表示しています。

以下を実行して、Dockerイメージをビルドします。

```bash
sudo docker build -t ctranslate2:latest .
```

## モデルデータの準備

CTranslate2で使用するモデルを用意します。本記事では、英独のOpenNMT-pyのモデルが公開されているので、これをCTranslate2用に変換して使用します。

OpenNMT-pyの英独のモデルを適当なディレクトリにダウンロードします。

```bash
mkdir /tmp/ctranslate2
cd !$
wget https://s3.amazonaws.com/opennmt-models/transformer-ende-wmt-pyOnmt.tar.gz
tar xf transformer-ende-wmt-pyOnmt.tar.gz
```

コンテナを起動して、コンテナ内でモデルをOpenNMT-pyのものからCTranslate2のものに変換します。

```bash
sudo docker run --rm -it --entrypoint bash -v $PWD:/var/opt/ctranslate2 ctranslate2:latest
# 以降、コンテナ内で実行
ct2-opennmt-py-converter --model_path averaged-10-epoch.pt --output_dir ctranslate2_model
exit
```

## 翻訳を実行する

コンテナを実行し、翻訳します。

```bash
$ sudo docker run --rm -v $PWD:/var/opt/ctranslate2 ctranslate2:latest 'Hello world!'
Hallo Welt!
```
