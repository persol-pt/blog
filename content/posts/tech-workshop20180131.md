---
title: "てくてく勉強会1月24日"
date: 2018-01-31T13:38:55+09:00
tags: ["勉強会", "もくもく会","会社紹介","WebRTC", "入門"]
categories: ["藤井 大祐(daitasu)"]
---

<br>

みなさんこんにちは、パーソルプロセスアンドテクノロジー株式会社のAS統括部、[daitasu](https://persol-pt.github.io/categories/%E8%97%A4%E4%BA%95-%E5%A4%A7%E7%A5%90daitasu/)です。<br>
今週も私たち、テクテクチームの毎週の勉強会の様子を発信していきます！<br>
<br>
技術的なブログについては現在QiitaのOrganizationに移行しております。<br>
<br>
テクテクチームのメンバーが各々に書き進めていますのでこちらもぜひご覧ください。<br>
<br>
[Qiita - パーソルプロセス&テクノロジー](https://qiita.com/organizations/persol-pt)<br>
<br>

<br>
さて、今週の発表者は毎週このブログをお届けしております、私、[藤井 大祐(daitasu)](https://persol-pt.github.io/categories/%E8%97%A4%E4%BA%95-%E5%A4%A7%E7%A5%90daitasu/)です。<br>
<br>
テーマは **「WebRTCコトハジメ」** 。<br>
<br>
Web RTCを始めるにあたっての基礎知識・導入を発表しました。
<br>
<br>

### 2018年1月22日の勉強会　
## テーマ「Web RTC コトハジメ」
---

<br>

{{< figure src="/images/tech-workshop/20180131/20180131-01.jpg" title="" >}}<br>

<br>
新卒でこの会社に入り、現在オンラインチャットアプリの開発に取り組んでいるのですが、<br>
その過程でビデオチャットを取り扱う機会がありました。<br>
<br>
Web RTCというのは、ビデオチャットをブラウザ上でPeer to Peerに取り扱うことができる技術になります。<br>
<br>
<br>
### Web RTCとは？
---
<br>
Werb RTCとは、Web Real-Time-Communicationのこと。<br>
ビデオチャットなどのリアルタイムコミュニケーションのために<br>
HTML5で新しく策定されたAPIの規格になります。<br>
<br>
Web RTCにおいて重要なのは以下の3つの機能。<br>
<br>

- カメラ/デバイスへのアクセス<br>
　**Media Capture and Stream**
- ビデオ/オーディオ/データ通信を行う<br>
　**Web RTC 1.0: Real-time Communication Between Browsers**
- カメラ/デバイスへのアクセス<br>
　**MediaStream Recording**

それらの技術に対し、<br>
CSS3やCanvas WebGL等々を組み合わせることで、<br>
音声を変換したり、カメラから受け取った映像のキャプチャを拾い、<br>
画像処理をして返す。顔検出をする。など様々な活用を行うことができます。<br>
<br>
<br>
今回は、上記の3つの機能に関し、仕組みやAPIの説明、<br>
実際に動かしてみたもののDEMOを行いました。<br>
<br>
詳しい内容については様々なサイトにより分かりやすい記載があるかと思うので、<br>
この記事では概要だけ書書かせて頂きます。<br>
<br>

### Media Capture and Stream
---
<br>
こちらはブラウザからカメラやマイクにアクセスするためのAPI規格。<br>
<br>
こちらで重要なので、`getUserMedia()` というメソッド。<br>
<br>
以下のように記述することでPromiseベースでカメラ/マイクへの接続を許可します。<br>
<br>
```
let Video = document.getElementById('local_video');
let Stream;

navigator.mediaDevices.getUserMedia({
      video: true,
      audio: true
    }).then(function (stream) {
      Stream = stream;
      Video.src = window.URL.createObjectURL(Stream);

    }).catch(function (error) {
      console.error('getUserMedia() error ->', error);
      return;
    });
```

<br>
ビデオとオーディオにそれぞれtrue/falseを指定して渡し、<br>
アクセス成功時にはそのstreamが返ってくるので、それをvideoタグの中に入れてあげます。<br>
<br>
また、videoやaudioには様々なオプションがあり、<br>
高さや幅、ビットレートなど指定してあげることが可能です。<br>
例：

```
 video: {
    width: 640,
    height: 480
    frameRate: { min: 10, max: 15}
 }
```

このようにして取得された映像や音声はvideoタグにsrcとして渡され、<br>
CSSで加工することが可能です。<br>
<br>

{{< figure src="/images/tech-workshop/20180131/20180131-02.jpg" title="" >}}<br>

<br>
色んな加工をして遊んでみてください。<br>
<br>
### Web RTC 1.0: Real-time Communication Between Browsers
---
<br>
次に、ブラウザ間の通信についてです。<br>
Web RTCでは、クライアントがサーバにデータの要求をする一般のモデルとは異なり、<br>
複数端末がそれぞれデータを保持し、他の端末に対し、データの送信・要求を直接的に行う<br>
**Peer to Peer**の形をとります。<br>
また、通信プロトコルにはTCP/IPの代わりにUDP/IPを用いて通信のリアルタイム性を<br>
上げていることも特徴です。<br>
<br>
P2P通信に必要な情報は以下の2点です。<br>

{{< figure src="/images/tech-workshop/20180131/20180131-03.png" title="" >}}<br>

<br>
P2P通信を行うためには、各ブラウザの情報を相手に渡してあげる必要があります。<br>
そのために、それらの情報が詰まったSDPプロトコルと経路情報を示したICEを渡さなくてはなりません。<br>
<br>
その際、社内LANなどだとNAT配下にあることが多くあるかと思います。<br>
<br>
Web RTCでは、このNAT配下にあるPC端末間の情報を渡しあうために、<br>
STUNサーバ・TURNサーバというものを用いています。<br>
<br>

{{< figure src="/images/tech-workshop/20180131/20180131-04.png" title="STUNサーバ概要" >}}<br>

{{< figure src="/images/tech-workshop/20180131/20180131-05.png" title="TURNサーバ概要" >}}<br>

<br>
<br>
また、Web SocketやSocket.IO等のそれらのデータを経由するためのシグナリングサーバを<br>
準備する必要があります。<br>
<br>
仕組みはSTUNサーバを用いて自身のIP等の情報を取得。<br>
シグナリングサーバ経由で自身のSDPを生成・登録後、情報を相手側に伝えます。<br>
相手側にSDPが登録されたあと、同様に相手からもSDPの生成・登録が行われ、送信側に送信・登録。<br>
<br>
このようにしてP2P通信を実現します。<br>
<br>
また、上記過程でP2Pができなかった際には、<br>
TURNサーバを経由してデータが送られます。<br>
<br>
SDPの生成・登録部分もPromiseベースで一気に行うことができます。<br>
SDP登録後、非同期でイベントが走り、相手側に情報が送られるようになっています。<br>
<br>
```
// Offer SDPを生成する
function makeOffer() {
    peerConnection = prepareNewConnection();
    peerConnection.onnegotiationneeded = function(){
        //createOffer()でSDP（ブラウザが利用可能なWebRTCの通信に必要な各種情報）が生成
        peerConnection.createOffer()
            .then(function (sessionDescription) {
                console.log('createOffer() succsess in promise');
                //setLocalDescription()で生成されたSDPをセット
                return peerConnection.setLocalDescription(sessionDescription);
            }).then(function() {
                //完了後、非同期でpeer.onicecandidateイベントが走るようになる
                console.log('setLocalDescription() succsess in promise');
        }).catch(function(err) {
            console.error(err);
        });
    }
}
```

<br>
### まとめ
---
<br>
Web RTCのAPIは年々更新されているため、まだ仕様が変わることがあるかと思います。<br>
とはいえ、自由にビデオチャット/音声チャットをプログラマブルに扱うことができ、<br>
自社製品などに組み込むことができるのは魅力的ですね。<br>
<br>
<br>
また、今回の発表ではコトハジメで終わってしまったので、<br>
次回の発表ではより実用的なモノを作ろうと考えています。<br>
<br>

{{< figure src="/images/tech-workshop/20180131/20180131-06.png" title="Web RTCで作りたいもの概要" >}}<br>

<br>
<br>
今回、以下のサイトを参考にさせていただきました。<br>
Web RTCの仕組みと実際に動かすまでの詳細が書かれていましたので、<br>
より詳しく深めたい方はそちらをご覧ください。<br>
<br>

[WebRTCハンズオン 本編](https://qiita.com/yusuke84/items/43a20e3b6c78ae9a8f6c)<br>
[Web RTCの技術解説](https://www.slideshare.net/nttwestcon/20140805-technical-descriptionofwebrtcpublicedition)<br>
<br>
<br>
<br>
<br>
