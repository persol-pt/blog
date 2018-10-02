---
title: "勉強会[Golang入門]"
date: 2018-08-27T14:19:06+09:00
tags: ["勉強会","会社紹介","Golang", "Go言語", "Go", "入門"]
categories: ["里井 嘉真(yoshi-sato)"]
---
皆さんこんにちは。    
パーソルプロセス&テクノロジーのyoshi-satoです。  
今週もテクテク部の勉強会の様子をお送りいたします。  
<br>
※技術的なブログについてはQiitaでメンバーが各々に書き進めています。  
ぜひこちらもご覧ください。  
[Qiita - パーソルプロセス&テクノロジー](https://qiita.com/organizations/persol-pt)  
<br>
今週の発表者は、テクテク部の[ビトル](https://persol-pt.github.io/categories/%E3%83%93%E3%83%88%E3%83%AB/)さんで、発表テーマは「Golang入門」でした。  



<br>

{{< figure src="/images/tech-workshop/20180827/20180824_172844098.jpg" title="" >}}

<br>

---
## Golang入門  

{{< figure src="/images/tech-workshop/20180827/index.JPG" title="" >}}  

---
### ディレクトリ構成について
Golangでは、開発者がコードを1つにまとめたものをworkspaceと呼びます。  

{{< figure src="/images/tech-workshop/20180827/directory.jpg" title="" >}}  

workspaceは、bin, pkg, srcの3つのディレクトリで構成されています。  
binには実行可能なコマンドのバイナリファイルが配置され、  
pkgには、インストールされたパッケージが配置されます。  
srcは各ソースファイルやリポジトリを配置するディレクトリで、リポジトリが異なる場合はディレクトリを分けます。  
Goのツールはパッケージとソースをビルドし、完成したバイナリをpkgとbinそれぞれにインストールします。

<br>
<br>

---

### GOPATHについて  
GOPATHとは、自分のworkspaceのパスを指定するための環境変数です。  
デフォルトでは、Unix系OSであれば$HOME\\goになっており、Windowsであれば%USERPROFILE%\\goになります。  
デフォルトで指定されているパス以外の場所で作業を行いたい場合、GOPATHを変更する必要があります。  
<br>

{{< figure src="/images/tech-workshop/20180827/gopath.JPG" title="" >}}  

<br>

---

### 文法について  
Golangの基本的な文法をご紹介いただきました。  
文法についてビトルさん自身は、Goのチュートリアルである *[A Tour of Go](https://tour.golang.org/welcome/1)* を参考にされたそうです。  

<br>



#### パッケージ
外部のコードを使用したい場合、他の言語と同じように、import文でインポートします。  
パッケージ名は""で囲みますが、複数のパッケージを同時にインポートしたい場合、さらに丸括弧で囲みグループであることを示します。  
標準パッケージでないものは、ビルド時にインストールされます。  

{{< figure src="/images/tech-workshop/20180827/package.JPG" title="" >}}  

<br>

#### 関数  
Golangで関数を定義する際は、Java等と同じように引数と返り値の両方の型を宣言します。  
また、返り値を宣言する際、複数の返り値を宣言しておくことで、2つ以上の値をreturnすることができます。  
さらに、返り値の変数名を定義することも可能で、関数内で変数として利用することもできます。  
関数内でreturnに値を渡していなければ、変数名を定義した返り値が自動的に返されます。  

{{< figure src="/images/tech-workshop/20180827/function.JPG" title="" >}}  

<br>

#### 変数・定数
変数を宣言する際は、varを使います。  
変数の定義時に変数の型を宣言する必要がありますが、定義と同時に初期値を代入していれば、  
初期値から自動的に型が判断されます。  
関数の中ではvarの省略形として:=を使うこともできますが、関数の外ではvar, func等のキーワードで始まる宣言しか  
利用できません。  
定数は、constで宣言します。定数宣言の省略形はありません。

{{< figure src="/images/tech-workshop/20180827/varandconst.JPG" title="" >}}  

<br>

#### 型
変数、定数を宣言する際に初期値を代入せずに、型宣言のみを行った場合、型によってそれぞれの初期値が  
自動的に代入されます。  
初期値は、  
・int,float等の数字型の場合は0  
・bool型の場合はfalse  
・string型の場合は""(空文字)  
になります。  

{{< figure src="/images/tech-workshop/20180827/type.JPG" title="" >}}  

<br>

#### for文
for文は他の言語とあまり変わらず、  
```
for 初期化ステートメント; 条件式; 後処理ステートメント {
  ...
  処理
  ...
}
```
と記述します。  
しかし、Golangには *while文が存在しない* ので、while文のような使い方をする場合、  
初期化ステートメントと後処理ステートメントを省略し条件式のみを指定します。  
条件式も省略しfor{...}としてしまうと無限ループになります。  
for文やif文の条件式で丸括弧を省略して書くことができるので、条件式をより簡略に書くことができますが、  
Golangではワンライナーで{}を省略したり、三項演算子を使用したりすることはできないので注意が必要です。  

{{< figure src="/images/tech-workshop/20180827/forstatement.JPG" title="" >}}  

<br>

#### if文  
Golangのif文では、条件文の直前に変数を宣言することができます。  
条件文の直前で宣言した変数は、if, else文内のスコープ限定で利用することができるようになります。  

{{< figure src="/images/tech-workshop/20180827/ifstatementb.JPG" title="" >}}  

<br>

### 次回  

{{< figure src="/images/tech-workshop/20180827/zikai.JPG" title="" >}}  

<br>
Golangの基礎知識を他の言語と比較しながらわかりやすくご説明いただきました。  
次回も楽しみにしています。  
[ビトル](https://persol-pt.github.io/categories/%E3%83%93%E3%83%88%E3%83%AB/)さん、ありがとうございました。  
<br>
