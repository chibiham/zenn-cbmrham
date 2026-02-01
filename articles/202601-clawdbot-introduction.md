---
title: "OpenClaw(旧Clawdbot,Moltbot)入門：自己増殖するパーソナルAIアシスタントの全貌"
emoji: "🦞"
type: "tech"
topics: ["ai", "agent", "openclaw", "llm", "oss"]
published: true
---

:::message
この記事はAIを使用して執筆しています。
:::

:::message alert
**2026年2月更新**: この記事は最新の「OpenClaw」（旧Moltbot/Clawdbot）に対応しています。コマンド名、ディレクトリ構成、スキルレジストリ名が変更されています。
:::

2026年は「パーソナルエージェント元年」と呼ばれている。ChatGPTやClaudeのようなWebベースのAIチャットから、自分のデバイスで動く、自分専用のAIアシスタントへ。その代表格が **OpenClaw** だ。

わずか2ヶ月で**10万GitHubスター**を獲得し、週間200万訪問者を記録。2026年1月には史上最速で成長したオープンソースプロジェクトの一つとなった。

この記事では、OpenClawとは何か、なぜ注目されているのか、そして開発者としてどう活用できるのかを解説する。特に「自己増殖」という特徴的な機能に焦点を当てる。

## OpenClawとは

**OpenClaw**は、Peter Steinberger氏（PSPDFKit創業者）が開発したオープンソースのパーソナルAIアシスタントだ。ロブスターがマスコットで、「Your assistant. Your machine. Your rules.」を標榜する。

### 3度の脱皮：名称変更の歴史

OpenClawは3度名前を変えている。ロブスターが成長のために殻を脱ぐように。

1. **Clawdbot**（2025年11月）— 「Claude + Claw」の言葉遊び。週末プロジェクトとして始まった
2. **Moltbot**（2025年12月）— Anthropicからの商標問題を受けて改名。Discordでのブレインストーミングで決定
3. **OpenClaw**（2026年1月）— 商標調査とドメイン取得を完了し、現在の名称に

従来のAIアシスタントとの最大の違いは、**ローカルファースト** であること。Gatewayと呼ばれるサーバーを自分のPC（Mac、Linux、Windows WSL2）で起動し、WhatsApp、Telegram、Discord、Slack、iMessage、Signalなど、普段使っているメッセージングアプリからAIを呼び出せる。

会話履歴、認証情報、カスタマイズ内容はすべて自分の手元に残る。

![OpenClawのアーキテクチャ図](/images/202601-clawdbot-introduction/openclaw-architecture.png)
*OpenClawのアーキテクチャ：Gatewayがローカルで動き、複数チャネルとLLM、スキルレジストリを接続する*

### 主な特徴

| 特徴 | 説明 |
|------|------|
| **マルチチャネル** | WhatsApp、Telegram、Discord、Slack、iMessage、Signal、Teams、Matrixなど |
| **永続メモリ** | セッションをまたいで記憶が持続。「金曜の会議」と言えば覚えている |
| **プロアクティブ** | ユーザーからの入力を待たず、自分から連絡してくる（朝のブリーフィング、リマインダーなど） |
| **ローカルファースト** | データは自分のマシンに。プライバシーを自分でコントロール |
| **オープンソース** | GitHubで公開、カスタマイズ自由 |

## なぜOpenClawが注目されているのか

### 1. 「自分専用」の意味が変わった

ChatGPTやClaudeも便利だが、「自分専用」ではない。会話履歴はクラウドにあり、カスタマイズには限界がある。OpenClawは文字通り「自分のマシンで動く、自分だけのAI」だ。

### 2. 既存のワークフローに溶け込む

新しいアプリを開く必要がない。普段使っているSlackやWhatsAppでそのままAIと会話できる。これは地味だが大きい。

### 3. 自己増殖する

ここが最も重要なポイントで、後述する。

## 自己増殖：OpenClawの核心

OpenClawの最も特徴的な機能は **自己増殖（self-evolution）** だ。これは単なるカスタマイズではなく、エージェントが自分で自分を拡張していく仕組みを指す。

![自己増殖のフロー図](/images/202601-clawdbot-introduction/openclaw-self-evolution.png)
*自己増殖フロー：ユーザーのリクエストに応じてスキルを自動取得し、学習結果を蓄積していく*

### レイヤー1: ClawHub（スキル自動発見）

ClawHubはスキルのレジストリ（マーケットプレイス）だ。ユーザーが「〇〇して」と依頼した時、必要なスキルが無ければ、**OpenClaw自身がClawHubを検索し、必要なスキルをインストールする**。

人間が「このスキル入れて」と指示する必要がない。エージェントが自分で判断して能力を拡張する。

### レイヤー2: self-hackable（自己修正）

