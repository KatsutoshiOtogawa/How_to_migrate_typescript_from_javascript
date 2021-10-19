# How_to_migrate_typescript_from_javascript

react を後からtypescript化する方法です。

ブランチを分けてやってみてください。

はっきりいってかなり*根気が必要*です。

## エディタについて、

vscodeがtypescriptのインテリセンス、またlintと相性がいいので、 \
*vscodeを使うべき*。 vimはとてもじゃないが、おすすめできない。vscodeにvimのプラグインを入れて我慢してください。

使うことが推奨される拡張を並べておく。
これらをvscodeの拡張としてインストールしておくこと。

- dsznajder.es7-react-js-snippets
- ms-vscode.vscode-typescript-next
- dbaeumer.vscode-eslint
- esbenp.prettier-vscode

vimmerなら下記のものもインストールする。なお、この拡張にはまだ使えるようになっていないvimコマンドがあるので注意すること。(:m, :g などのコマンドとか。)

- vscodevim.vim

## パッケージの追加

typescript, react, react-router, react-dom, jest, material-uiなどの@types追加(この時点では簡単のため、lint系のライブラリーを追加しない。build時にエラーが出るため。)

```shell
# 必要な@typesなどは調べること。
yarn add --dev typescript @types/node-sass @types/node @types/jest @material-ui/types @types/react @types/react-dom
```

## 設定ファイル作成のため、create-react-appでプロジェクト作成

設定ファイル作成のため、他にプロジェクトを作成する。 \
tsconfig.jsの設定は考えて書くのは難しく、シンタックスエラーの元になるので、このやり方推奨。

```shell
create-react-app dummy_project --template typescript
```

生成されたtsconfig.jsonをコピーして
新しくtypescript化したいプロジェクトに突っ込んでおく。

## jsconfig.jsをtsconfig.jsに置き換え

もし、typescript化したいプロジェクトでjsconfig.jsonファイルを使っていた場合、 \
typescriptではjsconfig.jsを使えないため、設定をtsconfig.jsに移す。

jsconfig.json

```json
{
  "compilerOptions": {
    "baseUrl": "src",
    "paths": {
      "*": ["src/*"]
    }
  }
}
```

tsconfig.json

```json
{
  "compilerOptions": {
    "target": "es5",
    "baseUrl": "src",
    "lib": [
      "dom",
      "dom.iterable",
      "esnext"
    ],
    "allowJs": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "noFallthroughCasesInSwitch": true,
    "module": "esnext",
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx"
  },
  "include": [
    "src"
  ]
}
```

対応するキーに設定を入れたらそれで良い。 \
pathsについてはincludeで対応できているので、キーを追加する必要はない。 \
jsconfig.jsonは削除しておくこと。残しておくとビルドに失敗する。 \
(どうしても残したいなら.txtみたいに拡張子変えればいいけど、git使ってるし必要ないよね。)

## ライブラリーや設定ファイルをいじったしビルド

ここでtsconfig.jsonなどに問題無いかチェックしたいのでビルドしてみてください \
うまくいっていればエラーは発生しません。

```shell
yarn build
```

## 各jsファイルのimportを修正

みたいにfrom部分が.jsとjsの拡張子になっているものの拡張子を無くす。 \
javascriptからtypescriptだから当然。

エディタの置換機能を使うと楽になるが、他の文字列を巻き込みやすいので注意。

```js
// import Hello from 'components/hello.js'
import Hello from 'components/hello'
```

## 拡張子をtypescriptに変更する

.jsを.tsに変更することによりtypescriptに変更します。 \
この処理はbashでも書けるが、pwshのほうが書きやすく安全なスクリプトになるので \
pwsh推奨。linux, macの人はpwshをインストールして使ってみよう。

下記はsrcフォルダ以下にjsファイルがあると想定した時の書き方。 \
jsx->tsxも同じ要領で書く。

```powershell
# -WhatIfにより、想定されたファイルがリネーム
Get-ChildItem -Recurse src/ |
Where-Object {$_.Name -match ".js$"} | 
Rename-Item -NewName { $_ -replace "\.js$", ".ts" } -WhatIf

# ファイルの拡張子を変更。
Get-ChildItem -Recurse src/ |
Where-Object {$_.Name -match ".js$"} | 
Rename-Item -NewName { $_ -replace "\.js$", ".ts" }
```

