# PPTエンジニアブログ

## 記事を書く準備

### Hugoをインストール

```sh
brew install hugo
```


## 記事を書く

### 記事を生成する

```sh
hugo new posts/[kiji-no-title].md
```

### 生成したmdを編集する

### サーバーを起動して確認する

```sh
hugo server -D
```

※この時点では後述のデプロイ作業をしても公開はされない


## 記事をデプロイする

### 対象記事を公開設定する

```sh
hugo undraft content/posts/[kiji-no-title].md
```

### 手動でデプロイする

```sh
./deploy.sh "Add new article"
```
