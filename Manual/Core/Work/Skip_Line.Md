# コマンドリファレンス：Skip_Line

## 概要

`Skip_Line` は、Term 非同期受信リングバッファ（または指定された `<SourceBuffer>`）の先頭から指定行数分（`<SkipCount>`、既定 1 行）を読み飛ばして破棄し、読み取りポインタを進める Term / GPIB 共通データ・ワーク処理系コマンド（`Common.Commands.Work.dll`）です。  
シリアル・Telnet 通信におけるコマンドエコーバック行や不要なバナーヘッダ行を除去してから後続の [`Set_Data`](./Set_Data.Md) / [`Get_Line`](./Get_Line.Md) で本データを抽出する際に使用します。

> 💡 **補足**  
> 本コマンドは 共通基盤コマンドです。

---

## 💡 書式・パラメータ仕様

### コマンド構文

```csv
,SEQUENCE, [ステップキー], Skip_Line, "[読み飛ばし行数(SkipCount)]", "[対象バッファキー(SourceBuffer)]"
```

### パラメータ（引数）一覧

| 引数位置 | 引数名 | 説明 | 指定方法 / 有効な入力値 |
| :--- | :--- | :--- | :--- |
| 引数1 | SkipCount | 先頭から読み飛ばして破棄する行数（任意・省略可・既定値: `"1"`）。 | 1 以上の整数（例: `"1"`, `"2"`）または `$PARAM$` 置換 |
| 引数2 | SourceBuffer | 操作対象のテキストを保持するパラメータキー名（任意・省略可）。 | 省略時は Term 非同期受信リングバッファを対象とする |

---

## 🔄 特殊置換規約・分岐規約の適用

本コマンドの引数における特殊記法および解釈ルールです。
* **動的置換（`$`）**: 引数内で `$PARAMキー$` のように記述すると、実行時に変数プール（PARAM）の値へ動的置換されます。（適用可能な引数: 全引数対応）
* **ジャンプ先指定（`#`）**: 本コマンドはジャンプ制御を行わないため適用されません。（適用可能な引数: 非適用）
* **共通コマンド指定（`@`）**: 本コマンドは外部マクロを呼び出さないため適用されません。（適用可能な引数: 非適用）
* **カンマのエスケープ**: 引数文字列自体にカンマを含める場合は、必ず `","` と二重引用符で囲んで記述します。
* **エイリアス（別名）**: `SkipLine`, `Analysis_Term_SkipLine`, `GpibSkipLine`

---

## 💡 動作例（正しい構文での記述例）

### スクリプト記述例

※COMMAND ブロック配下の子要素行（PARAM, SEQUENCE）は必ず行頭カンマから開始し、スペースインデントや行末コメント（`;`）は記述しないでください。

```csv
COMMAND, "READ_WITHOUT_ECHO", "RESULT_LINE"
,PARAM, "RESULT_LINE", Work, "エコー除去後データ", ""
,SEQUENCE, 010, Com_Send, "READ_ADC"
,SEQUENCE, 020, Com_WaitPrompt, ">"
; 先頭 1 行（コマンドエコーバック "READ_ADC"）を読み飛ばして破棄
,SEQUENCE, 030, Skip_Line, 1
; 残りの測定値行を RESULT_LINE へ取り込み
,SEQUENCE, 040, Set_Data, "RESULT_LINE"
```

### 実行時の挙動・変数変化

* **実行前の状態**: 受信リングバッファに `"READ_ADC\r\n4095\r\n"`（1 行目がエコーバック、2 行目が ADC 値）が蓄積されている状態。
* **実行後の状態**: ステップ `030` の `Skip_Line, 1` により先頭の `"READ_ADC\r\n"` が破棄され、ステップ `040` の `Set_Data` で `"RESULT_LINE"` に `"4095"` のみが格納されます。

---

## 🔄 動作フロー（シーケンス図）

本コマンド実行時における、シーケンス実行エンジン、コマンドクラス（`Skip_LineCommand`）、および受信リングバッファ間のインタラクションを示すシーケンス図です。

```mermaid
sequenceDiagram
    autonumber
    participant Engine as "シーケンス実行エンジン"
    participant Cmd as "Skip_LineCommand (Common.Commands.Work)"
    participant Ring as "TermRingBufferManager / SourceBuffer"

    Engine->>Cmd: ExecuteAsync(SkipCount, SourceBuffer)
    activate Cmd
    Cmd->>Cmd: SkipCount >= 1 の整数検証
    alt SkipCount >= 1
        Cmd->>Ring: DiscardLines(SkipCount)
        Ring-->>Cmd: 先頭 SkipCount 行破棄完了 (行数不足時は全クリア)
        Cmd-->>Engine: 実行完了 (Success)
    else SkipCount < 0 または 非数値
        Cmd->>Cmd: エラーログ出力 (_log.LogError)
        Cmd-->>Engine: 実行失敗・安全停止 (Halt)
    end
    deactivate Cmd
```

---

## 🚫 エラーハンドリング・安全設計（仕様制限）

入力データの異常や実行時エラーに対し、解析・実行エンジンが実行するセーフティ規約です。

1. **バッファ内の残行数より多い `<SkipCount>` が指定された場合**:
   - 例外クラッシュを発生させず、バッファ内の全行を安全に破棄（空状態にクリア）して正常終了します。
2. **負数または非数値の指定**:
   - `<SkipCount>` に負数や非数値文字列が指定された場合は、エラーログ（`_log.LogError`）を記録して直ちに安全停止（Halt）します。

---

## 🧪 ユニットテスト仕様

本コマンドクラス（`Skip_LineCommand.cs`）に実装されている `RunUnitTest()` メソッドが検証するユニットテストの仕様です。

### テスト実行前提条件

* **実行トリガー**: `RunUnitTest()` メソッドの呼び出し
* **有効化条件**: システム全体の最小ログレベル（`ErrorLogger.MinimumLevel`）が `LogLevel.Test` であること（それ以外の場合は無効化ガードにより早期リターン）
* **検証結果出力**: `_log.LogTest` によるログ出力。検証失敗時は `_log.LogError`、予期せぬ例外発生時は `_log.LogCritical` で記録。

### テストケース一覧

| No | テストカテゴリ | テスト条件（入力値） | 期待される結果（出力値） | 検証の目的 / 観点 |
| :---: | :--- | :--- | :--- | :--- |
| **1** | 正常系 | 3行データに対して `Skip_Line, "1"` を実行 | 先頭1行が削除され、残り2行が保持される | 先頭行スキップの基本動作検証 |
| **2** | 正常系 | 2行データに対して `Skip_Line, "10"`（超過行数）を実行 | 例外を投げずバッファが空文字列 `""` になる | 超過行数指定時の安全クリア検証 |
| **3** | 異常系 | 負の行数（`"-1"`）を指定 | エラーログが出力され `Failure` で安全停止する | 不正引数ガードの検証 |

