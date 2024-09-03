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