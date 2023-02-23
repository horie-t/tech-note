---
title: "赛昉科技(StarFive)社製VisionFive2のセットアップ"
emoji: "💨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["riscv", "visionfive2"]
published: true
---

本記事は赛昉科技(StarFive)社製の[VisionFive2](https://amzn.to/3Ihqnhb)をセットアップしたメモです。

## VisionFive 2とは

VisionFive 2がどういう趣旨で開発されたのかは、[RISC-V Days Tokyo 2022 Autumn](https://riscv.or.jp/risc-v-days-tokyo-2022-autumn/)のプレゼンテーションの[YouTube](https://www.youtube.com/watch?v=jGp6F2X-sl0)で詳しく紹介されています。

VisionFive2の特長は以下の通りです。
* GPU搭載 RISC-Vシングルボードコンピュータの世界初の量産品
* タブレットやラップトップへの用途を意識した、JH7110 SoCを搭載
* Debian/Ubuntu/Fedora等のLinuxディストリビューションをサポート

これまでのRISC-Vのボードは組み込み向けを意識している32bit CPUを搭載しているものが一般的だが、これはデスクトップでの利用を見据えた64bit CPU(SiFive社 [U74](https://www.sifive.com/cores/u74))を搭載しています。

箱はこんな感じです。
![](https://storage.googleapis.com/zenn-user-upload/17a1afa12496-20230212.jpg)

中身を出したところ。

![](https://storage.googleapis.com/zenn-user-upload/420b9cae8ac3-20230212.jpg)

## 公式ドキュメントや参考サイト

* [RVspace Wiki Site](https://rvspace.org/)
* [StarFiveTech VisionFive2 SDK](https://github.com/starfive-tech/VisionFive2)
が公式Documentationとして上がっているので、これを見ながらセットアップしていきます。

すでに、Kickstarterで注文していた人達が[ブログ](https://www.zukeran.org/shin/d/category/visionfive2/)や[Zennの記事](https://zenn.dev/uta8a/articles/87262048da5327)を書いているので、困った時は参考になります。

## セットアップ

### ドキュメントの確認

まずは、VisonFive 2の[Wiki](https://doc-en.rvspace.org/Doc_Center/visionfive_2.html)を読みます。

(翻訳)
> VisionFive 2世界初の高性能GPU内蔵RISC-Vシングルボードコンピュータ (SBC) 。
> StarFiveでは、VisionFive 2について次のドキュメントを提供しています。
> * 製品概要
> * データシート
> * クイックスタートガイド
> * 設計回路図
> * 開発と移植ガイド
> * SDKクイックスタートガイド
> * ソフトウェアテクニカルリファレンスマニュアル
> * 40ピンGPIOヘッダーユーザーガイド
> * Pythonアプリケーションノート

とあるので、クイックスタートガイドから読んでいきます。クイックスタートガイドの[はじめに](https://doc-en.rvspace.org/VisionFive2/Quick_Start_Guide/VisionFive2_QSG/getting_started.html)をを見ながらセットアップしていきます。

### ハードウェアの準備

クイックスタートガイドでは以下が必要と記載されています。


(翻訳)

> #### 必要なハードウェア
> 次のハードウェア項目が用意されていることを確認します。
> * VisionFive 2
> * Micro-SD card (32 GB or more)
> * PC with Linux/ Windows/ MacOS
> * USB to Serial Converter
> * Ethernet cable
> * Power adapter
> * USB Type-C cable

とありますが、全部を用意する必要はありません。

まずは、
* VisionFive 2
* Micro-SD card (64 GB)
* PC with Linux
* Ethernet cable
* USB Type-C cable

を用意して、デスクトップPCっぽく使いたかったので
* HDMI cable
* USB keyboard
* USB mouse

を追加で用意して、セットアップをする事にしました。

### Micro-SDカードへのOSのフラッシュ

準備ができたら、まずOSをMicro-SDカードに書き込みます。

> #### Micro-SDカードへのOSのフラッシュ
> 次に、Debian (Linuxディストリビューション) をmicro-SDカードに焼き付けて、VisionFive 2で実行できるようにする必要があります。この章では、LinuxまたはWindowsでDebianをMicro-SDカードにフラッシュする手順の例を示します。

とのことです。この辺は、Raspbery Piと似ています。

> #### LinuxまたはWindowsでのフラッシュ
> LinuxまたはWindowsでイメージをフラッシュするには、次の手順を実行します。
> 1. micro-SDカードリーダーを介して、またはノートパソコンの内蔵カードリーダーによって、micro-SDカードをコンピュータに挿入します。
> 2. 最新のDebianイメージを[次のリンク](https://debian.starfivetech.com/)からダウンロードしてください。
> 3. `.bz2`ファイルを抽出します。
> 4. [このリンク]()からBalenaEtcherをダウンロードしてください。BalenaEtcherソフトウェアを使用して、Debianイメージをmicro-SDカードにフラッシュします。
> 5. BalenaEtcherをインストールして開きます。
> 6. [Flash from file] をクリックし、次のファイルを解凍したイメージの場所を選択します。  
   starfive-jh 7110-VF 2-<バージョン>.img
> 7. [Select target] をクリックして、接続されているmicro-SDカードを選択します。
> 8. [Flash!]をクリックしてフラッシュタスクを開始します。

の手順通りに作業してSDカードにイメージを書き込みます。

イメージは、`Image-55`と`Image-69`がありますが、`Image-69`はファームウェアのアップデートが必要なので、`Image-55`を使用する。

OSのイメージは以下のように解凍します。
```bash
$ bunzip2 starfive-jh7110-VF2-VF2_515_v2.3.0-55.img.bz2
```

BalenaEtcherはダウンロード後に、以下のように実行権限を付与して起動します。
```bash
$ chmod +x balenaEtcher-1.14.3-x64.AppImage 
$ ./balenaEtcher-1.14.3-x64.AppImage 
```

以下の画面からSDカード内にOSのイメージを書き込みます。
![](https://storage.googleapis.com/zenn-user-upload/bba00dee4a41-20230212.png)

### 起動

SDカードをボードに差し込み、USB電源ケーブル、USBキーボード、マウス、LANケーブル(内側のポートに繋ぐ)を繋いで、起動すると以下のログイン画面が表示されます。

![](https://storage.googleapis.com/zenn-user-upload/461627f14127-20230212.jpg)

クイックスタートガイドにある、rootのパスワードでログインするとデスクトップ画面が表示されるので、File Manager、uxterm、FireFoxを起動すると以下のようになります。

![](https://storage.googleapis.com/zenn-user-upload/44fa60f385e3-20230212.png)
