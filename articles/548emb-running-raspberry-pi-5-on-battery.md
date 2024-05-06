---
title: "Raspberry Pi 5をバッテリーで稼働させる"
emoji: "✨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["raspberrypi"]
published: true
---

Raspberry Pi 5の電源の要件は、USB Type-Cコネクタ、5V/5A(Power Delivery)となっています。2024年5月現在では、この要件を満たすモバイルバッテリーは日本では市販されていないようです。
そのため、12Vのバッテリーから5Vに電圧を下げて、40ピンヘッダーから電源を供給して動かします。

## 使用ハードウェア

* [Raspberry Pi 5(8GB)](https://amzn.to/3JHBZtJ)
* [マイクロSDカード 64GB](https://amzn.to/4bgQPmT)
* [降圧コンバータモジュール24V/12V〜5V 5A](https://amzn.to/3JOahvB)
* [LiPoバッテリー](https://amzn.to/3QvTPnu)もしくは[12V モバイルバッテリーでDCコネクタがついているもの](https://www.amazon.co.jp/12V-%E3%83%A2%E3%83%90%E3%82%A4%E3%83%AB%E3%83%90%E3%83%83%E3%83%86%E3%83%AA%E3%83%BC/s?k=12V+%E3%83%A2%E3%83%90%E3%82%A4%E3%83%AB%E3%83%90%E3%83%83%E3%83%86%E3%83%AA%E3%83%BC)

## 構成図

下図のように接続します。

![](https://raw.githubusercontent.com/horie-t/omni-mouse/main/docs/images/RunOnBattery.svg)

## EEPROMの設定変更

起動後に以下の警告が表示されます。

> This power supply is not capable of supplying 5A
> 
> Power to peripherals will be restricted

以下のコマンドを実行して、電源の電流供給能力が5Aあるとみなすようにさせます。

```
sudo rpi-eeprom-config -e
```

コマンドの編集画面で以下の行を追加します。

```
PSU_MAX_CURRENT=5000
```