OpenClawは自分自身を修正できる。これは比喩ではなく、実際に起きた事例がある。

> "my @openclaw added Antigravity auth to @zeddotdev based on how it itself authenticates with Antigravity."

あるユーザーのOpenClawが、自分がAntigravityでどう認証しているかを参考に、**Zedエディタ用の認証機能を自分で追加した**。

この「自分で自分を拡張する」能力が、従来のAIアシスタントとの決定的な違いだ。ユーザーのコメントにもあるように：

> "The fact that it's hackable (and more importantly, self-hackable) and hostable on-prem will make sure tech like this DOMINATES."

### レイヤー3: Workspace Skills（ユーザー専用スキル）

スキルには3つのレイヤーがある：

1. **Bundled Skills** — OpenClaw本体に組み込まれたコア機能
2. **Managed Skills** — ClawHubでキュレーションされた公開スキル
3. **Workspace Skills** — ユーザーが自分で作成したスキル

Workspace Skillsは `~/.openclaw/workspace/skills/<skill>/SKILL.md` にMarkdownファイルを置くだけで追加できる。同名のスキルがある場合、Workspace Skillsが優先される。

### レイヤー4: 永続メモリとGit管理

OpenClawは日々の会話を `memory/YYYY-MM-DD.md` に記録する。これが「永続メモリ」の正体だ。

さらに重要なのは、**Agent Workspace全体をGitリポジトリにできる** こと。

```bash
cd ~/.openclaw/workspace
git init
git add AGENTS.md SOUL.md TOOLS.md IDENTITY.md USER.md memory/
git commit -m "Add agent workspace"
```

エージェントが間違った学習をしたら `git revert` で巻き戻せる。これはAIの「学習」を人間がコントロールできるという点で画期的だ。

## Agent Workspaceの構造

![Agent Workspaceのディレクトリ構造](/images/202601-clawdbot-introduction/openclaw-workspace.png)
*Agent Workspaceのディレクトリ構造：各ファイルがエージェントの人格・記憶・能力を定義する*

Workspaceは `~/.openclaw/workspace/` に配置され、以下のファイルで構成される：

| ファイル | 役割 |
|----------|------|
| `AGENTS.md` | エージェントの動作ルール、メモリ使用ガイドライン |
| `SOUL.md` | パーソナリティ、トーン、対話の境界 |
| `USER.md` | ユーザー情報、呼び方の好み |
| `TOOLS.md` | ローカルツールに関するメモ |
| `IDENTITY.md` | エージェント名、キャラクター、絵文字 |
| `memory/` | 日次メモリログ |
| `skills/` | カスタムスキル |

特に `SOUL.md` はエージェントの「人格」を定義するファイルで、ここを編集することでOpenClawの振る舞いを根本から変えられる。

## 開発者としてのOpenClaw活用

OpenClawは日常のアシスタントとしてだけでなく、**開発ワークフローの中核** としても機能する。

### マルチエージェント統合

**ntm（Named Tmux Manager）** というスキルを使うと、Claude Code、Codex、Geminiを同時にtmuxペインで起動・管理できる。

```
┌─────────────────┬─────────────────┐
│  Claude Code    │     Codex       │
│                 │                 │
├─────────────────┼─────────────────┤
│    Gemini       │   OpenClaw      │
│                 │   (orchestrator)│
└─────────────────┴─────────────────┘
```

複数のAIコーディングエージェントを並列で使い分け、OpenClawがオーケストレーターとして機能する構成だ。

**cass** というスキルは、全AIコーディングエージェントの履歴を統合検索できる。「さっきClaude Codeで何やったっけ？」を横断検索できるのは便利だ。

### 開発ワークフローの標準化

開発手法論をスキルとして組み込める：

- **planning-workflow**: 「85%の時間を計画に使う」方法論
- **agent-swarm-workflow**: 複数エージェントの調整パターン
- **ui-ux-polish**: Stripeレベルの反復的UIポリッシュ手法

これらは単なるドキュメントではなく、OpenClawが実際にその手法に従って動作するようになる。

### クラウド・ツール統合

以下のようなツールをスキルとして統合できる：

- **gcloud、Wrangler、Vercel、Supabase** — インフラ操作
- **GitHub CLI** — PRレビュー、Issue管理
- **claude-chrome** — 認証が必要なブラウザ操作の自動化

OpenClawに「Vercelにデプロイして」と言えば実行される。Slackから離れずに開発作業が完結する世界だ。

### 具体的なユースケース

**WhatsAppメモリボルト**
1000以上の音声メッセージを文字起こしし、Gitコミットと相互参照。「あのとき話したあれ」を検索できる。

