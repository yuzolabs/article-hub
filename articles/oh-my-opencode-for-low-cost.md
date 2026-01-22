---

title: "oh-my-opencodeで構築する低コストAIエージェント運用術"
emoji: "💰"
type: "tech"
topics: ["opencode", "githubcopilot", "AI駆動開発", "AIエージェント", "GLM"]
published: true
published_at: 2026-01-21 07:00
---

## はじめに

OpenCode という CLI 系 AI コーディングエージェントが X を中心に流行ってきています。

OpenCode は、任意の LLM API を AI エージェントのモデルとして使用できるため、非常に自由度が高いという特徴があります。
最近では Codex や GitHub Copilot のサブスクリプションとの公式連携が行われたこともあり、より手軽に利用できるようになりました。

OpenCode を利用するうえで、エージェント性能を強化する oh-my-opencode というプラグインがあります。
これは、システムプロンプトに様々な機能を追加することで、OpenCode に高度なオーケストレーションシステムを追加できるものです。

oh-my-opencode は非常に強力なプラグインですが、token の使用量が従来とは比較にならないため LLM API を使用するとコストが高くなりがちです。

そこで今回は、oh-my-opencode と GitHub Copilot、そして格安で実用的な精度を持つモデルが使用できる GLM Coding plan を組み合わせて、低コストで効率的な AI エージェント運用術を解説します。

:::message alert

