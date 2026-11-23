# コマンドリファレンス：Excel_Msg_Box

## 概要

`Excel_Msg_Box` は、ケーブル差し替え指示・治具設定確認・目視外観判定などのオペレーター対話ポップアップダイアログを表示し、選択されたボタン結果に応じて **継続進行（`Next`）** 、 **指定パラメータ番号の Excel シート行への分岐ジャンプ（`Jump`）** 、または **作業中止による安全停止（`Halt`）** を制御する Excel エクステンション共通コマンドです。  
既存の [`Msg_Box`](../Sequence/Msg_Box.Md) と同様、ヘッドレス自動テスト実行時（`ErrorLogger.MinimumLevel == LogLevel.Test`）にはモーダル待機でブロックしない自動承認ガードが作動します。

> [!NOTE]
> `Buttons` に `"OKCancel"` または `"YesNo"` を指定し、第 5 引数 `CancelOrNoJumpParamNo` に公開パラメータ番号を指定すると、オペレーターが `Cancel` または `No` を選択した際に再試行行や NG 処理行へ直接分岐できます。

---

## 💡 書式・パラメータ仕様

### コマンド構文

```csv
,SEQUENCE, [ステップキー], Excel_Msg_Box, "<Message>", "[Title]", "[Buttons]", "[ResultVar]", "[CancelOrNoJumpParamNo]"
```

### パラメータ（引数）一覧

| 引数位置 | 引数名 | 説明 | 指定方法 / 有効な入力値 |
| :--- | :--- | :--- | :--- |
| 引数1 | `Message` | ダイアログ本文に表示するメッセージ文字列（必須）。 | 任意の文字列または `$PARAMキー$` 置換 |
| 引数2 | `Title` | ダイアログウィンドウのタイトル文字列（任意・省略可）。 | 任意の文字列（省略時の既定値: `"確認"`）、`$PARAMキー$` 置換可 |
| 引数3 | `Buttons` | 表示するボタン構成（任意・省略可）。 | `"OK"`（既定値）、`"OKCancel"`、`"YesNo"`（未定義文字列指定時は警告ログ出力の上 `"OK"` へフォールバック） |
| 引数4 | `ResultVar` | 押下されたボタン名（`"OK"`, `"Cancel"`, `"Yes"`, `"No"`）を格納する `PARAM` キー名（任意・省略可）。 | 定義済みの `PARAM` キー名（なお `ResultVar` 省略時も含め、選択結果は常にシステム変数 `$LAST_MSGBOX_RESULT$` にも保存される） |
| 引数5 | `CancelOrNoJumpParamNo` | `Cancel` または `No` が選択された際の遷移先シート行を保持する公開パラメータ番号（任意・省略可）。 | `1` 以上の整数（`"1"`, `"2"`, ...）または省略 |

---

## 🔄 特殊置換規約・分岐規約の適用

ボタン応答別の拡張戻り値およびシート制御ルールです。
* **動的置換（`$`）**: 全引数において `$PARAMキー$` による実行時変数置換を利用できます。（適用可能な引数: 全引数対応）
* **ジャンプ先指定（`#`）**: ミニシーケンス内ラベル `#ラベル名#` ではなく、引数 5 `CancelOrNoJumpParamNo` の公開パラメータ番号を介して分岐先シート行を指定します。（適用可能な引数: 非適用）
* **共通コマンド指定（`@`）**: 本コマンドは外部マクロを呼び出さないため適用されません。（適用可能な引数: 非適用）
* **カンマのエスケープ**: メッセージやタイトルにカンマを含める場合は、必ず `","` と二重引用符で囲んで記述します。
* **エイリアス（別名）**: `ExcelMsgBox`
* **`OK` または `Yes` が選択された場合**: `ResultVar`（指定時）へ `"OK"` または `"Yes"` を格納し、`ExcelSequenceFlow.Next` で後続ステップおよび直下の次シート行へ進行します。
* **`Cancel` または `No` が選択された場合**:
  1. **`CancelOrNoJumpParamNo` 指定あり**: `ResultVar`（指定時）を更新したうえで `AnalysisResult.ExcelJump(CancelOrNoJumpParamNo)`（`Flow = Jump`）を返却し、該当パラメータ列が指すシート行へ分岐します。
  2. **`CancelOrNoJumpParamNo` 省略・`ResultVar` 指定あり**: `ResultVar` に `"Cancel"` または `"No"` を格納し、`Flow = Next` で後続ステップの条件判定へ委ねます。
  3. **`CancelOrNoJumpParamNo`・`ResultVar` ともに省略**: オペレーターによる検査中止要求として `Flow = Halt` を返却し、当該行番地を `現在位置` に保持して安全停止します。

---

## 💡 動作例（正しい構文での記述例）

### スクリプト記述例

※ `COMMAND` ブロック配下の子要素行（`PARAM`, `SEQUENCE`）は必ず行頭カンマから開始し、スペースインデントや行末コメント（`;`）は記述しないでください。

```csv
COMMAND, "VISUAL_CHECK_LED", "BTN_RES"
,PARAM, "NG_ROW", Text, "目視NG時ジャンプ先", "LED_NG_PROC"
,PARAM, "BTN_RES", Work, "目視判定結果", ""
,SEQUENCE, 010, Excel_Msg_Box, "緑色LEDが点灯していますか？", "外観目視検査", "YesNo", "BTN_RES", 1
```

### 実行時の挙動・シート状態変化

* **実行前の状態**: 第 1 公開パラメータに目視 NG 時の遷移先 `"LED_NG_PROC"` が設定され、`BTN_RES` が空文字列 `""` の状態。
* **実行後の状態**: オペレーターが `Yes` を選択すると `BTN_RES = "Yes"` となって直下の次シート行へ進行し、`No` を選択すると `BTN_RES = "No"` となって `"LED_NG_PROC"` のシート行へ `現在位置` が分岐ジャンプします。

