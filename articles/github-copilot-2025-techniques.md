---
title: "GitHub Copilot テクニック集 2025年版"
emoji: "🤖"
type: "tech"
topics: ["githubcopilot", "vscode"]
published: true
published_at: 2025-12-05 07:00
---

## はじめに

2025年は AI 関連で大きな進展があった年でした。1月の DeepSeek R1がリリースされたのをきっかけに、GPT-4.1や Gemini 2.5Pro、下期には Sonnet 4.5や GPT-5、Gemini 3 Pro など次々と高度な LLM が登場し、AI アシスタントの性能も飛躍的に向上しました。
AI コーディングエージェントにもその影響は大きく、特に MCP の登場を始めとし、モデルの進化によってツールをしっかり使ってくれるようになったことによる恩恵が非常に大きいです。

もちろん GitHub Copilot でもその恩恵を受けており、様々な機能の追加によってより便利に使えるようになりました。
去年の終わりには GitHub Copilot はただのコード補完として使っていた人でも、現在では Agent モードを多用して自分で書くコードよりも AI に任せているコードのほうが多い、という人も少なくないのではないでしょうか。

この記事では GitHub Copilot にまだ慣れていない人がより便利に使えるようになるための、2025年版のテクニックを紹介します。

## 1. Beast Modeの有効化（VSCode）

設定から、`github.copilot.chat.alternateGptPrompt.enabled`を有効にすることで、Beast Mode と呼ばれるタスクの完遂能力が高いモードをデフォルトで利用できるようになります。
この機能の特徴として1回のタスクでの完遂能力が高くなるため、複雑なタスクでもプレミアムリクエストの消費を抑えることができます。

VSCode 以外を使用している場合は、以下のリポジトリのファイルを`.github/agents/beast-mode.agent.md`として保存し、このカスタムエージェントを使用することで同様の効果が得られます。
モデル名は適時書き換えてください。

https://github.com/github/awesome-copilot/blob/86adaa48fe3f8e9e0372cd5b18c52e5bb58f156b/agents/4.1-Beast.agent.md

## 2. カスタム指示を充実させる

リポジトリ内に特定のファイルを配置することで、デフォルトのカスタム指示を与えることができます。
カスタム指示によってコーディング規約をある程度守らせることができるので、最低限リポジトリに沿ったルールを記載している以下のファイルは作成しておくとよいです。

- `AGENTS.md`：基本的なエージェントの振る舞いを定義するカスタム指示
- `copilot-instructions.md`：GitHub Copilot code review にて使用するカスタム指示
- プロジェクトで扱っている言語の`.github/instructions/**.instructions.md`：各言語ごとのコーディング規約やスタイルを定義するカスタム指示

余裕があれば以下のファイルも作成しておくと、PR が作成しやすくなります。

- `copilot-commit-message-instructions.md`：コミットメッセージ生成用のカスタム指示
- `copilot-pull-request-instructions.md`：PR 説明文生成用のカスタム指示

詳細な設定内容については、以下の記事を参照してください。

https://qiita.com/yuzora_yu/items/83404fd51ba1faeeeca0

## 3. カスタムエージェントを作成する

`.github/agents/**.agent.md`にカスタムエージェントを作成することで、エージェントごとに使用する MCP ツールや振る舞いを設定できます。

例えば、「リポジトリの内容を説明するエージェント」「コードレビューをセキュリティの観点から厳しく行うエージェント」などが挙げられます。
カスタムエージェントのメリットは使い回せる点です。チーム内で共有すれば同じ振る舞いをさせられるため、特に複数人が関わるプロジェクトではコードの品質を保つ観点から有効です。

また MCP サーバーのツールを限定できるので、単純に Agent Mode にタスクを投げるよりもコンテキストが圧縮されます。結果として、性能の向上が期待できます。

## 4. テンプレートリポジトリを活用する

1-3までの設定をテンプレートリポジトリとして用意しておくと、次からはこれらの設定ファイルのコピペなしに新規プロジェクトを始められます。

https://docs.github.com/ja/repositories/creating-and-managing-repositories/creating-a-template-repository

設定は簡単で、リポジトリの Settings から「General」→「Template repository」にチェックを入れるだけです。

![リポジトリテンプレート](/images/2025/12/repository-template.png)

これで、リポジトリを新規作成するとき、テンプレートリポジトリに設定したファイルが用意された状態で始められます。

## 5. コマンド自動実行の有効化

`settings.json`、もしくはリポジトリの`.vscode/settings.json`に`chat.tools.terminal.autoApprove`を設定することで、GitHub Copilot がコマンドを自動で実行できるようになります。

```json
"chat.tools.terminal.autoApprove": {
    "fd": true,
    "rg": true
}
```

ファイルを壊さない検索系コマンドを追加しておくと便利になります。`fd` など代替コマンドを使用するときは、`AGENTS.md` でそのコマンドを使用するよう指示してください。

もう少し自由に使いたい場合は、リンター・フォーマッターの実行コマンドを追加するのも有効です。

但し `rm` などの **ファイル操作、特に削除系コマンドを追加してはいけません**。最悪の場合、重要なファイルが消えます。

## 6. issueテンプレートを活用する

:::message
VSCode から直接 Coding Agent にタスクを投げられる、またカスタムエージェントのカスタム指示を使用できるようになったので優先度は下がりました。
:::

GitHub のランナーで動作する GitHub Copilot Coding Agent にタスクを投げる場合、issue テンプレートを用意しておくと複雑なプロンプトをいちいち考えなくても定型タスクが実行できて便利です。

https://docs.github.com/ja/communities/using-templates-to-encourage-useful-issues-and-pull-requests/configuring-issue-templates-for-your-repository

「モジュールのバージョン更新作業」「エラーハンドリングの追加」などが簡単なタスクとして挙げられます。

## おわりに

2025年の GitHub Copilot は Agent Mode の普及とともに Co-pilot ではなく Pilot と言えるほどに、AI が主導してコードを書く機会が増えた年となりました。
それでも依然として、指示の仕方を間違えると期待したコードが得られないことも多いです。

今回は主に、AI に正しい指示を与えるための土台作りに関するテクニックを紹介しました
2026年にはこれらのテクニックも不要になっている可能性があります。AI がどのように進化するのか楽しみです。
