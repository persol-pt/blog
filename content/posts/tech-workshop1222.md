---
title: "勉強会[http request multiplexerと文字列マッチング]"
date: 2017-12-29T23:58:18+09:00
tags: ["勉強会", "もくもく会","会社紹介","Go","httpルーティング"]
categories: ["藤井 大祐(daitasu)"]
---

皆さんこんにちは、新卒の藤井大祐(daitasu)です。<br>

<br>

今週も私たちパーソルプロセスアンドテクノロジーのテクテクチームが<br>
行っている技術勉強会のレポートをしていきたいと思います。<br>
<br>
今年の勉強会ブログはこれで最後になります。<br>
<br>
<br>
今週の発表者は定期的に自身でパッケージを作成し、OSSに貢献しているnissyさん。<br>
<br>
テーマは **「http request multiplexer と文字列マッチング」** でした。<br>
<br>
<br>

### 12月22日の勉強会　テーマ「http request multiplexer と文字列マッチング」
---

<br>

{{< figure src="/images/tech-workshop/20171222/tech1222-1.jpg" title="" >}}<br>

<br>
通常URLにリクエストする際、そのURL文字列にマッチするパターンを<br>
登録されているパターンの中から検索し、呼び出されるハンドラを決めようとします。<br>
このルーティング処理はその言語やフレームワークでの処理方法によって<br>
大きく処理速度が異なります。<br>
<br>
今回nissyさんは基数木(パトリシア木)という集合データ構造を用いて<br>
Go言語でルーティング処理を行うパッケージを作成しました。<br>
<br>

[Bon(https://github.com/nissy/bon)](https://github.com/nissy/bon)<br>

<br>

今回のお話では、<br>

基本的なマッチングパターンのルールから<br>
`トライ木(Trie)による複数パターンマッチング` や `KMP` といった<br>
代表的な文字列パターンマッチングのアルゴリズム紹介<br>
に始まり、<br>

<br>

いくつかの代表的なアルゴリズムの速度比較を行いまいした。<br>

以下はパトリシア木(Patricia Trie)の構造紹介とその速度表です。<br>

<br>

[Patricia Trie(https://github.com/nissy/mux/tree/patricia)](https://github.com/nissy/mux/tree/patricia)<br>

<br>

{{< figure src="/images/tech-workshop/20171222/tech1222-2.png" title="Patricia Trieの構造" >}}<br>

{{< figure src="/images/tech-workshop/20171222/tech1222-3.png" title="Patricia Trie 速度" >}}<br>

<br>

そしてその後、実際に自身がPatricia Trieをもとにチューニングし、<br>
Go言語で作成したルーティング手法のパッケージ「 **Bon** 」についての紹介を行い、<br>
他のGo言語フレームワークとの速度比較を行いました。<br>

<br>

{{< figure src="/images/tech-workshop/20171222/tech1222-4.png" title="Bon 比較" >}}<br>

{{< figure src="/images/tech-workshop/20171222/tech1222-5.png" title="BonとGo言語フレームワークの速度比較" >}}<br>

<br>


今回このルーティングパッケージBonを作成してみた感想として、<br>

* スケール当たり前の時代なので速度的にはプロセスの起動速度も気にする
* Routerは注目度が高いので作りがいがある
* 高速化はテンション上がる

といったことを述べられていました。<br>

<br>

やはり高速化は胸が高まりますね。<br>

<br>
<br>

さて、今年の勉強会はここまでとなります！<br>
みなさま、よいお年を！<br>

<br>
<br>
<br>
