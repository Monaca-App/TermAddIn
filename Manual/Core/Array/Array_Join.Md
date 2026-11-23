# コマンドリファレンス：Array_Join

## 概要

`Array_Join` は、対象の 2 次元テキストテーブル（`<TargetKey>`）の各行末尾に対して、追加テーブルデータ（`<Data>`）の各行を指定デリミタ（`<Delimiter>`）で水平方向に列結合（マージ）する共通配列操作系コマンドです。  
ベース側の行数が追加データ側の行数より少ない場合は、自動的にベース側の行を空セルで拡張（Auto-Resize）した上で安全に水平結合します。

> 💡 **補足**  
> 本コマンドは 共通基盤コマンドです。複数回測定した列データを既存のテーブルへ横方向に順次連結していく用途に最適です。

---

## 💡 書式・パラメータ仕様

### コマンド構文

```csv
,SEQUENCE, [ステップキー], Array_Join, "[結合先テーブルキー(TargetKey)]", "[追加データまたはキー(Data)]", "[区切り文字(Delimiter)]"
```

### パラメータ（引数）一覧

| 引数位置 | 引数名 | 説明 | 指定方法 / 有効な入力値 |
| :--- | :--- | :--- | :--- |
| 引数1 | TargetKey | 水平結合先となるベース 2 次元テーブルのパラメータキー名（必須）。 | 自コマンド内の `PARAM` キー名（例: `"MAIN_TABLE"`, `"1"`） |
| 引数2 | Data | 各行の末尾へ追加結合する 2 次元テキストデータまたはパラメータキー（必須）。 | パラメータキー名、`$PARAM$` 置換、または文字列リテラル |
| 引数3 | Delimiter | 列間を結合するデリミタ文字列（任意・省略可・既定値: `","`）。 | `","`（カンマ）、`"\t"`（タブ）等 |

---

## 🔄 特殊置換規約・分岐規約の適用

本コマンドの引数における特殊記法および解釈ルールです。
* **動的置換（`$`）**: 引数内で `$PARAMキー$` のように記述すると、実行時に変数プール（PARAM）の値へ動的置換されます。（適用可能な引数: 全引数対応）
* **ジャンプ先指定（`#`）**: 本コマンドはジャンプ制御を行わないため適用されません。（適用可能な引数: 非適用）
* **共通コマンド指定（`@`）**: 本コマンドは外部マクロを呼び出さないため適用されません。（適用可能な引数: 非適用）
* **カンマのエスケープ**: 引数 3 にカンマを指定する場合は、必ず `","` と二重引用符で囲んで記述します。
* **エイリアス（別名）**: `Gpib_Array_Join`, `Term_Array_Join`, `Analysis_Term_Array_Join`

---

## 💡 動作例（正しい構文での記述例）

### スクリプト記述例

※COMMAND ブロック配下の子要素行（PARAM, SEQUENCE）は必ず行頭カンマから開始し、スペースインデントや行末コメント（`;`）は記述しないでください。

```csv
COMMAND, "MERGE_COLUMNS", "BASE_TBL"
,PARAM, "BASE_TBL", Work, "ベース表", ""
,PARAM, "NEW_COL", Work, "追加測定列", ""
; 1. BASE_TBL (1列×2行: "CH1" / "CH2") と NEW_COL (1列×2行: "5.01" / "4.99") を構築
,SEQUENCE, 010, Array_Set, "BASE_TBL", 0, 0, ",", "CH1"
,SEQUENCE, 020, Array_Set, "BASE_TBL", 0, 1, ",", "CH2"
,SEQUENCE, 030, Array_Set, "NEW_COL", 0, 0, ",", "5.01"
,SEQUENCE, 040, Array_Set, "NEW_COL", 0, 1, ",", "4.99"
; 2. BASE_TBL の各行末に NEW_COL の各行をカンマ区切りで水平結合
,SEQUENCE, 050, Array_Join, "BASE_TBL", "$NEW_COL$", ","
```

### 実行時の挙動・変数変化

* **実行前の状態**: ステップ `010`〜`040` により `"BASE_TBL"` が `"CH1\r\nCH2"`（1 列×2 行）、`"NEW_COL"` が `"5.01\r\n4.99"`（1 列×2 行）に構築された状態。
* **実行後の状態**: ステップ `050` の `Array_Join` 実行により、各行が水平連結され、`"BASE_TBL"` が `"CH1,5.01\r\nCH2,4.99"`（2 列×2 行）に更新されます。

