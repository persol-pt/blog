---
title: "静的ファイルWebホスティングならS3よりFirebase Hosting"
date: 2017-12-03T03:13:06+09:00
tags: ["Firebase", "S3", "CloudFront"]
categories: ["boiyaa"]
---

<br>

[Firebase Advent Calendar 2017](https://qiita.com/advent-calendar/2017/firebase) 3 日目の記事です。

<br>

[Firebase Hosting](https://firebase.google.com/products/hosting/)はストレージ、Web ホスティング、CDN 、マネージド SSL を組み込んだサービスです。 <br><br>

AWS ではストレージと Web ホスティングは S3 だけでできますが、CDN は CloudFront、マネージド SSL は Certificate Manager と、複数のサービスを組み合わせ、設定しないといけません。 <br><br>

CDN 性能も、Firebase Hosting で使用されている Google Cloud CDN は CloudFront より[パフォーマンス](https://www.cedexis.com/google-reports/)がいいようです。 <br><br><br>

### 比較表

<div>

|                | Firebase Hosting           | AWS                                                                |
| -------------- | -------------------------- | ------------------------------------------------------------------ |
| 構築           | 1 コマンド                 | S3 を作成。HTTPS や CDN を使うなら CloudFront を作成して S3 と結合 |
| デプロイ       | 1 コマンド                 | 1 コマンド                                                         |
| SSL 証明書     | 自動で作成＆更新＆適用     | ACM で作成して CloudFront に適用。自動更新                         |
| キャッシュ     | ブラウザキャッシュ設定のみ | CDN キャッシュ設定含め柔軟に設定可                                 |
| アクセス制限   | 無し                       | WAF を作成して CloudFront に適用                                   |
| 料金（ストア） | $0.025/GB                  | $0.025/GB                                                          |
| 料金（転送）   | $0.15/GB                   | $0.085~0.250/GB                                                    |

</div><p>アクセス制限が必要なければ、Firebase がとにかく楽です。 </p><br><br><br><br>

## 実際の手順を見てみましょう

### CLI ツールのインストールと設定の比較

<br>
#### Firebase

* Node.js をインストール
* Firebase CLI をインストール

```sh
$ npm install -g firebase-tools
```

* Firebase にログイン

```sh
$ firebase login
```

<br>
#### AWS

* Python をインストール
* AWS CLI をインストール

```sh
$ pip install awscli --upgrade --user
```

* AWS 認証情報を設定

```sh
$ aws configure
```

* CloudFront コマンド有効化

```sh
$ aws configure set preview.cloudfront true
```

<br><br>

### Web ホスティング環境構築の比較

<br>
#### Firebase の場合

* Firebase 設定ファイル作成

プロジェクトディレクトリに移動して、

```sh
$ firebase init
```

`Hosting`という項目を選択（スペースキー）する

* [https://console.firebase.google.com](https://console.firebase.google.com)にアクセスしてプロジェクトを作成
* Firebase プロジェクトを紐付ける

```sh
$ firebase use --add
```

* デプロイ

```sh
$ firebase deploy
```

これで`https://[project name].firebaseapp.com`にホスティングされます。 <br>

参考：[Hosting を使ってみる](https://firebase.google.com/docs/hosting/quickstart)

<br><br>

##### Firebase Hosting のカスタムドメイン設定

* Firebase Hosting admin panel を開く

```sh
firebase open hosting
```

* `Connect Domain`または`ドメインを接続`ボタンを押す
* 指定された A レコードを自分の DNS に設定する

なんとこれだけでカスタムドメインの SSL 証明書の作成・更新・適用までやってくれます。<br>

参考：[カスタム ドメインを接続する](https://firebase.google.com/docs/hosting/custom-domain)

<br><br><br>

#### AWS の場合（HTTPS 非対応）

* S3 バケット作成

```sh
$ aws s3api create-bucket --bucket [bucket name]
```

* バケットポリシー設定

```sh
$ cat > bucket-policy.json << EOF
{
  "Statement": [{
    "Effect": "Allow",
    "Principal": "*",
    "Action": ["s3:GetObject"],
    "Resource": ["arn:aws:s3:::[bucket name]/*"]
  }]
}
EOF

$ aws s3api put-bucket-policy --bucket [bucket name] --policy file://bucket-policy.json
```

参考：[ウェブサイトアクセスに必要なアクセス許可](http://docs.aws.amazon.com/ja_jp/AmazonS3/latest/dev/WebsiteAccessPermissionsReqd.html)

* Web ホスティング有効化

```sh
$ aws s3 website s3://[bucket name] --index-document index.html
```

* ファイルのデプロイ

```sh
$ aws s3 sync . s3://[bucket name]
```

これで`http://[bucket name].s3-website-us-east-1.amazonaws.com/`にホスティングされます。

<br><br>

##### S3 のカスタムドメイン設定

* ドメインの DNS を Route 53 に切り替える
* A レコードのエイリアスを選択し、作成した S3 バケットを選択する

参考：[例 : 独自ドメインを使用して静的ウェブサイトをセットアップする](http://docs.aws.amazon.com/ja_jp/AmazonS3/latest/dev/website-hosting-custom-domain-walkthrough.html)

<br><br><br>

#### AWS の場合（HTTPS 対応）

S3 の Web ホスティングでは HTTP のみなので、HTTPS 対応するには CloudFront と組み合わせる必要があります。

* S3 バケット作成

```sh
$ aws s3api create-bucket --bucket [bucket name]
```

* CloudFront OAI 作成

```sh
$ aws cloudfront create-cloud-front-origin-access-identity --cloud-front-origin-access-identity-config CallerReference=access-identity-[bucket name],Comment=access-identity-[bucket name]
```

参考：[CloudFront オリジンアクセスアイデンティティを作成してディストリビューションに追加する](http://docs.aws.amazon.com/ja_jp/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html#private-content-creating-oai)

* バケットポリシー設定

```sh
$ cat > bucket-policy.json << EOF
{
  "Statement": [{
    "Effect": "Allow",
    "Principal": {"AWS": "arn:aws:iam::cloudfront:user/CloudFront Origin Access Identity [origin access identity id]"},
    "Action": ["s3:GetObject"],
    "Resource": ["arn:aws:s3:::[bucket name]/*"]
  }]
}
EOF

$ aws s3api put-bucket-policy --bucket [bucket name] --policy file://bucket-policy.json
```

参考：[Amazon CloudFront オリジンアイデンティティへのアクセス権限の付与](http://docs.aws.amazon.com/ja_jp/AmazonS3/latest/dev/example-bucket-policies.html#example-bucket-policies-use-case-6)

* CloudFront ディストリビューション作成

```sh
$ cat > distribution.json << EOF
{
  "Comment": "",
  "Origins": {
    "Items": [
      {
        "S3OriginConfig": {
          "OriginAccessIdentity": "origin-access-identity/cloudfront/[origin access identity id]"
        },
        "Id": "origin",
        "DomainName": "[bucket name].s3.amazonaws.com"
      }
    ],
    "Quantity": 1
  },
  "DefaultRootObject": "index.html",
  "PriceClass": "PriceClass_All",
  "Enabled": true,
  "DefaultCacheBehavior": {
    "TrustedSigners": {
      "Enabled": false,
      "Quantity": 0
    },
    "TargetOriginId": "origin",
    "ViewerProtocolPolicy": "allow-all",
    "ForwardedValues": {
      "Cookies": {
        "Forward": "none"
      },
      "QueryString": false
    },
    "MinTTL": 0,
    "Compress": true
  },
  "CallerReference": "distribution-[bucket name]"
}
EOF

$ aws cloudfront create-distribution --distribution-config file://distribution.json
```

* ファイルのデプロイ

```sh
$ aws s3 sync . s3://[bucket name]
```

これで`https://[cloudfront domain name]`にホスティングされます。

<br><br>

##### CloudFront のカスタムドメイン設定

* ACM 証明書リクエスト作成

```sh
aws acm request-certificate --domain-name [your domain]
```

* ドメイン所有者のメールアドレスや、admin@[your domain]などに承認メールが届くので、承認
* ドメインの DNS を Route 53 に切り替え
* CloudFront ディストリビューション作成

```sh
$ cat > distribution.json << EOF
{
  "Comment": "",
  "Origins": {
    "Items": [
      {
        "S3OriginConfig": {
          "OriginAccessIdentity": "origin-access-identity/cloudfront/[origin access identity id]"
        },
        "Id": "origin",
        "DomainName": "[bucket name].s3.amazonaws.com"
      }
    ],
    "Quantity": 1
  },
  "DefaultRootObject": "index.html",
  "PriceClass": "PriceClass_All",
  "Enabled": true,
  "DefaultCacheBehavior": {
    "TrustedSigners": {
      "Enabled": false,
      "Quantity": 0
    },
    "TargetOriginId": "origin",
    "ViewerProtocolPolicy": "allow-all",
    "ForwardedValues": {
      "Cookies": {
        "Forward": "none"
      },
      "QueryString": false
    },
    "MinTTL": 0,
    "Compress": true
  },
  "CallerReference": "distribution-[bucket name]",
  "ViewerCertificate": {
    "SSLSupportMethod": "sni-only",
    "ACMCertificateArn": "[acm arn]",
    "MinimumProtocolVersion": "TLSv1.1_2016"
  },
  "Aliases": {
    "Items": ["[your domain]"],
    "Quantity": 1
  }
}
EOF

$ aws cloudfront create-distribution --distribution-config file://distribution.json
```

* A レコードのエイリアスを選択し、作成した CloudFront ディストリビューションを選択

<br><br><br><br>

## まとめ

HTTP のみのホスティングまでなら、Firebase と AWS の手順差はあまりありませんが、\
HTTPS やカスタムドメイン SSL が入ってくると AWS は設定が複雑になってきます。\
フロントエンドエンジニアの私は初めて CloudFront 触った時辛かったです。\
SPA のプロジェクトとかでバックエンドとインフラが分離している構成であれば、Firebase Hosting を検討されると良いかと思います。 <br><br>
