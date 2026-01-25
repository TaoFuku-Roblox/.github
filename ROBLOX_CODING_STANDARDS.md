# Roblox コーディング規約

Roblox開発における規約、ワークフロー、ブランチ戦略を定義。

---

## 実装ワークフロー

**新機能実装の必須手順**:

1. **実装前の確認**:
   - 実装開始前に必ずユーザーの許可を求める
   - 設計書の作成・更新の有無を確認

2. **GitHub Issue作成**:
   - 機能概要、仕様、実装範囲を明記
   - Issue番号を取得

3. **ブランチ作成**:
   - `feature-#<issue番号>-<機能名>` の命名規則でブランチを作成
   - 例: `feature-#12-life-system`

4. **設計書作成**（新機能の場合）:
   - `/kiro:spec-init "機能名"` で新規spec作成
   - または既存specへの追加を検討
   - requirements.md, design.md, tasks.md を順次作成

5. **実装とテスト**:
   - 設計書に基づいて実装
   - 手動テスト実施とドキュメント化

6. **コミットとPR**:
   - 詳細なコミットメッセージ
   - PRボディにIssue参照を含める

---

## GitHub Issue/PR 作成ルール

Issue作成時には **Type** と **Label** を判断して付与する。

### Type判断基準

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
| 新しい配達システムを実装 | `Feature` | `enhancement` |
| NPC接触判定のバグを修正 | `Bug` | `bug` |
| READMEにセットアップ手順追加 | `Task` | `documentation` |
| 仕様について質問がある | - | `question` |

### Issue作成コマンド

```bash
# 新機能Issue
gh issue create \
  --assignee "@me" \
  --type "Feature" \
  --label "enhancement" \
  --title "タイトル" \
  --body "本文"

# バグ修正Issue
gh issue create \
  --assignee "@me" \
  --type "Bug" \
  --label "bug" \
  --title "タイトル" \
  --body "本文"

# ドキュメントタスク
gh issue create \
  --assignee "@me" \
  --type "Task" \
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
- `--type`: Issue Typeを指定（Issue作成時）
- `--label`: 内容に応じた適切なラベル

---

## ブランチ戦略（GitHub Flow）

シンプルなGitHub Flowを採用。Gitflowの複雑さは小〜中規模プロジェクトでは不要。

### ブランチ構成

```
main（本番）
  ├── feature-#<issue>-<name>（機能開発）
  └── hotfix-#<issue>-<name>（緊急修正）
```

### mainブランチ

- 常にデプロイ可能な状態を維持
- 直接コミット禁止（PRのみ）
- 仕様ドキュメント（`.kiro/specs/`）はmainで管理

### featureブランチ

- 命名: `feature-#<issue番号>-<機能名>`
- 例: `feature-#12-disc-flick-system`
- 1機能 = 1ブランチ = 1 PR
- mainから分岐、mainへマージ
- マージ後は削除

### hotfixブランチ

- 命名: `hotfix-#<issue番号>-<修正内容>`
- 例: `hotfix-#15-score-calculation`
- mainから分岐、即座にPR → マージ

### ブランチ作成タイミング

| フェーズ | ブランチ | 備考 |
|---------|---------|------|
| spec-init〜spec-tasks | main | 仕様は共有ドキュメント |
| Issue作成 | main | ブランチ名にIssue番号が必要 |
| spec-impl（実装開始） | feature | コード変更開始時に分岐 |

---

## コア技術

- **言語**: Luau (--!strict モード)
- **プラットフォーム**: Roblox Engine
- **ビルドツール**: Rojo

---

## 型安全

- **Strictモード**: すべてのLuauファイルで`--!strict`を使用
- **型エクスポート**: `Types.lua`で全ての型定義を一元管理し、`export type`でエクスポート
- **型アノテーション**: 関数パラメータと戻り値に明示的な型を指定

---

## コード品質

- **エラーハンドリング**: `ErrorHandler`モジュールを使用して一貫したエラーログ出力
- **バリデーション**: `Validation`モジュールでクライアント入力を検証（エクスプロイト対策）
- **命名規則**: PascalCaseモジュール名、camelCase関数名

---

## テスト戦略

**Robloxにおける自動テストの制約**:
- `Instance.new("Player")`が使用不可のため、Playerインスタンスを必要とする統合テストは自動化できない
- サーバースクリプトでのテスト実行にはRoblox Studio環境と実際のプレイヤー参加が必要

**テスト戦略**:

1. **手動統合テスト**: 実際のプレイヤー参加を前提とした手動テスト手順書
   - 各specの`manual-testing.md`に詳細な手順、期待される動作、ログ出力、成功基準を記載

2. **コード検査**: ロジックの正確性をコードレビューとLuau --strict型チェックで検証

3. **ログ出力検証**: ErrorHandlerモジュールによる一貫したログ出力で実行時の動作を追跡・確認

---

## 開発環境

**必要ツール**:
- Roblox Studio
- Rojo 7.x+
- Git

**ビルドファイル命名規則**:

| ファイル名 | 用途 | 説明 |
|-----------|------|------|
| `{リポジトリ名}.rbxl` | **本番用** | パブリッシュ用のビルドファイル |
| `{リポジトリ名}-dev.rbxl` | **開発用** | ローカルテスト・デバッグ用 |

**ルール**:
- 本番用ファイルのみGitで管理（`.gitignore`で開発用を除外）
- パブリッシュ前は必ず本番用ファイルでビルド
- 開発用ファイルは各開発者のローカル環境で使用

**よく使うコマンド**:
```bash
# 本番用ビルド
rojo build -o {リポジトリ名}.rbxl

# 開発用ビルド
rojo build -o {リポジトリ名}-dev.rbxl

# 同期: ライブ同期でRoblox Studioと連携
rojo serve
```

---

_コーディング規約・開発ワークフローに関するドキュメント_
