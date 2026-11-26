# コマンドリファレンス：LxiHiSlip_Open

👉 [← LxiHiSlip アドイン取扱説明書に戻る](./LxiHiSlip取扱説明書.Md)

## 概要

**LxiHiSlip_Open** は、指定された接続先情報またはプロパティ設定値に基づき、LXI 計測器との **HiSLIP / TLS 2.0 通信セッション** を開設する通信制御系コマンドです。
共通基底クラス `LxiOpenCommandBase`（`TermAnalysis.Lxi.Core`）を継承し、`HiSlipTransportClient.Shared` を介して IVI-6.1 仕様に基づき、同一 TCP ポート（既定 `4880`）に対して同期チャネル（`Initialize` ➔ `InitializeResponse`）および非同期チャネル（`AsyncInitialize` ➔ `AsyncInitializeResponse`）の 2 本の接続を `SessionID` で紐付けて確立します。TLS 2.0 有効時は `SslStream` による `StartTLS` 交渉を実行します。

---

## 💡 書式・パラメータ仕様

### コマンド構文

```csv
,SEQUENCE, [ステップキー], LxiHiSlip_Open, "[IP: 192.168.0.10]", "[Port: 4880]", "[SubAddr: hislip0]"
```

※ 引数を省略した場合は、プロパティ設定画面の現在値が自動適用されます。

### パラメータ（引数）一覧

| 引数位置 | 引数名 | 必須/任意 | 説明・制約 |
| :--- | :--- | :---: | :--- |
| 第 1 引数 | `IP` | 任意 | 接続先 HiSLIP 対応計測器の IPv4 / IPv6 アドレスまたはホスト名（既定: UI 設定値）。 |
| 第 2 引数 | `Port` | 任意 | HiSLIP サーバーの TCP ポート番号（`1`〜`65535`、既定: `"4880"`）。 |
| 第 3 引数 | `SubAddr` | 任意 | HiSLIP 論理サブアドレス（既定: `"hislip0"`）。 |

---

## 🔄 特殊置換規約・分岐規約の適用

本コマンドの引数における特殊記法および解釈ルールです。
* **動的置換（`$`）**: 引数内で `$PARAMキー$` のように記述すると、実行時に変数プール（PARAM）の値へ動的置換されます。（適用可能な引数: 全引数対応）
* **ジャンプ先指定（`#`）**: 本コマンドはジャンプ制御を行わないため適用されません。（適用可能な引数: 非適用）
* **共通コマンド指定（`@`）**: 本コマンドは外部マクロを呼び出さないため、共通コマンド呼び出し記法は適用されません。（適用可能な引数: 非適用）
* **カンマのエスケープ**: カンマを含むサブアドレス（例: `"gpib0,22"`）を指定する場合は、必ず `"..."` と二重引用符で囲んで記述します。
* **エイリアス（別名）**: `Analysis_LxiHiSlip_Open`, `Sqn_LxiHiSlip_Open`

---

## 💡 動作例（正しい構文での記述例）

### スクリプト記述例

※COMMAND ブロック配下の子要素行（PARAM, SEQUENCE）は必ず行頭カンマから開始し、スペースインデントや行末コメント（`;`）は記述しないでください。

```csv
COMMAND, "SAMPLE_LXIHISLIP_OPEN", "IDN_STR"
,PARAM, "IDN_STR", Work, "機器識別情報", ""
; 1. LXI 計測器へ接続
,SEQUENCE, 010, LxiHiSlip_Open, "192.168.0.10", 4880, "hislip0"
; 2. *IDN? を問い合わせて IDN_STR へ格納
,SEQUENCE, 020, LxiHiSlip_Query, "*IDN?", "IDN_STR", 5000
; 3. 接続を安全にクローズして正常終了
,SEQUENCE, 030, LxiHiSlip_Close
,SEQUENCE, 040, Exit, "$IDN_STR$", 0
```

### 実行時の挙動・変数変化

