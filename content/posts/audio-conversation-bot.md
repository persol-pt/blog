---
title: "Dialogflow + VoiceTextで音声会話Botをさくっと作ろう"
date: 2017-12-14T00:19:21+09:00
draft: true
tags: ["Bot", "音声会話", "自然言語処理", "機械学習", "チャットボット"]
categories: ["藤井 大祐(daitasu)"]
---

## dialogflow + VoiceTextでさくっと音声会話botを作ろう<br>

どうも、daitasuです。<br>
今回は新卒ながら自分もAdvent Calenderの記事投稿に加わらせて頂きました。<br>
<br>
この記事は、新卒専用 Advent Calendar 2017  19日目の記事になります。<br>

本記事では、dialogflowを用いて簡単にチャットボットを作り、そこにVoiceTextを足し合わせた音声会話Botの作り方をお伝えしたいと思います。<br>

<br>

### 今回作るものの概要
---
<br>
### 完成品
<br>
何はともあれ実物を見てみましょう。今回作成した音声会話Botはこちらです。↓↓↓<br>

<br>
<br>

youtube

<br>
<br>

① この動画のような感じでBotとの音声を文字列に変換しBotが認識<br>
② AIが反応し対応する言葉を返答<br>
③ テキストと同時に、音声にも変換して返事を返す<br>

<br>

今回は相手役としてカワウソちゃんとおしゃべりできるようにしようと思います！<br>

<br>

### 仕組み
<br>
流れ図としては以下のようになります。<br>
<br>
{{< figure src="/images/audio-conversation-bot/ACB1.png" title="音声会話Botの概要" >}}<br>
<br>

### 使った技術
今回使った技術は以下の通りです。<br>

- **Node.js** ... サーバサイドのJavascriptランタイム環境。フレームワークにはexpressを用い、テンプレートエンジンとしてejsを使用しました。<br><br>
- **VoiceText(Node.js SDK)** ... HOYA株式会社さんの音声合成ソフトウェア。流暢な発声と感情表現できるのが特徴です。<br><br>
- **Dialogflow** ... 自然言語対話のAIを簡単に作れるプラットフォームです。さくっとBotを作るのに向いています。<br><br>
<br>
<br>

