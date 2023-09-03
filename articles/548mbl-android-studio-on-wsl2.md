---
title: "WSL2上でAndroid StudioでAndroidアプリケーションを、usbipd-winを使って快適に開発する"
emoji: "😎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["android"]
published: false
---

WSLにWindows Subsystem for Linux GUI(WSLg)の仕組みが追加されてから、WSL上でIDE等のGUIアプリケーションを動かしやすくなりました。

しかし、Androidアプリを開発する場合、USBで接続したAndroidの実機を使ってデバッグしたくなります。本記事では[usbipd-win](https://github.com/dorssel/usbipd-win)を使ってWSL2上のUbuntu 22.04にAndroidデバイスを認識させ、WSL2で動作するAndroid Studioから直接アプリケーションをインストール・実行する方法を紹介します。

## usbipd-winのインストール

usbipd-winをインストールするには、予め[winget](https://apps.microsoft.com/store/detail/%E3%82%A2%E3%83%97%E3%83%AA-%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%A9%E3%83%BC/9NBLGGH4NNS1?hl=ja-jp&gl=jp&rtc=1)をインストールしておきます。wingetをインストール後に以下のコマンドを実行して、PowerShell等でusbipd-winをインストールします。

```powershell
PS C:\Users\horie-t> winget install usbipd
```

WSL上のUbuntuで以下のコマンドを実行して、ツール等をインストールします。

```bash
sudo apt install linux-tools-virtual hwdata
sudo update-alternatives --install /usr/local/bin/usbip usbip `ls /usr/lib/linux-tools/*/usbip | tail -n1` 20
```

## Android Studioのインストール・セットアップ

Android Studioの[ダウンロードサイト](https://developer.android.com/studio)からアプリケーションをダウンロードし、インストール先ディレクトリで以下の例のように展開する。

```bash
tar xvf ~/Downloads/android-studio-2022.3.1.19-linux.tar.gz
```

Android デバイスに対応する udev ルールがシステムにインストールされている必要があるので、以下のコマンドを実行して、インストール・セットアップします。

```bash
sudo apt-get install -y android-sdk-platform-tools-common && sudo cp /lib/udev/rules.d/51-android.rules /etc/udev/rules.d/
```

adb を使用するユーザーは、plugdev グループに属している必要があるので、以下のコマンドを実行してグループに追加します。

```bash
sudo usermod -aG plugdev $LOGNAME
```

セットアップ完了後に一度WSLを停止して再起動する必要があるので、PowerShell等で以下のコマンドを実行します。

```powershell
PS C:\Users\horie-t> wsl --shutdown
```

## WSLにAndroidデバイスをアタッチして、Android Studioからアプリケーションを起動する

Android デバイスをPCにUSBで接続します。(Android側で開発者向けオプションを有効にしていない場合は[こちら](https://developer.android.com/studio/debug/dev-options?hl=ja)を参考に設定しておきます)

接続後、PowerShell等で以下のコマンドを実行してUSBデバイスをリストアップします。以下の例では `BUSID` が `1-3` でPixel 4aが認識されています。

```powershell
PS C:\Users\horie-t> usbipd.exe wsl list
BUSID  VID:PID    DEVICE                                                        STATE
1-3    18d1:4ee7  Pixel 4a                                                      Not attached
1-5    27c6:639c  Goodix MOC Fingerprint                                        Not attached
1-6    0c45:6a1b  Integrated Webcam                                             Not attached
1-10   8087:0033  インテル(R) ワイヤレス Bluetooth(R)                           Not attached
2-1    0b95:1790  ASIX AX88179 USB 3.0 to Gigabit Ethernet Adapter #2           Not attached
4-2    04fe:000d  USB 入力デバイス                                              Not attached
4-3    056e:0101  USB 入力デバイス                                              Not attached
5-1    12d1:10b6  USB 入力デバイス, MateView GT                                 Not attached
5-2    10c4:92ca  USB 入力デバイス                                              Not attached
```

デバイスのBUSIDが確認できたら、以下のコマンド実行してデバイスをWSLにアタッチします。

```powershell
usbipd.exe wsl attach --busid=<BUSID>

# 例
PS C:\Users\horie-t> usbipd.exe wsl attach --busid=1-3
```

WSL上でAndroid Studioのインストール先ディレクトリに移動して、Android Studioを以下のように起動します。

```bash
$ android-studio/bin/studio.sh
```

起動後に、以下のようにAndroid StudioのDevice ManagerにAndroidデバイスが認識されている事が分かります。

![](https://github.com/horie-t/tech-note/blob/master/images/wsl-android-studio.png?raw=true)
