# TaoFuku-Roblox グローバルスタンダード

オーガニゼーション全体で共通するAI開発ガイドライン。

> **関連ドキュメント**:
> - [DEV_WORKFLOW.md](./DEV_WORKFLOW.md) - 開発ワークフロー
> - [ROBLOX_CODING_STANDARDS.md](./ROBLOX_CODING_STANDARDS.md) - コーディング規約
> - [ROBLOX_TECHNICAL_PATTERNS.md](./ROBLOX_TECHNICAL_PATTERNS.md) - 技術パターン・ベストプラクティス
> - [ROBLOX_API_REFERENCE.md](./ROBLOX_API_REFERENCE.md) - Roblox APIリファレンス（オンデマンド）

---

## セッション開始時の必須手順

**重要**: 各セッション開始時に以下のファイルをフェッチすること。

リポジトリがプライベートのため、`gh api` 経由で取得する:

```bash
# 必須（毎セッション）
gh api repos/TaoFuku-Roblox/.github/contents/CLAUDE.md --jq '.content' | base64 -d
gh api repos/TaoFuku-Roblox/.github/contents/DEV_WORKFLOW.md --jq '.content' | base64 -d
gh api repos/TaoFuku-Roblox/.github/contents/ROBLOX_CODING_STANDARDS.md --jq '.content' | base64 -d
gh api repos/TaoFuku-Roblox/.github/contents/ROBLOX_TECHNICAL_PATTERNS.md --jq '.content' | base64 -d
```

```bash
# オンデマンド（ビジュアル・エフェクト作業時のみ）
gh api repos/TaoFuku-Roblox/.github/contents/ROBLOX_API_REFERENCE.md --jq '.content' | base64 -d
```

ローカルステアリングのみを読み込んでグローバルファイルをフェッチしないと、開発ワークフロー・コーディング規約など重要なルールを見逃す可能性がある。

---

## AI-DLC と Spec-Driven Development

AI-DLC（AI Development Life Cycle）上でのKiroスタイル仕様駆動開発の実装。

### cc-sdd セットアップ

新規プロジェクトでは、サブエージェント版（`--claude-agent`）でcc-sddをセットアップ:

```bash
npx cc-sdd@latest --claude-agent --lang ja
```

これにより以下が自動生成される:
- `.claude/commands/kiro/` - 12個のスラッシュコマンド（薄いオーケストレーター）
- `.claude/agents/kiro/` - 9個のサブエージェント定義（実際の作業ワーカー）
- `.kiro/settings/` - テンプレートとルール

### パス
- ステアリング: `.kiro/steering/`
- 仕様: `.kiro/specs/`

### ステアリング vs 仕様

**ステアリング** (`.kiro/steering/`) - プロジェクト全体のルールとコンテキストでAIを誘導
**仕様** (`.kiro/specs/`) - 個別機能の開発プロセスを形式化

### cc-sdd コマンドマッピング

開発トラック（[DEV_WORKFLOW.md](./DEV_WORKFLOW.md) 参照）に対応するcc-sddコマンド:

- 構想対話（メイン）: ユーザーと方針を練り、`product.md` を更新 → `spec-init` の説明文を具体化
- フェーズ0（任意）: `/kiro:steering`, `/kiro:steering-custom`
- フェーズ1（仕様策定）:
  - `/kiro:spec-init "説明"`
  - `/kiro:spec-requirements {feature}`
  - `/kiro:validate-gap {feature}` （ブラウンフィールド: ギャップ分析）
  - `/kiro:spec-design {feature} [-y]`
  - `/kiro:validate-design {feature}` （ブラウンフィールド: 設計互換性検証）
  - `/kiro:spec-tasks {feature} [-y]`
- フェーズ2（実装）: `/kiro:spec-impl {feature} [tasks]`
  - `/kiro:validate-impl {feature}` （必須: 実装後）
- 一括実行: `/kiro:spec-quick "説明" [--auto]` （init → requirements → design → tasks を連続実行）
- 進捗確認: `/kiro:spec-status {feature}` （随時使用可能）

各フェーズで人間のレビューが必要。意図的なファストトラックの場合のみ `-y` を使用。

---

## コンテキスト管理とサブエージェント委譲

### 基本原則

メインセッションのコンテキスト肥大化による compact（圧縮）を回避するため、以下の方針を適用する:

- **メインセッションはオーケストレーターに徹する** — 構想対話、進捗管理、ステアリング更新のみ
- **仕様策定・実装・検証は cc-sdd の `--claude-agent` 版が自動的にサブエージェントへ委譲** — 各コマンドが専用エージェントを `Task` ツールで起動する
- サブエージェントは独立したコンテキストウィンドウを持ち、メインのコンテキストを消費しない

