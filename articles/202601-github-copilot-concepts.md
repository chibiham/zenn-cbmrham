---
title: "GitHub Copilotを使いこなすための概念整理"
emoji: "🪼"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["githubcopilot", "ai", "vscode", "開発ツール"]
published: true
---

:::message
筆者の考えを元にAIを使用して執筆しています。
:::

GitHub Copilotを「なんとなく使っている」状態から脱却するには、背後にある概念を理解することが近道だと思う。本記事では、Copilotの仕組みと各機能を概念レベルで整理し、「なぜそう動くのか」を理解することで使いこなせるようになることを目指す。

---

## 大前提：LLMがコードを生成する仕組み

Copilotの挙動を理解するには、まずLLM（大規模言語モデル）の基本的な動作原理を押さえておく必要がある。

LLMは**与えられたコンテキスト（入力）に基づいて、最も確率の高い続きを生成する**。Copilotも同様で、「現在のコード」「開いているファイル」「プロジェクトの情報」などをコンテキストとしてモデルに渡し、その続きとしてコードを生成している。

![LLMによるコード生成フロー](/images/202601-github-copilot-concepts/copilot-llm-flow.png)

つまり、**Copilotを使いこなすとは「適切なコンテキストを与えること」に他ならない**。これがこの記事全体を貫くテーマである。

---

## インライン補完：最も基本的な機能

コードを書いているときに薄いグレーで表示される「Ghost Text」がインライン補完である。Tabキーで受け入れ、Escで却下する、あの機能だ。

### コンテキストの収集範囲

インライン補完では、以下の情報がコンテキストとして使われる：

1. **現在のファイル**：カーソル位置より前のコード
2. **開いているタブ**：関連ファイルを開いておくと参照される
3. **ファイル名・パス**：命名から意図を推測する

だから、関連するファイルをタブで開いておくだけで補完の質が上がる。これは知っているだけで得をするTipsだ。

### Next Edit Suggestions（NES）

2025年に登場した新機能で、インライン補完の進化形といえる。従来の補完が「現在のカーソル位置の続き」を予測するのに対し、NESは**「次にどこを編集するか」まで予測する**。

例えば、関数名を変更したとき、NESはその関数を呼び出している他の箇所を検出し、「ここも変更が必要では？」と提案してくる。編集の履歴（diff）を見て、変更の意図を理解しようとしているわけだ。

NESが効果的に働くシナリオ：

- **変数・関数のリネーム**：一箇所変えると他も提案
- **引数の追加**：呼び出し元への変更を提案
- **タイポの修正**：同じミスを他でも検出
- **パターンの繰り返し**：同様の変更を別の場所で提案

---

## 4つのモード：Ask / Edit / Agent / Plan

Copilot Chatには4つのモードがあり、これがよく混乱を招く。違いを理解するポイントは**「コンテキストの範囲」と「自律性のレベル」**である。

![4つのモードのマトリクス](/images/202601-github-copilot-concepts/copilot-4modes-matrix.png)

### Ask モード

```
コンテキスト範囲：狭い（明示的に与えたもののみ）
自律性：なし（回答のみ、コード変更しない）
```

質問に答えるだけで、コードは一切変更しない。「このコードは何をしている？」「TypeScriptでnullチェックする方法は？」といった質問に最適。

**使いどころ**：
- コードの理解・学習
- 実装方針の相談
- エラーメッセージの解説

### Edit モード

```
コンテキスト範囲：中程度（指定したファイル）
自律性：中程度（提案するが、適用は人間が判断）
```

自然言語で変更内容を伝えると、diff形式で変更案を提示してくれる。**変更前に必ず差分を確認できる**のが特徴で、意図しない変更を防げる。

**使いどころ**：
- 特定ファイルのリファクタリング
- 関数の追加・修正
- コメントの追加

### Agent モード

```
コンテキスト範囲：広い（自動で探索）
自律性：高い（複数ステップを自律実行）
```

最も強力かつ「危険」なモード。タスクを伝えると、必要なファイルを自分で探し、複数ファイルにまたがる変更を自律的に行う。ターミナルコマンドの実行も（許可を求めた上で）行う。

**使いどころ**：
- 新機能の実装
- 複数ファイルにまたがるリファクタリング
- テストの追加

### Plan モード

