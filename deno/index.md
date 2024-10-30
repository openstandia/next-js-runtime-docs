## 概要

Deno（ディーノ）は、Node.js の作成者である Ryan Dahl 氏によって設計された JavaScript および TypeScript のランタイムです。Node.js の課題を踏まえた上で、Deno はより安全でモダンな開発体験を提供することを目指して作られました。

### Deno の特徴

Deno は Node.js と比較すると以下のような特徴を持っています。

##### オールインワン開発環境

Deno ではランタイム、パッケージ管理、ビルドツール、テストツールなど、開発に必要なツールが統合されています。

##### セキュリティ

Deno ではデフォルトでファイルシステムやネットワーク、環境変数へのアクセスが制限されています。そのため、Deno 実行時には特定の権限を明示的に与える必要があります。この仕組みにより、Deno では高いセキュリティを期待できます。

##### モジュールシステム

Deno では ES モジュール（ECMAScript Modules）をデフォルトでサポートしており、モジュールは import 構文でファイルの URL から直接インポートします。これにより、npm のようなパッケージマネージャを必要としません。また、Deno では公式が提供している標準モジュールだけでなくサードパーティモジュールや npm パッケージを使用することが可能です。

##### TypeScript のサポート

TypeScript をネイティブでサポートしており、特別な設定なしで TypeScript ファイルを直接実行できます。

##### ツールチェーン

Deno にはテストツールやフォーマッター、リンターなどが標準装備されています。これにより、追加のツールセットを導入する必要が少なくなります。

## セットアップ

Deno の開発を始めるための基本的な手順は以下の通りです。

- Deno のインストール
- 環境構築
- プロジェクトの作成

### インストール

環境によってインストール方法は異なるため、適切な手段を選んでください。

MacOS や Linux では、次のコマンドでインストールできます。

```bash
curl -fsSL https://deno.land/install.sh | sh
```

Windows では、次のコマンドでインストールできます。

```bash
irm https://deno.land/install.ps1 | iex
```

