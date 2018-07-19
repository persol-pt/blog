---
title: "勉強会[nissy's handy tools]"
date: 2018-07-13T18:30:57+09:00
tags: ["勉強会","会社紹介","nissy", "Go", "tail"]
categories: ["里井 嘉真(yoshi-sato)"]
---

<br>

皆さんこんにちは、はじめまして。<br>
パーソルプロセス＆テクノロジー株式会社、EP統括部18年入社のyoshi-satoと申します。<br>
今週は藤井さんに代わり私がテクテク部の勉強会の様子をお送りいたします。<br>
よろしくお願いします。<br>
<br>
※技術的なブログについてはQiitaでメンバーが各々に書き進めています。<br>
こちらもぜひご覧ください。<br>
<br>
[Qiita - パーソルプロセス&テクノロジー](https://qiita.com/organizations/persol-pt)<br>
<br>
今週の発表者は同じテクテク部の[nissy](https://github.com/nissy)さんです。<br>
テーマは、[nissy](https://github.com/nissy)さんが作ったツール紹介でした。<br>
それぞれ簡単にご紹介したいと思います。
<br>
<br>


### mackerel-plugin-timeline
---
**[mackerel-plugin-timeline](https://github.com/nissy/mackerel-plugin-timeline)**は、Mackerelのログ表示処理を高速化するプラグインです。<br>
ロジックとしては、ログファイルを最終行付近までシークし、<br>
下から読み込むようになっています。<br>
<br>
<br>

### check-md5
---
**[check-md5](https://github.com/nissy/check-md5)**は、Mackerel等でMD5のバイナリハッシュ値を監視するプラグインです。<br>
このプラグインを作った理由としては、バイナリハッシュ値を<br>
勝手に書き換える輩がいたので犯人を見つけるために作った。そうです。<br>
あまり治安が良くなかったようです。<br>
<br>
<br>


### smtping
---
**[smtping](https://github.com/nissy/smtping)**は、SMTPサーバを監視、検証するためのツールです。<br>
ピュアなGo言語で書かれています。<br>
「一番がんばったのにもはや誰もテキストコンテンツに興味がない・・・」<br>
とは、nissyさん談です。<br>
<br>
<br>

### toever
---
**[toever](https://github.com/nissy/toever)**は、EverNoteにノートを作成、ファイルの転送を行うコマンドラインツールです。<br>
pythonで書かれています。<br>
GitHubでのStarの数は**驚異の15**。激バズです。<br>
やはりメジャーなサービスが絡むと人の目に留まりやすいようです。<br>
<br>
<br>

### txtmsk
---
**[txtmsk](https://github.com/nissy/txtmsk)**は、標準入力に入力した文字列をaes256方式で暗号化するツールです。<br>
もちろん、同ツールで復号することもできます。<br>
MacOSのKeychainと連携させることもできます。<br>
作った理由としては、前の上司がslackにパスワードを貼りまくっていたので、<br>
注意喚起のために作ったそうです。<br>
<br>
<br>

### phck
---
**[phck](https://github.com/nissy/phck)**は、Webサーバーのプロセスをチェックするツールです。<br>
プロセスがすべて起動していればHTTPコード200を返し、<br>
プロセスが1つでも落ちていればHTTPコード500を返します。<br>
<br>
<br>

### taii
---
**[taii](https://github.com/nissy/taii)**は、tailコマンドのようなコマンドです。<br>
というよりは、tailコマンドです。<br>
作った経緯としては、新人さんの一人がいつまでたってもtailをtaiiと<br>
タイプしてしまうのを見かね、それならばtaiiコマンドを作ってあげよう。<br>
という経緯だそうです。<br>
エイリアスを設定すればいいのではないか、という声も上がりましたが、<br>
イチから作るところにロマンがあります。<br>
nissyさんいわく欠点として、システムコールなのでめちゃめちゃに重い<br>
という点が挙げられるそうです。<br>
<br>
<br>

### loggerkun
---
**[loggerkun](https://github.com/nissy/loggerkun)**は、ロガーのフォーマットを自由に変更するツールです。<br>
仕組みとしては、テンプレートエンジンを利用して実装されています。<br>
<br>
<br>

### awslogger
---
**[awslogger](https://github.com/nissy/awslogger)**は、Amazon CloudWatch Logsにログを書き出すツールです。<br>
<br>
<br>

### colle
---
**[colle](https://github.com/nissy/colle)**は、Go言語で書かれたRSSフィードリーダーです。<br>
このcolleのすごいところは、某DMMの女性陣の画像を3秒ほどですべて取ってくることができるところだそうです。<br>
仕組みとしては、非同期にガッと行ってガッと取ってくるロジックだそうですが、<br>
今現在も使えるかどうかは不明です。<br>
<br>
<br>

### bon
---
**[bon](https://github.com/nissy/bon)**は、Goで書かれたWebフレームワークです。<br>
特徴として、軽量であること、サードパーティのパッケージを利用していないことが挙げられます。<br>
<br>
<br>
<br>

### 感想
---
今回nissyさんにご紹介いただいたツール群は、1つを除いてGoで書かれており、<br>
nissyさんのGo愛を感じました。<br>
nissyさんは自分が欲しいと思ったものはまず作る、というエンジニア魂で作成に取り組み、<br>
完成したものをGitHubに公開しておられます。<br>
欲しいツールを作成するのはもちろんですが、そもそも、自分が欲しいものに気づくということも、<br>
簡単なようで難しいスキルのように感じます。<br>
エンジニア歴数ヶ月（もしくは数週間）の私ですが、<br>
普段からいろいろなものに目を向け、欲しいものに気づいたらまずは作る<br>
といったエンジニアとしての姿勢は参考にしていきたいと感じました。<br>
<br>
また、今回ご紹介したツール群は[nissyさんのGitHubページ](https://github.com/nissy)で<br>
公開されていますので、もし興味を持たれた方は是非ともご覧ください。<br>
<br>
nissyさん、どうもありがとうございました。
<br>
<br>
<br>
