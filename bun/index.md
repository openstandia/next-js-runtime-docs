## 概要

Bun（バン）は、高速なプログラミング言語のZigによって作成されたJavaScriptおよびTypeScriptのランタイムです。Node.jsの使用感を意識しつつ、速度向上やユーティリティの追加を図っています。

### Bun の特徴

Bun は Node.js と比較すると以下のような特徴を持っています。

#### オールインワン開発環境

単なるランタイムとしての機能だけでなく、Node.jsではnpmと別れていたパッケージ管理や、別ツールに頼っていたビルドツールやテストツールを内包しており、手軽かつ高速にこれらの機能を利用できます。

#### モジュールシステム

`package.json`で依存関係を管理するNode.js従来の方式を踏襲しているものの、その動作自体は非常に高速であり、ロックファイルをバイナリデータで管理するなどの特徴があります。

#### TypeScript のサポート

TypeScript をネイティブでサポートしており、特別な設定なしで TypeScript ファイルを直接実行できます。

#### Bun Shell

Bun Shellという機能を使えば、JavaScriptの構文でシェルライクなスクリプトを記述できます。

## セットアップ

### インストール

開発端末へBunをインストールする方法はOSによって異なります。

#### macOS / Linux
```shell
curl -fsSL https://bun.sh/install | bash
```

#### Windows（Powershell）
```shell
powershell -c "irm bun.sh/install.ps1 | iex"
```

※プロキシ環境下でインストールする場合、端末のプロキシ設定を行ってからインストールを実行してください。

インストールするとコマンドラインでBunのCLIを利用できます。以下のコマンドでインストールされたことを確認できます。

```shell
bun --version
```

### ネットワーク設定

開発環境が、ネットワーク疎通のために特別な設定が必要な環境の場合、Bunのためにあらかじめ設定しておく必要があります。

#### プロキシ

Bunはデフォルトで`HTTP_PROXY`や`HTTPS_PROXY`環境変数を読み込みます。下記のように設定してください。

```
HTTP_PROXY=http://<ユーザ名>:<パスワード>@<proxyサーバのアドレス>:<ポート>
HTTPS_PROXY=http://<ユーザ名>:<パスワード>@<proxyサーバのアドレス>:<ポート>
```

#### 証明書

特定の証明書を読み込ませる必要がある場合、`NODE_EXTRA_CA_CERTS`環境変数に設定します（下記はNetskopeを利用している端末での例です）。

```
NODE_EXTRA_CA_CERTS=/Library/Application Support/Netskope/STAgent/data/nscacert.pem
```

### Bunプロジェクトの作成

**Bunプロジェクトを作成したいディレクトリ**で以下のコマンドを実行します。

```shell
bun init
```

下記のようなプロンプトが表示されるので、パッケージ名（`package name`）とエントリーポイント（`entry point`）を指定すると、必要なファイルがカレントディレクトリに生成されます。

```
bun init helps you get started with a minimal project and tries to
guess sensible defaults. Press ^C anytime to quit.

package name (quickstart):
entry point (index.ts):

Done! A package.json file was saved in the current directory.
 + index.ts
 + .gitignore
 + tsconfig.json (for editor auto-complete)
 + README.md

To get started, run:
  bun run index.ts
```

BunはNode.jsとの互換性を意識しているため、従来のNode.jsプロジェクトと同様に`package.json`や`tsconfig.json`が生成されます。


### 設定ファイル

Bunは`package.json`や`tsconfig.json`を参照するので、既存のNode.jsプロジェクトと同様の設定のみであればこれらのファイルで設定は完結します。
一方、Bun固有の設定をファイルとして定義する場合は、`bunfig.toml`というファイルをプロジェクトルートに作成します。このファイルは任意であり、これがなくてもBunプロジェクトは動作します。

