# Roblox 技術パターン

Robloxゲーム開発で再利用可能な技術パターンとベストプラクティス。

---

## Luau言語パターン

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

## Roblox固有の制約

### 移動能力の復元

- テレポート後やゲームオーバー時に`humanoid.WalkSpeed`と`humanoid.JumpPower`が0のままになっている場合がある
- 明示的に`WalkSpeed = 16`（デフォルト）、`JumpPower = 50`を設定して復元

### Zファイティング対策

- 同一高さの重なるパーツはちらつき（Zファイティング）が発生
- 各レイヤーに微小な高さオフセット（0.01 studs）を追加して解決

### 二重検出防止パターン

イベント（Touched等）が短時間に複数回発火する問題の対策:

```lua
local isProcessing = false
connection = part.Touched:Connect(function()
    if isProcessing then return end
    isProcessing = true
    -- 処理
    task.delay(0.5, function() isProcessing = false end)
end)
```

### 物理シミュレーションのキャリブレーション

- 物理計算の理論値と実測値には乖離がある
- 物理ベースのゲームでは必ず実測データに基づいてパラメータを調整すること
- 例: 飛距離計算では理論式ではなく実測公式を使用

### Terrain APIボクセル解像度

- Terrain APIは4 studボクセル解像度で動作する
- `workspace.Terrain:FillBlock()`等で指定したY座標と、実際の表面Y座標にはずれが生じる
- 例: Y=40で地形生成 → 実際の表面はY=42付近になる
- Raycast等で地表面を検出する際はこのオフセットを考慮すること

### RaycastParams FilterDescendantsInstances管理

遅延生成されるフォルダ（ゲーム中に動的作成されるもの）をフィルターに含める場合、フォルダが存在してからrefreshFiltersを呼ぶ必要がある。

```lua
-- 1. フォルダを先行作成
local folder = Instance.new("Folder")
folder.Name = "DynamicObjects"
folder.Parent = workspace

-- 2. その後でフィルターを更新
FloorDetectionSystem.refreshFilters()
```

**注意**: フィルターリストは`table.clone()`でコピーしてから変更し、新しい配列を代入すること。

### プレイヤーキャラクターの動的フィルタリング

プレイヤーキャラクターは途中参加やリスポーンで動的に生成される。Raycast時にリアルタイムでフィルターリストを更新するパターン:

```lua
local currentFilter = raycastParams.FilterDescendantsInstances
local updatedFilter = table.clone(currentFilter)

for _, player in ipairs(Players:GetPlayers()) do
    if player.Character then
        -- 既存フィルターに含まれていなければ追加
        local alreadyIncluded = table.find(currentFilter, player.Character)
        if not alreadyIncluded then
            table.insert(updatedFilter, player.Character)
        end
    end
end

raycastParams.FilterDescendantsInstances = updatedFilter
```

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

_技術パターン・ベストプラクティスに関するドキュメント_
