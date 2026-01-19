# article_hub

Qiita/Zenn の記事を保存し、自動で投稿するリポジトリです。

## 自動投稿機能

このリポジトリは、Zenn/Qiita への記事の自動投稿機能を備えています。

### セットアップ

#### 1. QiitaアクセストークンをGitHub Secretsに設定

1. Qiita のアクセストークンを取得します。
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

記事は Zenn 形式で記述します。事前に bun をインストールしてください。

```bash
bun zenn new:article --slug 記事のスラッグ --title タイトル --type tech --emoji ✨
```

オプション：

- `--slug` - 記事のスラッグ（12〜50文字の英数字、ハイフン、アンダースコア）
- `--title` - 記事のタイトル
- `--type` - 記事の種類（`tech` または `idea`）
- `--emoji` - 記事の絵文字

#### 記事をプレビューする

```bash
bun zenn preview
```

[http://localhost:8000](http://localhost:8000) でプレビューを確認できます。

### 記事の投稿

1. `articles/` ディレクトリ内に記事を作成します。
2. フロントマターで `published: true` を設定します。
3. PR を作成し、`main` ブランチにマージします。
4. main ブランチにマージすると、以下の処理が自動的に実行されます。
   - Zenn には GitHub App 経由で記事が公開されます。
   - GitHub Actions が Qiita 形式に変換し、Qiita にも投稿します。

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

- Bun 1.x
- Python 3.x（pre-commit 用）

### pre-commitの導入

このプロジェクトでは、コミット前に自動でコードの品質をチェックする仕組みとして[pre-commit](https://pre-commit.com/)を導入しています。

markdownlint, textlint については自動修正が困難になる可能性があるので、GitHub Actions で検知することを予定しています。

#### 初回セットアップ

pre-commit をインストールし、リポジトリに設定します。

```bash
# pre-commitのインストール(pipを使用)
pip install pre-commit

# pre-commitフックの有効化
pre-commit install
```

#### 実行されるチェック項目

コミット時には以下のチェックが自動で行われます。

- `check-yaml` - YAML ファイルの構文チェック
- `end-of-file-fixer` - ファイル末尾に改行を追加
- `trailing-whitespace` - 行末の不要な空白を削除
- `check-json` - JSON ファイルの構文チェック
- `check-toml` - TOML ファイルの構文チェック
- `detect-private-key` - 秘密鍵の誤コミットを防止
- `no-commit-to-branch` - main ブランチへの直接コミットを防止

#### 手動でチェックを実行する

コミットせずに、すべてのファイルに対してチェックを実行したい場合は次のコマンドを使います。

```bash
pre-commit run --all-files
```

## Lintツール

このプロジェクトでは、以下の Lint ツールを使用しています。

```bash
# textlintの実行
bun textlint "path/to/markdown.md"

# markdownlint-cli2の実行
bun markdownlint-cli2 --fix "path/to/markdown.md"
```
