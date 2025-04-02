---
title: "Picamera2で魚眼レンズカメラ（Arducam 8MP IMX219）のカメラレンズシェーディング補正をする"
emoji: "🐷"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["raspberrypi"]
published: true
---

Raspberry Pi 5に[Raspberry Pi用魚眼レンズカメラモジュール（8MP / IMX219）](https://www.switch-science.com/products/8245?variant=42382205223110)を接続して、色ムラ等の補正をして撮影をしました。

最初に[以前の記事](./548emb-rpi-5-camera_lens_shading_calibration)の通りカメラを接続して、セットアップします。

次に公式サイトで公開してされている[チューニングファイル](https://docs.arducam.com/Raspberry-Pi-Camera/Native-camera/Lens-Shading-json-file/#imx219-camera_1)から、[160°のファイル](https://www.arducam.com/wp-content/uploads/2024/11/pi5_imx219_160d.json)を `/usr/share/libcamera/ipa/rpi/pisp` にコピーします。

```bash
sudo cp pi5_imx219_160d.json /usr/share/libcamera/ipa/rpi/pisp
```

次に以下のコード実行します。

```python:camera_fish_eye.py
from picamera2 import Picamera2, Preview

tuning = Picamera2.load_tuning_file("pi5_imx219_160d.json")

picam2 = Picamera2(tuning=tuning)
picam2.start_preview(Preview.NULL)
picam2.start_and_capture_file("fish_eye.jpg")
```

以下のような画像が撮影できます。

![](/images/RaspberryPiFishEyeCamera.jpg)