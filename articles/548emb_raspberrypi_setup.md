---
title: "Raspberry Pi Zero Wのセットアップ"
emoji: "🐈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["raspberrypi"]
published: true
---
キーボードやモニタを使わずにセットアップする方法のメモです。LinuxのPCを使ってセットアップする手順になっています。

# SDカードの準備

## OSイメージの書き込み

[Raspberry Pi Imager](https://www.raspberrypi.com/software/)をダウンロードし、OSイメージ(Raspberry Pi OS Lite: Bullseye)をSDカードに書き込みます。
`Raspberry Pi Imager`の画面では`設定`で`nable SSH`にチェックし、パスワードを設定してから書き込みます。

## 設定の変更

OSイメージの書き込みが完了したら、SDカードのファイルを編集します。

SDカード内の`boot/wpa_supplicant.conf`ファイルを以下の通り編集し、WiFiルータに接続できるようにします。

```text:boot/wpa_supplicant.conf
country=JP
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
network={
    ssid="<SSID名>"
    psk="<WiFi接続パスワード>"
}
```

IPアドレスを固定する場合は、`etc/dhcpcd.conf`ファイルに以下を追記します。

```text:etc/dhcpcd.conf
:
(略)
:
interface wlan0
static ip_address=192.168.1.XXX/24
static routers=192.168.1.1
static domain_name_servers=192.168.1.1
```

# アップデートなど

SDカードをSDカードスロットに挿入して電源を投入した後で、SSHでログインしてアップデートなどを行います。

SSHでログインするには以下のようにします。

```bash
ssh -X pi@raspberrypi.local
# または
ssh pi@192.168.1.XXX
```

アップデートをするには以下のコマンドを実行します。

```bash
sudo apt update
sudo apt upgrade
```

PCからSSH鍵を使用してログインできるようにします。以下の作業はPC側で実行します。

まず、SSH鍵を作成していない場合は以下を実行します。

```bash
ssh-keygen
```

鍵を作成したら以下を実行して、公開鍵をRaspbery Pi側にコビーします。

```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub pi@192.168.1.XXX
```