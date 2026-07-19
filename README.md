# WorkManage

**PC での作業を記録し、日報・残業集計までまとめて行う Windows デスクトップアプリ。**
インストール不要（単一 exe・.NET ランタイム不要）— ダウンロードして起動するだけ。

<!-- ここにスクリーンショットや操作GIFを貼ると、初見の人に一気に伝わります
![WorkManage overview](docs/images/overview.png)
-->

## なぜ WorkManage？

- 🟢 **入れてすぐ使える** — 自己完結の単一 exe。.NET のインストールや面倒なセットアップは不要。
- ⏱ **作業時間を可視化** — アクティブウィンドウの履歴からタイムラインを生成。手作業の打刻を減らせます。
- ✍️ **タイムラインで直感編集** — ドラッグで移動・幅変更、分割、手動追加。1 分単位で微調整できます。
- 📄 **日報・残業をワンストップ** — 記録から日報作成・残業集計まで一気通貫。
- 🗄 **データベース連携** — 記録をそのまま DB へ送信可能。
- 🧩 **拡張できる** — 拡張機能（プラグイン）で自分の運用に合わせられます。

## ダウンロード / インストール

[Releases](https://github.com/Rintaro-Sugishita/WorkManage-releases/releases) から最新版の `WorkManage-vX.Y.Z.zip` をダウンロード → 展開 → `WorkManage.exe` を実行。

- 自己完結（SelfContained）・単一ファイルのため **.NET のインストールは不要**。
- 対応 OS: Windows 10 (1809) 以降 / x64

## 使い方（3 ステップ）

1. 起動して作業を記録する
2. タイムライン ビューアで内容を確認・微調整する
3. 日報 / 残業レポートを出力する

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

[LICENSE.md](LICENSE.md) をご確認ください。**無償・個人/商用利用可**、**無改変での再配布可**（クローズドソース。MIT 等の OSS ライセンスではありません）。