ここでもう一度ビルドしてみよう。
型がわからないなどのエラーが出ると思う。 \
ここら辺で新しく作ったブランチに一度コミット、pushしてみては?  \
ここから後戻りが出ると大変です。

## linter系のライブラリーのtypescript化

linter系のライブラリーを追加する

```shell
yarn add @typescript-eslint/eslint-plugin @typescript-eslint/parser
```

.eslintrc.jsを下記のように変更。
変更追加が必要なもののみ書いています。parserがbabelからtypescript-eslintに変わるのは重要です。

```js

  // parser: "babel-eslint",
  parser: "@typescript-eslint/parser",
  plugins: ["react", "@typescript-eslint"],
  rules: {
    "no-unused-vars": "off",
    "@typescript-eslint/no-unused-vars": "error",
  },
```

.babelrc.jsは削除しておく。 \
typescriptでは特に利用が無い限り不要。

## lintも変更したしビルド

もう一度ビルドしてみます。 \
ライブラリ自体のエラーが出ないなら大丈夫です。 \
これよりあとは本格的にソースコードを変更します。 \
ここら辺で一度コミット推奨。

## ソースコードを変更

これ以降は

- ビルド時にどんなエラーがでるか
- vscodeに赤文字でlintとして何が問題か

など考えながら修正していきます。js,tsどちらの知識も必要です。

### makeStylesでエラー

スタイルの定義が書かれているオブジェクトに問題があります。

```js

const HelloStyle = {
    root: {
      flexGrow: 1,
      marginBottom: "20px",
    }
};

const useStyles = makeStyles(HelloStyle);

//
```

みたいにオブジェクトでstyleを指定しているなら、 \
*createStyles*を使ってください。

```ts
import { createStyles } from "@material-ui/core/styles";

const HelloStyle = createStyles({
    root: {
      flexGrow: 1,
      marginBottom: "20px",
    }
});

const useStyles = makeStyles(HelloStyle);

//
```

themeを使っている場合は、*Theme型である*と明示しないとエラーになります。 \
typescriptだから当然ですね

```ts
import { createStyles, Theme } from "@material-ui/core/styles";

const HelloStyle = (theme: Theme) =>
createStyles({
    root: {
    flexGrow: 1,
    marginBottom: "20px",
    }

    /// ... これ以降に引数のthemeを使っているとする。
});

const useStyles = makeStyles(HelloStyle);

// ...
```

#### createStylesと変更することで発生するエラー

#### fontWeight

createStylesと変更するとキーにエラーが発生する場合があります。 \
例えば、下をみてください。

```ts
import { createStyles } from "@material-ui/core/styles";

const HelloStyle = createStyles({
    root: {
      flexGrow: 1,
      marginBottom: "20px",
      fontWeight: "500",
    }
});

const useStyles = makeStyles(HelloStyle);
```

恐らく、fontWeightでエラーが発生していると思います。 \
typescriptにしたのでstring型は許されない。ということなので、 \
エラーが発生したのです。単純に""をとってnumberにしたらエラーでなくなります。

class名ごとに別のファイルにスタイルを分けて書いていて、 \
元のファイルをいじりたくないなら下記のように書く必要がある。

変更による影響範囲がわからないなら、こう書くしか無い。

```ts
import {
  title
} from "style/title";
import { createStyles } from "@material-ui/core/styles";
const HelloStyle = createStyles({
  // 本来ならクラス名を表すオブジェクトであるtitleをそのまま書いたらいいだけ
  // のはずだが、fontWeightがstring型で入っているので、parseIntにより変更する。
  // title,
  title: {
    ...title,
    fontWeight: parseInt(title.fontWeight),
  },
```

#### プロパティが特定の文字列のみ受け付けるがlintが理解してくれない

as(キャスト) により*特定の文字列である*ことを明示する。

