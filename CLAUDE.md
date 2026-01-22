# TaoFuku-Roblox グローバルスタンダード

オーガニゼーション全体で共通するAI開発ガイドライン。

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

### アクティブな仕様
- `.kiro/specs/` でアクティブな仕様を確認
- `/kiro:spec-status [feature-name]` で進捗を確認

## 開発ガイドライン
- 思考は英語、レスポンスは日本語で生成。プロジェクトファイル（requirements.md、design.md、tasks.md、research.md、検証レポートなど）に書き込むMarkdownコンテンツは、仕様で設定された言語（spec.json.language参照）で記述すること。

## 基本ワークフロー
- フェーズ0（任意）: `/kiro:steering`, `/kiro:steering-custom`
- フェーズ1（仕様策定）:
  - `/kiro:spec-init "説明"`
  - `/kiro:spec-requirements {feature}`
  - `/kiro:validate-gap {feature}` （任意: 既存コードベース向け）
  - `/kiro:spec-design {feature} [-y]`
  - `/kiro:validate-design {feature}` （任意: 設計レビュー）
  - `/kiro:spec-tasks {feature} [-y]`
- フェーズ2（実装）: `/kiro:spec-impl {feature} [tasks]`
  - `/kiro:validate-impl {feature}` （任意: 実装後）
- 進捗確認: `/kiro:spec-status {feature}` （随時使用可能）

## 開発ルール
- 3フェーズ承認ワークフロー: 要件 → 設計 → タスク → 実装
- 各フェーズで人間のレビューが必要。意図的なファストトラックの場合のみ `-y` を使用
- ステアリングを最新に保ち、`/kiro:spec-status` でアラインメントを確認
- ユーザーの指示に正確に従い、その範囲内で自律的に行動: 必要なコンテキストを収集し、このラン内で作業を最後まで完了する。質問は必須情報が不足している場合か、指示が致命的に曖昧な場合のみ。

## ステアリング設定
- `.kiro/steering/` 全体をプロジェクトメモリとしてロード
- デフォルトファイル: `product.md`, `tech.md`, `structure.md`
- カスタムファイルもサポート（`/kiro:steering-custom` で管理）

---

## Roblox 開発スタンダード

### 実装ワークフロー

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

### GitHub Issue/PR 作成ルール

Issue作成時には **Type** と **Label** を判断して付与する。

#### Type判断基準

| Type | 説明 | 判断基準 |
|------|------|----------|
| `Feature` | A request, idea, or new functionality | 新しい機能やユーザー価値の追加 |
| `Bug` | An unexpected problem or behavior | 既存機能の不具合修正 |
| `Task` | A specific piece of work | 技術的作業、リファクタリング、ドキュメントなど |

#### Label判断基準

| Label | 説明 | 判断基準 |
|-------|------|----------|
| `enhancement` | New feature or request | 新機能、既存機能の拡張・改善 |
| `bug` | Something isn't working | 既存機能が期待通り動作しない |
| `documentation` | Improvements or additions to documentation | README、設計書、コメントの追加・修正 |
| `help wanted` | Extra attention is needed | 外部からの協力が必要 |
| `good first issue` | Good for newcomers | 新規コントリビューター向け |
| `question` | Further information is requested | 追加情報が必要 |

#### TypeとLabelの組み合わせ例

| シナリオ | Type | Label |
|----------|------|-------|
| 新しい配達システムを実装 | `Feature` | `enhancement` |
| NPC接触判定のバグを修正 | `Bug` | `bug` |
| READMEにセットアップ手順追加 | `Task` | `documentation` |
| 仕様について質問がある | - | `question` |

#### Issue作成コマンド

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

#### PR作成コマンド

```bash
gh pr create \
  --assignee "@me" \
  --label "<適切なラベル>" \
  --title "タイトル" \
  --body "本文"
```

#### 関連Issue紐付け

PR本文に以下を含めてIssueと紐付ける:
```
Closes #<issue番号>
```

#### 必須設定

- `--assignee "@me"`: 作成者を自動アサイン
- `--type`: Issue Typeを指定（Issue作成時）
- `--label`: 内容に応じた適切なラベル

### ブランチ戦略（GitHub Flow）

シンプルなGitHub Flowを採用。Gitflowの複雑さは小〜中規模プロジェクトでは不要。

#### ブランチ構成

```
main（本番）
  ├── feature-#<issue>-<name>（機能開発）
  └── hotfix-#<issue>-<name>（緊急修正）
```

#### mainブランチ

- 常にデプロイ可能な状態を維持
- 直接コミット禁止（PRのみ）
- 仕様ドキュメント（`.kiro/specs/`）はmainで管理

#### featureブランチ

- 命名: `feature-#<issue番号>-<機能名>`
- 例: `feature-#12-disc-flick-system`
- 1機能 = 1ブランチ = 1 PR
- mainから分岐、mainへマージ
- マージ後は削除

#### hotfixブランチ

- 命名: `hotfix-#<issue番号>-<修正内容>`
- 例: `hotfix-#15-score-calculation`
- mainから分岐、即座にPR → マージ

#### ブランチ作成タイミング

| フェーズ | ブランチ | 備考 |
|---------|---------|------|
| spec-init〜spec-tasks | main | 仕様は共有ドキュメント |
| Issue作成 | main | ブランチ名にIssue番号が必要 |
| spec-impl（実装開始） | feature | コード変更開始時に分岐 |

### コア技術

- **言語**: Luau (--!strict モード)
- **プラットフォーム**: Roblox Engine
- **ビルドツール**: Rojo

### 型安全
- **Strictモード**: すべてのLuauファイルで`--!strict`を使用
- **型エクスポート**: `Types.lua`で全ての型定義を一元管理し、`export type`でエクスポート
- **型アノテーション**: 関数パラメータと戻り値に明示的な型を指定

