---
title: "Raspberry Pi 5に魚眼レンズカメラモジュール（8MP / IMX219）を接続する"
emoji: "🔥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["raspberrypi"]
published: true
---

Raspberry Pi 5に[Raspberry Pi用魚眼レンズカメラモジュール（8MP / IMX219）](https://www.switch-science.com/products/8245?variant=42382205223110)を接続してみました。

## ケーブルの接続

白色の15cm 22-22pin FFC cableが付属しているので、それをRaspberry Pi 5のカメラコネクタの0の方に接続します。長いケーブルが欲しい場合はAmazon等で22pinのFFCケーブルを購入します。([30cmの場合](https://amzn.to/4laGFKm))

## セットアップ

[公式サイトの説明](https://docs.arducam.com/Raspberry-Pi-Camera/Native-camera/8MP-IMX219/#bookworm-os-pi-5)の通り、`/boot/firmware/config.txt`ファイルを編集して、カメラの自動検出をOffにして、IMX219センサを有効にします。

```txt:/boot/firmware/config.txt
# 以下のように0に変更します。デフォルト値は1
camera_auto_detect=0
# [all]の後に以下を追加します。
dtoverlay=imx219,cam0
```

## チューニングファイルのコピー

そのままの状態だと赤みがかった写真になるので、公式サイトで公開してされている[チューニングファイル](https://docs.arducam.com/Raspberry-Pi-Camera/Native-camera/Lens-Shading-json-file/#imx219-camera_1)から、[160°のファイル](https://www.arducam.com/wp-content/uploads/2024/11/pi5_imx219_160d.json)をRaspberry Piにコピーします。(製品の説明には画角175°とあるのですが、ファイルが存在しないので近いものを選択しています)

## コマンドラインからの撮影

以下のコマンドを実行します。

```bash
rpicam-jpeg -n --tuning-file pi5_imx219_160d.json --output sample.jpg
```