### フェーズ別委譲ルール

| フェーズ | 実行場所 | 理由 |
|---------|---------|------|
| 構想対話 | **メイン** | ユーザーと方針を練る。コード読み込みなし |
| 要件定義 | **サブエージェント** | cc-sdd が `spec-requirements-agent` に委譲 |
| ギャップ分析 | **サブエージェント** | cc-sdd が `validate-gap-agent` に委譲 |
| 設計 | **サブエージェント** | cc-sdd が `spec-design-agent` に委譲 |
| 設計互換性検証 | **サブエージェント** | cc-sdd が `validate-design-agent` に委譲 |
| タスク分解 | **サブエージェント** | cc-sdd が `spec-tasks-agent` に委譲 |
| 実装（タスク単位） | **サブエージェント** | cc-sdd が `spec-tdd-impl-agent` に委譲 |
| 実装検証 | **サブエージェント** | cc-sdd が `validate-impl-agent` に委譲 |
| ステアリング更新 | **メイン** | 後続サブエージェントへのコンテキスト反映のため |

### 構想対話フェーズ

要件定義をサブエージェントに委譲する前に、メインセッションでユーザーと「何を作るか」を固める:

1. ユーザーと対話し、プロダクトの方向性・スコープ・ユーザー価値を明確化
2. 合意内容を `product.md` に反映（ステアリング更新）
3. 具体的な説明文で `spec-init` を実行 → 以降のサブエージェントが高精度で動作

構想が明確なほど、サブエージェントによる要件生成の精度が上がり、レビュー・修正のサイクルが減る。

### ステアリング更新タイミング

サブエージェントが最新のプロジェクトコンテキストで動作するよう、以下のタイミングでステアリングを更新する。更新はメインセッションで実行すること。

| タイミング | 更新対象 | 内容 |
|-----------|---------|------|
| 構想対話後 | `product.md` | プロダクトの方向性・スコープ・ユーザー価値 |
| 設計フェーズ完了後 | `tech.md` | 技術的意思決定（ライブラリ選定、アーキテクチャパターン等） |
| 実装完了後 | `structure.md` | ファイル構成の変更（新規ディレクトリ、主要ファイル等） |

### コンテキストリセット（推奨）

以下のタイミングで `/clear` によるコンテキストリセットを推奨する。状態はステアリング・仕様ファイルに永続化されているため、clear 後も作業を継続できる。

| タイミング | 理由 |
|-----------|------|
| 構想対話後、`spec-init` の前 | 長い対話の蓄積を解放。合意内容は `product.md` に記録済み |
| 機能完了後、次の機能に着手する前 | 完了報告・レビューの蓄積を解放。成果は `structure.md` に記録済み |

---

## 開発ガイドライン

- 思考は英語、レスポンスは日本語で生成。プロジェクトファイル（requirements.md、design.md、tasks.md、research.md、検証レポートなど）に書き込むMarkdownコンテンツは、仕様で設定された言語（spec.json.language参照）で記述すること。
- ユーザーの指示に正確に従い、その範囲内で自律的に行動: 必要なコンテキストを収集し、このラン内で作業を最後まで完了する。質問は必須情報が不足している場合か、指示が致命的に曖昧な場合のみ。
- ステアリングを最新に保ち、`/kiro:spec-status` でアラインメントを確認

---

## ステアリング設定

- `.kiro/steering/` 全体をプロジェクトメモリとしてロード
- デフォルトファイル: `product.md`, `tech.md`, `structure.md`
- カスタムファイルもサポート（`/kiro:steering-custom` で管理）
- 更新タイミング: 構想対話後に `product.md`、設計完了後に `tech.md`、実装完了後に `structure.md`（詳細は「コンテキスト管理とサブエージェント委譲」セクション参照）

### 記述原則

ステアリングは全サブエージェントが起動時にロードするため、簡潔さを意識する:

- **パターンを記述し、網羅的リストを避ける** — ファイル一覧ではなく、ディレクトリの役割と命名規則を記述する
- **カスタムステアリングは必要なものだけ作成する** — 判断に迷う領域のみ。テンプレートが用意されているからといって全て作る必要はない
- **自明になった記述は整理する** — コード全体に浸透した意思決定など、もはやガイダンスが不要な情報は削除してよい

---

_このファイルはオーガニゼーション全体のスタンダードを提供します。プロジェクト固有の詳細は各リポジトリの `.kiro/steering/` ディレクトリに配置してください。_
