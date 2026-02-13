# 開発ワークフロー

プラットフォーム非依存の開発プロセス定義。

---

## 開発トラックの判断

作業開始前に、以下のフローで適切なトラックを判断する。

```
設計判断が必要？
  ├─ Yes → 新規機能か？
  │          ├─ Yes → グリーンフィールド
  │          └─ No  → ブラウンフィールド
  └─ No  → 既存の動作が変わる？
             ├─ Yes → ブラウンフィールド
             └─ No  → 軽量トラック

※ バグ修正は常にホットフィックストラック
```

### トラック概要

| トラック | 対象 | プロセス |
|---------|------|---------|
| グリーンフィールド | 新機能開発 | フル仕様策定 |
| ブラウンフィールド | 改善・リファクタリング | フル仕様策定 + ギャップ分析・設計互換性検証 |
| 軽量 | 機械的変更（リネーム、typo修正、ドキュメント整理等） | Issue本文に方針 → 直接実装 |
| ホットフィックス | バグ修正 | 直接修正 → PR |

---

## ブランチ戦略（GitHub Flow）

シンプルなGitHub Flowを採用。

### ブランチ構成

```
main（本番）
  ├── feature-#<issue>-<name>（機能開発・改善・リファクタリング）
  └── hotfix-#<issue>-<name>（緊急修正）
```

### ルール

- mainは常にデプロイ可能な状態を維持
- mainへの直接コミット禁止（PRのみ）
- マージ後のブランチは削除
- Issue作成後、仕様策定・実装を問わずブランチを作成してから作業開始

---

## トラック別プロセス

### グリーンフィールド（新機能）

```
Issue作成 (Feature)
  → ブランチ作成
    → 要件定義
      → 設計
        → タスク分解
          → 実装
            → PR・レビュー
              → マージ
```

各フェーズで人間のレビューが必要。

### ブラウンフィールド（改善・リファクタリング）

```
Issue作成 (Task)
  → ブランチ作成
    → 要件定義
      → ギャップ分析（既存コードと要件の差分）
        → 設計
          → 設計互換性検証（既存アーキテクチャとの整合性）
            → タスク分解
              → 実装
                → PR・レビュー
                  → マージ
```

グリーンフィールドのフローに「ギャップ分析」と「設計互換性検証」を追加。既存コードへの影響を事前に評価する。

### 軽量トラック

```
Issue作成 (Task)
  → ブランチ作成
    → Issue本文に方針を記載
      → 直接実装
        → PR・レビュー
          → マージ
```

仕様策定は不要。Issue本文が設計ドキュメントを兼ねる。

### ホットフィックス

```
Issue作成 (Bug)
  → ブランチ作成 (hotfix)
    → 修正
      → PR・レビュー
        → マージ
```

仕様策定は不要。迅速な修正を優先する。

---

## GitHub Issue / PR 規約

### Issue Type判断基準

| Type | 説明 | 判断基準 |
|------|------|----------|
| `Feature` | A request, idea, or new functionality | 新しい機能やユーザー価値の追加 |
| `Bug` | An unexpected problem or behavior | 既存機能の不具合修正 |
| `Task` | A specific piece of work | 技術的作業、リファクタリング、ドキュメントなど |

### Label判断基準

| Label | 説明 | 判断基準 |
|-------|------|---------|
| `enhancement` | New feature or request | 新機能、既存機能の拡張・改善 |
| `bug` | Something isn't working | 既存機能が期待通り動作しない |
| `documentation` | Improvements or additions to documentation | README、設計書、コメントの追加・修正 |
| `help wanted` | Extra attention is needed | 外部からの協力が必要 |
| `good first issue` | Good for newcomers | 新規コントリビューター向け |
| `question` | Further information is requested | 追加情報が必要 |

### TypeとLabelの組み合わせ例

| シナリオ | Type | Label |
|----------|------|-------|
| 新しいシステムを実装 | `Feature` | `enhancement` |
| 判定ロジックのバグを修正 | `Bug` | `bug` |
| READMEにセットアップ手順追加 | `Task` | `documentation` |
| 仕様について質問がある | - | `question` |

### Issue作成コマンド

```bash
# 新機能Issue
gh issue create \
  --assignee "@me" \
  --label "enhancement" \
  --title "タイトル" \
  --body "本文"

# バグ修正Issue
gh issue create \
  --assignee "@me" \
  --label "bug" \
  --title "タイトル" \
  --body "本文"

# ドキュメントタスク
gh issue create \
  --assignee "@me" \
  --label "documentation" \
  --title "タイトル" \
  --body "本文"
```

### PR作成コマンド

```bash
gh pr create \
  --assignee "@me" \
  --label "<適切なラベル>" \
  --title "タイトル" \
  --body "本文"
```

### 関連Issue紐付け

PR本文に以下を含めてIssueと紐付ける:
```
Closes #<issue番号>
```

### 必須設定

- `--assignee "@me"`: 作成者を自動アサイン
- `--label`: 内容に応じた適切なラベル

---

_プラットフォーム非依存の開発ワークフロー定義_