```ts
import {
  title
} from "style/title";
import { createStyles } from "@material-ui/core/styles";
const HelloStyle = createStyles({
  // 本来ならクラス名を表すオブジェクトであるtitleをそのまま書いたらいいだけ
  // のはずだが、lintがpositionが特定の文字列か判断できないため、型を明示的にキャストする。
  // title,
  title: {
    ...title,
    position: title.position as
      | "static"
      | "relative"
      | "absolute"
      | "sticky"
      | "fixed",
    fontWeight: parseInt(title.fontWeight),
  },
```

### component関数の引数propsの型がわからないためエラー

これはソースコードをじっくりみる or \
書いた人に聞かないと判別できない。 \
できるだけ書いた人に何を想定していたか聞くこと。

interfaceを使って引数の型を定義していくことになるが、 \
interfaceのプロパティは下記のようにPropTypesで受け付ける型を参考にすることとよい。 \
なければ、ソースコードからプログラマが判断するしかない。

下記を参考にして欲しい。

```ts
import PropTypes from "prop-types";

// PropTypesを参考にプロパティの型を考える。
interface Props {
  active: number | number[];
  color?:
    | "warning"
    | "danger"
    | "success"
    | "info";
}

export default function HelloComponent(props: Props) {
// props
//
}

HelloComponent.propTypes = {
  // index of the default active collapse
  active: PropTypes.oneOfType([
    PropTypes.number,
    PropTypes.arrayOf(PropTypes.number),
  ]).isRequired,
  color: PropTypes.oneOf([
    "warning",
    "danger",
    "success",
    "info",
  ]),
};
```

#### interfaceのextendかコンポジットか

typescriptはinterfaceのextendもコンポジット(interfaceのプロパティにinterfaceを持たせるという形で)も両方できる。 \

例えば、CustomButtonというコンポーネントとどちらの方がいいか?
結論から言うと変更が少ない方が良い。大抵の場合はコンポジットの方が変更が少なく、変更に強い。

extendsを使うと下のような問題が起きる可能性がある。
このような変更が起きると修正箇所が増えるので気をつけること。

```ts

import { Props as CustomButtonProps } from 'components/CustomButton'
import { Props as CustomDiverProps } from 'components/CustomDiver'

interface Props extends CustomButtonProps, CustomDiverProps {
  active: number | number[];
  
}

export default function HelloComponent(props: Props) {
// props
//
}
```

このとき、CustomButtonProps, CustomDiverPropsそれぞれに同じ名前のプロパティがあり、 \
なおかつ型が違う場合は型不一致でエラーになる。 \
よくあるのが、colorというよくある名前のプロパティを継承元のPropsが持っている場合。 \
上の例だとCustomButtonProps, CustomDiverPropsともにColorというプロパティを持っており、 \
かつ型が違う場合はColorの型を揃える必要があるため、修正があった場合の影響範囲が大きくなる。 \
片一方が普通の型でもう片一方がオプショナル型(color?:)でもエラーになる。

これを避けるためにおすすめなのがコンポジットだ。下を見て欲しい。

```ts

import { Props as CustomButtonProps } from 'components/CustomButton'
import { Props as CustomDiverProps } from 'components/CustomDiver'

interface Props {
  active: number | number[];
  CustomButtonProps: CustomButtonProps;
  CustomDiverProps: CustomDiverProps;
  
}

export default function HelloComponent(props: Props) {
// props
//
}
```

もし、特定のプロパティだけ使わないなら下のように
もしくは下のように特定のプロパティだけtypeで型定義をするという方法もある。

```ts

import { Props as CustomButtonProps } from 'components/CustomButton'
import { Props as CustomDiverProps } from 'components/CustomDiver'

// interfaceのプロパティの定義へのアクセスは.(ドット)でなくて、["プロパティ名"]とする必要がある。
type CusomButtonPropsColor = CusomButtonProps["color"];
type CustomDiverPropsColor = CustomDiverProps["color"];

interface Props {
  active: number | number[];
  CustomButtonPropsColor: CusomButtonPropsColor;
  CustomDiverProps: CustomeDiverPropsColor;
  
}

export default function HelloComponent(props: Props) {
// props
//
}
```

#### propsの残余引数

javascriptでは下記のようにすると、変数からオブジェクトのキーを取り出して、 \
新たに変数を割り当てることができる。