* **実行前の状態**: Term 通信エンジンが初期化され、変数プール（`PARAM`）に `"IDN_STR"` が定義されている状態。
* **実行後の状態**: `LxiHiSlip_Open` により `HiSlipTransportClient.Shared` の接続が確立され、続く `LxiHiSlip_Query` で取得された計測器識別応答が `"IDN_STR"` に格納されます。

---

## 🔄 動作フロー（シーケンス図）

```mermaid
sequenceDiagram
    autonumber
    participant Driver as "TermSequenceDriver (実行エンジン)"
    participant Cmd as "LxiHiSlip_OpenCommand"
    participant Trans as "HiSlipTransportClient.Shared"
    participant Dev as "LXI 計測器 (HiSLIP 1.1 / 2.0)"

    Driver->>Driver: ライセンス検証 (ValidateStep)
    Driver->>Cmd: Func(parent, param)
    activate Cmd
    Cmd->>Cmd: IsExtensionLicensed("LxiHiSlip") ＆ ValidateParamRange(0, 3) ＆ 引数解析
    Cmd->>Trans: ConnectAsync(ip, port, subAddr, timeoutMs) (Sync/Async 2ch 確立 ＆ StartTLS)
    Trans->>Dev: 接続ハンドシェイク送出
    Dev-->>Trans: 接続確立応答
    alt オープン成功 (または IsSimulationMode == true)
        Trans-->>Cmd: true
        Cmd-->>Driver: 正常完了 (IsCommandGo = true)
    else ライセンス未認証・引数超過・接続失敗
        Trans-->>Cmd: false
        Cmd->>Cmd: SetErrorAndHalt(-1 / -100, failMsg)
        Cmd-->>Driver: 異常停止 (IsCommandGo = false)
    end
    deactivate Cmd
```

---

## 🚫 エラーハンドリング・安全設計（仕様制限）

1. **ライセンス検証および引数範囲ガード**:
   - `LxiHiSlip` 拡張ライセンスが未認証の場合はエラーコード `-100` で停止し、引数が 4 個以上指定された場合（`ValidateParamRange(0, 3)` 違反）は `_log.LogError` に記録して `SetErrorAndHalt` で停止します。
2. **既存セッションの安全解放と接続失敗停止**:
   - 再オープン時は既存セッションを先に切断してから新規接続します。ネットワーク到達不可・タイムアウト・ポート拒否時は `SetErrorAndHalt(-1, failMsg)` で安全停止します。

---

## 🧪 ユニットテスト仕様

本コマンドクラス（`LxiHiSlip_OpenCommand.cs`）に実装されている `RunUnitTest()` メソッドが検証するユニットテストの仕様です。

### テスト実行前提条件

* **実行トリガー**: `RunUnitTest()` メソッドの呼び出し
* **有効化条件**: `ErrorLogger.MinimumLevel == LogLevel.Test` であること（それ以外は早期リターン）
* **検証結果出力**: `_log.LogTest` へ出力。`finally` ブロックで `HiSlipTransportClient.Shared.IsSimulationMode` を開始前の状態へ復元します。

### テストケース一覧

| No | テストカテゴリ | テスト条件（入力値） | 期待される結果（出力値） | 検証の目的 / 観点 |
| :---: | :--- | :--- | :--- | :--- |
| **1** | 正常系 | `IsSimulationMode = true` 下で引数 `"192.168.0.20"`, `"4880"`, `"hislip0"` を指定して実行 | `!param.HasError` かつ `param.IsCommandGo == true`、`Host == "192.168.0.20"`、`Port == 4880`、`SubAddress == "hislip0"` となる | 接続先パラメータ反映およびセッションオープンの検証 |
| **2** | 異常系 | 余剰引数（`"192.168.0.20"`, `"4880"`, `"hislip0"`, `"EXTRA"`）を指定して実行 | `ValidateParamRange(0, 3)` により拒否され、`HasError == true` かつ `!IsCommandGo` となる | 引数上限超過時のバリデーション拒否検証 |