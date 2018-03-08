---
title: "てくてく勉強会2月22日"
date: 2018-03-08T11:40:24+09:00
tags: ["勉強会","会社紹介","React","Redux"]
categories: ["藤井 大祐(daitasu)"]
---

<br>

みなさんこんにちは、<br>
パーソルプロセスアンドテクノロジー株式会社のAS統括部、daitasuです。<br>
今回も毎週の勉強会の様子を発信していきます！<br>
<br>
※技術的なブログについては現在Qiitaの方に移行致しました。<br>
テクテクチームのメンバーが各々に書き進めていますのでこちらもぜひご覧ください。<br>
<br>
[Qiita - パーソルプロセス&テクノロジー](https://qiita.com/organizations/persol-pt)<br>
<br>
さて、今週の発表者は萩原さん。<br>
テーマは **「ReactとRedux」** でした。<br>
<br>
<br>

### 2018年2月22日の勉強会　
### テーマ「ReactとRedux」
---

<br>

{{< figure src="/images/tech-workshop/20180222/20180222-01.jpg" title="" >}}<br>

以前の勉強会では、Vue.jsとAngular、Reactの比較が行われましたが、<br>
今回萩原さんが説明したのは、ReactにおけるReduxの話でした。<br>
<br><br>

### React.js
---
<br>

{{< figure src="/images/tech-workshop/20180222/20180222-02.png" title="" >}}<br>

<br>

そもそもReactとは何かのおさらいです。<br>
<br>
ReactはJavascriptのライブラリであり、<br>
JSX形式でJS内にHTMLタグを埋め込むような形で書いていきます。<br>
通常のJSですと、HTMLの中にJavascriptを書いていきますよね？<br>
<br>
あくまでもメインがHTMLであった今までの文化から、<br>
JSにより重要性を持たせてきた近代。<br>
JSが主体となるという点で個人的にはReactはなにかと近代感を感じます。<br>
<br>
また、component思考やstateによる状態管理も他とは思想が違う部分です。<br>
<br>
<br>
さて、Reactの話が上がるとよく出てくるものとして、<br>
<br>

* Redux
* webpack
* babel
* npm/yarn

<br>
などが上がるかと思います。<br>
今回はReduxにフォーカスを当てたお話でした。<br>
<br>
### FluxとRedux
---
<br>

**Reduxとは？**<br>
<br>
Reactが扱うstateを一元管理するためのフレームワークです。<br>
ReduxはFluxというデザインパターンの思想を引き継いでいます。<br>
<br>
<br>
**Fluxとは？**<br>
<br>
facebookが提唱したアーキテクチャです。<br>

* **Action**　View等から発火されるイベント。
* **Dispatcher**　アクションを受けて、全てのstoreに対しアクションを受け渡す。
* **Store**　アプリケーションのデータ。アクションによってのみデータ更新される。
* **View**　storeのデータをもとに表示するコンポーネント。

という4つのパートから組み立てられています。<br>
<br>
[Flux参考](https://github.com/facebook/flux/tree/master/examples/flux-concepts)<br>
<br>

**特徴**<br>

* データの流れが一方通行
* 全てのデータのオペレーションがDispackerに集約される

<br>
Reduxはこういった思想を引き継いでいるそうです。<br>
アプリが大きくなってくると、stateの管理が難しくなるために<br>
Reduxを利用してReduxでstate自体の管理と更新の管理をしています。<br>
<br>

### 所感
---
最後に、萩原さんが色々調べた結果の所感です。<br>
<br>

{{< figure src="/images/tech-workshop/20180222/20180222-03.png" title="" >}}<br>


ありがとうございました！<br>
それでは、また来週もお楽しみに！
<br><br><br><br>