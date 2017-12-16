---
title: "DockerだけでRails、特にWebpackerはハマった"
date: 2017-12-16T16:50:12+09:00
tags: ["Rails", "Webpacker", "Docker"]
categories: ["boiyaa"]
---

すいません遅くなりました。。

[Ruby on Rails Advent Calendar 2017](https://qiita.com/advent-calendar/2017/ruby_on_rails) 14 日目の記事です。

<br><br>

ある日、

<br>

### 「docker run で rails new とか bundle install したい」

<br>

と、知り合いのそこそこ可愛い女の子が言ってきました。

<br>

同僚の作った[おじさん LINE ボット](https://persol-pt.github.io/posts/line-ojisan-bot/)の教師データになるほどのオジサンである私は、

誠意（下心）を持って取り組んだのであります。

しかし、やってみたら色々ハマったので共有します。

<br>

ちなみに最終的な状態は[こちらのリポジトリ](https://github.com/boiyaa/webpacker-in-docker)にありますので、ここからの長々とした過程を飛ばしたい方はこちらをご覧ください。

<br><br><br>

## Rails の Docker イメージを作る

一般的に Rails アプリケーションと Docker を組み合わせる場合、[Quickstart: Compose and Rails](https://docs.docker.com/compose/rails/)にあるように、「Rails アプリケーションを起動する Docker イメージを作る」という使い方が主流のようです。

<br>

`docker run`で rails コマンドを使用したいのであれば、Rails とその依存するものがインストールされた Docker イメージが必要ですが、

[Rails 公式 Docker イメージ](https://hub.docker.com/_/rails/)は非推奨でもうメンテされていないので、

最初はこの Dockerfile を参考にして作りました。

<br>

```
FROM ruby

RUN apt-get update && apt-get install -y nodejs --no-install-recommends && rm -rf /var/lib/apt/lists/*

RUN apt-get update && apt-get install -y mysql-client postgresql-client sqlite3 --no-install-recommends && rm -rf /var/lib/apt/lists/*

RUN gem install rails
```

しかしこれで作ったイメージで`bundle install --path vendor/bundle`しても`.bundle`ファイルが生成されなくて上手く動きません。

ので`BUNDLE_PATH`を設定しました。

<br>

```
FROM ruby

RUN apt-get update && apt-get install -y nodejs --no-install-recommends && rm -rf /var/lib/apt/lists/*

RUN apt-get update && apt-get install -y mysql-client postgresql-client sqlite3 --no-install-recommends && rm -rf /var/lib/apt/lists/*

RUN gem install rails

WORKDIR /workdir

ENV BUNDLE_PATH /workdir/vendor/bundle/ruby/2.4.0
```

では Docker イメージをビルドします。

```sh
$ docker build -t myrails .
```

最新の Rails （執筆時点では 5.1.4） の Docker イメージができます。

ホストとコンテナを同期してホスト側に生成されたファイルが来るようにしつつ、 `rails new`、`bundle install`、`rails server`します。

```sh
$ docker run -it --rm -v $(pwd):/workdir myrails rails new myproject --skip-bundle
$ cd myproject
$ docker run -it --rm -v $(pwd):/workdir myrails bundle install --path vendor/bundle
$ docker run -it --rm -v $(pwd):/workdir -p 3000:3000 myrails rails server -b 0.0.0.0
=> Booting Puma
...
* Listening on tcp://0.0.0.0:3000
Use Ctrl-C to stop
```

[http://localhost:3000/](http://localhost:3000/)で確認できるところまでいけました。

<br><br><br>

## Webpacker が動作する Docker イメージを作る

せっかく Rails5.1 なら Webpacker が使いたいところですが、

先ほどのイメージの Node.js は 0.10.29、

Webpacker に必要な Node.js は 6.0.0 以上、さらに Yarn 0.25.2 以上が必要です。

なので、先ほどの Dockerfile の Node.js のインストール部分だけ変更して対応したいところです。

<br>

```
FROM ruby

# この部分を
# RUN apt-get update && apt-get install -y nodejs --no-install-recommends && rm -rf /var/lib/apt/lists/*
# こう変える
RUN curl -sL https://deb.nodesource.com/setup_8.x | bash - &&\
    curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - &&\
    echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list &&\
    apt-get update && apt-get install -y nodejs yarn --no-install-recommends && rm -rf /var/lib/apt/lists/*

RUN apt-get update && apt-get install -y mysql-client postgresql-client sqlite3 --no-install-recommends && rm -rf /var/lib/apt/lists/*

RUN gem install rails

WORKDIR /workdir

ENV BUNDLE_PATH /workdir/vendor/bundle/ruby/2.4.0
```

が、このイメージでプロジェクトを作って`webpacker:install`しても、実行に必要な webpack が入りません。

<br>

```sh
$ docker run -it --rm -v $(pwd):/workdir myrails bin/webpack
docker: Error response from daemon: oci runtime error: container_linux.go:265: starting container process caused "exec: \"bin/webpack\": stat bin/webpack: no such file or directory".
```

Ruby のバージョン変えたり、Debian イメージから作り直したりしても解決しないので、やけくそで Fedora でイメージ作ってみたら、入りました。

<br>

```
FROM fedora

RUN dnf -y update && \
    dnf -y group install "C Development Tools and Libraries" && \
    curl --silent --location https://rpm.nodesource.com/setup_8.x | bash - && \
    curl --silent https://dl.yarnpkg.com/rpm/yarn.repo -o /etc/yum.repos.d/yarn.repo && \
    dnf -y install \
    redhat-rpm-config \
    wget \
    zlib-devel \
    sqlite-devel \
    mysql-devel \
    postgresql-devel \
    ruby-devel \
    ruby \
    rubygem-json \
    nodejs \
    yarn && \
    dnf clean all && \
    gem install bundler rails

WORKDIR /workdir
EXPOSE 3000
```

こんな感じで、もう一回作り直して、

```sh
$ docker build -t myrails .
$ docker run -it --rm -v $(pwd):/workdir myrails rails new myproject --skip-bundle
$ cd myproject
$ echo "gem 'webpacker'" >> Gemfile
$ docker run -it --rm -v $(pwd):/workdir myrails bundle install --path vendor/bundle
$ docker run -it --rm -v $(pwd):/workdir myrails rails webpacker:install
$ docker run -it --rm -v $(pwd):/workdir myrails rails webpacker:install:elm
$ docker run -it --rm -v $(pwd):/workdir myrails rails bin/webpack
...
   [8] ./app/javascript/packs/application.js 515 bytes {1} [built]
   [9] ./app/javascript/packs/hello_react.jsx 739 bytes {0} [built]
    + 25 hidden modules
```

できました。

<br><br><br>

## DB も Docker でたてて Rails から使う

そのような場合は`docker-compose.yml`で構成するといいでしょう。

<br>

```yml
version: "3"
services:
  myproject:
    container_name: myproject
    image: myrails
    command: rails server -b 0.0.0.0
    environment:
      # MySQLを使う場合
      DATABASE_URL: "mysql2://root:11111111@mysql/myproject"
      # PostgreSQLを使う場合
      # DATABASE_URL: "postgresql://postgres:11111111@postgres/myproject"
    volumes:
       - "./myproject:/workdir"
    ports:
      - "3000:3000"
  # MySQLを使う場合
  mysql:
    container_name: mysql
    image: "mysql:5.6"
    environment:
      MYSQL_ROOT_PASSWORD: 11111111
      MYSQL_DATABASE: myproject
    volumes:
       - /var/lib/mysql/data
  # PostgreSQLを使う場合
  # postgres:
  #   container_name: postgres
  #   image: "postgres:9.6"
  #   environment:
  #     POSTGRES_PASSWORD: 11111111
  #     POSTGRES_USER: postgres
  #     POSTGRES_DB: myproject
  #     PGDATA: /var/lib/postgresql/data/pgdata
  #   volumes:
  #      - /var/lib/postgresql/data/pgdata
```

```sh
$ docker-compose up -d
$ docker-compose ps
  Name                Command             State           Ports
------------------------------------------------------------------------
myproject   rails server -b 0.0.0.0       Up      0.0.0.0:3000->3000/tcp
mysql       docker-entrypoint.sh mysqld   Up      3306/tcp
```

<br><br>

## アプリケーションの Docker イメージを作る

Rails プロジェクトディレクトリに以下のような Dockerfile を作り、

```
FROM myrails

ADD . /workdir

CMD rails server -b 0.0.0.0
```

ビルドするといいでしょう。

```sh
$ docker build -t myapp .
```

<br><br>

## まとめ

なんとかできなくはない状態にはなりましたが、とにかく遅いです。

<br><br>
