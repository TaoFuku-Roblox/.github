# Roblox API リファレンス

Robloxゲーム開発で再利用可能な技術的知見をまとめたドキュメント。

---

## Luau言語

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

### 空配列・ゼロ値の判定

**問題**: Luaでは`0`や空配列`{}`はfalsyではないが、条件分岐で誤解しやすい。

```lua
-- ❌ 問題あり
if sushiTypes and #sushiTypes > 0 then
    updateInventory(sushiTypes)
elseif sushiCount then  -- 0はtruthy（Luaでは0もtrue）
    updateCount(sushiCount)
end

-- ✅ 解決策: nilチェックを使用
if sushiTypes then  -- 空配列もtruthy
    updateInventory(sushiTypes)
elseif sushiCount ~= nil then  -- 明示的なnilチェック
    updateCount(sushiCount)
end
```

---

## プレイヤーテレポート

### 安全なテレポートパターン

```lua
--[[
    プレイヤーを安全にテレポートする
    CFrame設定直後にAnchored=trueで物理を一時停止しないと、意図しない移動が発生する
]]
local function safeTeleport(player: Player, targetPosition: Vector3): boolean
    local character = player.Character
    if not character then
        return false
    end

    local humanoidRootPart = character:FindFirstChild("HumanoidRootPart") :: BasePart?
    if not humanoidRootPart then
        return false
    end

    -- CFrameを設定
    humanoidRootPart.CFrame = CFrame.new(targetPosition)

    -- 物理を一時停止
    humanoidRootPart.Anchored = true

    -- 待機（物理エンジンの安定化）
    task.wait(0.05)

    -- 物理を再開
    humanoidRootPart.Anchored = false

    return true
end
```

### 注意点

- `CFrame`設定直後に`Anchored = true`で物理を一時停止しないと、意図しない移動が発生する
- テレポート後は0.05秒待機してから`Anchored = false`で物理を再開
- WorkspaceのSpawnLocationは自動再配置を引き起こすため、サーバー起動時に削除が必要

---

## アバタークローン

プレイヤーのアバターを複製して表示する（表彰台、マネキン、ゴースト表示など）。

### 基本的なクローン作成

```lua
--[[
    プレイヤーのキャラクターをクローンする
    @param player 対象プレイヤー
    @return Model? クローンしたキャラクター（失敗時はnil）
]]
local function clonePlayerAvatar(player: Player): Model?
    local character = player.Character
    if not character then
        return nil
    end

    -- Archivableを明示的にtrue設定（クローン可能にする）
    character.Archivable = true

    -- クローン作成
    local clone = character:Clone()

    -- 元に戻す（セキュリティ考慮）
    character.Archivable = true  -- デフォルトはtrueだが明示

    return clone
end
```

### クローンの固定配置

```lua
--[[
    クローンを指定位置に固定配置する
    @param clone クローンしたキャラクター
    @param position 配置位置
    @param lookAt 向き先（オプション）
]]
local function placeClone(clone: Model, position: Vector3, lookAt: Vector3?)
    local humanoidRootPart = clone:FindFirstChild("HumanoidRootPart") :: BasePart?
    if not humanoidRootPart then
        return
    end

    -- 位置と向きを設定
    local cf: CFrame
    if lookAt then
        cf = CFrame.lookAt(position, lookAt)
    else
        cf = CFrame.new(position)
    end
    clone:PivotTo(cf)

    -- 全パーツをAnchoredに設定（物理無効化）
    for _, part in clone:GetDescendants() do
        if part:IsA("BasePart") then
            part.Anchored = true
            part.CanCollide = false  -- 衝突無効化
        end
    end

    -- Humanoidの状態をリセット（アニメーション停止）
    local humanoid = clone:FindFirstChild("Humanoid") :: Humanoid?
    if humanoid then
        humanoid:ChangeState(Enum.HumanoidStateType.Physics)
    end

    clone.Parent = workspace
end
```

### 重要な注意点

1. **Archivable設定**: `character.Archivable = true`を設定しないと`Clone()`が`nil`を返す
2. **物理無効化**: 全BasePart を`Anchored = true`にしないとクローンが落下する
3. **衝突無効化**: `CanCollide = false`でプレイヤーとの干渉を防止
4. **クリーンアップ**: 不要になったクローンは必ず`:Destroy()`で削除
5. **サーバーサイド**: クローンはサーバーで作成し、自動的にクライアントにレプリケーションされる

### 使用例

- **表彰台**: マッチ勝者のアバターを表彰台に配置
- **マネキン**: ショップで装備プレビュー表示
- **ゴースト**: タイムアタックでの過去の自分を表示
- **観戦モード**: 他プレイヤーの視点表示

---

## Terrain Materials

**Roblox Terrain用マテリアル一覧**:

| カテゴリ | マテリアル | 用途例 |
|----------|------------|--------|
| **水系** | Water | 草原・氷原テーマの海 |
| **砂系** | Sand, Sandstone | 砂漠テーマの地形（海の代替） |
| **草系** | Grass, LeafyGrass | 草原テーマの島 |
| **氷系** | Ice, Glacier, Snow | 氷原テーマの島 |
| **岩系** | Rock, Basalt, Slate, Limestone | 岩場・崖 |
| **土系** | Ground, Mud | 乾燥地帯 |
| **溶岩系** | CrackedLava | 火山テーマ |