### 作成の流れ
---
<br>
では実際に作成していきましょう。<br>
<br>
### **Dialogflow**
まずはDialogflowで対話するAIBotを作っていきます。<br>
<br>
[Dialogflow](https://console.dialogflow.com/)にログインし、「Create new agent」で新しいエージェントを作成します。<br>
<br>

- **Entity** <br>

<br>
Entityというのは、文字の表記揺れであったり、似たような言葉をひとくくりにまとめて認識してくれる機能です。<br>
<br>

{{< figure src="/images/audio-conversation-bot/dialogflow1.png" title="Entityの作成画面" >}}<br>

<br>
このようにあるくくりで似たような言葉をグループ化していきます。<br>
<br>
{{< figure src="/images/audio-conversation-bot/dialogflow2.png" title="Entity一覧" >}}<br>

とりあえずいくつか作ってみましょう。<br>
<br>

- **Intents**<br>

<br>
さて、次は会話のメインとなるIntentsの作成です。<br>
<br>
Intentsでは、ユーザからの会話とそのレスポンス方法を記載することができます。<br>

{{< figure src="/images/audio-conversation-bot/dialogflow3.png" title="Intent作成画面" >}}<br>

Intentの項目は「User says」「Events」「Action」「Response」の4つです。<br>
Webhookで他の自作アプリと連携したりすることもできますが、<br>
今回はシンプルにテキストの応答をするので、「user says」「Response」のみを使います。<br>

<br>
「User says」には、AIが聞かれる質問を入力します。<br>
この際、先ほど作成した「Entity」の言葉を含む場合、自動で認識され、<br>
グループ化した他の言葉にも対応されるようになるため、より精度向上に繋がります。<br>
<br>
また、質問がわからなかった場合は「Default Fallback Intent」という項目の中のテキストで返されます。<br>
<br>
一通り言葉の学習を終えたら、テストをしてみましょう。<br>
右側のコンソールで応答の確認をすることができます。<br>
<br>
結果はJSON形式で返ってきます。Entityにセットしたものがあれば、 `parameters` に含まれます。<br><br>

```
{
  "id": "7f0edf23-5de4-4221-b964-f1cd26c6dd6a",
  "timestamp": "2017-12-14T16:53:36.671Z",
  "lang": "ja",
  "result": {
    "source": "agent",
    "resolvedQuery": "カワウソちゃん大好き！",
    "action": "",
    "actionIncomplete": false,
    "parameters": {
      "love": "大好き"
    },
    "contexts": [],
    "metadata": {
      "intentId": "9fd91bf3-e7ef-4417-a136-50f2dd3bdd70",
      "webhookUsed": "false",
      "webhookForSlotFillingUsed": "false",
      "intentName": "大好きカワウソ"
    },
    "fulfillment": {
      "speech": "きゅ〜。ぼくも愛してるよ〜！今日もいっぱいおしゃべりしてね！",
      "messages": [
        {
          "type": 0,
          "speech": "大好き〜。いっぱい喋ってくれると嬉しいなぁ"
        }
      ]
    },
    "score": 0.8799999952316284
  },
  "status": {
    "code": 200,
    "errorType": "success",
    "webhookTimedOut": false
  },
  "sessionId": "125658e8-405e-4edd-ba18-2d814aef2c01"
}
```

以上がDialogflowの基本的な動きです。<br>
<br>
<br>

### チャット画面の作成<br>
<br>
次に、音声会話を成立するための画面を作っていこうと思います。<br>
画面の基礎にはDialogflowの [HTML5 SDK](https://gist.github.com/Gugic/cfc008599fa9a82eeba4127648009132) を利用します。<br>
Dialogflowには他にも多くのSDKが準備してあるようなので、慣れてきたら自分の好きな言語で作ってみてください。<br>
<br>
URL先のhtmlファイルをごっそりコピペか `git clone` してきましょう。<br>
アクセストークンは順次自分のも作ったエージェントに書き換えて下さい。<br>
以下のような画面になるかと思います。<br>
<br>
image<br>
<br>
では応答を試してみましょう。<br>
<br>
image<br>
<br>
問題無さそうですね。<br>
<br>
しかし、このままだとカワウソちゃんには少し殺風景なので、<br>
画面の改造をしていきます。<br>
<br>
CSS部分をこのように修正します。<br>
<br>
チャット風に追加されていくようにしたいので、Javascriptも少し変更します。<br>
<br>
また、今回はNode.jsを使いたいので、HTMLの代わりにejsを使うよう、修正していきます。<br>
<br>
Node.js側の動きはこのようになっています。<br>
<br>
app.js, index.js<br>
<br>
次に、ejsを上記までに作ったHTMLをコピペして持って行きましょう<br>
<br>
さて、改めて見てみましょう。<br>
<br>
image<br>
<br>
いいですね、チャットっぽさが出てきました。<br>
<br>
<br>
<br>
さて、これで終わり！<br>
<br>
と行きたいところですが、もうひとアクセント加えて音声チャットにして行きましょう。<br>
<br>
ここで使うのは、VoiceTextというライブラリです。<br>
こちらでDialogflowから戻ってきた文字列を合成音声のwavファイルに変え、再生するようにします。<br>
<br>
今回はNode.jsのSDKを使ってやって行きましょう。<br>
<br>
このSDKを使うと、サーバ側にwavファイルが生成されます。<br>
このwavファイルが更新されるたびに自動で呼び出されるようにしていきます。<br>
<br>
まずは `npm install --save voicetext` でVoiceTextをインストールし、<br>
Nodeで音声ファイル用のURLを準備し、利用します。<br>
<br>
node<br>
<br>
次に、先ほどのejsファイルから呼び出します。<br>
ajaxでシンプルに呼び出しましょう。``メソッドに追加していきます。<br>
<br>
どうでしょうか、これで文字列と同時に音声が返ってくるようになったのではないかと思います。<br>
<br>
<br>
## まとめ
DialogflowとVoiceTextの SDKを組み合わせることで簡易に音声会話チャットを作ることができた。<br>
ただ、注意点として、httpsのURLでないとマイクが使えません。<br>
<br>
また、今回私はHeroku、 EC2これを試したのですが、<br>
<br>
① ajaxでNodeを呼ぶ<br>
② voicetextで合成音声をwavファイルとして生成<br>
③ audioファイルとしてクライアント側に送り、再生<br>
<br>
という工程がどうしても時間がかかり文字の出力から音声の再生まで、<br>
日本リージョンのEC2で1〜3秒、Herokuで4〜5秒となっていました。<br>
<br>
おそらくwavファイルの生成に時間がかかっているので、バイナリファイルが返ってきた時点でクライアント側に返せば<br>
あんまり時間はかからなそうでした。(僕にはやり方は分からなかったですが...)<br>
<br>
特段Nodeを使う必要性もないかと思いますので、誰かいい方法を見つけたら教えて下さい。<br>
<br>
ではでは。<br>
<br>
<br>