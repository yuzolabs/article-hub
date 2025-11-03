---
title: "copilot-instructions.mdだけじゃない！GitHub Copilotのカスタム指示まとめ"
emoji: "⚙️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["githubcopilot", "vscode"]
published: true
---

ちょっと前に`copilot-instructions.md`を作成すると良い、という記事が話題になりました。

https://qiita.com/TooMe/items/873540da84567733d16b

実はこれ以外にも、GitHub Copilotには設定しとくとよいカスタム指示ファイルがいくつかあります。

この記事ではそれらを紹介していきます。

:::message
この記事ではVSCodeの設定について説明します。他のIDEを使用している場合は使用できない設定もあるので注意してください。
:::

## コードルールに関する指示

プロジェクト全体に適用したいルールやガイドラインを記述します。

### `.github/copilot-instructions.md`

リポジトリのルート直下にある`.github/`ディレクトリへ配置する、GitHub Copilotにおいて最も基本的なカスタム指示ファイルです。

また、このルールがGitHub Copilot code review（GitHub上でCopilotをレビュワーにできる機能）にも適用されるので、レビュー目線のルールを加えておくのも良いです。

詳細な書き方については上記記事で紹介されているので、ここでは割愛します。

### ファイルの種類に応じたカスタム指示

他のコーディングエージェントを使用せず、GitHub Copilotだけを使用する場合は`[NAME].instructions.md`に詳細なルールを記述することをおすすめします。
このカスタム指示では、**特定のファイルパターンにのみ適用される指示**を記述できます。
このルールもGitHub Copilot code reviewに適用されます。

`.github/instructions/`ディレクトリに、`[NAME].instructions.md`という名前で配置し、ファイルの先頭に以下を付け加えます。
`[NAME]`の部分は何でも良いですが、Pythonに関するカスタム指示では`python.instructions.md`などとするとわかりやすいです。

```markdown
---
applyTo: "glob-pattern"
---
```

`applyTo`フィールドでは、以下のようなglobパターンを指定できます。

- `"**/*.py"` - すべてのPythonファイル
- `"test/*.py"` - `test`ディレクトリ内のすべてのPythonファイル
- `"**/*.ts,**/*.tsx"` - すべてのTypeScriptファイル（複数指定可能）

このカスタム指示を設定しておくと、`.github/copilot-instructions.md`のルールを分離できるので管理が楽になるだけではなく、GitHub Copilotに与えるコンテキストサイズが小さくなるのでパフォーマンス向上も期待できます。

具体的には、Pythonプロジェクトではファイルに変更を加えた後にRuffリンターを適用したり、MarkdownファイルではMarkdownlintを適用したりといったルールを個別に設定できます。

### `AGENTS.md`

GitHub Copilotだけではなく、CodexやCursorなどでも採用されているエージェント向けのカスタム指示ファイルです。
GitHub CopilotではAgentモードだけではなく、Askモードでも採用されます。

プロジェクト内で他のコーディングAIを採用している場合は、`.github/copilot-instructions.md`に記述する内容を`AGENTS.md`に配置しておくと小回りが利いて便利です。
その場合、`.github/copilot-instructions.md`はGitHub Copilot code review用として運用するのが良いでしょう。

ちなみに、`CLAUDE.md`や`GEMINI.md`といったモデル固有のカスタム指示もGitHub Copilotではサポートされています。

## カスタムスラッシュコマンド

カスタムスラッシュコマンドは、あらかじめ登録したプロンプトを`/`から始まるコマンド形式で呼び出す機能です。

### ファイル構成と保存場所

プロンプトファイルはMarkdownファイルで、`.prompt.md`拡張子を使用します。

保存場所は以下の2つから選択できます。

| タイプ | 保存場所 | 対象範囲 |
|--------|---------|---------|
| ワークスペース | `.github/prompts`フォルダ | リポジトリ内 |
| ユーザー | VS Codeプロファイルフォルダ | 複数のワークスペース |