### コード品質
- **エラーハンドリング**: `ErrorHandler`モジュールを使用して一貫したエラーログ出力
- **バリデーション**: `Validation`モジュールでクライアント入力を検証（エクスプロイト対策）
- **命名規則**: PascalCaseモジュール名、camelCase関数名

### テスト戦略

**Robloxにおける自動テストの制約**:
- `Instance.new("Player")`が使用不可のため、Playerインスタンスを必要とする統合テストは自動化できない
- サーバースクリプトでのテスト実行にはRoblox Studio環境と実際のプレイヤー参加が必要

**テスト戦略**:

1. **手動統合テスト**: 実際のプレイヤー参加を前提とした手動テスト手順書
   - 各specの`manual-testing.md`に詳細な手順、期待される動作、ログ出力、成功基準を記載

2. **コード検査**: ロジックの正確性をコードレビューとLuau --strict型チェックで検証

3. **ログ出力検証**: ErrorHandlerモジュールによる一貫したログ出力で実行時の動作を追跡・確認

### 開発環境

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

---

## Luau ベストプラクティス

### 関数定義順序

**重要**: Luauでは関数を呼び出す前に定義されている必要がある。

```lua
-- ❌ エラー: CreateSpotlightが未定義
local function CreateIsland()
    CreateSpotlight(island)  -- エラー
end

local function CreateSpotlight()
    -- ...
end

-- ✅ 正しい順序
local function CreateSpotlight()
    -- ...
end

local function CreateIsland()
    CreateSpotlight(island)  -- OK
end
```

**ベストプラクティス**:
- ヘルパー関数は呼び出し元より前に定義
- 相互参照が必要な場合は`local function`の代わりに変数宣言後に代入

### デバッグ用設定パターン

**テスト用フラグの実装パターン**:

```lua
-- Config.lua
local Debug = {
    infiniteLives = false,  -- テスト時にtrueに設定
}

function ConfigModule.IsInfiniteLives(): boolean
    return Debug.infiniteLives
end
```

**注意**: 本番リリース前に必ず`false`に戻すこと。

---

## Roblox APIリファレンス

### Terrainマテリアル

| カテゴリ | マテリアル | 用途例 |
|----------|------------|--------|
| **水系** | Water | 草原・氷原テーマの海 |
| **砂系** | Sand, Sandstone | 砂漠テーマの地形 |
| **草系** | Grass, LeafyGrass | 草原テーマの島 |
| **氷系** | Ice, Glacier, Snow | 氷原テーマの島 |
| **岩系** | Rock, Basalt, Slate, Limestone | 岩場・崖 |
| **土系** | Ground, Mud | 乾燥地帯 |
| **溶岩系** | CrackedLava | 火山テーマ |

**公式リファレンス**: https://create.roblox.com/docs/reference/engine/enums/Material

### Lighting API

| プロパティ | 用途 | 値の例 |
|------------|------|--------|
| `Lighting.Ambient` | 屋内/影の色調 | Color3.fromRGB(150, 150, 150) |
| `Lighting.OutdoorAmbient` | 屋外の環境光色 | Color3.fromRGB(170, 180, 200) |
| `Lighting.Brightness` | 光の強度 | 0-3（昼: 2、夜: 0.5） |
| `Lighting.ClockTime` | 時刻設定（0-24） | 12（昼）、18（夕）、0（夜） |

**公式リファレンス**: https://robloxapi.github.io/ref/class/Lighting.html

### ParticleEmitter API

| プロパティ | 用途 | 値の例 |
|------------|------|--------|
| `Rate` | 粒子放出率 | 20-50 |
| `Speed` | 粒子速度 | NumberRange.new(5, 10) |
| `Lifetime` | 生存時間 | NumberRange.new(2, 4) |
| `Acceleration` | 重力/風向き | Vector3 |

**パフォーマンス考慮**:
- **同時表示粒子数** = Rate × Lifetime（平均）
- モバイル向け: Rate 20-30、Lifetime 2-3秒
- PC向け: Rate 50、Lifetime 4秒

**公式リファレンス**: https://robloxapi.github.io/ref/class/ParticleEmitter.html

### Roblox固有の制約

**プレイヤーテレポート**:
- `CFrame`設定直後に`Anchored = true`で物理を一時停止しないと、意図しない移動が発生する
- テレポート後は0.05秒待機してから`Anchored = false`で物理を再開

**移動能力の復元**:
- テレポート後やゲームオーバー時に`humanoid.WalkSpeed`と`humanoid.JumpPower`が0のままになっている場合がある
- 明示的に`WalkSpeed = 16`（デフォルト）、`JumpPower = 50`を設定して復元

**Zファイティング対策**:
- 同一高さの重なるパーツはちらつき（Zファイティング）が発生
- 各レイヤーに微小な高さオフセット（0.01 studs）を追加して解決

**二重検出防止パターン**:
- イベント（Touched等）が短時間に複数回発火する問題の対策:
```lua
local isProcessing = false
connection = part.Touched:Connect(function()
    if isProcessing then return end
    isProcessing = true
    -- 処理
    task.delay(0.5, function() isProcessing = false end)
end)
```

**物理シミュレーションのキャリブレーション**:
- 物理計算の理論値と実測値には乖離がある
- 物理ベースのゲームでは必ず実測データに基づいてパラメータを調整すること
- 例: 飛距離計算では理論式ではなく実測公式を使用

**存在しないプロパティ**:
- `player.CharacterAutoLoads`: このプロパティは存在しない（使用不可）

---
_このファイルはオーガニゼーション全体のスタンダードを提供します。プロジェクト固有の詳細は各リポジトリの `.kiro/steering/` ディレクトリに配置してください。_
