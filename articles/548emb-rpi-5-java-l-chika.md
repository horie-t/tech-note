---
title: "JavaでLチカをやってみた"
emoji: "🙌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["java", "raspberrypi", "pi4j"]
published: true
---

Javaは元々は[家電製品のマイクロコントローラ向けに開発された](https://ja.wikipedia.org/wiki/Java#%E5%AE%B6%E9%9B%BB%E5%90%91%E3%81%91%E3%83%97%E3%83%AD%E3%82%B8%E3%82%A7%E3%82%AF%E3%83%88%E3%81%AE%E7%AB%8B%E3%81%A1%E4%B8%8A%E3%81%92%EF%BC%881990%E5%B9%B412%E6%9C%88%EF%BC%89)言語で、組み込みシステムの分野でも広く使われています。今回は、Raspberry Piを使ってJavaでLチカをやってみました。周辺機器の制御には、Pi4JというJavaのライブラリを使用しました。

## LEDの接続

Raspberry PiのGPIOピンを使ってLEDを接続します。今回は、GPIO25番ピンを使用しました。LEDのアノード（長い方）を抵抗(330Ω)を介してGPIO25番ピンに接続し、カソード（短い方）をGNDに接続します。

![LEDの接続図](/images/LChika_Breadboard.png)

## プロジェクトの作成

適当なディレクトリに移動して、以下のコマンドでプロジェクトを作成します。

```bash
mvn archetype:generate -DarchetypeArtifactId=maven-archetype-quickstart \
    -DinteractiveMode=false -DgroupId=com.example -DartifactId=l-chika
```

プロジェクトが作成されたら、`pom.xml`にPi4Jの依存関係を追加します。

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.example</groupId>
  <artifactId>l-chika</artifactId>
  <packaging>jar</packaging>
  <version>1.0-SNAPSHOT</version>
  <name>l-chika</name>
  <url>http://maven.apache.org</url>
  <properties>
    <java.version>25</java.version>
    <slf4j.version>2.0.17</slf4j.version>
    <pi4j.version>4.0.0</pi4j.version>
  </properties>
  <dependencies>
    <!-- ログ出力用 -->
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-api</artifactId>
      <version>${slf4j.version}</version>
    </dependency>
    <dependency>
      <groupId>org.slf4j</groupId>
      <artifactId>slf4j-simple</artifactId>
      <version>${slf4j.version}</version>
    </dependency>
    <!-- Pi4J Core -->
    <dependency>
      <groupId>com.pi4j</groupId>
      <artifactId>pi4j-core</artifactId>
      <version>${pi4j.version}</version>
    </dependency>

    <!-- Pi4J Plugin (Platforms and I/O Providers) -->
    <dependency>
      <groupId>com.pi4j</groupId>
      <artifactId>pi4j-plugin-ffm</artifactId>
      <version>${pi4j.version}</version>
    </dependency>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
  <build>
    <plugins>
      <!-- Fat JAR を作成するための設定-->
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <configuration>
          <release>${java.version}</release>
          <showDeprecation>true</showDeprecation>
          <showWarnings>true</showWarnings>
          <verbose>false</verbose>
        </configuration>
      </plugin>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-jar-plugin</artifactId>
        <configuration>
          <archive>
            <manifest>
              <mainClass>com.example.l_chika.App</mainClass>
            </manifest>
            <manifestEntries>
              <!-- Javaネイティブアクセスを許可するためのエントリ -->
              <Enable-Native-Access>ALL-UNNAMED</Enable-Native-Access>
            </manifestEntries>
          </archive>
        </configuration>
      </plugin>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-shade-plugin</artifactId>
        <configuration>
          <transformers>
            <transformer implementation="org.apache.maven.plugins.shade.resource.ServicesResourceTransformer"/>
          </transformers>
        </configuration>
        <executions>
          <execution>
            <phase>package</phase>
            <goals>
              <goal>shade</goal>
            </goals>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
</project>
```

## コードの実装

`src/main/java/com/example/l_chika/App.java`を以下の内容で作成します。

```java
package com.example.l_chika;

import com.pi4j.Pi4J;
import com.pi4j.io.gpio.digital.DigitalState;

public class App {
    private static final int PIN_LED = 25;

    public static void main( String[] args ) {
        var pi4j = Pi4J.newAutoContext();

        try (var led = pi4j.digitalOutput().create(PIN_LED);){
            for (int i = 0; i < 10; i++) {
                if (led.state() == DigitalState.HIGH) {
                    led.low();
                } else {
                    led.high();
                }
                    Thread.sleep(500);
            }
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }

        pi4j.shutdown();
    }
}
```

## ビルドと実行

以下のコマンドでビルドします。

```bash
mvn clean package
```

ビルドが成功すると、`target/l-chika-1.0-SNAPSHOT.jar`が生成されます。JarファイルをRaspberry Pi上で以下のコマンドで実行します。

```bash
java -jar target/l-chika-1.0-SNAPSHOT.jar
```