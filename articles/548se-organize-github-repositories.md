---
title: "GitHubのリポジトリの整理"
emoji: "🔥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["github"]
published: true
---

最近、AIコーディングエージェントを使って実験的なコードを書く事が多くなりました。

AIコーディングエージェントは、Gitリポジトリ単位でコンテキストを認識しているように見えます。なので、実験的なコードを書く毎にリポジトリを作成することになります。ただ、GitHubのリポジトリ一覧に実験の跡が残っているのは避けたいです。できれば、一つの実験用リポジトリにディレクトリを分けて残しておきたいと思いました。

そこで、実験が終わったリポジトリは、以下のようにgit subtreeで記録用リポジトリのディレクトリに移動するようにしました。

## Git Subtreeのインストール

```bash
cd /tmp
git clone https://github.com/git/git.git
cd git/contrib/subtree
make
sudo make install
```

## リポジトリの移動

```bash
cd repo/<記録用リポジトリ名>
git subtree add --prefix=<実験用コードディレクトリ名> https://github.com/<GitHubユーザ名>/<実験用リポジトリ名> main
git push
```