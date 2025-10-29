# article_hub

Qiita/Zennの記事を保存し、自動で投稿するリポジトリです。

## 自動投稿機能

このリポジトリは、Zenn/Qiitaへの記事の自動投稿機能を備えています。

### セットアップ

#### 1. QiitaアクセストークンをGitHub Secretsに設定

1. Qiitaのアクセストークンを取得します。
2. リポジトリの `Settings` → `Secrets and variables` → `Actions` にアクセスします。
3. `New repository secret` をクリックします。
4. Name: `QIITA_TOKEN`、Secret: コピーしたトークンを入力します。
5. `Add secret` をクリックします。

#### 2. Zenn連携の設定

1. [Zenn](https://zenn.dev/)にログインします。
2. ダッシュボードから「リポジトリ連携」を選択します。
3. このリポジトリを連携します。

### 記事の作成

#### 新しい記事を作成する

記事はZenn形式で記述します。

```bash
npx zenn new:article --slug 記事のスラッグ --title タイトル --type tech --emoji ✨
```

オプション：

- `--slug` - 記事のスラッグ（12〜50文字の英数字、ハイフン、アンダースコア）
- `--title` - 記事のタイトル
- `--type` - 記事の種類（`tech` または `idea`）
- `--emoji` - 記事の絵文字

#### 記事をプレビューする

```bash
npx zenn preview
```

[http://localhost:8000](http://localhost:8000) でプレビューを確認できます。

### 記事の投稿

1. `articles/` ディレクトリ内に記事を作成します。
2. フロントマターで `published: true` を設定します。
3. PRを作成し、`main` ブランチにマージします。
4. mainブランチにマージすると、以下の処理が自動的に実行されます。
   - ZennにはGitHub App経由で記事が公開されます。
   - GitHub ActionsがQiita形式に変換し、Qiitaにも投稿します。

### ディレクトリ構造

```text
.
├── articles/        # Zenn記事（Markdown形式）
├── books/          # Zennブック（オプション）
├── images/         # 記事内で使用する画像
└── qiita/
    └── public/     # Qiita記事（自動生成）
```

## 開発環境のセットアップ

### 必要な環境

- Node.js 18.0.0以上
- Python 3.x（pre-commit用）

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

- `check-yaml` - YAMLファイルの構文チェック
- `end-of-file-fixer` - ファイル末尾に改行を追加
- `trailing-whitespace` - 行末の不要な空白を削除
- `check-json` - JSONファイルの構文チェック
- `check-toml` - TOMLファイルの構文チェック
- `detect-private-key` - 秘密鍵の誤コミットを防止
- `no-commit-to-branch` - mainブランチへの直接コミットを防止

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
