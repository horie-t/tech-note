---
title: "Unleash + Spring Bootでフィーチャーフラグを実装する"
emoji: "🌊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["unleash", "featureflag", "springboot"]
published: true
---

## フィーチャーフラグとは

フィーチャーフラグは、フィーチャートグル、フィーチャースイッチ等とも呼ばれます。フィーチャーフラグはコードを変更せずにシステムの動作を変更するテクニックの総称です。フィーチャーフラグを使うことによって、

1. 開発途上のコードをmainブランチにマージしても無効化して動作しないようにする事ができ、頻繁にmainブランチにマージして開発ブランチが長期間存在しないようにして、マージ時のコンフリクトを減らす事ができ、また
2. フラグによってA/Bテストを実行する事もでき、
3. 新しい機能やライブラリを導入した事による悪影響があった場合に、切り戻しのデプロイをせずに機能を無効化する事によって元に戻す事ができ、
4. 社内ユーザやベータユーザにのみ新機能を公開する事ができ

るようになります。

[Unleash](https://www.getunleash.io/)はオープンソースのフィーチャーフラグ管理のプラットフォームです。本記事ではUnleashのサーバを構築して、Spring Bootからフィーチャーフラグの値を取得する実装方法を解説します。

## UnleashをローカルPCでセットアップ

UnleashのサーバをローカルPCでセットアップします。(本記事での方法はローカルPC上での動作確認のための方法であり、デフォルトのパスワードを使用するなどのセキュリティ上の問題があります。本番環境で使用する場合は別の方法でセットアップするようにしてください。)

#### Unleashサーバの起動

UnleashのサーバはDockerイメージとして提供されているので、Dockerを使ってローカルPC上で起動させます。適切なディレクトリ上で、以下のようにUnleashサーバを起動します。

```bash
wget https://raw.githubusercontent.com/Unleash/unleash/refs/heads/main/docker-compose.yml
docker compose up -d
```

#### 管理画面へのサインイン

ブラウザで `http://localhost:4242` にアクセスします。以下のサインイン画面が表示されます。以下の認証情報を入力して`Sign in`ボタンをクリックします。

User name: `admin`
Password: `unleash4all`

![](/images/unleash-mng-sigin.png)

#### フィーチャーフラグの作成

左上のメニューから `Projects` を選択します。
![](/images/unleash-menu-projects.png)

`Default` プロジェクトを選択します。
![](/images/unleash-projects-default.png)

`Create flag` ボタンをクリックします。
![](/images/unleash-default-project-create-flag.png)

`Feature flag name`、`Description`にフラグの名前と説明を入力し `Create feature flag` ボタンをクリックします。本記事ではタスク管理サービスで締め切り日時の管理機能の追加を例に解説するので名前を `support-due-date-time` としました。
![](/images/unleash-create-flag-name.png)

#### トークンの発行

`Connect SDK` ボタンをクリックします。
![](/images/unleash-default-project-connect-sdk.png)

`Server side SDKs`の`Java`を選択します。
![](/images/unleash-connect-sdk-select-sdk.png)

`Enviroment` を選択して、`Next`ボタンをクリックします。今回はローカルPCで動かすだけなので`development`を選択しています。
![](/images/unleash-connect-sdk-environment.png)

接続のための設定が表示され、クライアントからの接続待ち状態になります。
![](/images/unleash-connect-sdk-wait-connect.png)

## Spring Bootからの接続テスト

Unleashサーバでの準備ができたので、サーバに接続できるかをテストします。

#### Spring Bootプロジェクトの作成

[Spring Intializr](https://start.spring.io/)で以下のように設定して `GENERATE`ボタンをクリックします。
![](/images/unleash-spring-initializr.png)

プロジェクトをダウンロードして適切なディレクトリで展開します。

pom.xmlに以下を追加します。テストが終わったらこの記述は削除します。
```xml:pom.xml
<dependency>
    <groupId>io.getunleash</groupId>
    <artifactId>unleash-client-java</artifactId>
    <version>LATEST</version>
</dependency>
```

Spring BootのApplicationクラスに以下のように記述して動作確認します。テストが終わったら追加した喜寿るは削除します。

```java:src/main/java/com/t_horie/unleash_demo/UnleashDemoApplication.java
@SpringBootApplication
public class UnleashDemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(UnleashDemoApplication.class, args);

		// UnleashClientを初期化
		UnleashConfig config = UnleashConfig.builder()
				.appName("unleash-onboarding-java")
				.instanceId("unleash-onboarding-instance")
				.unleashAPI("http://localhost:4242/api/")
				.apiKey("default:development.unleash-insecure-api-token") // in production use environment variable
				.build();

		Unleash unleash = new DefaultUnleash(config);

		while (true) {
			boolean featureEnabled = unleash.isEnabled("support-due-date-time");
			System.out.println("Feature enabled: " + featureEnabled);
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }
	}

}
```

Spring Bootのプロジェクトの直下のディレクトリで、以下のようにSpring Bootを起動してUnleashサーバに接続できるかをテストします。コンソールに `Feature enabled: false`が繰り返し表示されれば成功です。

```bash
$ ./mvnw spring-boot:run
[INFO] Scanning for projects...
# 中略
[INFO] Attaching agents: []

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/

 :: Spring Boot ::                (v3.4.0)
# 中略
2024-11-30T12:59:56.615+09:00  INFO 75693 --- [unleash-demo] [           main] i.g.repository.FeatureBackupHandlerFile  : Unleash will try to load feature toggle states from temporary backup
Feature enabled: false
Feature enabled: false
Feature enabled: false
```

接続が成功するとUnleashの管理画面のダイアログの`Connection status`が`Connected`に変化します。
![](/images/unleash-connect-sdk-connected.png)

## フィーチャーフラグを使ったAPIサービスの実装

タスク管理アプリのAPIサーバで、フィーチャーフラグが有効になるとレスポンスに締め切り日時の`dueDateTime`フィールドが追加されるケースを考えてみます。

#### 締め切り機能追加前の状態を実装

締め切りがサポートされる前のController、Service、Responseは以下の通りだったとします。

```java:src/main/java/com/t_horie/unleash_demo/TasksController.java
@RestController
@RequestMapping("/tasks")
public class TasksController {
    private final GetTasksUsecase getTasksUsecase;

    public TasksController(GetTasksUsecase getTasksUsecase) {
        this.getTasksUsecase = getTasksUsecase;
    }

    @GetMapping
    public TasksResponse getTasks() {
        return new TasksResponse(getTasksUsecase.getTasks());
    }
}
```

```java:src/main/java/com/t_horie/unleash_demo/TasksController.java
public interface GetTasksUsecase {
    List<Task> getTasks();
}
```

```java:src/main/java/com/t_horie/unleash_demo/GetTaskService.java
@Service
public class GetTaskService implements GetTasksUsecase {
    @Override
    public List<Task> getTasks() {
        // 説明を簡略化するためリポジトリは作成せずに固定のデータを返す実装にしています。
        return List.of(
                new Task("1", "task1", "description1"),
                new Task("2", "task2", "description2"),
                new Task("3", "task3", "description3")
        );
    }
}
```

```java:src/main/java/com/t_horie/unleash_demo/TasksResponse.java
@Data
public class TasksResponse {
    private final List<Task> results;
}
```

```java:src/main/java/com/t_horie/unleash_demo/Task.java
@Data
public class Task {
    private final String id;
    private final String title;
    private final String description;
}
```

上記を実装して、Spring Bootのプロジェクトのディレクトリ直下で、以下のようにSpring Bootを起動します。

```bash
./mvnw spring-boot:run
```

アプリケーションを起動後に、別のコンソール画面でタスク取得APIを呼び出すと以下のレスポンスが返ってきます。

```json
$ curl -s http://localhost:8080/tasks | jq .
{
  "results": [
    {
      "id": "1",
      "title": "task1",
      "description": "description1"
    },
    {
      "id": "2",
      "title": "task2",
      "description": "description2"
    },
    {
      "id": "3",
      "title": "task3",
      "description": "description3"
    }
  ]
}
```

#### 締め切り機能をフィーチャーフラグと共に実装

##### Spring Bootプロジェクトの構成の変更

まず、pom.xmlファイルでテスト接続の記述を以下のように書き換えます。

```xml:pom.xml
		<dependency>
			<groupId>io.getunleash</groupId>
			<artifactId>springboot-unleash-starter</artifactId>
			<version>1.1.0</version>
		</dependency>
```

次に、Unleashサーバへの接続構成をapplication.yamlに記述します。

```yaml:src/main/resources/application.yaml
io:
  getunleash:
    app-name: unleash-onboarding-java
    instance-id: unleash-onboarding-instance
    environment: development
    api-url: "http://localhost:4242/api"
    api-token: "default:development.unleash-insecure-api-token"
```

##### 締め切り機能の実装

準備ができたら、Controller、Service、Responseクラスを変更して締め切り機能を実装していきます。

ユースケースのインタフェースに、以下のようにUnleashのコンテキストやアノテーションを追加します。 `@Toggle`の`name`はUnleashの管理画面で登録したフィーチャーフラグの名前を指定します。`alterBean`は後述の、フィーチャーフラグが有効になった時に使うServiceの名前です。

```java:src/main/java/com/t_horie/unleash_demo/GetTasksUsecase.java
public interface GetTasksUsecase {
    @Toggle(name = "support-due-date-time", alterBean = "getTasksServiceNew")
    List<Task> getTasks(UnleashContext context);
}
```

Serviceはフィーチャーフラグの有効/無効の2つの状態に対応したものを実装します。

**フラグが無効の場合用**

```java:src/main/java/com/t_horie/unleash_demo/GetTaskService.java
@Service("getTaskServiceOld")
public class GetTaskService implements GetTasksUsecase {
    @Override
    public List<Task> getTasks(UnleashContext context) {
        // 説明を簡略化するため固定のデータを返しています。
        return List.of(
                new Task("1", "task1", "description1"),
                new Task("2", "task2", "description2"),
                new Task("3", "task3", "description3")
        );
    }
}
```

**フラグが有効の場合用**

```java:src/main/java/com/t_horie/unleash_demo/GetTaskServiceDue.java
@Service("getTasksServiceNew")
public class GetTaskServiceDue implements GetTasksUsecase {
    public List<Task> getTasks(UnleashContext context) {
        // 説明を簡略化するため固定のデータを返しています。
        return List.of(
                new TaskDue("1", "task1", "description1", "2021-01-01T00:00:00Z"),
                new TaskDue("2", "task2", "description2", "2021-01-02T00:00:00Z"),
                new TaskDue("3", "task3", "description3", "2021-01-03T00:00:00Z")
        );
    }
}
```

ここで `TaskDue` クラスは下記の通りです。

```java:src/main/java/com/t_horie/unleash_demo/TaskDue.java
@Data
@EqualsAndHashCode(callSuper = true)
public class TaskDue extends Task {
    private String dueDateTime;

    public TaskDue(String id, String title, String description, String dueDateTime) {
        super(id, title, description);
        this.dueDateTime = dueDateTime;
    }
}
```

Controllerでは、ユースケースに機能が無効になっている場合の `Qualifier` を指定しています。またユースケース呼び出し時には、Unleashのコンテキストを渡すようにします。

```java:src/main/java/com/t_horie/unleash_demo/TasksController.java
@RestController
@RequestMapping("/tasks")
public class TasksController {
    private final GetTasksUsecase getTasksUsecase;

    public TasksController(@Qualifier("getTaskServiceOld") GetTasksUsecase getTasksUsecase) {
        this.getTasksUsecase = getTasksUsecase;
    }

    @GetMapping
    public TasksResponse getTasks() {
        return new TasksResponse(getTasksUsecase.getTasks(UnleashContext.builder().build()));
    }
}
```

この状態では、Unleashサーバ側でのフィーチャーフラグが有効になっていないので無効の場合のServiceクラスが呼ばれ、締め切りのレスポンスは返ってきません。

##### 締め切りフィーチャーフラグの有効化

Unleashの管理画面で `support-due-date-time` のフィーチャーフラグを `develop`環境で有効化します。

![](/images/unleash-flag-enable.png)

有効化してしばらくすると締め切りの入ったレスポンスが返ってくるようになります。

```json
$ curl -s http://localhost:8080/tasks | jq .
{
  "results": [
    {
      "id": "1",
      "title": "task1",
      "description": "description1",
      "dueDateTime": "2021-01-01T00:00:00Z"
    },
    {
      "id": "2",
      "title": "task2",
      "description": "description2",
      "dueDateTime": "2021-01-02T00:00:00Z"
    },
    {
      "id": "3",
      "title": "task3",
      "description": "description3",
      "dueDateTime": "2021-01-03T00:00:00Z"
    }
  ]
}
```