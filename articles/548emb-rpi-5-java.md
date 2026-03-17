---
title: "Raspberry PiでJavaのコードを実行する方法"
emoji: "👋"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Raspberry Pi", "Java", "SDKMAN!", "jbang"]
published: true
---

Javaは、クロスプラットフォームで動作する人気のあるプログラミング言語であり、Raspberry Pi上でも動作させることができます。この記事では、Raspberry PiでJavaを動かす方法について説明します。と言っても、Raspberry PiはLinuxベースのOSを使用しているため、Linux上でJavaを動かす方法とほぼ同じです。

## Javaのインストール

ここでは、SDKMAN!を使用してJavaをインストールする方法を紹介します。SDKMAN!は、複数のJavaバージョンを簡単に管理できるツールです。

```bash
sudo apt update
sudo apt install zip
curl -s "https://get.sdkman.io" | bash
source "$HOME/.sdkman/bin/sdkman-init.sh"
```

インストールが完了したら、以下のコマンドでインストール可能なJavaバージョンを確認できます。

```bash
$ sdk list java
================================================================================
Available Java Versions for Linux ARM 64bit
================================================================================
 Vendor        | Use | Version      | Dist    | Status     | Identifier
--------------------------------------------------------------------------------
 Corretto      |     | 25.0.2       | amzn    |            | 25.0.2-amzn
               |     | 24.0.2       | amzn    |            | 24.0.2-amzn
               |     | 23.0.2       | amzn    |            | 23.0.2-amzn
               |     | 21.0.10      | amzn    |            | 21.0.10-amzn
               |     | 17.0.18      | amzn    |            | 17.0.18-amzn
               |     | 11.0.30      | amzn    |            | 11.0.30-amzn
               |     | 8.0.472      | amzn    |            | 8.0.472-amzn
 Dragonwell    |     | 21.0.10      | albba   |            | 21.0.10-albba
# ... (中略) ...
               |     | 8.0.482      | zulu    |            | 8.0.482-zulu
================================================================================
Omit Identifier to install default version 25.0.2-tem:
    $ sdk install java
Use TAB completion to discover available versions
    $ sdk install java [TAB]
Or install a specific version by Identifier:
    $ sdk install java 25.0.2-tem
Hit Q to exit this list view
================================================================================
```

ここでは、Eclipse TemurinのJava 25.0.2をインストールする例を示します。以下のコマンドを実行してください。

```bash
sdk install java 25.0.2-tem
```

インストールが完了したら、以下のコマンドでJavaが正しくインストールされたことを確認できます。

```bash
$ java -version
openjdk version "25.0.2" 2026-01-20 LTS
OpenJDK Runtime Environment Temurin-25.0.2+10 (build 25.0.2+10-LTS)
OpenJDK 64-Bit Server VM Temurin-25.0.2+10 (build 25.0.2+10-LTS, mixed mode, sharing)
```

Javaのコードをシェルスクリプトと同様に扱いやすくする、「jbang」もインストールしてみましょう。以下のコマンドでjbangをインストールできます。

```bash
sdk install jbang 0.137.0
```

## Javaの実行

jbangを使用して、Javaのコードを簡単に実行できます。例えば、以下のようなJavaコードを含むファイルを作成します。

```bash
jbang init HelloWorld.java
```

このコマンドは、HelloWorld.javaというファイルを作成し、基本的なJavaコードのテンプレートを生成します。

```java
///usr/bin/env jbang "$0" "$@" ; exit $?
//JAVA 25+

void main(String... args) {
    IO.println("Hello World");
}
```

以下のようにシェルスクリプトと同様に実行できます。

```bash
$ ./HelloWorld.java
[jbang] Building jar for HelloWorld.java...
Hello World
```

最初の実行時には、`[jbang] Building jar for HelloWorld.java...`というメッセージが表示されますが、2回目以降はビルドされたjarファイルが再利用されるため、すぐに実行されます。このメッセージを表示させないようにするには、以下のように`--quiet`オプションを使用します。

```java
///usr/bin/env jbang --quiet "$0" "$@" ; exit $?
//JAVA 25+

void main(String... args) {
    IO.println("Hello World");
}
```