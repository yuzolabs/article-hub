# article_hub

Qiita/Zennの記事を保存し、自動で投稿するリポジトリです。

## 開発環境のセットアップ

### pre-commitの導入

このプロジェクトでは、コミット前に自動でコードの品質をチェックする仕組みとして[pre-commit](https://pre-commit.com/)を導入しています。

markdownlint, textlintについては自動修正が困難になる可能性があるので、GitHub Actionsで検知することを予定しています。

#### 初回セットアップ

pre-commitをインストールし、リポジトリに設定します。

```bash
# pre-commitのインストール(pipを使用)
pip install pre-commit

# pre-commitフックの有効化
pre-commit install
```

#### 実行されるチェック項目

コミット時には以下のチェックが自動で行われます。

- check-yaml — YAMLファイルの構文チェック
- end-of-file-fixer — ファイル末尾に改行を追加
- trailing-whitespace — 行末の不要な空白を削除
- check-json — JSONファイルの構文チェック
- check-toml — TOMLファイルの構文チェック
- detect-private-key — 秘密鍵の誤コミットを防止
- no-commit-to-branch — mainブランチへの直接コミットを防止

#### 手動でチェックを実行する

コミットせずに、すべてのファイルに対してチェックを実行したい場合は次のコマンドを使います。

```bash
pre-commit run --all-files
```

## Lintツール

このプロジェクトでは、以下のLintツールを使用しています。

```bash
# textlintの実行
npx textlint "path/to/markdown.md"

# markdownlint-cli2の実行
npx markdownlint-cli2 --fix "path/to/markdown.md"
```
