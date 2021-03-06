---
title: Reactのtypescript化について
subtitle: ts化以外の例も含む
author: KatusothiOtogawa
date: 2021-12-18
---

## 自己紹介

音川 勝俊 \
フリーのバックエンドエンジニア。DB周り、table設計とか。 \
php, python, goとかのバックエンド側のframeworkも普通にわかります。 \
[twitter: @k_otogawa](https://twitter.com/k_otogawa)

## 人はなぜtypescriptにするか

別に変えなくてもいいじゃん?

## javascriptの難点

1. phpと同じく文字列か数字か曖昧

と文字列の3と数字としての3があったりとか

2. ブラケット記法とかjs独自の記法や考え方多い。

この他にも他の言語でもあるけど、スプレッド構文や{}を多用するので、 \
読みづらかったりします。

3. Java並みにネットの情報が古かったり書き方に問題あるコードがある。

これはjsが悪い訳ではないのですが、 \
プログラミングスクールだとそのコードが知らなくていい書き方だったり、 \
jsだったら、getElementByIdよりもquerySelector使った方が良いだったり、 \
C++だったらヤバイコードだったり \
\nで改行と書かれているけど、メジャーな言語はOSの改行コードの文字を挿入する関数があるから \
それ書いた方がいいとか色々ツッコミどころあったね。

## 結果として発生する問題

1. vscodeのインテリセンスがすべてany...

2. 職人じゃ無いと読めないコードが描きがち

3. なんかよくわからない、けど動く。

4. コメント書かないやつ、全体見れないやつが出てくると*崩壊*する。

割とnodejsは一人親方の所が採用しているパターンが多いから、 \
ここら辺の難解さや暗黙知に対して感覚マヒしている人が多かったりします。 \
typescript採用して、ts周りのライブラリの管理考えたらts採用しなくても \
いいのでは？とりあえず動かすには邪魔と思う人もいる。

reactだったら、関数の引数にpropsという変数を作って、キーを省略して...というのが多い。 \
これ何が嫌かと言うとキー省略するとインテリセンス効かないんですよ。 \
OSSでみんなが使うライブラリをこれ一つのコンポーネントを追っていって、 \
こういう感じになってるからという暗黙知がめちゃくちゃ多い。

ちゃんとドキュメント書いてるならいいけどこの運用方法って全体を理解していること \
前提なんですよ。

だから全体見れない人が増えてきて、jsわからんよねとなってダメになったりします。

## typescript化による恩恵

書いてある通り

1. vscodeなどエディタで型情報が見えるようになる。
2. エディタでインテリセンスが効くようになる。
3. jsに強い人でなくても、割と読めるようになる。

こういう恩恵があるから選択するわけですが、どの手段を取るかですね。

## どの手段を取るか？

手段複数あるのか?と思うじゃ無いですか？

1. 直接typescriptにする。(jsdoc形式などちゃんとしたコメントが無かったり、書いた本人に効かないと難しい。)
2. allowJSを使う(本番のnodejsを少しずつtypescriptに切り替える方法。)
3. // @ts-checkを使う(デメリット無し)

## 直接typescriptにする方法1

1. typescript用のライブラリの追加

```bash
npm install --devDependency typescript @types/node-sass @types/node @types/jest @matetial-ui/types/ @types/react @types/react-dom
```

2. tsconfig.json を取得するためにcreate-react-appでプロジェクト作成

これはなぜかというと、tsconfig.jsonという設定ファイルをガリガリ書くのが難しいから、
新しくプロジェクトを作ったものを使おうという事です。

3. jsconfig.jsonの内容をtsconfig.jsonに移す

jsconfig.jsonはtsconfig.jsonのjs版でnodejsの設定を色々書けるやつです。
jsconfig.jsonとtsconfig.jsonは両立できないので、設定を移して、jsconfig.jsonは消します。

## 直接typescriptにする方法2

4. 各jsファイルのimportが.jsになっていたら修正。

import from 'components/hello.js'
みたいなやつのjsを消す。
jsからtsに帰るから当然

5. ファイルの拡張子をjsからtsに変更する

手打ちで拡張子を変えるのは時間かかるのと危険なので、 \
powershellで一括で拡張子を帰る事をおすすめします。

ここで一度ビルドしてみてください。ファイルの拡張子が間違っているやつがあったら、 \
拡張子のエラーがでるけど、ファイルの拡張子に問題なかったら、型エラーが出ます。

6. linter系のライブラリのtypescript化

linter、っていうのは何かというとこう書いた方が、綺麗だよとか \
コード規約に則ってエラーを出すツールです。

## 直接typescriptにする方法3

あとは*根気*です。 \
釈の都合上でここまでの説明とします。 \
このやり方の続きはgithubにあるので参照してください。

## 直接typescriptにする方法4

メリット

1. 完全にtypescriptにできる。
2. 曖昧なところが他のやり方に比べて生じ辛い。

## 直接typescriptにする方法5

デメリット

1. 開発しながら、typescriptにすべて変えるのが時間的に難しい。
2. jsを書いた人が現場にいなかったら手が止まりやすい。

## allowJSを使う方法1

つい１週間前に知った方法。 \
tsconfig.jsonに下記のようにallowJSのキー追加

## allowJSを使う方法2

メリット

1. javascriptとtypescriptを混ぜることができる。
2. 少しづつ本番のソースコードをtypescript化するときに良いかも。
3. create-react-app --template typescript create-next-app --ts としたときにデフォルトでできるtsconfig.jsonに書かれているのでバグ少なそう

## allowJSを使う方法3

デメリット

1. ちょっとだけなら大丈夫かもだけど、本番でこれやっているとか聞かない。
2. もし不具合があった場合に問題の切り分けが難しそう。
3. そもそもtypescriptからjavascriptも使えるよ？というフラグ。


## ts-checkを使う方法1

// @ts-check \
とjsファイルの一行目に記述してやる。

```shell
# typescriptの方情報だけ下のようにインストールしておく。
npm install @types/react
```

## ts-checkを使う方法2

jsdocでちゃんと型を書いておく。

```js
// 変数の場合
/** @type {number} */
let count = 0;

```

vscodeの方で型のインテリセンスが効くようになるので
tsを使っているのと同じ感覚で開発する事ができます。
そういう方法です。

## ts-checkを使う方法3

もう一つ例を挙げますと、関数リテラルだと
次のようになります。
見切れているからちょっとスクリーン変えますと

これはa+b+c+d=abcdを満たす自然数を求める関数。

## ts-checkを使う方法4

reactの場合は次のようになります。

これは最初のユーザー登録するときのSignupのボタンを押したときに
発動する処理です。
フォームの内容を受け取って、console.logに出力するという処理です。
こういう風にどういうイベントかって書いたら、ここを見ただけで
ああそういうformが来るのか、型が来るのかとわかるようになるので
全体見なくても楽になると思います。

## ts-checkを使う方法5

こうするとvscodeでtypescriptとしてのチェックが入り、文法的にエラーだったら赤く表示されるになる。
ただし、*jsdocで型を書いて*おくこと。こうするとエラーもインテリセンスも効く。

## ts-checkを使う方法6

メリット

1. 開発と並行してtypescript化を進めることができる。
2. クライアントサイドでtsのためだけにwebpackなどを導入する必要がなくなる。

## ts-checkを使う方法7

デメリット

1. typescriptにしかないinterfaceなどのキーワードを使えない。

## 結論

1. 直接typescriptにする。
2. allowJSを使う
3. // @ts-checkを使う

これらを上手に使い分けてください。
以上です。
ご静聴ありがとうございました。