**基本的にはワークスペース内に含めておきたい**ですが、プロジェクトに依らず使用できるカスタムスラッシュコマンド（自分の考えた観点でレビューさせたいなど）はユーザープロンプトファイルとして保存するのも良いです。

GitHub公式の[awesome-copilot](https://github.com/github/awesome-copilot/tree/main/prompts)リポジトリでは、様々なカスタムスラッシュコマンドが公開されているのでこれらをコピペするだけでも便利に使えます。

### プロンプトファイルの例

例として、指定したファイルをわかりやすく説明するプロンプトファイル（`.github/prompts/explain-code.prompt.md`）を紹介します。
このカスタムスラッシュコマンドを呼び出すには、`/explain-code <対象ファイルのパス>`と入力します。

```markdown
---
mode: 'agent'
description: '指定したパスのファイルをわかりやすく説明する。'
model: 'Grok Code Fast 1'
tools: ['search', 'todos']
---

提示したパスのファイルについて、初心者にもわかりやすく説明してください。

以下の内容を含めてください。

- このコードが何をするものかの概要
- 主な関数ごとの手順や流れの説明

できるだけやさしい言葉で、専門用語は必要最低限にしてください。

```

各プロパティが表すものは以下の通りです。

- mode: 'agent' | 'ask' | 'edit' のいずれかを指定します。基本的には'agent'を使用します。
- description: プロンプトの簡単な説明を記述します。
- model: 使用するモデルを指定します。指定しない場合は現在指定しているモデルが使用されます。
- tools: 使用するツール・MCPを指定します。

このカスタムスラッシュコマンドでは、toolsへ`edit`を指定していないので、もしLLM側でファイルを修正する指示が出されてもカスタムスラッシュコマンドの設定によって無効化されます。

## カスタムチャットモード（カスタムエージェント）

Agentモードでよく使用するプロンプトを設定しておくことで、毎回同じ内容を入力する手間を省けます。
振る舞いとしてはカスタムスラッシュコマンドの`mode: agent`にした時とほぼ同じですが、こちらはスラッシュコマンドを忘れても「モードの設定」リストから選択できるメリットがあります。

こちらもGitHub公式の[awesome-copilot](https://github.com/github/awesome-copilot/tree/main/chatmodes)リポジトリには様々なカスタムチャットモードが公開されています。
特に、[Beast Mode](https://github.com/github/awesome-copilot/blob/main/chatmodes/4.1-Beast.chatmode.md)についてはGitHub Copilot界隈で話題になり、そのプロンプトが公式に採用される見込みです。

https://x.com/code/status/1955322927886274928

### 例

以下のファイルを`.github/chatmodes/create-readme.chatmode.md`として保存します。

:::message
Insider版ではパス名を`.github/agents/create-readme.agent.md`にします。
今後通常版でもこのディレクトリに移行すると思われます。
:::

```markdown
---
description: 'リポジトリの内容を元にREADMEを作成する'
tools: ['edit', 'search', 'todos']
model: Claude Sonnet 4.5
---
このリポジトリの内容を検索し、以下の内容を含んだ`README.md`ファイルを作成する。既に`README.md`ファイルが存在する場合は、その内容を改善すること。

- プロジェクトの目的
- 主な機能や特徴
- インストール方法
```

このチャットモードを使用して、「指示に従いREADME.mdを作成して。」と指示すると上記の内容でREADME.mdが生成されます。
カスタムスラッシュコマンドと同様で、`fetch`のようなtoolsを使用できなくしているため、無駄な情報を取得せずに動作させることができます。

## settings.jsonでの設定

`settings.json`で設定できるカスタム指示関連の設定を紹介します。

### コミットメッセージの設定

GitHub Copilotではサイドバーの「ソース管理」→「コミットメッセージの生成」から自動でコミットメッセージを生成できます。

![コミットメッセージの生成](/images/2025/11/copilot-instruction-guide/commit-message.png)

この時使用されるカスタム指示の設定方法としては2つあります。

#### `settings.json`で直接指示を記述する方法

`settings.json`に以下を追加します。

```json
{
  "github.copilot.chat.commitMessageGeneration.instructions": [
    { "text": "日本語で出力すること" }
  ]
}
```

これによって、GitHub Copilotで自動生成するコミットメッセージが日本語になります。

#### `settings.json`で指示ファイルを指定する方法

長いプロンプトを指定したい場合は、`settings.json`の管理面から指示ファイルで指定する方法がおすすめです。

```json
{
  "github.copilot.chat.commitMessageGeneration.instructions": [
    { "file": ".github/copilot-commit-message-instructions.md" }
  ]
}
```

以下はConventional Commits規約に準拠したコミットメッセージを生成するためのカスタム指示ファイル例です。
これを`.github/copilot-commit-message-instructions.md`として保存します。

```markdown
コミットメッセージは Conventional Commits 規約に準拠すること。

## フォーマット

<type>[optional scope]: <description>

[optional body]

[optional footer(s)]

## Type

- `feat`: 新機能の追加
- `fix`: バグ修正
- `docs`: ドキュメントのみの変更
- `style`: コードの意味に影響しない変更(空白、フォーマット、セミコロンの欠落など)
- `refactor`: バグ修正も機能追加もしないコード変更
- `perf`: パフォーマンスを向上させるコード変更
- `test`: 不足しているテストの追加や既存のテストの修正
- `build`: ビルドシステムや外部依存関係に影響する変更(例: npm, webpack)
- `ci`: CI設定ファイルやスクリプトの変更
- `chore`: その他の変更(src や test ファイルを変更しないもの)
- `revert`: 以前のコミットを取り消す

## 基本ルール

- descriptionは日本語で50文字以内とすること。
- 破壊的な変更の場合は、`!`をtypeの後に追加すること。

## 例

feat: ユーザー認証機能を追加

JWTベースの認証システムをリフレッシュトークン付きで実装。

```

これにより、Conventional Commits規約に準拠したコミットメッセージが生成されるようになります。

### PRメッセージの設定

「GitHub Pull Requests」拡張機能を導入すると、PRメッセージの自動生成機能が使用できます。
左サイドバー「GitHub Pull Requests」→「プルリクエストの作成」から使用できます。

PRメッセージ生成用のカスタム指示も、コミットメッセージ生成用と同様に`settings.json`で直接記述する方法と指示ファイルを指定する方法があります。

但し、こちらは思ったよりも指示に従ってくれない印象があります。これ専用のカスタムチャットモードを作成したほうが良いかもしれません。

```json
{
  "github.copilot.chat.pullRequestDescriptionGeneration.instructions": [
    { "text": "日本語で、変更したファイルの説明を記載すること" }
  ]
}
```

```json
{
  "github.copilot.chat.pullRequestDescriptionGeneration.instructions": [
    { "file": ".github/copilot-pr-description-instructions.md" }
  ]
}
```

:::details インラインレビュー機能について（非推奨）

GitHub Copilotには`github.copilot.chat.reviewSelection.instructions`という設定が存在します。
この設定を使用すると、コードを選択した際のインラインレビュー機能にカスタム指示を追加できます。

```json
{
  "github.copilot.chat.reviewSelection.instructions": [
    { "text": "セキュリティの観点からコードをレビューすること" }
  ]
}
```

しかし、この機能の使用は個人的には推奨しません。理由は以下の通りです。

- chatmodeでレビューするとMCPツールも使用でき、より高度なレビューが可能になる
- 外部の情報を取得できないので、レビューの質がLLMモデルのカットオフに依存してしまう（例としてSonnet 4.5は存在しないなどを指摘されてしまう）

:::

## まとめ

GitHub Copilotには`.github/copilot-instructions.md`だけでなく、さまざまなカスタム指示ファイルと設定方法があります。

特にカスタムスラッシュコマンドやカスタムチャットモードについては一度設定しておくと何度も入力する長いプロンプトを省けるのでとても便利です。

これらの設定をチーム内で共有して、日々の作業の効率と品質を上げていきましょう。