他の方法や詳しいインストール方法は[公式のドキュメント](https://docs.deno.com/runtime/fundamentals/installation/)を参照してください。

以下のコマンドを実行してバージョンが表示されればインストールは成功しています。

```bash
deno --version
```

### 環境構築

#### エディター

Visual Studio Code（VSCode）、JetBrain IDE には Deno での開発をサポートする拡張機能が存在するため、エディターとしては Visual Studio Code または JetBrain IDE をお勧めします。

VSCode では拡張機能検索タブから`Deno`と検索し、[Denoland から提供されている拡張機能](https://marketplace.visualstudio.com/items?itemName=denoland.vscode-deno)をインストールします。その後、`Ctrl+Shift+P`で表示されるコマンドパレットで`Deno: Initialize Workspace Configuration`を入力し実行することでワークスペースに Deno が設定され、 VSCode で Deno 開発を行うための設定が完了します。

JetBrain IDE では File > Settings の Plugins 画面から`Deno`と検索し公式から提供されているプラグインをインストールしてください。Deno で開発を行うプロジェクトにてフレームワークとして Deno が認識されていれば設定は完了です。

#### ネットワーク関連設定

プロキシや証明書を使用したい場合、Deno ランタイムでは特別な環境変数を設定する必要があります。

プロキシの環境設定

```text
HTTP_PROXY=http://<ユーザ名>:<パスワード>@<proxyサーバのアドレス>:<ポート>
HTTPS_PROXY=http://<ユーザ名>:<パスワード>@<proxyサーバのアドレス>:<ポート>
```

証明書の環境設定（特定の証明書を参照する場合）

```text
DENO_CERT=<証明書までのパス>
```

証明書の環境設定（システム証明書ストアを参照する場合）

```text
DENO_TLS_CA_STORE=system
```

Deno コマンド実行時に証明書を指定して実行することも可能です。

```bash
deno --cert <証明書までのパス> main.ts
```

### プロジェクトの作成

Deno では以下のコマンドで新しいプロジェクトを作成できます。

```bash
deno init my_project
```

以下のような構造を持つプロジェクトのディレクトリが作成されます。

```text
my_project
├── deno.json // プロジェクト設定ファイル
├── main_test.ts // アプリケーションファイル
└── main.ts // アプリケーションテストファイル
```

作成された`main.ts`には２つの数字を足し合わせるシンプルなプログラムが記述されています。以下のコマンドで実行できます。

```bash
deno run main.ts
```

#### 設定ファイル

Deno には標準的な設定ファイルが存在しません。Deno は設定なしで動作し、必要な設定は基本的にコマンドラインオプションで行います。ただし、オプションとしていくつかの設定を deno.json に記述することができます。

deno.json には以下の内容を指定できます。

- `tsconfig.json`の設定
- `import-map`の設定
- Deno のサブコマンド関連の設定
- `deno task`コマンドの設定
- ロックファイルの設定

##### `tsconfig.json`の設定

`tsconfig.json`で記述する設定を同様の書き方で Deno に反映させることができます。

```json
{
  "compilerOptions": {
    "allowJs": true,
    "lib": ["deno.window"],
    "strict": true
  }
}
```

##### `import-map`の設定

`import-map`を設定することで Deno で使用したいモジュールを一元管理することができます。Deno での一元管理については # モジュールの一元管理 を参照してください。

<!-- #TODO:モジュールの一元管理を記述したらアンカーリンクを設定予定 -->

```json
{
  "imports": {
    "@luca/cases": "jsr:@luca/cases@^1.0.0",
    "cowsay": "npm:cowsay@^1.6.0",
    "cases": "https://deno.land/x/case/mod.ts"
  }
}
```

##### Deno のサブコマンド関連の設定

Deno のサブコマンドである`deno lint`、`deno fmt`の設定を記述することができます。
実際の設定項目やサブコマンドの使い方は # リンター・フォーマット で詳しく記述しています。

<!-- #TODO:リンター・フォーマットを記述したらアンカーリンクを設定予定 -->

```json
{
  "lint": {
    // lint対象ファイル
    "include": ["src/"],
    // lint対象除外ファイル
    "exclude": ["src/testdata/", "src/fixtures/**/*.ts"],
    // lintのルール設定
    "rules": {
      "tags": ["recommended"],
      "include": ["ban-untagged-todo"],
      "exclude": ["no-unused-vars"]
    }
  },
  "fmt": {
    // フォーマット設定
    "useTabs": true,
    "lineWidth": 80,
    "indentWidth": 4,
    "semiColons": false,
    "singleQuote": true,
    "proseWrap": "preserve",
    // フォーマット対象ファイル
    "include": ["src/"],
    // フォーマット対象除外ファイル
    "exclude": ["src/testdata/", "src/fixtures/**/*.ts"]
  }
}
```

##### `deno task`コマンドの設定

Node.js の`npm run script`に相当するスクリプト実行を簡略化するための設定を記述することができます。

```json
{
  "tasks": {
    "start": "deno run --allow-read main.ts"
  }
}
```

上記の設定を記述しておくことで、`npm run script`と同様に、以下のコマンドで`deno run --allow-read main.ts`を省略化した形で実行できます。

```bash
deno task start
```

##### ロックファイルの設定

Deno が依存関係の整合性を保証するために使用するロックファイルの設定を行うことができます。ロックファイルの場所を指定したり、使用を無効にすることも可能です。何も指定しない場合、デフォルトでロックファイルは使用され、パスは`./deno.lock`に設定されています。

```json
{
  "lock": {
    "path": "./deno.lock",
    "frozen": true
  }
}
```

```json
{
  "lock": false
}
```

## TypeScript

Deno は TypeScript をネイティブでサポートしています。そのため、Deno CLI さえインストールすれば、TypeScript を実行したりインポートしたりすることが可能です。Deno には TypeScript コンパイラが組み込まれており、追加の設定なしで TypeScript コードを JavaScript にコンパイルします。また、Deno は TypeScript コードの型チェックも行うことができ、tsc のような別の型チェックツールを使う必要はありません。

### 型チェック

Deno では、コードを実行せずに`deno check`サブコマンドから型チェックを行うことができます。

```bash
deno check module.ts
# リモートモジュールやnpmパッケージも型チェックする場合
deno check --all module.ts
# JSDocに書かれたコードスニペットも型チェックする場合
deno check --doc module.ts
# マークダウンファイル内のコードスニペットを型チェックする場合
deno check --doc-only markdown.md
```

`deno run`コマンドを使用する際、Deno は型チェックをスキップしてコードを直接実行します。実行前にモジュールの型チェックを行うには、`deno run`に`--check`フラグを付けて使用します。

```bash
deno run --check module.ts
# リモートモジュールやnpmパッケージも型チェックする場合
deno run --check=all module.ts
```

## CLI

Deno の CLI は、スクリプトの実行、依存関係の管理、さらにはコードをスタンドアロンの実行ファイルにコンパイルすることもできます。Deno の CLI で使用できるコマンドの一部を記載します。他のコマンドやオプションに関しては[公式ドキュメント](https://docs.deno.com/runtime/reference/cli/all_commands/)を参照してください。`fmt`、`lint`、`test`に関しては別の章で詳細を説明しています。

| コマンド       | 内容                                                                                                                 |
| -------------- | -------------------------------------------------------------------------------------------------------------------- |
| `deno run`     | TypeScript や JavaScript のプログラムを実行します。                                                                  |
| `deno task`    | `deno.json`に定義したスクリプトを実行します。                                                                        |
| `deno install` | `deno.json / package.json` に基づいて依存関係をインストールします。                                                  |
| `deno add`     | `deno.json` に依存関係を追加します。                                                                                 |
| `deno fmt`     | コードを標準的なコードスタイルに整えます。`deno.json`でフォーマットの設定をカスタマイズできます。                    |
| `deno lint`    | TypeScript や JavaScript のコードの品質やスタイルをチェックします。`deno.json`でルールの設定をカスタマイズできます。 |
| `deno test`    | `Deno.test`で宣言したテストを実行し、結果を出力します。                                                              |
| `deno compile` | TypeScript や JavaScript のプログラムを単一実行可能ファイルへコンパイルします。                                      |

### watch フラグ

`deno run`、`deno test`、`deno compile`、`deno fmt`に `--watch`フラグを付けることで、ビルトインのファイル監視機能を有効にできます。ファイル監視機能は、ソースファイルに変更があるとアプリケーションを自動でリロードします。特に開発中に便利で、アプリを手動で再起動せずに変更の効果をすぐに確認できます。

```bash
deno run --watch main.ts
deno test --watch
deno fmt --watch
```

## パッケージマネージャ

<!-- パッケージのインストール方法やバージョン管理方法 -->

## 標準ライブラリ・API

<!-- Denoが提供する標準ライブラリやAPIの一覽 -->

## npm パッケージの利用

Deno では、基本的には標準モジュールやサードパーティーモジュールを使用しますが、npm パッケージを利用することもできます。主に以下の２つの方法で npm パッケージの使用がサポートされています。

### URL 形式での利用

モジュールを使用する際と同じ方法で npm レジストリからパッケージを取得できます。また、モジュールと同様に初回実行時にパッケージをインストールするため、事前にパッケージのインストールをする必要はありません。

#### パッケージのインポート

`npm:<パッケージ名>@<バージョン>`という形式で対象のパッケージを npm レジストリからインポートすることが可能です。

```js
import cowsay from 'npm:cowsay@1.6.0';
console.log(cowsay.say({ text: 'Hello' }));
```

モジュールと同様に初回実行時以降はローカルにキャッシュされたパッケージが利用されます。

#### パッケージのバージョン管理

`import`文の中でバージョンを指定して管理します。モジュールと同様に`import_map.json`や`deps.ts`を使用した一元管理も可能です。詳しくは #モジュールのバージョン管理 を参照してください。

<!-- #モジュールのバージョン管理を記述したらアンカーリンクを設定予定 -->

### `package.json`を使用した利用

`package.json`を使用して npm パッケージを取得できます。これにより、Node.js で定義した依存関係を Deno でも使用できるようになるため、Node.js からの移行が容易になります。`npm install`のようなパッケージの事前インストールは必要なく、初回実行時にパッケージが自動でインストールされます。

#### `package.json`を使用したパッケージのインポート

以下のような`package.json`が定義されている場合、Node.js と同様に`import`文を記述できます。実行すると`node_modules`と Deno の lock ファイルである`deno.lock`が作成されます。

```json
{
  "dependencies": {
    "cowsay": "^1.6.0"
  }
}
```

```js
import cowsay from 'cowsay';
console.log(cowsay.say({ text: 'Hello' }));
```

#### `package.json`を使用したパッケージのバージョン管理

Node.js と同様に、`package.json`でバージョンが管理されます。そのため、`import_map.json`や`deps.ts`を使用せずにパッケージの一元管理ができます。また、`deno.lock`ファイルを使用することで、依存関係の整合性を保つことができます。

## 環境変数

Deno で環境変数を使用する方法はいくつかあります。

### `Deno.env`の組み込みサポート

Deno では、`Deno.env`を使用して環境変数の組み込みサポートを提供しています。
`Deno.env`には、getter と setter のメソッドがあります。使用例は以下の通りです。

```typescript
Deno.env.set('FIREBASE_API_KEY', 'examplekey123');
Deno.env.set('FIREBASE_AUTH_DOMAIN', 'firebasedomain.com');

console.log(Deno.env.get('FIREBASE_API_KEY')); // examplekey123
console.log(Deno.env.get('FIREBASE_AUTH_DOMAIN')); // firebasedomain.com
console.log(Deno.env.has('FIREBASE_AUTH_DOMAIN')); // true
```

環境変数を扱う場合は実行時に`--allow-env`オプションが必要です。

```bash
deno run --allow-env main.ts
```

### `.env`ファイルのサポート

Deno は`.env`ファイルをデフォルトでサポートしています。実行時に`--env`フラグを付けることで自動的に`.env`ファイルから環境変数を読み込みます。

`.env`ファイルの中には、`key=value`の形で環境変数名とその値を設定します。また、環境変数の展開も可能です。

```bash
HOST=localhost
PORT=3000
URL=http://${HOST}:${PORT}
```

読み込んだ環境変数は`Deno.env.get`でソースコード内で使用できます。

```typescript
console.log(Deno.env.get('URL')); // http://localhost:3000
```

環境変数を扱う場合は実行時に`--allow-env`オプションが必要です。

```bash
deno run --allow-env --env main.ts
```

その他にも、`dotenv/load`モジュールを使用することで`--env`を付けなくても`.env`で定義された環境変数を読み込むことができます。

```typescript
import 'jsr:@std/dotenv/load';

console.log(Deno.env.get('URL')); // http://localhost:3000
```

`dotenv/load`を使用する場合は`--allow-read`オプションも追加で必要です。

```bash
deno run --allow-env --allow-read main.ts
```

## ビルド

<!-- 各環境向けのビルド方法 -->

### SPA などのブラウザ向け

### Node.js 向け

### Deno 向け

## プラグイン

<!-- Denoプラグインの書き方 -->

## モジュールシステム

<!-- Denoでのモジュールに関する考え方 -->
<!-- パッケージやライブラリと重なる部分もあるかもなので必要に応じて削除 -->

## テスト

<!-- Deno標準のテストランナー -->

## セキュリティ

Deno はデフォルトでセキュアであるという特徴を持ちます。特に有効にしない限り、Deno で実行されるプログラムは、ファイルシステムへのアクセス、ネットワーク接続、環境アクセスなどの機密性の高い API にはアクセスできません。これらのリソースへのアクセスは、コマンドラインオプションまたはランタイムパーミッションプロンプトで明示的に許可する必要があります。

### スクリプト実行の許可設定

アクセスを許可するコマンドラインオプションは以下の表のとおりです。

| **オプション**        | **説明**                                                                                               | **例**                                                                                                                                    |
| --------------------- | ------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------- |
| `--allow-net`、`-N`   | ネットワークアクセス（HTTP、WebSocket など）を許可します。                                             | `deno run --allow-net script.ts`<br>`deno run --allow-net=github.com,jsr.io script.ts`<br>`deno run --allow-net=example.com:80 script.ts` |
| `--allow-read`、`-R`  | ファイルの読み取りを許可します。                                                                       | `deno run --allow-read script.ts`<br>`deno run --allow-read=foo.txt,bar.txt script.ts`                                                    |
| `--allow-write`、`-W` | ファイルの書き込みを許可します。                                                                       | `deno run --allow-write script.ts`<br>`deno run --allow-write=foo.txt,bar.txt script.ts`                                                  |
| `--allow-env`、`-E`   | 環境変数へのアクセスを許可します。                                                                     | `deno run --allow-env script.ts`<br>`deno run --allow-env=HOME,FOO script.ts`                                                             |
| `--allow-run`         | サブプロセスの実行を許可します。                                                                       | `deno run --allow-run script.ts`<br>`deno run --allow-run="curl,whoami" script.ts`                                                        |
| `--allow-ffi`         | FFI（Foreign Function Interface）による Rust、C、C++などの他の言語で書かれたコードの実行を許可します。 | `deno run --allow-ffi script.ts`<br>`deno run --allow-ffi=./libfoo.so script.ts`                                                          |
| `--allow-import`      | Web からのコードのインポートを許可します。。                                                           | `deno run --allow-import=example.com main.ts`                                                                                             |
| `--allow-all`、`-A`   | 全ての権限を許可します。                                                                               | `deno run --allow-all script.ts`                                                                                                          |
| `--allow-sys`、`-S`   | システム情報（`hostname`や`osRelease`など）へのアクセスを許可します。                                  | `deno run --allow-sys script.ts`<br>`deno run --allow-sys="systemMemoryInfo,osRelease" script.ts`                                         |
