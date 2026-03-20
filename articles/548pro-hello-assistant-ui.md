---
title: "assistant-uiを使ってみた"
emoji: "😊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["生成ai", "assistantui", "llm", "react"]
published: true
---

[assistant-ui](https://zenn.dev/thorie/scraps/a500497588d820)は、LLM（大規模言語モデル）を利用したAIチャットアプリケーションを簡単に構築できるオープンソースのReactライブラリです。Thoughtworksの[Technology Radar](https://www.thoughtworks.com/radar/languages-and-frameworks/assistant-ui)で以下のように紹介されています。

> We’ve successfully built a simple chat interface with assistant-ui and have been pleased with the results. (私たちはassistant-uiでシンプルなチャットインターフェースを構築し、その成果に満足しています。)

とあり、実際にどのように構築するのか気になるところです。この記事では、assistant-uiでアプリケーションを構築して起動するまでの手順を紹介します。

## システム要件

- Node.js
- npm

## プロジェクトの作成

以下の実行例のようにコマンドを実行して、Reactプロジェクトを作成します。

```bash
$ npx assistant-ui@latest create
Need to install the following packages:
assistant-ui@0.0.67
Ok to proceed? (y) y

Creating project with template: default

Need to install the following packages:
create-next-app@16.0.7
Ok to proceed? (y) y

✔ What is your project named? … hello-assistant-ui
```

プロジェクトの作成が完了したら、以下のコマンドでプロジェクトディレクトリに移動します。

```bash
cd hello-assistant-ui
```

## 環境変数の設定

本記事では、LLMプロバイダーとしてOpenAIを利用するため、[OpenAIのAPIキー](https://platform.openai.com/api-keys)を環境変数として設定する必要があります。プロジェクトルートに`.env`ファイルを作成し、以下のようにAPIキーを設定します。

```bash
cat <<END >.env
OPENAI_API_KEY="sk-proj-jZaMDm..."
END
```

OpenAIのAPIを使用する場合はデフォルトのモデルが `gpt-5-nano` になっており、利用するにはOpenAIのアカウントが組織の認証を必要とします。認証を受けていない場合は、以下の部分を編集して認証の不要なモデルを指定します。(gpt-4.1など)

```typescript:app/api/chat/route.ts
export async function POST(req: Request) {
 const { messages }: { messages: UIMessage[] } = await req.json();

 const result = streamText({
   model: openai.responses("gpt-5-nano"),
```

最後に、以下のコマンドを実行して起動します。

```bash
npm run dev
```

ブラウザで `http://localhost:3000` にアクセスすると、以下のようなチャットUIが表示されます。

![](/images/hello-assistant-ui.png)

以上で、assistant-uiを使ったチャットアプリケーションの構築と起動が完了しました。独自のバックエンドと組み合わせる等の応用については『[Spring AI入門](https://amzn.asia/d/0eAmDt2W)』を参照してください。