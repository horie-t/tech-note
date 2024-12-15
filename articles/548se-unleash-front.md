---
title: "Unleash + Next.jsでフィーチャーフラグを実装する"
emoji: "🙆"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["unleash", "featureflag", "nextjs", "react", "typescript"]
published: true
---

[以前の記事](./548se-unleash)でバックエンドのUnleashを使ってフィーチャーフラグを実装しました。今回はフロントエンドのNext.jsでUnleashを使ってフィーチャーフラグを実装する方法を紹介します。

## Next.jsのプロジェクトを作成

まずはNext.jsのプロジェクトを作成します。

```bash
$ npx create-next-app@latest --ts
Need to install the following packages:
create-next-app@15.1.0
Ok to proceed? (y) y

✔ What is your project named? … unleash-client-demo
✔ Would you like to use ESLint? … Yes
✔ Would you like to use Tailwind CSS? … Yes
✔ Would you like your code inside a `src/` directory? … Yes
✔ Would you like to use App Router? (recommended) … Yes
✔ Would you like to use Turbopack for `next dev`? … Yes
✔ Would you like to customize the import alias (`@/*` by default)? … No
```

ブラウザで `http://localhost:3000` にアクセスします。以下のデフォルト画面が表示されます。

![](/images/unleash-client-init.png)

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

`Feature flag name`、`Description`にフラグの名前と説明を入力し `Create feature flag` ボタンをクリックします。本記事ではタスク管理サービスで締め切り日時の管理機能の追加を例に解説するので名前を `support-due-date-time-frontend` としました。
![](/images/unleash-client-create-flat-name.png)

#### トークンの発行

`Connect SDK` ボタンをクリックします。
![](/images/unleash-default-project-connect-sdk.png)

`Client side SDKs`の`React`を選択します。
![](/images/unleash-client-connect-sdk-select-sdk.png)

`Enviroment` を選択して、`Next`ボタンをクリックします。今回はローカルPCで動かすだけなので`development`を選択しています。
![](/images/unleash-client-connect-sdk-environment.png)

接続のための設定が表示され、クライアントからの接続待ち状態になります。
![](/images/unleash-client-connect-sdk-wait-connect.png)

## フィーチャーフラグを使ったWeb画面の実装

タスク管理アプリのAPIサーバで、フィーチャーフラグが有効になるとレスポンスに締め切り日時が表示されるケースを考えてみます。

#### 締め切り機能追加前の状態を実装

まずは締め切り機能が追加されていない状態を実装します。

Todoの型を定義します。

```typescript:src/app/Todo.ts
export type Todo = {
  id: string;
  title: string;
  description: string;
};
```

Todoのコンポーネントを作成します。

```typescript:src/app/TodoComponent.tsx
import { Todo } from "./Todo";

export const TodoComponent = (todo: Todo) => {
    return (
        <li key={todo.id}>
            <h2 className='text-lg font-bold'>{todo.title}</h2>
            <p>{todo.description}</p>
        </li>
    )
};
```

page.tsxを以下のように修正します。

```typescript:src/app/page.tsx
'use client'
import { Todo } from "./Todo";

const unleashConfig = {
    url: 'http://localhost:4242/api/frontend/',
    clientKey: 'default:development.unleash-insecure-frontend-api-token',
    appName: 'unleash-onboarding-react'
}

const todos: Todo[] = [
    {
        id: '1',
        title: 'Learn Next.js',
        description: 'Learn Next.js and build something great',
        dueDate: '2025-01-01'
    },
    {
        id: '2',
        title: 'Build something great',
        description: 'Learn Next.js and build something great',
        dueDate: '2025-01-02'
    }
]

export default function Home() {
  return (
      <FlagProvider config={unleashConfig}>
          <div className='container mx-auto p-4'>
              <h1 className='text-3xl font-bold mb-4'>Todo</h1>
              <ul className='list-disc'>
                  {todos.map(todo => (
                      <TodoComponent key={todo.id} {...todo} />
                  ))}
              </ul>
          </div>
      </FlagProvider>
  );
}
```

ブラウザには以下のように表示されます。

![](/images/unleash-client-todo.png)

#### フィーチャーフラグを使った締め切り機能の追加

Todoの型を`dueDate`を追加します。

```typescript:src/app/Todo.ts
export type Todo = {
  id: string;
  title: string;
  description: string;
  dueDate?: string;
};
```

Todoのコンポーネントを修正します。

```typescript:src/app/TodoComponent.tsx
import { useEffect } from "react";
import { useFlag } from "@unleash/proxy-client-react";
import { Todo } from "./Todo";

export const TodoComponent = (todo: Todo) => {
    const hasVisited =
        typeof window !== "undefined" && !!localStorage.getItem("hasVisited");

    useEffect(() => {
        if (!hasVisited) localStorage.setItem("hasVisited", "true");
    }, []);

    const enabled = useFlag('support-due-date-time-frontend');

    return (
        <li key={todo.id}>
            <h2 className='text-lg font-bold'>{todo.title}</h2>
            <p>{todo.description}</p>
            {enabled ? <p>{todo.dueDate}</p> : null}
        </li>
    )
};
```

page.tsxを以下のように修正します。

```typescript:src/app/page.tsx
'use client'
import dynamic from "next/dynamic";
import { FlagProvider } from "@unleash/proxy-client-react";
import { Todo } from "./Todo";

const unleashConfig = {
    url: 'http://localhost:4242/api/frontend/',
    clientKey: 'default:development.unleash-insecure-frontend-api-token',
    appName: 'unleash-onboarding-react'
}

const TodoComponent = dynamic(() => import('./TodoComponent').then((mod) => mod.TodoComponent), {
    ssr: false,
});

const todos: Todo[] = [
    {
        id: '1',
        title: 'Learn Next.js',
        description: 'Learn Next.js and build something great',
        dueDate: '2025-01-01'
    },
    {
        id: '2',
        title: 'Build something great',
        description: 'Learn Next.js and build something great',
        dueDate: '2025-01-02'
    }
]

export default function Home() {
  return (
      <FlagProvider config={unleashConfig}>
          <div className='container mx-auto p-4'>
              <h1 className='text-3xl font-bold mb-4'>Todo</h1>
              <ul className='list-disc'>
                  {todos.map(todo => (
                      <TodoComponent key={todo.id} {...todo} />
                  ))}
              </ul>
          </div>
      </FlagProvider>
  );
}
```
#### 締め切りフィーチャーフラグの有効化

Unleashの管理画面で `support-due-date-time-frontend` のフィーチャーフラグを `develop`環境で有効化します。

![](/images/unleash-client-flag-enable.png)

ブラウザには以下のように表示されます。

![](/images/unleash-client-todo-due-date.png)