# WorkManage 拡張機能 開発ガイド

`WorkManage.Contracts` を参照して WorkManage の拡張機能（外部 exe）を開発するための手引きです。

---

## 1. 拡張機能の仕組み（概要）

WorkManage は特定のタイミング（イベント）で、登録された**外部実行ファイル(exe)** を呼び出します。

```
[WorkManage 本体] --(イベント発火)--> あなたの拡張exe を起動
   ・引数 -d <入力JSON> を渡す
   ・拡張exe は 標準出力(stdout) に <出力JSON> を書く
   ・終了コードで本体の後続動作を制御する
```

- 拡張は「YAML 設定ファイル」で登録します。
- 1つのイベントに複数の拡張を登録でき、`Priority` 順に呼ばれます。
- データの受け渡しは JSON（本ライブラリの DTO クラス）です。

> 補足: 現状サポートされるのは **exe 型** の拡張です。

---

## 2. 全体の流れ

1. あなたが拡張 exe を作る（C# コンソールアプリなど）。
2. YAML でイベントと exe を紐づけて登録する。
3. 本体が該当イベントに達すると、`-d <入力JSON>` を付けて exe を起動する。
4. exe は入力 JSON を `WorkManage.Contracts` の `InputData` 型にデシリアライズして処理する。
5. exe は結果を対応する `OutputData` 型の JSON にして **stdout** へ出力する。
6. exe は**終了コード**で本体の後続処理を指示する（下表）。

### 終了コード（`ExecutedReturnCode`）

| コード | 意味 | 本体の挙動 |
|---:|---|---|
| `0` | Continue | そのまま続行（既定） |
| `-1` | Cancel | 以降の処理を中断 |
| `1` | Fail | 失敗として中断 |
| `2` | Overwrite | 本体の既定処理を上書き（拡張の結果を採用） |

出力 JSON の `ExceptionText`（`OutputBase` の項目）に文字列を入れると、本体側で例外として扱われ**ユーザーにメッセージ表示**されます。

---

## 3. YAML での登録

拡張設定ファイル（本体の設定フォルダに配置）に、`Items` として関数拡張を追加します。

```yaml
Name: MyExtension
Description: 報告完了時に外部システムへ送信する拡張
Author: your-name
Version: 1.0.0
Items:
  - ExType: function          # 関数(イベント)拡張
    Name: send-on-reported
    ExtensionEventName: Reported   # ← ExtensionEvents の名前と一致させる
    Path: ex/MyExtension.exe       # 実行ファイル(YAMLからの相対 or 絶対)
    NeedWait: true                 # 出力/終了コードを待つ場合は true
    Priority: 100                  # 小さいほど先に実行
    Arguments: []                  # -d の前に付ける追加引数（任意）
```

`ExtensionEventName` は後述の `ExtensionEvents` 列挙体の**名前と完全一致**させます（不一致の登録は無視されます）。

---

## 4. イベント一覧（`WorkManage.ExtensionEvents`）

代表的な発火タイミングと、渡される入力 DTO の目安です（正確なプロパティは各 `InputData`/`OutputData` クラスを参照）。

| イベント | タイミング | 主な入出力 DTO |
|---|---|---|
| `Loading` / `Loaded` | 起動・読み込み時 | `LoadInputData` / `SavingInputData` |
| `LoadingProject` / `LoadedProject` | プロジェクト読み込み | `ProjectInputData` / `ProjectOutData` |
| `DataSaving` / `DataSaved` | データ保存前後 | `SavingInputData` |
| `SelectingTime` / `SelectedTime` | 時刻選択の前後 | `TimeSelectInputData` / `TimeSelectOutputData` |
| `ReportConfirming` / `Reporting` / `Reported` / `ReportGrouping` | 一括報告の各段階 | `ReportingInputData` / `ReportingOutData` |
| `ReportTimeSelecting` / `ReportTimeSelected` / `ReportTimeReported` | 「時刻を報告(打刻)」の各段階 | `TimeSelectInputData` / `TimeSelectOutputData` |
| `WorkReportGrouping` / `WorkReportConfirming` / `WorkReporting` / `WorkReported` | 「作業内容を報告」の各段階 | `ReportingInputData` / `ReportingOutData` |
| `OvertimeSelecting` / `OvertimeSelected` / `OvertimeCommitting` | 残業入力の各段階 | 残業関連 DTO |
| `CopyLog` / `SavedProject` | ログコピー・プロジェクト保存 | 各対応 DTO |

代表的な DTO（`WorkManage.InputData` / `WorkManage.OutputData`）:

- `ReportingInputData` … `Works`(作業行), `StartTime`, `EndTime`
- `TimeSelectInputData` … `SelectedDate`, `StartTime`, `EndTime`
- `SavingInputData` … `Works`, `Date`
- 出力系はすべて `OutputBase` を継承し、`ExceptionText` を持ちます。

作業行 1件は `WorkManage.Data.WorkViewItem`（`ProjectCode` / `ProjectName` / `Content` / `Tag` / `Memo` / `ProcessCode` / `ProcessName` / `WorkTime` / `WorkDay` / `CustomItems` など）で表されます。

---

## 5. 最小サンプル（C# コンソール拡張）

`WorkManage.Contracts` を参照した exe の例です。`Reported`（報告完了）で呼ばれ、受け取った作業を標準エラーにログ出力して正常終了します。

```csharp
using WorkManage.InputData;
using WorkManage.OutputData;

// 起動引数から -d <JSON> を取り出す
string? json = null;
for (int i = 0; i < args.Length - 1; i++)
    if (args[i] == "-d") { json = args[i + 1]; break; }

if (json is null) return 0; // 入力が無ければ何もしない(Continue)

// 入力JSONを Contracts の DTO にデシリアライズ
var input = ReportingInputData.FromJson(json);

// …ここで外部システム送信などの処理…
Console.Error.WriteLine($"reported works = {input?.Works.Count}, {input?.StartTime}〜{input?.EndTime}");

// 出力(必要なら)。ExceptionText を入れると本体側でエラー表示される
var output = new ReportingOutData();
Console.Out.Write(output.ToJson(false));

return 0; // 0 = Continue（続行）
```

ポイント:
- 入力/出力の JSON 変換は各 DTO の `FromJson` / `ToJson`（`WorkManage.Contracts` が提供）を使う。
- `-d` の次の引数が入力 JSON（`ProcessStartInfo.ArgumentList` 渡しなので手動アンエスケープ不要）。
- `NeedWait: true` のイベントでは stdout の JSON と終了コードが本体に読まれる。

---

## 6. 注意点

- `ExtensionEventName` は `ExtensionEvents` の名称と大文字小文字まで一致させること。
- 出力 JSON は **stdout のみ**。ログや診断は **stderr** に出す（stdout を JSON 以外で汚さない）。
- 例外や失敗はプロセスをクラッシュさせず、`ExceptionText` か終了コードで返す方が本体で扱いやすい。
- 各イベントで渡る DTO の**正確なプロパティ**は、参照している `WorkManage.Contracts` の該当クラス定義を確認すること。
