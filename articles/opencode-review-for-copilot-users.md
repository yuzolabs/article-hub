---
title: "OpenCodeをGitHub Copilot ユーザーが使ってみた"
emoji: "🔍"
type: "tech"
topics: ["opencode", "開発効率化", "CLI", "githubcopilot"]
published: true
---

## はじめに

最近、X 上で話題になっている AI コーディングツールの OpenCode を使ってみました。
普段 GitHub Copilot（CLI ではなく、VSCode 上の拡張機能）を使用しているので、どちらがより使いやすいかを比較レビューします。

:::message alert
現在、OpenCode でサブスクリプション連携をサポートしているプロバイダについて公式の連携方法ではない手段を用いているものが何個かあります。
Antigravity がこれに該当し、使用するとこれらのサービスの利用規約に抵触する可能性があります。

利用する場合は、正式に連携している Codex や Z.ai などのサブスクリプションを使用するか、API キーを用いて連携することを **強く** 推奨します。
:::

:::message
2026/01/16追記：
GitHub Copilot のサブスクリプションが正式に OpenCode でサポートされました！
v1.1.21以降で利用可能です。
:::

## GitHub Copilotの機能との比較

まず、GitHub Copilot で使える各機能と OpenCode における対応表を以下に示します。

| GitHub Copilot | OpenCode | 補足 |
| :--- | :--- | :--- |
| **.github/prompts** | `.opencode/commands` | カスタムスラッシュコマンド |
| **.github/instructions** | 明確な対応機能なし | カスタム命令<br>OpenCode では `AGENTS.md` などに記述し、条件分岐を設定して読み込ませる運用になります。 |
| **.github/agents** | `.opencode/agent/` | カスタムエージェント |
| **.github/skills** | `.opencode/skill/` | Agent Skills |

GitHub Copilot にはない OpenCode 独自の特徴として、以下の 2 点が挙げられます。

- 自動フォーマット：ファイル編集後に自動で整形ツールを実行できます
- プラグイン（Hook）：独自のスクリプトを拡張機能として実行できます

それぞれの機能について、詳しく見ていきます。

### カスタムスラッシュコマンド

GitHub Copilot の `.github/prompts` に相当するのが `.opencode/commands` です。
ここに登録したコマンドは、エディタ上（または OpenCode のインターフェース上）で`/`を入力することで呼び出せます。

設定ファイルの例は以下の通りです。

```md:.opencode/commands/<command-name>.md
---
description: <カスタムスラッシュコマンド名>
agent: <カスタムエージェントに委譲する場合のエージェント名>
---

<template>
```

テンプレート部分には、そのコマンドで何を実行するかを記述します。
ここで重要なのは `$ARGUMENTS` 変数です。この変数を含めておかないと、スラッシュコマンドを実行した際にユーザーからの引数を受け取ることができません。

例えば、`/explain`というカスタムスラッシュコマンドを作成した時に、`/explain hoge.py` と入力することで `hoge.py` を解説してほしい時は以下のような template になります。

```md:.opencode/commands/explain.md
---
description: ファイルの内容を解説する
---
$ARGUMENTS のパスにあるファイルを読み込み、その動作を解説して
```

### カスタムエージェント

GitHub Copilot の `.github/agents` に相当するのが `.opencode/agent/` です。

GitHub Copilot の場合、一覧からエージェントを選択して切り替えますが、OpenCode では Tab キーでエージェントを切り替えます。

全てのカスタムエージェントを `primary` モードに設定してしまうと、Tab キーでの切り替え候補が増えすぎて操作が煩雑になります。
そのため、使用頻度の高いエージェントだけを `primary` にし、そうでないものはカスタムスラッシュコマンド経由で呼び出す、といった運用が良さそうです。

設定ファイルの例は以下の通りです。

```md:.opencode/agent/<agent-name>.md
---
description: <カスタムエージェントの説明>
mode: <primary | subagent>
model: <実行するLLMモデル名>
permissions: # 使用するツールのリスト
  read: allow
  list: allow
  write: false
  edit: false
  bash: false
---

<prompt>
```