**使用上の注意**:
- `Enum.Material.Sand`はBasePartとTerrain両方に適用可能
- マテリアル変更はConfig.luaで一元管理し、コード変更なしで調整可能にする

**公式リファレンス**: https://create.roblox.com/docs/reference/engine/enums/Material

---

## Lighting API

**環境光制御プロパティ**:

| プロパティ | 用途 | 値の例 |
|------------|------|--------|
| `Lighting.Ambient` | 屋内/影の色調 | Color3.fromRGB(150, 150, 150) |
| `Lighting.OutdoorAmbient` | 屋外の環境光色 | Color3.fromRGB(170, 180, 200) |
| `Lighting.Brightness` | 光の強度（コントラスト） | 0-3（昼: 2、夜: 0.5） |
| `Lighting.ColorShift_Top` | 太陽/月からの反射色 | 夕暮れ時にオレンジ系 |
| `Lighting.ColorShift_Bottom` | 太陽と反対方向の反射色 | 夕暮れ時に紫系 |
| `Lighting.ClockTime` | 時刻設定（0-24） | 12（昼）、18（夕）、0（夜） |

**時間帯別の推奨設定**:
- **昼**: Brightness高め（2）、Ambient明るめ、ClockTime = 12
- **夕暮れ**: ColorShift_Topにオレンジ、Brightness中程度（1.5）、ClockTime = 18
- **夜**: Ambient暗め、Brightness低め（0.5）、ClockTime = 0

**公式リファレンス**: https://create.roblox.com/docs/reference/engine/classes/Lighting

---

## ParticleEmitter API

**主要プロパティ**:

| プロパティ | 用途 | 値の例 |
|------------|------|--------|
| `Rate` | 粒子放出率（particles/sec） | 20-50 |
| `Speed` | 粒子速度（NumberRange） | NumberRange.new(5, 10) |
| `Lifetime` | 粒子生存時間（NumberRange） | NumberRange.new(2, 4) |
| `Acceleration` | 重力/風向き（Vector3） | 雪: (0, -10, 0)、砂嵐: (20, -5, 0) |
| `Drag` | 速度減衰率 | 0-1 |
| `Color` | 粒子色 | ColorSequence |
| `Transparency` | 透明度 | NumberSequence |

**パフォーマンス考慮**:
- **同時表示粒子数** = Rate × Lifetime（平均）
- モバイル向け: Rate 20-30、Lifetime 2-3秒（同時40-90粒子）
- PC向け: Rate 50、Lifetime 4秒（同時200粒子）

**エフェクト例**:
- **雪**: Acceleration = (0, -10, 0)、白色、ゆっくり落下
- **砂嵐**: Acceleration = (20, -5, 0)、砂色、横向きの動き
- **紙吹雪**: Acceleration = (0, -15, 0)、複数色、5秒間

**公式リファレンス**: https://create.roblox.com/docs/reference/engine/classes/ParticleEmitter

---

## BillboardGui

**頭上UI設定**:

```lua
local billboard = Instance.new("BillboardGui")
billboard.Size = UDim2.new(0, 120, 0, 60)
billboard.StudsOffset = Vector3.new(0, 3, 0)    -- 頭上オフセット
billboard.AlwaysOnTop = false                    -- 壁越しに見えない
billboard.MaxDistance = 100                      -- 自動カリング距離
billboard.LightInfluence = 0                     -- 照明の影響なし
```

**LOD（Level of Detail）パターン**:

| 距離 | 表示内容 |
|------|----------|
| 0-30 studs | フル表示（アイコン + テキスト + 詳細） |
| 30-60 studs | 中程度（アイコン + 簡易情報） |
| 60-100 studs | 最小（アイコンのみ） |
| 100+ studs | 非表示（MaxDistanceで自動カリング） |

**公式リファレンス**: https://create.roblox.com/docs/reference/engine/classes/BillboardGui

---

## 存在しないプロパティ・API

開発中に遭遇した、存在しないまたは非推奨のプロパティ:

| プロパティ/API | 状態 | 代替案 |
|----------------|------|--------|
| `player.CharacterAutoLoads` | 存在しない | `Players.CharacterAutoLoads`（サービスプロパティ） |
| `BodyVelocity` | 非推奨 | `AssemblyLinearVelocity` |
| `BodyForce` | 非推奨 | `VectorForce` |
| `BodyPosition` | 非推奨 | `AlignPosition` |
| `BodyGyro` | 非推奨 | `AlignOrientation` |

---

## テスト制約

**Robloxにおける自動テストの制約**:
- `Instance.new("Player")`が使用不可のため、Playerインスタンスを必要とする統合テストは自動化できない
- サーバースクリプトでのテスト実行にはRoblox Studio環境と実際のプレイヤー参加が必要

**推奨テスト戦略**:
1. **手動統合テスト**: 実際のプレイヤー参加を前提とした手動テスト手順書
2. **コード検査**: Luau --strictモードで型チェック
3. **ログ出力検証**: 一貫したログ出力で実行時の動作を追跡

---

_Last updated: 2026-01-26_
