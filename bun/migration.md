## 概要

既存のNode.jsプロジェクトにBunを導入し、Bunプロジェクトへマイグレーションする方法について解説します。

## 設定ファイル

### `package.json`

Bunは`package.json`をサポートしているため、既存のものをそのまま流用できます。

`bun install`を実行することで、`package.json`に書かれた依存関係通りにパッケージをインストールします（ロックファイルとして`bun.lockb`が新たに生成されます）。

`bun run <script-name>`を実行することで、`package.json`に書かれたnpm scriptsを実行できます（例：`bun run build`）。

### `.npmrc`

Bunは`.npmrc`をサポートしているため、既存のものを流用できます。

ただし、公式としては`bunfig.toml`への移行を推奨しています。Bun固有の設定項目もカバーしており、可読性が高いためです。

`.npmrc`で設定する項目のうち、主なものについてBunでの設定方法を説明します。

#### パッケージマネージャ

npmの`save-exact`オプションは、Bunの`install.exact`オプションに対応します。このオプションを`true`にすると、パッケージのインストール時にバージョンを完全に固定した状態で`package.json`に追記します。

```toml:bunfig.toml
[install]
exact = true
```

#### レジストリ

レジストリの指定は`install.registry`で指定します。認証はトークンやユーザ名/パスワード方式に対応しています。

```toml:bunfig.toml
[install]
registry = "https://registry.npmjs.org"
registry = { url = "https://registry.npmjs.org", token = "123456" }
registry = "https://username:password@registry.npmjs.org"
```

特定の[スコープ](https://docs.npmjs.com/cli/v10/using-npm/scope)のURLを設定したい場合は`install.scope`で指定します。`key=value`形式の`key`にあたる部分にスコープ名を指定します。

```toml:bunfig.toml
[install.scopes]
// @myorg/<パッケージ>をインストールするときに使用するレジストリ。パスワードを環境変数から読み込む場合は`{}`で囲む
myorg = { username = "myusername", password = "$npm_password", url = "https://registry.myorg.com/" }
```

#### プロキシ

Bunは設定ファイルではなく、環境変数の`HTTP_PROXY`や`HTTPS_PROXY`を参照します。そのため、プロキシを使用する端末にてこれらの環境変数を設定するようにしてください。

### `.env`

Bunはdotenvライクに`.env`ファイルをサポートしているので、プロジェクトで既存のものを流用できます。
`.env`ファイルが複数あり読み込みたいファイルを指定する場合は、`--env-file`オプションを利用します。指定しない場合、`NODE_ENV`に基づきます。

```bash
bun --env-file=.env.1 src/index.ts
```

### tsconfig.json

既存のものを流用できます。

### ESLint・Prettier

Bunはデフォルトのリンターやフォーマッターを持たないため、既存のサードパーティツールをそのまま使用します。
ESLintやPrettierは`bun`コマンドからでも実行できます。

## npmパッケージの利用

npmパッケージを`import`して利用するのに追加の手順は不要です。

## workspaces

npmが提供するmonorepo向け機能の[workspaces](https://docs.npmjs.com/cli/v9/using-npm/workspaces)はそのまま使用できます。

ただし、yarnやpnpmが独自に提供している同様のworkspaces機能には対応していません。Bunへのマイグレーション時には`package.json`へ転記する必要があります。


## テストフレームワーク（Vitestからの移行）

VitestはBunでも実行できるので、テストスクリプト自体はそのまま実行できます。

```bash
# "test:unit": "vitest"と定義されている前提
bun run test:unit
```

Bunにはネイティブのテストランナーが搭載されているので、そちらを使うようにマイグレーションすることもできます。BunのテストランナーはJest互換であるため、Jestのテストコードに加え、同じようにJest互換であるVitestのソースコードを実行できます。
下記のように、`vitest`パッケージから`describe`や`test`などをimportしている場合でも、ソースコードの変更なしで`bun test`を実行できます。

__`math.spec.ts`（テストファイル）__
```ts
import { describe, expect, test } from 'vitest'
import { add } from './math'

describe('math', () => {
  test('add', () => {
    expect(add(2, 3)).toBe(5)
  })
})

```

__コマンド__
```bash
bun test

# bun test v1.1.29 (6d43b366)

# src/utils/math.spec.ts:
# ✓ math > add [0.56ms]

#  1 pass
#  1 skip
#  0 fail
#  1 expect() calls
# Ran 2 tests across 2 files. [188.00ms]
```

BunのテストランナーはVitest向けの設定は読み込まないため、適宜Bun向けに書き直す必要があります。

## サンプル：SPA（Vite）プロジェクト

前提として、`npm create vite@latest`によって作成されたプロジェクトとします。フレームワーク・ライブラリはVueやReactを想定します。

### 依存関係のインストール

`bun install`を実行して必要なパッケージをインストールします。

### 開発サーバの起動

`bun --bun run dev`を実行して開発サーバを起動します。

Viteプロジェクトは`vite`コマンドで開発サーバの起動などを行いますが、このコマンドはNode.jsを前提として動作します。`--bun`オプションで、ランタイムとしてBunを選択させることができます。

※`package.json`の`scripts`の方を編集しても同様です。その場合、実行は単に`bun run dev`となります。

```json:package.json
{
  "scripts": {
    "dev": "bunx --bun vite",
    "build": "tsc -b && bunx --bun vite build"
  }
}
```

```bash
bun run dev
```

### ビルド

`bun --bun run build`でビルドを実行します。開発サーバの起動と同様、`--bun`オプションを付与します。

ブラウザ向けのビルドでViteを使用しているため、ビルド成果物は`npm run build`でも`bun --run run build`でも同じになります。

## サンプル：バックエンド（Next.js）プロジェクト

前提として、`npx create-next-app@latest`によって作成されたプロジェクトとします。

依存関係のインストール・開発サーバの起動・ビルドに関しては前項のViteプロジェクトの例と同様です。ただし、2024年10月現在リリースされているNext.js 15では、`--bun`オプションをつけて実行すると一部機能に制限がかかります。

```
Failed to install `require('node:crypto').randomUUID()` extension. When using `experimental.dynamicIO` calling this function will not correctly trigger dynamic behavior.
Failed to install `require('node:crypto').randomBytes(size)` extension. When using `experimental.dynamicIO` calling this function without a callback argument will not correctly trigger dynamic behavior.
```

## Tips

### シェルスクリプト

<!-- TODO -->


