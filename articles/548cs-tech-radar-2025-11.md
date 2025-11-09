---
title: "Technology Radar 2025年11月版(Vol. 33)を読む"
emoji: "🐥"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["technologyradar"]
published: true
---

[Technology Radar](https://www.thoughtworks.com/radar)の[Nov. 2025](https://www.thoughtworks.com/content/dam/thoughtworks/documents/radar/2025/11/tr_technology_radar_vol_33_en.pdf)版が出ていたので、気になったブリップについてまとめる。

読んでみて、やっておきたいことを以下に挙げておく。

* Pydantic AIとassistant-uiを使ってみる
* AWS EKSでArmインスタンスを使ってみる
* OpenFeatureを使ってみる
* Restateを使ってみる
* Context7、SerenaをClaude Codeと一緒に使ってみる

過去のTechnology Radarを読んでやってみたこと一覧

* ライブラリ等のパッケージの最新版を返す[MCPサーバを作ってみる](https://zenn.dev/thorie/scraps/3b0c144b820d2d#comment-7b9ea4c28d7451)。
* [Continuous Deployment](https://amzn.to/40BtRVx)を[読んだ](https://zenn.dev/thorie/articles/926-2025-3q-retrospective#%E3%82%B3%E3%83%B3%E3%83%94%E3%83%A5%E3%83%BC%E3%82%BF%E6%8A%80%E8%A1%93%E6%9B%B8)



## Techniques

### 採用(Adopt)

#### 2. ソフトウェアチーム向け共有指示のキュレーション（Curated shared instructions for software teams）

これはツールを使っているかではなく、AGENTS.mdのようなファイルに**何を書くかのノウハウ**が重要になりそう。

#### 4. GenAIによるレガシーコードベースの理解（Using GenAI to understand legacy codebases）

採用になっているから、何かしらのツールを使ってサクッとコードベースの理解をできるようにならないといけないな。

### 試してみよう(Trial)

### 9. LLMの構造化出力（Structured output from LLMs）

LLMのテキストって、その後は人間が読むか、LLMの入力にするかで、ソフトウェアでの処理に向かないもんな。構造化出力にファインチューニングされたLLMモデルもその内出てきそう。

### 10. TCR（Test && Commit || Revert）

 [Kent Beckの記事](https://medium.com/@kentbeck_7670/test-commit-revert-870bbd756864)が元ネタ。結局、「小さくコミットしろ、小さいとは小さなテストが通った時だ」と言う事。

### 触ってみよう(Access)

#### 15. LLM向けデータアクセスパターンとしてのGraphQL（GraphQL as data access pattern for LLMs）

> REST APIは、過剰なデータ取得やユースケースごとに新しいエンドポイントやフィルタを追加する必要がある傾向がありますが、GraphQLはモデルが必要なデータのみを取得できるため、ノイズを減らし、コンテキストの関連性を高め、トークン使用量を削減します。
さらに、適切に設計されたGraphQLスキーマは、LLMが利用可能なエンティティや関係性を推論するためのメタデータを提供し、エージェント的なユースケースで動的かつスキーマ認識型のクエリを可能にします。
> ただし、このアプローチは、よく構造化されたスキーマと意味のあるフィールド名に依存します。

本当にそう? REST APIでも[REST APIでフィルタリングをサポートする](https://blog.linkode.co.jp/entry/2024/11/12/161437)ようにすればいいような気もする。

#### 18. デバイス上での情報検索（On-device information retrieval）

面白そうだけど、性能面でダメな気がする。
軍事関連とかのセキュリティ要件が厳しい用途で、ぼったくれる顧客がいれば… そんな仕事は、世界を変えないし、つまらなさそう。 

#### 20. サイドカーなしのサービスメッシュ（Service mesh without sidecar）

> サイドカー型サービスメッシュのコストと運用の複雑さが依然として課題である

やっぱり、サービスメッシュは運用コストが高くなるよね。
[Istio ambient mode](https://istio.io/latest/docs/ambient/)なんてのがあるのか。Istio自体を触った事がないけど…

#### 21. 小型言語モデル（Small language models）

> 現在のエージェントワークフローの多くは、複雑な推論を必要としない狭い範囲の反復タスクに集中しており、SLMはこれらに適しています。Phi-3、SmolLM2、DeepSeekなどのSLMの進歩により、SLMは十分な能力を提供

自分でエージェントをSLMで作れたら楽しそう。

#### 22. 仕様駆動開発（Spec-driven development）

> 長大な仕様ファイルを生成してレビューが困難になる場合や、PRDやユーザーストーリーの対象が不明確になるケースもあります。私たちは、AIに詳細なルールを手作業で教え込むことはスケールしないという「苦い教訓」を再び学んでいるのかもしれません。

「歴史は繰り返さず韻を踏む。」か… 仕様書を参照させながら、適切な変更指示のプロンプトを書いていくのがいいのかな。

### 保留（Hold）

#### 27. キャパシティ駆動開発（Capacity-driven development）

> チームが余剰キャパシティを使って他の製品やストリームの機能を引き受けることです。一見効率的に見えますが、これは局所的な最適化であり、突発的な需要増に対応する場合を除き、通常化すると認知負荷や技術的負債を増やし、最悪の場合、複数プロダクト間のコンテキストスイッチによる混乱で開発が停滞します。余剰キャパシティがある場合は、システムの健全性改善に注力する方が望ましいです。

当たり前の事が未だ分かっていない管理職が多いってことかな。

どうして無能な上司どもは部下の手待ちの時間を嫌がるのかな。余剰キャパシティがある(手待ち)なら、常に余剰キャパシティがある状態にするように改善する(つまり少人化するための)作業をすればいいのに。

つまり、
* チームの多能工化で広いストリームを1チームで持てるようにする
* 仕事を平準化して、そもそも突発的な需要増というものをなくす
事が、上司の仕事。


## Platforms

### 採用（Adopt）

#### 32. クラウドにおけるArm（Arm in the cloud）

[マルチアーキテクチャDockerイメージ](https://www.docker.com/blog/multi-arch-build-and-images-the-simple-way/)とかのCI/CDができないといけないのか。

### 試してみよう(Trial)

#### 39. モデルコンテキストプロトコル（Model Context Protocol, MCP）

4月には間に合わなかったから、今回初登場。

[Terraform Moduleの最新バージョンを取得するMCP Serverを作る](https://zenn.dev/thorie/scraps/3b0c144b820d2d#comment-7b9ea4c28d7451)事もしてみて、便利な事は分かったけど、認可周りをちゃんとしないと製品化はしにくいかな。AIのSaaSの裏側でまずは実用化されそう。


### 触ってみよう(Access)

#### 51. OpenFeature

[OpenFeature](https://openfeature.dev/docs/reference/intro/)なんてものがあるのか、結構、色んな言語や、Feature flagのサービスに対応しているけど、Javaで[AWS AppConfig](https://docs.aws.amazon.com/ja_jp/appconfig/latest/userguide/what-is-appconfig.html)に対応しているものは[個人で開発されたprovider](https://github.com/lavenderses/aws-appconfig-openfeature-provider-java)しかないみたい。

[フィーチャーフラグを使ったストラングラーフィグパターンでのリアーキテクチャ](https://miraitranslate-tech.hatenablog.jp/entry/migrating-with-feature-flags-strangler-fig-pattern)のような使い方もできるから使っていきたいが、サービスは自由に変えられる方が便利だよな。

#### 53. Restate

マイクロサービスのSaaSを作って、Sagaパターンを実装しようとする時に何が面倒かというと補償トランザクションなんだよね。[Restateでの例](https://docs.restate.dev/guides/sagas)を使えば、メンテナンスが楽になりそうかな。


## Tools

### 採用（Adopt）

#### 59. pnpm

どんだけ、Node.jsのパッケージマネージャは湧いてくるんだ?

#### 60. Pydantic

> 特にLLMの予測不能な出力を扱う際、構造化出力のテクニックと組み合わせることで、自由形式のテキストを決定論的で型安全なPythonオブジェクト（例：JSON）に変換できます。

なのか、これは魅力的。

### 試してみよう(Trial)

#### 65. Context7

> Context7 は、MCP（Model Context Protocol）サーバーであり、AI生成コードの不正確さを解消することを目的

これは、LLMを使ってコーディングする時に入れておくとよさそう。

### 触ってみよう(Access)

#### 75. Docling

> Doclingはコンピュータビジョンベースの手法を用いてドキュメントのレイアウトや意味構造を解釈します。

ここは凄そう。

#### 82. Serena

> Serenaは、AIコーディングエージェント（例：Claude Code）にIDEライクな機能を提供する強力なコーディングツールキットです。

Context7と一緒に使っていきたい。

## Languages and Frameworks

### 採用(Adopt)

#### 85. Fastify

> Fastifyは、Node.js向けの軽量で高性能なWebフレームワークです。Express.jsと同等の機能を提供

ホンマに、Node.js関係は、次から次へとパッケージ管理ツールやフレームワークが湧いてくる。ついていけん。

#### 86. LangGraph

LangChainは4月に保留(Hold)になったけど、こっちは採用(Apopt)なんだ…

### 試してみよう(Trial)

#### 90. FastMCP

自分でも[使ってみた](https://zenn.dev/thorie/scraps/3b0c144b820d2d#comment-827789c36e010d)けど、いい感じだった。


#### 96. Pydantic AI

> Pydantic AIは、GenAIエージェントを本番環境で構築するためのオープンソースフレームワークです。
> 基盤：信頼性の高いPydanticライブラリをベースに構築。
> LangChainやLangGraphと並ぶ人気フレームワーク。

### 触ってみよう(Access)

#### 100. assistant-ui

> AIチャットインターフェースを構築するためのオープンソースTypeScript＋Reactライブラリです。
> ストリーミング対応（メッセージの逐次表示）。メッセージ編集や分岐切り替えの状態管理

Pydantic AIと合わせて触ってみたい。

#### 114. Vercel AI SDK

> Vercel AI SDKは、TypeScriptエコシステム向けのオープンソース・フルスタックツールキット
> ストリーミング、状態管理、リアルタイムUI更新をサポート
> React、Vue、Next.js、Svelteなどに対応

TypeScriptでフルスタックってのは良さそう。Pythonはサーバサイドで使うにはまだまだな感じがする。 ところでJavaはどこに行った…