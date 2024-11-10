---
title: "Technology Radar 2024年10月版を読む"
emoji: "🙌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["technologyradar"]
published: true
---

[Technology Radar](https://www.thoughtworks.com/radar)の[Oct. 2024](https://www.thoughtworks.com/content/dam/thoughtworks/documents/radar/2024/10/tr_technology_radar_vol_31_en.pdf)が出ていたので、気になったブリップについてまとめる。全般的に、生成AI関連とRustのツールのブリップが多かった。

[Testcontainers](https://www.testcontainers.org/)はDBのテストに使っているぐらいで、どこまでできるのかを把握していないので、ちゃんと調べたい。[Instructor](https://github.com/jxnl/instructor)は実際に動かして試しておきたい。[Unleash](https://www.getunleash.io/)は今のプロジェクトに導入させたい。

書籍に関しては
* [Continuous Deployment](https://amzn.to/40BtRVx)
* [Domain Storytelling: A Collaborative, Visual, and Agile Way to Build Domain-Driven Software](https://amzn.to/4enhboy)

を読んでおきたい。

## 2. Component testing
Adopt
コンポーネントテストをメモリ内で実行するために[jsdom](https://github.com/jsdom/jsdom)のようなライブラリを使用することを推奨。

## 3. Continuous deployment
Adopt
継続的デプロイは、ソフトウェアデリバリーの他の分野で高いレベルの成熟度を必要とするプラクティスであり、すべてのチームに適しているわけではない。
Valentina Servileの最近の著書である[Continuous Deployment](https://amzn.to/40BtRVx)は、組織でこのプラクティスを実装するための包括的なガイドを提供。似たような書籍に[入門 継続的デリバリー](https://www.oreilly.co.jp/books/9784814400690/)がある。

## 4. Retrieval-augmented generation (RAG)
Adopt
再ランク付けとハイブリッド検索が行われている。Elasticsearch Relevance Engine(https://www.elastic.co/jp/elasticsearch/elasticsearch-relevance-engine)などの検索ツールや、LLMの助けを借りて作成されたナレッジグラフを利用する[GraphRAG](https://microsoft.github.io/graphrag/)などのアプローチがある。

## 5. Domain storytelling
Trial
[ドメインストーリーテリング](https://domainstorytelling.org/)は、ビジネスの専門家がビジネスのアクティビティを説明するように促されるファシリテーション手法。また、DDDのより包括的なアプローチとしては、DDDを始めるために使う手法であるイベントストーミングと組み合わせることもできる。書籍は[Domain Storytelling: A Collaborative, Visual, and Agile Way to Build Domain-Driven Software](https://amzn.to/4enhboy)がある。

## 7. Function calling with LLMs
Trial
OpenAIのGPTシリーズのようなモデルは関数呼び出しをサポートしており、Gorillaのような微調整されたモデルは、自然言語命令から実行可能なAPI呼び出しを生成する精度と一貫性を高めるように特別に設計されている。

## 10. Small language models
Trial
適切なコンテキストで正しく設定すれば、SLMはLLMと同等かそれ以上のパフォーマンスを発揮し、そのサイズによってエッジデバイスでの実行が可能になる。
* GoogleのGemini Nano
* MicrosoftのPhi-3

## 17. Observability 2.0
Assess
単一のデータストア内の構造化されたカーディナリティの高いイベントデータを活用する統一されたアプローチへの移行

## 19. Structured output from LLMs
Assess
OpenAIは構造化出力をサポートするようになり、開発者はJSON Schema、pydantic、またはZodオブジェクトを指定してモデルの応答を制約できるようになった。

## 33. FoundationDB
Assess
FoundationDBのコアは、厳密なシリアル化トランザクションを提供する分散キーバリューストア。

## 34. Golem
Assess
Durable Computingは、明示的ステートマシンのアーキテクチャスタイルを使用して、サーバレスサーバのメモリを永続化し、フォールトトレランスとリカバリを向上。Golemは、この動きの1つ。この概念は、長時間実行されるマイクロサービスのサガやAlエージェントオーケストレーションでの長時間実行ワークフローなど、一部のシナリオで役立つ。

## 41. Unblocked
Assess
[Unblocked](https://getunblocked.com/)。アプリケーションライフサイクル管理 (ALM) およびコラボレーションツールと統合して、チームがコードベースと関連リソースを理解するのに役立つ。日本語に対応しているかは不明。

## 45. Visual regression testing tools
Adopt
[Applitools](https://applitools.com/)や[Percy](https://percy.io/)などのいくつかの商用ツールは、視覚回帰テストでAlを使用していると主張。チームの1つは、Applitools Eyesを広範囲に使用し、結果に満足。

## 50. Devbox
Trial
[Devbox](https://www.jetpack.io/devbox/)は、仮想マシンやコンテナーを使用せずにNixパッケージマネージャーを活用して、再現可能なプロジェクトごとのローカル開発環境を作成するための使いやすいインターフェイスを提供するコマンドラインツール

## 51. Difftastic
Trial
[Difftastic](https://difftastic.wilfred.me.uk/)は、構文を認識する方法でコードファイル間の違いを強調表示するツール。[Gitと組み合わせ](https://dev.classmethod.jp/articles/git-difftool-use-difftastic/)られる。

## 53. pgvector
Trial
[pgvector](https://github.com/pgvector/pgvector)はPostgreSQL用のオープンソースのベクトル類似性検索拡張機能。

## 54. Snapcraft build tool
Trial
[Snapcraft](https://snapcraft.io/docs/snapcraft)は、Ubuntu、その他のLinuxディストリビューション、macOSでスナップと呼ばれる自己完結型アプリケーションをビルドおよびパッケージ化するためのオープンソースのコマンドラインツール

## 55. Spinnaker
Trial
[Spinnaker](http://www.spinnaker.io/)は、Netflixによって作成されたオープンソースの継続的デリバリープラットフォーム。
マルチクラウド、カナリアリリース、承認フローが必要な環境に適している。

## 56. TypeScript OpenAPI
Trial
[TypeScript OpenAPI](https://github.com/lukeautry/tsoa) (またはtsoa) は、コードからOpenAPI仕様を生成するためのSwaggerの代替手段。コードファーストである。[springdoc-openapi](https://springdoc.org/)のTypeScript版。

## 57. Unleash
Trial
[可能な限り単純な機能トグルを使用](https://www.thoughtworks.com/radar/techniques/simplest-possible-feature-toggle)することを推奨。チームのスケーリングと開発の高速化により、手作りのトグルの管理がより複雑になる。[Unleash](https://www.getunleash.io/)は、この複雑さに対処しCI/CDを有効にするために広く使用されている。Spring Bootアプリケーションや、Reactで使用できる。

## 60. Cursor
Assess
[Cursor](https://www.cursor.com/)は、Alをコーディングワークフローに深く統合することで開発者の生産性を向上させるように設計された、Alファーストのコードエディター。CursorはVSCodeからフォーク。

## 76. Testcontainers
Adopt
[Testcontainers](https://www.testcontainers.org/)は、テストを実行するための信頼性の高い環境を作成する。データベース、キューテクノロジ、クラウドサービス、WebブラウザーなどのUIテストに使える。テストセッションを視覚的に管理できる[デスクトップバージョン](https://testcontainers.com/desktop/)がリリース。

## 80. Instructor
Trial
[Instructor](https://github.com/jxnl/instructor)は、LLMから構造化された出力を要求するのに役立つライブラリであり、目的の出力構造を定義し、LLMが要求した構造を返さない場合は再試行。

## 103. shadcn
Assess
[shadcn](https://ui.shadcn.com/)は、コードベースの一部となる再利用可能なコピー&ペーストコンポーネントを提供。Reactベースのアプリケーションにシームレスに統合される。
