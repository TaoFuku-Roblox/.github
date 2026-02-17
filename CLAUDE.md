# TaoFuku-Roblox グローバルスタンダード

オーガニゼーション全体で共通するAI開発ガイドライン。

> **関連ドキュメント**:
> - [DEV_WORKFLOW.md](./DEV_WORKFLOW.md) - 開発ワークフロー
> - [ROBLOX_CODING_STANDARDS.md](./ROBLOX_CODING_STANDARDS.md) - コーディング規約
> - [ROBLOX_TECHNICAL_PATTERNS.md](./ROBLOX_TECHNICAL_PATTERNS.md) - 技術パターン・ベストプラクティス
> - [ROBLOX_API_REFERENCE.md](./ROBLOX_API_REFERENCE.md) - Roblox APIリファレンス

---

## セッション開始時の必須手順

**重要**: 各セッション開始時に以下の全ファイルをフェッチすること:

1. https://raw.githubusercontent.com/TaoFuku-Roblox/.github/main/CLAUDE.md
2. https://raw.githubusercontent.com/TaoFuku-Roblox/.github/main/DEV_WORKFLOW.md
3. https://raw.githubusercontent.com/TaoFuku-Roblox/.github/main/ROBLOX_CODING_STANDARDS.md
4. https://raw.githubusercontent.com/TaoFuku-Roblox/.github/main/ROBLOX_TECHNICAL_PATTERNS.md
5. https://raw.githubusercontent.com/TaoFuku-Roblox/.github/main/ROBLOX_API_REFERENCE.md

ローカルステアリングのみを読み込んでグローバルファイルをフェッチしないと、開発ワークフロー・コーディング規約・技術パターンなど重要なルールを見逃す可能性がある。

---

## AI-DLC と Spec-Driven Development

AI-DLC（AI Development Life Cycle）上でのKiroスタイル仕様駆動開発の実装。

### cc-sdd セットアップ

新規プロジェクトでは、以下のコマンドでcc-sddをセットアップ:

```bash
npx cc-sdd@latest --lang ja
```

これにより以下が自動生成される:
- `.claude/commands/kiro/` - 11個のスラッシュコマンド
- `.kiro/settings/` - テンプレートとルール

### パス
- ステアリング: `.kiro/steering/`
- 仕様: `.kiro/specs/`

### ステアリング vs 仕様

**ステアリング** (`.kiro/steering/`) - プロジェクト全体のルールとコンテキストでAIを誘導
**仕様** (`.kiro/specs/`) - 個別機能の開発プロセスを形式化

### cc-sdd コマンドマッピング

開発トラック（[DEV_WORKFLOW.md](./DEV_WORKFLOW.md) 参照）に対応するcc-sddコマンド:

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
- 進捗確認: `/kiro:spec-status {feature}` （随時使用可能）

各フェーズで人間のレビューが必要。意図的なファストトラックの場合のみ `-y` を使用。

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

---

_このファイルはオーガニゼーション全体のスタンダードを提供します。プロジェクト固有の詳細は各リポジトリの `.kiro/steering/` ディレクトリに配置してください。_
