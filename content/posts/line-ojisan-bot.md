---
title: "おじさんLINEごっこBOTを作ってみた"
date: 2017-12-15T16:10:08+09:00
tags: ["BOT", "ボット","LINE","Dialogflow", "AWS", "Lambda", "Serverless"]
categories: ["ビトル"]
---

かなり遅れてしまいましたが、今回は[ボット (Bot) Advent Calendar 2017](https://qiita.com/advent-calendar/2017/bot)の10日目の記事を書きます。

<br>

突然ですが、みなさん「 [おじさんLINEごっこ](http://diamond.jp/articles/-/143111) 」という遊びをご存知ですか？
最近若い女子同士で流行っているこの遊びですが、今時のSNSにありそうな「おじさんの文章」を真似しながら送り合うというものです。

まず特徴としては以下が挙げられます：

- 長い
- 句読点が多い
- 無駄に褒めたがる
- 名前は基本的にちゃん(チャン)付け
- ちょくちょく自分語りを挟んでくる
- 隠しきれない下心

<br>

結構エグい感じがしますね。

ではこの情報を元にそれに近いLINEおじさんBOTを作ってみようと思います。

用意するものはこちらです：

- LINE Developersアカウント
- Dialogflowアカウント

構成はシンプルにこんな感じにしています。

{{< figure src="/images/line-ojisan-design.png" title="LINEおじさんBOT構成図" >}}

<br>

↑の流れをざっくり説明すると：

<br>

①LINEおじさんアカウントに投げたメッセージを

<br>

②Lambdaで受け取って

<br>

③そのメッセージをdialogflowに投げて

<br>

④dialogflowから返ってきた返答を

<br>

⑤そのままLINEに返す

<br>

Lambdaで実装したいので、serverless frameworkでデプロイをちゃちゃっとできるようにしておきました。

まずは構成定義用のyaml。

```yaml
# serverless.yml

service:
  name: line-ojisan

provider:
  name: aws
  runtime: nodejs6.10
  stage: dev
  region: ap-northeast-1


custom:
  region: ${self:provider.region}
  prefix: ${self:service}-${self:provider.stage}
  config: ${self:custom.prefix}-config
  webpackIncludeModules: true

functions:
  lineWebhookReceiver:
    handler: webhook.handler
    environment:
      region: ${self:custom.region}
      stage: ${self:provider.stage}
      NODE_ENV: ${self:provider.stage}
      LINE_CHANNEL_SECRET: xxxxxxxxxxxxxxxxxxxxxxxxx
      LINE_ACCESS_TOKEN: xxxxxxxxxxxxxxxxxxxxxxxxx
      DIALOGFLOW_CLIENT_ACCESS_TOKEN: xxxxxxxxxxxxxxxxxxxxxxxxx
    events:
      - http:
          method: post
          path: webhooks/line
    timeout: 60
```

そしてハンドラ内の処理。

絵文字は [emojione](https://www.npmjs.com/package/emojione)を使って送れるようにしています。

```js
// webhook.js

const utils = require('functions/utils');
const dialogflow = require('apiai');
const emojione = require('emojione');
const dfClient = dialogflow(process.env.DIALOGFLOW_CLIENT_ACCESS_TOKEN);
const line = require('@line/bot-sdk');
const Line = new line.Client({
  channelAccessToken: process.env.LINE_ACCESS_TOKEN,
  channelSecret: process.env.LINE_CHANNEL_SECRET,
});

exports.handler = (event, context, callback) => {
  try {
    let body = null;
    if (typeof event.body === "string" || Buffer.isBuffer(event.body)) {
      body = utils.validateSignature(
        event.body,
        process.env.LINE_CHANNEL_SECRET,
        event.headers['X-Line-Signature']
      )
    }

    if (body === null) {
      throw new Error('body parsing failed')
    }

    body.events.forEach((webhookData) => {
      const messageType = webhookData.type;
      const userId = webhookData.source.userId;
      const replyToken = webhookData.replyToken;
      switch (messageType) {
        case  'follow': // 友達追加時
          Line.getProfile(userId).then((profileData) => {
            const displayName = profileData.displayName;
            const replyMessage = utils.welcomeMessage(displayName);
            Line.replyMessage(replyToken, { type: 'text', text: replyMessage});
          });
          break;
        case  'unfollow': // ブロック時
          break;
        case  'message': // メッセージ送信時
          Line.getProfile(userId).then((profileData) => {
            const displayName = profileData.displayName;
            const text = webhookData.message.text;
            // dialogflowへメッセージを送信
            const request = dfClient.textRequest(
              text, { sessionId: userId }
            );
            request.on('response', function(response) {
              // agentからの応答
              const message = response.result.fulfillment.speech;
              let replyMessage = utils.convertPlaceholders(message, 'displayName', displayName);
              replyMessage = utils.convertPlaceholders(replyMessage, '¥n', "\n");
              replyMessage = emojione.shortnameToUnicode(replyMessage);
              Line.replyMessage(replyToken, { type: 'text', text: replyMessage});
            });
            request.on('error', function(error) {
              console.log(error);
            });
            request.end();
          });
          break;
      }
    });
  } catch (e) {
    throw e;
  }
  callback(null,{ statusCode:200 });
};
```

お次に定義している関数をまとめたjs。

友達追加時は特に発言が送られるわけではないので、dialogflowへは何も送信せずひとまず固定のメッセージを返しています。

<br>

```js
// functions/utils.js

const emojione = require('emojione');
const JSONParseError = require('@line/bot-sdk').JSONParseError;
const validateSignature = require('@line/bot-sdk').validateSignature;
const SignatureValidationFailed = require('@line/bot-sdk').SignatureValidationFailed;

// X-Line-Signatureリクエストヘッダーに含まれる署名を検証
module.exports.validateSignature = function (body, secret, signature) {
  if (!validateSignature(body, secret, signature)) {
    throw new SignatureValidationFailed("signature validation failed", signature)
  }
  const strBody = Buffer.isBuffer(body) ? body.toString() : body;
  try {
    return JSON.parse(strBody);
  } catch (err) {
    throw new JSONParseError(err.message, strBody);
  }
};

// 返答メッセージに入っているプレースホルダを任意の文字列に変換
module.exports.convertPlaceholders = function (message, placeholderString, replaceValue) {
  const placeholder = `<${placeholderString}>`;
  const convertedMessage = message.replace(new RegExp(placeholder, 'g'), replaceValue);
  return convertedMessage;
};

// 友達追加時メッセージ
module.exports.welcomeMessage = function (displayName) {
  return emojione.shortnameToUnicode(
    `${displayName}ちゃん、はじめまして:exclamation::smiley:` + "\n" +
    `おじさんは${displayName}ちゃんとお知り合いになれて、とてもうれしいです。(^_^)` + "\n\n" +
    `最近おじさんね、近くでお寿司屋さん見つけてさ:sushi:ピチピチのお魚が、とても美味しかったです:fish:` + "\n" +
    `でも、${displayName}ちゃんの方がピチピチだと思うなぁ〜:kissing_heart:` + "\n" +
    `今度一緒にどうカナ？お寿司と一緒に${displayName}ちゃんのことも食べちゃいたいなぁ〜:heart_eyes:ナンチャッテσ(^_^;)` + "\n" +
    `今後ともヨロシクね:grinning:`
  )
};


```

ではdialogflow側のagentを作成しましょう。

まずはagentから。

とりあえず名前をline-ojisanにして、言語は日本語に設定しておきます。

{{< figure src="/images/dialog-flow-1.png" title="" >}}

次にentitiesを登録します。

entitiesは会話の流れでdialogflowが反応するキーワードを登録します。

まずは挨拶関連の単語から。

{{< figure src="/images/dialog-flow-2.png" title="" >}}

次にお天気周りの単語の登録。

{{< figure src="/images/dialog-flow-3.png" title="" >}}

では次にintentsを登録します。intentsは会話・文章自体の登録を行います。

まずはお天気周りのintentsの登録から。会話の中で先ほど登録したentityが含まれているこういう風に黄色くハイライトされます。

{{< figure src="/images/dialog-flow-5.png" title="" >}}

次に会話に対する反応を登録します。

ここでは拾ったentityをそのまま変数として応答の中に埋め込む事ができます。

この場合は `$temperature` がそれにあたります。

また、\<displayName\>と\<¥n\>は、それぞれLINEのMessaging APIで取得したプロフィール名と改行に変換します。


{{< figure src="/images/dialog-flow-7.png" title="" >}}

次は挨拶関連のintentsの登録をします。

{{< figure src="/images/dialog-flow-8.png" title="" >}}
{{< figure src="/images/dialog-flow-9.png" title="" >}}

もし会話の中で、該当するentityが見つからなかった場合にデフォルトで返すメッセージを設定できます。
Default Fallback Intentで設定が可能となっています。

{{< figure src="/images/dialog-flow-4.png" title="" >}}

これで必要最低限のデータは登録できたので、次にデプロイをします

```sh
$ npm install
$ sls deploy
```

デプロイされたAPI GatewayのエンドポイントをLINE Developersぼwebhook URLに登録しておきましょう。

では実際におじさんと会話してみましょう

まずはアカウント友達追加。

{{< figure src="/images/line-ojisan-friend-add-message.png" title="" >}}

もう初っ端からキツイですね。

では最初に挨拶してみましょうか。

{{< figure src="/images/line-ojisan-chat1.jpg" title="" >}}

{{< figure src="/images/line-ojisan-chat2.jpg" title="" >}}

おじさん成分高めですね〜

では次にお天気の話をしてみましょう

{{< figure src="/images/line-ojisan-chat7.jpg" title="" >}}

{{< figure src="/images/line-ojisan-chat8.jpg" title="" >}}

急な自分語りを挟んできて素敵ですね。

では登録されていないキーワードを送信するとどうなるか。

{{< figure src="/images/line-ojisan-chat3.jpg" title="" >}}

「ゆたんぽおじさん」・・・

{{< figure src="/images/line-ojisan-chat5.jpg" title="" >}}

{{< figure src="/images/line-ojisan-chat4.jpg" title="" >}}

<br>

## 長ぇ・・・

自分で作っておいてなんですが、結構キツイですねこれ・・・

<br>

---

<br>

## まとめ

DialogflowとLINEの組み合わせはこちらの想像力次第色々と面白いBOTが作れますね。

ちなみに今回つくったLINEおじさんBOTのLINEアカウントは公開してますので、ぜひぜひ登録して
おじさんとの素敵な(?)会話を楽しんでください！！

QRコードはコチラ↓↓

{{< figure src="/images/line-ojisan-qr-code.png" title="" >}}

今後は会話のパターンも増やしていく予定です！

ではでは！！

<br><br><br>