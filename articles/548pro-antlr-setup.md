---
title: "ANTLR4のインストールとセットアップ　〜ANTLR入門 その1〜"
emoji: "📚"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["antlr", "java", "compiler"]
published: true
---

## ANTLRとは？

ANTLR（Another Tool for Language Recognition）は、構文解析器を生成する、パーサジェネレータもしくはコンパイラコンパイラと呼ばれるツールでです。主にプログラミング言語やデータフォーマットのパーサーを作成するために使用されます。ANTLRで文法を定義すると、その文法からJavaなどの構文解析のためのプログラムを自動生成できます。

## ANTLRのターゲット言語

ANTLRが構文解析器を生成できるターゲット言語は以下の通りです：

1. Java
2. C#
3. Python
4. JavaScript
5. TypeScript
6. C++
7. Swift
8. Go
9. PHP
10. Dart

## ANTLRの用途

ANTLR（Another Tool for Language Recognition）は、主に以下の用途で使用されます：

1. **プログラミング言語のパーサー作成**：  
   新しいプログラミング言語やDSL（ドメイン固有言語）の構文解析器を作成するために使用されます。
2. **データフォーマットのパーサー作成**：  
   JSON、XML、CSVなどのデータフォーマットを解析するためのパーサーを作成できます。
3. **トランスパイラ**：  
   あるプログラミング言語から別のプログラミング言語への変換ツールを作成するために使用されます。
4. **コード解析ツール**：  
   ソースコードの静的解析ツールやリファクタリングツールを作成するために使用されます。
5. **コンパイラ**：  
   プログラミング言語のコンパイラの一部として、構文解析フェーズを担当します。

## ANTLRのインストール

ANTLRを使用するためには、まずJavaがインストールされている必要があります。Javaをインストールし終えたら、以下の手順でANTLRをインストールします。

1. ANTLRのJARファイルをダウンロードします。バージョンは[ANTLRのダウンロードページ](https://www.antlr.org/download.html)で確認してください
   ```sh
   cd /usr/local/lib
   curl -O https://www.antlr.org/download/antlr-4.13.2-complete.jar
   ```
2. 環境変数を設定して、ANTLRを簡単に実行できるようにします。
   ```sh
   export CLASSPATH=".:/usr/local/lib/antlr-4.13.2-complete.jar:$CLASSPATH"
   alias antlr4='java -jar /usr/local/lib/antlr-4.13.2-complete.jar'
   alias grun='java org.antlr.v4.runtime.misc.TestRig'
   ```

## ANTLRの使い方

ANTLRを使用して四則演算をパースする方法を紹介します。

1. **文法ファイルの作成**    
   適当なディレクトリを作成し、`Arithmetic.g4`という名前の文法ファイルを作成します。
   ```antlr:Arithmetic.g4
   grammar Arithmetic;
   
   expr: expr op=('*'|'/') expr
       | expr op=('+'|'-') expr
       | NUMBER
       | '(' expr ')'
       ;
   
   NUMBER: [0-9]+ ('.' [0-9]+)?;
   WS    : [ \t\r\n]+ -> skip;
   ```
2. **文法ファイルからコードを生成**  
   ANTLRを使用して文法ファイルからJavaコードを生成します。
   ```sh
   antlr4 -visitor Arithmetic.g4
   ```
3. **生成されたJavaファイルをコンパイル**  
   生成されたJavaファイルをコンパイルします。
   ```sh
   javac Arithmetic*.java
   ```
4. **パーサーを実行**  
   ANTLRのテストリグを使用して、四則演算を解析できるかテストしてみます。`Ctrl+D`で入力を終了させると、入力された式がパースされ、結果のウィンドウが表示されます。
   ```sh
   grun Arithmetic expr -gui
   3 + 5 * (2 - 8)
   Ctrl+D  # 入力を終了させます。
   ```
  ![](/images/antlr-grun-example.png)

これで、ANTLRを使用して四則演算をパースする基本的な手順が完了しました。[次回](./548pro-antlr-arithmetic)は、このパーサを使用して四則演算の式を評価する方法について説明します。

