---
title: "VSCodeにてgit worktreeをGUI操作で使いこなす"
emoji: "🌿"
type: "tech"
topics: ["vscode", "git", "worktree"]
published: true
---

## 三行要約

- `git worktree`を使えば、stashやcloneせずに複雑な管理なしに複数ブランチを別ディレクトリで同時に扱える
- VSCodeはworktreeをGUIで作成・管理でき、視覚的に操作できるので初心者でも扱いやすい
- AI駆動開発など並行作業が増えた現代では、worktreeでブランチを管理することでミスを減らせる

## はじめに

最近ではAI駆動開発によって、複数の機能を並行して開発するケースが増えてきています。
worktreeを使用しない場合、Aのブランチを維持しつつBのブランチで作業したい時は、

1. `git stash`してAの変更を一時退避してから`git switch`でBに切り替え、作業する
2. 作業が終わったらAに戻り、`git stash pop`でAの変更を復元する

という手順が主流だったのではないでしょうか。もしくは別ディレクトリに同じリポジトリをcloneしてからBのブランチを作成する……という方法もありますね。

しかしこれらの手順は**管理する変更差分やディレクトリが煩雑になり、ミスが起こりがち**です。私も一度stash関連で痛い目を見たことがあります。

そこで、今回紹介する`git worktree`の出番です。

`git worktree`は、1つのGitリポジトリから複数の作業ディレクトリを作成し、それぞれに別のブランチをチェックアウトできる機能です。
この機能を使用することで、自分で変更差分やディレクトリを管理することなく、複数のブランチを行き来できます。

しかし、`git worktree`ではコマンド操作に慣れていないと少し扱いづらい面もあります。

Visual Studio Code（以下、VSCode）は、この`git worktree`のGUI操作に公式対応しており、**視覚的にworktreeを管理**できます。

本記事では、VSCodeのGUI操作によるgit worktreeの使い方を紹介します。

::: message
CLI で`git worktree`を使う場合は最近CodeRabbitがOSSにした[`git-worktree-runner`](https://github.com/coderabbitai/git-worktree-runner)というツールもあります。

今回これを紹介していないのは、個人的に`git worktree`についてGUI操作に慣れてしまい、あまりメリットが無いように感じたからです。

CLI系AIコーディングエージェント（Claude Code, Codex）を使っている場合はこちらを検討しても良いかもしれません。
:::

## VSCodeにおけるgit worktreeの使い方

### 前提条件

デフォルトで有効になっていると思いますが、[git.detectWorktrees](vscode://settings/git.detectWorktrees)が有効になっていることを確認してください。

## Worktreeを作成する

VSCodeから新しいworktreeを作成する方法は2通りあります。

### アクティビティバーのソース管理から作成する

1. 左側のアクティビティバーから「ソース管理」アイコンを選択します。
2. ソース管理ビュー右上の`…`メニューから 「ワークツリー」 → 「ワークツリーの作成...」 を選択します。

![ワークツリーメニュー](/images/2025/11/vscode-git-worktree/worktree-menu.png)

3. ブランチ選択画面が出るので、新規作成するブランチ名を決めるか、既存のブランチを選択します。
   既存のブランチを使用する場合の一例としてリモートブランチにあるものを選択することで、**PRのレビューに使用できます**。

![ブランチ指定画面](/images/2025/11/vscode-git-worktree/worktree-branch-select.png)

![ブランチ作成画面](/images/2025/11/vscode-git-worktree/worktree-branch-create.png)

その後にワークツリーの作成先を指定する画面が表示されますが、基本的にデフォルトのままにすることをおすすめします。

![ワークツリー作成画面](/images/2025/11/vscode-git-worktree/worktree-location-select.png)

### コマンドパレットから作成する

1. `Ctrl+Shift+P`でコマンドパレットを開きます。
2. `Git: Create Worktree`と入力します。
3. ブランチ選択画面が出るので、同様にブランチ・作成先ディレクトリを指定します。

## Worktreeを開く

作成したworktreeは、次の方法で開けます。

- 直接フォルダを開く
- ソース管理ビューから開く
- コマンドパレットから開く

いずれの方法でも、指定したブランチにてチェックアウトされた状態で開きます。

### 直接フォルダを開く

エクスプローラーやターミナルから、作成する時に指定したworktreeのフォルダをVSCodeで開くだけです。

### ソース管理ビューから開く

ソース管理ビューで、開きたいworktreeを選択し「ワークツリー」を選択します。
新しいウィンドウで開く、現在のウィンドウで開くのどちらかを選びます。

### コマンドパレットから開く

1. `Ctrl+Shift+P`でコマンドパレットを開きます。
2. `Git: Open Worktree in Current Window` または `Git: Open Worktree in New Window` を選択します。
3. 一覧から対象のworktreeを選びます。

![worktree一覧画面](/images/2025/11/vscode-git-worktree/worktree-list.png)

## Worktreeを管理する（一覧・削除など）

VSCodeはそのリポジトリで認識している全てのworktreeをソース管理ビューに表示します。

Worktreeを削除したい場合は、ソース管理ビューで対象のworktreeの`…`メニューから「ワークツリー」→「ワークツリーの削除」を選択します。

または、コマンドパレットから `Git: Delete Worktree` を実行し、削除したいworktreeを選択します。

ここでworktree内に未コミットの変更がある場合は削除前にメッセージが出ます。親切ですね！

![削除前メッセージ](/images/2025/11/vscode-git-worktree/worktree-delete-confirm.png)

:::message
VSCode上では他に「Compare with Workspace」と「Migrate Worktree Changes」の機能がありますが、使ったことがないので省略させて頂きます。
小さい変更を別のworktreeへ移す場合に使うのだと思いますが、これならworktreeをそのまま使い続けて小さいPRに分割したほうが良さそうです。
:::

## まとめ

`git worktree`はターミナルから使用すると管理が煩雑になりやすいですが、VSCodeのGUI操作によって直感的に扱えるようになります。

特に複数ブランチを同時に扱うことになるAI駆動開発においては非常に相性が良く、積極的に使用していくことでミスを減らすことができます。

gitコマンドには慣れてきたけどミスが怖い、という方は一度試してみてはいかがでしょうか。
