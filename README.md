# WorkManage

作業時間の記録・集計・レポートを行う Windows デスクトップアプリケーションです。

## ダウンロード / インストール

[Releases](https://github.com/Rintaro-Sugishita/WorkManage-releases/releases) から最新版の `WorkManage-vX.Y.Z.zip` をダウンロードし、展開して `WorkManage.exe` を実行してください。

- 自己完結（SelfContained）・単一ファイル発行のため、**.NET のインストールは不要**です。
- 対応 OS: Windows 10 (1809) 以降 / x64

## 主な機能

- 作業時間の記録・集計
- 日報 / 残業レポート
- タイムライン ビューア（手動追加・分割・移動・幅変更）
- データベースへの送信
- 拡張機能（プラグイン）対応

## 拡張機能の開発

拡張機能は `WorkManage.Contracts` パッケージを参照して開発します。

```
dotnet add package WorkManage.Contracts
```

ドキュメント（nupkg をダウンロードしなくても閲覧できます）:

- [拡張開発ガイド (EXTENSION_GUIDE.md)](docs/EXTENSION_GUIDE.md)
- [拡張インターフェース仕様](docs/extension-interface-spec.md)
- [DB スキーマ](docs/db-schema.md)

## ライセンス

本プロジェクトのライセンスに準じます。
