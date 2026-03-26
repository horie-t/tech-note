---
title: "2026年3月の振り返り"
emoji: "🐈"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["振り返り", "ポエム"]
published: true
---

2026年も3ヶ月が過ぎたので、ここで振り返ってみる。大きなアウトプットとしては、3月に『[Spring AI入門](https://amzn.asia/d/0gDKrbBn)』を出版したことがある。2025年の12月から3月までの4ヶ月間、ずっとこの本の執筆をしていた。内容的には、Spring AIの基本的な使い方を解説する入門書である。Spring AIはJavaでAIアプリケーションを開発するためのフレームワークで、Spring Bootに馴染みがある人ならすんなりと理解できる。JavaでAIアプリケーションを開発したい人にはお勧めの本になったと思う。最近はAIに色々な事を調査させたり、質問したりしている。その内容をZennのScrapにまとめていたら[その中の圏論に関する調査](https://zenn.dev/thorie/scraps/6c8422254a51d9)がGoogleのDiscoverに載ってしまって、他の記事の数十倍のアクセスがあった。みんな、圏論に興味があるみたい。

## 主にやっていた事

* **1月:** 『[Spring AI入門](https://amzn.asia/d/0gDKrbBn)』の執筆。
* **2月:** 『[Spring AI入門](https://amzn.asia/d/0gDKrbBn)』の執筆。
* **3月:** 『[Spring AI入門](https://amzn.asia/d/0gDKrbBn)』の執筆。[OmniMouse](https://github.com/horie-t/omni-mouse)の開発を再始動させた。今度はJavaでAI駆動開発をしてみることにした。組込み関連の開発へのAIの活用事例になると思う。[関数型ドメインモデリング](https://tatsu-zine.com/books/domain-modeling-made-functional)を[Javaで実装](https://github.com/horie-t/programming-study/tree/master/functional_domain_modeling_in_java)してみている。

## 勉強したプログラミング言語・ライブラリ等

## 読書

### コンピュータ技術書

* [Domain Storytelling: A Collaborative, Visual, and Agile Way to Build Domain-Driven Software](https://learning.oreilly.com/library/view/domain-storytelling-a/9780137458974/): (Chapter 1, 2, 3, 4, 5, 6, 7)  
  Technology Radarで[紹介されていた](https://www.thoughtworks.com/radar/techniques/domain-storytelling)ので読んでみた。既存のシステムの全体把握にはEvent Stormingが良いと思うが、全くの新規サービスを構築する場合は過去形で語るのは難しいのでDomain Storytellingの方が良さそう。

### 数学・科学系専門書

* [圏論の道案内 ～矢印でえがく数学の世界～ 数学への招待](https://amzn.to/3YqJtc3): (第1, 2, 3, 4章)  
  前に読んだ[はじめての圏論　ブンゲン先生の現代数学入門](https://amzn.to/44Urv5j)の続きとして読んでいる。4章までは圏論の基礎的な概念の説明が丁寧で分かりやすい。5章以降の説明はあまり丁寧ではないので、他の参考書をあたった方が良さそう。
* [活躍する圏論 ―具体例からのアプローチ―](https://www.kyoritsu-pub.co.jp/book/b10025461.html): 
  読む本が迷走中。色んな本に手を出しているが、求めているのはコレジャナイ感があって、今はこれを読んでいる。まずは圏論がソフトウェア開発に役立つのかどうかを見極めたい。
* [みんなの圏論 ―演習中心アプローチ―](https://www.kyoritsu-pub.co.jp/book/b10003365.html): (第1章)  
  ologに興味があって読んでいる。まずはこの本を読了したい。

### その他専門書

* [恐れのない組織――「心理的安全性」が学習・イノベーション・成長をもたらす](https://eijipress.co.jp/products/2288): (第1, 2, 3, 4, 5, 6, 7, 8章; 読了)
  心理的安全性と仰々しい名前を付けているが、要は「風通しの良い組織」を作るための方法論である。アメリカの企業では上司が部下を首にする権利を持っていて、ある意味ではビッグモーターみたいに生殺与奪の権が上司に与えられている。そのため、部下が上司に対して意見を言いづらい状況が生まれやすい。日本の場合は解雇規制が厳しいため、上司が一方的に部下を首にすることは難しく、日本の法制度が日本の競争力を維持している面が良く分かる。
  アメリカでは小学校の頃からディベート教育等をしており、意見を言うことが奨励されるようになっている。でも、書籍では「でも、常識でしょう、誰とも絶対に衝突しちゃいけないなんてことは」と書かれており、アメリカの学校教育も無意味な努力をしていることが分かる。結局、社会に出てからも「空気を読む」ことが求められるのは日本もアメリカも同じである。
* [はじめての構造主義](https://www.kodansha.co.jp/book/products/0000146360): (読了)
  圏論を勉強していくなかで、圏論で重視されているものの一つに構造がある事が分かってきた。そういえば、現代思想の中で構造主義というのがあったなと思い、読んでみた。レビィ=ストロースの構造主義は、文化人類学の分野で、神話や婚姻などの文化的な現象を構造的に分析する方法論である。その中で群論の概念が使われていることが分かり、構造主義と圏論の関係性が少し見えてきた。
* [哲学マップ](https://www.chikumashobo.co.jp/product/9784480061829/): (読了)
  哲学の入門書。哲学の歴史を俯瞰することができる。名前を知っているだけの哲学者の思想の概要や歴史的な位置づけが分かった。

### 小説等の一般書

* [ハイペリオン（上）](https://amzn.to/49vzEQv): 読了  
  それなりに面白いけど、期待していたほどではなかった。残り3巻もあると思うとちょっと気が重いので、別の本を読むことにする。