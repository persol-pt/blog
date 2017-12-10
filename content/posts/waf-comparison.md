---
title: "Cloud-based WAF(Web Application Firewall) 比較"
date: 2017-12-06T01:32:59+09:00
tags: ["WAF", "Incapsula", "Silverline", "Akamai", "Cloudflare", "Sucuri", "Scutum", "AWS"]
categories: ["boiyaa"]
---

<br>

[Security Advent Calendar 2017](https://qiita.com/advent-calendar/2017/security) 6 日目の記事です。

<br>

WAF とは XSS や SQL インジェクションなどの HTTP 攻撃を防ぐための Firewall で、

Cloud-based WAF では、リクエストが WAF を通るよう WAF サービスに DNS を向け、安全なリクエストのみオリジンへ通す、という方式をとります。

私は WAF には疎くて、AWS WAF くらいしか知らなかったので、WAF ってのは自分で設定するものなんだろうなぁと思っていたんですが、調べてみたらマネージド WAF サービスって色々あるんですね。

<br>

サービス比較するにあたり、海外の WAF 紹介記事調べたり、ガートナーの WAF 市場の図を参考にしました。

<br>

{{< figure src="/images/mqwaf2016.png" title="Gartner Magic Quadrant for Web Application Firewalls 2016" >}}

<br>

{{< figure src="/images/mqwaf2017.png" title="Gartner Magic Quadrant for Web Application Firewalls 2017" >}}

<br><br>

ということで、市場リーダーの Imperva の[Incapsula](https://www.incapsula.com/)と Akamai の[クラウド・セキュリティ・ソリューション](https://www.akamai.com/jp/ja/products/cloud-security/)と F5 Networks の[Silverline](https://f5.com/jp/products/deployment-methods/silverline)、そして海外の WAF 紹介記事でよく見かける[Cloudflare](https://www.cloudflare.com/)と[Sucuri](https://sucuri.net/website-firewall/)、国産の[Scutum](https://www.scutum.jp/)、AWS を比較してみました。

<br><br><br>

## 比較表

<div>

|                      | Imperva<br>Incapsula                                     | F5 Networks<br>Silverline          | Akamai                                             | Cloudflare                                                 | Sucuri                                                                                | Scutum                          | AWS                                                      |
| -------------------- | -------------------------------------------------------- | ---------------------------------- | -------------------------------------------------- | ---------------------------------------------------------- | ------------------------------------------------------------------------------------- | ------------------------------- | -------------------------------------------------------- |
| WAF 料金             | $59~/ 月                                                 | お問い合わせ                       | お問い合わせ                                       | $20~/ 月                                                   | $9.99~/ 月                                                                            | ¥29,800~/ 月                    | $5~/ 月                                                  |
| DDoS 保護            | ✔️                                                       | ✔️                                 | ✔️                                                 | ✔️                                                         | ✔️                                                                                    | ✔️                              | ✔️                                                       |
| CDN                  | ✔️                                                       |                                    | ✔️                                                 | ✔️                                                         | ✔️                                                                                    |                                 | ✔️                                                       |
| HTTP/2 対応          | ✔️                                                       |                                    | ✔️                                                 | ✔️                                                         | ✔️                                                                                    |                                 | ✔️                                                       |
| WebSocket 対応       | ✔️                                                       |                                    |                                                    | ✔️                                                         |                                                                                       |                                 |                                                          |
| マネージド SSL       | ✔️                                                       |                                    | ✔️                                                 | ✔️                                                         | ✔️                                                                                    |                                 | ✔️                                                       |
| DNS                  |                                                          |                                    | ✔️                                                 | ✔️                                                         | ✔️                                                                                    |                                 | ✔️                                                       |
| 日本語サポート       | ✔️                                                       | ✔️                                 | ✔️                                                 |                                                            |                                                                                       | ✔️                              | ✔️                                                       |
| WAF 市場での位置付け | リーダー (2 年連続 )                                     | リーダー                           | リーダー                                           | チャレンジャー                                             |                                                                                       |                                 | ニッチプレイヤー                                         |
| 特徴                 | スモールビジネスからエンタープライズまで対応した料金設定 | ネットワーク・アプライアンスで有名 | 全 Web トラフィックの 15~30% を配信する CDN 最大手 | 世界シェア 35% の DNS 最大手<br>個人でも手が出せる料金設定 | 個人でも手が出せる料金設定<br>最安プランから WAF と DDoS 保護が付いていて全体的にお得 | 国産<br>Struts 脆弱性対応が早い | AWS リソースと連携しやすい<br>個人でも手が出せる料金設定 |

</div>
<br>

料金プランのあるサービスはプランによって内容が異なりますので、それも比較します。

<br><br>

## プラン別機能比較

### Incapsula

<div>

|                | Free   | Pro     | Business | Enterprise   |
| -------------- | ------ | ------- | -------- | ------------ |
|                | $0/ 月 | $59/ 月 | $299/ 月 | お問い合わせ |
| WAF            |        | ✔️      | ✔️       | ✔️           |
| DDoS 保護 L7   |        |         | ✔️       | ✔️           |
| DDoS 保護 L3-4 |        |         |          | ✔️           |
| DDoS 保護 DNS  |        |         |          | ✔️           |
| CDN            | ✔️     | ✔️      | ✔️       | ✔️           |
| HTTP/2 対応    |        | ✔️      | ✔️       | ✔️           |
| WebSocket 対応 |        | ✔️      | ✔️       | ✔️           |
| マネージド SSL |        | ✔️      | ✔️       | ✔️           |
| カスタム SSL   |        |         | ✔️       | ✔️           |
| DNS            |        |         |          |              |

</div>
<br>

### Cloudflare

<div>

|                | Free   | Pro                    | Business | Enterprise  |
| -------------- | ------ | ---------------------- | -------- | ----------- |
|                | $0/ 月 | $20/ 月                | $200/ 月 | $5,000~/ 月 |
| WAF            |        | ✔️<br>カスタムできない | ✔️       | ✔️          |
| DDoS 保護 L7   | ✔️     | ✔️                     | ✔️       | ✔️          |
| DDoS 保護 L3-4 | ✔️     | ✔️                     | ✔️       | ✔️          |
| DDoS 保護 DNS  | ✔️     | ✔️                     | ✔️       | ✔️          |
| CDN            | ✔️     | ✔️                     | ✔️       | ✔️          |
| HTTP/2 対応    | ✔️     | ✔️                     | ✔️       | ✔️          |
| WebSocket 対応 | ✔️     | ✔️                     | ✔️       | ✔️          |
| マネージド SSL | ✔️     | ✔️                     | ✔️       | ✔️          |
| カスタム SSL   |        |                        | ✔️       | ✔️          |
| DNS            | ✔️     | ✔️                     | ✔️       | ✔️          |

</div>
<br>

### Sucuri

<div>

|                | Basic     | Pro        | Business   |
| -------------- | --------- | ---------- | ---------- |
|                | $9.99/ 月 | $19.98/ 月 | $69.93/ 月 |
| WAF            | ✔️        | ✔️         | ✔️         |
| DDoS 保護 L7   | ✔️        | ✔️         | ✔️         |
| DDoS 保護 L3-4 |           | ✔️         | ✔️         |
| DDoS 保護 DNS  |           | ✔️         | ✔️         |
| CDN            | ✔️        | ✔️         | ✔️         |
| HTTP/2 対応    |           | ✔️         | ✔️         |
| WebSocket 対応 |           |            |            |
| マネージド SSL | ✔️        | ✔️         | ✔️         |
| カスタム SSL   |           |            | ✔️         | ✔️ |
| DNS            | ✔️        | ✔️         | ✔️         |

</div>
<br>

### Scutum

<div>

|                | 基本料金    | DDoS 対策オプション |
| -------------- | ----------- | ------------------- |
|                | ¥29,800/ 月 | お問い合わせ        |
| WAF            | ✔️          |                     |
| DDoS 保護 L7   |             | ✔️                  |
| DDoS 保護 L3-4 |             | ✔️                  |
| DDoS 保護 DNS  |             |                     |
| CDN            |             |                     |
| HTTP/2 対応    |             |                     |
| WebSocket 対応 |             |                     |
| マネージド SSL |             |                     |
| カスタム SSL   | ✔️          |                     |
| DNS            |             |                     |

</div>
<br>

### AWS

<div>

|                | WAF(Shield Standard)                           | Shield Advanced                                   | CloudFront                                            | ACM | Route 53                                           |
| -------------- | ---------------------------------------------- | ------------------------------------------------- | ----------------------------------------------------- | --- | -------------------------------------------------- |
|                | [料金](https://aws.amazon.com/jp/waf/pricing/) | [料金](https://aws.amazon.com/jp/shield/pricing/) | [料金](https://aws.amazon.com/jp/cloudfront/pricing/) | $0  | [料金](https://aws.amazon.com/jp/route53/pricing/) |
| WAF            | ✔️<br>セルフマネージド                         | ✔️<br>DRT に委託可能                              |                                                       |     |                                                    |
| DDoS 保護 L7   | ✔️<br>セルフマネージド                         | ✔️<br>DRT に委託可能                              |                                                       |     |                                                    |
| DDoS 保護 L3-4 | ✔️                                             | ✔️                                                |                                                       |     |                                                    |
| DDoS 保護 DNS  |                                                | ✔️                                                |                                                       |     |                                                    |
| CDN            |                                                |                                                   | ✔️                                                    |     |                                                    |
| HTTP/2 対応    |                                                |                                                   | ✔️                                                    |     |                                                    |
| WebSocket 対応 |                                                |                                                   |                                                       |     |                                                    |
| マネージド SSL |                                                |                                                   |                                                       | ✔️  |                                                    |
| カスタム SSL   |                                                |                                                   |                                                       | ✔️  |                                                    |
| DNS            |                                                |                                                   |                                                       |     | ✔️                                                 |

</div><p>WAF のみだとセルフマネージドですが、設定済み CloudFormation テンプレートが用意されています。詳細は[AWS WAF Security Automations](https://aws.amazon.com/answers/security/aws-waf-security-automations/)</p>

これを設定すると料金はこうなるそうです。

<br>
<div>

| リクエスト | 料金 / 月 |
| ---------- | --------- |
| 100 万     | $14.65    |
| 1000 万    | $46.50    |
| 1 億       | $79.00    |

</div>
<br><br>

## まとめ

一言 WAF と言っても各社提供している攻撃対策が異なり、対象サイトに必要な対策が何かにもよるので、

一概にどれがお勧めとは言えませんが、

<br>

Incapsula と Cloudflare は、機能が豊富で、かつ機能を絞って低価格な料金プランを選ぶこともでき、また HTTP/2 や WebSocket など色々な Web 技術に対応していたりと、オプションの豊富なサービスでした。

<br>

また、Incapsula はサポートにも力が入っているようで、登録するとすぐに CS から連絡が来て、セールスやエンジニアにすぐ聞いて答えてくれました。

<br>

なんでもいいから安いのがいいということであれば Sucuri ですね。

<br>

Incapsula が他製品との詳細な機能比較表を作っていまして、こちらも参考になりました。

[INCAPSULA COMPETITIVE LANDSCAPE](https://www.incapsula.com/incapsula-competitive-landscape.html)

<br>

以上、門外漢がお送りしました。

<br>
