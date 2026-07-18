# WorkManage 拡張機能インターフェース仕様書

## 1. 概要

WorkManage の拡張機能は **外部 EXE** として実装します。  
本体は EXE を起動し、引数に JSON データを渡します。EXE は処理結果を標準出力に JSON で返し、プロセスの終了コードでリターンコードを伝えます。  
DLL 方式は使用しません（共通ライブラリのバージョン差異による不具合防止のため）。

---

## 2. YAML 設定ファイル形式

拡張機能は **YAML ファイル** で登録します。  
本体の設定画面「拡張機能」タブからインポートします。

### 2.1 トップレベルフィールド

| フィールド | 型 | 必須 | 説明 |
|---|---|---|---|
| `Name` | string | ○ | 拡張機能の識別名 |
| `Description` | string | | 説明 |
| `Author` | string | | 作成者 |
| `CreatedDate` | datetime | | 作成日 |
| `Version` | string | | バージョン文字列 |
| `Path` | string | | 全 Items の既定の EXE パス（Items 個別で上書き可） |
| `SettingPath` | string | | 設定ファイルのパス（設定ボタンで開く） |
| `SettingArgs` | string[] | | `SettingPath` 起動時に渡す追加引数 |
| `Items` | list | ○ | イベント・ボタン・カスタムカラムの定義リスト |

### 2.2 Items 共通フィールド

すべての Item に共通のフィールドです。

| フィールド | 型 | 必須 | 説明 |
|---|---|---|---|
| `ExType` | `"function"` \| `"button"` \| `"custom"` | ○ | Item の種別 |
| `Name` | string | ○ | Item の識別名 |
| `Description` | string | | 説明 |
| `Path` | string | | EXE パス（相対パスはYAMLファイルからの相対） |
| `NeedWait` | bool | | `true` の場合、EXE 終了を待って結果を受け取る。`false` の場合は起動して即返す（既定: `false`） |
| `Priority` | int | | 同一イベントに複数登録した場合の実行順（小さいほど先。既定: 0） |

### 2.3 ExType: function（イベント関数）

イベント発生時に呼ばれる処理を定義します。

追加フィールド：