特に注目したいのは、`permissions` でツールの使用許可を明示的に指定できる点です。
例えば、「コードの提案はしてほしいが、ファイルの書き換え（write）やコマンド実行（bash）は許可したくない」といった相談役のエージェントを作る場合、これらを `false` に設定することで実現できます。
特に、bash コマンドについて個別に設定が可能なので、GitHub Copilot よりも安全側に倒した設計が可能と言えます。

活用方法として、あるカスタムエージェントに全部のコマンドを許容するように設定をすることで DevContainers などの独立した環境では自律して動作させるようにしつつ、別のカスタムエージェントではコマンドに制限をかけて安全のバランスを取る運用ができます。

### Agent Skills

GitHub Copilot の `.github/skills` に相当するのが `.opencode/skill` です。

Agent Skills の形式は共通化されているので、GitHub Copilot のものをそのまま流用できます。

```md:.opencode/skill/<skill-name>/SKILL.md
---
name: <Agent Skills 名>
description: <Agent Skills の説明>
---

実際のスキルをここに記述
```

作成したスキルは、必要なカスタムエージェントにのみ紐付けて許可できます。
また、スキル名に接頭辞（prefix）などを付けて一括使用を許可するなど、管理を楽にする機能もあります。

### 自動フォーマット機能

AI がコードを生成・修正した直後に、プロジェクトのルールに合わせて自動で整形してくれます。

上で紹介した機能と違い、設定は `opencode.json` で行います。

```json:opencode.json
{
  "$schema": "https://opencode.ai/config.json",
  "formatter": {
    // 独自のフォーマッターを追加・変更する
    "my-formatter": {
      "command": ["npx", "prettier", "--write", "$FILE"],
      "extensions": [".js", ".ts"],
      "environment": {
        "NODE_ENV": "development"
      }
    }
  }
}
```

`$FILE` という変数が、実際に編集されたファイルのパスに置き換わってコマンドが実行されます。
GitHub Copilot でも Agent Skills やカスタム指示で近いことはできますが、機能として組み込まれているのは OpenCode の強みです。

### プラグイン（Hook）

さらに高度なカスタマイズとして、任意のタイミングでコマンドを実行できるプラグイン（Hook）の仕組みがあります。
定義自体は JavaScript または TypeScript で行いますが、その中から Python や Shell スクリプトなどを呼び出せるため、実質的に何でもできる拡張性を持っています。
公開されているプラグインの利用も可能です。

まずは、「1つのタスクが終わったら音を鳴らす」といった簡単なプラグインを導入してみるのがおすすめです。
これを自作できるような人は OpenCode を更に便利に使うことができるはずです。

## GitHub Copilot との差別化

基本的に CLI 系ツールということを考慮しなければ、**OpenCode は GitHub Copilot よりも多くの機能が備わっていると感じています。**

しかし、OpenCode を一般企業の開発環境に導入するのはハードルが高いように感じられました。

理由としては 3 点あり、

- 使用できる MCP やプラグインを管理する機能が備わっておらず、自由度が高すぎるため（一般企業レベルでは）セキュリティ面で考慮することが多すぎる
- 無料の LLM を使えてしまい、誤って選択して使用した時にデータを学習される恐れがある
https://opencode.ai/docs/zen/#privacy
- ~~企業で使えるレベルの正式なサブスクリプション連携が存在しない~~
**（Codex 連携が追加されたので使いやすくなりました）**

といった点が挙げられます。

しかし、機密データを扱わない個人開発などセキュリティリスクをある程度無視できる場合は、OpenCode の自由度の高さを活かすことができるため、非常に有用な AI エージェントツールとなります。
安全に使用するために、無制限に API キーを用いたリクエストが許されるエンジニアは限られているため、今後のサブスクリプション連携の充実に期待しています。

## まとめ

個人的には、GitHub Copilot を OpenCode へ全面的に置き換えるのは、まだ先になると考えています。

GitHub Copilot の大きなメリットは、様々な LLM モデルを安価に利用できる点ですが、現状の OpenCode ではまだ実現できていません。

~~公式に GitHub Copilot のサブスクリプション連携をサポートする予定と発表されたため、今後に期待したいです。~~

2026/01/16 追記：
GitHub Copilot のサブスクリプション連携が正式にサポートされました！

https://x.com/github/status/2011822451613712646
