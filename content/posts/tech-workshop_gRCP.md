---
title: "勉強会[gRPC]"
date: 2018-06-11T13:53:08+09:00
tags: ["勉強会","会社紹介","gRPC"]
categories: ["藤井 大祐(daitasu)"]
---

<br>

みなさんこんにちは、<br>
パーソルプロセスアンドテクノロジー株式会社のEP統括部、daitasuです。<br><br>
<br>
今回も私たちの部の勉強会の様子をお送りいたします。<br>
<br>
※技術的なブログについてはQiitaでメンバーが各々に書き進めています。<br>
こちらもぜひご覧ください。<br>
<br>
[Qiita - パーソルプロセス&テクノロジー](https://qiita.com/organizations/persol-pt)<br>
<br>

さて、今週の発表者は[vitor](https://qiita.com/vitor)さん。テーマは「gRPC」です。<br>
<br>
<br>

## テーマ「gRPC」
---

{{< figure src="/images/tech-workshop/20180518/20180518-01.jpg" title="" >}}<br>

<br>

本日の勉強会は、gRPCという2015年にGoogleが開発した<br>
RPCフレームワークについてのお話しでした。<br>
<br>

## gRPCとは
---

<br>

gRPCというのは、RPC(リモートプロシージャコール)を実現するために開発されたプロトコルの一つです。<br>
<br>
リモートプロシージャコールというのは、ネットワークに接続された他のサーバからプログラムを呼び出し、<br>
実行させるための手法やプロトコルのことを言います。<br>
<br>
例えば、あるサーバから別サーバ(別PC)に存在するアプリ内のJavaコードを実行させる等
<br>
<br>
gRPCは、このRPCを実現するために2015年にGoogle社によって開発されました。<br>
gRPCを用いることで、以下のようなことが可能になるそうです。<br>
<br>

{{< figure src="/images/tech-workshop/20180518/20180518-02.png" title="" >}}<br>


<br>

gRPCはProtocol Buffersというフォーマットを用いてデータをシリアライズ化して書かれています。<br>

<br>
<br>

## Protocol Buffers
---

<br>
プロトコルバッファーとは、IDL(インタフェース定義言語)で構造を定義し、<br>
データをシリアライズ化するフォーマットで、データ通信・永続化を目的としています。<br>
<br>
データをシリアライズ化することで、データサイズの縮小・高速送信に繋が流そうです。<br>
<br>
<br>

## 仕組み
---

<br>

gRPCでは、IDL(インターフェース定義言語)を用いてあらかじめAPI仕様を **.proto**ファイルとして定義します。<br>
言語に依存しないIDLを利用して、先にインターフェースを定義することで、多種のプログラミング言語で書かれた<br>
サーバ間の通信を可能にしています。<br>
<br>
以下の図の例だとサーバサイドをC++で開発しており、クライアントをRubyやAndroid-Javaで書いてあります。<br>
参考サイト：[What is gRPC?](https://grpc.io/docs/guides/)<br>

{{< figure src="/images/tech-workshop/20180518/20180518-04.png" title="" >}}<br>


<br>
<br>

## protoファイルの定義
---
<br>

protoファイルでのgRPCの設定の例も取り上げていました。<br>
下記のようにリクエストやレスポンスの定義を行います。<br>
このように指定することでデータ構造をシリアル化し、様々な言語での取り出しを可能にしています。<br>
<br>

```
// The greeter service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
message HelloRequest {
  string name = 1;
}

// The response message containing the greetings
message HelloReply {
  string message = 1;
}
```

参考サイト：[What is gRPC?](https://grpc.io/docs/guides/)
<br>
<br>
<br>

## 感想
---

今回、vitorさんはgRPCを用いてフロントがNode、サーバサイドがRubyで描かれた<br>
簡易なdemoを行なってくださいました。<br>
<br>

感想としては、以下のような利点や課題があるそうです。<br>
<br>

{{< figure src="/images/tech-workshop/20180518/20180518-03.png" title="" >}}<br>

RESTにパラメータの型付けができるのはとてもありがたいですね。<br>
とはいっても、RESTがやはり根強く浸透していますし、<br>
ネット上の情報量も考えると、なかなか導入は難しいようです。<br>
<br>

vitorさん、ありがとうございました！！<br>

<br><br><br>