---

## 🔄 動作フロー（シーケンス図）

本コマンド実行時における、シーケンス実行エンジン、コマンドクラス（`Array_JoinCommand`）、および `TableBuffer` 間のインタラクションを示すシーケンス図です。

```mermaid
sequenceDiagram
    autonumber
    participant Engine as "シーケンス実行エンジン"
    participant Cmd as "Array_JoinCommand"
    participant Target as "TargetResolver / ArrayStorage"
    participant Buffer as "TableBuffer (2次元テーブル処理)"

    Engine->>Cmd: ExecuteAsync(context, args, cancellationToken)
    activate Cmd
    Cmd->>Target: TryResolve(TargetKey, context, out targetBuffer)
    alt 対象キーが解決済み かつ Data / Delimiter の展開成功
        Target-->>Cmd: targetBuffer.Value ＆ appendText 解決
        Cmd->>Buffer: JoinColumnsHorizontally(targetBuffer.Value, appendText, delimiter)
        activate Buffer
        Buffer->>Buffer: 行数比較・不足行の空セル自動補填 (Auto-Resize)
        Buffer-->>Cmd: 水平結合後のテーブル文字列 mergedText
        deactivate Buffer
        Cmd->>Target: targetBuffer.Value = mergedText ＆ ArrayStorage.SetTable(Key, TableBuffer.Parse(...))
        Cmd-->>Engine: AnalysisResult.Success(...)
    else 未定義キー指定 または マクロ展開エラー
        Cmd->>Cmd: エラーログ出力 (_log.LogError)
        Cmd-->>Engine: AnalysisResult.Halt(errMsg, errorCode)
    end
    deactivate Cmd
```

---

## 🚫 エラーハンドリング・安全設計（仕様制限）

入力データの異常や実行時エラーに対し、解析・実行エンジンが実行するセーフティ規約です。

1. **ベース表と追加データの行数不一致時の自動行拡張**:
   - 追加データ `<Data>` の行数がベース表 `<TargetKey>` の行数より多い場合でもデータを切り捨てず、ベース表の不足行を既存列数分の空セルで自動パディングした上で全行を安全に結合します。
2. **未定義のターゲットパラメータキー指定**:
   - `<TargetKey>` が `PARAM` で定義されていない場合はエラーログ（`_log.LogError`）を記録して直ちに安全停止（Halt）します。

---

## 🧪 ユニットテスト仕様

本コマンドクラス（`Array_JoinCommand.cs`）に実装されている `RunUnitTest()` メソッドが検証するユニットテストの仕様です。

### テスト実行前提条件

* **実行トリガー**: `RunUnitTest()` メソッドの呼び出し
* **有効化条件**: システム全体の最小ログレベル（`ErrorLogger.MinimumLevel`）が `LogLevel.Test` であること（それ以外の場合は無効化ガードにより早期リターン）
* **検証結果出力**: `_log.LogTest` によるログ出力。検証失敗時は `_log.LogError`、予期せぬ例外発生時は `_log.LogCritical` で記録。

### テストケース一覧

| No | テストカテゴリ | テスト条件（入力値） | 期待される結果（出力値） | 検証の目的 / 観点 |
| :---: | :--- | :--- | :--- | :--- |
| **1** | 正常系 | 同行数テーブルの水平結合（`"BASE_TBL" = "CH1\r\nCH2"`, `"NEW_COL" = "5.01\r\n4.99"` に対し `("BASE_TBL", "$NEW_COL$", ",")` を実行） | `"BASE_TBL"` が `"CH1,5.01\r\nCH2,4.99"` に更新され `Success` が返る | 基本的な行末列結合および `$PARAM$` 展開の検証 |
| **2** | 正常系 | 追加データ側が行数過多の場合の結合（`"SHORT_TBL" = "ROW0"` に対し `("SHORT_TBL", "10\r\n20", ",")` を実行） | ベース 2 行目が空セル補完され `"SHORT_TBL"` が `"ROW0,10\r\n,20"` に更新される | 自動行拡張マージ（Auto-Resize）の検証 |
| **3** | 異常系 | 未定義の `<TargetKey>` を指定 `("UNDEFINED_TBL", "1\r\n2", ",")` | エラーログが出力され `Halt` で安全停止する | 未定義キーアクセス防御の検証 |

