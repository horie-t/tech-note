---
title: "jbangからProcessingを動かしてみる"
emoji: "🙌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["java", "jbang", "processing"]
published: true
---

ProcessingはJavaベースのビジュアルプログラミング環境で、グラフィックスやインタラクティブなアプリケーションを簡単に作成できるツールです。jbangはJavaのスクリプトランナーで、Javaコードを簡単に実行できるようにするツールです。今回は、jbangを使ってProcessingを動かしてみる方法を紹介します。

## 前提条件

- [sdkman](https://sdkman.io/)を使用してJava 17がインストールされていること

## jbangのインストール

jbangは、Javaコードを簡単に実行できるようにするツールです。以下のコマンドでjbangをインストールします。

```bash
sdk install jbang
```

## Processingのスケッチの作成

ボールが跳ねる簡単なProcessingのスケッチを作成します。以下のコードを`BouncingBall.java`というファイルに保存します。

```java
///usr/bin/env jbang "-bash" "" ; exit 0
//JAVA 17+
//DEPS org.processing:core:4.5.3

import processing.core.PApplet;

public class BouncingBall extends PApplet {

    // ボールの位置と速度
    float x, y, z;
    float vx, vy, vz;
    float cameraAngle = 0;

    public static void main(String[] args) {
        PApplet.main("BouncingBall");
    }

    @Override
    public void settings() {
        size(800, 600, P3D); // 3Dモード
    }

    @Override
    public void setup() {
        sphereDetail(30);
        // 初期状態
        x = 0; y = -200; z = 0;
        vx = 2.5f; vy = 0; vz = 1.5f;
    }

    @Override
    public void draw() {
        background(20, 20, 40);
        lights();

        // カメラを中心に置いて回転させる
        translate(width / 2f, height / 2f, 0);
        rotateY(cameraAngle);
        cameraAngle += 0.005f;

        // 床
        pushMatrix();
        translate(0, 200, 0);
        fill(80, 80, 120);
        box(600, 10, 600);
        popMatrix();

        // 物理計算(重力)
        vy += 0.3f;
        x += vx;
        y += vy;
        z += vz;

        // 壁・床での跳ね返り
        if (y > 175) { y = 175; vy *= -0.92f; }
        if (x > 285 || x < -285) vx *= -1;
        if (z > 285 || z < -285) vz *= -1;

        // ボール
        pushMatrix();
        translate(x, y, z);
        fill(255, 100, 100);
        noStroke();
        sphere(25);
        popMatrix();
    }
}
```

## jbangでスケッチを実行

以下のコマンドで、jbangを使ってProcessingのスケッチを実行します。

```bash
jbang BouncingBall.java
```

これで、Processingのスケッチが実行され、以下のようにボールが跳ねるアニメーションが表示されます。

![Processingのスケッチの実行結果](/images/processing-bouncing-ball.png)
