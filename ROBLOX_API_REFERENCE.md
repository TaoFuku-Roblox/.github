# Roblox API リファレンス

Roblox APIの使用例と推奨設定値。

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

_Roblox APIリファレンスに関するドキュメント_