```
コンテキスト範囲：広い（コードベース全体を分析）
自律性：中程度（計画作成は自律的、実行は承認後）
```

2025年に追加された新モードで、**実装前に詳細な計画を作成する**ことに特化している。Agent Modeがすぐに実装を始めるのに対し、Plan Modeはまず「何をすべきか」を整理し、ユーザーの承認を得てから実装に移る。

**Plan Modeの流れ**：

![Plan Modeのワークフロー](/images/202601-github-copilot-concepts/copilot-plan-mode-workflow.png)

1. 高レベルなタスクを入力（例：「OAuth2認証を実装して」）
2. Copilotがコードベースを分析し、不明点があれば質問してくる
3. 実行可能なステップに分解した計画を提示
4. ユーザーが計画をレビュー・修正
5. 承認後、実装エージェントに引き継ぎ

**重要な特徴**：
- **承認するまでコードは変更されない**：計画段階で要件の抜け漏れを発見できる
- **Todoリストの自動生成**：複雑なタスクは自動的にステップに分解される
- **反復的な改善**：計画が不十分なら、何度でも修正できる

**使いどころ**：
- 複雑な新機能の実装前
- 影響範囲が不明確なリファクタリング
- チームでレビューしたい大きな変更
- 「何から手をつければいいかわからない」とき

Agent Modeとの使い分けとしては、**確実に進めたい大きなタスクはPlan**、**試行錯誤しながら進めたい小〜中規模タスクはAgent**という感覚が良い。

### モード選択の指針

| 状況 | 推奨モード |
|------|-----------|
| コードの意味を知りたい | Ask |
| 特定ファイルを確実に編集したい | Edit |
| 何をすればいいかも含めて任せたい | Agent |
| 大きなタスクを計画的に進めたい | Plan |
| 影響範囲が不明確なタスク | まずPlanで計画 → Agent |

私の感覚では、**迷ったらAskから始める**のが安全だ。大規模な変更の場合は**Planで計画を立ててからAgentで実装**という流れが効果的である。

---

## コンテキスト管理：明示的に渡す方法

Copilotに「もっとこの情報を見てほしい」と思うことがある。そのための明示的なコンテキスト指定方法がいくつかある。

### `#file:パス`

特定のファイルをコンテキストに含める。

```
#file:src/types.ts を参照して、User型に合うバリデーション関数を書いて
```

### `#codebase`

リポジトリ全体から関連コードを検索させる。

```
#codebase 認証処理はどこで行っている？
```

### `@workspace`

ワークスペース（VS Codeで開いているフォルダ）全体を参照。`#codebase`と似ているが、GitHubリポジトリとの連携機能も使える。

### ドラッグ＆ドロップ

チャットウィンドウにファイルをドラッグ＆ドロップすると、そのファイルがコンテキストに追加される。地味だが便利。

---

## カスタム指示：プロジェクトのルールを教える

Copilotに「うちのプロジェクトではこう書く」というルールを教えることで、提案の質が大きく向上する。設定ファイルには複数の種類があり、それぞれ適用範囲が異なる。

### `.github/copilot-instructions.md`

リポジトリ全体に適用される基本ルール。チームで共有するコーディング規約を書くのに適している。

```markdown
# コーディング規約

- TypeScriptを使用し、`any`型は禁止
- 関数にはJSDocコメントを付ける
- エラーハンドリングには必ずtry-catchを使う
- テストはVitestで書く
```

### `*.instructions.md`

`applyTo`フロントマターで、特定のファイルパターンにのみ適用される指示を書ける。

```markdown
---
applyTo: "src/components/**/*.tsx"
---

# Reactコンポーネントのルール

- 関数コンポーネントのみ使用
- Propsの型は`XxxProps`の命名規則
- カスタムフックは`useXxx`で始める
```

配置場所はデフォルトで`.github/instructions/`だが、設定で変更可能。

### `AGENTS.md`

モノレポ対応の設定ファイル。ディレクトリごとにネストして配置でき、編集中のファイルに応じて適切な指示が自動で読み込まれる。

```
/monorepo/
├── AGENTS.md                 # 共通ルール
├── packages/
│   ├── frontend/
│   │   └── AGENTS.md         # フロントエンド固有
│   └── backend/
│       └── AGENTS.md         # バックエンド固有
```

`chat.useNestedAgentsMdFiles`設定を有効にすると機能する。

