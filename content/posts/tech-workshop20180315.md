---
title: "勉強会[Kubernetesを使ってmicroserviceっぽいものを作ってみた]"
date: 2018-04-03T17:00:45+09:00
tags: ["勉強会","会社紹介","kubernetes"]
categories: ["藤井 大祐(daitasu)"]
---

<br>

みなさんこんにちは、<br>
パーソルプロセスアンドテクノロジー株式会社のAS統括部、daitasuです。<br><br>
<br>
今週も私たちの部の勉強会の様子をお送りいたします。<br>
<br>
※技術的なブログについてはQiitaでメンバーが各々に書き進めています。<br>
こちらもぜひご覧ください。<br>
<br>
[Qiita - パーソルプロセス&テクノロジー](https://qiita.com/organizations/persol-pt)<br>
<br>

さて、今週の発表者はKotlin愛好家の[宍戸](https://qiita.com/keitaro_1020)さん。<br>
テーマは「Kubernetesを使ってmicroserviceっぽいものを作ってみた」でした。<br>
<br>
<br>

### 2018年3月15日の勉強会　
### テーマ「Kubernetesを使ってmicroserviceっぽいものを作ってみた」
---

{{< figure src="/images/tech-workshop/20180315/20180315-01.jpg" title="" >}}<br>

<br>

宍戸さんはKotlinに魅了され、<br>
Kotlinで個人開発したり、業務でもKotlinを使ったりと<br>
Kotlinライフを送っています。<br>
<br>
そんな宍戸さんですが、<br>
今週はKubernetesでmicroserviceを作ったそうで、<br>
Kubernetesの概要と実際に作成したもののDEMOを発表して頂きました。<br>
<br>

### Kubernetesとは
---
<br>

Kubernetesはクーバネティス、クーベネティス、クーバネイティス、クバネティスなどと呼ばれ、<br>
人によって読み方はまちまちです。<br>
<br>
略称としては「k8s」と呼ばれており、Goで作られています。<br>
元々はGoogleが開発していたそうな。<br>
<br>
では実際にどんなことができるものなのでしょうか？<br>
<br>

KubernetesはDockerコンテナ群のオーケストレーションツールで、<br>
Dockerコンテナをいい感じに管理してくれるツールだそうです。<br>

* アプリのデプロイ
* 稼働中にアプリをスケーリング
* 新機能をシームレスに追加
* 自動修復

などができ、コンテナ群の運用自動化を目的として設計されたものです。<br>

<br>

### Kubernetesのアーキテクチャ
---

{{< figure src="/images/tech-workshop/20180315/20180315-02.png" title="" >}}<br>

Kubernetesのアーキテクチャには、<br>

* マスタ・コンポーネント
* ノード・コンポーネント

というものがあるそうです。<br><br><br>

### マスタ・コンポーネント
スケジューリングやイベントの検知などクラスタ全体の制御や管理を受け持つもの。<br>
構成要素は以下のようになっています。<br>

* API Server<br>
　　クラスタ制御のフロント・エンド。RESTful APIを提供。<br><br>
* Scheduler<br>
　　Podの展開を制御。<br><br>
* Controller<br>
　　クラスタ内のタスクを処理するバックグラウンド<br><br>
* etcd<br>
　　クラスタデータの保存<br>

<br><br>

### ノード・コンポーネント
Kubernetesランタイム環境を提供し、コンテナ化されたアプリケーションを実行します。<br>

構成要素としては、<br>
<br>

* kubelet<br>
　　ノードとマスターノード間の通信を行う。<br><br>
* proxy<br>
　　クラスタ内のサービスを仮想IPとする。<br><br>
* pod<br>
　　コンテナのセット<br><br>
* service<br>
　　podの集合に対するロードバランスを提供する<br><br>

からなるそうです。<br>

<br>

また、発表の中ではPodやServiceについても<br>
より詳しく説明をしてくださりました。<br>

<br>

### Kubernetes環境の選択肢
---

{{< figure src="/images/tech-workshop/20180315/20180315-03.png" title="" >}}<br>

Kubernetes環境の選択肢としては、<br>
ローカル環境での構築や温プレサーバ、仮想サーバへの自力構築などがあるそうですが、<br>
この辺は闇が深いので、パブリッククラウドのものを使うのが一番いいみたいですね。<br>
<br>

### まとめ
---

今回はKubernetesの概要や実際の扱い方までを教わりました。<br>
また、実際にKubernetesを利用して作成した本棚アプリのDEMOもお見せ頂きました。<br>
<br>
覚えるのは大変そうですが、<br>
マネージドサービス上で展開すれば、一定の設定だけで容易で作れますし、<br>
サーバ台数と関係なく複数アプリケーションを起動できる点や<br>
サーバを変えることなくアプリのデプロイ、シームレスな新機能の追加ができることなど<br>
メリットは非常に多そうな印象でした。<br><br>

宍戸さん、ありがとうございました。<br>

<br><br><br>