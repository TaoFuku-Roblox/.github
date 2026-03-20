# Roblox コーディング規約

> 開発ワークフロー・ブランチ戦略・Issue/PR規約は [DEV_WORKFLOW.md](./DEV_WORKFLOW.md) を参照。

---

## コア技術

- **言語**: Luau (`--!strict` モード)
- **ビルドツール**: Rojo
- **型管理**: `Types.lua`で全型定義を一元管理（`export type`）
- **エラーハンドリング**: `ErrorHandler`モジュールで一貫したログ出力
- **バリデーション**: `Validation`モジュールでクライアント入力を検証（エクスプロイト対策）
- **命名規則**: PascalCaseモジュール名、camelCase関数名

---

## テスト戦略

`Instance.new("Player")`が使用不可のため、自動テストは制限される。

1. **手動統合テスト**: 各specの`manual-testing.md`に手順・期待動作・ログ出力・成功基準を記載
2. **コード検査**: Luau `--strict`型チェックとコードレビュー
3. **ログ出力検証**: ErrorHandlerによる実行時動作追跡

---

## ビルドファイル命名規則

| ファイル名 | 用途 |
|-----------|------|
| `{リポジトリ名}.rbxl` | 本番用（Gitで管理） |
| `{リポジトリ名}-dev.rbxl` | 開発用（`.gitignore`で除外） |

```bash
rojo build -o {リポジトリ名}.rbxl    # 本番用ビルド
rojo serve                            # ライブ同期
```

---

_Roblox コーディング規約に関するドキュメント_
