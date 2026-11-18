# コマンドリファレンス：Excel_Next

## 概要

`Excel_Next` は、直近の [`Excel_Loop`](./Excel_Loop.Md) によって開始された Excel シート行反復ループの終端を示し、実行回数が最大回数に達するまでループ先頭行へ `現在位置` を折り返す Excel エクステンション共通コマンドです。  
指定回数分の反復が完了した時点でシート単位ループスタックの最上位フレームを解放（Pop）し、直下の次シーケンス行（`targetRow + 2`）へ進行します。

> [!NOTE]
> 戻り先のセル番地（`StartCell`）および現在の反復回数（`CurrentCount` / `MaxCount`）はすべてシート単位ループスタックの最上位フレームから自動解決されるため、`Excel_Next` 自体に引数は不要です。外側ループと内側ループを重ねた多重ネストループにも対応しています。

---

## 💡 書式・パラメータ仕様

### コマンド構文

```csv
,SEQUENCE, [ステップキー], Excel_Next
```

### パラメータ（引数）一覧

（引数なし：戻り先番地および残り回数はシート単位ループスタックから自動解決されます。）

---

## 🔄 特殊置換規約・分岐規約の適用

本コマンドにおける制御ルールです。
* **動的置換（`$`）**: 引数を持たないため適用されません。（適用可能な引数: 非適用）
* **ジャンプ先指定（`#`）**: 折り返し先はシート単位ループスタックから自動解決されるため適用されません。（適用可能な引数: 非適用）
* **共通コマンド指定（`@`）**: 本コマンドは外部マクロを呼び出さないため適用されません。（適用可能な引数: 非適用）
* **カンマのエスケープ**: 引数を持たないため適用されません（一般規則として引数内カンマは `","` で囲みます）。
* **エイリアス（別名）**: `ExcelNext`
* **多重ネストループ対応**: 内側の [`Excel_Loop`](./Excel_Loop.Md) 〜 `Excel_Next` は常にスタック最上位（LIFO）のフレームを参照・更新するため、二重・三重ループをシート上でそのまま構成できます。
* **単体実行ボタン押下時の非干渉**: 各行先頭の単体実行（`実行`）ボタンから実行された場合は、制御ブロックの `現在位置` やシート単位ループスタックは変更されません。

---

## 💡 動作例（正しい構文での記述例）

### スクリプト記述例

※ `COMMAND` ブロック配下の子要素行（`PARAM`, `SEQUENCE`）は必ず行頭カンマから開始し、スペースインデントや行末コメント（`;`）は記述しないでください。

```csv
COMMAND, "EXCEL_LOOP_END", ""
,SEQUENCE, 010, Excel_Next
```

### 実行時の挙動・シート状態変化

* **実行前の状態**: シート単位ループスタックにフレーム（`StartCell = "A6"`, `MaxCount = 5`, `CurrentCount = 1` または `5`）が保持されている状態。
* **実行後の状態**: 反復継続中（`CurrentCount < MaxCount`）は `CurrentCount` が加算されて `"A6"` へ `現在位置` が折り返され、反復満了時（`CurrentCount == MaxCount`）は最上位フレームが Pop 解放されて直下の次ステップ行へ進みます。

---

## 🔄 動作フロー（シーケンス図）

本コマンド実行時における、解析エンジン・`ExcelNextCommand`・Excel ブリッジ間のインタラクションを示すシーケンス図です。

```mermaid
sequenceDiagram
    autonumber
    participant Bridge as "CommunicationExcelBridgeBase"
    participant Engine as "解析エンジン (ParameterizedSequenceDriver)"
    participant Cmd as "ExcelNextCommand"
    participant Stack as "シート単位ループスタック"

    Bridge->>Engine: ExecuteStepAsync(rowData)
    Engine->>Cmd: Execute()
    activate Cmd
    Cmd-->>Engine: AnalysisResult.ExcelLoopNext()
    deactivate Cmd
    Engine-->>Bridge: ExcelStepExecutionResult (Flow=LoopNext)
    alt ループスタックが空 (対応する Excel_Loop 未実行)
        Bridge->>Bridge: エラーログ出力 ＆ 当該行番地を保持して安全停止 (Halt)
    else CurrentCount < MaxCount (反復継続)
        Bridge->>Stack: CurrentCount += 1
        Bridge->>Bridge: 現在位置を frame.StartCell へ折り返し更新
    else CurrentCount >= MaxCount (反復満了)
        Bridge->>Stack: Pop() 最上位フレーム解放
        Bridge->>Bridge: 現在位置を次行 (targetRow + 2 または空白) へ更新
    end
```

---

## 🚫 エラーハンドリング・安全設計（仕様制限）

実行時の構造不整合に対し、ブリッジ層が実行するセーフティ規約です。

1. **対応する `Excel_Loop` が存在しない状態での実行**:
   - `Step` 実行または `Start` 一括実行において、シート単位ループスタックが空の状態で `Excel_Next` に到達した場合、ループ構造不整合エラーを `_log.LogError` に記録し、当該行番地を `現在位置` に保持したまま安全停止（`Halt`）します。
2. **累積ステップ上限による無限ループ防御**:
   - 多重ループの回数設定過大等により総実行ステップ数が `MaxExecutionSteps`（既定値 `10000`）を超えた場合、自動的にループを安全停止します。

---

## 🧪 ユニットテスト仕様

本コマンドクラス（`ExcelNextCommand.cs`）に実装されている `RunUnitTest()` メソッドが検証するユニットテストの仕様です。

### テスト実行前提条件

* **実行トリガー**: `RunUnitTest()` メソッドの呼び出し
* **有効化条件**: システム全体の最小ログレベル（`ErrorLogger.MinimumLevel`）が `LogLevel.Test` であること
* **検証結果出力**: `_log.LogTest` によるログ出力。検証失敗時は `_log.LogError`、予期せぬ例外発生時は `_log.LogCritical` で記録

### テストケース一覧

| No | テストカテゴリ | テスト条件（入力値） | 期待される結果（出力値） | 検証の目的 / 観点 |
| :---: | :--- | :--- | :--- | :--- |
| **1** | 正常系 | 引数なしで実行 | `Flow = LoopNext` を持つ成功結果が返る | ループ折り返し識別子返却の基本検証 |
| **2** | 異常系 | 対応する `Excel_Loop` なし（スタック空）または余剰引数指定時 | ブリッジ層で `Flow = Halt` 安全停止または検証拒否となる | ループ構造不整合に対する安全防護検証 |
