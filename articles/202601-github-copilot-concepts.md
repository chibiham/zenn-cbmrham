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

## エージェントの実行環境

Agent Modeには2つの実行環境があり、それぞれ異なる特性を持つ。

### Agent Mode（ローカル実行）

VS Code内で動作するエージェント。自律的にファイルを探索し、編集し、ターミナルコマンドを実行する。すべてローカルで動作するため、未コミットの変更も見える。

### GitHub Copilot CLI（ローカル実行）

ターミナルから直接利用できるエージェント。コマンドライン上でCopilotの機能を利用でき、カスタムエージェント（`/agent`）やMCP連携も可能。Agent Skillsもサポート。

### Coding Agent（クラウド実行）

GitHub上で動作するエージェント。Issueやコメントから起動し、バックグラウンドでPRを作成する。GitHub Actionsの環境で動くため、コミット済みのコードのみアクセス可能。

![エージェントの実行環境](/images/202601-github-copilot-concepts/copilot-execution-environments.png)

---

## コンテキスト分離の4つの仕組み

Agent Modeを使いこなす上で重要なのが「コンテキストの分離と管理」である。大規模なタスクでは、すべての情報を1つのコンテキストに詰め込むと混乱する。Copilotは4つの仕組みでコンテキストを分離・管理する。

### 概要：4つの仕組みの役割

| 仕組み              | 役割         |                            |
| ---------------- | ---------- | -------------------------- |
| **サブエージェント**     | タスク分割・並列実行 | コンテキストと実行環境をメインのエージェントから分離 |
| **カスタムエージェント**   | 専門家の人格定義   | 特定の役割に特化したAI専門家を定義         |
| **Agent Skills** | 専門知識の提供    | 手順の定義                      |
| **MCP**          | 外部ツールとの接続  | データベースやAPIなど外部連携定義         |

### 全体像

![Copilot エージェントアーキテクチャ](/images/202601-github-copilot-concepts/copilot-agent-architecture.png)

**ポイント**：
- **実行環境**：メインエージェントと複数のサブエージェント（1:n関係）
  - 各エージェントは独立したコンテキストを持つ
  - メインからサブを起動可能
- **定義ファイル**：カスタムエージェント（`.agent.md`）、Agent Skills（`SKILL.md`）、MCP設定（`mcp-config.json`）
  - メイン・サブすべてのエージェントが、すべての定義を読み込み可能

これらは排他的ではなく、**組み合わせて使う**ことで最大の効果を発揮する。

---

### サブエージェント：タスク分割と並列実行

**概念**：メインのAgent Modeから独立したコンテキストを持つ一時的なエージェントを起動する仕組み。

**起動方法**：
```
#runSubagent を使って認証周りのコードを調査して、セキュリティ上の問題があれば報告して
```

**特徴**：
- メインのコンテキストを汚染しない
- 複数を並列実行可能
- 結果のサマリーのみが返る

**使いどころ**：
- 大きなタスクを複数の調査タスクに分割
- 異なる観点での並列分析（例：セキュリティとパフォーマンスを同時に）
- 関係ない情報でメインコンテキストを汚染したくない場合

---

### カスタムエージェント：専門家の人格定義

**概念**：特定の役割を持った専門家エージェントを定義し、明示的に呼び出せる仕組み。

**設定場所**：`.github/agents/[name].agent.md`

**ファイル例**：
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

**特徴**：
- エンドツーエンドのワークフローを一貫して処理
- 利用可能なツールを制限できる
- GitHub Copilot専用（オープンスタンダードではない）

**使いどころ**：
- セキュリティレビュー、パフォーマンス最適化など、特定の専門性を持つ作業
- 一貫した視点で複数ステップを処理してほしいとき

---

### Agent Skills：専門知識の再利用

**概念**：特定のタスクを再現可能な方法で実行するための「フォルダ形式のナレッジパッケージ」。プロンプトから自動判定でロードされる。

**設定場所**：
- プロジェクト固有：`.github/skills/[skill-name]/SKILL.md`
- 個人用：`~/.copilot/skills/[skill-name]/SKILL.md`

