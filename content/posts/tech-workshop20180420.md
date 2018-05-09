---
title: "勉強会[物体検出アルゴリズム YOLO]"
date: 2018-05-09T14:49:20+09:00
tags: ["勉強会","会社紹介","画像認識","YOLO", "機械学習"]
categories: ["藤井 大祐(daitasu)"]
---

<br>

みなさんこんにちは、<br>
パーソルプロセスアンドテクノロジー株式会社のEP統括部、daitasuです。<br><br>
<br>
今週も私たちの部の勉強会の様子をお送りいたします。<br>
<br>
※技術的なブログについてはQiitaでメンバーが各々に書き進めています。<br>
こちらもぜひご覧ください。<br>
<br>
[Qiita - パーソルプロセス&テクノロジー](https://qiita.com/organizations/persol-pt)<br>
<br>

さて、今週の発表者は私の直属のMGRであります、[川崎](https://persol-pt.github.io/categories/%E5%B7%9D%E5%B4%8E-%E5%8D%93%E6%BA%80/)さんのプレゼンです。<br>
テーマは「YOLOってみた」です。<br>
<br>
<br>

## テーマ「YOLOってみた」
---

{{< figure src="/images/tech-workshop/20180420/20180420-01.png" title="" >}}<br>

<br>

みなさん、YOLOってご存知でしょうか？<br>
私は今回初めて知ったのですが、YOLOというのは、<br>
'You only look once'、<br>
リアルタイムに画像認識を行い、物体を検出するアルゴリズムを指します。<br>
<br>
<br>
川崎さんは、今回自宅の動画を撮影し、このYOLOを利用して、<br>
物体検出がどのくらいの制度か、どのくらいの変換速度なのかを検証したお話を<br>
してくださいました。<br>
<br>
<br>

## そもそもYOLOってどんなもの？
---
<br>

{{< figure src="/images/tech-workshop/20180420/20180420-02.png" title="" >}}<br>

<br>

上記のように、YOLOはDarknetというフレームワークを用いて<br>
画像/動画からオブジェクトを検出しています。<br>
<br>
そして、そのオブジェクトが何なのか(人なのか、車なのか、植物なのか等)を<br>
判断し、分類をリアルタイムに行っています。<br>
<br>
実際の画像を見てみましょう！<br>
<br>

{{< figure src="/images/tech-workshop/20180420/20180420-03.png" title="" >}}<br>

<br>
「Person」と「laptop」が検出されていますね。<br>
このように、画像/動画から物体検出し、分類した結果を枠で囲い伝えてくれます。<br>
<br>
川崎さんが撮影した自宅の動画では、人や植物、机、TVなど<br>
想像以上に細かに分類がなされていました。<br>
<br>
以下のサイトなどを見て頂けると、<br>
YOLOの面白さが伝わるのではないかと思います。<br>
[YOLO動画](https://www.youtube.com/watch?v=_kxX09i4fds)
<br>
<br>
<br>

## 検証環境
---
<br>

{{< figure src="/images/tech-workshop/20180420/20180420-04.png" title="検証した環境" >}}<br>

{{< figure src="/images/tech-workshop/20180420/20180420-05.png" title="用意するもの" >}}<br>

<br>
YOLOはCPU単体だと非常に動きが悪くなるようで、GPUを積んだマシンで行う必要があります<br>
<br>
今回の検証では、CPU単体での処理と、<br>
GPUを利用した場合の2種類のプログラムで検証しています。<br>
<br><br>

## 検証結果
---

{{< figure src="/images/tech-workshop/20180420/20180420-06.png" title="検証結果" >}}<br>

CPU単体だと速度が全く出ず、<br>
動画でもすごく遅いGIF画像を見ているような動きになっていました。<br>
<br>
ラズパイなど処理能力の弱いPCを使う際には、<br>
GPUを持ったマシンに映像だけ転送して解析結果を返してもらうなどの<br>
工夫が必要とのこと。<br>
<br>
<br>
処理能力を一定担保しないといけないのはなかなかコストがかかる気もしますが、<br>
ラズパイのカメラでリアルタイムに動画を撮り続けて、YOLOって人の検出時に何かを行う、など<br>
うまく使いこなせばサービスの幅がうんと広げられそうな気がしますね。<br>
<br>
<br>

川崎さん、ありがとうございました！！<br>

<br><br><br>