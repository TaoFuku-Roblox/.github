# Roblox プロジェクトセットアップガイド

Roblox プロジェクトの初期設定手順とツール構成を説明する。

## 必須ツール

| ツール | 用途 | インストール |
|--------|------|--------------|
| [Rokit](https://github.com/rojo-rbx/rokit) | ツールチェーン管理 | [公式ガイド参照](https://github.com/rojo-rbx/rokit#installation) |
| [Rojo](https://rojo.space/) | ファイルシステム ↔ Roblox Studio 同期 | Rokit 経由 |

## 新規プロジェクト作成手順

### 1. リポジトリ作成

```bash
# GitHub でリポジトリを作成後
git clone https://github.com/TaoFuku-Roblox/<project-name>.git
cd <project-name>
```

### 2. Rokit 初期化

```bash
rokit init
rokit add rojo-rbx/rojo
rokit install
```

これにより `rokit.toml` が作成される:

```toml
[tools]
rojo = "rojo-rbx/rojo@7.x.x"
```

### 3. ディレクトリ構造作成

```bash
mkdir -p src/server src/client src/shared
```

**最小構成**:
```
project/
├── src/
│   ├── server/     # ServerScriptService
│   ├── client/     # StarterPlayerScripts
│   └── shared/     # ReplicatedStorage
├── default.project.json
└── rokit.toml
```

### 4. default.project.json 作成

**最小コアテンプレート**:

```json
{
  "name": "project-name",
  "servePort": 34872,
  "tree": {
    "$className": "DataModel",
    "ServerScriptService": {
      "$path": "src/server"
    },
    "ReplicatedStorage": {
      "$path": "src/shared"
    },
    "StarterPlayer": {
      "StarterPlayerScripts": {
        "$path": "src/client"
      }
    }
  }
}
```

## 拡張パターン

ゲームの種類に応じて以下の設定を追加する。

| 用途 | サービス | 設定 | 使用例 |
|------|----------|------|--------|
| UI | StarterGui | `"StarterGui": { "$path": "src/starter/UI" }` | HUD、メニュー |
| ツール/武器 | StarterPack | `"StarterPack": { "$path": "src/starter/Pack" }` | インベントリアイテム |
| サーバー専用アセット | ServerStorage | `"ServerStorage": { "$path": "src/storage" }` | 大量データ、秘匿アセット |
| 静的マップ | Workspace | `"Workspace": { "$path": "assets/map" }` | 地形、建物 |

## Rojo 設定オプション

### ignoreUnknownInstances

Rojo で管理していないインスタンス（Studio で直接作成したパーツ等）の扱いを制御する。

| 設定値 | 動作 | 使用ケース |
|--------|------|----------|
| `true`（デフォルト） | 管理外インスタンスを削除 | コードのみのプロジェクト |
| `false` | 管理外インスタンスを保持 | マップ、Lighting、Terrain を Studio で作成 |

**設定例**:

```json
{
  "name": "my-game",
  "tree": {
    "$className": "DataModel",
    "Workspace": {
      "$ignoreUnknownInstances": false
    },
    "Lighting": {
      "$ignoreUnknownInstances": false
    },
    "Terrain": {
      "$ignoreUnknownInstances": false
    },
    "ServerScriptService": {
      "$path": "src/server"
    }
  }
}
```

**推奨パターン**:

| サービス | 推奨設定 | 理由 |
|----------|----------|------|
| Workspace | `false` | マップ、パーツを Studio で配置することが多い |
| Lighting | `false` | 照明設定は Studio で調整することが多い |
| Terrain | `false` | 地形は Studio のツールで編集する |
| ServerScriptService | `true`（デフォルト） | コードは全て Rojo で管理 |
| ReplicatedStorage | `true`（デフォルト） | コードは全て Rojo で管理 |

### その他のオプション

| オプション | 説明 | デフォルト |
|-----------|------|-----------|
| `servePort` | `rojo serve` のリッスンポート | `34872` |
| `servePlaceIds` | ライブ同期を許可する Place ID リスト | `[]` |
| `globIgnorePaths` | 除外するパスの Glob パターン | `[]` |

**参照**: [Rojo Project Format](https://rojo.space/docs/v7/project-format/)

## 開発ワークフロー

### ライブ同期（開発時）

```bash
rojo serve
```

Roblox Studio で Rojo プラグインを使用して接続。

### ビルド（本番用）

```bash
# 本番ビルド
rojo build -o project-name.rbxl

# 開発ビルド（.gitignore に追加推奨）
rojo build -o project-name-dev.rbxl
```

### .gitignore 推奨設定

```gitignore
# Development build
*-dev.rbxl

# Roblox Studio lock files
*.rbxlx.lock
*.rbxl.lock
```

## 参照

- [Rojo 公式ドキュメント](https://rojo.space/docs/v7/)
- [Rokit GitHub](https://github.com/rojo-rbx/rokit)
- [ROBLOX_CODING_STANDARDS.md](./ROBLOX_CODING_STANDARDS.md)
- [ROBLOX_TECHNICAL_PATTERNS.md](./ROBLOX_TECHNICAL_PATTERNS.md)
