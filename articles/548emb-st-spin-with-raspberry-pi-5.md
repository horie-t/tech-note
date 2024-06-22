---
title: "Raspberry Pi 5からst-spinライブラリを使ってステッピングモーターを駆動させる"
emoji: "💭"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["raspberrypi"]
published: true
---

[前回](https://zenn.dev/thorie/articles/548emb-l6470-with-raspberry-pi-5)の記事では、spidevパッケージを使用してL6470モータードライバにコマンドを送信してステッピング・モーターを回転させました。この方法ではコマンドのバイナリ・コードのフォーマットやコード体系を意識してコードを読み書きする必要があり、開発効率が良くありません。今回はst-spinというパッケージを使用して、直感的にモーターを駆動させる方法を見ていきます。

## 必要なもの

* Raspberry Pi 5
* [L6470 ステッピングモーター・ドライバー](https://akizukidenshi.com/catalog/g/g107024/)
* [バイポーラ・ステッピングモータ](https://jp.misumi-ec.com/vona2/detail/221005433134/?HissuCode=SS2421-5041)
* [12V ACアダプター](https://amzn.to/3QObFSW)
* [DCジャック](https://akizukidenshi.com/catalog/g/g105148/)

## 準備

下図のようにステッピングモーター等を接続します。

![](https://raw.githubusercontent.com/horie-t/omni-mouse/main/docs/images/L6470StepperMotor.png)

## 実行

まず、st-spinパッケージをインストールします。(下記の方法はシステムのPythonにパッケージを追加します、仮想環境にインストールする場合は後述の方法でインストールします。)

```bash
sudo python -m pip install -U st-spin --break-system-packages
```

仮想環境にインストールする場合は以下のように仮想環境を作成してインストールします。

```bash
python -m venv ~/.spin_env
. ~/.spin_env/bin/activate
pip install st-spin
```

以下のプログラムを作成し実行します。

```python:st-spin_motor_drive.py
import time
from stspin import (
    SpinChain,
    Constant as StConstant,
    Register as StRegister,
)

# ディジー・チェイン(芋づる式)でモータードライバを接続できるが、1台だけ(total_devices=1)接続する
st_chain = SpinChain(total_devices=1, spi_select=(1, 0))
# 最初(と言っても1台だけだが…)のモータードライバのインスタンスを生成する
motor = st_chain.create(0)

# 200ステップ/秒で10秒間回転させる
motor.run(200)
time.sleep(10)
motor.stopSoft()
time.sleep(1)

# 逆方向に台形加速しながら500,000ステップ回転させる
motor.setRegister(StRegister.SpeedMax, 0x22)
motor.setRegister(StRegister.Acc, 0x5)
motor.setRegister(StRegister.Dec, 0x10)
motor.setDirection(StConstant.DirReverse)
motor.move(steps=500000)

# 回転しきるまで待つ
while motor.isBusy():
    print("Motor is rolling.")
    time.sleep(1)

# モーターの保持電流を止める
motor.hiZHard()
```
