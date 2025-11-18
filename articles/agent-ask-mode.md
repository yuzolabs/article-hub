---
title: "Askモードを拡張したAgent-Askエージェントを使ってみよう"
emoji: "🤖"
type: "tech"
topics:
  - "githubcopilot"
  - "vscode"
  - "agent"
published: true
published_at: 2025-12-01 07:00
---

## はじめに

GitHub Copilot には「Ask」「Edit」「Plan」「Agent」の4つのモードがあります。

その中のAskモードはちょっとした質問には便利ですが、MCPサーバーや組み込みツールを使用できません。
そのため、LLMのカットオフ知識以降の情報を取得したり、コマンド実行を伴う外部の情報を参照する質問には不向きです。

この記事では、この問題を解消するためのAskモードを拡張した **「Agent-Ask」カスタムエージェント** について紹介します。

## AgentモードをAskモードの代替として使えば良いのでは？

ごもっともな意見です。

今のAgentモードは既に「Edit」モードを使用する意味が無いほどに高度な機能を持っています。しかし、AgentモードをAskモードの代替として使用するには致命的な弱点があります。

それは、Agentモードでは**指示していないのに勝手にファイルの修正する事がある**、ということです。
「計画案だけ出すように言ったのに勝手に実装された」「レビューを指示したのに勝手に修正された」といった経験をした方も多いのではないでしょうか。

このため、現状ではAgentモードを単純にAskモードの代替として使うことは難しいと言えます。

## Agent-Askモード

そこで提案するのが、**Agent-Ask** というカスタムエージェントです。

カスタムエージェント機能の基本的な使い方については、以下の記事で詳しく解説しています。

https://qiita.com/yuzora_yu/items/83404fd51ba1faeeeca0#%E3%82%AB%E3%82%B9%E3%82%BF%E3%83%A0%E3%83%81%E3%83%A3%E3%83%83%E3%83%88%E3%83%A2%E3%83%BC%E3%83%89%E3%82%AB%E3%82%B9%E3%82%BF%E3%83%A0%E3%82%A8%E3%83%BC%E3%82%B8%E3%82%A7%E3%83%B3%E3%83%88

簡単に解説しますと、カスタムエージェント機能とはAgent機能を使う際に与えるプロンプトをあらかじめ設定しておくものです。

この際、使用するMCPサーバーのtoolsや組み込みツールを自由に指定できます。
つまり、ファイルを修正するツールをAgentモードから取り除けば、Agentモードの振る舞いを維持しながら、ファイルを変更しない「拡張版Askモード」を作れるわけです。

## Agent-Askの設定ファイル

私が使用しているAgent-Askエージェントの設定ファイルになります。
リポジトリ内で使用する場合は`.github/agents/Agent-Ask.agent.md`として保存するとそのまま使用できます。

```markdown
---
description: 'Agent機能を維持しつつ、ファイルを修正しない'
tools: ['runCommands', 'runTasks', 'search', 'new', 'extensions', 'usages', 'vscodeAPI', 'problems', 'changes', 'testFailure', 'openSimpleBrowser', 'fetch', 'githubRepo', 'github.vscode-pull-request-github/copilotCodingAgent', 'github.vscode-pull-request-github/issue_fetch', 'github.vscode-pull-request-github/suggest-fix', 'github.vscode-pull-request-github/searchSyntax', 'github.vscode-pull-request-github/doSearch', 'github.vscode-pull-request-github/renderIssues', 'github.vscode-pull-request-github/activePullRequest', 'github.vscode-pull-request-github/openPullRequest', 'ms-python.python/getPythonEnvironmentInfo', 'ms-python.python/getPythonExecutableCommand', 'ms-python.python/installPythonPackage', 'ms-python.python/configurePythonEnvironment', 'todos', 'runSubagent']
name: "Agent-Ask"
---

ユーザーから指示された内容を遂行すること。但し、ファイルに変更を加えることを禁止する。

```

プロンプトにもファイル修正を禁止する文言を加えることで、`runCommands`によってコマンドからファイルの追加を行うことを防いでいます。
MCPサーバーを使用する際は、適時必要なツールを追加してください。

## おわりに

本記事では、Askモードを拡張したAgent-Askエージェントについて紹介しました。

Agentモードが出てからしばらく経ちますが、読み取り専用のAgentモードが公式実装されないことから個人的にこの機能は今後も実装されないのかなと感じています。

このカスタムエージェントを使用して、AIが暴走しない体験をぜひ味わってみてください。