| フィールド | 型 | 必須 | 説明 |
|---|---|---|---|
| `ExtensionEventName` | string | ○ | 対象イベント名（[4章参照](#4-イベント一覧)） |
| `Arguments` | string[] | | EXE に渡す固定の追加引数。イベントデータ `-d <JSON>` の前に付加される |

### 2.4 ExType: button（ボタン）

指定画面にボタンを追加します。ボタンクリック時に EXE を起動します。

追加フィールド：

| フィールド | 型 | 必須 | 説明 |
|---|---|---|---|
| `Screen` | string | ○ | ボタンを追加する画面名（[5章参照](#5-screentypeスクリーン種別)） |
| `ButtonText` | string | ○ | ボタンのラベル文字列 |
| `Arguments` | string[] | | EXE に渡す固定の追加引数 |

ボタンの EXE は `NeedWait: false` の場合、起動後すぐに制御を返します。

### 2.5 ExType: custom（カスタムカラム）

作業一覧グリッドにカラムを追加します。EXE の起動はありません。

追加フィールド：

| フィールド | 型 | 必須 | 説明 |
|---|---|---|---|
| `Key` | string | ○ | カラムの識別キー。ドットなしの場合は `{拡張機能名}.{Key}` が完全キーになる |
| `Type` | string | ○ | カラム型: `text` / `comboBox` / `button` |
| `IsReadOnly` | bool | | 読み取り専用にする（既定: `false`） |
| `Visibility` | string | | `Visible` / `Hidden` / `Collapsed`（既定: `Visible`） |
| `Width` | string | | 列幅（数値 or `"*"` or `"Auto"`） |
| `Options` | list | | `comboBox` 型の選択肢（`Value` / `Label` を持つオブジェクトのリスト） |
| `IsGroupingKey` | bool | | `true` の場合、報告時のグループ化キーとして扱う |
| `MergeType` | string | | 報告グループ化時の統合方法: `first` / `last` / `concat` |
| `DefaultValue` | string | | 新規行作成時の既定値 |

---

## 3. EXE 呼び出し規約

### 3.1 引数フォーマット

本体はイベントデータを以下の引数で EXE に渡します。

```
<exe_path> [固定引数...] -d <JSON文字列>
```

- 固定引数は YAML の `Arguments` フィールドで定義します
- `-d` の後に続く JSON は、イベントごとの **InputData** オブジェクトをシリアライズしたものです

### 3.2 標準出力（戻り値）

`NeedWait: true` のとき、本体は EXE の標準出力を読み取ります。  
出力は **OutputData** オブジェクトの JSON 文字列を 1 行で出力してください。

```json
{"ExceptionText":"","Works":[...],"StartTime":"2025-01-01T09:00:00"}
```

- 出力が不要な場合は空文字列または `{}` を出力します
- エラー時は `ExceptionText` フィールドにエラーメッセージをセットしてください（本体がメッセージボックスで表示します）

### 3.3 終了コード（リターンコード）

| 終了コード | 定数名 | 説明 |
|---|---|---|
| `0` | `Continue` | 正常終了。本体の既定処理を続行する |
| `-1` | `Cancel` | 処理をキャンセルする。後続の拡張・本体処理を停止する |
| `1` | `Fail` | エラー終了。後続の処理を停止する |
| `2` | `Overwrite` | 正常終了。本体の既定処理を **上書き**（スキップ）する |

同一イベントに複数の拡張が登録されている場合：
- `Cancel` または `Fail` が返ったら、その時点で残りの拡張の呼び出しを中断します
- `Overwrite` が返った場合、全拡張を実行し終えた後に本体の既定処理をスキップします
- `Continue` のみであれば本体の既定処理を実行します

---

## 4. イベント一覧

### 4.1 起動・ロード系

| イベント名 | タイミング | InputData | OutputData | NeedWait |
|---|---|---|---|---|
| `Loading` | アプリ起動直後（データロード前） | `LoadInputData` | `OutputBase` | 任意 |
| `Loaded` | 全データロード完了後 | `LoadInputData` | `OutputBase` | 任意 |
| `LoadingProject` | プロジェクト一覧ロード後 | `ProjectInputData` | `ProjectOutData` | 推奨 `true` |

#### LoadInputData
```json
{
  "LastBootTime": "2025-01-01T09:00:00",
  "LastBootVersion": "3.4.0"
}
```

#### ProjectInputData / ProjectOutData
```json
{
  "Projects": [
    { "ProjectCode": "001", "CommonName": "プロジェクトA", "OfficialName": "Project Alpha", "IsValid": true }
  ]
}
```
`ProjectOutData.Projects` が空でない場合、プロジェクト一覧をその内容で上書きします。

---

### 4.2 保存系

| イベント名 | タイミング | InputData | OutputData | NeedWait |
|---|---|---|---|---|
| `DataSaving` | 生データ保存直前（`SaveRawLog` 呼び出し時） | `SavingInputData` | `OutputBase` | 任意 |
| `DataSaved` | 生データ保存直後 | `SavingInputData` | `OutputBase` | 任意 |

#### SavingInputData
```json
{
  "Date": "2025-01-15T00:00:00",
  "Works": [
    {
      "Id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "ProjectCode": "001",
      "ProjectName": "プロジェクトA",
      "ProcessCode": "10",
      "ProcessName": "設計",
      "Content": "基本設計書作成",
      "Tag": "設計",
      "Memo": "",
      "WorkTime": "02:30:00",
      "AdjustTime": "00:00:00",
      "CustomItems": {}
    }
  ]
}
```

---

### 4.3 残業系

| イベント名 | タイミング | InputData | OutputData | NeedWait |
|---|---|---|---|---|
| `OvertimeSelecting` | 残業予定入力ダイアログ表示前 | `OvertimeWorkInputData` | `OvertimeWorkOutData` | 推奨 `true` |
| `OvertimeCommitting` | 残業予定確定直前 | `OvertimeWorkInputData` | `OvertimeWorkOutData` | 推奨 `true` |

#### OvertimeWorkInputData / OvertimeWorkOutData
```json
{
  "StartTime": "2025-01-15T18:00:00",
  "EndTime": "2025-01-15T20:00:00",
  "Remarks": "緊急対応"
}
```
`OvertimeWorkOutData` の `StartTime` / `EndTime` / `Remarks` を返すと本体の値を上書きします。

---

### 4.4 報告系

報告操作は「一括報告」「時刻を報告」「作業内容を報告」の 3 種に分かれており、それぞれ異なるイベントを発火します。

#### 一括報告（時刻選択 → 集計 → 確認 → 報告）

| イベント名 | タイミング | InputData | OutputData | NeedWait |
|---|---|---|---|---|
| `SelectingTime` | 時刻選択ダイアログ表示前 | `TimeSelectInputData` | `TimeSelectOutputData` | 推奨 `true` |
| `SelectedTime` | 時刻選択後 | `TimeSelectInputData` | `TimeSelectOutputData` | 任意 |
| `ReportGrouping` | 作業集計・グループ化後、確認前 | `ReportingInputData` | `ReportingOutData` | 推奨 `true` |
| `ReportConfirming` | 確認ダイアログ表示前 | `ReportingInputData` | `ReportingOutData` | 推奨 `true` |
| `Reporting` | 報告確定時（既定の報告処理直前） | `ReportingInputData` | `ReportingOutData` | 推奨 `true` |
| `Reported` | 報告完了後 | `ReportingInputData` | `OutputBase` | 任意 |

#### 時刻を報告（時刻選択のみ・保存）

| イベント名 | タイミング | InputData | OutputData | NeedWait |
|---|---|---|---|---|
| `ReportTimeSelecting` | 時刻選択ダイアログ表示前 | `TimeSelectInputData` | `TimeSelectOutputData` | 推奨 `true` |
| `ReportTimeSelected` | 時刻保存後 | `TimeSelectInputData` | `TimeSelectOutputData` | 任意 |

#### 作業内容を報告（保存済み時刻を使用して集計 → 確認 → 報告）

保存済みの時刻がない場合は時刻選択ダイアログを表示します（この場合は一括報告と同じ `SelectingTime` / `SelectedTime` を発火）。

| イベント名 | タイミング | InputData | OutputData | NeedWait |
|---|---|---|---|---|
| `WorkReportGrouping` | 作業集計・グループ化後、確認前 | `ReportingInputData` | `ReportingOutData` | 推奨 `true` |
| `WorkReportConfirming` | 確認ダイアログ表示前 | `ReportingInputData` | `ReportingOutData` | 推奨 `true` |
| `WorkReporting` | 報告確定時（既定の報告処理直前） | `ReportingInputData` | `ReportingOutData` | 推奨 `true` |
| `WorkReported` | 報告完了後 | `ReportingInputData` | `OutputBase` | 任意 |

#### TimeSelectInputData / TimeSelectOutputData
```json
{
  "SelectedDate": "2025-01-15T00:00:00",
  "StartTime": "2025-01-15T09:00:00",
  "EndTime": "2025-01-15T18:00:00"
}
```
`TimeSelectOutputData` の各フィールドを返すと本体の選択時刻を上書きします。

#### ReportingInputData / ReportingOutData
```json
{
  "StartTime": "2025-01-15T09:00:00",
  "EndTime": "2025-01-15T18:00:00",
  "Works": [
    {
      "Id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "ProjectCode": "001",
      "ProjectName": "プロジェクトA",
      "ProcessCode": "10",
      "ProcessName": "設計",
      "Content": "基本設計書作成",
      "Tag": "設計",
      "WorkTime": "02:30:00",
      "AdjustTime": "00:00:00",
      "CustomItems": { "MyExtension.TicketNo": "TICKET-123" }
    }
  ]
}
```
`ReportingOutData.Works` が空でない場合、作業一覧をその内容で上書きします。  
`ReportGrouping` で `Overwrite` を返すと本体の集計処理をスキップできます。  
`Reporting` で `Overwrite` を返すと本体の既定報告処理（メッセージボックス表示）をスキップできます。

---

## 5. ScreenType（スクリーン種別）

`button` の `Screen` フィールドで指定します。

| 値 | 画面 |
|---|---|
| `MainWindow` | メインウィンドウ（作業一覧の左メニュー） |
| `ProjectSetting` | 設定画面 > プロジェクト |
| `ProcessSetting` | 設定画面 > 工程 |
| `LogWindow` | ログウィンドウ |
| `ExtensionSetting` | 拡張機能設定ウィンドウ |

---

## 6. OutputBase（全 OutputData の共通フィールド）

```json
{
  "ExceptionText": ""
}
```

- `ExceptionText` が空でない場合、本体はエラーダイアログを表示して処理を中断します

---

## 7. WorkViewItem（作業アイテム）

InputData / OutputData に含まれる作業アイテムの形式です。

| フィールド | 型 | 説明 |
|---|---|---|
| `Id` | GUID string | 作業アイテムの識別子 |
| `ProjectCode` | string | プロジェクトコード |
| `ProjectName` | string | プロジェクト名（表示用） |
| `ProcessCode` | string | 工程コード |
| `ProcessName` | string | 工程名（表示用） |
| `Content` | string | 作業内容 |
| `Tag` | string | タグ |
| `Memo` | string | メモ |
| `WorkTime` | TimeSpan string | 計測作業時間（`"HH:mm:ss"` 形式） |
| `AdjustTime` | TimeSpan string | 調整時間（`"HH:mm:ss"` 形式） |
| `CustomItems` | object | カスタムカラムの値（キー: `{拡張機能名}.{カラムキー}`） |

---

## 8. サンプル YAML

```yaml
Name: MyReporter
Description: チケットシステムへ報告する拡張機能
Author: Your Name
CreatedDate: 2025-01-01
Version: 1.0.0
Path: ./MyReporter.exe
SettingPath: ./MyReporter.config.json

Items:
  # 起動時にサーバー接続確認
  - ExType: function
    Name: ConnectionCheck
    Description: 起動時にサーバー接続確認
    ExtensionEventName: Loading
    NeedWait: true

  # 報告時にチケットシステムへ送信
  - ExType: function
    Name: SendToTicket
    Description: 報告確定時にチケットシステムへ送信
    ExtensionEventName: Reporting
    NeedWait: true
    Priority: 10

  # 保存のたびにデータをキャッシュ
  - ExType: function
    Name: CacheOnSave
    Description: 保存時にローカルキャッシュへ保存
    ExtensionEventName: DataSaved
    NeedWait: false

  # メインウィンドウにボタン追加
  - ExType: button
    Name: OpenDashboard
    Description: ダッシュボードを開く
    Screen: MainWindow
    ButtonText: ダッシュボード
    NeedWait: false

  # チケット番号カラムを追加
  - ExType: custom
    Name: TicketNo
    Description: チケット番号
    Key: TicketNo
    Type: text
    IsReadOnly: false
    Width: 100
    IsGroupingKey: false
    MergeType: first
```

### サンプル EXE 実装（C# / .NET）

```csharp
// Program.cs
using System.Text.Json;

var args = Args.Parse(Environment.GetCommandLineArgs());
var input = ReportingInputData.FromJson(args.Data);

var output = new ReportingOutData();
try
{
    // 処理...
    foreach (var work in input.Works)
    {
        await TicketSystem.SendAsync(work);
    }
}
catch (Exception ex)
{
    output.ExceptionText = ex.Message;
    Console.WriteLine(output.ToJson());
    Environment.Exit(1);  // Fail
    return;
}

Console.WriteLine(output.ToJson());
Environment.Exit(0);  // Continue
```

---

## 9. ベストプラクティス

- **NeedWait は必要なときだけ `true` に**: 待機しない拡張はバックグラウンドで実行されます。報告系など結果を本体に返す必要がある場合のみ `true` にしてください
- **ExceptionText でエラーを通知**: 例外をキャッチして `ExceptionText` に設定してください。標準エラー出力はログには書かれますが本体 UI には表示されません
- **CustomItems のキー命名**: `{拡張機能Name}.{カラムKey}` の形式で他の拡張との衝突を避けてください
- **Priority で実行順を制御**: 同一イベントに複数の拡張を登録する場合、`Priority` 値の小さいものから実行されます
