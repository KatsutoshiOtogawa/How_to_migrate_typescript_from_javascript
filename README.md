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
できるだけ書いた人に何を想定していたか聞くこと。 \

