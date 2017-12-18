---
title: "TerraformでAmazon ECS上にGitLabを構築"
date: 2017-12-18T00:56:09+09:00
tags: ["GitLab", "AWS", "ECS", "EFS", "Terraform"]
categories: ["boiyaa"]
---

[GitLab Advent Calendar 2017](https://qiita.com/advent-calendar/2017/gitlab) 17 日目の記事です。

<br>

私のプロジェクトではプロジェクト管理に GitLab を使っていて、元々シングルインスタンスに docker-compose で構築していて、[こんな記事](http://qiita.com/boiyaa/items/20a4fd0cc79f7d3b4a56)も書いたのですが、コンテナクラスターを一度触ってみたかったので、勉強がてら ECS 上に GitLab 構築してみました。

<br>

その結果の構成コードが[こちらのリポジトリ](https://github.com/boiyaa/gitlab-ecs-terraform)になりますので、同じことをやりたい方の参考になればと思います。

<br><br><br>

## 最初に結論

GitLab だけしか立てないなら ECS 上に構築するのはイマイチ旨味がありません。

もし HA を求めるなら、公式が[High Availability on AWS - GitLab Documentation](https://docs.gitlab.com/ce/university/high-availability/aws/)というページを用意していますので、これに従った方がいいと思います。

<br>

主に負荷の増減が大きいのは GitLab Runner の方なんですが、仕様上 ECS で管理できなかったので、スケーリングする機会がなく、また思ってた以上に ECS 構築が超絶面倒臭くて、Fat Terraform になってしまい、ただただ手間のかかった docker-compose みたいなものになっています。

<br>

先日発表された、ECS インスタンスをフルマネージドしてくれる新サービス[AWS Fargate](https://aws.amazon.com/jp/fargate/)とか、最近 GCP を触るようになってすっかり惚れたので GKE とかに移行を検討しています。

<br>

あと、Gitlab Meetup Tokyo #2 でこちらの[Ansible で作る、AWS で「器の大きい」Omnibus-GitLab // Speaker Deck](https://speakerdeck.com/attakei/ansibletezuo-ru-awste-qi-falseda-kii-omnibus-gitlab)という LT を拝聴して、「俺も 8,390,000TB にしたい！」という衝動にかられ、バージニアの EFS に構築したせいか普通に立てた時よりあきらかにもっさりしています。。

<br><br><br>

## ECS に GitLab 立てるときのポイント

### RDS を使う

GitLab イメージ内部の PostgreSQL を使っていると、ちょっとした拍子で

```
[EROR] Failed to ping DB retrying in 10 seconds err=dial unix /var/opt/gitlab/postgresql/.s.PGSQL.5432: connect: no such file or directory
```

になって起動しなくなりますので、RDS を使いましょう。

<br><br>

### GitLab Runner のオートスケール

[Runners autoscale configuration - GitLab Documentation](https://docs.gitlab.com/runner/configuration/autoscale.html)にあるように、Runner に用意されている、Docker Machine を利用した設定を使いました。

<br>

今回は AWS 上に構築したので、[Docker Machine AWS driver](https://docs.docker.com/machine/drivers/aws/)を使った設定をしましたが、以下の点でつまずきました。

* `--amazonec2-ami`で Amazon Linux AMI を指定すると`ERROR: OS type not recognized`となり非対応だったので、デフォルト（Ubuntu16.04）のままにした。
* 既存の Key Pair 使おうと思って`--amazonec2-keypair-name`と`--amazonec2-ssh-keypath`指定したら、`Waiting for SSH to be available...`のまま止まってしまったので諦めた。
* `--amazonec2-security-group`で自前のセキュリティグループを使うとき、ポート 22 と 2376 の設定が無いと勝手に 0.0.0.0/0 のインバウンドルールを追加してくるので、全トラフィックみたいな指定方法だと上書きされてしまう。
  * 上書きされてしまうと、`--amazonec2-private-address-only`を指定しても public ip で接続しようとするので、Private Subnet 内に立てていると、`ERROR: Too many retries waiting for SSH to be available.`となって、止まる。

<br>
