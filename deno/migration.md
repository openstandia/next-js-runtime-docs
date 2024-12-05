## 概要

既存のNode.jsプロジェクトにDenoを導入し、Denoプロジェクトへマイグレーションする方法について解説します。

## 設定ファイル

### `package.json`

Denoは`package.json`をサポートしているため、既存のものをそのまま流用できます。また、 同じディレクトリにDenoの設定ファイルである`deno.json`と`package.json`の両方が存在する場合、Denoは`deno.json`と`package.json`の両方で指定された依存関係を理解し、Deno固有の設定には`deno.json`ファイルを使用します。

`deno install`を実行することで、`package.json`に書かれた依存関係通りにパッケージをインストールします（ロックファイルとして`deno.lock`が新たに生成されます）。

`deno task <script-name>`を実行することで、`package.json`に書かれたnpm scriptsを実行できます（例：`deno task build`）。

### `.npmrc`

Deno 2.0以降のバージョンでは`.npmrc`をサポートしているため、既存の物を流用できます。

### `.env`

Denoはdotenvライクに`.env`ファイルをサポートしているので、プロジェクトで既存のものを流用できます。
`.env`ファイルが複数あり読み込みたいファイルを指定する場合は、`--env-file`オプションを利用します。指定しない場合、デフォルトで`.env`を読み込みます。

```bash
deno run --env-file=.env.1 src/index.ts
```

### `tsconfig.json`

Denoでは、既存の`tsconfig.json`を`--config`オプションで明示的に指定することで利用できます。

```bash
deno run --config tsconfig.json a.ts
```

しかし、公式としては、TypeScriptを設定ファイルなしでシンプルに利用するか、`tsconfig.json`の設定を`deno.json`（または`deno.jsonc`）に統合することを推奨しています。`deno.json`を使用することで設定を一元管理でき、Denoは自動的にこのファイルを読み込むため、追加のオプション指定が不要になります。

`tsconfig.json`の`compilerOptions`を`deno.json`の`compilerOptions`へ移行します。

```json:deno.json
{
  "compilerOptions": {
    "allowJs": true,
    "lib": ["deno.window"],
    "strict": true
  }
}
```

Denoでは`tsconfig.json`に必要だったいくつかの設定が不要になる場合があります。
例えば、Denoでは基本的にESモジュールのみをサポートしているため`module`オプションは設定不要です。
プロジェクトの要件やDeno環境に応じて、設定を取捨選択してください。

### ESLint・Prettier

Denoはデフォルトのリンターとフォーマッターを持ちます。

#### `deno lint`

Denoの組み込みリンターである`deno lint`は、ESLintの推奨ルールセットに基づいています。そのため、Node.jsプロジェクトでESLintの推奨ルールを使用している場合、新たな設定をせずにそのまま移行できます。

`deno lint`を実行すると、デフォルトで現在のディレクトリとそのサブディレクトリ内のすべてのTypeScriptファイルおよびJavaScriptファイルを解析します。特定のファイルやディレクトリを指定して解析したい場合は、コマンドの引数としてそれらを渡すことができます。

```bash
deno lint           # デフォルト設定で実行
deno lint src/      # src/ディレクトリ内を解析
```

`deno lint`のルールは、`deno.json`で設定をカスタマイズできます。必要に応じてルールの適用範囲やカスタムルールの追加・除外を指定できます。以下は設定例です。

```json:deno.json
{
  "lint": {
    "include": ["src/"],
    "exclude": ["src/testdata/", "src/fixtures/**/*.ts"],
    "rules": {
      "tags": ["recommended"],
      "include": ["ban-untagged-todo"],
      "exclude": ["no-unused-vars"]
    }
  }
}
```

上記の設定内容は以下の通りです。

- `include`:`src/`ディレクトリにあるファイルのみlintする
- `exclude`:`src/testdata/`ディレクトリにあるファイルや、`src/fixtures/`ディレクトリにあるTypeScriptファイルをlintしない
- `rules.tags`:推奨されるルールを適用することを指定する
- `rules.include`:`ban-untagged-todo`ルールを追加する
- `rules.exclude`:`no-unused-vars`ルールを除外する

