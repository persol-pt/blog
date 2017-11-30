# PPT エンジニアブログ

## 記事を書く準備

### Hugo をインストール

```sh
brew install hugo
```

### このリポジトリをフォーク

### フォークしたリポジトリをクローン

git submodules 使ってるので recursive オプション付けてね

```sh
git clone --recursive git@github.com:[user name]/blog.git
```

##  記事を書く

### 記事を生成する

```sh
hugo new posts/[kiji-no-title].md
```

※ tags, categories を設定しましょう

### 生成した md を編集する

### サーバーを起動して確認する

```sh
hugo server -D
```

※ この時点では後述のデプロイ作業をしても公開はされない

### 記事を公開設定する

```sh
hugo undraft content/posts/[kiji-no-title].md
```

### コミットしてプルリクする

# ここから先管理者

## 記事をデプロイする

### 手動でデプロイする

```sh
./deploy.sh "Add new article"
```
