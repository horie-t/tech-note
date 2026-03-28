---
title: "Javaのrecordクラスの一部のフィールドを変更したオブジェクトを作成する方法"
emoji: "🐥"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Java", "record", "lombok"]
published: true
---

Javaのrecordクラスは、イミュータブルなデータクラスを簡潔に定義するための機能ですが、全てのフィールドが自動的にfinalになります。そのため、一部のフィールドを変更可能にすることはできません。しかし、Lombokを使用することで、recordクラスの一部のフィールドを変更したオブジェクトを作成することができます。

```java
import lombok.With;

@With
public record User(String name, int age, String email) {}
```

この例では、`User` recordクラスに`@With`アノテーションを付けることで、各フィールドに対して変更可能なメソッドが自動生成されます。例えば、`name`フィールドを変更したい場合は、以下のようにします。

```java
User user1 = new User("Alice", 30, "alice@example.com");
User user2 = user1.withName("Bob").withEmail("bob@example.com");
System.out.println(user1); // User[name=Alice, age=30, email=alice@example.com]
System.out.println(user2); // User[name=Bob, age=30, email=bob@example.com]
```
