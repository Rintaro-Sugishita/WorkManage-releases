# WorkManage プライバシーポリシー / Privacy Policy

最終更新日: 2026-11-15 / Last updated: 2026-11-15

日本語を正文とし、参考として英訳を併記します。/ The Japanese text is authoritative; an English translation follows for reference.

---

## 日本語（正文）

### 1. 基本方針

WorkManage（以下「本ソフトウェア」）は、作業時間の記録・集計・レポートを行う Windows デスクトップアプリケーションです。

**本ソフトウェアは、利用者のデータを開発者へ送信しません。利用状況の収集（テレメトリ）も一切行いません。** 記録されたデータは、既定では利用者の PC の中だけに保存されます。

### 2. 本ソフトウェアが記録する情報

作業時間を自動的に記録するという目的のため、本ソフトウェアは次の情報を取得します。

#### 2.1 アクティブウィンドウの情報（既定で有効）

利用者が操作している最前面ウィンドウの**タイトル**と**プロセス名**を定期的に記録します。これは「どのアプリでどんな作業をしていたか」を後から振り返るための機能です。

- 設定画面から**記録を停止**できます。
- **マスクパターン**（正規表現）を設定することで、タイトルの一部を伏せた状態で記録・送信できます。既定で、4桁以上の数字・ファイルパス・ファイル名を伏せる設定が入っています。
- **保持日数**を設定すると、期間を過ぎた履歴は自動的に削除されます。既定は無期限（自動削除しない）です。

#### 2.2 操作の有無（離席の判定）

離席していた時間を作業時間から除くため、本ソフトウェアはキーボードとマウスのイベントフックを使用します。

**取得するのは「入力操作があったかどうか」だけです。押されたキーの種類、入力された文字、マウスの座標やクリック対象は、一切記録も送信もしません。** いわゆるキーロガーの動作は行いません。

#### 2.3 利用者が入力した作業記録

プロジェクト名、工程名、作業時間、コメントなど、利用者自身が入力した内容を保存します。

### 3. データの保存場所

いずれも利用者の PC 内のみです。

| 内容 | 保存場所 |
|---|---|
| アプリの設定・実行ログ | `%APPDATA%\WorkManage` |
| 作業記録・履歴・レポート | 利用者が設定画面で指定したデータフォルダ |

### 4. 外部への送信

既定では、本ソフトウェアは外部へ一切データを送信しません。次の2つの機能は**既定で無効**であり、利用者が明示的に有効化した場合にのみ通信が発生します。

#### 4.1 AI による作業内容の分析（既定で無効）

有効にすると、マスク適用後のウィンドウタイトルなどを、**利用者自身が設定した AI プロバイダ**（OpenAI、Anthropic 等）へ送信し、作業内容の推定を行います。

- 認証には**利用者自身の API キー**を使用します。開発者は関与しません。
- 送信先・送信内容は、利用者の設定に従います。
- 送信されたデータの取り扱いは、各 AI プロバイダの定めるポリシーに従います。

#### 4.2 データベース連携（既定で無効）

有効にすると、作業記録を**利用者が指定した PostgreSQL サーバー**へ送信します。送信先は利用者が指定したものに限られます。

### 5. 拡張機能について

本ソフトウェアは、利用者が登録した外部プログラム（拡張機能）を起動し、データを受け渡す仕組みを持っています。

**拡張機能は本ソフトウェアの一部ではありません。** 拡張機能がデータをどのように扱うかは、その拡張機能の提供者の責任範囲です。信頼できる提供元のものだけを登録してください。

### 6. 第三者への提供

本ソフトウェアは、利用者のデータを第三者へ販売、貸与、または提供しません。開発者がデータを受け取ることがないため、そもそも提供できる情報を保持していません。

### 7. データの削除

- アクティブウィンドウ履歴は、設定画面の保持日数で自動削除できます。
- 保存先フォルダごと削除すれば、作業記録はすべて消去されます。
- アプリをアンインストールしても、利用者が指定したデータフォルダの内容は残ります。不要な場合は手動で削除してください。

### 8. 本ポリシーの変更

本ポリシーは必要に応じて変更されることがあります。重要な変更がある場合は、公開リポジトリ上で告知するよう努めます。

### 9. お問い合わせ

本ポリシーおよびデータの取り扱いに関するお問い合わせは、公開リポジトリの Issue よりご連絡ください。

https://github.com/Rintaro-Sugishita/WorkManage-releases/issues

---

## English (reference translation)

### 1. Overview

WorkManage (the "Software") is a Windows desktop application for recording, aggregating and reporting working time.

**The Software does not send any of your data to the developer, and collects no usage telemetry.** By default, everything it records stays on your PC.

### 2. What the Software records

To track working time automatically, the Software collects the following.

#### 2.1 Active window information (enabled by default)

The Software periodically records the **title** and **process name** of your foreground window, so that you can review what you were working on.

- You can **turn this off** in settings.
- You can configure **mask patterns** (regular expressions) so that parts of the title are redacted before being recorded or sent. By default, sequences of four or more digits, file paths and file names are masked.
- You can set a **retention period**, after which old history is deleted automatically. The default is unlimited (no automatic deletion).

#### 2.2 Whether input occurred (idle detection)

To exclude time away from the desk, the Software installs keyboard and mouse event hooks.

**It records only whether input occurred. The keys pressed, the characters typed, mouse coordinates and click targets are never recorded or transmitted.** The Software does not behave as a keylogger.

#### 2.3 Work records you enter

Project names, task names, durations and comments that you enter yourself.

### 3. Where data is stored

All of it stays on your PC.

| Data | Location |
|---|---|
| Application settings and execution logs | `%APPDATA%\WorkManage` |
| Work records, history and reports | The data folder you choose in settings |

### 4. Network transmission

By default the Software transmits nothing. The two features below are **disabled by default** and communicate only after you explicitly enable them.

#### 4.1 AI analysis of work content (disabled by default)

When enabled, masked window titles and related data are sent to **the AI provider you configure** (OpenAI, Anthropic, and so on) to estimate what you were working on.

- Authentication uses **your own API key**. The developer is not involved.
- The destination and the content sent follow your own configuration.
- Data you send is handled according to that AI provider's own policy.

#### 4.2 Database integration (disabled by default)

When enabled, work records are sent to **a PostgreSQL server that you specify**, and only there.

### 5. Extensions

The Software can launch external programs (extensions) that you register, and exchange data with them.

**Extensions are not part of the Software.** How an extension handles data is the responsibility of whoever supplies it. Register only extensions from sources you trust.

### 6. No disclosure to third parties

The Software does not sell, lend or disclose your data to third parties. Since the developer never receives your data, there is nothing to disclose.

### 7. Deleting your data

- Active window history can be deleted automatically via the retention setting.
- Deleting your data folder removes all work records.
- Uninstalling the app does not remove the data folder you chose; delete it manually if you no longer need it.

### 8. Changes to this policy

This policy may be revised as needed. Significant changes will be announced in the public repository where practical.

### 9. Contact

For questions about this policy or about data handling, please open an issue in the public repository.

https://github.com/Rintaro-Sugishita/WorkManage-releases/issues
