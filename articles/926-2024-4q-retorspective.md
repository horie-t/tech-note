---
title: "2024年10-12月の振り返り"
emoji: "🌊"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["振り返り", "ポエム"]
published: true
---
9月に[振り返り](./926-2024-3q-retorspective)をしたので、今回は10月から12月の間を振り返ってみる。

## 主にやっていた事

* **10月:** 
  * [関数型ドメインモデリング](https://amzn.to/3BRjrqq)の内容をJavaで実装する。
  * [ANTLR](https://www.antlr.org/)でSQLのWhere句の式をパースする。パーサコンビネータやパーサジェネレータの調査をする。
* **11月:** 
  * [ANTLRについてZennの記事](https://zenn.dev/thorie/articles/548pro-antlr-setup)を書く。
  * TerraformのコードからPlantUMLのアーキテクチャ図を生成する[ツール](https://github.com/horie-t/tf2puml)を作り始めた。
  * [Technology Radar](https://www.thoughtworks.com/radar)の2024年10月版を読んで感じた事を[書いた](https://zenn.dev/thorie/articles/548cs-tech-radar-2024-10)。
* **12月:**  
  * TerraformのコードからPlantUMLのアーキテクチャ図を生成する[ツール](https://github.com/horie-t/tf2puml)の開発を続けた。
  * [Technology Radar](https://www.thoughtworks.com/radar)を読んで感じた事を書いた。([2022年10月](https://zenn.dev/thorie/articles/548cs-tech-radar-2022-10), [2024年4月](https://zenn.dev/thorie/articles/548cs-tech-radar-2024-04))
  * [Unleash](https://www.getunleash.io/)についてのZennの記事を書く。([*1](https://zenn.dev/thorie/articles/548se-unleash), [*2](https://zenn.dev/thorie/articles/548se-unleash-front))

## やってみたいと思った事とその結果

* [ANTLR](https://www.antlr.org/)の解説記事を書く  
  **結果:** 記事を2つ書いた。([*1](https://zenn.dev/thorie/articles/548pro-antlr-setup), [*2](https://zenn.dev/thorie/articles/548pro-antlr-arithmetic))
* ANTLRを使って、REST APIにSQL形式のフィルタリングオプションを実装してみる  
  **結果:** 会社の技術ブログに記載した。
* ANTLRを使ったアプリケーションを作ってみる  
  **結果:** [tf2puml](https://github.com/horie-t/tf2puml)という、TerraformのコードからPlantUMLのアーキテクチャ図を生成するツールを作り始めた。
* [関数型ドメインモデリング](https://amzn.to/3BRjrqq)の形式をvavr等のライブラリを使ってみて実現できるかを検証する
  **結果:** 手をつけたが、tf2pumlの開発で手一杯だった。
* PluntUMLを使えるオンラインホワイトボードSaaSを作る  
  WebSocketや、イベント駆動型アーキテクチャの勉強として。[excalidraw](https://github.com/excalidraw/excalidraw)を参考に
  **結果:** tf2pumlの開発で手一杯で未着手となった。
* [omni-mouse](https://github.com/horie-t/omni-mouse)を完成させる  
  迷走しているマイクロマウスの作成プロジェクトを完了させる。Projct PanamaやVert.xを使ってJavaで実装するのもいいかもと思い始めている  
  **結果:** tf2pumlの開発で手一杯で未着手となった。
* [ゼロから作るDeep Learning ❸―フレームワーク編](https://amzn.to/4fmqrdg)を読む  
  FPGAをアクセラレータとして使うフレームワークを作れないかを考える。  
  **結果:** 8割ぐらいは読めた。FPGAを使うまでにはnumpyの実装まで知る必要があった。予想外の収穫としてPyTorchの実装のイメージができて理解が進んだ。
* [Build a Large Language Model (From Scratch)](https://amzn.to/4fqXgpv)を読む  
  自前LLMを作れるようになっていれば、ドメイン特化のSLMを仕事で作るようになるのではないかと思う  
  **結果:** 『ゼロから作るDeep Learning ❸―フレームワーク編』を読んだ後で読む予定にした。
* [Database Design and Implementation: Second Edition](https://amzn.to/3C5Npal)を読んで写経する  
  自作OSや自作CPUはやってみたので、今度はRDBMSをやりたい。  
  **結果:** 1, 2年は時間がとれなさそう。
* アジャイル開発での技法についてまとめたものを書く  
  **結果:** 書く項目を少しだけ挙げ始めた。
* [情熱プログラマー ソフトウェア開発者の幸せな生き方](https://amzn.to/4hoOcTO)の内容を実践する
  * 週単位の目標を立て、その結果を記録する。今日は何をすべきか？を意識する  
    **結果:** 2週間をIterationとしてGitHub Projectで予定を立てて、Togglで記録し、Miroで振り返るようにした。
  * 週に2時間は新しい技術を調査する  
    **結果:** Technology Radarを読む習慣を身につけるようにした。読んで気になる技術([Unleash](https://www.getunleash.io/))の技術記事を2つ書いた。
  * 自分のロードマップを作成する  
    **結果:** 半年ぐらいはIterationを続けて、やり遂げられる量を見極める事ができるようになってから作成する事にした。

## 勉強したプログラミング言語・ライブラリ等

* [ANTLR](https://www.antlr.org/): Javaで書かれたパーサジェネレータ
* [Unleash](https://www.getunleash.io/): オープンソースのフィーチャーフラグ管理のプラットフォーム

## 読書履歴と感想

### コンピュータ技術書

* [脳に収まるコードの書き方](https://amzn.to/4cPSxN6): (第14, 15, 16章; 読了)
  悪くはなかったけど、得たものが少なかったかなあ。
* [すごいHaskellたのしく学ぼう！](https://amzn.to/40eTVpo): (モナド関連を拾い読み; 読了)
  [関数型ドメインモデリング](https://amzn.to/3Y3t12c)でモナドについて少し触れていたのでモナドの復習になった。
* [The Definitive ANTLR 4 Reference](https://amzn.to/40a8F8V): (Chapter 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12; 残りはリファレンスなので、読了とする)
  Webアプリでパーサが必要になるようなオプションを設計できないかと考えANTLRを使おうとして勉強するため。
* [ふつうのコンパイラをつくろう](https://amzn.to/4dRr9xX): (第1, 2, 3, 4, 5, 6, 7章)
  ANTLRに辿り着く前にパーサの復習をしようと思って読み始めた。
* [情熱プログラマー ソフトウェア開発者の幸せな生き方](https://amzn.to/4hoOcTO) (第１, 2, 3, 4, 5章; 読了)
  ネットでお勧めしているのをみたので読んでみた。昔の話って感が強い。
* [ルールズ・オブ・プログラミング ―より良いコードを書くための21のルール](https://amzn.to/40ugN47): (ルール 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21; 読了)
  C++の環境だと問題になるかもしれないけど、他の言語の環境だと言語機能やライブラリ、フレームワークで解決済みの問題を議論している感が強い。
* [ドメイン駆動設計をはじめよう ―ソフトウェアの実装と事業戦略を結びつける実践技法](https://amzn.to/3YZucQK): (第1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16章; 読了)  
  具体例がなく心構えみたいな内容が多く類書との違いも少なかった。ただ、イベント駆動型アーキテクチャや、データメッシュについては理解していなかったので少し参考になった。
* [スクラム実践者が知るべき97のこと](https://amzn.to/3CLOILV): (第I, II, III, IV, V, VI, VII, VIII, XI部)
* [ゼロから作るDeep Learning ❸ ―フレームワーク編](https://amzn.to/3ZpJ3UQ): (第1, 2, 3, 4, 5ステージ; 読了)
  もっと早く読むべきだったと後悔。Pytorchなどが裏でやっている事が分かりやすく説明されている。

### 数学・科学系専門書

* [初学の編集者がわかるまで書き直した　基礎から鍛える量子力学](https://amzn.to/3YdEdtd): (第1, 2, 3, 4, 5章)
  6章から難しい。

### その他専門書

* [ChatGPT翻訳術](https://amzn.to/4h4hSFE): (読了)
  命題とモーダルという分け方があるのが勉強になった。用語集を作りながら翻訳させると訳語が統一できそう。
* [THE MODEL](https://amzn.to/4fJ1Ks5): (第1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15章; 読了)  
  タイトルがでかい。The Sales Modelぐらいの内容。営業のパイプランという考え方が勉強になった。若い頃に読みたかったが、30年前にこの概念があったのだろうか…

### 小説等の一般書

読めなかった。