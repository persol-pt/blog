---
title: "react-sketchappを使った開発ワークフロー"
date: 2017-12-10T23:10:32+09:00
tags: ["React", "Sketch"]
categories: ["boiyaa"]
---

<br>

[React #1 Advent Calendar 2017](https://qiita.com/advent-calendar/2017/react) 10 日目の記事です。

<br><br>

## react-sketchapp

[react-sketchapp](https://github.com/airbnb/react-sketchapp)とは、デザインツールの [Sketch](https://www.sketchapp.com/) 用にビルドできる React コンポーネントです。

README からの引用ですが、こんな概要です。

* デザインシステムの管理 - react-sketchapp は、[Airbnb’s design system](http://airbnb.design/building-a-visual-language/)用に作られました。大規模なデザインシステムで Sketch のアセットを管理する最も簡単な方法です。
* デザインに実際のコンポーネントを使用 - コードで React コンポーネントとして実装し、Sketch に描画します。
* 実際のデータを使用したデザイン - データを使用したデザインは重要ですが困難です。 react-sketchapp は Sketch ファイルに実際のデータを取得して埋め込むのを簡単にします。
* Sketch 上に新しいツールを作る - Sketch をカスタムデザインツールのキャンバスとして使用する最も簡単な方法

実際に使用した画面の雰囲気は[Painting with Code](https://airbnb.design/painting-with-code/)をご覧ください。

<br><br>

## react-sketchapp を導入するメリット

一般的にプロダクトの開発ワークフローは、例えばデザイナーからデザインファイルを受け取り、プログラマーがシステムに組み込む、という感じになりますが、そこで react-sketchapp を使ってデザイナー側で React コンポーネントを実装してもらえると、プログラマーは HOC などを利用してロジックを被せるように作っていけば、デザイナーにデザインの修正や改善を直接プロダクトコードに反映してもらえるようになって、手間やミスを削減できます。

<br>

デザイナーからすると React コンポーネントを作るというとハードルが高そうに聞こえるかもしれませんが、ステートレスコンポーネントなら HTML に結構似ているので、マークアップできる方であれば学習コストはそれほどない（と思っています）。

<br><br>

## はじめてみる

はじめるにあたり以下の環境が必要です。

* node.js 4+ ( 筆者は 8.9.3 でやりました。)
* Sketch 43+ ( ドキュメントには書いてませんが、version 48 未満でないとビルドに失敗します )

こちらのリポジトリ [boiyaa/react-sketchapp-example](https://github.com/boiyaa/react-sketchapp-example)を参考にしてください。

<br><br>

## Web の React との違い

Web の React では、JSX には HTML とほぼ同じ名前のコンポーネント（div, p など）を使用していましたが、

Sketch の React では、react-sketchapp の提供する数種類のコンポーネント（View, Text など）を使用しないとビルドできません。

React Native も react-native の提供するコンポーネント（View, Text など）を使用しないとビルドできないのと同じで、

あくまで React の実装方法を利用できるというだけで、HTML で Sketch デザインを表現できるものではありません。

<br><br>

## react-sketchapp で Web サイトを作る時は依存関係が難しい

まず下記パッケージが最小構成になります。

```js
{
  "dependencies": {
    "prop-types": "^15.6.0",
    "react": "^15.6.2",
    "react-dom": "^15.6.2",
    "react-primitives": "^0.4.4",
    "react-sketchapp": "^0.12.1"
  },
  "devDependencies": {
    "@skpm/builder": "0.1.4",
    "react-test-renderer": "^15.6.2"
  }
}
```

残念ながら React 16 には非対応です。

上記で、Web なら Web の、Sketch なら Sketch の、コンポーネントを実装しないといけないと書きましたが、

[react-primitives](https://github.com/lelandrichardson/react-primitives)の提供するコンポーネントを使うと、1 つのコンポーネント実装で Web や Sketch や Native など各プラットフォームへビルドできるようになります。

これはとても便利なのですが、react-primitives が Web ビルド用に依存している react-native-web のバージョンが古いため、React 16 が使えません。PR 出ているので、近いうち対応すると思います。[Fix React 16 for Web. #95](https://github.com/lelandrichardson/react-primitives/pull/95)

<br>

また、Sketch ビルドに使用する @skpm/builder は最新 0.2 系で、react-sketchapp の examples でもそれが使われているのですが、

筆者の環境では、0.1.4 でないとビルドに失敗します。[^1]

<br>

[^1]: ちなみに OS のせいかなと思い、先ほど High Sierra にアップグレードしたら、Sketch に反映されなくなってしまいました。。 [basic-setup doesn't work on macOS High Sierra #141](https://github.com/airbnb/react-sketchapp/issues/141) とか [Error while running `npm run render` for `basic-setup` example #155](https://github.com/airbnb/react-sketchapp/issues/155) 見ても解決できなくて、今ハマっています。

<br><br>

## コンポーネントを作る

コンポーネントファイルの基本的な構成は、

* 使用するモジュール
* スタイル
* React コンポーネント

となります。Sample.js というコンポーネントファイルを作ってみます。

```js
// 使用するモジュール
import React from "react";
import { Text, View } from "react-primitives";
import { colors, fontSizes } from "./designSystem";

// スタイル
const styles = {
  root: {
    backgroundColor: colors.white
  },
  body: {
    fontSize: fontSizes.body
  }
};

// React コンポーネント
export default ({ message }) => (
  <View name="Sample" style={styles.root}>
    <Text style={styles.body}>{message}</Text>
  </View>
);
```

上記では色やタイポグラフィなどを designSystem.js という別ファイルに分けています。 SASS などと同じ感じです。

```js
export const colors = {
  black: "#000000",
  white: "#ffffff"
};

export const fontSizes = {
  body: 14
};
```

Sketch 用のエンドポイントファイルはこんな感じです。

```js
import React from "react";
import { render, Artboard } from "react-sketchapp";
import Sample from "./Sample";

const Sketch = () => (
  <Artboard>
    <Sample message="hoge fuga piyo" />
  </Artboard>
);

export default () => {
  render(<Sketch />, context.document.currentPage());
};
```

これだけ書けば、`skpm-build`コマンドで Sketch 上に描画されます。

オプションを付ければ、保存の度に反映されるので、インブラウザデザインのノリで進める事ができます。

<br><br>

## まとめ

やっぱり純粋なマークアップに比べると難しいので、デザイナーにはハードル高いかもしれません。

<br>

例えばデザイン初稿はデザインファイルからプログラマーが Sketch コンポーネントで実装して、

以降の細かい修正はデザイナーにお任せする、みたいな感じで取り入れてみようか、みたいな話をうちのチームではしています。

<br>

ただ、今のところインブラウザデザインを Sketch でできるようにしたものという印象しかなく、

[Storybook](https://github.com/storybooks/storybook) とかを使ってインブラウザデザインする以上のメリットがどこにあるのか、実はよくわかっていません。

<br>
