# WorkManage.Contracts

WorkManage 拡張機能開発のための共有コントラクトライブラリです。

## 概要

WorkManage アプリケーションの拡張機能を開発する際に必要な、データモデル・入出力データクラス・拡張イベント定義を提供します。

## 含まれる主なクラス

| 名前空間 / クラス | 説明 |
|---|---|
| `WorkManage.Data.WorkClass` | 作業データのメインクラス |
| `WorkManage.Data.ProjectData` | プロジェクトデータ |
| `WorkManage.Data.ProcessData` | 工程データ |
| `WorkManage.ExtensionEvents` | 拡張機能イベント定義 |
| `WorkManage.ScreenType` | 画面種別の列挙体 |
| `WorkManage.WorkManageExtensionMethods` | 拡張メソッド群 |
| `WorkManage.SettingClass` | 設定クラス |

## インストール

```
dotnet add package WorkManage.Contracts
```

## 依存関係

- .NET 9.0 (Windows 10 1803 以降)
- FrostNova.Core

## ライセンス

本プロジェクトのライセンスに準じます。
