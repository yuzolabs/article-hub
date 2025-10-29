---
title: "GitHub Copilot Pro+でCodexが使えるようになったので試してみた"
emoji: "✨"
type: "tech"
topics: ["githubcopilot", "vscode", "codex"]
published: true
published_at: 2025-10-30 07:00
---

## 三行要約

- GitHub Universe 2025でAgent HQが発表され、Copilot Pro+ライセンスでCodexが使えるようになった
- 現状は`medium`モデルのみ利用可能で、クラウド版やWeb検索機能には非対応
- まだpublic reviewの段階で制限が多く、今のところこれ目的でPro+を契約する意味は薄い

## はじめに

GitHub Universe 2025にてAgent HQが発表されました。
これは、GitHub CopilotのライセンスでCodexやClaude Codeなどの他社のAIコーディングエージェントが使用できるようになるというものです。

https://x.com/github/status/1983205334756839605

第一弾として、VS Code InsidersからCodexが使用できるようになっています。

本記事では、実際にGitHub Copilot Pro+でCodexを試してみた感想をまとめます。

## 前提条件

- **GitHub Copilot Pro+**
- VS Code Insiders（プレビュー版）

現状ではEnterpriseプランですら使用できず、対応しているのがGitHub Copilot Pro+のみなので、導入ハードルは非常に高いです。

## セットアップ手順

### 1. OpenAIアカウントからのログアウト（既存ユーザーの場合）

既にCodexをOpenAIアカウントで使用している場合は、一度ログアウトしてください。
拡張機能の「Settings」 → 「Log out」を選択します。

まだCodexを使用していない場合は「Codex – OpenAI’s coding agent」の拡張機能をインストールしてください。

### 2. GitHub Copilotでのサインイン

拡張機能の画面にログイン画面が表示されるので、「Sign in with GitHub Copilot」を選択します。

![ログイン画面](/images/2025/10/github-copilot-proplus/codex_login.png)

使用するにあたり、注意事項が表示されます。
特に、CodexとGitHubの両方の利用規約に同意する必要があることに注意してください（企業ライセンスで使用できないので問題ないとは思いますが……）。

![注意事項](/images/2025/10/github-copilot-proplus/codex_notification.png)

この手順を終えるとCodexが使用可能になります。

![セットアップ完了](/images/2025/10/github-copilot-proplus/codex_only_medium.png)

## 実際に使ってみた感想

実際に使用してみましたが、先の画像でも分かる通り **`low`と`high`モデルが使用できず**、`medium`モデルのみが利用可能です。
ちょっと残念な感じがします。

ちなみに使用するプレミアムリクエストは1リクエストで1プレミアムリクエスト消費されるようです。
本家のCodexはPlusプランでもそこそこ`high`モデルが使用できるので、2プレミアムリクエストくらい消費して`high`モデルが使えれば良かったのですが……

なお、制限事項は他にもあり、以下の通りです。

- クラウド版のCodexは使用できません。よって、GitHub上のコードレビュー機能も使用できません。
- **Web検索機能（`web_search=True`）が使用できません。** 以下のエラーになります。

```txt
stream disconnected before completion: stream closed before response.completed
```

逆に、これ以外の使用感は特に変わりません。
検証していませんが、本場のCodexと違い5h制限や週制限のタブがないのでもしかすると集中して使用してもCopilot Pro+のリクエスト制限内であれば問題ないのかもしれません。

## まとめ

GitHub Copilot Pro+でOpenAI Codexが使えるようになりましたが、まだまだpublic reviewということもあり発展途上な感じが否めないです。
CodexのPlusプランと同じものは提供できない可能性がありますが、それならCopilot本体を使えば良いという風潮にならないかは心配なところです。

今後に期待しつつ、**現時点ではこれ目当てにPro+を契約するのはあまりお勧めできません。**

しかし、1500リクエストが月額$39で使用でき、Claude Opus 4.1など強力なモデルが利用可能なことを考えると、将来的には魅力的な選択肢になる可能性があります。
