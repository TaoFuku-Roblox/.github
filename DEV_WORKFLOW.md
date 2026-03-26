# 開発ワークフロー

プラットフォーム非依存の開発プロセス定義。

---

## 開発サイクル全体像

```
構想フェーズ (Phase)
  │ 大きなゴールを定義、Issueを洗い出しMilestoneに紐付け
  │ ステアリングファイルに構想を記録（phase-N.md）
  ▼
開発トラック判断 → CC-SDD実装サイクル (Issue単位)
  │ feature branch → PR → main (GitHub Flow)
  ▼
Milestone完了 → リリース → バージョンタグ
  ▼
次のMilestone or 次の構想フェーズへ
```

---

## 構想フェーズ (Phase)

リリース計画の最上位単位。次の期間に何をリリースするかの大きなゴールを決める。

### 定義

- **期間**: 柔軟（1〜3ヶ月）
- **成果物**: ステアリングの構想ドキュメント（`phase-N.md`）+ Milestone紐付けIssue群
- **1フェーズ内に複数Milestoneを持てる**（インクリメンタルリリース）

### Phase と Milestone の関係

```
Phase N (期間は柔軟)
  ├── Milestone（月末）  ← 短期目標をリリース
  ├── Milestone（翌月末）← 追加機能をリリース
  └── Milestone（翌々月末）← メジャー機能活性化してリリース
```

- Milestone = 時間ベースのリリースポイント（原則1ヶ月以内、月末区切り）
- Phase = 複数Milestoneを束ねる構想単位
- バージョンタグはMilestone完了時・Phase完了時にのみ付与（隔週タグ付けは行わない）

### 構想ドキュメント（phase-N.md）

各プロジェクトの `.kiro/steering/phase-N.md` に以下を記録:
- フェーズのゴール・テーマ
- 対象Issue一覧とMilestone紐付け
- 進捗状況

過去フェーズのファイルはアーカイブとしてそのまま残す（振り返りに活用）。

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

### メジャーバージョン管理: 非活性パターン

メジャー機能（新ステージ、大型リニューアル等）も通常のGitHub Flowでmainにマージする。
ただし、コード上で非活性化した状態で統合する。

**ルール**:
- メジャー機能の準備作業（フォルダ構造、データ定義等）はmainに入れてよい
- 機能が完成するまでコード上で非活性化を維持する
- 全準備が完了したら、活性化コミットでリリース対象にする
- マイナー/パッチリリースはメジャー機能が非活性の間も継続できる

---

## リリース管理

セマンティックバージョニングを採用（開発側のリリース履歴追跡用）。

### バージョン形式

**バージョン形式**: `v<MAJOR>.<MINOR>.<PATCH>`

| バージョン | インクリメント基準 | 例 |
|-----------|-------------------|-----|
| MAJOR | 大型アップデート（新ステージ、ゲーム性の大幅変更等） | v1.0.0 → v2.0.0 |
| MINOR | 機能追加・改善 | v1.0.0 → v1.1.0 |
| PATCH | バグ修正・微調整 | v1.0.0 → v1.0.1 |

### バージョンタグ付与タイミング

- **Milestone完了時**: Minor/Patchバージョンタグ + リリースノート
- **Phase完了時**: Major/Minorバージョンタグ + リリースノート
- **隔週・定期的なタグ付けは行わない**

### リリースフロー

1. Milestone内の全Issueがクローズ
2. 最終動作確認
3. プラットフォーム固有のリリース手順を実行（各プロジェクトのステアリングに記載）
4. タグ付与 + GitHub Release作成
5. Milestoneをクローズ

**タグ付与コマンド**:
```bash
gh release create v<MAJOR>.<MINOR>.<PATCH> \
  --title "v<MAJOR>.<MINOR>.<PATCH>" \
  --notes "リリースノート"
```

---

## GitHub Project Board 運用

### ステータスカラム

| カラム | 用途 | 移動タイミング |
|---|---|---|
| Backlog | 構想フェーズで洗い出したIssue | Issue作成時 |
| Ready | 仕様策定完了、実装可能 | CC-SDD validate-design通過後 |
| In Progress | 実装中 | CC-SDD spec-impl開始時 |
| Review | PR作成・レビュー待ち | PR作成時 |
| Done | マージ完了 | PRマージ時 |

### カスタムフィールド（推奨）

- **Phase**: フェーズ番号（Phase 1, Phase 2...）
- **Size**: 作業規模（S, M, L, XL）

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
            → 実装検証（仕様との整合性確認）
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
                → 実装検証（仕様との整合性確認）
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
