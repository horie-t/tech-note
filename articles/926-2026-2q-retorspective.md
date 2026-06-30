---
title: "2026年6月の振り返り"
emoji: "📌"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["振り返り", "ポエム"]
published: true
---

2026年も6ヶ月が過ぎたので、ここで振り返ってみる。[前回](https://zenn.dev/thorie/articles/926-2026-1q-retorspective)は3月までを振り返ったので、4〜6月を振り返る。

Zennのいくつかの記事やScrapのページビューが1000を超えることがあった。Scrapの内容はAIとの対話をまとめたものなので、こんな内容でアクセスが増えるのはちょっと虚しい。圏論の勉強が進んで「[レベル3](https://zenn.dev/thorie/articles/548pro-cat-user-lv3)」ぐらいには到達できた気がして嬉しい。半年でレベル3まで到達できたのだから、年内にレベル6ぐらいまで到達できるのではないかと期待してしまう。

ここ3ヶ月は、ソースコードを仕事でもプライベートでもほぼ書いていない。中学生から40年間プログラミングをしてきたが、まさかこんな時代になるとは思わなかった。ただし、プログラミングはしなくなってもエンジニアリングはまだまだ続きそう。

LLMのモデルの進捗によって英語の書籍に対する見方が大きく変化した。以前は余程関心があって和書がない場合に、関心のある部分だけを苦労しながら読んでいた。オライリーのサブスクはブラウザ上で翻訳ブラウザ拡張を使って読んでいて便利だなとは思っていたがまだまだ一部の書籍が読めるだけの認識だった。[Computational Quantum Mechanics](https://link.springer.com/book/10.1007/978-3-319-99930-2)ではPDF版を購入して、章ごとのPDFに分割してからClaudeに「Markdownに変換して」とお願いしてから読んでいる。もうこれで十分だと思う。英語の論文は分割なしでいきなり読めるようになったので、これからは原論文を読むことが増えそう。

[2026年の抱負](https://zenn.dev/thorie/articles/926-2026-resolutions)として挙げたものはだいぶ達成できた。今年の後半は、圏論と、量子力学の数値計算の勉強を中心に進めていきたい。後は、人文系の本ももう少し読むようにしたい。

## 主にやっていた事

* **4月:**
  * [OmniMouse](https://github.com/horie-t/omni-mouse)の開発。AI駆動開発によるプログラミングは順調に進んでいる。しかし、ユニバーサル基板での配線のはんだ付けをミスってモータ・ドライバの回路を壊してしまい、モータ・ドライバを買い直す羽目になった。はんだ付けは老眼には厳しいと実感する年頃になってきた。跳ね上げ式の老眼鏡を買おうか、悩ましい。
  * [関数型ドメインモデリング](https://tatsu-zine.com/books/domain-modeling-made-functional)を[Javaで実装](https://github.com/horie-t/programming-study/tree/master/functional_domain_modeling_in_java)してみている。今のJavaなら関数型ドメインモデリングを実装するのもそんなに苦労しないことが分かってきた。vavrをどの程度使うかが悩ましいところ。
* **5月:** 
  * [みんなの圏論 ―演習中心アプローチ―](https://www.kyoritsu-pub.co.jp/book/b10003365.html)を読了した。
  * [プログラマーのための圏論](https://www.ohmsha.co.jp/book/9784274234866/)のBartosz Milewskiの[YouTubeのコース](https://youtube.com/playlist?list=PLbgaMIhjbmEnaH_LTkxLI7FMa2HsnawM_&si=on5wBWhx31RqfAD3)を見ている。本には載っていない、圏論における考え方の話が面白い。
  * [関数型ドメインモデリング](https://tatsu-zine.com/books/domain-modeling-made-functional)を[Javaで実装](https://github.com/horie-t/programming-study/tree/master/functional_domain_modeling_in_java)を続ける。
* **6月:** 
  * [関数型ドメインモデリング](https://tatsu-zine.com/books/domain-modeling-made-functional)を[Javaで実装](https://github.com/horie-t/programming-study/tree/master/functional_domain_modeling_in_java)するのを完成させて[Zennの記事](https://zenn.dev/thorie/articles/548pro-domain-modeling-made-func-in-j)にまとめた。ページビューが300弱ぐらい。
  * [AI駆動開発でスクラッチからSaaSアプリケーションを実装する](https://zenn.dev/thorie/articles/548se-ai-driven-development)。まさか2日でここまでのものが作れるとは思わなかった。
  * [活躍する圏論](https://www.kyoritsu-pub.co.jp/book/b10025461.html)を読むのを再開する。

## 勉強したプログラミング言語・ライブラリ等

* [Processing](https://processing.org/): [VPython](https://vpython.org/)の代わりにJavaで科学計算結果の可視化をできないかと検討している。グラフの描画ができないのでJzy3dと併用かなと思う。
* [Jzy3d](http://www.jzy3d.org/): 3Dグラフの描画ができるライブラリ。

## 読書

### コンピュータ技術書

* [Effective Java 第3版](https://www.maruzen-publishing.co.jp/book/b10120153.html): (第1, 2, 3章)  
  今頃? と思われるかもしれないが、今頃読んでいる。Javaは長年使っているが、Effective Javaは読んだことがなかったので。
* [アーキテクチャモダナイゼーション 組織とビジネスの未来を設計する](https://amzn.asia/d/0hox0Nlb): (第1, 2, 3, 4, 5, 6, 7, 8, 9章)  
  ソフトウェアのアーキテクチャを改善するための方法論の本ではあるが、組織の構造やビジネスの構造を改善も一緒に考える必要があることを説いている。アーキテクチャの改善は、組織の構造やビジネスの構造の改善とセットで考える必要があることを改めて認識させられた。
* [LLMの原理、RAG・エージェント開発から読み解く コンテキストエンジニアリング](https://gihyo.jp/book/2026/978-4-297-15419-6): (第1, 2, 3, 4, 5章; 読了)
  LLMの回答やタスクの性能を向上させるためには、どのようにLLMに渡すコンテキストを設計すれば良いのか、ということをLLMの原理やRAGについての説明を交えて解説している本。LLMの原理については、Transformerの説明から始まって、Attentionの説明やLLMの学習方法の説明などが丁寧にされている。LLMの仕組みについて良く理解せずに開発をしてきて、大きなタスクを実装しようとして行き詰った人にはお勧めの本。逆に、LLMやRAGの仕組みについてある程度理解している人には、物足りない内容かもしれない。
* [実践 AIエージェント開発 ― マルチエージェントシステムの設計と実装](https://www.oreilly.co.jp/books/9784814401598/): (第1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13章; 読了)  
  [Spring AI](https://amzn.asia/d/08TYpmZw)を使ってみてLLMを使ったワークフローの構築まではできたが、マルチエージェントシステムの設計や実装についてはまだまだ勉強が必要だと感じた。AIエージェントの設計や実装について、もう少し体系的に学びたいと思って読んでみたが、概念的は説明が多く、実装の具体的な方法についてはあまり詳しく説明されていなかった。AIエージェントの設計や実装について、もう少し具体的な方法を知りたいと思っている人には、物足りない内容かもしれない。
* [Computational Quantum Mechanics](https://link.springer.com/book/10.1007/978-3-319-99930-2): (第1, 2, 3, 4章)  
  [初学の編集者がわかるまで書き直した　基礎から鍛える量子力学](https://amzn.to/3YdEdtd)を読んでから、実際に量子力学の計算をプログラミングしてみたいと思って色々な日本語の本を探していたが、どれもしっくりこなくて、結局英語の本を読むことにした。
* [ゼロから作るDeep Learning ❻―LLM編](https://www.oreilly.co.jp/books/9784814401611/): (第1章)  
  昨年[つくりながら学ぶ！LLM 自作入門](https://book.mynavi.jp/ec/products/detail/id=146901)を読んだが、選好チューニングは扱っていなかったので、選好チューニングについて学ぶために読んでいる。

### 数学・科学系専門書

* [みんなの圏論 ―演習中心アプローチ―](https://www.kyoritsu-pub.co.jp/book/b10003365.html): (第2, 3, 4, 5, 6, 7章; 読了)  
  色々と読む本が迷走していたが、これを腰を据えて読んでみた。何回か最初から読み返しているが、圏論の基礎的な概念を理解するのに役立っている。米田の補題も、なんとなくではあるが理解できた。
* [活躍する圏論](https://www.kyoritsu-pub.co.jp/book/b10025461.html): (第1, 2, 3, 4章)  
  『みんなの圏論』で分かりにくかった所が、具体的な例を使って説明されているので腹落ちしやすい。応用例を見ながら、プログラミングにおける圏論の考え方を理解するのに役立ちそう。

### その他専門書

* [若い読者のための哲学史](https://www.subarusya.jp/book/b355621.html): (1, 2, 3, 4, 5, 6, 7, 8, 9)  
  Kindleで半額セールをしていたので購入。人物とその思想の概要について超簡単に説明されているだけの本なので、これで何かが分かったつもりになるのは危険な気がする。

### 小説等の一般書

* [指輪物語1 旅の仲間](https://www.hyoronsha.co.jp/search/9784566023895/): (読了)  
  これまで指輪物語は映画でしか見たことがなかったので、原作を読んでみた。