---
title: Reactのtypescript化について
subtitle: サブタイトル名
author: KatusothiOtogawa
date: 2021-12-18
---

## 自己紹介

音川 勝俊 \
フリーのバックエンドエンジニア。DB周り、table設計とか。 \
統計、分析システムのDB検索を1hから6mに短縮する男。 \
php, python, goとかのバックエンド側のframeworkも普通にわかります。 \
[twitter: @k_otogawa](https://twitter.com/k_otogawa)

## 人はなぜtypescriptにするか

別に変えなくてもいいじゃん?

## javascriptの難点

1. phpと同じく文字列か数字か曖昧
2. ブラケット記法とかjs独自の記法や考え方多い。
3. Java並みにネットの情報が古かったり書き方に問題あるコードがある。

## 結果として発生する問題

1. vscodeのインテリセンスがすべてany...
2. 職人じゃないと読めないコード書きがち
3. なんかよくわからない, けど*動く*。
4. コメント書かないやつ、全体見れないやつが出てくると*崩壊*する。

## typescript化による恩恵

1. vscodeなどエディタで型情報が見えるようになる。
2. エディタでインテリセンスが効くようになる。
3. jsに強い人でなくても、割と読めるようになる。

## どの手段を取るか？

1. 直接typescriptにする。(jsdoc形式などちゃんとしたコメントが無かったり、書いた本人に効かないと難しい。)
2. allowJSを使う(本番のnodejsを少しずつtypescriptに切り替える方法。)
3. // @ts-checkを使う(デメリット無し)

## 直接typescriptにする方法1

1. typescript用のライブラリの追加(linter除く)

```shell
npm install --save-dev @types/react
```

2. tsconfig.json を取得するためにcreate-react-appでプロジェクト作成

3. jsconfig.jsonの内容をtsconfig.jsonに移す

## 直接typescriptにする方法2

4. 各jsファイルのimportが.jsになっていたら修正。
5. ファイルの拡張子をjsからtsに変更する
6. linter系のライブラリのtypescript化

## 直接typescriptにする方法3

あとは*根気*です。

## 直接typescriptにする方法4

メリット

1. 完全にtypescriptにできる。
2. 曖昧なところが他のやり方に比べて生じ辛い。

## 直接typescriptにする方法5

デメリット

1. 開発しながら、typescriptにすべて変えるのが時間的に難しい。
2. jsを書いた人が現場にいなかったら手が止まりやすい。

## allowJSを使う方法1

tsconfig.jsonに下記のようにallowJSのキー追加

```json
{
  "compilerOptions": {
    "target": "es5",
    "baseUrl": "src",
    "lib": [
        ...
    ],
    "allowJs": true,
    ...
}
```

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

## ts-checkを使う方法3

関数リテラル

```js
/** 
 * @type {() => number[][]} 戻り値は答えの組み合わせを配列で返す。インデックスが若い方からa, b, c, dとする。
 * @description a+b+c+d=abcdを満たす自然数を求める関数。
*/
const abcdQuetion = () => {
  // a*b*c*d=a+b+c+dを変形すると
  // 1<=abcd=a+b+c+d≦4d=d+d+d+d
  // よって下記のことが成り立つのでループで探せば良い。
  // 1 <=abc<=4 (dも同じことが言える。)
  /** 
   * @type {number[][]} 
   * @description 問を満たす自然数の組み合わせを入れる変数。インデックスが若い方からa, b, c, dと値を入れる。
  */
  const result = [];
  for (let a = 1;a <= 4;a++) {
    for (let b = 1;b <= 4;b++) {
      for (let c = 1;c <= 4;c++) {
        for (let d = 1;d <= 4;d++) {
          // 組み合わせを満たす自然数が見つかったら、答えの配列に追加する。
          if (a + b + c + d == a * b * c * d) {
            result.push([a, b, c, d]);
          }
        }
      }
    }
  }
  return result;
}
```

## ts-checkを使う方法4

reactの場合は下のような感じ。

```js
  /** 
   * @type {(event: React.FormEvent<HTMLFormElement>) => void} 
   * @description サインアップのボタンを押したらこの関数が実行される。
  */
  const handleSubmit = (event) => {
    event.preventDefault();
    const data = new FormData(event.currentTarget);
    // eslint-disable-next-line no-console
    console.log({
      email: data.get('email'),
      password: data.get('password'),
    });
  };
```

## ts-checkを使う方法5

こうするとvscodeでtypescriptとしてのチェックが入り、文法的にエラーだったら赤く表示されるになる。
ただし、*jsdocで型を書いて*おくこと。

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
