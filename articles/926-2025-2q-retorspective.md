---
title: "2025年6月の振り返り"
emoji: "😸"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["振り返り", "ポエム"]
published: true
---

2025年も6ヶ月が過ぎたので、ここで振り返ってみる。[前回](https://zenn.dev/thorie/articles/926-2025-1q-retorspective)は3月までを振り返ったので、4〜6月を振り返る。

AIコーディング・エージェントが話題なので、[Claude Code](https://zenn.dev/thorie/scraps/d464f885571fee)、[OpenAI Codex CLI](https://zenn.dev/thorie/scraps/d464f885571fee)、[Junie](https://zenn.dev/thorie/scraps/5c0c1c11e247b8)を使ってみる。全く話題になっていないけど、Junieが一番優秀な気がした。セッション外でコードを弄ってもちゃんとそれについて来ている。IntelliJはプロジェクトのインデックスを作成しているのでそれを利用している感じがある。

MCPに対応したSaaSサービスが続々と出ているので、MCP Serverを自分でも[実装してみた](https://zenn.dev/thorie/articles/548ics-mcp-server)。ローカルファイルやバイナリデータの処理方法やリモートのServerでの認証周り等を深く調べてみたい。

## 主にやっていた事

* **4月**: [OmniMouse](https://github.com/horie-t/omni-mouse)の開発。カメラの校正関係を実装。Zennの記事もいくつか([*1](https://zenn.dev/thorie/articles/548emb-rpi-5-camera_lens_shading_calibration), [*2](https://zenn.dev/thorie/articles/548emb-rpi-5-calib-fisheye-image), [*3](https://zenn.dev/thorie/articles/548emb-rpi-5-venv-system-site-package))書けたので良かった。Pythonの仮想環境についての理解ができたのも収穫だった。
* **5月**: ゴールデン・ウィークは[Kubernetes](https://kubernetes.io/)の勉強。その後は、各種AIコーディング・エージェントを触ってみた。[*4](https://zenn.dev/thorie/scraps/d464f885571fee), [*5](https://zenn.dev/thorie/scraps/6db02729714657), [*6](https://zenn.dev/thorie/scraps/5c0c1c11e247b8)。[AWS EKS on EC2環境でArgoCDの継続的デプロイをIaC化する](https://zenn.dev/thorie/scraps/5ee2529197a47f)のに成功。会社の技術ブログにまとめておいた。
* **6月**: [MCP(Model Context Protocol) Server](https://modelcontextprotocol.io/specification/2025-06-18)を[実装する](https://zenn.dev/thorie/articles/548ics-mcp-server)。[OmniMouse](https://github.com/horie-t/omni-mouse)の開発。カメラの校正関係をリファクタリング。


## 勉強したプログラミング言語・ライブラリ等

* [Model Context Protocol](https://modelcontextprotocol.io/introduction)
* [Kubernetes](https://kubernetes.io/)
* [ArgoCD](https://argoproj.github.io/cd/)
* [Claude Code](https://docs.anthropic.com/ja/docs/claude-code/overview)
* [OpenAI Codex CLI](https://github.com/openai/codex)
* [Junie](https://www.jetbrains.com/ja-jp/junie/)

## 読書

### コンピュータ技術書

* [Continuous Deployment](https://amzn.to/4lGMLlO): (Chap. 1, 2, 3, 4, 5, 6)  
  thoughtworksの[Technorogy Radarの項目](https://www.thoughtworks.com/radar/techniques/continuous-deployment)で紹介されていたので読んでいる。
* [Human-in-the-Loop 機械学習](https://amzn.to/3YBSFdZ): (第1, 2, 3章)  
  機械学習には興味があるが、実際に使うとなるとアノテーションとかしている時間は取れないと思って勉強だけで終わってしまうのを止めたいので読む。
* [マルチテナントSaaSアーキテクチャの構築](https://amzn.to/4ietwOt): (第4, 5, 6, 7, 8, 9, 10, 11, 12, 13章)  
  6章まで要件とアーキテクチャの分類が延々と続くし、繰り返しが多い。具体例に入っても、実装のコードや設定の説明が少ない。
* [Kubernetes完全ガイド 第2版](https://amzn.to/3O0zcyS): (第1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20章; 読了)  
  一度読んだけれど、EKS上で動かしながら読んでみる。EKSだとPersistentVolume関連が全く違うので難しい。
* [スプリントゴールで価値を駆動しよう](https://amzn.to/4nep1Gx)  (第1, 2, 3, 4, 5, 6, 7, 8, 9, 10章)  
  定期的に、今のスクラムのやり方に疑問を感じてしまって、この手の本に手を出してしまっている。他のスクラムの本とは違う部分も多いので業務にそのまま取り入れるのは難しそう

### 数学・科学系専門書

* [初学の編集者がわかるまで書き直した　基礎から鍛える量子力学](https://amzn.to/3YdEdtd): (第7, 8, 9章)  
  かなり難しくなってきたので、前に戻ったりしながら読み進めている。

### その他専門書

* [これからの「正義」の話をしよう ──いまを生き延びるための哲学](https://amzn.to/46HmlYk): (第1, 2章)  
  ずっと積読状態だったのを読み出した。[利己的な遺伝子](https://amzn.to/449gygV)の考え方を持ち込むと、この本の問い自体を破壊してしまいそう。
* [キャズム Ver.2 増補改訂版](https://amzn.to/3GigBNo): (第1, 2, 3章)  
  初版を20年ぐらい前に読んだが、記憶が薄れているので読んでみる。

### 小説等の一般書

* [フォース・ウィング－第四騎竜団の戦姫](https://amzn.to/44mLOsX)  
  「2025年本屋大賞 翻訳小説部門　第1位」ということで、久し振りに本屋大賞の受賞作を読んでみた。アメリカのAmazonでも100週以上もベスト20に入り続けているベストセラー。「ロマンタジー」(ロマンス + ファンタジー)と呼ばれるジャンルの作品。優秀な親・兄弟の中でちょっと落ちこぼれた感じの女の子が、大学で家族の仇の子供同士の関係にあるイケメンの先輩といろいろあって恋に落ちる話。テンプレ要素がぎっしりの作品で、アメリカでもこういう話は大人気なのね。