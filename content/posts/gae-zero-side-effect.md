---
title: "純粋な App Engine"
date: 2017-12-25T00:36:45+09:00
tags: ["GCP", "GAE", "Elm", "Haskell"]
categories: ["boiyaa"]
---

[Google Cloud Platform Advent Calendar 2017](https://qiita.com/advent-calendar/2017/gcp) 24 日目の記事です。

<br>

先日弊社のシシドさんが書いた[GCP Meets Kotlin](https://persol-pt.github.io/posts/gcp-meets-kotlin/)を参考に、

サンタクロースを信じる私の娘のように純粋な関数型言語で GAE(Google App Engine) アプリケーションを作ってみます。

※私はガチ勢ではなく、Elm の簡単さが好きなだけの人ですので、内容超浅いです。

<br>

[今回のソースコード](https://github.com/boiyaa/gae-zero-side-effect-stack-example)

<br><br>

## App Engine の準備

Google にログインしている状態で、[GCP コンソールのプロジェクト作成画面](https://console.cloud.google.com/projectcreate)を開き、プロジェクトを作成します。

{{< figure src="/images/gae-zero-side-effect/gcp-createproject.png" >}}

[Google App Engine Admin API](https://console.cloud.google.com/apis/library/appengine.googleapis.com/)を有効にします。

<br>

[Google Cloud SDK](https://cloud.google.com/sdk/)をインストールします。

```sh
$ curl https://sdk.cloud.google.com | bash
$ exec -l $SHELL
$ gcloud init
```

先ほど作成したプロジェクトに切り替えます。

```sh
$ gcloud config set project your-project
```

App Engine を作成します。

```sh
$ gcloud app create
```

以上です。

<br><br>

## Web API の実装

人生初の Haskell コーディングです。

[What are the best Haskell web frameworks for building RESTful web services?](https://www.slant.co/topics/727/~best-haskell-web-frameworks-for-building-restful-web-services)という記事を参考に、

今回は[Spock](https://www.spock.li/)という、Java 屋には紛らわしい名前の Web Framework を使います。

理由は Sinatra インスパイアで、公式サイトのデザインとチュートリアルの内容が他より Welcome な感じだったからという、怒られそうな理由です、すいません。

<br>

では[Building a REST API](https://www.spock.li/tutorials/rest-api)を参考に実装してみます。

`Stack`という Haskell の npm 的なやつをインストールします。

```sh
$ curl -sSL https://get.haskellstack.org/ | sh
```

Haskell プロジェクト作ります。

```sh
$ stack new your-project
$ cd your-project
```

`stack.yaml`を以下のように編集します。

```diff
-resolver: lts-10.0
+resolver: lts-8.13
```

`package.yaml`を以下のように編集します。

```diff
executables:
  your-project-exe:
    main:                Main.hs
    source-dirs:         app
    ghc-options:
    - -threaded
    - -rtsopts
    - -with-rtsopts=-N
    dependencies:
-    - your-project
+    - aeson
+    - Spock
+    - text
```

`app/Main.hs`を作ります。

```elm
{-# LANGUAGE DeriveGeneric #-}
{-# LANGUAGE OverloadedStrings #-}

import Web.Spock
import Web.Spock.Config

import Data.Aeson hiding (json)
import Data.Monoid ((<>))
import Data.Text (Text, pack)
import GHC.Generics


data Person = Person
  { name :: Text
  , country :: Text
  , city :: Text
  , salary :: Text
  } deriving (Generic, Show)

instance ToJSON Person

instance FromJSON Person


type Api = SpockM () () () ()

type ApiAction a = SpockAction () () () a

main :: IO ()
main = do
  spockCfg <- defaultSpockCfg () PCNoDatabase ()
  runSpock 8080 (spock spockCfg app)

corsHeader = do
  ctx <- getContext
  setHeader "Access-Control-Allow-Origin" "*"
  pure ctx

app :: Api
app = prehook corsHeader $ do
  get "people" $ do
    json [ Person { name = "Dakota Rice", country = "Niger", city = "Oud-Turnhout", salary = "$36,738" }
         , Person { name = "Minerva Hooper", country = "Curaçao", city = "Sinaai-Waas", salary = "$23,789" }
         , Person { name = "Sage Rodriguez", country = "Netherlands", city = "Baileux", salary = "$56,142" }
         , Person { name = "Philip Chaney", country = "Korea, South", city = "Overland Park", salary = "$38,735" }
         , Person { name = "Doris Greene", country = "Malawi", city = "Feldkirchen in Kärnten", salary = "$63,542" }
         , Person { name = "Mason Porter", country = "Chile", city = "Gloucester", salary = "$78,615" }
         ]
```

ビルドします。

```sh
$ stack build
```

起動します。

```sh
$ stack exec your-project-exe
```

叩いてみます。

```sh
$ curl localhost:8080/people
[{"salary":"$36,738","country":"Niger","name":"Dakota Rice","city":"Oud-Turnhout"},{"salary":"$23,789","country":"Curaçao","name":"Minerva Hooper","city":"Sinaai-Waas"},{"salary":"$56,142","country":"Netherlands","name":"Sage Rodriguez","city":"Baileux"},{"salary":"$38,735","country":"Korea, South","name":"Philip Chaney","city":"Overland Park"},{"salary":"$63,542","country":"Malawi","name":"Doris Greene","city":"Feldkirchen in Kärnten"},{"salary":"$78,615","country":"Chile","name":"Mason Porter","city":"Gloucester"}]
```

意外にさらっと動きますね。では GAE にデプロイしていきます。

`app.yaml`を作ります。

```yaml
runtime: custom
env: flex
```

Linux ビルド用の`Dockerfile.build`を作成します。

```Dockerfile
FROM haskell:8.0.2
WORKDIR /workdir
ADD . .
RUN stack build
EXPOSE 8080
CMD ["stack", "exec", "your-project-exe"]
```

ビルドします。

```sh
$ docker build -t gcr.io/your-project/web-api -f Dockerfile.build .
```

GCP にデプロイします。

```sh
$ gcloud docker -- push gcr.io/your-project/web-api
```

GAE 用の`Dockerfile`を作成します。

```Dockerfile
FROM gcr.io/your-project/web-api
```

アプリケーションをデプロイします。

```sh
$ gcloud app deploy
```

叩いてみます。

```sh
$ curl https://your-project.appspot.com/people
[{"salary":"$36,738","country":"Niger","name":"Dakota Rice","city":"Oud-Turnhout"},{"salary":"$23,789","country":"Curaçao","name":"Minerva Hooper","city":"Sinaai-Waas"},{"salary":"$56,142","country":"Netherlands","name":"Sage Rodriguez","city":"Baileux"},{"salary":"$38,735","country":"Korea, South","name":"Philip Chaney","city":"Overland Park"},{"salary":"$63,542","country":"Malawi","name":"Doris Greene","city":"Feldkirchen in Kärnten"},{"salary":"$78,615","country":"Chile","name":"Mason Porter","city":"Gloucester"}]
```

Web API できあがり。

<br><br>

## Web UI の実装

フロントエンドは [Elm](http://elm-lang.org/) でやります。使いこなすことを目指さなければ、下手すりゃ最近の JS フレームワークより簡単だと思ってます。

<br>

Elm を npm でインストールします。

```sh
$ npm install -g elm
```

必要なパッケージをインストールします。

```sh
$ elm-package install -y elm-lang/http
```

`app/Main.elm`を作ります。

```elm
module Main exposing (..)

import Html exposing (..)
import Html.Attributes exposing (..)
import Http
import Json.Decode as Decode

main : Program Never Model Msg
main =
    Html.program
        { init = ( [], getPeople )
        , view = view
        , update = update
        , subscriptions = \_ -> Sub.none
        }

-- MODEL
type alias Person =
    { name : String
    , country : String
    , city : String
    , salary : String
    }

type alias People =
    List Person

type alias Model =
    People

-- UPDATE
type Msg
    = ListPeople (Result Http.Error People)

update : Msg -> Model -> ( Model, Cmd Msg )
update msg model =
    case msg of
        ListPeople (Ok people) ->
            ( people, Cmd.none )

        ListPeople (Err _) ->
            ( model, Cmd.none )

-- VIEW
view : Model -> Html Msg
view model =
    table [ class "table" ]
        [ thead [ class "text-primary" ]
            [ th []
                [ text "Name" ]
            , th []
                [ text "Country" ]
            , th []
                [ text "City" ]
            , th []
                [ text "Salary" ]
            ]
        , tbody [] (List.map personRow model)
        ]

personRow : Person -> Html Msg
personRow person =
    tr []
        [ td []
            [ text person.name ]
        , td []
            [ text person.country ]
        , td []
            [ text person.city ]
        , td [ class "text-primary" ]
            [ text person.salary ]
        ]

-- HTTP
getPeople : Cmd Msg
getPeople =
    Http.send ListPeople (Http.get "https://your-project.appspot.com/people" decodePeople)

decodePeople : Decode.Decoder People
decodePeople =
    Decode.list
        (Decode.map4 Person
            (Decode.field "name" Decode.string)
            (Decode.field "country" Decode.string)
            (Decode.field "city" Decode.string)
            (Decode.field "salary" Decode.string)
        )
```

開発サーバーを起動します。

```sh
$ elm-reactor
```

[http://localhost:8000/app/Main.elm](http://localhost:8000/app/Main.elm)にアクセスしてみます。

{{< figure src="/images/gae-zero-side-effect/people-elm.png" >}}

よさそうですね。

ビルドします。

```sh
$ elm-make app/Main.elm --yes --output=public/main.js
```

このままだと味気ないので、カリスマ管理画面デザイナー（と私が勝手に呼んでいる）である Creative Tim さんの[Material Dashboard](https://www.creative-tim.com/product/material-dashboard)を使います。

ダウンロードしたら、`examples/table.html`を`public/index.html`に持ってきて、必要な`assets`も持ってきて、テーブル部分を Elm アプリケーションにします。

```diff
-                                <div class="card-content table-responsive">
+                                <div id="people" class="card-content table-responsive">
...

+<script src="main.js"></script>
+<script type="text/javascript">Elm.Main.embed(document.getElementById("people"))</script>
```

簡易 Web サーバーを起動して確認してみます。

```sh
$ serve public
```

[http://localhost:5000](http://localhost:5000)にアクセスしてみます。

{{< figure src="/images/gae-zero-side-effect/people-tim.png" >}}

う〜ん、ティム！

`public/app.yaml`を作ります。

```yaml
runtime: php55
service: web-ui
handlers:
  -
    url: /(.*)
    static_files: \1
    upload: .*
  -
    url: /
    static_files: index.html
    upload: index.html
```

静的ファイルをデプロイします。

```sh
$ cd public
$ gcloud app deploy
```

`https://web-ui-dot-your-project.appspot.com/`にアクセスしてみます。

{{< figure src="/images/gae-zero-side-effect/people-gae.png" >}}

できあがり。

<br><br>

## まとめ

長い上に役に立たないものを書いてしまった。。

haskell イメージからビルドすると何十分もかかるので、やり方考えないと難しいですね。

ちなみに[Elm 入門者向けハンズオン](https://elm-tokyo.connpass.com/event/72654/)というイベントにメンターで参加予定（私に何ができるのだろうか・・）ですので、興味あれば是非いらしてください。

<br>
