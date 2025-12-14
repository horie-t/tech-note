---
title: "Spring AIを使ってみる"
emoji: "🔥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["spring", "springai", "llm", "生成ai", "aiエージェント"]
published: true
---

Spring AIは、Spring Frameworkに統合された生成AIのためのフレームワークです。この記事では、Spring AIを使って簡単なCLIアプリケーションを作成し、OpenAIのAPIを利用してテキスト生成を行う方法を紹介します。

## プロジェクトを作成

以下のコマンドを実行して、Spring Initializrを使用して新しいSpring Bootプロジェクトを作成します。

```bash
curl -G https://start.spring.io/starter.tgz -d dependencies=spring-ai-openai,spring-shell \
    -d type=maven-build -d javaVersion=25 -d bootVersion=3.5.8 \
    -d configurationFileFormat=yaml  -d baseDir=hello-spring-ai | tar -xzvf -
```

## OpenAI APIキーの取得

OpenAIの[API Keys](https://platform.openai.com/api-keys)のページで「Create new secret key」ボタンをクリックしてAPIキーを発行します。

## アプリケーションの設定

`src/main/resources/application.yaml`ファイルを以下の内容に編集し、CLIモードを有効にし、OpenAIのAPIキーを設定します。

```yaml:src/main/resources/application.yaml
spring:
  application:
    name: demo
  main:
    web-application-type: none
  shell:
    interactive:
      enabled: true
  ai:
    openai:
      api-key: ${SPRING_AI_OPENAI_API_KEY}
```

## CLIコマンドの実装

`src/main/java/com/example/demo/ChatCommand.java`ファイルを作成し、以下のコードを追加します。

```java:src/main/java/com/example/demo/ChatCommand.java
package com.example.demo;

import org.springframework.ai.chat.client.ChatClient;
import org.springframework.shell.standard.ShellComponent;
import org.springframework.shell.standard.ShellMethod;

@ShellComponent
public class ChatCommand {
    private final ChatClient chatClient;

    public ChatCommand(ChatClient.Builder chatClient) {
        this.chatClient = chatClient.build();
    }

    @ShellMethod("chat")
    public void chat(String prompt) {
        var answer = chatClient.prompt()
                .user(prompt)
                .call()
                .content();
        System.out.println(answer);
    }
}
```

## アプリケーションの実行

以下のコマンドを実行してアプリケーションを起動します。`SPRING_AI_OPENAI_API_KEY`環境変数に先ほど取得したAPIキーを設定してください。

```bash
export SPRING_AI_OPENAI_API_KEY="your_openai_api_key"
./mvnw spring-boot:run
```

アプリケーションが起動したら、以下のように`chat`コマンドを使用してプロンプトを送信できます。

```shell
$ ./mvnw spring-boot:run
# 中略
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/

 :: Spring Boot ::                (v3.5.8)

2025-12-14T09:00:56.169+09:00  INFO 3842 --- [demo] [           main] com.example.demo.DemoApplication         : Starting DemoApplication using Java 25.0.1 with PID 3842 (/home/tetsuya/repo/programming-study/hello-spring-ai/target/classes started by tetsuya in /home/tetsuya/repo/programming-study/hello-spring-ai)
2025-12-14T09:00:56.170+09:00  INFO 3842 --- [demo] [           main] com.example.demo.DemoApplication         : No active profile set, falling back to 1 default profile: "default"
2025-12-14T09:00:57.156+09:00  INFO 3842 --- [demo] [           main] com.example.demo.DemoApplication         : Started DemoApplication in 1.28 seconds (process running for 1.483)
shell:>chat こんにちは。
こんにちは！どういったことをお話ししましょうか？
shell:>
```

## もっと知りたい方へ

Spring AIをWebアプリケーションにしたり、MCPを実装して利用したりする方法については『[Spring AI入門](https://zenn.dev/thorie/books/spring-ai-for-ai-engineering)』を参照してください。