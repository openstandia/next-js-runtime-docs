## 概要

<!-- Bunの特徴のサマリ -->

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

<!-- TODO: 既存のNode.jsプロジェクトへの導入は別ドキュメントにする -->

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

<!-- Bun標準のテストランナー -->

## Bun Shell

<!-- Bunでシェルスクリプトを書く機能 -->