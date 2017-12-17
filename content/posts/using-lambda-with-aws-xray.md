---
title: "AWS X-Rayを使ってLambda内の処理をトレースしてみる"
date: 2017-12-17T17:51:27+09:00
tags: ["AWS","Lambda","X-Ray","Node.js", "Javascript"]
categories: ["ビトル"]
---

どもどもビトルです。
今回は [AWS Lambda Advent Calendar 2017](https://qiita.com/advent-calendar/2017/aws-lambda)の１７日目の記事を書きます。

僕は今年の [Serverlessconf Tokyo](http://tokyo.serverlessconf.io/)に行ってきたのですが、その時にちょくちょくAWSのX-Rayというサービスを耳にして気になったので、それについて書きます。

<br>

## AWS X-Rayとは？

まず以下の特徴があります：

- リクエストのトレース
- 例外の収集
- プロファイリング機能
- 分散型アプリケーションの動作分析の支援

概要としては、開発者が作ったアプリケーション内のリクエストに関するデータを収集し、
それらを可視化・フィルタリングできるようにしてアプリケーション内のボトルネックの発見などを手助けします。
また、収集したリクエストのデータを元にリクエストとレスポンスだけでなく、アプリ内でコールされたDBやWeb APIなどのAWS内のリソース・サービスに関する情報も収集することができます。

<br>

## 実際に使ってみた

ではさっそく試して見ましょうか。

今回もちゃちゃっとServerless Frameworkを使ってデプロイします。まずプロジェクトを作成しましょう。

```sh
sls create -t aws-nodejs-ecma-script -p aws-xray
cd aws-xray
```

そして `serverless.yml` を以下のように編集します。

```yml
service:
  name: aws-xray

plugins:
  - serverless-webpack
  - serverless-plugin-tracing # トレース有効用のプラグイン

provider:
  name: aws
  runtime: nodejs6.10
  stage: dev
  region: ap-northeast-1
  tracing: true # X-Rayでのトレースを有効にする
  iamRoleStatements:
    - Effect: Allow
      Action:
        - dynamodb:GetItem
        - dynamodb:PutItem
        - xray:PutTraceSegments #X-Ray周りの権限も忘れずに
        - xray:PutTelemetryRecords #X-Ray周りの権限も忘れずに
      Resource: "*"

custom:
  region: ${self:provider.region}
  prefix: ${self:service}-${self:provider.stage}
  config: ${self:custom.prefix}-config
  webpackIncludeModules: true

functions:
  apiGet:
    handler: api.get
    environment:
      region: ${self:custom.region}
      stage: ${self:provider.stage}
      NODE_ENV: ${self:provider.stage}
    events:
      - http:
          method: get
          path: xray/{id}
        request:
          parameters:
            paths:
              id: true
  apiPost:
      handler: api.post
      environment:
        region: ${self:custom.region}
        stage: ${self:provider.stage}
        NODE_ENV: ${self:provider.stage}
      events:
        - http:
            method: post
            path: xray

resources:
  Resources:
    usersTable:
      Type: 'AWS::DynamoDB::Table'
      Properties:
        TableName: datas
        AttributeDefinitions:
          - AttributeName: _id
            AttributeType: S
        KeySchema:
          - AttributeName: _id
            KeyType: HASH
        ProvisionedThroughput:
          ReadCapacityUnits: 20
          WriteCapacityUnits: 20

```

次に必要なモジュールをインストールします

```
npm i -D serverless-plugin-tracing
npm i -s aws-sdk aws-xray-sdk
```

ではLambdaファンクションのコードを書きます。今回は簡単なPOST・GETのエンドポイントを作成します。

```js
const awsXRay = require('aws-xray-sdk');
const AWS = awsXRay.captureAWS(require('aws-sdk'));
const Dynamo = new AWS.DynamoDB.DocumentClient({ region: 'ap-northeast-1' });

const tableName = 'datas';

function createData() {
  return new Promise((resolve, reject) => {
    const id = Math.random().toString(36).slice(-8).toString();
    const params = {
      TableName: tableName,
      Item: {
        _id: id,
        createdAt: Date.now().toString()
      }
    };
    Dynamo.put(params, function(err, data) {
      if (err) {
        reject(err);
      }
      resolve(data);
    });
  });
}

function getData(id) {
  return new Promise((resolve, reject) => {
    const params = {
      TableName : tableName,
      Key: {
        _id: id
      }
    }
    Dynamo.get(params, (err, data) => {
      if (err) {
        reject("Error occured", err);
      }
      resolve(data);
    })
  })
}


module.exports.get = (event, context, callback) => {
  getData(event.pathParameters.id).then((data) => {
    const response = {
      statusCode: 200,
      body: JSON.stringify({
        result: data,
      })
    };

    callback(null, response);

  }).catch((err) => {
    const response = {
      statusCode: 500,
      body: JSON.stringify({
        error: err,
      })
    };

    callback(null, response);
  });
};

module.exports.post = (event, context, callback) => {
  createData().then((data) => {
    const response = {
      statusCode: 200,
      body: JSON.stringify({
        result: data,
      })
    };

    callback(null, response);

  }).catch((err) => {
    const response = {
      statusCode: 500,
      body: JSON.stringify({
        error: err,
      })
    };

    callback(null, response);
  });
};
```

POSTでDynamoDBにデータを作成し、GETデータのIDをパスに指定してデータの中身を返すようなシンプルな内容です。
これをX-Rayでトレースしたいと思います。

ではまずデプロイ

```sh
sls deploy
```

デプロイが完了したらまずデータの作成をしてみましょうか。

```sh
curl -XPOST https://xxxxxxx.execute-api.ap-northeast-1.amazonaws.com/dev/xray
{"result":{}}
```

AWSコンソール画面でトレースできたか確認してみますか。まずサービスの中からX-Rayを選択し、画面内の「Trace」という項目を選択しましょう。

{{< figure src="/images/xray1.png" title="" >}}

トレースできているようですね。では↑の画像のIDをクリックしてみましょう。

{{< figure src="/images/xray2.png" title="" >}}

Lambda内で起きた処理とそれにかかった時間が詳細に見れます。

また、DynamoDBでPutItemを使っていることもここでわかります。

では次にService Mapを確認してみましょう。

{{< figure src="/images/xray3.png" title="" >}}

ここでは処理の順番が可視化されていてよりわかりやすくなっていますね。

では次にGETエンドポイントで先ほど作成したデータを取得してみましょう。

```sh
curl https://xxxxxxx.execute-api.ap-northeast-1.amazonaws.com/dev/xray/ibfhia4i
{"result":{"Item":{"_id":"ibfhia4i","createdAt":"1513498363715"}}}
```

こちらもトレースできているか確認してみましょう。

{{< figure src="/images/xray4.png" title="" >}}

{{< figure src="/images/xray5.png" title="" >}}

見れています。

では次にもし処理にある程度時間がかかった場合、それも確認できるか見てみましょう。

今回はGETエンドポイントに通常の処理+3秒待機時間が起きるように変更します。


```
module.exports.get = (event, context, callback) => {
  setTimeout(() => {
    getData(event.pathParameters.id).then((data) => {
      const response = {
        statusCode: 200,
        body: JSON.stringify({
          result: data,
        })
      };

      callback(null, response);

    }).catch((err) => {
      const response = {
        statusCode: 500,
        body: JSON.stringify({
          error: err,
        })
      };

      callback(null, response);
    });
  }, 3000)
};
```

変更をデプロイ後、またエンドポイントにアクセスします。

```sh
curl https://xxxxxxx.execute-api.ap-northeast-1.amazonaws.com/dev/xray/ibfhia4i
{"result":{"Item":{"_id":"ibfhia4i","createdAt":"1513498363715"}}}
```

トレース結果を確認してみると・・・

{{< figure src="/images/xray6.png" title="" >}}

{{< figure src="/images/xray7.png" title="" >}}

待機した分の時間が変化していることが把握できました！

<br>

## まとめ

今回みたいなシンプルな作りのアプリケーションであればあまりX-Rayの恩恵を実感できないかもしれませんが、
処理ごとに複数のLambdaで分けていたり、さらにそのLambdaから他のAWSのリソースに何回もアクセスしているような複雑な作りなアプリケーションなお場合だと処理の流れを把握するのは中々困難だと思います。

そういったアプリケーションにとってはX-Rayも活かすことができて、開発者もアプリケーションを監視しやすくなるのではないかと思います。

<br>

では良い開発ライフを！

<br><br>