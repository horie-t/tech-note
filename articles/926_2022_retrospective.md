---
title: "2022年の振り返り"
emoji: "🍣"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["振り返り", "ポエム"]
published: true
---

気が付けば2022年もわずかとなってしまいました。今年は有意義に過ごせたのだろうかと思い、2022年を振り返ってみます。

## 主にやっていた事

こうして見ると、勉強や調べものをして簡単なものを作ったりしていたけど、アウトプットと呼べるようなものはないなあ。

* **1月:** マイクロマウス[tarf](https://github.com/horie-t/tarf)の開発。
* **2月:** マイクロマウス[tarf](https://github.com/horie-t/tarf)の開発の続き。開発を一時停止。
* **3月:** 読書。
* **4月:** TimescaleDBを[調べていた](https://zenn.dev/thorie/articles/548dbs_timescaledb)。  
  Amazon Timestreamを[調べていた](https://zenn.dev/thorie/articles/548dbs_aws_timestream)。  
  [Minix3](https://www.minix3.org/)のソースコードを[読み始める](https://zenn.dev/thorie/scraps/5f602856ab84ba)。
* **5月:** Minix3のソースコード読みの続き。現在、停止中。
* **6月:** 読書。
* **7月:** 読書。
* **8月:** Reactの技術習得。  
  量子コンピュータの勉強。
* **9月:** CTranslate2を[試してみる](https://zenn.dev/thorie/articles/548nl_ctranslate2_setup)。  
  JParaCrawlを使って[翻訳モデルを構築した](https://zenn.dev/thorie/articles/548nl_sagemaker_jparacrawl)。  
  AWS SageMakerを[試してみる](https://zenn.dev/thorie/articles/548nl_sagemaker_serverless)。  
  Kubernetesの技術習得。  
  量子コンピュータの勉強の続き。
* **10月:** Whisperを[試してみる](https://zenn.dev/thorie/articles/548nl_sagemaker_whisper)。  
  PyTorchの技術習得。  
  AWS Amplifyの技術習得。  
  量子コンピュータの勉強の続き。  
  Transformersの勉強。
* **11月:** PyTorchの技術習得。  
  Transformersの勉強。  
  数学の勉強。
* **12月:** Raspberry Piで[CO2濃度を測定](https://zenn.dev/thorie/articles/548emb_raspberrypi_room_condition)してみる。  
  数学の勉強。


## 読書

近年は本を読まなくなっていたけど、今年は例年以上に本を読んでいたと思う。今年読んだ本の中で良かったのは[みんなの量子コンピュータ](https://amzn.to/3iSf9pl)と[本質から理解する 数学的手法](https://amzn.to/3W8gLcI)だった。

### コンピュータ技術書

* [Reactハンズオンラーニング 第2版 ―Webアプリケーション開発のベストプラクティス](https://amzn.to/3W0Qzke)
* [Kubernetes完全ガイド 第2版](https://amzn.to/3uBQx6K)
* [Kubernetes CI/CDパイプラインの実装](https://amzn.to/3iHxzsC)
* [最短コースでわかる PyTorch ＆深層学習プログラミング](https://amzn.to/3uFJ0nw)
* [AWS Amplify Studioではじめるフロントエンド+バックエンド統合開発](https://amzn.to/3iKJRjX)
* [セキュア・バイ・デザイン: 安全なソフトウェア設計](https://amzn.to/3YcFhLG)
* [ディープラーニングを支える技術](https://amzn.to/3PjWzlZ)
* [オペレーティングシステム 第3版](https://amzn.to/3uC2dGE)(ソースコードも読んでいるので進みが悪い。2.6 プロセスの実装まで)
* [みんなの量子コンピュータ](https://amzn.to/3iSf9pl)

### 科学系

* [本質から理解する 数学的手法](https://amzn.to/3W8gLcI)
* [時間は存在しない](https://amzn.to/3HoqTtI)
* [神の方程式: 「万物の理論」を求めて](https://amzn.to/3Yf2WuQ)
* [「ネコひねり問題」を超一流の科学者たちが全力で考えてみた](https://amzn.to/3FkRFkl)

### ビジネス

* [リーン・スタートアップ　ムダのない起業プロセスでイノベーションを生みだす](https://amzn.to/3Wayel3)
* [アジャイルサムライ――達人開発者への道](https://amzn.to/3hcyH7p)
* [チームトポロジー　価値あるソフトウェアをすばやく届ける適応型組織設計](https://amzn.to/3UH7ZRP)
* [ジョブ理論　イノベーションを予測可能にする消費のメカニズム](https://amzn.to/3G4AYtO)
* [保健所のコロナ戦記](https://amzn.to/3HmNwyH)
* [GE帝国盛衰史――「最強企業」だった組織はどこで間違えたのか](https://amzn.to/3Y5EDQd)
* [限りある時間の使い方](https://amzn.to/3Ce82OD)

### 小説

* [同志少女よ、敵を撃て](https://amzn.to/3W26nE1)
* [プロジェクト・ヘイル・メアリー](https://amzn.to/3Y9YFZQ)

## 初めて触ったプログラミング言語・ライブラリ等

無節操に色んなものに手を出していたなあ。

* [React](https://ja.reactjs.org/)
* [Kubernetes](https://kubernetes.io/ja/)
* [TimescaleDB](https://www.timescale.com/)
* [PyTorch](https://pytorch.org/)
* [OpenNMT-py](https://github.com/OpenNMT/OpenNMT-py)
* [CTranslate2](https://github.com/OpenNMT/CTranslate2)
* [JParaCrawl](https://www.kecl.ntt.co.jp/icl/lirg/jparacrawl/)
* [Whisper](https://github.com/openai/whisper)
* [Amazon Timestream](https://aws.amazon.com/jp/timestream/)
* [AWS SageMaker](https://aws.amazon.com/jp/sagemaker/)
* [AWS Amplify](https://aws.amazon.com/jp/amplify/)
* [AWS IoT Core](https://aws.amazon.com/jp/iot-core/)

## まとめ

ぼんやり過ごしていたのでは？って心配になったけど、充実していたのであっという間に一年が終わった気分になっているのかも。