```js
const point = {
  x: 34,
  y: 12,
  z: 100
};

const { 
  x,
  ...rest
  } = point;

// restの値は下のようになっている。
// rest = {
//   y: 12
//   z: 100
// }

```

上の場合は変数xにpointのキーxの値が入り、それ以外のキーは全てrestに入る。 \
reactでどういうときにこれを使うかというと、下のようにPropsのプロパティをさらに、 \
他のコンポーネントに渡す時だ。

この場合は下のようにrestパラメーターでちゃんとキーを受けれるように、 \
interfaceのプロパティに*restパラメーターとして想定しているキー*を書く必要がある。

```ts
import Grid, { GridSize } from "@material-ui/core/Grid";

interface Props {
  x: number;
  color?:
    | "danger"
    | "success"
    | "info";
  xs?: GridSize;
}

export default function HelloComponent(props: Props) {

  const { 
    x,
    color,
    ...rest
    } = props;
  return (
    <Grid item {...rest} >
      {children}
    </Grid>
  );
// props
//
}
```

#### Propsのプロパティがオブジェクト型でそのキーがあるかどうかで動作を分けている

javascriptは動的にキーを簡単に追加できるので、下記のようなコードもかけます。

```js

export default function HelloComponent(props) {
// props
//
}

// nameは真偽値でもないし、そもそもプロパティとして持っているとは限らない。
if (props.name) {
  ...
  // nameの値を利用した処理に入る。
}

```

個人的にはそもそもそんなjsの仕様を利用した作りは丁寧に作らないとバグの温床になるため、 \
しっかり考える時間がない限りやめた方がいいと思いますし、たいていよくわからないバグを作りがちです。 \
tsでも型によりエラーになるので、下記のように*横着せずプロパティの存在を確認*してください。

```ts

interface Props {
  name?: string
}

export default function HelloComponent(props: Props) {
// props
//
}

// nameが存在して、空文字でなければ処理に入る。
if ("name" in props && props.name !== "") {
  ...
}
```

### ブラケット記法

javascriptにあまり詳しくない人は面食らうと思うが、 \
javascriptは下のようにundefinedかもしれない変数をキーに \
オブジェクトを作成することができる。

```js

// nameがundefinedかもしれないがエラーにならない。
const helloObject = {
  x: 30,
  [name]: false

}
```

プログラマの意図としては、nameがundefinedならnameキー自体存在しない、またはfalseとなることを想定していると思う。 \
しかし、これはバグの温床になるせいか大抵のプログラミング言語では許されておらず、 \
typescriptでも当然許されていない。そのため上の書き方をすると*typescriptでは構文エラー*になる。

この書き方をしたい場合nameが下記のようにundefinedでないことを保証する必要がある。

```ts

// typescriptで後からオブジェクトにプロパティを追加するときには下のようにオブジェクトの型を宣言する必要がある。
const helloObject:{ [key: string]: boolean | number } = {
  x: 30
}

if ( name !== undefined ) {
  // if文に入る条件がundefinedでないことであるため、undefinedでないことが保証されている。
  helloObject[name] = false;
}

const HelloClasses = classNames(cardClassesShape);
```

これをどのような時に使うかというと例えば下のようにreactでclass名を引数にとるかどうか。
引数が渡されるかわからない時に役に立つ。

```ts

const classes = useStyles();

const helloClassesShape: { [key: string]: boolean } = {};

  if (color !== undefined) {
    helloClassesShape[classes[color]] = true;
  }

  const helloClasses = classNames(helloClassesShape);
```

### useStateがnull

```js
const [element, setElement] = React.useState(null);
```

nullやundefinedを使うとany型になってしまうため、インテリセンスが効かなくなります。 \
使う型は決まっているはずなので、 \
下記のようにジェネリックを使って型を指定してください。

```ts
const [element, setElement] = React.useState<HTMLElement | null>(null);
```

string型や、number型などの値型はgolangみたく、useStateの初期化時に \
それぞれ空文字列,0で初期化するのもいいと思います。

```ts
// 暗黙的に型が決まるため、補完が効くようになります。
const [cnt, setCnt] = React.useState(0);
const [message, setMes] = React.useState('');
```