`bunfig.toml`で設定できる項目の全量は[公式ドキュメント](https://bun.sh/docs/runtime/bunfig)を参照してください。
ここでは、その中から抜粋して説明します。

#### ランタイム

ランタイム全般に関することはトップレベルに記載します。

| 項目 | 説明 |
| - | - |
| `preload` | `bun run`コマンドでファイルやスクリプトを実行する前に実行したいファイルを指定します。 |
| `loglevel` | ログレベルを指定します。`debug`・`warn`・`error`から選択します。 |


```toml
preload = ["./preload.ts"]
loglevel = "debug"
```

#### テスト設定（`[test]`）

| 項目 | 説明 |
| - | - |
| `test.preload` | テスト実行前に実行したいファイルを指定します（モックの設定など）。 |
| `test.coverage` | カバレッジを取得するかどうかを指定します。デフォルト値は`false`です。 |
| `test.coverageThreshold` | カバレッジ率のしきい値を設けます。しきい値を下回る場合、`bun test`は非ゼロ値で終了します。デフォルトは設定されていません。 |

```toml
[test]
preload = ["./setup.ts"]
coverage = true
coverageThreshold = 0.9
# 行・関数・命令単位でも指定できる
# coverageThreshold = { line = 0.7, function = 0.8, statement = 0.9 }
```

#### パッケージマネージャ（`[install]`）

| 項目 | 説明                                                                                                          |
| - |-------------------------------------------------------------------------------------------------------------|
| `install.production` | `bun install`時に、`package.json`の`devDependencies`に含まれているパッケージを対象外にするか指定します（`true`だと対象外にする）。デフォルト値は`false`です。 |
| `install.frozenLockfile` | `bun install`時に、`bun.lockb`を更新せずに`bun.lockb`に基づいてインストールします。デフォルト値は`fasle`です。                                |
| `install.registry` | パッケージのレジストリを指定します。デフォルト値は `https://registry.npmjs.org/` です。                                                 |
| `install.lockfile.print` | ロックファイルを`bun.lockb`に加えてどのフォーマットで生成するか指定します。`yarn`のみサポートされています。                                              |

```toml
[install]
production = false
frozenLockfile = true

[install.lockfile]
print = "yarn"
```

## TypeScript

### `.ts`の実行
BunはネイティブでTypeScriptをサポートしています。すなわち、`.ts`拡張子のファイルを追加のパッケージを必要とせず実行します。

※実行環境のランタイムがBunであれば、開発～ビルドの過程でトランスパイルは不要です。一方で、開発ではBunを使うがアプリはNode.jsランタイムやブラウザ上で動かす場合は、トランスパイルが必要です。

### `tsconfig.json`

Bunは、トップレベルの`await`や`.ts`のimportといった、デフォルトのTypeScriptが許可しない機能をサポートしています。そのため、Bun公式が推奨する`compilerOptions`が存在します。

このファイルは、[Bunプロジェクトの作成](#bunプロジェクトの作成)で書いたように、`bun init`コマンドを実行してプロジェクトを作成した場合、デフォルトで生成されます。

```json
{
  "compilerOptions": {
    // Enable latest features
    "lib": ["ESNext"],
    "target": "ESNext",
    "module": "ESNext",
    "moduleDetection": "force",
    "jsx": "react-jsx",
    "allowJs": true,

    // Bundler mode
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "verbatimModuleSyntax": true,
    "noEmit": true,

    // Best practices
    "strict": true,
    "skipLibCheck": true,
    "noFallthroughCasesInSwitch": true,

    // Some stricter flags
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noPropertyAccessFromIndexSignature": true
  }
}
```

### Bun標準APIの型

`Bun.serve`のような、Bunが提供する標準APIをTypeScriptで使用するためには、それらAPIの型情報をインストールする必要があります。以下のコマンドで型情報をインストールできます。

```bash
bun add -d @types/bun
```

## CLI

BunのCLIで使えるコマンドの一部を記載します。コマンドによっては別の章で詳細を説明しています。

| コマンド | 内容 |
| - | - |
| `bun run <ファイル名>` | ファイルを実行します。対象の拡張子は`.js` `.jsx` `.ts` `.tsx`のいずれかです。 |
| `bun run <スクリプト名>` | `package.json`の`scripts`で定義されたスクリプトを実行します。 |
| `bunx <npmパッケージ名>` | `npx`と同様、npmパッケージをインストールして実行します。 |
| `bun build <ファイル名> --outdir <出力先ディレクトリ名>` | ファイルをビルドし、その成果物を指定したディレクトリに出力します。 |
| `bun install` | `package.json`に基づいてパッケージをインストールします。 |
| `bun add` | パッケージを追加します。 |
| `bun test` | テストを実行します。 |

### Node.js前提のCLIを動かす場合

一部のフレームワークが提供するCLI（例：Viteプロジェクトの`vite`コマンド）は、ランタイムとしてNode.jsを前提としているものがほとんどです。
そのCLIを使用する`scripts`をそのまま`bun run`で実行するとBunではなくNode.jsが起動します。
Bunを使用して実行したい場合は、`--bun`オプションを付与します。

```bash
bun --bun run dev // Bunを使用してnpm run dev実行
```

`package.json`の`scripts`を、`bunx --bun`を使うように修正することもできます。

```json:packages.json
{
  "scripts": {
    "dev": "bunx --bun vite",
    "build": "tsc -b && bunx --bun vite build"
  }
}
```

この場合、実行自体は`bun run`のみで大丈夫です。

```bash
bun run dev
```

## パッケージマネージャ

Bunのパッケージマネージャは、Node.jsの公式パッケージマネージャであるnpmと同じように使用できます。

### パッケージのインストール

`bun add <npmパッケージ名>`でパッケージをインストールできます。パッケージはプロジェクトルート（`bun add`を実行したディレクトリ）配下の`node_modules`ディレクトリに格納されます。

```shell
bun add zod
```

バージョンを指定することができます。

```shell
bun add zod@3.20.0
```

開発用にインストールする場合は`-d`オプションを付与します。

```shell
bun add -d prettier
```

※`--dev`や`-D`でも同様です。

### パッケージの削除

`bun remove`で特定のパッケージを依存関係から削除できます。

```bash
bun remove zod
```

### パッケージのアップデート

`bun update`ですべてのパッケージを最新版にアップデートします。

特定のパッケージだけをアップデートすることもできます。

```bash
bun update zod
```

`bun outdated`を実行すると、期限切れ（最新版でない）のパッケージを表形式で一覧化できます。

```
$ bun outdated

|--------------------------------------------------------------------|
| Package                                | Current | Update | Latest |
|----------------------------------------|---------|--------|--------|
| @types/bun (dev)                       | 1.1.6   | 1.1.7  | 1.1.7  |
|----------------------------------------|---------|--------|--------|
| @types/react (dev)                     | 18.3.3  | 18.3.4 | 18.3.4 |
|----------------------------------------|---------|--------|--------|
| @typescript-eslint/eslint-plugin (dev) | 7.16.1  | 7.18.0 | 8.2.0  |
|----------------------------------------|---------|--------|--------|
| @typescript-eslint/parser (dev)        | 7.16.1  | 7.18.0 | 8.2.0  |
|----------------------------------------|---------|--------|--------|
| esbuild (dev)                          | 0.21.5  | 0.21.5 | 0.23.1 |
|----------------------------------------|---------|--------|--------|
| eslint (dev)                           | 9.7.0   | 9.9.1  | 9.9.1  |
|----------------------------------------|---------|--------|--------|
| typescript (dev)                       | 5.5.3   | 5.5.4  | 5.5.4  |
|--------------------------------------------------------------------|
```

### バージョン管理

Node.jsと同様に、インストールしたパッケージは`package.json`で管理されます。

```json
  "dependencies": {
    "figlet": "^1.7.0",
    "zod": "^3.23.8"
  }
```

ロックファイル（実際にインストールしたバージョンやハッシュ値を記録するファイル）は`bun.lockb`として保存されます。このファイルは処理の高速化のためにバイナリ形式になっています。

ロックファイルはGit管理対象にして差分を確認できるようにすることが望ましいですが、`bun.lockb`はバイナリファイルなので差分はHuman Readable（人間にとって読みやすいもの）ではありません。`bun.lockb`の差分を確認したい場合はいくつかの方法があります。

参考：[bun.lockbのVersion管理をGitでどうやる？問題
](https://zenn.dev/watany/articles/e21a54cf3d56d8)

#### Gitの設定を変える方法

`.gitattributes`に下記を追記します。

```
*.lockb binary diff=lockb
```

次に、以下のコマンドを実行します。

```shell
git config diff.lockb.textconv bun
git config diff.lockb.binary true
```

`textconv bun`と書くことで、差分を計算する直前にそのファイルを`bun`コマンドに渡しその結果を差分の計算に用いるようになります。`bun`コマンドに`bun.lockb`を渡すと`yarn.lock`を出力します。この`yarn.lock`はHuman Readableなので、差分も確認しやすくなります。

#### `yarn.lock`も同時に管理する方法

`bun install --yarn`のようにオプションを付けるか、`bunfig.toml`に下記のように設定すると、`bun.lockb`と同時に`yarn.lock`も生成されるようになります。

```toml
[install.lockfile]

# whether to save the lockfile to disk
save = true

# whether to save a non-Bun lockfile alongside bun.lockb
# only "yarn" is supported
print = "yarn"
```

これで生成された`yarn.lock`を使って差分を確認します。

### `package.json`や`bun.lockb`に基づいたインストール

下記コマンドを実行すると、`package.json`の内容に応じてパッケージを一括でインストールします。

```shell
bun install
```

`package.json`には（デフォルトでは）キャレット付きでバージョンが指定されているので、`bun install`の実行タイミングによっては必ずしも常に同じバージョンがインストールされるとは限りません。

`bun.lockb`には最後に`bun install`されたときにインストールされた実際のバージョンが記録されているので、これが存在している場合はこれに基づいてインストールすることが望ましいです。

下記のように、`--frozen-lockfile`オプションを付与すると、`bun.lockb`に基づいてパッケージをインストールします。（`npm ci`と同様の働きをします）

```shell
bun install --frozen-lockfile
```

## 標準ライブラリ・API

Bunはいくつかの機能をランタイム標準として提供しており、そのうちのいくつかを説明します。全量は公式ドキュメントを参照してください。

### HTTP関係

#### HTTPサーバ

`Bun.serve`でHTTPサーバを起動できます。fetch APIやNode.jsの`http`モジュールと`https`モジュールを再実装しているため、それらの仕様に則った実装が可能です。下記の例では、fetch API標準の`Response`オブジェクトを返却します。

```js
Bun.serve({
  fetch(req) {
    return new Response("Bun!");
  },
});
```

`Bun.serve`の引数にはオブジェクトを渡します。リクエストを受け取ったときのハンドラは`fetch`プロパティに関数として定義します。`fetch`ハンドラは`async`や`await`をサポートします。

```js
import { sleep } from "bun";
Bun.serve({
  async fetch(req) {
    const start = performance.now();
    await sleep(10);
    const end = performance.now();
    return new Response(`Slept for ${end - start}ms`);
  },
});
```

返却する`Response`オブジェクトが静的な場合は`static`プロパティを使用できます。ルートごとに静的なレスポンスを定義することができます。

```js
Bun.serve({
  static: {
    // 例：ヘルスチェック用
    "/api/health-check": new Response("All good!"),

    // 例：301レスポンスを返却してリダイレクトを促す
    "/old-link": Response.redirect("/new-link", 301),

    // 例：静的な文字列を返す
    "/": new Response("Hello World"),

    // 例：静的なファイルを返す
    "/index.html": new Response(await Bun.file("./index.html").bytes(), {
      headers: {
        "Content-Type": "text/html",
      },
    }),
    "/favicon.ico": new Response(await Bun.file("./favicon.ico").bytes(), {
      headers: {
        "Content-Type": "image/x-icon",
      },
    }),

    // 例：JSONを返す
    "/api/version.json": Response.json({ version: "1.0.0" }),
  },

  // それ以外のルートに対するレスポンス
  fetch(req) {
    ...
  },
});
```

HTTPメソッドの種類は`Request`オブジェクトから取得できます。

```js
const server = Bun.serve({
  port: process.env.SERVER_PORT,
  async fetch(req) {
    const method = req.method;
    switch (method) {
      case "GET":
        ...
      case "POST":
        ...
      default:
        ...
    }

  }
})
```

#### HTTPクライアント

Bunはfetch APIをサポートしているので、外部サーバへのリクエストは`fetch`を用いることができます。

```js
const response = await fetch("http://example.com");

console.log(response.status);

const text = await response.text(); // 場合によってはresponse.json()やresponse.formData()などを使用
```

POSTメソッドを送りたい場合は、`method`プロパティに`"POST"`を指定した`Request`オブジェクトを`fetch`に渡します。

```js
const request = new Request("http://example.com", {
  method: "POST",
  body: "Hello, world!",
});

const response = await fetch(request);
```

その他、レスポンスボディの扱い方などは[MDNのドキュメント](https://developer.mozilla.org/ja/docs/Web/API/Fetch_API)を参照してください。

#### WebSocket

`Bun.serve`はWebSocketに対応しています。

WebSocketによる通信を受け付けるためには、`fetch`ハンドラ内でプロトコルのアップグレードを受け入れる必要があります（メカニズムの詳細は[MDNドキュメント](https://developer.mozilla.org/ja/docs/Web/HTTP/Protocol_upgrade_mechanism)を参照してください）。

※`fetch`の第二引数で`Server`オブジェクトを参照できます。このオブジェクトはサーバ側で使えるユーティリティを提供します。

```js
Bun.serve({
  fetch(req, server) {
    // WebSocketへのプロトコルのアップグレード
    if (server.upgrade(req)) {
      return; // アップグレード時は何もレスポンスを返さなくてよい
    }
    return new Response("Upgrade failed", { status: 500 });
  },
});
```

`websocket`プロパティにWebSocketのイベントハンドラを指定できます。

```js
Bun.serve({
  fetch(req, server) {},
  websocket: {
    message(ws, message) {}, // メッセージを受け取ったとき
    open(ws) {}, // WebSocket通信が確立されたとき
    close(ws, code, message) {}, // WebSocket通信が閉じられたとき
    drain(ws) {}, // バッファに空きができて追加のデータを受信できるようになったとき
  },
});
```

### バイナリやファイル関係

#### バイナリデータ

BunはJavaScript標準のバイナリに関する型やユーティリティを提供します。具体的には`TypedArray`、`Buffer`、`DataView`を提供します。
これらの詳細は[Bunの公式ドキュメント](https://bun.sh/docs/api/binary-data)で解説されています。

Node.jsのAPIである`Buffer`もBunで再実装されているので使用できます。配列的な使用感と`DataView`的な使用感の両方を兼ね備えています。
※サーバサイドでのみ利用できるAPIであるため、ブラウザでは使用できません。

その他にも、`Blob`オブジェクトや`File`オブジェクトのようなWeb標準のクラスをサポートしています。

#### ストリーム

ストリームは、バイナリデータの読み書きを一度に行わずに、都度都度行う方式です。Web標準のAPIとして`ReadableStream`と`WritableStream`が定義されており、Bunはどちらもサポートしています。

加えて、BunはDirect Readable Streamという仕組みをサポートしています。従来の`ReadableStream`では、データのチャンクはメモリ上に一度エンキューされ、コンシューマ側がデキューするまではチャンクが残り続けます。それに対し、Direct Readable Streamではキューを使わずに直接送信先のストリームに送信されます。

```js
const stream = new ReadableStream({
  type: "direct",
  pull(controller) {
    controller.write("hello");
    controller.write("world");
  },
});
```

#### ファイル

`Bun.file`を使うことでファイルへの参照（`BunFile`インスタンス）を作成できます。`BunFile`インスタンスはBlobインターフェースを実装しているので、`Blob`と同様のメソッドを利用してファイルを読み込めます。

```js
const foo = Bun.file("foo.txt");

await foo.text(); // 文字列として取得
await foo.stream(); // ReadableStreamとして取得
await foo.arrayBuffer(); // ArrayBufferとして取得
await foo.bytes(); // Uint8Arrayとして取得
```

標準入出力への参照も`BunFile`として提供されます。

```js
Bun.stdin; // readonly
Bun.stdout;
Bun.stderr;
```

ファイルの書き込みには`Bun.write`が使用できます。第一引数で書き込むファイルのパスを、第二引数で書き込む内容を指定します。

```js
const data = `あいうえお`;
await Bun.write("output.txt", data);
```

引数には`BunFile`インスタンスを渡すことができるので、コピー元とコピー先のそれぞれの`BunFile`インスタンスを使うことでファイルのコピーが行えます。

```js
const input = Bun.file("input.txt");
const output = Bun.file("output.txt"); // コピー先の参照（まだ存在していない）
await Bun.write(output, input);
```

### ユーティリティ

`Bun.sleep`で`Promise`を返すタイマーを起動できます。スレッドをブロックする場合は`Bun.sleepSync`を使用します。

```js
console.log("hello");
await Bun.sleep(1000);
console.log("hello one second later!");
```

オブジェクトの比較には`Bun.deepEquals`を使用します。ネストされていても判定が可能です。

```js
const foo = { a: 1, b: 2, c: { d: 3 } };

// true
Bun.deepEquals(foo, { a: 1, b: 2, c: { d: 3 } });

// false
Bun.deepEquals(foo, { a: 1, b: 2, c: { d: 4 } });
```

プロセス開始からの経過時間（ナノ秒）を`Bun.nanoseconds`で取得できます。

```js
Bun.nanoseconds();
```

ファイルパスやモジュールのパスは、`Bun.resolveSync`で解決できます。第一引数に対象のファイルやモジュールを、第二引数にルートとなるパスを指定します。
カレントディレクトリをルートにする場合は`process.cwd()`を、実行ファイルの場所をルートにする場合は`import.meta.dir`を第二引数に設定します。

```js
Bun.resolveSync("./foo.ts", "/path/to/project");
// => ファイル名なので、"/path/to/project/foo.ts"を返す

Bun.resolveSync("zod", "/path/to/project");
// => モジュールなので、"/path/to/project/node_modules/zod/index.ts"を返す

Bun.resolveSync("./foo.ts", process.cwd());
// => プロセスを起動したディレクトリからのパスを返す

Bun.resolveSync("./foo.ts", import.meta.dir);
// => このファイルが存在するディレクトリからのパスを返す

```

## npmパッケージの利用

Bunは標準でnpmパッケージを利用でき、パッケージマネージャもnpmパッケージを前提として動作します。そのため、npmパッケージを利用するのに特別な準備や設定は必要ありません。

npmパッケージのインストール方法などは[パッケージマネージャ](#パッケージマネージャ)を参照してください。

## 環境変数

### 設定

Bunはルートディレクトリにある`.env`ファイルを自動で読み取ります。
`.env`ファイルの中には、`key=value`の形で環境変数名とその値を設定します。

```
DB_USER=admin
DB_PASSWORD=password
```

`.env`ファイルにはいくつか種類があります。

| `.env.`ファイル | 説明 |
| - | - |
| `.env` | 必ず読み込まれます（`--env-file`使用時を除く）。 |
| `.env.development` | 開発モード時に読み込まれます。（`NODE_ENV=development`のとき） |
| `.env.production` | プロダクションモード時に読み込まれます。（`NODE_ENV=production`のとき） |
| `.env.local` | 必ず読み込まれます（`--env-file`使用時を除く）。通常、Gitでは管理されません。 |

特定の`.env`ファイルを指定したい場合は、`--env-file`オプションで指定します。複数指定した場合は、後に指定したもので上書きされます。

```bash
bun --env-file=.env.abc --env-file=.env.def run build
```

`.env`ファイル内では環境変数の展開が可能です。下記の例では、`BAR`の値は`helloworld`になります。

```bash
FOO=world
BAR=hello$FOO
```

`.env`ファイルで定義していない、もしくは定義したものの値を上書きしたい環境変数がある場合は、`bun`コマンド実行時に指定します。
下記コマンドでは環境変数`FOO`に`hello world`がセットされます。

```bash
FOO=helloworld bun run dev
```

### 取得

ソースコード内で環境変数を取得するには、いくつか方法があります。下記は、`API_TOKEN`という環境変数を取得する例です。

- `process.env.API_TOKEN`
- `Bun.env.API_TOKEN`
- `import.meta.env.API_TOKEN`

Bunが取得できる環境変数の一覧は、下記のコマンドで出力できます。

```bash
bun --print process.env
```

## ビルド

Bunはネイティブのバンドラを備えています。2024/11現在、バンドラ機能はベータ版となっています。

ビルドはCLIとBun API（JavaScript）のどちらかで実行できます。

```bash
bun build ./index.tsx --outdir ./build
```

```js
await Bun.build({
  entrypoints: ['./index.tsx'],
  outdir: './build',
});
```

Bunのバンドラはあくまでバンドルのみを行い、`tsc`のような、型チェックや型定義ファイルの生成は行いません。

上記のサンプルコードでは、ビルド成果物はコマンドを実行したカレントディレクトリの`.build`ディレクトリ配下に生成されます。

### ビルド設定

ビルド設定についていくつか説明します。全量と詳細な説明は[公式ドキュメント](https://bun.sh/docs/bundler#api)を参照してください。

#### `target`

```js
await Bun.build({
  entrypoints: ['./index.ts'],
  outdir: './out',
  target: 'browser', // デフォルト
})
```

どの環境向けにバンドルするかを指定します。

| 設定値 | デフォルト | 説明 |
| - | - | - |
| `browser` | ◯ | ブラウザ向けのバンドルを行います。ブラウザ環境で使えない一部の関数（`fs.readFile`など）は使えません。 |
| `bun` | | Bunランタイム向けのバンドルを行います。BunはネイティブでTypeScriptをサポートしており、サーバサイドで実行するため必ずしもバンドルは必要ありません（コードを直接実行できるため）。エントリポイントになるファイルの先頭に`#!/usr/bin/env bun`というシェバンがあると、`target`のデフォルト値は`bun`に設定されます。 |
| `node` | | Node.jsランタイム向けのバンドルを行います。現時点では実装されていませんが、将来的には、`bun:*`のようにBun APIを使うよう書いていても、自動でNode.js向けにpolyfillされるようになる予定です。 |

#### `format`

```js
await Bun.build({
  entrypoints: ['./index.tsx'],
  outdir: './out',
  format: "esm",
})
```

生成されたファイルのフォーマットを指定します。

| 設定値 | デフォルト | 説明 |
| - | - | - |
| `esm` | ◯ | ES Moduleで出力します。 |
| `cjs` | | CommonJSで出力します。こちらを選択すると、`target`のデフォルト値は`node`に変更されます。 |

#### `splitting`

```js
await Bun.build({
  entrypoints: ['./index.tsx'],
  outdir: './out',
  splitting: false, // default
})
```

複数のエントリポイントで共有されているコードをチャンクとして切り出すかどうかを指定します。

| 設定値 | デフォルト | 説明 |
| - | - | - |
| `falase` | ◯ | チャンクを生成しません。 |
| `true` | | チャンクを生成します。複数のエントリポイントからimportされているファイルを別ファイルとして生成します。 |

```js
await Bun.build({
  entrypoints: ['./entry-a.ts', './entry-b.ts'],
  outdir: './out',
  splitting: true,
})
```

```text
.
├── entry-a.tsx
├── entry-b.tsx
├── shared.tsx // このファイルがentry-aとentry-bから参照されている
└── out
    ├── entry-a.js
    ├── entry-b.js
    └── chunk-2fce6291bf86559d.js // shared.tsx相当
```

## プラグイン

BunはプラグインAPIを提供しており、ランタイムやバンドラで利用できるプラグインを作成することができます。
プラグインは、主にソースコードやファイルの読み込み時をインターセプトし、独自のローダーとしてふるまいます。例えば、デフォルトでは読み込めない`.scss`や`.yaml`などのファイルや、`.js`への変換が必要な`.svelte`（Svelteコンポーネント）といったフレームワーク固有のファイルを読み込むことができます。

### プラグインの記法

プラグインは以下の手順で有効化します。

1. `BunPlugin`型のオブジェクトを定義
2. `bun`が提供する`plugin`関数を使ってオブジェクトを登録
3. 2.のソースコードを`bunfig.toml`の`preload`に追加

#### `BunPlugin`型のオブジェクトを定義 & `bun`が提供する`plugin`関数を使ってオブジェクトを登録

```js
import { plugin, type BunPlugin } from "bun";

// `BunPlugin`型のオブジェクトを定義 
const myPlugin: BunPlugin = {
  name: "Custom loader",
  setup(build) {
    // プラグインを実装
  },
};

//  `bun`が提供する`plugin`関数を使ってオブジェクトを登録
plugin(myPlugin);
```

`BunPlugin`は、`name`と`setup`の２つのプロパティを持つオブジェクトで、`setup`には初期処理の関数を定義します。

`bun`からimportした`plugin`に`BunPlugin`オブジェクトを渡すことで、プラグインを登録します。

#### `bunfig.toml`の`preload`に追加

`bunfig.toml`の`preload`で、プラグインを登録するソースコードのパスを指定します。こうすることで、Bunのランタイムやバンドルが実行される前にプラグインを登録できます。

```toml
preload = ["./myPlugin.ts"]
```

### ローダー

プラグインの主な役割はローダーです。ローダーは、ソースコードでimportできるファイルを拡張したり、前処理で変形したりする役割を持ちます。

#### 例：YAMLファイルのローダー

[js-yaml](https://github.com/nodeca/js-yaml)を使って、`.yaml`ファイルをオブジェクトとしてimportできるようにするローダーです。
ファイルの読み込み時に処理を挟み込みたい場合は`build.onLoad`というライフサイクルフックを使用します。

戻り値はオブジェクトで、`loader`プロパティを含む必要があります。`loader`には、戻り値をどう解釈すればよいかが指示されています。今回は、`js-yaml`で読み込んだYAMLをオブジェクトとして扱いたいため、`object`を設定します。`object`を設定したとき、オブジェクト自体は`exports`プロパティで返します。

```ts
import { plugin } from "bun";

await plugin({
  name: "YAML",
  async setup(build) {
    const { load } = await import("js-yaml");

    // YAMLファイルを読み込んだとき（`filter`でフィルタ）
    build.onLoad({ filter: /\.(yaml|yml)$/ }, async (args) => {

      // YAMLファイルをパース
      const text = await Bun.file(args.path).text();
      const exports = load(text) as Record<string, any>;

      // モジュールとして返却
      return {
        exports,
        loader: "object",
      };
    });
  },
});
```

#### 例：Svelteコンポーネントのローダー

Svelteのソースコード（`.svelte`）を扱うには`.js`ファイルにコンパイルする必要があります。`svelte/compiler`としてコンパイラが公開されているので、そのコンパイラを通じてコンパイルしたものを返却します。`.js`ファイル（を含む、Bunの組み込みローダーが対応しているもの）としてローダーから返却する場合は`contents`というプロパティに`js`ファイルの文字列を設定します。
今回は、コンパイル後のソースコードを一つの文字列として`contents`に格納しています。

```ts
import { plugin } from "bun";

await plugin({
  name: "svelte loader",
  async setup(build) {
    const { compile } = await import("svelte/compiler");

    // Svelteコンポーネントを読み込んだとき
    build.onLoad({ filter: /\.svelte$/ }, async ({ path }) => {

      // ファイルの読み込みとコンパイル
      const file = await Bun.file(path).text();
      const contents = compile(file, {
        filename: path,
        generate: "ssr",
      }).js.code;

      // コンパイラ済みのコードを`js`として返却
      return {
        contents,
        loader: "js",
      };
    });
  },
});
```

このローダーを使うと、以下のようにimportできます。

```js
import "./sveltePlugin.ts";
import MySvelteComponent from "./component.svelte";

console.log(MySvelteComponent.render());
```


## テスト

Bunには標準でテストランナーが備わっています。
おおまかな記法は[Jest](https://jestjs.io/ja/)に則っているので、すでにJestやVitestを利用したことがある方は違和感なく記述できます。

### テストケースの書き方

`test`関数を使ってテストケースを定義します。複数のテストケースは`describe`関数でまとめることができます。

アサーションは`expect`関数を使います。

```ts
import { describe, test, expect } from 'bun:test'
import { add, sub } from './math.ts'

describe('計算関数のテスト', () => {
  test('足し算関数のテスト', () => {
    expect(add(2, 3)).toBe(5); // PASS
  })

  test('引き算関数のテスト', () => {
    expect(sub(5, 2)).toBe(1); // FAIL
  })
})
```

`expect`関数でアサーションできる内容（Matcher）の全量は[公式ドキュメント](https://bun.sh/docs/test/writing#matchers)を参照してください。基本的にはJestが備えているものをBun側で改めて実装する方針となっています。

### テストの実行

以下のコマンドでテストを実行します。

```shell
bun test
```

実行ディレクトリ配下から、以下のパターンにマッチするテストファイルが探査され実行されます。

- `*.test.{js|jsx|ts|tsx}`
- `*_test.{js|jsx|ts|tsx}`
- `*.spec.{js|jsx|ts|tsx}`
- `*_spec.{js|jsx|ts|tsx}`

ファイル名でフィルタリングする場合は`-t`オプション（または`--test-name-pattern`）を使います。下記の例ではファイル名に`addition`が含まれているテストファイルが実行されます。

```shell
bun test --test-name-pattern addition
```

特定のテストファイルを指定したい場合は引数で指定します。

```shell
bun test ./test/specific-file.test.ts
```

タイムアウトやリラン（同じテストを複数回実行する）などはオプションで設定できます。詳細は[公式ドキュメント](https://bun.sh/docs/cli/test)を参照してください。

### モック

モック関数を作りたい場合は`mock`関数に関数式を渡します。

```ts
import { mock } from 'bun:test'

const mockGetId = mock((base: string) => `${base}-mock`);
```

モック関数は呼び出し回数や呼び出されたときの引数などをアサーションできます。

---

実際のふるまいをモック化せずに、呼び出し回数などをアサーションしたい場合は`spyOn`を使用します（関数がオブジェクトのプロパティである必要があります）。

__`math.ts`__
```ts
export const add = (x: number, y: number) => {
    return x + y;
}

export const doubleAdd = (x: number, y: number) => {
    return add(add(x, y), y)
}
```

__`math.spec.ts`__
```ts
import { describe, test, expect, spyOn } from 'bun:test'
import * as math from '../math';

const spyMath = spyOn(math, 'add')

describe('math', () => {
    test('doubleAdd', () => {
        expect(math.doubleAdd(2, 3)).toBe(8);     // PASS
        expect(spyMath).toHaveBeenCalledTimes(2); // PASS
    })
}); 
```

---

モジュール全体をモック化する場合は`mock.module`を使用します。

__`random.ts`__
```ts
export const getRandom = () => Math.random();
```

__`math.ts`__
```ts
import { getRandom } from "./random";

export const randomTimes = (x: number) => {
    return x * getRandom();
}
```

__`math.spec.ts`__
```ts
import { describe, test, expect, spyOn, mock } from 'bun:test'
import { randomTimes } from '../math';

mock.module("../random", () => {
    return {
        getRandom: () => 100,
    }
})

describe('math', () => {
    test('random', () => {
        expect(randomTimes(1)).toBe(100);
    })
}); 

```

### ライフサイクルフック

テストの前後に決まった処理を実行したい場合は、ライフサイクルフックを使用します。以下の4つが用意されています。

| フック | 説明 |
| - | - |
| `beforeAll`	| 最初のテストの前に全体で1度だけ実行されます。 |
| `beforeEach` | 各テストの前に都度実行されます。 |
| `afterEach`	| 各テストの後に都度実行されます。 |
| `afterAll` | 最後のテストの後に全体で1度だけ実行されます。 |

ライフサイクルフックは後述のセットアップファイルや各テストファイル、`describe`内のそれぞれに記述できます。ライフサイクルフックのスコープは記述したブロックに限られます。

### プレロード

モックやライフサイクルフックの設定をテスト全体で共通化する場合、各テストファイルに書くのではなく、一つのファイルにまとめてテストコマンドの実行時に読み込ませることができます。

モック化の設定などをファイルに記述し、`bun test`のオプションもしくは`bunfig.toml`で指定します。

__`my-preload.ts`__
```ts
import { mock } from "bun:test"

mock.module("./utils/random", () => {
    return {
        getRandom: () => 100,
    }
})
```

```shell
bun test --preload my-preload.ts
```

※もしくは下記の内容を`bunfig.toml`に追記します。

```toml
[test]
preload = "./my-preload.ts"
```

### DOMテスト

Bunのテストランナーを使ってDOMのテストを実行したい場合、Bun公式は[happy-dom](https://github.com/capricorn86/happy-dom)の利用を推奨しています。DOMのテストのためには、テストランナーが`document`や`window`のようなブラウザAPIにアクセスする必要があるため、必要なパッケージをインストールします。

```shell
bun add -d @happy-dom/global-registrator
```

グローバルスコープでブラウザAPIを登録する関数（`GlobalRegistrator.register`）を呼び出す処理をプレロードするように設定します。

__`happydom.ts`__
```ts
import { GlobalRegistrator } from "@happy-dom/global-registrator";

GlobalRegistrator.register();
```

__`bunfig.toml`__
```toml
[test]
preload = "./happydom.ts"
```

この設定でテストケース内でDOMにアクセスできるようになるので、DOMテストを記述します。
先頭行に`/// <reference lib="dom" />`を追記することで、TypeScriptによるコンパイラエラーを防ぎます。

```ts
/// <reference lib="dom" />

import {test, expect} from 'bun:test';

test('dom test', () => {
  document.body.innerHTML = `<button>My button</button>`;
  const button = document.querySelector('button');
  expect(button?.innerText).toEqual('My button');
});
```

### カバレッジ

カバレッジを計測する場合、`bun test`のオプションもしくは`bunfig.toml`で指定します。

```shell
bun test --coverage
```

__`bunfig.toml`__
```toml
[test]
coverage = true
```

カバレッジのしきい値を設ける場合は`bunfig.toml`で`coverageThreshold`を指定します。

__`bunfig.toml`__
```toml
[test]
coverage = true
coverageThreshold = 0.9
```
## Bun Shell

Bun Shellは、Javascript / TypeScriptのソースコード上でシェルスクリプトを記述・実行できる機能です。
bashライクなシェルスクリプトを記述でき、Bun Shellによって実装されているコマンドについてはクロスプラットフォームで動作します。

### `$`によるコマンドの実行

`bun`モジュールが提供する`$`に、コマンドをテンプレートリテラルとして渡すことでコマンドを実行できます。

```js
import { $ } from "bun";

await $`echo "Hello World!"`; // 標準出力に"Hello World!"が表示される

await $`echo "Hello World!"`.quiet(); // .quiet()で標準出力に表示されなくなる

const welcome = await $`echo "Hello World!"`.text(); // .text()で出力内容をstringで取得できる

console.log(welcome); // Hello World!\n

const { stdout, stderr } = await $`echo "Hello World!"`.quiet(); // stdoutやstderrを分割代入するとBufferが取得できる

console.log(stdout); // Buffer(6) [ 72, 101, 108, 108, 111, 32 ]
console.log(stderr); // Buffer(0) []


// 行ごとに複数のコマンドを実行できる
await $`
echo 1
echo 2
echo 3
`
/*
  1
  2
  3
*/
```

### 例外

コマンドの実行結果が非ゼロ値の場合、`$`は例外（`ShellError`）を発生させます。

```js
import { $, ShellError } from "bun";

try {
    const result = await $`no-exist-command`.text();
    console.log(result)
} catch (err) {
    console.log(`Failed with code ${err.exitCode}`); // Failed with code 1
}

// .nothrow()で例外の発生を抑止できる
const { exitCode } = await $`no-exist-command`.nothrow();
console.log(exitCode)

// $.nothrow()でグローバルに抑止できる
$.nothrow();

// $.throws(boolean)でもグローバルに制御できる（trueで例外を発生させる）
$.throws(true);
```

### リダイレクト

bashと同様に、`>`や`<`を使って標準入出力に対してリダイレクトできます。

標準出力は以下のオブジェクトに`>`を使ってリダイレクトできます。

- `Buffer`
- `Uint8Array`
- `Uint16Array`
- `Uint32Array`
- `Int8Array`
- `Int16Array`
- `Int32Array`
- `Float32Array`
- `Float64Array`
- `ArrayBuffer`
- `SharedArrayBuffer`

ファイルへの書き込みは以下のオブジェクトにリダイレクトできます。

- `Bun.file(path)`
- `Bun.file(fd)`

以下のオブジェクトは標準入力に`<`を使ってリダイレクトできます。

- `Buffer`
- `Uint8Array`
- `Uint16Array`
- `Uint32Array`
- `Int8Array`
- `Int16Array`
- `Int32Array`
- `Float32Array`
- `Float64Array`
- `ArrayBuffer`
- `SharedArrayBuffer`
- `Bun.file(path)`
- `Bun.file(fd)`
- `Response` （レスポンスボディ）

リダイレクトの使用例は[Redirection](https://bun.sh/docs/runtime/shell#redirection)を参照してください。

### パイプ

bashと同様に、`|`を使ってあるコマンドの出力を別のコマンドの入力につなげられます。

```js
import { $ } from "bun";

const result1 = await $`echo "Hello World!" | wc -w`.text();

console.log(result1); // 2\n

// JavaScriptのオブジェクトもパイプの対象にできる

const response = new Response("hello i am a response body");

const result2 = await $`cat < ${response} | wc -w`.text();

console.log(result2); // 6\n

```

### コマンド置換

コマンドの中で`$(<別のコマンド>)`とすることで、別のコマンドの実行結果で置換できます。

```js
import { $ } from "bun";

// 現在時刻を出力
await $`echo current time: $(date)`; // current time: 2024年 10月 4日 金曜日 21時55分28秒 JST

// コマンドの実行結果を変数に代入して利用できる
await $`
  REV=$(git rev-parse HEAD)
  docker built -t myapp:$REV
  echo Done building docker image "myapp:$REV"
`;
```

### 環境変数

bashと同様に、コマンドより前に`key=value`の形で環境変数をセットできます。
もしくは、`.env`または`$.env`でオブジェクト形式でセットできます。

```js
import { $ } from "bun";

await $`FOO=foo bun -e 'console.log(process.env.FOO)'`; // foo\n

// string変数の埋め込みも可能
const foo = "bar123";
await $`FOO=${foo + "456"} bun -e 'console.log(process.env.FOO)'`; // bar123456\n

await $`echo $FOO`.env({ ...process.env, FOO: "bar" }); // bar

// $.envでグローバルにセット
$.env({ FOO: "bar" });

await $`echo $FOO`; // bar
```

### ワーキングディレクトリの変更

コマンド単位でワーキングディレクトリを指定する場合は`.cwd`でチェーンします。
グローバルに指定する場合は`$.cwd`を使用します。

```js
import { $ } from "bun";

await $`pwd`.cwd("/tmp"); // /tmp

// グローバルに指定
$.cwd("/app");
await $`pwd`; // /app
```

### 出力形式

コマンドの出力は様々な形式で取得できます。

```js
const result = await $`echo "Hello World!"`.text(); // string
const result = await $`echo '{"foo": "bar"}'`.json(); // JSON

// 行ごと
for await (let line of $`echo "Hello World!"`.lines()) {
  console.log(line); // Hello World!
}

// Blob
const result = await $`echo "Hello World!"`.blob();

```