GLM Coding plan は Zhipu AI 社のシンガポール法人が提供するサブスクリプションプランです。
シンガポールでホストされており、[Data Policy](https://docs.z.ai/devpack/overview#data-privacy)には入力データを保存しないと明記されています。
しかし、中国関連企業が展開しているためチャイナリスクに注意して使用するかを決めるようにしてください。

特にセキュリティ要件のあるプロジェクトでの使用は避けることをお勧めします。

本記事では GitHub Copilot のみを使用した場合の運用方法も解説しますので、そちらも参考にしてください。

:::

## 前提条件

以下のどちらか、もしくは両方のサブスクリプションを契約していることを前提とします。

- [GitHub Copilot](https://github.com/features/copilot) (Pro 以上の有料プラン)
- [GLM Coding plan](https://z.ai/subscribe?ic=NFK0Y2X5XB)

:::message

GitHub Copilot の無料プランでは、無料モデルに対してもプレミアムリクエストが消費されるため、本記事で紹介する運用方法は適していません。

:::

## OpenCode/oh-my-opencodeの導入

### OpenCode

OpenCode を[公式ドキュメント](https://opencode.ai/)に従ってインストールします。
**oh-my-opencodeでbunを使用するので、bunでのインストールを推奨します。**

```bash
curl -fsSL https://opencode.ai/install | bash

# npm管理の場合
npm i -g opencode-ai

# bun管理の場合
bun add -g opencode-ai
```

OpenCode の設定ファイル`opencode.jsonc`を作成します。
パスは以下の2つがあるので、好きな方を選びます。

個人的にはリポジトリ毎に設定を分けたい派です。

- グローバル設定: `~/.config/opencode/opencode.json`
- リポジトリ設定: `<リポジトリルート>/opencode.jsonc`

```json:opencode.jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "instructions": [
    "./AGENTS.md"
  ],
  "permission": {
    "webfetch": "allow",
    "skill": {
      "*": "allow"
    }
  },
  "plugin": [
    "oh-my-opencode"
  ]
}
```

### oh-my-opencode

次に、oh-my-opencode プラグインをインストールします。

```bash
bunx oh-my-opencode install --no-tui --claude=<yes|no|max20> --chatgpt=<yes|no> --gemini=<yes|no> --copilot=<yes|no>
```

引数には各 AI サブスクリプションの使用有無を指定します。
今回は GitHub Copilot を使用するので、`--copilot=yes`のみを指定し、それ以外は no を指定します。

その後、`opencode auth login`を実行し、GitHub Copilot の認証を行います。

```txt
┌  Add credential
│
◆  Select provider

│  Search: _
│  ● OpenCode Zen (recommended)
│  ○ Anthropic
│  ○ GitHub Copilot
│  ○ OpenAI
│  ○ Google
│  ○ OpenRouter
│  ○ Vercel AI Gateway
│  ...
│  ↑/↓ to select • Enter: confirm • Type: to search
└
```

選択肢から GitHub Copilot を選び、指示に従って認証を完了させます。

#### GLM Coding planについて

::: message

使用しない場合は読み飛ばしてください。
GitHub Copilot のみを使用する運用方法も後ほど説明します。

:::

GLM Coding plan は Zhipu AI 社が提供するサブスクリプションプランです。

初月は最低3ドルからで、Claude Pro プランの3倍程度の GLM 系モデルが使用可能です。
もし足りなければ Claude Max 5x プランの3倍程度使える Pro プラン、20x プランの3倍程度使える Max プランもあります。

これらは他のサブスクリプションと比べても非常に安価です。そのうえ、Sonnet 4.5 程度の性能を持っている点が強みです。

常時 Opus 4.5 を使いたいような人でない限りはおすすめしたいサブスクリプションとなっています。

以下の紹介 URL より登録すると、**更に10％オフ** で使い始めることができます。
私にとっても紹介料が入るため、もしこの記事が気に入って使い始めたい場合はこちらから登録して頂けると嬉しいです。

https://z.ai/subscribe?ic=NFK0Y2X5XB

GLM Coding plan を使用する場合、追加で`opencode auth login`を実行し、`Z.AI Coding Plan`を選択します。
その後、API キーを入力すれば認証が完了します。

## oh-my-opencodeのエージェント設定

上記の設定で oh-my-opencode が使用可能になるのですが、サブエージェントの細かい設定を調整するために追加で設定します。
各サブエージェントについては単純に使用するだけであれば理解する必要はありませんが、もし興味があれば[公式ドキュメント](https://github.com/code-yeongyu/oh-my-opencode/blob/dev/docs/features.md)を参照してください。

ここからの設定は、

- GitHub Copilot のみを使用する場合
- GitHub Copilot と GLM Coding plan を併用する場合
- GLM Coding plan のみを使用する場合

に分かれますので、各項を確認してください。

以下の項目の JSON ファイルは`.opencode/oh-my-opencode.jsonc`に保存します。

### GitHub Copilotのみを使用する場合

GitHub Copilot のみを使用する場合、Opus 4.5をメインセッションに設定し、サブエージェントに1x モデルを設定します。
情報を探索など、大量に使用するカスタムエージェントには0.33x モデルを使用します。場合によっては Grok Code Fast 1などの無料モデルに置き換えるのもよいです。

メインセッションはユーザーが追加でプロンプトを入力するまで追加のプレミアムリクエストを消費しません。そのため、Opus 4.5を使用しても消費を抑えることができます。

:::details 設定内容

```json:oh-my-opencode.jsonc
{
  "$schema": "https://raw.githubusercontent.com/code-yeongyu/oh-my-opencode/master/assets/oh-my-opencode.schema.json",
  "agents": {
    "Sisyphus": {
      "model": "github-copilot/claude-opus-4.5"
    },
    "oracle": {
      "model": "github-copilot/gpt-5.2"
    },
    "explore": {
      "model": "github-copilot/grok-code-fast-1"
    },
    "librarian": {
      "model": "github-copilot/gemini-3-flash-preview"
    },
    "frontend-ui-ux-engineer": {
      "model": "github-copilot/gemini-3-pro-preview"
    },
    "document-writer": {
      "model": "github-copilot/gemini-3-flash-preview"
    },
    "Prometheus (Planner)": {
      "model": "github-copilot/claude-opus-4.5"
    },
    "multimodal-looker": {
      "model": "github-copilot/gemini-3-flash-preview"
    },
    "orchestrator-sisyphus": {
      "model": "github-copilot/claude-opus-4.5"
    },
    "Metis (Plan Consultant)": {
      "model": "github-copilot/gpt-5.2"
    },
    "Momus (Plan Reviewer)": {
      "model": "github-copilot/gpt-5.2"
    },
    "Sisyphus-Junior": {
      "model": "github-copilot/claude-sonnet-4.5"
    }
  },
  "categories": {
    "visual-engineering": {
      "model": "github-copilot/gemini-3-pro-preview"
    },
    "ultrabrain": {
      "model": "github-copilot/gpt-5.2"
    },
    "artistry": {
      "model": "github-copilot/gemini-3-pro-preview"
    },
    "quick": {
      "model": "github-copilot/claude-haiku-4.5"
    },
    "most-capable": {
      "model": "github-copilot/claude-opus-4.5"
    },
    "writing": {
      "model": "github-copilot/gemini-3-flash-preview"
    },
    "general": {
      "model": "github-copilot/gpt-5.2"
    },
    "visual": {
      "model": "github-copilot/gemini-3-pro-preview"
    },
    "business-logic": {
      "model": "github-copilot/gpt-5.2"
    }
  },
  "background_task": {
    "defaultConcurrency": 5
  }
}

```

:::

### GitHub CopilotとGLM Coding planを併用する場合

GitHub Copilot と GLM Coding plan を併用する場合、メインセッションには GitHub Copilot の Opus 4.5を使用し、サブエージェントには GLM Coding plan の GLM-4.7や1x モデルを使用します。

GitHub Copilot の Pro プランを想定しているので、メインセッションに Opus 4.5を使うとプレミアムリクエストの消費が早いかも知れません。
その場合は、Opus 4.5を GPT-5.2に置き換えても良いです。

`modelConcurrency`については、GLM Coding plan のプランによって並列実行数が変化するので、適宜調整してください。

:::details 設定内容

```json:oh-my-opencode.jsonc
{
  "$schema": "https://raw.githubusercontent.com/code-yeongyu/oh-my-opencode/master/assets/oh-my-opencode.schema.json",
  "agents": {
    "Sisyphus": {
      "model": "github-copilot/claude-opus-4.5"
    },
    "oracle": {
      "model": "zai-coding-plan/glm-4.7"
    },
    "explore": {
      "model": "github-copilot/grok-code-fast-1"
    },
    "librarian": {
      "model": "github-copilot/grok-code-fast-1"
    },
    "frontend-ui-ux-engineer": {
      "model": "github-copilot/gemini-3-pro-preview"
    },
    "document-writer": {
      "model": "github-copilot/gemini-3-flash-preview"
    },
    "Prometheus (Planner)": {
      "model": "github-copilot/claude-opus-4.5"
    },
    "multimodal-looker": {
      "model": "github-copilot/gemini-3-flash-preview"
    },
    "orchestrator-sisyphus": {
      "model": "github-copilot/claude-opus-4.5"
    },
    "Metis (Plan Consultant)": {
      "model": "zai-coding-plan/glm-4.7"
    },
    "Momus (Plan Reviewer)": {
      "model": "zai-coding-plan/glm-4.7"
    },
    "Sisyphus-Junior": {
      "model": "zai-coding-plan/glm-4.7"
    }
  },
  "categories": {
    "visual-engineering": {
      "model": "github-copilot/gemini-3-pro-preview"
    },
    "ultrabrain": {
      "model": "zai-coding-plan/glm-4.7"
    },
    "artistry": {
      "model": "github-copilot/gemini-3-pro-preview"
    },
    "quick": {
      "model": "github-copilot/claude-haiku-4.5"
    },
    "most-capable": {
      "model": "zai-coding-plan/glm-4.7"
    },
    "writing": {
      "model": "github-copilot/gemini-3-flash-preview"
    },
    "general": {
      "model": "zai-coding-plan/glm-4.7"
    },
    "visual": {
      "model": "github-copilot/gemini-3-pro-preview"
    },
    "business-logic": {
      "model": "zai-coding-plan/glm-4.7"
    }
  },
  "background_task": {
    "defaultConcurrency": 5,
    "modelConcurrency": {
      "zai-coding-plan/glm-4.7": 3
    }
  }
}

```

:::

### GLM Coding planのみを使用する場合

GLM Coding plan のみを使用する場合、メインセッションに GLM-4.7を使用し、サブエージェントにも GLM-4.7を使用します。
軽いタスクについては GLM-4.5-Flash を使用します。

もし Lite プランを使用している場合は、全て GLM-4.7を使用するとカスタムエージェントの同時実行数制限に引っかかる恐れがあります。
その場合は、GLM-4.6などの別モデルを併用することを考える必要があります。

:::details 設定内容

```json:oh-my-opencode.jsonc
{
  "$schema": "https://raw.githubusercontent.com/code-yeongyu/oh-my-opencode/master/assets/oh-my-opencode.schema.json",
  "agents": {
    "Sisyphus": {
      "model": "zai-coding-plan/glm-4.7"
    },
    "oracle": {
      "model": "zai-coding-plan/glm-4.7"
    },
    "explore": {
      "model": "zai-coding-plan/glm-4.5-flash"
    },
    "librarian": {
      "model": "zai-coding-plan/glm-4.5-flash"
    },
    "frontend-ui-ux-engineer": {
      "model": "zai-coding-plan/glm-4.7"
    },
    "document-writer": {
      "model": "zai-coding-plan/glm-4.7"
    },
    "multimodal-looker": {
      "model": "zai-coding-plan/glm-4.6V"
    },
    "Prometheus (Planner)": {
      "model": "zai-coding-plan/glm-4.7"
    },
    "orchestrator-sisyphus": {
      "model": "zai-coding-plan/glm-4.7"
    },
    "Metis (Plan Consultant)": {
      "model": "zai-coding-plan/glm-4.7"
    },
    "Momus (Plan Reviewer)": {
      "model": "zai-coding-plan/glm-4.7"
    },
    "Sisyphus-Junior": {
      "model": "zai-coding-plan/glm-4.7"
    }
  },
  "categories": {
    "visual-engineering": {
      "model": "zai-coding-plan/glm-4.7"
    },
    "ultrabrain": {
      "model": "zai-coding-plan/glm-4.7"
    },
    "artistry": {
      "model": "zai-coding-plan/glm-4.7"
    },
    "quick": {
      "model": "zai-coding-plan/glm-4.5-flash"
    },
    "most-capable": {
      "model": "zai-coding-plan/glm-4.7"
    },
    "writing": {
      "model": "zai-coding-plan/glm-4.7"
    },
    "general": {
      "model": "zai-coding-plan/glm-4.7"
    },
    "visual": {
      "model": "zai-coding-plan/glm-4.7"
    },
    "business-logic": {
      "model": "zai-coding-plan/glm-4.7"
    }
  },
  "background_task": {
    "defaultConcurrency": 5,
    "modelConcurrency": {
      "zai-coding-plan/glm-4.7": 3
    }
  }
}
```

:::

## oh-my-opencodeの使い方

ここまで設定すれば、oh-my-opencode が使用可能になります。

`opencode`コマンドを実行すると、通常の OpenCode の画面が表示されます。
左下に注目すると、`Sisyphus`と通常の OpenCode では存在しないエージェントが表示されています。

![oh-my-opencodeのメニュー](/images/2026/01/oh-my-opencode-menu.png)

これが oh-my-opencode における基本的なエージェントです。

基本的な使い方として、

- プランニングは`Prometheus (Planner)`エージェントに依頼する
- それ以外は`Sisyphus`エージェントに依頼する

となります。

`Orchestrator-Sisyphus` エージェントも手動で指定できるのですが、基本的に`Sisyphus`エージェントを使用していると、オーケストレーションシステムが必要になったタイミングで自動的に切り替わるため、あまり意識する必要はありません。

`oh-my-opencode`の設定で使用するモデルを事前に決めておいたので、何のタスクでどのモデルを使用するかはエージェントが勝手に判断してくれます。
OpenCode でも設定することで似たようなことは可能ですが、最初の設定を簡単に済ませられることが`oh-my-opencode`の強みです。

## おわりに

本記事では、oh-my-opencode に GitHub Copilot、GLM Coding Plan のサブスクリプションを使用して、低コストで開発効率を高める AI エージェント運用術を解説しました。

私自身、oh-my-opencode を使った OpenCode での開発はまだ1週間程度しか経験していませんが、明らかにプランニングの質が上がっていると感じています。
プランニングの質は開発効率に直結するため、oh-my-opencode は **2026年におけるAI駆動開発のスタンダードになる** と考えているほどです。

まだ oh-my-opencode は発展途上ですが、この記事が oh-my-opencode を使った AI 駆動開発の参考になれば幸いです。
