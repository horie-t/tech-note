---
title: "Raspberry Pi 5からL6470モータードライバ経由でステッピングモーターを駆動させる"
emoji: "💭"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["raspberrypi"]
published: true
---

[パソコン並の性能を持つ](https://xtech.nikkei.com/atcl/nxt/column/18/02756/022600001/)と言われるRaspberry Pi 5ですが、RP1と呼ばれるI/Oコントローラーが追加されました。これに伴い[GPIOの番号に互換性がなかった](https://xtech.nikkei.com/atcl/nxt/column/18/01109/040700054/)りして、良く使われてきたドライバーが動かなかったりします。ここでは、秋月電子の[L6470使用 ステッピングモータードライブキット](https://akizukidenshi.com/catalog/g/g107024/)を使ってモーターを動かしながらSPI通信の方法をみていきます。

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

Pythonで制御するために、[公式サイトで紹介](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#using-spidev-from-python)されている[spidev](https://pypi.org/project/spidev/)を使用します。インストールされていない場合は以下のコマンドを実行してインストールします。

```bash
pip install spidev
```

次に、以下のプログラムを作成し実行します。

```python:L6470_motor_drive.py
import spidev
import time

# SPIの初期化
spi = spidev.SpiDev()
spi.open(0, 0)
spi.bits_per_word = 8
spi.cshigh = False
spi.loop = True
spi.no_cs = False
spi.lsbfirst = False
spi.max_speed_hz = 4000000
spi.mode = 3
spi.threewire = False

# しばらく何もしない(NOP)
spi.xfer([0x00])
spi.xfer([0x00])
spi.xfer([0x00])
spi.xfer([0x00])

# HOMEポジションへ
spi.xfer([0x70])

# 最大回転速度設定
spi.xfer([0x07])
spi.xfer([0x20])

# モータ停止中の電圧
spi.xfer([0x09])
spi.xfer([0xFF])

# モータ定速回転中の電圧
spi.xfer([0x0A])
spi.xfer([0xFF])

# モータ加速中の電圧
spi.xfer([0x0B])
spi.xfer([0xFF])

# モータ減速中の電圧
spi.xfer([0x0C])
spi.xfer([0xFF])

# ステップモード
spi.xfer([0x16])
spi.xfer([0x00])

time.sleep(3)

while True:
    # 目標速度で正転させる
    spi.xfer([0x50])
    # NOP
    spi.xfer([0x00])
    # 回転速度を設定
    spi.xfer([0x20])
    # NOP
    spi.xfer([0x00])

    # 1秒回転
    time.sleep(1)

    # 停止
    spi.xfer([0xB8])
    # 1秒停止
    time.sleep(1)
```