利用可能なリンタールールの詳細は[Deno公式のルールリファレンス](https://lint.deno.land/)をご参照ください。

#### `deno fmt`

Denoの組み込みフォーマッタである`deno fmt`は、[dprint](https://dprint.dev/)エンジンを使用しています。dprintは高速かつ柔軟なフォーマットを提供するため、既存プロジェクトで使用していたフォーマッター（例: Prettierなど）からの移行を公式では推奨しています。

`deno fmt`を実行すると、デフォルトで現在のディレクトリとそのサブディレクトリ内のすべてのTypeScriptファイルおよびJavaScriptファイルを整形します。特定のファイルやディレクトリを指定して整形したい場合は、コマンドの引数としてそれらを渡すことができます。

```bash
deno fmt           # デフォルト設定で実行
deno fmt src/      # src/ディレクトリ内を整形
```

`deno fmt --check`コマンドを使用すると、コードがデフォルトのフォーマットルールに従って適切にフォーマットされているかを確認できます。フォーマットに問題がある場合は、修正が必要なファイルがリストアップされます。
このコマンドは、既存プロジェクトでCI/CDパイプラインやpre-commitフックを使用してフォーマットチェックを行っていた場合、簡単にDenoに移行できるため便利です。

`deno fmt`のフォーマットルールは、`deno.json`でカスタマイズできます。必要に応じてフォーマットの適用範囲やフォーマットルールを指定できます。以下は設定例です。

```json:deno.json
{
  "fmt": {
    "useTabs": true,
    "lineWidth": 80,
    "indentWidth": 4,
    "semiColons": true,
    "singleQuote": true,
    "proseWrap": "preserve",
    "include": ["src/"],
    "exclude": ["src/testdata/", "src/fixtures/**/*.ts"]
  }
}

```

上記の設定内容は以下の通りです。

- `useTabs`:インデントにスペースではなくタブを使用する
- `lineWidth`:1行の長さを80文字に制限する
- `indentWidth`:インデント幅を4スペースに設定する
- `semiColons`:文の末尾にセミコロンを追加する
- `singleQuote`:文字列にはシングルクォートを使用する
- `proseWrap`:テキストの改行スタイルを保持する
- `include`:`src/`ディレクトリ内のファイルをフォーマットする。
- `exclude`:`src/testdata/`ディレクトリ内のファイルおよび`src/fixtures/`ディレクトリ内のすべてのTypeScriptファイルを除外する

利用可能なフォーマットの詳細は[Deno公式のオプション一覧](https://docs.deno.com/runtime/fundamentals/linting_and_formatting/#available-options)をご参照ください。

## npmパッケージの利用

Denoはnpmパッケージのインポートをネイティブにサポートしています。ただし、既存のNode.jsプロジェクトから移行する場合、インポート宣言に`npm:`指定子を付ける必要があります。インポート形式は以下の通りです。

```text
npm:<package-name>[@<version-requirement>][/<sub-path>]
```

以下は、`node-emoji`パッケージを使用したサンプルコードとその実行結果です。

```typescript
import * as emoji from "npm:node-emoji";

console.log(emoji.emojify(`:sauropod: :heart:  npm`));
```

```bash
deno run main.js
🦕 ❤️ npm
```

移行時にすべてのインポート宣言を編集するのは大変です。そこで、Denoではモジュールのバージョンを一元管理するために[`import-maps`](https://github.com/WICG/import-maps)を使用することを推奨しています。`import-maps`は`deno.json`ファイルの`imports`フィールドを使用します。

```json:deno.json
{
  "imports": {
    "node-emoji": "npm:node-emoji",
    "react": "npm:react",
  }
}
```

既存のプロジェクトに合わせて`import-maps`を設定することでインポートの編集を最小限にすることが可能です。

```typescript
import { node-emoji } from "node-emoji";
import { react } from "react";
```

Denoは基本的にESModulesをサポートしていますが、多くのnpmパッケージはCommonJS形式で書かれています。Denoはこれらのパッケージを自動的に検出し、インポート時にシームレスに動作するよう対応しています。そのため、既存のプロジェクトで使用していたnpmパッケージも、ほとんどの場合そのまま移行して使用できます。

## workspaces

npmが提供するmonorepo向け機能の[workspaces](https://docs.npmjs.com/cli/v9/using-npm/workspaces)はそのままDenoで使用できます。

また、`deno.json`へ転記することで、Denoネイティブのworkspaceとして設定できます。

```json:deno.json
{
  "workspace": ["./add", "./subtract"]
}
```

npmの`workspaces`と同様の動作が期待できます。ただし、設定フィールド名が`workspaces`ではなく`workspace`である点に注意してください。

## テストフレームワーク（Vitestからの移行）

VitestはDenoでも実行できるので、テストスクリプト自体はそのまま実行できます。

```bash
# "test": "vitest"と定義されている前提
deno task test
```

Denoにはネイティブのテストランナーが搭載されているので、そちらを使うようにマイグレーションすることもできます。 ネイティブのテストランナーを使用すると、各テストのパーミッションをきめ細かく制御できるため、よりセキュアで統合されたテスト環境を構築できます。
ネイティブのテストランナーを使用する場合はDeno特有の書き方にテストコードを修正する必要があります。また、テストファイル名を`{*_,*.,}test.{ts, tsx, mts, js, mjs, jsx}`の形式に合わせる必要があります。

__`math_test.ts`（テストファイル）__

```typescript
import { assertEquals } from "jsr:@std/assert";

Deno.test("足し算のテスト", () => {
  const x = 1 + 2;
  assertEquals(x, 3);
});
```

__コマンド__

```bash
deno test

# $ deno test
# Check [File Path]
# running 3 tests from ./math_test.ts
# addTest1 ... ok (0ms)
# addTest2 ... ok (0ms)
# addTest3 ... ok (0ms)

# ok | 3 passed | 0 failed (3ms)
```

## サンプル：Next.js プロジェクト

前提として、`npx create-next-app@latest`によって作成されたプロジェクトとします。
DenoでNext.jsを動作させる場合、Denoが直接Next.jsを実行するわけではなく、Next.js自体を介して動作させます。
そのため、基本的にはコードや設定ファイルを修正しなくてもNext.jsは動作します。

### 依存関係のインストール

`deno install`を実行して必要なパッケージをインストールします。
`deno.lock`ファイルと`node_modules`ディレクトリが作成されることを確認してください。

### 開発サーバの起動

`deno task dev`を実行して開発サーバを起動します。
denoは`package.json`内の`scripts`の内容を参照するため、追加の設定をせずに開発サーバの起動ができます。

#### 起動失敗時の対応

起動に失敗する場合、ランタイムがNode.jsからDenoに変わったことに起因するエラーが発生する可能性があります。
以下の原因と対策を確認し、必要に応じて対応してください。

| 原因 | 対策 |
|-|-|
|CommonJSを使用しており、Denoがサポートできない|<ul><li>npmパッケージ内のモジュールであればDenoがサポートしているので、使用するモジュールがDenoでサポートされているか確認する</li><li>[公式ドキュメント](https://docs.deno.com/runtime/fundamentals/node/#commonjs-support)を参照し、設定ファイルを修正する</li></ul>|
|Node.jsの組み込みモジュールを使用しているためDenoで動作しない|<li>import文に`node:`を追加する（例：`import * as http from "node:http";`）</li>|
|実行スクリプトにNode.js環境でのみ動作する機能を使用している|<li>Denoで動作する代替手段で実行スクリプト修正する</li>|

### ビルド

`deno task build`でビルドを実行します。
開発サーバの起動スクリプトと同様に、ビルドスクリプトにNode.js環境専用の機能が含まれている場合、必要に応じてビルドコマンドを修正してください。

### `script`プロパティ

`package.json`の`script`プロパティに記述された定義は、修正なしで`deno task <script>`で実行できますが、`npm`指定子を付けることでも実行可能です。

例えば、Next.jsのプロジェクトであれば、以下のように`npm`指定子を使った定義に修正できます。

```json:package.json
"scripts": {
    "dev": "deno run npm:next dev",
    "build": "deno run npm:next build",
    "start": "deno run npm:next start",
    "lint": "deno run npm:next lint"
  },
```

`npm`指定子を使った方法では、`deno run`に対応したオプションを追加することができます。例えば、以下のように細かくセキュリティ設定を制限したり、環境変数定義ファイルを明示的に指定することができます。

```json:package.json
    "build": "deno run -E -env-file=.env.local npm:next build",
```

## Tips

### シェルスクリプト

<!-- TODO -->
