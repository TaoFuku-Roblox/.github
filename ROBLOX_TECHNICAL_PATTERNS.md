# Roblox 技術パターン

Robloxゲーム開発で再利用可能な技術パターンとベストプラクティス。

---

## Luau言語パターン

### 関数定義順序

Luauでは関数を呼び出す前に定義されている必要がある。ヘルパー関数は呼び出し元より前に定義。相互参照が必要な場合は変数宣言後に代入。

### 空配列・ゼロ値の判定

Luaでは`0`や空配列`{}`はtruthyであり、falsyではない。`0`の判定は`~= nil`で明示チェック。

---

## プレイヤーテレポート

### 安全なテレポートパターン

1. `humanoidRootPart.CFrame = CFrame.new(targetPosition)`
2. 直後に`Anchored = true`で物理を一時停止（しないと意図しない移動が発生）
3. `task.wait(0.05)`で物理エンジン安定化
4. `Anchored = false`で物理再開

**注意**: WorkspaceのSpawnLocationは自動再配置を引き起こすため、サーバー起動時に削除が必要。

---

## アバタークローン

### 重要ポイント

1. **`character.Archivable = true`** を設定しないと`Clone()`が`nil`を返す（最重要）
2. **`Model:PivotTo(cf)`** でモデル全体を移動（個別CFrame設定は不可）
3. 全BasePart を`Anchored = true, CanCollide = false`に設定
4. 不要になったクローンは必ず`:Destroy()`で削除
5. サーバーで作成 → 自動的にクライアントにレプリケーション

---

## Roblox固有の制約

### 移動能力の復元

テレポート後やゲームオーバー時に`humanoid.WalkSpeed`と`JumpPower`が0のままになる場合がある。明示的にデフォルト値を設定して復元。

### Zファイティング対策

同一高さの重なるパーツはちらつきが発生。各レイヤーに微小な高さオフセット（0.01 studs）を追加。

### 二重検出防止パターン

Touchedイベント等が短時間に複数回発火する問題。`isProcessing`フラグ + `task.delay()`でクリア。

### 物理シミュレーション

物理計算の理論値と実測値には乖離がある。物理ベースのゲームでは必ず実測データに基づいてパラメータを調整。

### Terrain APIボクセル解像度

4 studボクセル解像度で動作。`FillBlock()`等で指定したY座標と実際の表面Y座標にずれが生じる（例: Y=40指定 → 表面Y≈42）。

### RaycastParams FilterDescendantsInstances

- 遅延生成フォルダはフォルダ作成後にフィルターを更新する
- フィルターリストは`table.clone()`でコピーしてから変更し、新しい配列を代入
- プレイヤーキャラクターは途中参加・リスポーンで動的生成されるため、Raycast時にリアルタイムで追加

---

## 非推奨API

| 非推奨 | 代替 |
|--------|------|
| `BodyVelocity` | `AssemblyLinearVelocity` |
| `BodyForce` | `VectorForce` |
| `BodyPosition` | `AlignPosition` |
| `BodyGyro` | `AlignOrientation` |
| `player.CharacterAutoLoads` | `Players.CharacterAutoLoads`（サービスプロパティ） |

---

_技術パターン・ベストプラクティスに関するドキュメント_
