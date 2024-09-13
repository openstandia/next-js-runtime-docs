## 概要

<!-- Bunの特徴のサマリ -->

## セットアップ

<!-- 開発を始めるのに必要な操作 -->

### インストール

<!-- 開発端末へのインストール方法 -->

### 設定ファイル

<!-- CLIなどのための設定ファイルの種類と記法 -->

## TypeScript

<!-- TypeScriptで書くために必要な設定 -->

## CLI

<!-- Bunで使えるコマンド一覽 -->

## パッケージマネージャ

<!-- パッケージのインストール方法やバージョン管理方法 -->

## 標準ライブラリ・API

<!-- Bunが提供する標準ライブラリやAPIの一覽 -->

## npmパッケージの利用

Bunでは、Node.jsとほぼ同様にnpmパッケージを利用できます。

### パッケージのインストール

`bun add <npmパッケージ名>`でnpmパッケージをインストールできます。パッケージはプロジェクトルート（`bun add`を実行したディレクトリ）配下の`node_modules`ディレクトリに格納されます。

```shell
bun add zod
```

npmでインストールするときと同様、バージョンを指定することができます。

```shell
bun add zod@3.20.0
```

開発用にインストールする場合は`-d`オプションを付与します。

```shell
bun add -d prettier
```

※`--dev`や`-D`でも同様です。

### パッケージのインポート

Node.jsと同様に`import`文を書けます。

```js
import figlet from "figlet"
import { z } from 'zod'
```

### バージョン管理

Node.jsと同様に、インストールしたnpmパッケージは`package.json`で管理されます。

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

## 環境変数

<!-- 環境変数の読み込み方 -->

## ビルド

<!-- 各環境向けのビルド方法 -->

### SPAなどのブラウザ向け

### Node.js向け

### Bun向け

## プラグイン

<!-- Bunプラグインの書き方 -->

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

<!-- Bunでシェルスクリプトを書く機能 -->