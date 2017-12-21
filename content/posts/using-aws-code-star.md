---
title: "Aws CodeStarを試してみる"
date: 2017-12-21T12:35:16+09:00
tags: ["AWS","CodeStar","CodeCommit","CodeBuild","CodeDeploy","Lambda","Node.js"]
categories: ["ビトル"]
---

今回は [Amazon Web Services Advent Calendar 2017](https://qiita.com/advent-calendar/2017/aws)の22日目の記事を書きたいと思います。

<br>

AWS CodeStarについて書こうと思います。

CodeStarは一言で言うとCI/CD環境を全てAWSで構築・運用できるサービスです。

Code三兄弟(Code Commit、Code Build、Code Deploy)などのサービスと連携して、
アプリケーションコードのコーディング、ビルド、テスト、デプロイできる環境の構築を可能にします。

開発者は素早く必要なアプリケーション開発環境と動作するコードを用意することでできるそうです。

<br>

<br>

## 試してみる

まずはアプリケーションのテンプレートを一つ選びます。

今回はNode.jsのLambdaアプリケーションを選びます。

<br>

{{< figure src="/images/codestar1.png" >}}

次にプロジェクトの詳細を設定します。プロジェクト名は好きなものを入れてください。

レポジトリはAWS CodeCommitとGitHubのどちらかを選べますが、今回はCodeCommitを選びます。

<br>

{{< figure src="/images/codestar2.png" >}}

ここではプロジェクトのどの部分でどのサービスが利用されるかが確認できます。

ソースの管理からデプロイまで全てAWS のサービスで管理できることがわかります。

<br>

{{< figure src="/images/codestar3.png" >}}

次にコードの編集方法を選択します。

EclipseやVisual StudioなどのIDEと連携もできますが、今回はコマンドラインで編集します。

<br>

{{< figure src="/images/codestar4.png" >}}

次の画面ではアプリケーションのソースの取得方法の手順を確認できます。

今回はsshでソースを取得しますので、ssh周りの設定が終わったらこの画面に表示されているソースのレポジトリのURLをコピーしましょう。

<br>

{{< figure src="/images/codestar5.png" >}}

ではコピーしたレポジトリのURLでgit cloneしましょう。

<br>

```sh
$ git clone ssh://git-codecommit.ap-northeast-1.amazonaws.com/v1/repos/awsCodeStartTest
Cloning into 'awsCodeStartTest'...
The authenticity of host 'git-codecommit.ap-northeast-1.amazonaws.com (xx.xxx.xxx.xxx)' can't be established.
RSA key fingerprint is SHA256:xxx/xxxxx/x/xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'git-codecommit.ap-northeast-1.amazonaws.com,xx.xxx.xxx.xxx' (RSA) to the list of known hosts.
remote: Counting objects: 16, done.
Receiving objects: 100% (16/16), 6.25 KiB | 0 bytes/s, done.
```

レポジトリの中身はこんな感じになります。
CodeBuildの `buildspec.yml` と CloudFormation `template.yml` があることが確認できますね。

<br>

```sh
├── README.md
├── buildspec.yml
├── index.js
├── public
│   ├── assets
│   │   ├── css
│   │   │   ├── gradients.css
│   │   │   └── styles.css
│   │   ├── img
│   │   │   └── tweet.svg
│   │   └── js
│   │       └── set-background.js
│   └── index.html
└── template.yml
```

では実際のアプリケーションのコードを見てみましょう。
単純にHTMLを返しているだけのシンプルなコードになっています。

<br>

```js
// index.js

'use strict';
var fs = require('fs');

 exports.get = function(event, context) {
   var contents = fs.readFileSync("public/index.html");
   context.succeed({
     statusCode: 200,
     body: contents.toString(),
     headers: {'Content-Type': 'text/html'}
   });
 };
```

次にもう一度CodeStarのコンソール画面に戻って作成したプロジェクトを選択してみましょう。

{{< figure src="/images/codestar6.png" >}}

これがCodeStarプロジェクトのダッシュボード画面になります。

右下の「アプリケーションのエンドポイント」の下に記載されているURLにアクセスしてましょう。

{{< figure src="/images/codestar7.png" >}}

アプリケーションのテンプレートのページが表示されました。

数秒でアプリケーションが出来上がりましたね。

{{< figure src="/images/codestar8.png" >}}

では次にソースを変更してpushしてみましょう。

<br>

```js
  'use strict';
- var fs = require('fs');

  exports.get = function(event, context) {
-   var contents = fs.readFileSync("public/index.html");
    context.succeed({
      statusCode: 200,
-     body: contents.toString(),
+     body: 'Hello world',
      headers: {'Content-Type': 'text/html'}
    });
  };
```

```sh
$ git add index.js
$ git commit -m "modify my code"
$ git push
```

CodeStarのダッシュボードに戻ると、先ほどpushしたコミットが反映されていることが確認できます。

{{< figure src="/images/codestar9.png" >}}

ソースの反映が終わったら、次にビルドが走りました。

buildspec.ymlに記載されているタスクが順番に実行されていきます。


{{< figure src="/images/codestar10.png" >}}

少し待つと、ビルドが無事完了しました。

{{< figure src="/images/codestar11.png" >}}


次にデプロイが走り、CloudFormationによって各リソースが更新されます。

{{< figure src="/images/codestar12.png" >}}

また少し待つと、デプロイが完了しました。

では先ほどと同じエンドポイントにアクセスしてみましょう。

{{< figure src="/images/codestar13.png" >}}

無事反映されました！！

{{< figure src="/images/codestar15.png" >}}

## まとめ

アプリケーションの実行環境だけでなく、ソース管理・ビルド・テスト・デプロイまで一通りできるCI環境が数分で構築できるのはかなり魅力的ですね。

開発者もアプリのソースコードにより集中できるので、生産性がかなり上がるかと思います。

今後自社サービスを作るときには使ってみたいですね。

<br>

ではまた！

<br>

<br>