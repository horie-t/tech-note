---
title: "ChatGPTに機械翻訳Webアプリケーションを作成させた"
emoji: "📌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["ai", "chatgpt"]
published: true
---

ChatGPTに、質問・指示を出しながら、機械翻訳のWebアプリケーションを作成させました。翻訳エンジンにはGoogle Cloud Translationを使用し、バックエンドはSpring Web、フロントエンドはReactを使いました。

## バックエンドの構築


![](https://github.com/horie-t/tech-note/blob/master/images/chatgpt-mt-spring-prj.png?raw=true)


プロジェクトの雛形をダウンロードして、zipを展開して、ディレクトリ名を変更する。

```bash
unzip ~/ダウンロード/simple_translator.zip
mv simple_translator backend
```

![](https://github.com/horie-t/tech-note/blob/master/images/chatgpt-mt-openapi.png?raw=true)

YAMLファイルを保存するディレクトリを作成して、yamlファイルを保存する。

```bash
mkdir -p backend/docs/api
```

![](https://github.com/horie-t/tech-note/blob/master/images/chatgpt-mt-backend-1.png?raw=true)
![](https://github.com/horie-t/tech-note/blob/master/images/chatgpt-mt-backend-2.png?raw=true)
![](https://github.com/horie-t/tech-note/blob/master/images/chatgpt-mt-backend-3.png?raw=true)
![](https://github.com/horie-t/tech-note/blob/master/images/chatgpt-mt-backend-4.png?raw=true)

![](https://github.com/horie-t/tech-note/blob/master/images/chatgpt-mt-backend-boot.png?raw=true)

![](https://github.com/horie-t/tech-note/blob/master/images/chatgpt-mt-backend-test.png?raw=true)

テストしてみると、ちゃんとレスポンスが返ってくる。

```bash
 curl -X POST -H "Content-Type: application/json" -d '{"text": "こんにちは、世界！", "source_language": "ja", "target_language": "en"}' http://localhost:8080/v1/translate
{"translatedText":"Translated Text: こんにちは、世界！, Source Language: null, Target Language: null"}
```

![](https://github.com/horie-t/tech-note/blob/master/images/chatgpt-mt-backend-gct-1.png?raw=true)
![](https://github.com/horie-t/tech-note/blob/master/images/chatgpt-mt-backend-gct-2.png?raw=true)
![](https://github.com/horie-t/tech-note/blob/master/images/chatgpt-mt-backend-gct-3.png?raw=true)
![](https://github.com/horie-t/tech-note/blob/master/images/chatgpt-mt-backend-gct-4.png?raw=true)
![](https://github.com/horie-t/tech-note/blob/master/images/chatgpt-mt-backend-gct-5.png?raw=true)
![](https://github.com/horie-t/tech-note/blob/master/images/chatgpt-mt-backend-gct-6.png?raw=true)
![](https://github.com/horie-t/tech-note/blob/master/images/chatgpt-mt-backend-gct-7.png?raw=true)
![](https://github.com/horie-t/tech-note/blob/master/images/chatgpt-mt-backend-gct-8.png?raw=true)
![](https://github.com/horie-t/tech-note/blob/master/images/chatgpt-mt-backend-gct-9.png?raw=true)

環境変数を設定して実行する。your_api_keyはGCPで設定した値を使う。

```bash
export TRANSLATION_API_KEY=your_api_key
./gradlew bootRun
```

curlでテストする。

```bash
$ curl -X POST -H "Content-Type: application/json" -d '{"text": "こんにちは、世界！", "source_language": "ja", "target_language": "en"}' http://localhost:8080/v1/translate
{"translatedText":"Hello World!"}
```

以上で、バックエンドが完成。

## フロントエンドの構築

![](https://github.com/horie-t/tech-note/blob/master/images/chatgpt-mt-react-prj.png?raw=true)

無事、アプリが起動。

![](https://github.com/horie-t/tech-note/blob/master/images/chatgpt-mt-react-boot.png?raw=true)

![](https://github.com/horie-t/tech-note/blob/master/images/chatgpt-mt-front-1.png?raw=true)
![](https://github.com/horie-t/tech-note/blob/master/images/chatgpt-mt-front-2.png?raw=true)
![](https://github.com/horie-t/tech-note/blob/master/images/chatgpt-mt-front-3.png?raw=true)
![](https://github.com/horie-t/tech-note/blob/master/images/chatgpt-mt-front-4.png?raw=true)

画面が表示される。タイトルに比べて、画面の下端の入力画面が小さい…

![](https://github.com/horie-t/tech-note/blob/master/images/chatgpt-mt-tranlate-page.png?raw=true)

翻訳しようとしたら、ブラウザのコンソールログに以下のエラーログが出力され翻訳できない。

![](https://github.com/horie-t/tech-note/blob/master/images/chatgpt-mt-tranlate-error.png?raw=true)

![](https://github.com/horie-t/tech-note/blob/master/images/chatgpt-mt-tranlate-debug-1.png?raw=true)
![](https://github.com/horie-t/tech-note/blob/master/images/chatgpt-mt-tranlate-debug-2.png?raw=true)

完成しました。翻訳結果は下端にちっちゃく表示されている。

![](https://github.com/horie-t/tech-note/blob/master/images/chatgpt-mt-tranlate-page-fixed.png?raw=true)

## 作成結果

作成したコードは[GitHub](https://github.com/horie-t/simple-translator-by-chatgpt)にあります。

## 最後に

どこまでChatGPTができるのかの限界を探るつもりが、まさかの完成と言うことになってしまいました。このアプリケーションではデータベースを使ってないですが、単純なToDo管理のCRUDアプリケーションぐらいなら作成できそうな気がします。

JSON色付け係や、JSON詰替え係のようなエンジニアは淘汰されてしまうのか。それとも、ChatGPT指示係に転職でしょうか。

時間があれば、ChatGPTに指示してOpenNMT、Fairseq の翻訳エンジンを構築させてみたいと思います。




