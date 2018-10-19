---
title: "勉強会[RPAを学ぶ　～UiPath～]"
date: 2018-10-19T10:46:23+09:00
tags: ["勉強会","会社紹介","RPA", "自動化", "UiPath", "todaken"]
categories: ["里井 嘉真(yoshi-sato)"]
---
<br>

皆さんこんにちは。<br>
パーソルプロセス＆テクノロジー株式会社のyoshi-satoです。<br>
今週もテクテクの勉強会の様子をお送りいたします。<br>
よろしくお願いします。<br>
<br>
※技術的なブログについてはQiitaでメンバーが各々に書き進めています。<br>
こちらもぜひご覧ください。<br>
<br>
[Qiita - パーソルプロセス&テクノロジー](https://qiita.com/organizations/persol-pt)<br>
<br>
今週の発表者は[todaken](https://qiita.com/todaken)さんです。テーマは「RPAを学ぶ ～UiPath～」でした。


<br>
## テーマ「RPAを学ぶ ～UiPath～」
---
RPA(Robotic Process Automation)といえば、人間の代わりにロボットを使って業務を行うものですが、  
定型業務とか自動化できるものはガンガン自動化しちゃおうぜ！ってことで昨今バズワードになっています。  
todakenさんは前回も[RPAについての発表](https://persol-pt.github.io/posts/tech-workshop_rpa/)をしていますが、  
今回は主にUiPathと呼ばれるものについてのお話でした。  
{{< figure src="/images/tech-workshop/20180928/20180928_173219995.jpg" title="" >}}
<br>
<br>

## RPAとは
---
そもそもRPAとは？  
Robotic Process Automation、直訳してしまうと「ロボットによるプロセスの自動化」  
ですが、ホワイトカラー労働者の間接業務を自動化するためのテクノロジーで、  
同じ作業の繰り返しや単純なフロント/バックオフィス業務を自動化することが出来ます。  
<br>
<br>

## UiPathでできること
---
RPAを実現するためのツールはたくさんの種類がありますが、その中でも今回はUiPathを取り上げます。  
UiPathで実現できることは多岐に渡りますが、  

- Excel、Word、Access等のデスクトップ上で行うアプリケーション操作
- クラウドサービスを含むWebアプリケーションの操作
- Webブラウザからのデータ取得（スクレイピング）

などが可能です。  

Community Editionであれば、多少の制限はありますが無料で利用できます。  
<br>
<br>

## UiPathの構成
---
UiPathは以下の3つの要素で構成されています。  
順番に見ていきましょう。  
<br>

#### 1. UiPath Studio
もっともRPAといえば！というようなツールが、UiPath Studioです。  
UiPath Studioでは、アクティビティと呼ばれるロボットの動作をドラッグ＆ドロップで指定し、ワークフローを作成します。  
直感的な操作でワークフローを組み立てることが出来るため、プログラミングコードは使用せずに進められます。  
実際に人間の操作を記録して、ロボットを作成することも可能です。  
<br>

#### 2. UiPath Orchestrator
UiPath Orchestratorは、作成したロボットの稼働状況の管理や、  
ジョブのスケジューリングやキューインが出来る、運用ツールです。    
Webベースで利用でき、リリース管理やログ管理もできます。  
また、APIを用いて外部のアプリケーションと連携をとることも出来ます。  
<br>

#### 3. UiPath Robot
こちらはUiPath Studioで作成した作業シナリオを実際に実行するツールです。  
業務で利用している個々端末で実行されるFront Office Robot(FOR)と、  
操作不要でサーバー上で動作するため常に稼動し続けられるBack Office Robot(BOR)の2種類があります。  
<br>
<br>
<br>

## Community Edition
---
Ui Path Community Editionは、ライセンスや自動更新などの面で制限がありますが、  
製品版Ui Pathで出来るほとんどのことが実現可能です。  
{{< figure src="/images/tech-workshop/20181019/11.JPG" title="" >}}
{{< figure src="/images/tech-workshop/20181019/12.JPG" title="" >}}  
<br>
<br>
## 個人的感想
---
最後に、todakenさんUi Pathに対する感想を共有いただきました。

- 日本語に対応している
- 操作のレコーディングが容易
- UIオブジェクト認識によるデータ入力が楽
- RPAExpressというRPAツールよりも動作が軽い
- とにかく出来ることが多い

など、かなり好感触だったそう。  
今回は出来なかった条件分岐やAPIを使った連携なども試してみたいとのことでした。  

{{< figure src="/images/tech-workshop/20181019/14.JPG" title="" >}}  
<br>

ロボットの動作を決める際に、コードを書かなくても良いというのは私個人的にはさびしく感じる面もありますが、  
小規模かつ学習コストも低く導入できるというのは、IT化が進んでいない業界や企業の選択肢を広げることになるか思います。  
[todaken](https://qiita.com/todaken)さん、どうもありがとうございました！  
