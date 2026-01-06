---
title: "VPythonを使ってみた"
emoji: "💬"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Python", "VPython"]
published: false
---

VPythonは、Pythonでグラフや3Dグラフィックス等の可視化をできるライブラリです。物理シミュレーションや教育目的でよく使われています。この記事では、VPythonの基本的な使い方を紹介します。

## VPythonのインストール

VPythonを使用するには、まずPython環境にインストールする必要があります。uvを使っている場合は、以下のコマンドでインストールできます。

```bash
uv add vpython
```

## 基本的な使い方

VPythonを使って簡単なグラフを描画してみます。

```python:sin_graph.py
from vpython import *

scene = canvas()
graph(title = 'sin関数', xtitle = 'x', ytitle = 'sin(x)')
sin_plot = gcurve(color = color.red)
for x in arange(0.0, 2.0 * pi + 0.1, 0.1):
  sin_plot.plot(x, sin(x))
```

このコードでは、sin関数のグラフを描画しています。`canvas()`で描画領域を作成し、`gcurve()`でグラフの属性を設定しています。`arange()`を使ってxの値を生成し、`sin_plot.plot()`で各点をプロットしています。

以下のコマンドを実行して、上記のコードを実行できます。

```bash
uv run sin_graph.py
```

コマンドを実行すると、ブラウザが開き、sin関数のグラフが表示されます。

![](/images/vpython-sin-curve.png)
