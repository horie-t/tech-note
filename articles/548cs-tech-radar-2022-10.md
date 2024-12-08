---
title: "Technology Radar 2022年10月版を読む"
emoji: "🐡"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["technologyradar"]
published: true
---

これからしばらく過去の[Technology Radar](https://www.thoughtworks.com/radar)を読んでみる事にする。[前回](./548cs-tech-radar-2024-10)は[Oct. 2024](https://www.thoughtworks.com/content/dam/thoughtworks/documents/radar/2024/10/tr_technology_radar_vol_31_en.pdf)を読んだので、今回は少し遡って[Oct. 2022](https://www.thoughtworks.com/content/dam/thoughtworks/documents/radar/2022/10/tr_technology_radar_vol_27_en.pdf)を読んでみる。この時期はChatGPTの登場で生成AIバブルが始まる直前のレポートだ。Oct. 2024と比較しながら読んでみるのも面白そうだったのでこれを選んだ。

## 「Oct. 2024」との比較

* 2022年時点だと機械学習に焦点が当たっていて、生成AI（Generative AI）や大規模言語モデル（LLM）への言及がない。
* CI/CDパイプランを観測して効率を上げる点を議論
* Software Bill of Materials（SBOM）やサプライチェーンセキュリティ等のコードの依存関係の管理を注目
* データの管理と分析に関する話題が多い

## 気になった項目

## 20. SLIs and SLOs as code（コード化されたSLIとSLO）

[OpenSLO](https://openslo.com/)を使ってサービスレベル目標(Service Level Objective: SLO)を仕様をコード化する。Prometheus、Datadog、Grafanaなどの監視ツールと簡単に統合可能。Infrastructure as Code（IaC）の延長として、信頼性目標も一元的に管理。

**OpenSLOの記述例**

```yaml
apiVersion: openslo/v1alpha
kind: SLO
metadata:
  name: example-slo
  displayName: Example Service Level Objective
  labels:
    team: backend
spec:
  service: example-service
  indicator:
    type: latency
    metadata:
      description: "API latency less than 100ms"
  objective:
    target: 99.9
  timeWindows:
    - duration: 30d
```

面白そうだとは思うが、実用性はあるのだろうか？

## 52. k6

負荷テストツールで、JavaScriptにテストシナリオを記述して負荷テストを実行する。

Adopt（採用）なのか。

## 62. Spectral

APIのスキーマ検証ツール。OpenAPIやGraphQLのスキーマに対して、設定したルールに従って検証を行う。

どの程度の機能があるのか気になる。コードからスキーマを生成するようにしているので必要ないかもしれないが。

## 63. Styra Declarative Authorization Service

認可サービス。Open Policy Agent（OPA）を使って、アクセスポリシーを管理する。以前にOPAやポリシー言語のRegoについて調べなけければと思っていたので、この辺の仕様から調べたい。

## 80. React Query

Reactのデータフェッチライブラリ。キャッシュやリフレッシュの制御が簡単にできる。栄枯盛衰の激しいReactエコシステムで、今も使われているのか気になる。

## 82. Yjs

分散システムのためのCRDT（Conflict-free Replicated Data Type）ライブラリ。リアルタイムコラボレーションアプリケーションに使える。Adopt（採用）になっている。

バックエンドのストレージが使いやすいのか気になる。類似のライブラリと比較して、共同編集ツールを作ってみたい。

**比較表**

| ライブラリ	      | 主な用途	| 特徴	| サポートプロトコル |
|-------------|---|---|---|
| Yjs	        | | リアルタイムコラボレーション	| 高速、軽量、カスタマイズ性	| WebSocket, WebRTC |
| Automerge	  | ドキュメント同期	| シンプルなJSONモデル	| 任意のネットワーク | 
| Logux	      | Redux互換のリアルタイム同期	| アクションログ管理、競合解決	| WebSocket |
| Replicache	 | クライアント主導の同期	| ローカルキャッシュ、分散データ同期	| 任意のバックエンド |
| ShareDB	    | リアルタイムコラボレーション	| OT/CRDT対応	| WebSocket |
| Gun.js	     | 分散データベース	| オフライン対応、暗号化	| WebRTC, WebSocket |
| Collabs	   | 軽量なデータ同期	| CRDTに特化、シンプル	| 任意のネットワーク |

以下は、CRDTを実装した主要なライブラリと、それらがサポートするデータベースの対応状況をまとめた表です。

| **ライブラリ**    | **サポートするデータベース**                     | **備考**                                                                 |
|-------------------|---------------------------------------------|-----------------------------------------------------------------------|
| **Yjs**           | 任意のデータベース（直接サポートなし）           | データ永続化はアプリケーションが管理（LevelDBやIndexedDBとの統合が可能）。             |
| **Automerge**     | 任意のデータベース（直接サポートなし）           | データの保存や復元を任意の形式で実装可能。永続化にはファイルシステムやデータベースが利用される。 |
| **Logux**         | 任意のデータベース（直接サポートなし）           | ログストレージとしてSQL/NoSQLデータベースをカスタム統合可能。                             |
| **Replicache**    | IndexedDB                                   | ローカルデータキャッシュ用にIndexedDBを使用。バックエンドは独自構築が必要。                       |
| **ShareDB**       | MongoDB、PostgreSQL                        | MongoDBまたはPostgreSQLをバックエンドとしてサポート（カスタムアダプターで他DBにも対応可能）。    |
| **Hypermerge**    | 任意のデータベース（直接サポートなし）           | CRDTドキュメントを保存するためにファイルやデータベースを選択可能。                                |
| **Gun.js**        | 任意のデータベース（直接サポートなし）           | データはデフォルトでWeb Storage（ブラウザ）やファイルに保存。プラグインで他DBにも統合可能。       |
| **Collabs**       | 任意のデータベース（直接サポートなし）           | 永続化にはカスタム実装が必要。CRDTデータをシリアライズして保存する形式に依存。                  |
| **SyncOT**        | MongoDB                                    | MongoDBをバックエンドストレージとしてネイティブサポート。                                   |

## 84. Camunda

ビジネスプロセスの設計、実行、監視を行うためのオープンソースのワークフローおよび意思決定自動化プラットフォーム。特に、BPMN（Business Process Model and Notation）やDMN（Decision Model and Notation）などの標準規格に準拠。

Spring Bootとの統合があるので、試してみたい。

BPMNはXMLで記述されるが、YAMLから変換するツールがあるので、YAMLで記述してからCamundaに読み込ませることもできる。ちょっとしたワークフローの管理に使うのは大げさかもしれないので、開発の業務では使わないかも。このようツールで管理しないといけないワークフロー自体がカイゼンの対象になりそう。

