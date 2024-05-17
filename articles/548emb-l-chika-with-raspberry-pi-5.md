---
title: "Raspberry Pi 5でgpiozeroを使ってLチカ"
emoji: "🙌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["raspberrypi"]
published: true
---

Raspberry PiでLチカをやる時は、[pigpio](https://abyz.me.uk/rpi/pigpio/) と言うライブラリを使う事が多いのですが、[Rapberry Pi 5では pigpio が使えなく](https://github.com/joan2937/pigpio/issues/589)なりました。

代わりに[gpiozero](https://gpiozero.readthedocs.io/en/latest/)と言うライブラリでLチカをしてみます。

## 準備

下図のようにLEDと抵抗(330Ω)を接続します。Raspberry Piのピンは22番(GPIO 25)と39番(GND)を使用します。

![](https://raw.githubusercontent.com/horie-t/omni-mouse/main/docs/images/LChika_Breadboard.svg)

## 実行

以下のプログラムを作成し、実行します。

```python:l_chika.py
from gpiozero import LED
from time import sleep

led = LED(25)
led.on()

while True:
    sleep(1)
    led.toggle()
```