### 設定ファイルの優先順位

複数の設定ファイルがある場合、以下の順で適用される（後のものが優先）：

1. 組織レベルの設定
2. リポジトリの`.github/copilot-instructions.md`
3. `*.instructions.md`（`applyTo`にマッチするもの）
4. `AGENTS.md`

---

## エージェント機能：より高度な自動化

### Agent Mode（ローカル実行）

VS Code内で動作するエージェント。自律的にファイルを探索し、編集し、ターミナルコマンドを実行する。すべてローカルで動作するため、未コミットの変更も見える。

### Coding Agent（クラウド実行）

GitHub上で動作するエージェント。Issueやコメントから起動し、バックグラウンドでPRを作成する。GitHub Actionsの環境で動くため、コミット済みのコードのみアクセス可能。

![エージェントの種類](/images/202601-github-copilot-concepts/copilot-agent-types.png)

### サブエージェント

Agent Mode内で`#runSubagent`を使うと、独立したコンテキストを持つサブエージェントを起動できる。大きなタスクを分割したり、特定の調査を並列で行わせたりするのに使う。

```
#runSubagent を使って認証周りのコードを調査して、セキュリティ上の問題があれば報告して
```

サブエージェントの特徴：
- メインのコンテキストを汚染しない
- 複数を並列実行可能
- 結果のサマリーのみが返る

### カスタムエージェント

`.github/agents/`に`.agent.md`ファイルを配置すると、専門化したエージェントを定義できる。

```markdown
---
name: security-reviewer
description: セキュリティ観点でコードをレビュー
tools:
  - read
  - search
---

# Security Reviewer

OWASP Top 10の観点でコードをレビューする。
脆弱性を発見したら、重大度と修正方法を報告する。
```

---

## MCP：外部ツールとの連携

MCP（Model Context Protocol）は、Copilotを外部のツールやデータソースと連携させるためのプロトコルである。Agent Modeと組み合わせることで、Copilotの能力を大幅に拡張できる。

例えば：
- データベースへのクエリ実行
- 外部APIとの連携
- ブラウザの自動操作

MCPサーバーを設定すると、Agent Modeがそのツールを使えるようになる。「このDBのスキーマを見て、適切なクエリを書いて」といった指示が可能になる。

---

## 使いこなすための心構え

### 1. コンテキストを意識する

この記事で繰り返し述べてきたように、Copilotの出力品質は入力（コンテキスト）で決まる。

- 関連ファイルをタブで開いておく
- `#file`や`#codebase`で明示的にコンテキストを渡す
- `instructions.md`でプロジェクトのルールを教える

### 2. モードを使い分ける

- **迷ったらAsk**から始める
- 変更箇所が明確なら**Edit**
- 複雑なタスクは**Agent**
- 大規模な変更は**Plan**で計画してから実装

### 3. 出力を鵜呑みにしない

Copilotは「確率的に最もありそうなコード」を生成しているに過ぎない。常にレビューし、理解した上で受け入れる姿勢が重要だ。特に：

- セキュリティに関わる部分
- ビジネスロジック
- エッジケースの処理

### 4. カスタマイズに投資する

`.github/copilot-instructions.md`を整備するだけで、チーム全員のCopilot体験が向上する。初期投資の価値は十分にある。

---

## 参考リンク

- [Copilot ask, edit, and agent modes - GitHub Blog](https://github.blog/ai-and-ml/github-copilot/copilot-ask-edit-and-agent-modes-what-they-do-and-when-to-use-them/)
- [Planning in VS Code chat - VS Code Docs](https://code.visualstudio.com/docs/copilot/chat/chat-planning)
- [Inline suggestions from GitHub Copilot - VS Code Docs](https://code.visualstudio.com/docs/copilot/ai-powered-suggestions)
- [Use custom instructions in VS Code](https://code.visualstudio.com/docs/copilot/customization/custom-instructions)
- [Adding repository custom instructions - GitHub Docs](https://docs.github.com/copilot/customizing-copilot/adding-custom-instructions-for-github-copilot)
- [Copilot Next Edit Suggestions - GitHub Next](https://githubnext.com/projects/copilot-next-edit-suggestions/)
- [Enhancing GitHub Copilot agent mode with MCP - GitHub Docs](https://docs.github.com/en/copilot/tutorials/enhance-agent-mode-with-mcp)
