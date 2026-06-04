---
title: "関数型ドメインモデリングをJavaで実装してみる"
emoji: "👌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["関数型ドメインモデリング", "java", "vavr", "springboot"]
published: true
---
注意: 本記事の記述及び、移植したコードはAIによる生成物です。人による作業は、採用する技術スタック、アーキテクチャの選定、及びレビューのみを行っています。AIへの指示(プロンプト)は記事の末尾を参照してください。このような指示のみで、F# のコードを Java に移植できることを示すのが本記事の目的です。

## はじめに

[『関数型ドメインモデリング』(Domain Modeling Made Functional)](https://tatsu-zine.com/books/domain-modeling-made-functional) は、Scott Wlaschin による型駆動・関数型のドメインモデリングの入門書です。サンプルコードは F# で書かれており、「不正な状態を表現できない型」を組み立て、ワークフローを純粋関数の合成として表現していくスタイルが一貫しています。

この本の受注システム (OrderTaking、書籍 9〜12 章のコード) を **Java + Spring Boot に移植**してみました。([GitHub リポジトリ](https://github.com/horie-t/programming-study/tree/master/functional_domain_modeling_in_java)) 狙いは「関数型の型駆動設計を、F# の言語機能に頼らず Java でどこまで自然に表現できるか」を手を動かして確かめることです。

結論から言うと、**`sealed interface` + `record` + Vavr の `Either`** という 3 点セットで、F# のコードの「形」をかなり忠実に再現できました。本記事では F# と Java を対比しながら、移植のパターンとハマりどころを紹介します。

## 技術スタック

- **Java 25**(`record`、`sealed interface`、パターンマッチング switch)
- **Spring Boot 4**(REST API)
- **[Vavr](https://www.vavr.io/) 0.10.3**(`Either` / `Option` / 永続コレクション)

アーキテクチャはヘキサゴナル(Ports & Adapters)で、ドメイン層は Spring に依存させず純粋に保ちました。

```
com.example.fdmj/
├── domain/
│   ├── model/     # 値オブジェクト・代数的データ型 (sealed interface + record)
│   └── service/   # ワークフローの各ステップ (純粋関数)
├── adapter/
│   ├── in/web/    # REST コントローラ・DTO
│   └── out/       # 製品カタログ等のダミー実装
├── application/   # ユースケース実装 (各ステップを Either で合成)
│   └── port/{in,out}
└── config/        # Spring の DI 配線
```

## 移植の基本方針

F# の道具を Java の何に対応させるかを、最初に決めておきました。

| F# | Java |
| --- | --- |
| 単一ケース判別共用体 + スマートコンストラクタ | `record` + 静的ファクトリ(`Either<String, T>` を返す) |
| 判別共用体 (DU / 直和型) | `sealed interface` + `record` |
| `Result<'T, 'E>` | Vavr `Either<L, R>` |
| `'T option` | Vavr `Option<T>` |
| `result { let! ... }`(コンピュテーション式) | `Either` の `flatMap` ネスト |
| 関数引数による依存性注入 | コンストラクタ注入 + Spring `@Configuration` |
| `'T list` | Vavr `io.vavr.collection.List`(不変リスト) |

以下、要点を順に見ていきます。

## 1. 制約付き値オブジェクト

「50 文字以下の文字列」のような制約付きの型を、F# は**プライベートコンストラクタを持つ単一ケース DU + スマートコンストラクタ**で表現します。

```fsharp
type String50 = private String50 of string

module String50 =
    let value (String50 str) = str
    let create fieldName str =
        ConstrainedType.createString fieldName String50 50 str
```

Java では `record` + 静的ファクトリにしました。生成失敗を例外ではなく Vavr の `Either` で返すのがポイントです。

```java
public record String50(String value) {

    private static final int MAX_LENGTH = 50;

    public static Either<String, String50> create(String fieldName, String value) {
        return ConstrainedType.createString(fieldName, String50::new, MAX_LENGTH, value);
    }

    public static Either<String, Option<String50>> createOption(String fieldName, String value) {
        return ConstrainedType.createStringOption(fieldName, String50::new, MAX_LENGTH, value);
    }
}
```

F# の `ctor` パラメータ(`String50` 自身をコンストラクタ関数として渡す)は、Java では **`String50::new` というメソッド参照**でそのまま表現できました。共通のバリデーションは `ConstrainedType` というヘルパに集約しています。

```java
public static <T> Either<String, T> createString(
        String fieldName, Function<String, T> ctor, int maxLength, String value) {
    if (value == null || value.isEmpty()) {
        return Either.left(fieldName + " must not be null or empty");
    }
    if (value.length() > maxLength) {
        return Either.left(fieldName + " must not be more than " + maxLength + " chars");
    }
    return Either.right(ctor.apply(value));
}
```

> ⚠️ `record` の正規コンストラクタは常に public なので、F# のように「コンストラクタを完全に隠す」ことはできません。今回は「不変性 + 静的ファクトリでの検証」という性質だけを採り、コンストラクタ直呼びは規約で抑止する方針にしました。

## 2. 判別共用体は sealed interface + record

`ProductCode` は「Widget か Gizmo のどちらか」という直和型です。

```fsharp
type ProductCode =
    | Widget of WidgetCode
    | Gizmo of GizmoCode
```

Java では `sealed interface` で表現し、各ケースを `record` が実装します。

```java
public sealed interface ProductCode permits WidgetCode, GizmoCode {
    String value();
    // create(...) は "W" 始まりなら WidgetCode、"G" 始まりなら GizmoCode に振り分け
}

public record WidgetCode(String value) implements ProductCode { /* W + 4桁 */ }
public record GizmoCode(String value) implements ProductCode { /* G + 3桁 */ }
```

うれしいのは、**Java 21 以降のパターンマッチング switch が網羅性をコンパイル時に保証**してくれること。F# の `match` と同等の安全性が得られ、`default` 句は不要です。

```java
String label = switch (productCode) {
    case WidgetCode w -> "widget:" + w.value();
    case GizmoCode g  -> "gizmo:" + g.value();
};
```

同じパターンで `OrderQuantity`(`UnitQuantity | KilogramQuantity`)、`PlaceOrderEvent`、`PlaceOrderError` なども表現しました。

## 3. ワークフローの合成: `result { }` を flatMap で

F# の最大の武器のひとつがコンピュテーション式です。`let!` で `Result` を取り出し、どこかで失敗したら全体が短絡する、というフローを直線的に書けます。

```fsharp
let toCustomerInfo (unvalidatedCustomerInfo: UnvalidatedCustomerInfo) =
    result {
        let! firstName = unvalidatedCustomerInfo.FirstName |> String50.create "FirstName" |> Result.mapError ValidationError
        let! lastName  = unvalidatedCustomerInfo.LastName  |> String50.create "LastName"  |> Result.mapError ValidationError
        let! email     = unvalidatedCustomerInfo.EmailAddress |> EmailAddress.create "EmailAddress" |> Result.mapError ValidationError
        return { Name = {FirstName=firstName; LastName=lastName}; EmailAddress = email }
    }
```

Java にコンピュテーション式はないので、`Either.flatMap` のネストで表現しました。冗長にはなりますが、意味は 1:1 対応します。

```java
private static Either<ValidationError, CustomerInfo> toCustomerInfo(UnvalidatedCustomerInfo c) {
    return String50.create("FirstName", c.firstName()).mapLeft(ValidationError::new).flatMap(firstName ->
            String50.create("LastName", c.lastName()).mapLeft(ValidationError::new).flatMap(lastName ->
            EmailAddress.create("EmailAddress", c.emailAddress()).mapLeft(ValidationError::new).map(email ->
                    new CustomerInfo(new PersonalName(firstName, lastName), email))));
}
```

ワークフロー全体(`PlaceOrderService`)も同じ要領です。F# の `asyncResult { }` が:

```fsharp
asyncResult {
    let! validatedOrder = validateOrder ... |> AsyncResult.mapError PlaceOrderError.Validation
    let! pricedOrder    = priceOrder ...    |> AsyncResult.mapError PlaceOrderError.Pricing
    let acknowledgement = acknowledgeOrder ...
    return createEvents pricedOrder acknowledgement
}
```

Java ではこうなりました。

```java
public Either<PlaceOrderError, List<PlaceOrderEvent>> place(UnvalidatedOrder order) {
    return Either.<PlaceOrderError, ValidatedOrder>narrow(validateOrder.validate(order))
            .flatMap(validated -> Either.<PlaceOrderError, PricedOrder>narrow(priceOrder.price(validated))
                    .map(priced -> createEvents.create(priced, acknowledgeOrder.acknowledge(priced))));
}
```

確認メール送信は「失敗しても処理を止めない」ベストエフォートなので、`Either` の連鎖に乗せず `Option<OrderAcknowledgmentSent>` として独立に扱っています。この設計判断も F# 版と同じです。

## 4. 依存性の注入

F# では各ステップが依存(製品コードの存在確認など)を**関数の引数**として受け取り、最後にまとめて部分適用します。Java ではこれをコンストラクタ注入に置き換え、組み立てを Spring の `@Configuration` に集約しました。

```java
@Configuration
public class PlaceOrderConfiguration {
    @Bean
    public ValidateOrder validateOrder(CheckProductCodeExists check, CheckAddressExists addr) {
        return new ValidateOrder(check, addr);
    }
    // ... PriceOrder, AcknowledgeOrder, CreateEvents, PlaceOrderUseCase
}
```

出力ポート(`CheckProductCodeExists` など)は `@FunctionalInterface` として定義し、ダミー実装を `adapter/out` に置いています。**ドメイン層・アプリケーション層には Spring のアノテーションを一切付けず**、配線はすべて `config` パッケージに閉じ込めました。

## 5. DTO とシリアライゼーションの落とし穴

ここが一番 Java(というか Spring Boot 4)固有の苦労でした。

### Jackson 3 と vavr-jackson は別物

Spring Boot 4 は **Jackson 3**(`tools.jackson` パッケージ)を使います。一方 `vavr-jackson`(Vavr 型を JSON 化するモジュール)は **Jackson 2**(`com.fasterxml.jackson`)用です。両者はパッケージが別なので、`vavr-jackson` の `VavrModule` は Spring の Jackson 3 マッパーでは効きません。

そこで **DTO は素の Java 型(`String`、`BigDecimal`、`java.util.List`)だけで定義**し、Vavr 型を JSON に直接さらさない方針にしました。「DTO はプリミティブでシリアライズ可能な型で定義する」という書籍の意図とも合致し、結果として `vavr-jackson` 自体が不要になりました。

### 多態イベントの `type` 判別子が消える問題

`PlaceOrderEvent` の発信用 DTO は、Jackson の `@JsonTypeInfo` で多態にしました。

```java
@JsonTypeInfo(use = JsonTypeInfo.Id.NAME, include = JsonTypeInfo.As.PROPERTY, property = "type")
public sealed interface PlaceOrderEventDto
        permits OrderPlacedDto, BillableOrderPlacedDto, OrderAcknowledgmentSentDto {

    static PlaceOrderEventDto fromDomain(PlaceOrderEvent event) {
        return switch (event) {
            case OrderPlaced e             -> OrderPlacedDto.fromDomain(e);
            case BillableOrderPlaced e     -> BillableOrderPlacedDto.fromDomain(e);
            case OrderAcknowledgmentSent e -> OrderAcknowledgmentSentDto.fromDomain(e);
        };
    }
}
```

ところが、`List<PlaceOrderEventDto>` を `Object` として渡してシリアライズすると、**ジェネリクスが消去されて各要素が具象クラスとして直接シリアライズされ、`type` フィールドが出力されません**。

対策として、コントローラの成功レスポンスは **`PlaceOrderEventDto[]`(配列)** で返すことにしました。配列はコンポーネント型が reify されている(消去されない)ため、`@JsonTypeInfo` が確実に効きます。書籍の Api.fs にも `List.toArray // arrays are json friendly` というコメントがあり、奇しくも同じ結論でした。

```java
@PostMapping
public ResponseEntity<Object> place(@RequestBody OrderFormDto orderForm) {
    return placeOrder.place(orderForm.toUnvalidatedOrder()).<ResponseEntity<Object>>fold(
            error  -> ResponseEntity.status(httpStatus(error)).body(PlaceOrderErrorDto.fromDomain(error)),
            events -> ResponseEntity.ok(events.map(PlaceOrderEventDto::fromDomain)
                                            .toJavaArray(PlaceOrderEventDto[]::new)));
}
```

## ハマったところ・小ネタ

移植中に踏んだ落とし穴をいくつか。

- **`Either` の型は不変 (invariant)**: `Either<ValidationError, ...>` は `Either<PlaceOrderError, ...>` に自動アップキャストできません。Vavr の `Either.narrow(...)` で安全に広げます(`ValidationError` が `PlaceOrderError` を実装しているので安全)。`Result.map Widget` のような F# の型持ち上げに相当します。
- **`BigDecimal` の比較は `compareTo`**: `equals` はスケール(精度)まで区別してしまうため、範囲チェックや等価判定は必ず `compareTo` を使います。
- **Spring Boot 4 で `@AutoConfigureMockMvc` が標準 starter から消えた**: 別モジュール `spring-boot-webmvc-test` に移動しています。依存追加を避け、`spring-test` だけで動く `MockMvcBuilders.webAppContextSetup(wac)` 方式で E2E テストを書きました。
- **日本語のテストメソッド名は数字始めにできない**: `void 50文字ちょうど…()` はコンパイルエラー。Java の識別子規則(数字始まり禁止)に引っかかります。`void ちょうど50文字…()` のように回避しました。

## 動かしてみる

ダミーの出力アダプタ(製品価格は一律 1、住所確認・通知送信は常に成功)を繋いで、実際に REST API として動きます。

```bash
$ ./mvnw spring-boot:run
$ curl -X POST http://localhost:8080/orders -H "Content-Type: application/json" -d '{
    "orderId": "ORD-100",
    "customerInfo": { "firstName": "Taro", "lastName": "Yamada", "emailAddress": "taro@example.com" },
    "shippingAddress": { "addressLine1": "1-2-3", "city": "Tokyo", "zipCode": "10001" },
    "billingAddress": { "addressLine1": "4-5-6", "city": "Osaka", "zipCode": "20002" },
    "lines": [ { "orderLineId": "L1", "productCode": "W1234", "quantity": 5 } ]
  }'
```

```json
[
  { "type": "OrderAcknowledgmentSent", "orderId": "ORD-100", "emailAddress": "taro@example.com" },
  { "type": "OrderPlaced", "orderId": "ORD-100", "amountToBill": 5, "lines": [ ... ] },
  { "type": "BillableOrderPlaced", "orderId": "ORD-100", "amountToBill": 5 }
]
```

商品コードの形式が不正なら、バリデーションエラーが `400` で返ります。

```json
{ "code": "ValidationError", "message": "ProductCode: Format not recognized 'X9999'" }
```

## まとめ

『関数型ドメインモデリング』の OrderTaking を Java 25 + Spring Boot 4 + Vavr に移植し、HTTP で動くところまで作りました。

やってみて感じたのは:

- **`sealed interface` + `record` + パターンマッチング switch** は、F# の直和型と `match` をほぼ過不足なく置き換えられる。網羅性チェックも効く。
- **Vavr の `Either` / `Option`** で、例外に頼らない関数型のエラーハンドリングが Java でも書ける。コンピュテーション式が無いぶん `flatMap` のネストは冗長だが、意味は素直に対応する。
- 一番の苦労は言語ではなく**境界(シリアライゼーションと DI)**側。Spring Boot 4 / Jackson 3 の移行期特有のハマりどころが多かった。
- 逆に言えば、**ドメインのコアロジックは F# の「形」をかなり忠実に Java へ写せる**。型で不正な状態を排除する設計思想は、言語を越えて持ち込める。

関数型のエッセンスは特定の言語の専売特許ではなく、`record` と `sealed` を備えた現代の Java でも十分実践できる、というのが今回の手応えでした。

## 参考: AI へのプロンプト

1. Vavr を使えるようにしてください。
2. ./DomainModelingMadeFunctional/src/OrderTaking ディレクトリのファイルの構成について解説してください。
3. このプロジェクトを Java + Spring Boot で再実装したいのですが、どこの部分から再実装するのが良いですか?
4. まずはこのロードマップを docs ディレクトリに markdown 形式で書き出してください。進捗状況が分かるようにチェックボックスを付けて書いてください。
5. Phase 1 の String50 から始めて
6. EmailAddress に進んで
7.（ZipCode について）はい、お願いします。
8.（OrderId について）はい、お願いします。
9.（OrderLineId について）はい、お願いします。
10.（重複の共通化の選択肢提示に対して)2. でお願いします。（= ConstrainedType ヘルパを抽出する）
11. ここまでをコミット & push してください。
12.（SSH push 失敗を受けて)SSH agent を使えるようにする方法を教えてください。
13. ~/.bashrc に書き加えてください。
14. createLike (regex ヘルパ) も ConstrainedType に足して、EmailAddress/ZipCode も一緒にリファクタしてください。
15.（WidgetCode / GizmoCode / ProductCode について)はい、お願いします。
16. はい、続けてください。（= UnitQuantity / KilogramQuantity / OrderQuantity / Price / BillingAmount、および createInt / createDecimalの追加）
17. はい、コミット & push をお願いします。
18. Phase 2 (PersonalName / CustomerInfo / Address) に進んでください。
19.（Phase 3 について)はい、お願いします。
20. 一旦コミット & push してください。
21. Phase 4 の各項目はどのような順序で実装しますか?
22. この順序で良いです。まず、Slice 1 を実装してください。
23. はい、お願いします。（= Slice 1 のコミット & push、その後 Slice 2 へ)
24. Slice 2 をお願いします。
25. そのまま Slice 3 (AcknowledgeOrder) に進みます。
26. はい、お願いします。（= Slice 4 CreateEvents）
27. コミットしてから進みます。
28.（Slice 5 / Phase 4 完了のコミット & push について)はい、コミット & push をお願いします。
29. Phase 5 に進みます。Phase 4 の時と同様にどのように実装を進めていくのが良いか考えてください。
30. Slice 2 まで実装してまとめてコミットします。
31. Phase 6 に進んでください。
32. Phase 7 の README の整備 (起動方法、サンプル curl) のみ実施してください。残りの 2 つは実施しません。
33. 本移植について、/path/to/articles/548pro-domain-modeling-made-func-in-j.md に技術ブログとして紹介する内容を書いてください。