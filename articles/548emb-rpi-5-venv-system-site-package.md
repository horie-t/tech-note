---
title: "Raspberry Pi 5上でシステムのパッケージを利用する仮想環境を構築する"
emoji: "🙆"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["raspberrypi", python]
published: true
---

Raspberry Pi OS(bookwormベース)から、OSが使用しているPythonの環境にpipのようなパッケージ管理ツールでPythonのパッケージをインストールするのは非推奨になりました。そのため、仮想環境を構築してその環境にパッケージをインストールする必要があります。

ですが、仮想環境を作成すると通常はOSのPythonにはあるパッケージが利用できなくなります。Raspberry Piに特有のハードウェアを制御するようなパッケージはOSのパッケージとして提供されaptコマンド等を使ってOSのPythonの環境にインストールされます。つまり、ハードウェアを制御するPythonのプログラムを書こうとすると、aptコマンドでインストールできる限られた種類のパッケージしか利用できなくなります。

本記事では、この制限を回避する方法についてuvを使った例を記述します。(uvはRustで書かれた高速なパッケージ・プロジェクト管理ツールです)

## 前提条件

uvのバージョン0.6.8以降をインストールしてください。

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

古いバージョンのuvを使っている場合はアップデートしてください。

```bash
uv self update
```

## プロジェクトの作成

uvの管理化にある(つまりuvでインストールした)Pythonは使用しないように指定して、OSのPythonを使用するプロジェクトを作成します。(OSのPythonと違うバージョンのPythonを使用すると、パッケージが正常に動かなくなる場合があるためOSのPythonを使用します)

```bash
uv init sample-prj --no-managed-python
```

次にプロジェクト内の `pyproject.toml` ファイルに以下の記述を追加します。

```toml
[tool.uv]
python-preference = "only-system"
```

## 仮想環境の構築

仮想環境からOSのPython環境のパッケージを使う事ができるように仮想環境を作成します。

```bash
uv venv --system-site-packages
```

なお、仮想環境の構成ファイルは以下の例のようになりました。

```conf
$ cat .venv/pyvenv.cfg 
home = /usr/bin
implementation = CPython
uv = 0.6.14
version_info = 3.11.2
include-system-site-packages = true
prompt = uv-sys-python
```