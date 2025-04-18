---
title: "Technology Radar 2023年09月版を読む"
emoji: "💬"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["technologyradar"]
published: true
---

[Technology Radar](https://www.thoughtworks.com/radar)の[Sep. 2023](https://www.thoughtworks.com/content/dam/thoughtworks/documents/radar/2023/09/tr_technology_radar_vol_29_en.pdf)を読んだので、気になった項目をまとめて、感想を記録しておく。多くは、LLMを始めとした生成AI関連だった。

今後調べたいものは以下。

14. アラートルールのユニットテスト
25. CloudEvents
59. KEDA
83. Playwright
86. Armeria
92. Netflix DGS
107. Spring Modulith

その他に気になったものを取り上げる。

## Techniques

### 3. アクセシビリティに配慮したコンポーネントテスト設計

Chai-a11y-axeのようなテストフレームワークプラグイン
ARIAロール
Testing Library

### 9. ReActプロンプト手法
試行 (Trial)

OpenAIがそのAPIに「関数呼び出し」機能を導入し、LangChainのような外部ツールを使わずにReActや類似のプロンプト手法を容易に実装できるようになりました。

### 12. LLM向け半構造化自然言語
試行 (Trial)

構造化された入力に自然言語のコメントや注釈を追加することで、自然言語または構造化入力のみの場合よりも良い応答が得られることがわかっています。

### 14. アラートルールのユニットテスト
試行 (Trial)

Prometheusのようなツールはアラートルールのユニットテストをサポート

## Platforms

### 25. CloudEvents
試行 (Trial)
イベントは、イベント駆動型アーキテクチャやサーバーレスアプリケーションにおいて一般的な仕組みです。しかし、イベントの発行元やクラウドプロバイダーはそれぞれ異なる形式でイベントをサポートする傾向があり、その結果、プラットフォームやインフラ間での相互運用性が妨げられることがあります。

CloudEventsは、共通フォーマットでイベントデータを記述するための仕様であり、サービス、プラットフォーム、システム間の相互運用性を提供します。CloudEventsは複数の言語に対応したSDKを提供しており、これをアプリケーションやツールチェーンに組み込むことができます。私たちのチームでは、クロスクラウドプラットフォームでの利用だけでなく、ドメインイベントの仕様化など、さまざまなシナリオでCloudEventsを使用しています。

CloudEventsはCloud Native Computing Foundation (CNCF) によってホストされており、現在はインキュベータープロジェクトとして運営されています。業界での注目がますます高まっていることも特筆すべき点です。

### 33. ActivityPub
評価中 (Assess)
現在、ミクロブログプラットフォーム分野での大きな変動に伴い、ActivityPubプロトコルが注目を集めています。ActivityPubは、投稿、出版物、日付などの情報を共有するためのオープンプロトコルです。これを使用してソーシャルメディアプラットフォームを実装することができますが、最大の利点は、異なるソーシャルメディアプラットフォーム間の相互運用性を提供することです。

私たちは、ActivityPubがこの分野で重要な役割を果たすと予想していますが、ここで取り上げる理由は、ソーシャルメディアの明白なユースケースを超えた可能性に興味を抱いているからです。その一例として、GitLabのマージリクエストにActivityPubサポートを追加する提案が最近行われました。

## Tools

### 54. Devbox
試行 (Trial)

Devboxは、Nixパッケージマネージャーを活用して、プロジェクトごとの開発環境を再現可能にするターミナルベースのツールです。仮想マシンやコンテナを使用せずに動作し、軽量で効率的な環境構築を可能にします。

主な機能：

バージョンおよび設定の不一致解消：CLIツールやカスタムスクリプトのバージョンや設定の不一致を解消します。
簡単なオンボーディング：コードベースの環境が一度設定されていれば、新しいマシンで「devbox shell」という1つのCLIコマンドを実行するだけで環境を立ち上げることが可能です。
拡張性：シェルフック、カスタムスクリプト、VSCodeとの統合のためのdevcontainer.json生成をサポート。
私たちのチームは、Devboxを使用してプロジェクトごとの開発環境を標準化し、オンボーディングプロセスを大幅に効率化しました。また、異なるプロジェクト間でのツールやスクリプトの管理も容易になり、ワークフローの改善に寄与しました。

これらの利点から、Devboxを「試行 (Trial)」段階に位置付けています。

### 59. KEDA
試行 (Trial)

KEDA（Kubernetes Event-Driven Autoscaler）は、その名の通り、処理すべきイベントの数に応じてKubernetesクラスターのスケーリングを可能にするツールです。

主な特徴：

イベント駆動型スケーリング：CPU使用率のような遅延指標ではなく、キューの深さなどの先行指標に基づいてスケーリングを実行します。
多様なイベントソースのサポート：KEDAは、クラウドプラットフォーム、データベース、メッセージングシステム、テレメトリーシステム、CI/CDシステムなど、50以上のスケーラーを提供しています。
マイクロサービスとの統合：Kubernetes内でイベント処理を維持できるため、イベント処理コードをサーバーレス機能に移行する必要がある場合でも一貫した統合が可能です。
私たちのチームは、KEDAの統合の容易さを評価しており、Kubernetes上でのマイクロサービスの機能を維持する際に役立つと報告しています。その結果、KEDAを「試行 (Trial)」段階に位置付けています。

## Languages and Frameworks

### 83. Playwright
Adopt

Playwrightを使えば、Chrome、Firefox、WebKitで動作するエンドツーエンドのテストを書くことができる。Chrome DevTools Protocol (CDP)を使用することで、Playwrightは新しい機能を提供し、WebDriverで見られる問題の多くを取り除くことができます。同じAPIを使用することで、WebDriverが抱える問題の多くが解消され、一連の新機能が導入される。ChromiumベースのブラウザはCDPを直接実装しています。しかし、FirefoxとWebkitをサポートするために、Playwrightチームはこれらのブラウザにパッチを提出しなければならず、フレームワークが制限されることがある。
Playwright の多くの機能には以下が含まれます： 組み込みの自動待機機能により、より信頼性が高く、理解しやすいテストが実現されます。ブラウザコンテキストにより、タブをまたいだセッションの永続化が適切に機能することをテストできます。私たちのチームは、Playwrightがテストスイートにもたらす安定性に感銘を受け、テストを並して実行することで、より迅速にフィードバックを得られることを気に入っています。Playwrightを際立たせるその他の機能には、遅延ロードやトレースのサポートが充実していることが挙げられる。例えば、コンポーネントのサポートは現在実験的なものであるなど、Playwrightにはいくつかの制限があるが、我々のチームはPlaywrightをテストフレームワークとして使っており、場合によってはCypressやPuppeteerから移行している。

### 85. Ajv
Trial

Ajvは、JSONスキーマを使用して定義された構造に対してデータオブジェクトを検証するために使用される一般的なJavaScriptライブラリです。Ajvは、複雑なデータ型の検証を高速かつ柔軟に行うことができます。カスタムキーワードやフォーマットを含む、幅広いスキーマ機能をサポートしています。多くのオープンソースのJavaScriptアプリケ
ーションやライブラリで使用されています。私たちのチームは、CIワークフローにコンシューマ主導のコントラクトテストを実装するためにAjvを使用しており、JSONスキーマでモックデータを生成するための他のツールとともに、非常に強力だ。TypeScriptの世界では、スキーマの定義とデータの検証のための宣言的なAPIを持つZodが人気のある
代替手段である。

### 86. Armeria
Trial

Armeria は、マイクロサービスを構築するためのオープンソースのフレームワークです。私たちのチームは、非同期APIを構築するためにこのフレームワークを使用しており、分散トレースやサーキットブレーカのような横断的な懸念を、サービスデコレーターを介して解決するアプローチが非常に気に入っている。このフレームワークには、gRPCとRESTトラフィックの両方に対するポートの再利用が含まれており、他の賢い設計の選択も含まれている。Armeriaを使えば、RESTの既存のコードベースの上にgRPCの新機能をインクリメンタルに追加したり、その逆も可能だ。全体として、Armeriaは柔軟なマイクロサービス・フレームワークで、すぐに使える統合機能がいくつかある。

https://armeria.dev/

Armeriaとは何か?
Armeriaは、あらゆる状況で利用できるマイクロサービスフレームワークだ。gRPC、Thrift、Kotlin、Retrofit、Reactive Streams、Spring Boot、Dropwizardなど、お好みのテクノロジを活用して、あらゆる種類のマイクロサービスを構築できる。

### 90. Kotlin with Spring
Trial

5年前、私たちはKotlinをAdoptリングに移しました。今日、私たちのチームの多くは、KotlinがJVM上のデフォルトの選択肢であるだけでなく、彼らが書くソフトウェアにおいてJavaをほぼ完全に置き換えていると報告しています。同時に、マイクロサービスに対する羨望は薄れつつあるようだ。Spring Modulithなどのフレームワークを使い、より大きなデプロイ可能なユニットを持つアーキテクチャを模索し始めている。しかし、場合によっては、Springフレームワークの成熟度と機能の豊富さは本当の資産であり、私たちはSpringでKotlinをうまく使ってきた。

### 92. Netflix DGS
Trial

私たちはサーバーサイドのリソース集約にGraphQLを使うことが多く、様々なテクノロジーを使ってサーバーサイドを実装してきました。Spring Bootで書かれたサービスについて、我々のチームはNetflix DGSで良い経験をしてきた。これはgraphql-javaの上に構築されており、GraphQLエンドポイントの実装やSpring SecurityのようなSpringの機能との統合を容易にするSpring Bootプログラミングモデルの機能と抽象化を提供します。Kotlinで書かれているが、DGSはJavaでも同様に動作する。

https://netflix.github.io/dgs/


### 105. promptfoo
Assess

promptfoo は、テスト駆動型のプロンプト・エンジニアリングを可能にする。アプリケーションにLLMを統合する際、最適で一貫性のある出力を生成するためのプロンプトのチューニングには時間がかかることがあります。promptfooは CLI としてもライブラリとしても使用でき、 あらかじめ定義されたテストケースに対してプロンプトを系統的にテストすることができる。テストケースは、アサーションとともにシンプルな YAML 設定ファイルで設定することができます。この設定ファイルには、テストするプロンプト、モデルプロバイダ、アサーション、そしてプロンプト内で置換される変数値が含まれます。 promptfoo は、等式、JSON 構造、類似性、カスタム関数のチェック、あるい
は LLM を使用したモデル出力の採点など、多くのアサーションをサポートしています。プロンプトやモデルの品質に関するフィードバックの自動化をお考えなら、ぜひromptfooを評価してみてください。

### 107. Spring Modulith
Assess

私たちはマイクロサービスを早くから提唱し、無数のシステムでこのパターンがうまく使われているのを見てきましたが、マイクロサービスが誤って適用され乱用されることも見てきました。新しいシステムを個別にデプロイされたプロセスの集合から始めるよりも、十分にファクタリングされたモノリスから始めて、アプリケーションが分散システム特有の追加の複雑さをマイクロサービスの利点が上回る規模に達したときにのみ、個別にデプロイ可能なユニットを分割することがしばしば推奨される。最近、このアプローチへの関心が再燃しており、ファクタの整ったモノリスの構成要素について、より詳細な定義が行われるようになっている。Spring Modulithは、適切な時にマイクロサービスを簡単に作り出せるようにコードを構造化するフレームワークです。ドメインや境界コンテキストの論理的な概念と、ファイルやパッケージ構造の物理的な概念を一致させるように、コードをモジュール化する方法を提供します。この整合性によって、必要なときにモノリスをリファクタリングし、ドメインを分離してテストすることが容易になります。Spring Modulithはプロセス内のイベントメカニズムを提供し、単一のアプリケーション内のモジュールをさらにデカップリングするのに役立ちます。そして何より、ArchUnitやjmoleculesと統合することで、ドメイン駆動型の設計ルールの検証を自動化できる。

https://spring.io/projects/spring-modulith