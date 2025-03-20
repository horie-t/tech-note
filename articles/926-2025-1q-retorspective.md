---
title: "2025年3月の振り返り"
emoji: "🔥"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["振り返り", "ポエム"]
published: false
---
2024年も3ヶ月が過ぎたので、ここで振り返ってみる。

## 主にやっていた事

* **1月:** Python、PyTorchの勉強。[OmniMouse](https://github.com/horie-t/omni-mouse)の開発を再開した。  
  今までなんとなく見様見真似で使っていたPythonについてちゃんと勉強する。なんとなく嫌ってたPythonの一部の仕様にもそれなりの意図があっての事だったのかと理解できた。[Technology Radar](https://www.thoughtworks.com/radar)を過去に遡って読む。
  ROS2でPythonを使うのは悪手な事が判明したので、Pythonのフレームワークのみで実装し直す方針に転換した。
* **2月:** [OmniMouse](https://github.com/horie-t/omni-mouse)の開発。[Technology Radar](https://www.thoughtworks.com/radar)を過去に遡って読む。  
  PythonのActorモデルのライブラリである[Ray](https://www.ray.io/)を触り始める。Akkaや、そのオープンソースのフォークである[Apache Pekko](https://pekko.apache.org/)やRubyの[Ractor](https://docs.ruby-lang.org/en/3.4/ractor_md.html)と違って、簡単に使い始められるのが良い。
* **3月:** [OmniMouse](https://github.com/horie-t/omni-mouse)の開発。[Technology Radar](https://www.thoughtworks.com/radar)を過去に遡って読む。  
  月の初めに体調を崩してしまい開発が停滞気味。原書を読んでいる途中に和訳が発行されてしまってショックだった。

## 勉強したプログラミング言語・ライブラリ等

* [Python](https://www.python.org/)
* [PyTorch](https://pytorch.org/)
* [Ray](https://www.ray.io/)
* [unleash](https://www.getunleash.io/)

## 読書

### コンピュータ技術書

* [Python Distilled ―プログラミング言語Pythonのエッセンス](https://amzn.to/4ap5Ofi): (第1, 2, 3, 4, 5, 6, 7, 8, 9, 10章;読了)  
  [入門 Python 3 第2版](https://amzn.to/3PISYz0)とどちらを読むか迷ったが、まったくの初心者ではないのでエッセンスだけ読めばなんとかなると判断した。
* [ハイパーモダンPython](https://amzn.to/4amZ1CW): (第1, 2, 3, 4, 5, 6, 7, 8, 9, 10章;読了)    
  今時はPythonのバージョン管理やパッケージ管理やプロジェクト管理って何を使うのがいいのか全くわからなかったので読んだ。2023年の本なのでまだ古びていないと思う。
* [スクラム実践者が知るべき97のこと](https://amzn.to/3CLOILV): (第X, XI部; 読了)  
  エッセイが無秩序にならんでいるだけなので系統だった知識を得るには役に立たなかった。時々、気分転換や普段の実践を振り返って見るネタとしてちょっとずつ読むといい感じの本だった。
* [オブジェクト設計スタイルガイド](https://amzn.to/3WnuBug): (第1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11章;読了)  
  著者の妄想した実在しない言語の疑似コードやフレームワークを例題にして、オブジェクト指向のプログラムの書き方を説明している。オブジェクト指向言語の使用経験だけでよいと対象読者に書いているにも関わらず、内容はドメイン駆動設計をある程度勉強している事や、xUnit系のテストツールに慣れている事を前提にしていたりする。分かっている人はもう知っている事だし、分かっていない人には良く分からない説明になっていた。疑似コードも色々な言語のつまみ食いの仕様なので、全部のスタイルを実在の言語に適用するとその言語の設計思想とで反発がおきそう。
* [PyTorch実践入門](https://amzn.to/4aqaLo5): (第1, 2, 3, 4, 5章)  
  PyTorchでのテンソルクラスなど基礎的な事がしっかり書かれている。もっと昔に見つけられていたらと悔やんでしまう。
* [Build a Large Language Model (From Scratch)](https://amzn.to/3Ed0f7x): (Chap. 1, 2, 3, 4, 5; 読了)  
  埋め込みの処理の説明が良かった。下記の和訳が出たので残りは和訳を読む事にした。
* [つくりながら学ぶ！LLM 自作入門](https://amzn.to/4bNxziw): (第6, 7章; 読了)  
  [Build a Large Language Model (From Scratch)](https://amzn.to/3Ed0f7x)の和訳。昨年の9月に原書が発行されたばかりなのに2月に和訳が発行される。2年ぐらいは翻訳されないと思っていたのが…
* [RxJavaリアクティブプログラミング](https://amzn.asia/d/2qXV5sX): (Part 1, 2)  
  サーバサイドでのリアクティブプログラミングについてはこの本ぐらいしかコードの書き方について説明している本がない。一見、[関数型ドメインモデリング](https://amzn.to/3Y3t12c)の実装に使えるフレームワークのように思えてしまうが、逆に邪魔になるかもしれない。
* [マルチテナントSaaSアーキテクチャの構築](https://amzn.to/4ietwOt): (第1, 2, 3章)  
  前置きが長くなかなか本題に入っていかない…

### 数学・科学系専門書

* [初学の編集者がわかるまで書き直した　基礎から鍛える量子力学](https://amzn.to/3YdEdtd): (第1, 2, 3, 4, 5, 6章)  
  5章まで読んで難しくなってきたので、頭から再読。

### その他専門書

* [なぜ働いていると本が読めなくなるのか](https://amzn.to/4iIW6aI): (第1, 2, 3, 4, 5, 6, 7, 8, 9, 最終章)  
  最近の労働環境と読書の関係を論じた本なのかと思いきや、明治時代からの自己啓発の歴史と労働環境を俯瞰した上での現在の働き方への問題提起の本だった。読書をして教養を身に付けるという行為の動機は、現在の自己啓発本を読むのと同じものであると分析されていたのが意外だった。

### 小説等の一般書

やっぱり働いていると本が読めなくなるのか…