**グローサリーオートパイロット**
レシピ写真を送ると、食材を抽出し、店舗にマッピングし、カートに追加するまでを自動化。開発とは直接関係ないが、OpenClawの「実世界タスク処理能力」を示す良い例だ。

**開発環境の自動セットアップ**
新しいプロジェクトを始める時、「いつもの構成でセットアップして」と言えば、過去の記憶からESLint設定、Git hooks、CI/CD設定まで自動生成する。

## セットアップ概要

### 前提条件

- Node.js 22以上
- macOS、Linux、またはWindows（WSL2推奨）

### インストール

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

または npm 経由：

```bash
npm install -g openclaw@latest
openclaw onboard --install-daemon
```

`openclaw onboard` はウィザード形式でGateway、Workspace、チャネル、スキルの設定を案内してくれる。

### 最初にやること

1. **SOUL.mdの編集** — エージェントのパーソナリティを自分好みに
2. **チャネル接続** — 普段使うメッセージングアプリを接続
3. **Gitリポジトリ化** — Workspaceをバージョン管理下に
4. **スキルの探索** — ClawHubで便利なスキルを見つける

## OpenClawと他のツールの比較

| 観点 | OpenClaw | Claude Code | ChatGPT |
|------|----------|-------------|---------|
| 実行環境 | ローカル（Gateway） | ローカル（CLI） | クラウド |
| 主なUI | 既存のメッセージングアプリ | ターミナル | Webブラウザ |
| 永続メモリ | あり（ローカル） | なし | 限定的 |
| 自己増殖 | あり | なし | なし |
| カスタマイズ | 完全 | 設定ファイル | 限定的 |
| プロアクティブ | あり | なし | なし |

Claude Codeとの違いは、Claude Codeが「ターミナルで使うコーディング特化ツール」なのに対し、OpenClawは「生活と開発を横断するパーソナルアシスタント」という点だ。用途が異なるので、併用するのが現実的だろう。

## 自己増殖AIエージェントの意味

OpenClawが示しているのは、**AIエージェントの新しいパラダイム** だ。

従来のAIツール：
- 人間が能力を定義する
- 人間が学習データを与える
- 人間がカスタマイズする

自己増殖AIエージェント：
- エージェント自身が必要な能力を判断する
- エージェント自身が学習し記憶する
- エージェント自身が自分を拡張する

これは「ツール」から「パートナー」への変化と言えるかもしれない。

ただし、これにはリスクもある。エージェントが「間違った学習」をしたり、「望ましくない拡張」をする可能性がある。だからこそ、Git管理による巻き戻しや、SOUL.mdによる人格定義といった「人間によるコントロール」の仕組みが重要になる。

:::message alert

## セキュリティ上の注意

OpenClawの強力さは、裏を返せばリスクでもある。以下の点に注意してほしい。

**権限の安易な許可**
OpenClawはファイルシステム操作、ターミナルコマンド実行、外部API呼び出しが可能だ。「Allow All」で全権限を与えるのは危険。allowlistで許可するチャネル・ツールを厳格に制限するか、機微情報のないVM・VPCなど隔離環境で実行し、万が一の暴走時の被害を限定すべきだ。

**ClawHubスキルの盲信**
ClawHubのスキルは便利だが、すべてが安全とは限らない。スキルフォルダは「信頼されたコード」として扱われ、Gateway内で実行される。インストール前にSKILL.mdの内容を確認し、何をするスキルか理解してから導入し、自動取得に任せきりにしない。

**ネットワーク露出**
デフォルトはループバックにバインドされるが、LAN公開やポート開放を安易に行うと、OpenClawが外部からの攻撃対象になる。調査により、少なくとも42,665のインスタンスがインターネット上に露出していることが判明している。

公式ドキュメントの[セキュリティガイド](https://docs.openclaw.ai/guides/security/)を参照し、定期的に `openclaw security audit --deep` を実行し、設定の見直しを行うこと。

:::

## まとめ

OpenClawは、以下の点で従来のAIアシスタントと一線を画している：

1. **ローカルファースト** — データは自分の手元に
2. **マルチチャネル** — 既存のメッセージングアプリから利用
3. **自己増殖** — スキルを自動取得し、自分で自分を拡張
4. **永続メモリ + Git管理** — 学習を人間がコントロール
5. **開発ワークフロー統合** — コーディングエージェントのオーケストレーター

2026年は「パーソナルエージェント元年」と呼ばれているが、OpenClawはその最前線にいる。自己増殖するAIエージェントが当たり前になる未来の、先行事例として注目に値する。

## 参考リンク

- [OpenClaw公式サイト](https://openclaw.ai/)
- [GitHub - openclaw/openclaw](https://github.com/openclaw/openclaw)
- [公式ドキュメント](https://docs.openclaw.ai/)
- [ClawHub - スキルレジストリ](https://clawhub.com/)