**ファイル例**：
```markdown
---
name: webapp-testing
description: ウェブアプリケーションのE2Eテストを実行する
---

# Web App Testing Skill

このスキルはPlaywrightを使ったE2Eテストの作成を支援します。

## テストの命名規則
- ファイル名は `*.spec.ts`
- テストケースは `test('should ...')`

## 実行コマンド
```bash
npm run test:e2e
```

**Progressive Disclosure（段階的読み込み）**：

効率的なコンテキスト管理のため、3段階で情報を読み込む：

1. **スキル発見**：YAML frontmatter（name, description）のみで関連性判断
2. **指示読み込み**：プロンプトがdescriptionに該当したら本文を読み込み
3. **リソースアクセス**：必要時のみスクリプトや例を参照

この仕組みにより、**多数のスキルをインストールしても、関連するものだけが自動的にロードされる**。

**特徴**：
- プロンプトから自動判定（明示的な呼び出し不要）
- オープンスタンダード（GitHub Copilot、Claude Code、Cursorなど共通）
- プロジェクト間で再利用可能

**使いどころ**：
- プロジェクト固有のワークフロー（デプロイ手順、テスト方法）
- 社内の開発プラクティス
- 特定技術スタックの専門知識

**スキルはコミュニティで共有されている**：
- [anthropics/skills](https://github.com/anthropics/skills)
- [github/awesome-copilot](https://github.com/github/awesome-copilot)

---

### MCP：外部ツールとの連携

**概念**：Copilotを外部のツールやデータソースと連携させるためのプロトコル。

**設定場所**：VS Code settings.json等

**特徴**：
- Agent Modeが使えるツールを拡張
- データベース、外部API、ブラウザなどにアクセス可能
- MCPサーバーが提供する機能をツールとして利用

**使いどころ**：
- データベースへのクエリ実行
- 外部APIとの連携
- ブラウザの自動操作

**例**：
```
このDBのスキーマを見て、適切なクエリを書いて
```

---

### 補足：環境別の利用可能性

2025/01/25現在、これらの仕組みは実行環境によって利用可否が異なる：

| 仕組み | VS Code Agent Mode | GitHub Copilot CLI | Copilot coding agent |
|--------|-------------------|-------------------|---------------------|
| **サブエージェント** | ✅ `#runSubagent` | ❌ 未対応（将来予定） | 不明 |
| **カスタムエージェント** | ✅ `.agent.md` | ✅ `/agent` | ✅ |
| **Agent Skills** | ✅ Insiders版<br>⏳ Stable版（近日対応） | ✅ | ✅ |
| **MCP** | ✅ | ✅ `mcp-config.json` | ✅ |

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

一度設定すれば継続的に効果を生む。投資対効果の高い順に：

1. **カスタム指示**（`.github/copilot-instructions.md`）
   - コーディング規約、命名規則を記述
   - チーム全員の体験が向上

2. **Agent Skills**（`.github/skills/`）
   - 繰り返しタスク（デプロイ、テスト実行）を定義
   - オープンスタンダードで他ツールでも再利用可

3. **カスタムエージェント**（`.github/agents/`）
   - 頻繁な専門タスク（セキュリティレビュー等）を定義

4. **MCP連携**
   - 外部ツール（DB、CI/CDなど）と統合

---

## 参考リンク

- [Copilot ask, edit, and agent modes - GitHub Blog](https://github.blog/ai-and-ml/github-copilot/copilot-ask-edit-and-agent-modes-what-they-do-and-when-to-use-them/)
- [Planning in VS Code chat - VS Code Docs](https://code.visualstudio.com/docs/copilot/chat/chat-planning)
- [Inline suggestions from GitHub Copilot - VS Code Docs](https://code.visualstudio.com/docs/copilot/ai-powered-suggestions)
- [Use custom instructions in VS Code](https://code.visualstudio.com/docs/copilot/customization/custom-instructions)
- [Adding repository custom instructions - GitHub Docs](https://docs.github.com/copilot/customizing-copilot/adding-custom-instructions-for-github-copilot)
- [Copilot Next Edit Suggestions - GitHub Next](https://githubnext.com/projects/copilot-next-edit-suggestions/)
- [Enhancing GitHub Copilot agent mode with MCP - GitHub Docs](https://docs.github.com/en/copilot/tutorials/enhance-agent-mode-with-mcp)
- [About GitHub Copilot coding agent - GitHub Docs](https://docs.github.com/en/copilot/concepts/agents/coding-agent/about-coding-agent)
- [About Agent Skills - GitHub Docs](https://docs.github.com/en/copilot/concepts/agents/about-agent-skills)
- [Use Agent Skills in VS Code](https://code.visualstudio.com/docs/copilot/customization/agent-skills)
