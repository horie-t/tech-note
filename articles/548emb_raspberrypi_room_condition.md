---
title: "Raspberry Pi Zero WHにCO2センサとLCDディスプレイを繋いで室内環境を表示する"
emoji: "📌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["raspberrypi", "組み込み"]
published: true
---

マイコンボードに二酸化炭素濃度センサ等を接続して室内環境を測定する情報は数多あり、自分でもWio Terminalでやってみた事もあるのですが([*1](https://zenn.dev/thorie/articles/548dbs_timescaledb), [*2](https://zenn.dev/thorie/articles/548dbs_aws_timestream))、今回はRaspberry Piを使ってやってみました。(実際はWio Terminalを別目的で使う必要があり、代わり安くすむ方法を探してみただけです)

## 使用機器

* [Raspberry Pi Zero WH](https://www.switch-science.com/collections/raspberry-pi/products/3646)  
  2022年12月時点ではどこのショップでも売り切れなのが難点ですが、たまにAmazonで良心的な価格で販売されている事があります。
* [Grove - SCD30搭載 CO2・温湿度センサ（Arduino用）](https://www.switch-science.com/products/7000)
* [Hailege 2.4" ILI9341 240x320 SPI TFT LCDディスプレイ2.4インチタッチパネルLCD 5V / 3.3V](https://www.amazon.co.jp/dp/B08C5D8ZCQ/)  
  ILI9341を使っているLCDディスプレイならなんでも大丈夫だと思います。

## 前提条件

[Raspberry Pi Zero Wのセットアップ](https://zenn.dev/thorie/articles/548emb_raspberrypi_setup)が完了している事。

## センサとディスプレイの接続

以下の図のように、CO2センサとLCDディスプレイを接続します。

![](https://raw.githubusercontent.com/horie-t/tech-note/master/images/RaspberryPiRoomCondition_Breadboard.png)

## Raspberry Piの設定変更

COセンサのSDC30とは[I2C](https://ja.wikipedia.org/wiki/I2C)、ディスプレイのILI9341とは[SPI](https://ja.wikipedia.org/wiki/%E3%82%B7%E3%83%AA%E3%82%A2%E3%83%AB%E3%83%BB%E3%83%9A%E3%83%AA%E3%83%95%E3%82%A7%E3%83%A9%E3%83%AB%E3%83%BB%E3%82%A4%E3%83%B3%E3%82%BF%E3%83%95%E3%82%A7%E3%83%BC%E3%82%B9)を使って、通信するのでRaspberry Pi側でI2CとSPIを有効化します。

Raspberry PiにSSHでログインして以下のコマンドを実行します。

```bash
sudo raspi-config
```

![](https://raw.githubusercontent.com/horie-t/tech-note/master/images/RaspberryPiConfig.png]


`Interface Options`を選択します。

![](https://raw.githubusercontent.com/horie-t/tech-note/master/images/RaspberryPiConfigInterface.png')


`SPI`と`I2C`を選択し、enabled?の問合せに`Yes`を選択します。

## パッケージの追加

python3はインストールされていたので、pip3のみインストールします。

```bash
sudo apt-get -y install python3-pip
```

I2C用にsmbus2をインストールします。

```bash
sudo pip install smbus2
```

SCD30用のライブラリをインストールします。

```bash
sudo pip install scd30-i2c
```

液晶コントローラLSIのILI9341用にAdafruit_CircuitPython_RGB_Displayをインストールします。

```bash
sudo pip install adafruit-circuitpython-rgb-display
```

結果を日本語で表示したいので日本語フォントをインストールします。

```bash
sudo apt-get -y install fonts-ipaexfont
```

## SCD30の動作確認

SCD30の動作確認として、測定した結果を標準出力に出力するだけのプログラムを実行してみます。

まず、以下の`scd_sample.py`のファイルを作成します。

```python:scd_sample.py
#!/usr/bin/env python3

import time
from scd30_i2c import SCD30

scd30 = SCD30()

scd30.set_measurement_interval(2)
scd30.start_periodic_measurement()

time.sleep(2)

while True:
    if scd30.get_data_ready():
        m = scd30.read_measurement()
        if m is not None:
            print(f"CO2: {m[0]:.2f}ppm, temp: {m[1]:.2f}'C, rh: {m[2]:.2f}%")
        time.sleep(2)
    else:
        time.sleep(0.2)
```

ファイルに実行属性を追加します。

```bash
chmod +x scd30_sample.py
```

実行してみましょう。筆者の室内は、結構CO2濃度が高い状態でした。

```bash
pi@raspberrypi:~ $ ./scd30_sample.py 
CO2: 1040.33ppm, temp: 23.38'C, rh: 40.58%
CO2: 1039.25ppm, temp: 23.38'C, rh: 40.56%
CO2: 1040.45ppm, temp: 23.36'C, rh: 40.63%
```

## LCDディスプレイに測定値を表示

ディスプレイと標準出力に測定値を表示してみましょう。

以下のファイルを作成します。

```python:/home/pi/measure_room_condition.py
#!/usr/bin/env python3

import time

import digitalio
import board
from PIL import Image, ImageDraw, ImageFont
from adafruit_rgb_display import ili9341

from scd30_i2c import SCD30

def init_display():
    """Initialize display, and return display object"""
    # Configuration for CS and DC pins (these are PiTFT defaults):
    cs_pin = digitalio.DigitalInOut(board.CE0)
    dc_pin = digitalio.DigitalInOut(board.D25)
    reset_pin = digitalio.DigitalInOut(board.D24)

    # Config for display baudrate (default max is 24mhz):
    BAUDRATE = 24000000

    # Setup SPI bus using hardware SPI:
    spi = board.SPI()

    # pylint: disable=line-too-long
    # Create the display:
    disp = ili9341.ILI9341(
        spi,
        rotation=0,  # 2.2", 2.4", 2.8", 3.2" ILI9341
        cs=cs_pin,
        dc=dc_pin,
        rst=reset_pin,
        baudrate=BAUDRATE,
    )

    return disp

disp = init_display()

height = disp.height
width = disp.width

image = Image.new("RGB", (width, height))

# Get drawing object to draw on image.
draw = ImageDraw.Draw(image)

# Draw a black filled box to clear the image.
draw.rectangle((0, 0, width, height), outline=0, fill=(0, 0, 0))
disp.image(image)


# Load a ipaexfont-gothic font.
font1 = ImageFont.truetype("/usr/share/fonts/opentype/ipaexfont-gothic/ipaexg.ttf", 70)
font2 = ImageFont.truetype("/usr/share/fonts/opentype/ipaexfont-gothic/ipaexg.ttf", 55)
font3 = ImageFont.truetype("/usr/share/fonts/opentype/ipaexfont-gothic/ipaexg.ttf", 25)
font4 = ImageFont.truetype("/usr/share/fonts/opentype/ipaexfont-gothic/ipaexg.ttf", 22)
font5 = ImageFont.truetype("/usr/share/fonts/opentype/ipaexfont-gothic/ipaexg.ttf", 17)
font6 = ImageFont.truetype("/usr/share/fonts/opentype/ipaexfont-gothic/ipaexg.ttf", 12)

text_color1 = "#D9E5FF"

scd30 = SCD30()

measurment_interval_sec = 10
scd30.set_measurement_interval(measurment_interval_sec)
scd30.start_periodic_measurement()

time.sleep(2)

day_of_week = ["月", "火", "水", "木", "金", "土", "日"]

while True:
    if not scd30.get_data_ready():
        time.sleep(0.2)
        continue

    m = scd30.read_measurement()
    if m is None:
        time.sleep(measurment_interval_sec)
        continue

    local_time = time.localtime()
    current_date = '{}月{}日({})'.format(local_time.tm_mon, local_time.tm_mday, day_of_week[local_time.tm_wday])
    current_time = time.strftime('%H:%M', local_time)
    print(time.strftime('%Y-%m-%dT%H:%M:%S%z', local_time) + "," + f"{m[0]:.1f},{m[1]:.1f},{m[2]:.1f}")
    
    # Draw a black filled box to clear the image.
    draw.rectangle((0, 0, width, height), outline=0, fill=(0, 0, 0))

    # date and time
    draw.text((width / 2, 15), current_date, align='center', anchor="ma", font=font3, fill=text_color1)
    draw.text((width / 2, 55), current_time, align='center', anchor="ma", font=font1, fill=text_color1)
    
    # Temperature
    temp =format(float(m[1]), '.1f')
    draw.text((195, 145), temp, align='right', anchor='rt', font=font2, fill=text_color1)
    draw.text((197, 145), "℃", font=font3, fill=text_color1)

    # Concentration of CO2
    co2 =format(float(m[0]), '.1f')
    draw.text((195, 205), co2, align='right', anchor='rt', font=font2, fill=text_color1)
    draw.text((197, 205), "ppm", font=font5, fill=text_color1)
    draw.text((197, 230), "CO", font=font5, fill=text_color1)
    draw.text((225, 235), "2", font=font6, fill=text_color1)

    # Humidity
    humi =format(float(m[2]), '.1f')
    draw.text((195, 265), humi, align='right', anchor='rt', font=font2, fill=text_color1)
    draw.text((197, 265), "%", font=font4, fill=text_color1)
    draw.text((197, 288), "RH", font=font4, fill=text_color1)

    disp.image(image)

    time.sleep(measurment_interval_sec)
```

ファイルに実行属性を追加します。

```bash
chmod +x measure_room_condition.py
```

実行してみましょう。

```bash
pi@raspberrypi:~ $ ./measure_room_condition.py 
2022-12-07T18:45:09+0900,1070.6,21.6,44.4
2022-12-07T18:45:19+0900,1087.3,21.6,44.3
2022-12-07T18:45:29+0900,1077.4,21.6,44.2
```

測定の様子は以下のようになります。

![](https://raw.githubusercontent.com/horie-t/tech-note/master/images/RaspberryPiRoomCondition.jpg)

## 自動起動設定

OS起動時に自動で起動するように設定してみましょう。

起動スクリプトを以下の通り作成します。

```shell:/etc/init.d/measure_room_condion
#! /bin/sh

### BEGIN INIT INFO
# Provides:             scd30d
# Required-Start:       $remote_fs $syslog
# Required-Stop:        $remote_fs $syslog
# Default-Start:        2 3 4 5
# Default-Stop:
# Short-Description:    Measure room condition
### END INIT INFO

/home/pi/measure_room_condition.py &
```

起動スクリプトをsystemdに登録して自動的起動するようにします。

```bash
cd /etc/init.d
sudo chmod +x measure_room_condion
sudo update-rc.d measure_room_condition defaults
```

次回のOS起動時から自動起動するようになります。
