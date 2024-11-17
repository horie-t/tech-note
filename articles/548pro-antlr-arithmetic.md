---
title: "ARTLR4を使って四則演算の式を計算する 〜ANTLR入門 その2〜"
emoji: "🗂"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["antlr", "java", "compiler"]
published: true
---

[以前の記事](./548pro-antlr-setup)で四則演算の文法を定義して、式を以下のように認識させました。このような認識結果を解析木と呼びます。

![](/images/antlr-grun-example.png)

今回は、Javaで式をパースして解析木を生成し、その解析木を以下の図のようにスキャンしていきます。スキャンながら各exprに対応する値を計算していって、式全体の値(一番上のexpr)を計算します。

![](/images/antlr-grun-arithmetic-expr.png)

前回の`antlr4 -visitor Arithmetic.g4`の実行の結果、以下のソースコードが生成されています。

```bash
$ ls *.java
ArithmeticBaseListener.java  ArithmeticLexer.java     ArithmeticParser.java
ArithmeticBaseVisitor.java   ArithmeticListener.java  ArithmeticVisitor.java
```

上記のクラスを使用しながら、以下のようにコマンドラインから式を受け取って計算するクラスを作成します。

```java

import org.antlr.v4.runtime.CharStream;
import org.antlr.v4.runtime.CharStreams;
import org.antlr.v4.runtime.CommonTokenStream;

public class Calc extends ArithmeticBaseVisitor<Double> {
    public static void main(String[] args) throws Exception {
        // 式をパースする
        CharStream input = CharStreams.fromString(args[0]);
        ArithmeticLexer lexer = new ArithmeticLexer(input);
        CommonTokenStream tokens = new CommonTokenStream(lexer);
        ArithmeticParser parser = new ArithmeticParser(tokens);
        parser.setBuildParseTree(true);
        ArithmeticParser.ExprContext expr = parser.expr();

        // 式を計算する
        Calc calc = new Calc();
        double result = calc.visit(expr);
        System.out.println(result);
    }

    @Override
    public Double visitExpr(ArithmeticParser.ExprContext ctx) {
        if (ctx.NUMBER() != null) {
            // 子ノードが数字の場合
            return Double.parseDouble(ctx.NUMBER().getText());
        } else if (ctx.expr().size() == 1) {
            // 括弧で囲まれた式の場合
            return visit(ctx.expr(0));
        } else {
            // 四則演算の場合
            double left = visit(ctx.expr(0));
            double right = visit(ctx.expr(1));
            return switch (ctx.op.getText()) {
                case "+" -> left + right;
                case "-" -> left - right;
                case "*" -> left * right;
                case "/" -> left / right;
                default -> throw new RuntimeException("Unknown operator: " + ctx.op.getText());
            };
        }
    }
}
```

ここでは以下のように実行される事を想定しています。

```bash
java Calc "3 + 5 * (2 - 8)"
```

以下の部分で、引数の式をパースします。

```java
    public static void main(String[] args) throws Exception {
        // 式を解析する
        CharStream input = CharStreams.fromString(args[0]);
        ArithmeticLexer lexer = new ArithmeticLexer(input);
        CommonTokenStream tokens = new CommonTokenStream(lexer);
        ArithmeticParser parser = new ArithmeticParser(tokens);
        parser.setBuildParseTree(true);
        ArithmeticParser.ExprContext expr = parser.expr();
```


`Calc`クラスは、以下のように`ArithmeticParser`がパースした式(`expr`)を引数に`visit`メソッドを呼び出すと、前述の図のexprをスキャンしていきます。

```java
        // 式を計算する
        Calc calc = new Calc();
        double result = calc.visit(expr);
        System.out.println(result);
```

スキャン時に以下のメソッドが各expr毎に呼び出されます。ここで`ctx`はexprに対応するオブジェクトです。

```java
    @Override
    public Double visitExpr(ArithmeticParser.ExprContext ctx) {
        if (ctx.NUMBER() != null) {
            // 子ノードが数字の場合
            return Double.parseDouble(ctx.NUMBER().getText());
        } else if (ctx.expr().size() == 1) {
            // 括弧で囲まれた式の場合
            return visit(ctx.expr(0));
        } else {
            // 四則演算の場合
            double left = visit(ctx.expr(0));
            double right = visit(ctx.expr(1));
            return switch (ctx.op.getText()) {
                case "+" -> left + right;
                case "-" -> left - right;
                case "*" -> left * right;
                case "/" -> left / right;
                default -> throw new RuntimeException("Unknown operator: " + ctx.op.getText());
            };
        }
    }
```

以下のような数値に対応する`expr`の場合

![](/images/antlr-grun-arithmetic-number.png)

`ctx.NUMBER()`は`3`のトークンが返します。以下のように3を返すようにします。

```java
        if (ctx.NUMBER() != null) {
            // 子ノードが数字の場合
            return Double.parseDouble(ctx.NUMBER().getText());
```

また、以下のような減算に対応する`expr`の場合

![](/images/antlr-grun-arithmetic-sub.png)

以下のように、2つの子ノード、expr(0), expr(1)から値を取得して、演算子の文字列`op`から演算子を判定して、減算を実行した結果を返すようにします。

```java
            double left = visit(ctx.expr(0));
            double right = visit(ctx.expr(1));
            return switch (ctx.op.getText()) {
                // 中略
                case "-" -> left - right;
```

上記のように、ArithmeticBaseVisitorのvisitExprメソッドをオーバーライドしてやると

```java
        double result = calc.visit(expr);
```

は下図のexprの計算結果(式全体の計算結果)を返すようになります。

![](/images/antlr-grun-arithmetic-root.png)

ソースコードをコンパイルして実行すると以下のようになります。

```bash
$ javac *.java
$ java Calc "3 + 5 * (2 - 8)"
-27.0
```

以上で、ANTLRを使用して四則演算を計算できるようになりました。