---

## 🔄 動作フロー（シーケンス図）

本コマンド実行時における、オペレーター・`ExcelMsgBoxCommand`・Excel ブリッジ間のインタラクションを示すシーケンス図です。

```mermaid
sequenceDiagram
    autonumber
    actor Operator as "オペレーター"
    participant Bridge as "CommunicationExcelBridgeBase"
    participant Engine as "解析エンジン (ParameterizedSequenceDriver)"
    participant Cmd as "ExcelMsgBoxCommand"

    Bridge->>Engine: ExecuteStepAsync(rowData, cancellationToken)
    Engine->>Cmd: ExecuteAsync(context, args, cancellationToken)
    activate Cmd
    Cmd->>Operator: SelectDialogButton / MessageBox.Show (Test・ヘッドレス時は自動応答)
    Operator-->>Cmd: selectedButton (OK / Yes / Cancel / No)
    Cmd->>Cmd: context.SetVariable("$LAST_MSGBOX_RESULT$", selectedButton)
    opt ResultVar 指定あり (TargetResolver.TryResolve 解決済み)
        Cmd->>Cmd: resultBuffer.Value = selectedButton (PARAM バッファへ格納)
    end
    alt OK または Yes 選択
        Cmd-->>Engine: AnalysisResult.Success(selectedButton) (Flow=Next)
        Engine-->>Bridge: 次ステップ行へ進行 (Flow=Next)
    else Cancel または No 選択 (CancelOrNoJumpParamNo 指定あり)
        Cmd-->>Engine: AnalysisResult.ExcelJump(cancelJumpParamNo, jumpMsg)
        Engine-->>Bridge: 指定パラメータ番号のシート行へ分岐 (Flow=Jump)
    else Cancel または No 選択 (CancelOrNoJumpParamNo 省略・ResultVar 指定あり)
        Cmd-->>Engine: AnalysisResult.Success(selectedButton) (Flow=Next)
        Engine-->>Bridge: 選択結果を保持して次ステップへ進行 (Flow=Next)
    else Cancel または No 選択 (CancelOrNoJumpParamNo・ResultVar 両方省略)
        Cmd-->>Engine: AnalysisResult.ExcelHalt(cancelMsg, -15)
        Engine-->>Bridge: 当該行番地を現在位置に保持して安全停止 (Flow=Halt)
    end
    deactivate Cmd
```

---

## 🚫 エラーハンドリング・安全設計（仕様制限）

入力データの異常に対し、解析エンジンが実行するセーフティ規約です。

1. **本文メッセージ未指定・未定義 `ResultVar`・不正なパラメータ番号の防御、および未対応 `Buttons` の安全フォールバック**:
   - 引数 1 `Message` が省略された場合、未定義の `ResultVar` が指定された場合、または `CancelOrNoJumpParamNo` に `0` 以下の値や非数値が指定された場合は `_log.LogError` に記録し `ExcelSequenceFlow.Halt` で安全停止します。一方、`Buttons` に未対応の文字列が渡された場合は停止せず、警告ログ（`_log.LogWarning`）を出力した上で既定の `"OK"` ボタンへ安全にフォールバックして処理を継続します。
2. **`$LAST_MSGBOX_RESULT$` 保存およびヘッドレス単体テスト時の非ブロッキング保証**:
   - 選択されたボタン文字列は常にコンテキスト変数 `$LAST_MSGBOX_RESULT$` に保存されます。また、`ErrorLogger.MinimumLevel == LogLevel.Test` 環境下ではモーダルダイアログを表示せず既定応答（または `MockDialogResult`）で即時通過するため、CI/CD コンソールテストが停止しません。

---

## 🧪 ユニットテスト仕様

本コマンドクラス（`ExcelMsgBoxCommand.cs`）に実装されている `RunUnitTest()` メソッドが検証するユニットテストの仕様です。

### テスト実行前提条件

* **実行トリガー**: `RunUnitTest()` メソッドの呼び出し
* **有効化条件**: システム全体の最小ログレベル（`ErrorLogger.MinimumLevel`）が `LogLevel.Test` であること
* **検証結果出力**: `_log.LogTest` によるログ出力。検証失敗時は `_log.LogError`、予期せぬ例外発生時は `_log.LogCritical` で記録

### テストケース一覧

| No | テストカテゴリ | テスト条件（入力値） | 期待される結果（出力値） | 検証の目的 / 観点 |
| :---: | :--- | :--- | :--- | :--- |
| **1** | 正常系 | `Test` モードでの既定自動承認（`"YesNo"`, `ResultVar = "ANS"`） | モーダル待機せず `ANS == "Yes"` かつ `Flow = Next` が返る | ヘッドレス自動承認および変数格納の検証 |
| **2** | 正常系 | `No` 応答かつ `CancelOrNoJumpParamNo = 2` 指定 | `Flow = Jump`（`TargetParamNumber = 2`）かつ `ANS == "No"` が返る | 否定応答時のパラメータ番号分岐ジャンプ検証 |
| **3** | 正常系 | `Cancel` 応答かつ `CancelOrNoJumpParamNo` 省略・`ResultVar = "ANS"` 指定 | `ANS == "Cancel"` が格納され `Flow = Next` が返る | `ResultVar` のみ指定時の後続ステップ継続検証 |
| **4** | 異常系 | `Cancel` 応答かつ `CancelOrNoJumpParamNo`・`ResultVar` ともに省略 | 警告ログが出力され `Flow = Halt`（エラーコード `-15`）で安全停止する | 分岐・結果変数省略時のオペレーター中止停止検証 |
