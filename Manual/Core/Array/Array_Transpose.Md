# コマンドリファレンス：Array_Transpose

## 概要

`Array_Transpose` は、指定された 2 次元テキストテーブル（`<Target>`）の行（Row）と列（Column）を入れ替え（行列転置：$M \text{ 列} \times N \text{ 行} \to N \text{ 列} \times M \text{ 行}$）、指定された格納先（`<DstTarget>`、省略時は `<Target>` 自身に上書き）へ格納する共通配列操作系コマンドです。  
横方向に蓄積した時系列データを縦方向の CSV レポート形式へ変換する際などに使用します。

> 💡 **補足**  
> 本コマンドは 共通基盤コマンドです。各行の列数が不揃い（ジャグ配列状態）の場合は、最大列数に合わせて不足セルを空文字で自動補完した上で正確に転置します。

---

## 💡 書式・パラメータ仕様

### コマンド構文

```csv
,SEQUENCE, [ステップキー], Array_Transpose, "[対象テーブルキー(Target)]", "[格納先キー(DstTarget)]", "[区切り文字(Delimiter)]"
```

### パラメータ（引数）一覧

| 引数位置 | 引数名 | 説明 | 指定方法 / 有効な入力値 |
| :--- | :--- | :--- | :--- |
| 引数1 | Target | 転置元の 2 次元テーブルを保持するパラメータキー名（必須）。 | 自コマンド内の `PARAM` キー名（例: `"SRC_TABLE"`, `"1"`） |
| 引数2 | DstTarget | 転置後のテーブルを格納するパラメータキー名（任意・省略可）。 | 自コマンド内の `PARAM` キー名。省略時または空文字時は `<Target>` 自身へ上書き格納 |
| 引数3 | Delimiter | 列区切りデリミタ文字列（任意・省略可・既定値: `","`）。 | `","`（カンマ）、`"\t"`（タブ）等 |

---

## 🔄 特殊置換規約・分岐規約の適用

本コマンドの引数における特殊記法および解釈ルールです。
* **動的置換（`$`）**: 引数内で `$PARAMキー$` のように記述すると、実行時に変数プール（PARAM）の値へ動的置換されます。（適用可能な引数: 全引数対応）
* **ジャンプ先指定（`#`）**: 本コマンドはジャンプ制御を行わないため適用されません。（適用可能な引数: 非適用）
* **共通コマンド指定（`@`）**: 本コマンドは外部マクロを呼び出さないため適用されません。（適用可能な引数: 非適用）
* **カンマのエスケープ**: 引数 3 にカンマを指定する場合は、必ず `","` と二重引用符で囲んで記述します。
* **エイリアス（別名）**: `ArrayTranspose`, `Transpose`

---

## 💡 動作例（正しい構文での記述例）

### スクリプト記述例

※COMMAND ブロック配下の子要素行（PARAM, SEQUENCE）は必ず行頭カンマから開始し、スペースインデントや行末コメント（`;`）は記述しないでください。

```csv
COMMAND, "TRANSPOSE_REPORT", "TRANSPOSED_TBL"
,PARAM, "RAW_TBL", Text, "元テーブル(3列×2行)", "CH1,CH2,CH3\r\n5.0,3.3,1.8"
,PARAM, "TRANSPOSED_TBL", Work, "転置後テーブル(2列×3行)", ""
; RAW_TBL の行と列を転置し、TRANSPOSED_TBL へ格納
,SEQUENCE, 010, Array_Transpose, "RAW_TBL", "TRANSPOSED_TBL", ","
```

### 実行時の挙動・変数変化

* **実行前の状態**: `"RAW_TBL"` が `"CH1,CH2,CH3\r\n5.0,3.3,1.8"`（3 列×2 行）の状態。
* **実行後の状態**: ステップ `010` の `Array_Transpose` 実行により、行と列が入れ替わり、`"TRANSPOSED_TBL"` に `"CH1,5.0\r\nCH2,3.3\r\nCH3,1.8"`（2 列×3 行）が格納されます。

---

## 🔄 動作フロー（シーケンス図）

本コマンド実行時における、シーケンス実行エンジン、コマンドクラス（`Array_TransposeCommand`）、および `TableBuffer` 間のインタラクションを示すシーケンス図です。

```mermaid
sequenceDiagram
    autonumber
    participant Engine as "シーケンス実行エンジン"
    participant Cmd as "Array_TransposeCommand"
    participant Target as "TargetResolver / パラメータ辞書"
    participant Buffer as "TableBuffer (2次元テーブル処理)"

    Engine->>Cmd: ExecuteAsync(Target, DstTarget, Delimiter)
    activate Cmd
    Cmd->>Target: GetTargetValue(Target)
    Target-->>Cmd: 元テーブル文字列
    Cmd->>Buffer: TransposeTable(SourceText, Delimiter)
    activate Buffer
    Buffer->>Buffer: 最大列数算出・不揃い行の空セル正規化
    Buffer->>Buffer: Grid[y][x] から Grid[x][y] へ転置再構築
    Buffer-->>Cmd: 転置後のテーブル文字列
    deactivate Buffer
    Cmd->>Target: SetTargetValue(DstTarget ?? Target, TransposedText)
    Cmd-->>Engine: 実行完了 (Success)
    deactivate Cmd
```

---

## 🚫 エラーハンドリング・安全設計（仕様制限）

入力データの異常や実行時エラーに対し、解析・実行エンジンが実行するセーフティ規約です。

1. **行ごとに列数が異なる不揃いテーブル（ジャグ配列）の安全な正規化**:
   - 一部の行で列数が少ない場合でも `IndexOutOfRangeException` を発生させず、テーブル全体の最大列数に合わせて不足セルを空文字 `""` で補完してから転置を行います。
2. **空テーブルに対する転置実行**:
   - 転置元 `<Target>` が空文字列 `""` の場合は、エラーとせず格納先にも空文字列 `""` を設定して正常終了します。

---

## 🧪 ユニットテスト仕様

本コマンドクラス（`Array_TransposeCommand.cs`）に実装されている `RunUnitTest()` メソッドが検証するユニットテストの仕様です。

### テスト実行前提条件

* **実行トリガー**: `RunUnitTest()` メソッドの呼び出し
* **有効化条件**: システム全体の最小ログレベル（`ErrorLogger.MinimumLevel`）が `LogLevel.Test` であること（それ以外の場合は無効化ガードにより早期リターン）
* **検証結果出力**: `_log.LogTest` によるログ出力。検証失敗時は `_log.LogError`、予期せぬ例外発生時は `_log.LogCritical` で記録。

### テストケース一覧

| No | テストカテゴリ | テスト条件（入力値） | 期待される結果（出力値） | 検証の目的 / 観点 |
| :---: | :--- | :--- | :--- | :--- |
| **1** | 正常系 | 3列×2行テーブルの転置（`"A,B,C\r\n1,2,3"`） | `"A,1\r\nB,2\r\nC,3"`（2列×3行）が出力される | 行列転置の基本動作検証 |
| **2** | 正常系 | `<DstTarget>` 省略時のインプレース転置 | `<Target>` 自身が転置後のテーブルで上書きされる | 省略時インプレース更新の検証 |
| **3** | 正常系 | 列数が不揃いな行を含むテーブルの転置 | 不足セルが空文字補完されてクラッシュせず転置される | ジャグ配列に対する安全正規化の検証 |

