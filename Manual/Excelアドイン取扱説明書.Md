# Excelアドイン取扱説明書 (User Manual)

本ドキュメントは、Excel 上からシリアル機器（RS-232C / UART）、ネットワーク機器（Telnet）、産業用 Modbus 機器、ならびに三菱電機 MELSEC・オムロン SYSMAC・キーエンス KV シーケンサのテストシーケンスを構築・実行するための **Excel アドイン（TermAddIn）** の総合取扱説明書です。リボン機能、4 列制御ブロックと 3 種実行ボタン（単体・`Step`・`Start`）、Excel エクステンション順序制御、設定ファイル（`Setting.xml`）、除外キャッシュ機構、32bit / 64bit 動作環境、および関連マクロコマンドリファレンスを本書 1 冊に統合して解説します。

---

## 目次

- [📖 1. 概要](#-1-概要)
- [🛠️ 2. リボンメニューの説明 (個別機能)](#️-2-リボンメニューの説明-個別機能)
  - [2.1 【ツール】グループ (全製品共通リボン機能)](#21-ツールグループ-全製品共通リボン機能)
  - [2.2 【Term制御】グループ (Term専用リボン機能)](#22-term制御グループ-term専用リボン機能)
- [🗂️ 3. シート上の実行ボタンと順序制御仕様](#️-3-シート上の実行ボタンと順序制御仕様)
- [💾 4. 設定の自動保存・復元機能](#-4-設定の自動保存復元機能)
- [⚙️ 5. 設定ファイル (Setting.xml) について](#️-5-設定ファイル-settingxml-について)
- [⚡ 6. 高速ファイルオープンと除外キャッシュ機能（オーバーヘッド防止）](#-6-高速ファイルオープンと除外キャッシュ機能オーバーヘッド防止)
- [💻 7. 動作環境と Excel 32bit / 64bit 運用上の留意事項](#-7-動作環境と-excel-32bit--64bit-運用上の留意事項)
- [📚 8. マクロコマンド・関連マニュアル一覧](#-8-マクロコマンド関連マニュアル一覧)

---

## 📖 1. 概要
本 Excel アドイン（TermAddIn）は、Excel のワークシート上に通信コマンドのシーケンス（実行手順）を 2 行 1 ステップ構成で定義し、単体実行・1 ステップ実行（`Step`）・連続一括実行（`Start`）を非同期に行えるアドインです。コマンド名や待機時間（ウェイト）、パラメータ、条件分岐・ループ遷移先、実行結果を Excel 画面上で視覚的に管理・編集できます。

### 💡 本アドイン最大の特長：VBAマクロ不要・標準 `.xlsx` 形式での完全動作
*   **マクロ無効形式（`.xlsx`）のまま動作** :
    一般的なマクロ機能とは異なり、ブック内部に VBA コードを一切保持しません。Excel シート上のコマンド実行ボタンはアドイン側から動的に制御されるため、 **標準的な `.xlsx` 形式のままボタンの配置・保存・自動実行が可能** です。
*   **セキュリティ警告・ブロックの解消** :
    マクロ有効ブック（`.xlsm`）を使用しないため、セキュリティポリシーによるマクロ実行ブロックや警告に煩わされることなく、安全にシーケンスシートを作成・運用・共有できます。
*   **VBE（マクロ）の保守不要** :
    Excel のバージョンアップによる参照設定トラブルやマクロコード破損の心配がなく、長期的なシステムの安定性を保ちます。

### 📌 安心・快適なシート運用のポイント
1. **標準 `.xlsx` ブックでの完結運用** :
   * マクロ有効形式（`.xlsm`）や VBA 開発環境へのアクセス許可を必要とせず、通常の `.xlsx` ワークブックとしてそのまま作成・配布・共有できます。
2. **行挿入・行削除への自動追従** :
   * シート上に配置された各実行ボタンは、クリックされた時点のセル行位置をリアルタイムに判別して動作するため、途中に新しい行を挿入したり不要な行を削除したりしても設定ズレを起こさず自動的に追従します。
3. **クリーンな終了処理** :
   * シートの操作や連続実行を終了して Excel を閉じた際、バックグラウンドに不要な Excel プロセスが残らないよう安全にリソースを解放します。
4. **オフラインマニュアル・サンプルの標準インストール連携** :
   * インストーラーにより HTML 形式の取扱説明書一式は `C:\Program Files (x86)\Monaca_App_Store\Monaca html\Apr\` に、各通信プロトコルのサンプル資材は `C:\Program Files (x86)\Monaca_App_Store\Monaca Sample\` に配置され、デスクトップの `Monaca\Manual\` および `Monaca\Sample` ショートカットからすぐに参照できます。

### 🚀 はじめての Excel 自動実行シート作成（4 ステップ具体例）
Excel アドインを初めて使う方は、以下の 4 ステップで「通信接続 → データ読出 → 切断」の自動実行シートを 3 分で作成できます。
1. **ステップ 1（制御ブロックを置く）**: 空のシートの `A2` セルを選び、リボン **`[アドイン]` ➔ `[ツール]` ➔ `[Startボタン配置]`** をクリックします（`[ Start ]`・`[ Step ]`・`開始位置`・`現在位置` の 4 列ブロックが自動配置されます）。
2. **ステップ 2（コマンドを 3 行並べる）**: `A5` セルを選び、リボン **`[Term制御]` ➔ `[シーケンス配置]`** をクリック → タスク `P1` を選択 → コマンド編集画面で `Com_Open`（または `Modbus_Open` / `Mc_Open` 等）を選んで **`終了`** で閉じます。同様に `A7` に読出コマンド、`A9` にクローズコマンドを並べます。
3. **ステップ 3（1 ステップずつ試す）**: 上段の **`[ Step ]`** ボタンを押すと 1 行だけ実行され、`現在位置` が次の行（`A7`）へ進みます。読出コマンドを実行すると、`Result` セルに取得値が表示されます。
4. **ステップ 4（一括自動実行する）**: **`[ Start ]`** ボタンを押すと `開始位置`（`A5`）から最終行まで自動で連続実行されます。このまま通常の `.xlsx` 形式で保存してチームへ配布できます。

---

## 🛠️ 2. リボンメニューの説明 (個別機能)
Excel の上部メニュー（ **「アドイン」** タブ）に追加される専用リボン内の各ボタンの個別説明です。  
リボンは共通機能である **「ツール」** グループと、個別機能である **「Term制御」** グループの 2 つで構成されています。  
※ `GpibAddIn` と `TermAddIn`（および将来の拡張アドイン）を同一 PC に同時導入した場合、アドイン間の自動調停機能により、 **「アドイン」** タブ内には共通の **「ツール」** グループが 1 つだけ表示され、その横に **「GPIB制御」「Term制御」** が並んで表示されます。

![図1: TermAddIn の Excel リボン構成と「タスク番号の選択」ダイアログ](./images/term_excel_ribbon.svg)

| 図番 | グループ表示名 | ボタン表示名 | 主な機能概要 |
| :---: | :--- | :--- | :--- |
| **①〜②** | **ツール** | **スタイル保存** | 選択行（単体実行行 9 キー／制御ブロック行 6 キー）のセル書式を既定テンプレートとして `Setting.xml` へ保存 |
| **②** | | **Startボタン配置** | シート上に 4 列制御ブロック（`Start` ボタン・`Step` ボタン・`開始位置`・`現在位置`）を動的配置 |
| **②** | | **シーケンス削除** | 選択中のシーケンス行（2行分）または制御ブロック行（`Start` / `Step` 両ボタン含む）を安全に削除 |
| **②** | | **テレメトリ分析** | テレメトリログ（`telemetry_*.jsonl`）のコマンド別集計・棒グラフ・ウォーターフォール・構造化ログを GUI で可視化分析（[操作説明書](./Core/TelemetryAnalyzer.Md)） |
| **③〜④** | **Term制御** | **ツール起動** | ④ タスク番号の選択ダイアログ（`P1`〜`P10`）で選択したターミナルメイン画面を起動・前面表示 |
| **③〜④** | | **シーケンス配置** | ④ タスク番号選択後にコマンド編集画面をモーダル表示し、選択・編集したコマンドとパラメータの 2 行構成を基準列へ自動生成 |

---

### 2.1 【ツール】グループ (全製品共通リボン機能)

#### 🔘 スタイル保存 ボタン
*   **機能** : 現在アクティブなセルが属する 2 行ブロックの各種書式スタイル情報を読み取り、既定テンプレートとして設定ファイル（`Setting.xml` の共通 `Common` テーブル内 `Styles` セクション）に保存します。
*   **用途** : セルの背景色や罫線、フォント、文字揃え、列幅などのデザインをカスタマイズした後に、それを今後の新規配置時の「既定のスタイル」として登録したい場合に使用します。
*   **保存対象のスタイル要素と全 15 キー別管理** :
    1. **保存されるセル装飾要素** :
       * セルの背景色、罫線の太さ・線種・色（上下左右）、フォント名・サイズ・太字・斜体・文字色、文字配置（水平・垂直・折り返し）
       * 列の幅（ **※単体実行シーケンス行の 9 キーのみ保存・復元対象** ）
    2. **単体実行シーケンス行（9 キー）と制御ブロック行（6 キー）の自動判別・別管理** :
       * **単体実行シーケンス行を選択して保存した場合** : `Label`（実行ボタン列）、`WaitLabel` / `WaitValue`、`CommandLabel` / `CommandValue`、`ResultLabel` / `ResultValue`、`ParamHelp` / `ParamValue` の **9 キー** が列幅を含めて保存されます。
       * **Start / Step 制御ブロック行を選択して保存した場合** : 基準列から順に `StartButton`（`col`）、`StepButton`（`col + 1`）、`StartLabel` / `StartValue`（`col + 2`）、`CurrentLabel` / `CurrentValue`（`col + 3`）の **6 キー** が個別スタイルとして保存されます。ただし **制御ブロック 4 列の列幅は保存・復元の管理対象から除外されており、次回配置時も単体実行シーケンス行側で設定された列幅がそのまま維持されます** （列幅の上書き干渉を防止）。

#### 🔘 Startボタン配置 ボタン
*   **機能** : シーケンス全体の一括実行および 1 ステップずつのデバッグ実行を統括する **4 列構成の制御ブロック** を、シートの基準列に揃えて縦 2 行で動的配置します。
*   **配置される 4 列制御ブロックの構成（基準列 `col` 起点）** :
    1. **`col` 列：`[ Start ]` ボタン（縦 2 行結合）** : 一括実行用のコマンドボタンです。`現在位置`（空白時は `開始位置`）から連続実行を開始・途中再開します。
    2. **`col + 1` 列：`[ Step ]` ボタン（縦 2 行結合）** : ステップ実行用のコマンドボタンです。`現在位置`（空白時は `開始位置`）の 1 ステップのみを実行し、次の遷移先番地を `現在位置` へ記録します。
    3. **`col + 2` 列：`開始位置`（上段ラベル／下段ドロップダウン）** : シート内に配置されているすべての単体実行ボタン行のセル番地（例: `A5`, `A7` 等）が自動収集されて入力規則リストに設定されます（初期値は先頭のシーケンス行番地）。Excel 名前定義も指定可能です。
    4. **`col + 3` 列：`現在位置`（上段ラベル／下段プログラムカウンタ）** : 次に実行すべき対象行のセル番地を保持するセルです（初期値は空白）。実行完了時または `Excel_Exit` 到達時に自動で空白へクリアされます。
*   **双方向の基準列整列と既存ボタン行での自動シフト挿入確認** :
    *   先に単体実行シーケンス行がある場合も、先に Start 制御ブロックを配置してから単体実行行を追加する場合も、自動的に同一の先頭基準列に揃えて配置されます。
    *   選択した 2 行領域にすでにボタンが存在する場合は確認ダイアログを表示し、 **`[ はい ]`** 選択時は既存行の手前に **2 行を自動挿入（既存行を下へシフト）して Start / Step 制御ブロックを配置** します。
*   **実行の中断と安全ガード** :
    *   `Start` 一括実行中は中断ダイアログが表示され、任意のタイミングで安全に中断できます（中断時点の再開番地は `現在位置` に保持されます）。
    *   最大走査行数 `MaxRows`（既定 `500`）および最大連続実行ステップ数 `MaxExecutionSteps`（既定 `10,000`）により、意図しない無限ループによるフリーズを自動防止します。

#### 🔘 シーケンス削除 ボタン
*   **機能** : アクティブセルが存在する単体実行シーケンス行（2 行分）、または制御ブロック行（`Start` ボタン・`Step` ボタンを含む 2 行分）をシート上から安全に削除します。
*   **確認ダイアログとボタンの安全な削除** : 削除確認ダイアログで [はい] を選択した場合のみ、対象 2 行上のすべてのボタン（`Start` / `Step` / 単体 `実行` ボタン）を安全に解除・削除したうえで行削除（上方向シフト）を実行します。

#### 🔘 テレメトリ分析 ボタン
*   **機能** : 解析エンジンが出力するテレメトリログ（`telemetry_traces.jsonl` / `telemetry_logs.jsonl` / `telemetry_metrics.jsonl`）を読み込み、KPI サマリー・コマンド別処理時間集計・水平棒グラフ・`TraceId` ウォーターフォールタイムライン・構造化ログ・メトリクスを統合分析する GUI 画面（`TelemetryAnalyzer`）を開きます。
*   **詳細** : 画面構成や CSV 出力・フィルタ操作の詳細は [Monaca Telemetry Analyzer 操作説明書](./Core/TelemetryAnalyzer.Md) を参照してください。

---

### 2.2 【Term制御】グループ (Term専用リボン機能)

#### 🔘 ツール起動 ボタン
*   **機能** : 「タスク番号の選択」ダイアログを表示し、選択されたタスク番号（`P1`〜`P10`）に対応する通信制御メイン画面（[メイン画面取扱説明書](./メイン画面取扱説明書.Md)）を起動して表示します。
*   **用途** : シーケンスを編集・実行する前に各タスク（最大 10 系統）の機器接続状態や送受信ターミナルログを確認したい場合、またはコマンド編集画面（[コマンド編集画面取扱説明書](./コマンド編集画面取扱説明書.Md)）・プロパティ設定画面（[プロパティ設定画面取扱説明書](./プロパティ設定画面取扱説明書.Md)）を開きたい場合に使用します。
*   **マルチタスク管理仕様（`P1`〜`P10`）** :
    *   アドイン内では最大 10 個の独立した通信タスク（`P1`〜`P10`）を一元管理しています。
    *   すでに該当タスク番号の画面が起動済みの場合は多重起動せず、既存の画面を最前面にアクティブ化（最小化時は通常サイズへ復元）します。

#### 🔘 シーケンス配置 ボタン
*   **機能** : 「タスク番号の選択」ダイアログで操作対象タスク（`P1`〜`P10`）を選択したのち、該当タスクの **コマンド編集画面（Termコマンドパラメータ編集）をモーダル起動** します。コマンド編集画面上で目的のコマンドを選択し各パラメータ初期値を編集して閉じると、その選択コマンドと編集済みパラメータ値を反映した 2 行構成シーケンス行（個別「実行」ボタン、待機時間セル `+1` 列、コマンド名セル `+2` 列、結果指定セル `+3` 列、外部公開パラメータセル `+4` 列以降・列数上限なし）を基準列へ自動生成します。
*   **ミニシーケンス未読み込み時の安全ガード** :
    *   利用可能なコマンドが 1 件も登録されていない状態で実行した場合は、タスク選択前に警告メッセージを表示し、空行の不正配置を防止します。
*   **双方向の基準カラム自動統一と任意列実行** :
    *   シート内で最初にボタン（単体実行行または Start 制御ブロック）を配置した任意の列（A 列以外でも可）が基準カラムとして自動認識され、配置時・実行時ともにその基準カラムを起点に読み書きします。
*   **既存ボタン行での自動シフト挿入確認** :
    *   既存ボタンがある行を選択して実行した場合は確認ダイアログを表示し、[はい] 選択時に直下へ 2 行を自動挿入して新規シーケンスを配置します。
*   **外部公開パラメータ展開（`Work` 型除外）とジャンプ先候補リスト自動付与** :
    *   ミニシーケンス内で内部作業用に定義された `Work` 型（`-Work` 含む）パラメータはシート列へ展開せず、外部公開パラメータのみを `1`〜`N` 番目としてシートへ展開します。
    *   コマンド内の `Excel_Jump`・`Excel_Judge`・`Excel_Loop`・`Excel_Break`・`Excel_Msg_Box` からジャンプ先・脱出先として参照されている公開パラメータ列には、 **シート内の全単体実行ボタン番地（`A5`, `A7` 等）と Excel 名前定義の統合候補リスト** がドロップダウン入力規則として自動設定されます。
*   **`Result`（結果出力セル）の名前定義セル転記仕様** :
    *   結果出力項目が定義されたコマンドでは、ブック内の名前定義から出力先を選択できます。
    *   `Result` 下段セルに名前定義（例: `応答データ_CH1`）や別セル番地が設定されている場合、実行結果は **その指定先セルへ転記され、`Result` 下段セル自身の名前定義文字列は上書きされずに保持** されます（`Result` 下段が空欄の場合のみ `Result` 下段セル自身へ書き込まれます）。

---

## 🗂️ 3. シート上の実行ボタンと順序制御仕様

シート上に配置される 3 種類の ActiveX ボタンはいずれも配置直後から行挿入・行削除に自動追従し、ブック保存・再オープン時には単体実行ボタンの有効接続数が 1 件以上ある場合に `Start`・`Step` の双方が自動活性化されます。

![図2: Excel ワークシート上の 4 列制御ブロックおよび 2 行 1 ステップ単体実行シーケンスの画面構成](./images/term_excel_sheet.svg)

*   **① `[ Start ]` / `[ Step ]` ボタン（基準列・`+1` 列、縦 2 行結合）** : 連続一括実行および 1 ステップデバッグ実行を開始します。
*   **② `開始位置` / `現在位置` セル（`+2` 列・`+3` 列）** : `開始位置` にはシート内の全シーケンス行番地のドロップダウンが自動設定され、`現在位置` には実行中または一時停止中の行番地（プログラムカウンタ）がリアルタイム表示されます。
*   **③ 単体 `[ 実行 ]` ボタンと基本 4 列（`Wait(ms)` / `Command` / `Result`）** : `Command` 列にはタスク番号接頭辞付きのコマンド名（例: `P1:REG_READ`, `P2:PORT_CLOSE`）が入り、ドロップダウン変更時は右側のパラメータ列が即座に自動再構築されます。
*   **④ `Result` 名前定義セルへの自動転記** : `Result` 下段セルに Excel の名前定義（例: `応答データ_CH1`）を指定しておくと、受信・抽出した結果データはその名前定義セルへ自動転記され、`Result` セル自身の名前定義指定は上書きされずに保持されます。
*   **⑤ 外部公開パラメータ列（`+4` 列以降）** : CSV 定義の `Work` 型（内部作業変数）は自動的に除外され、外部公開パラメータのみが整然と展開されます（`List` 型選択肢や `Excel_Judge` 等の遷移先番地ドロップダウンも自動付与されます）。

### 🔘 1. 単体実行ボタン（各シーケンス行の「実行」ボタン）
*   **配置** : 各シーケンス行の基準列（縦 2 行結合セル）に配置されます。
*   **動作** : クリックしたその行の 1 ステップのみを単発実行します。コマンド内に `Excel_Jump` 等のフロー制御コマンドが含まれていても、制御ブロックの `開始位置`・`現在位置` およびシート単位ループスタックは **一切変更しません** 。

### 🔘 2. ステップ実行ボタン（制御ブロックの「Step」ボタン）
*   **配置** : 制御ブロックの `col + 1` 列（縦 2 行結合セル）に配置されます。
*   **動作** : `現在位置` セル（空欄時は `開始位置` セル）が指すシーケンス行を **1 ステップだけ実行** し、コマンド実行結果のフロー制御指定（次行進行・ジャンプ・ループ開始／折り返し・脱出・終了）を判定して、 **次に実行すべき行番地を `現在位置` セルへ書き込みます** （末尾到達または `Excel_Exit` 実行時は `現在位置` を空白クリア）。1 行ずつ値や遷移先を確認しながら進めるデバッグ実行に最適です。

### 🔘 3. 一括実行ボタン（制御ブロックの「Start」ボタン）
*   **配置** : 制御ブロックの基準列 `col`（縦 2 行結合セル）に配置されます。
*   **動作** : `現在位置` セルが空欄の場合は `開始位置` から新規開始し、`現在位置` に番地が入っている場合（`Step` 実行の途中、`Excel_Break` による一時停止後、またはキャンセル中断後）は **その `現在位置` の続きから途中再開** して、1 ステップごとに `現在位置` を動的更新しながら連続実行します。ライセンス未認証等で無効状態となっているボタン行は `"NOP"`（何もしない行）として自動スキップして進行し、正常完走または `Excel_Exit` 到達時に `現在位置` とループスタックをクリアします。

### 🔀 4. Excel エクステンション共通コマンドによる順序制御
専用ボタンを増やすことなく、通常のミニシーケンス（`,SEQUENCE`）内に `Core\Excel` ドメインの 8 コマンド（`Excel_Jump`, `Excel_Judge`, `Excel_Loop`, `Excel_Next`, `Excel_Break`, `Excel_Wait`, `Excel_Msg_Box`, `Excel_Exit`）を記述することで、測定値に応じたシート行の条件分岐ジャンプや複数行区間の反復ループを実行できます。ミニシーケンス側ではジャンプ先を **1 始まりの公開パラメータ番号** で指定し、実際の遷移先セル番地（`A12` 等）または Excel 名前定義はシート側の対応パラメータ列で柔軟に指定します。

---

## 💾 4. 設定の自動保存・復元機能
*   **Excel セルの装飾テンプレート（全 15 キー）** : 「スタイル保存」で登録した単体実行行（9 キー）および制御ブロック行（6 キー、列幅非干渉）の書式は `Setting.xml` に自動保存され、次回配置時にそのまま再現されます。
*   **各通信プロトコルおよび画面座標設定** : COM ポート条件、Telnet 接続先、Modbus 32 チャネル設定、MELSEC 計算機リンク / MCプロトコル設定、OMRON FINS / Sysmac 設定、KEYENCE KV 上位リンク設定、および各タスク画面の表示位置・非公開コマンド表示状態は `Setting.xml` に自動保存・復元されます。

---

## ⚙️ 5. 設定ファイル (Setting.xml) について
本システムの動作環境や装飾スタイル情報は、以下の設定ファイルに一元管理されています（メイン画面の `[設定(S)] ➔ [設定(S)...]` から開く環境設定ダイアログにて保存先確認や表示言語切替を行い、 **`[ OK ]`** ボタンで確定、 **`[ キャンセル ]`** ボタンで破棄できます）。

*   **保存先パス** : `%USERPROFILE%\Desktop\Monaca\Setting.xml`（既定のデスクトップ `Monaca` フォルダ）
*   **ファイル形式** : XML 形式（全製品共通の `Common` テーブル、およびタスク別の `Monaca Term` / `Monaca Term-2`〜`Monaca Term-10` テーブルで構成）

### ① 主な記録内容と設定詳細

| 分類 | 設定テーブル / セクション | 主な設定項目 (キー) | 説明 |
| :--- | :--- | :--- | :--- |
| **Excelスタイル関連** | `Common` / `Styles` | 単体実行行 9 キー（`Label`, `WaitLabel`, `WaitValue`, `CommandLabel`, `CommandValue`, `ResultLabel`, `ResultValue`, `ParamHelp`, `ParamValue`）<br>制御ブロック 6 キー（`StartButton`, `StepButton`, `StartLabel`, `StartValue`, `CurrentLabel`, `CurrentValue`） | 各キーにつきカンマ（`,`）区切り 1 行 CSV 形式（4 辺罫線は `:` 区切り）で背景色・罫線・フォント・配置・列幅（制御ブロック 6 キーは列幅除外）を保持します（旧形式の設定が存在する場合は自動移行）。 |
| **シーケンス実行上限** | `Common` / `Sequence` | `MaxRows`（既定 `500`）<br>`MaxExecutionSteps`（既定 `10000`） | シーケンス走査時の最大行数および `Start` 連続実行時の無限ループ防止上限ステップ数を指定します。 |
| **通信モード・コマンド集** | `Monaca Term` / `ModeSetting`, `Command` | `ModeTerm`（通信モード）、`SELECT`（有効選択数）、`COMMAND00`〜（`タイトル,CSVパス`） | タスクごとの選択中通信モード、およびコマンド集タブで登録・有効化されたマクロスクリプト一覧を保持します。 |
| **基本通信設定（COM / Telnet / Modbus）** | `Monaca Term` / `Comm`, `Telnet`, `ModbusRTU`, `ModbusSocket` | `CommPort`, `BaudRate`, `Parity`, `DataBits`, `StopBits`, `Handshake`, `Echo`, `WaitPrompt` / `HostIp`, `Port`, `ConnectTimeoutSec`, `LocalEcho`, `NewLine` / `SelectedChannel`, `CH01`〜`CH32` | シリアル（UART）条件、Telnet 接続条件、および Modbus RTU / Modbus TCP の全 32 チャネル設定をタスク別に保持します。 |
| **PLC拡張設定（MELSEC / OMRON / KEYENCE）** | `Monaca Term` / `MELSEC`, `MELSEC_MC`, `OMRON_FINS`, `KEYENCE_KV` | `PortName`, `ProtocolType`, `StationNo`, `PcNo` / `Host`, `Port`, `Protocol`, `NetworkNo`, `FrameFormat` / `DestNet`, `DestNode`, `SrcNode`, `UnitNo` / `SerialPortName`, `SerialFormat`, `DefaultDataSuffix`, `AutoSessionHandshake`, `TimeoutMs` 等 | 三菱電機 MELSEC（計算機リンク・MCプロトコル QnA 互換 3E バイナリフレーム）、オムロン FINS / HostLink、およびキーエンス KV 上位リンクの通信パラメータをタスク別に保持します。 |
| **システムログ設定** | `Monaca Term` / `Log` | `Dir` / `File` / `Level` | ログ保存先フォルダ、ファイル名、および出力レベル（全 8 種類：`Test`, `Debug`, `Warning`, `Error`, `Save`, `Info`, `Critical`, `None`）を指定します。 |
| **画面の位置・サイズ** | `Monaca Term` / `Point` | 各画面識別名 | メイン画面および **コマンド編集画面** の表示座標・ウィンドウサイズ・画面スケール情報（5 要素）、ならびに **プロパティ設定画面** の表示座標 `(X, Y)`（2 要素）を保持します。 |
| **表示フォント・表示制御・言語設定** | `Monaca Term` / `Properties`<br>`Common` / `Properties`, `System` | `Font` / `VisibleCont` / `Language` | ターミナルログ表示フォント設定（`[フォント名],[サイズ],[スタイル],[文字色]`）、コマンド編集画面の非公開コマンド隠蔽状態（`VisibleCont`）、および UI 表示言語（`""`, `"ja-JP"`, `"en-US"`）を保持します。 |

### ② 保守・管理について
*   **設定のバックアップ** : PC 移行時は `Setting.xml` を移行先の `%USERPROFILE%\Desktop\Monaca\` へコピーすることで、通信条件や 15 キーのスタイル設定をそのまま引き継げます。
*   **設定変更履歴（監査ログ）** : 設定保存時にはログファイル（既定 `info.log`）へ変更前後の履歴が自動記録されます。
*   **設定の初期化** : `Setting.xml` を削除（または退避）して再起動すると、初期設定で自動再生成されます。

---

## ⚡ 6. 高速ファイルオープンと除外キャッシュ機能（オーバーヘッド防止）

日常業務で無関係な Excel ブックを開いた際のオーバーヘッドを実質ゼロにする高速スキップ機構を備えています。

### ① 拡張子による早期判定（Fast Path スキップ）
シート上の実行ボタンを保持できない非ブック形式やアドイン形式（`.csv`, `.tsv`, `.txt`, `.prn`, `.dif`, `.slk`, `.xla`, `.xlam`）を開いた場合は、シート走査やライセンス検証を行わず **0ms で即座にスキップ** します（標準対応形式である `.xlsx` / `.xlsm` はボタン存在確認を実施）。

### ② ボタン非存在時の無効判定と除外キャッシュの永続化
走査の結果、本アドインの実行ボタンが存在しない通常ブックは「対象外」と判定され、`%USERPROFILE%\Desktop\Monaca\IgnoredWorkbooks.txt`（全 Monaca アドイン共通）へ記録されて 2 回目以降のオープン時は即座にバイパスされます。

### ③ 手動キャッシュクリアとボタン操作時の自動昇格・降格
*   `%USERPROFILE%\Desktop\Monaca\IgnoredWorkbooks.txt` を削除するだけでキャッシュは即座にリセットされます。
*   リボンから「Startボタン配置」「シーケンス配置」「シーケンス削除」を実行した際は、該当ブックの除外キャッシュが自動解除され、最新のボタン有無に応じて再判定されます。

---

## 💻 7. 動作環境と Excel 32bit / 64bit 運用上の留意事項

本アドイン（TermAddIn）は `.NET Framework 4.8` の標準シリアル通信および TCP/UDP ネットワーク通信機能をベースに 32bit / 64bit 両対応で構築されているため、 **Excel 32bit 版および Excel 64bit 版のいずれの環境でも追加ドライバ制約なく完全に動作** します。

| 通信プロトコル / 機能ドメイン | Excel 32bit 版 | Excel 64bit 版 | 備考・動作方式 |
| :--- | :---: | :---: | :--- |
| **COM シリアル通信（`Comm`）** | **◯ 完全動作** | **◯ 完全動作** | OS 標準 COM ポート（オンボード RS-232C / USB-シリアル変換器）を介して動作。 |
| **Telnet ソケット通信（`Telnet`）** | **◯ 完全動作** | **◯ 完全動作** | OS 標準 TCP/IP ソケットを介して動作。 |
| **Modbus 通信（`Modbus`）** | **◯ 完全動作** | **◯ 完全動作** | Modbus RTU / ASCII（COM）および Modbus TCP（Ethernet）ともに完全動作。 |
| **MELSEC 計算機リンク（`Melco`）** | **◯ 完全動作** | **◯ 完全動作** | 形式 1〜5 フレーム送受信および形式 1〜4 高水準デバイス操作ともに標準シリアル回線上で完全動作。 |
| **MELSEC MCプロトコル（`MelcoMC`）** | **◯ 完全動作** | **◯ 完全動作** | QnA / Q / L / iQ-R / iQ-F 向け **QnA 互換 3E フレーム（バイナリ形式・TCP 専用）** で完全動作。 |
| **OMRON FINS / Sysmac（`Omron`）** | **◯ 完全動作** | **◯ 完全動作** | FINS/UDP・FINS/TCP およびシリアル HostLink（FA / Cモード）ともに完全動作。 |
| **KEYENCE KV 上位リンク（`Keyence`）** | **◯ 完全動作** | **◯ 完全動作** | Ethernet TCP/UDP（Port 8501）および RS-232C シリアル上位リンクともに完全動作。 |

*   市販の USB-シリアル変換ケーブルを使用する場合は、ご使用の Windows OS（64bit / 32bit）に対応した仮想 COM ポート（VCP）ドライバーがデバイスマネージャー上で認識されていることを確認してください。
*   実機のシリアル機器や PLC が手元にない環境で事前開発・デバッグを行う場合は、公式無償ツール **[c2c（仮想ヌルモデム＆通信モニター）](https://github.com/Monaca-App/c2c)** および **[TermEmulator（総合仮想デバイス＆産業用プロトコルスレーブ）](https://github.com/Monaca-App/TermEmulator)** を併用することで、カーネルドライバの導入や管理者権限なしに仮想 COM ポートペアやローカル TCP/UDP 上で折り返しテスト・PLC 仮想応答検証を行えます。

---

## 📚 8. マクロコマンド・関連マニュアル一覧

本アドインのシーケンス内で利用可能な Term 固有通信コマンド（ **7 ドメイン・67 コマンド** ）および全製品共通基盤コマンド（ **7 ドメイン・63 コマンド** ）のリファレンス案内です（ **総合計 130 コマンド** ）。

### ① 画面操作マニュアル・マクロ総合ガイド・関連ツール
*   [メイン画面取扱説明書 (`メイン画面取扱説明書.Md`)](./メイン画面取扱説明書.Md)
*   [コマンド編集画面取扱説明書 (`コマンド編集画面取扱説明書.Md`)](./コマンド編集画面取扱説明書.Md)
*   [プロパティ設定画面取扱説明書 (`プロパティ設定画面取扱説明書.Md`)](./プロパティ設定画面取扱説明書.Md)
*   [マクロスクリプト取扱説明書 (`マクロスクリプト取扱説明書.Md`)](./マクロスクリプト取扱説明書.Md)
*   [c2c - ドライバ不要の仮想ヌルモデム＆通信モニター案内 (`README.md`)](https://github.com/Monaca-App/c2c)
*   [TermEmulator - 総合仮想デバイス＆産業用プロトコルスレーブ案内 (`README.md`)](https://github.com/Monaca-App/TermEmulator)

### ② Term 固有通信制御コマンド（全 7 ドメイン・67 コマンド）

#### 1. COM ポート通信制御（`Comm`：全 6 コマンド）
RS-232C / UART シリアルインターフェースを介した機器制御コマンド群です（[Comm エクステンション専用マニュアル](https://github.com/Monaca-App/TermAddIn_Comm)）。
*   [`Com_Open`](./Comm/Com_Open.Md) / [`Com_Close`](./Comm/Com_Close.Md) / [`Com_Send`](./Comm/Com_Send.Md) / [`Com_SendFile`](./Comm/Com_SendFile.Md) / [`Com_WaitPrompt`](./Comm/Com_WaitPrompt.Md) / [`Com_Mode`](./Comm/Com_Mode.Md)

#### 2. Telnet 通信制御（`Telnet`：全 6 コマンド）
TCP/IP ネットワーク（Telnet / 生ソケット）を介した機器制御コマンド群です（[Telnet エクステンション専用マニュアル](https://github.com/Monaca-App/TermAddIn_Telnet)）。
*   [`Telnet_Open`](./Telnet/Telnet_Open.Md) / [`Telnet_Close`](./Telnet/Telnet_Close.Md) / [`Telnet_Send`](./Telnet/Telnet_Send.Md) / [`Telnet_SendFile`](./Telnet/Telnet_SendFile.Md) / [`Telnet_WaitPrompt`](./Telnet/Telnet_WaitPrompt.Md) / [`Telnet_Mode`](./Telnet/Telnet_Mode.Md)

#### 3. Modbus 通信制御（`Modbus`：全 5 コマンド）
Modbus RTU / ASCII / TCP プロトコルによる産業機器・PLC レジスタ制御コマンド群です（[Modbus エクステンション専用マニュアル](https://github.com/Monaca-App/TermAddIn_Modbus)）。
*   [`Modbus_Open`](./Modbus/Modbus_Open.Md) / [`Modbus_Close`](./Modbus/Modbus_Close.Md) / [`Modbus_Read`](./Modbus/Modbus_Read.Md) / [`Modbus_Write`](./Modbus/Modbus_Write.Md) / [`Modbus_Mode`](./Modbus/Modbus_Mode.Md)

#### 4. 三菱電機 MELSEC 計算機リンク通信制御（`Melco`：全 23 コマンド）
MELSEC 計算機リンク（形式 1〜5 送受信および高水準デバイス操作）によるシーケンサ通信コマンド群です（[Melco エクステンション専用マニュアル](https://github.com/Monaca-App/TermAddIn_Melco)）。
*   **回線制御・通信条件（3コマンド）** : [`Melsec_Open`](./Melco/Melsec_Open.Md) / [`Melsec_Close`](./Melco/Melsec_Close.Md) / [`Melsec_Mode`](./Melco/Melsec_Mode.Md)
*   **高水準デバイス操作・診断（5コマンド）** : [`Melsec_ReadWord`](./Melco/Melsec_ReadWord.Md) / [`Melsec_WriteWord`](./Melco/Melsec_WriteWord.Md) / [`Melsec_ReadBit`](./Melco/Melsec_ReadBit.Md) / [`Melsec_WriteBit`](./Melco/Melsec_WriteBit.Md) / [`Melsec_Loopback`](./Melco/Melsec_Loopback.Md)
*   **形式 1〜5（15コマンド）** : [`Melsec_P1s_S`](./Melco/Melsec_P1s_S.Md) / [`Melsec_P1s_R`](./Melco/Melsec_P1s_R.Md) / [`Melsec_Res_P1s_R`](./Melco/Melsec_Res_P1s_R.Md) / [`Melsec_P2s_S`](./Melco/Melsec_P2s_S.Md) / [`Melsec_P2s_R`](./Melco/Melsec_P2s_R.Md) / [`Melsec_Res_P2s_R`](./Melco/Melsec_Res_P2s_R.Md) / [`Melsec_P3s_S`](./Melco/Melsec_P3s_S.Md) / [`Melsec_P3s_R`](./Melco/Melsec_P3s_R.Md) / [`Melsec_Res_P3s_R`](./Melco/Melsec_Res_P3s_R.Md) / [`Melsec_P4s_S`](./Melco/Melsec_P4s_S.Md) / [`Melsec_P4s_R`](./Melco/Melsec_P4s_R.Md) / [`Melsec_Res_P4s_R`](./Melco/Melsec_Res_P4s_R.Md) / [`Melsec_P5s_S`](./Melco/Melsec_P5s_S.Md) / [`Melsec_P5s_R`](./Melco/Melsec_P5s_R.Md) / [`Melsec_Res_P5s_R`](./Melco/Melsec_Res_P5s_R.Md)

#### 5. 三菱電機 MELSEC MCプロトコル通信制御（`MelcoMC`：全 9 コマンド）
Ethernet（TCP）経由の QnA 互換 3E フレーム（バイナリ形式）によるシーケンサ通信コマンド群です（[MelcoMC エクステンション専用マニュアル](https://github.com/Monaca-App/TermAddIn_MelcoMC)）。
*   **回線制御・通信条件（3コマンド）** : [`Mc_Open`](./MelcoMC/Mc_Open.Md) / [`Mc_Close`](./MelcoMC/Mc_Close.Md) / [`Mc_Mode`](./MelcoMC/Mc_Mode.Md)
*   **高水準デバイス操作（4コマンド）** : [`Mc_ReadWord`](./MelcoMC/Mc_ReadWord.Md) / [`Mc_WriteWord`](./MelcoMC/Mc_WriteWord.Md) / [`Mc_ReadBit`](./MelcoMC/Mc_ReadBit.Md) / [`Mc_WriteBit`](./MelcoMC/Mc_WriteBit.Md)
*   **汎用電文送受信（2コマンド）** : [`Mc_Send`](./MelcoMC/Mc_Send.Md) / [`Mc_Receive`](./MelcoMC/Mc_Receive.Md)

#### 6. オムロン FINS / Sysmac 通信制御（`Omron`：全 9 コマンド）
FINS/UDP・FINS/TCP およびシリアル HostLink による SYSMAC シーケンサ通信コマンド群です（[Omron エクステンション専用マニュアル](https://github.com/Monaca-App/TermAddIn_Omron)）。
*   **回線制御・経路設定（3コマンド）** : [`Fins_Open`](./Omron/Fins_Open.Md) / [`Fins_Close`](./Omron/Fins_Close.Md) / [`Fins_Mode`](./Omron/Fins_Mode.Md)
*   **高水準デバイス操作（4コマンド）** : [`Fins_ReadWord`](./Omron/Fins_ReadWord.Md) / [`Fins_WriteWord`](./Omron/Fins_WriteWord.Md) / [`Fins_ReadBit`](./Omron/Fins_ReadBit.Md) / [`Fins_WriteBit`](./Omron/Fins_WriteBit.Md)
*   **汎用電文送受信（2コマンド）** : [`Fins_Send`](./Omron/Fins_Send.Md) / [`Fins_Receive`](./Omron/Fins_Receive.Md)

#### 7. キーエンス KV 上位リンク通信制御（`Keyence`：全 9 コマンド）
Ethernet TCP/UDP（Port 8501）および RS-232C シリアル経由の KV 上位リンク通信コマンド群です（[Keyence エクステンション専用マニュアル](https://github.com/Monaca-App/TermAddIn_Keyence)）。
*   **回線制御・条件設定（3コマンド）** : [`Kv_Open`](./Keyence/Kv_Open.Md) / [`Kv_Close`](./Keyence/Kv_Close.Md) / [`Kv_Mode`](./Keyence/Kv_Mode.Md)
*   **高水準デバイス操作（4コマンド）** : [`Kv_ReadWord`](./Keyence/Kv_ReadWord.Md) / [`Kv_WriteWord`](./Keyence/Kv_WriteWord.Md) / [`Kv_ReadBit`](./Keyence/Kv_ReadBit.Md) / [`Kv_WriteBit`](./Keyence/Kv_WriteBit.Md)
*   **汎用電文送受信（2コマンド）** : [`Kv_Send`](./Keyence/Kv_Send.Md) / [`Kv_Receive`](./Keyence/Kv_Receive.Md)

### ③ 共通基盤マクロコマンド（全 7 ドメイン・63 コマンド：[共通基盤マニュアル総合目次](../README.md)）
通信プロトコルやハードウェアの種類に依存しない、全製品共通の Excel シート順序制御・シーケンス内部制御・パラメータ演算・配列操作・ワーク・ファイル・透過ポート通信コマンド群です。

*   **1. Excel シート順序制御エクステンション（`Core/Excel`：全 8 コマンド）** :
    *   [シート行無条件ジャンプ (`Excel_Jump`)](./Core/Excel/Excel_Jump.Md) / [シート行条件分岐判定 (`Excel_Judge`)](./Core/Excel/Excel_Judge.Md)
    *   [シート行ループ開始 (`Excel_Loop`)](./Core/Excel/Excel_Loop.Md) / [シート行ループ終端 (`Excel_Next`)](./Core/Excel/Excel_Next.Md) / [ループ脱出・一時中断 (`Excel_Break`)](./Core/Excel/Excel_Break.Md)
    *   [ミリ秒待機 (`Excel_Wait`)](./Core/Excel/Excel_Wait.Md) / [対話確認・分岐 (`Excel_Msg_Box`)](./Core/Excel/Excel_Msg_Box.Md) / [シート実行終了・停止 (`Excel_Exit`)](./Core/Excel/Excel_Exit.Md)
*   **2. パラメータ・サブルーチン・ループ制御（`Core/Param`：全 17 コマンド）** :
    *   [パラメータ設定 (`Param_Set`)](./Core/Param/Param_Set.Md) / [パラメータ取得 (`Param_Get`)](./Core/Param/Param_Get.Md) / [パラメータ転送 (`Param_Move`)](./Core/Param/Param_Move.Md)
    *   [サブルーチン呼出 (`Call`)](./Core/Param/Call.Md) / [シーケンス終了・復帰 (`Exit`)](./Core/Param/Exit.Md) / [回数指定ループ (`Loop`)](./Core/Param/Loop.Md) / [ループ終端 (`Next`)](./Core/Param/Next.Md) / [ループ脱出 (`Break`)](./Core/Param/Break.Md)
    *   [加算 (`Param_Add`)](./Core/Param/Param_Add.Md) / [乗算 (`Param_Mul`)](./Core/Param/Param_Mul.Md) / [除算 (`Param_Div`)](./Core/Param/Param_Div.Md) / [剰余 (`Param_Mod`)](./Core/Param/Param_Mod.Md)
    *   [直接減算 (`ParamSub`)](./Core/Param/ParamSub.Md) / [ステップ加算 (`ParamInc`)](./Core/Param/ParamInc.Md) / [ステップ減算 (`ParamDec`)](./Core/Param/ParamDec.Md) / [16進加算 (`ParamHexAdd`)](./Core/Param/ParamHexAdd.Md) / [数式評価 (`CalcExpression`)](./Core/Param/CalcExpression.Md)
*   **3. 配列操作（`Core/Array`：全 7 コマンド）** :
    *   [配列初期化 (`Array_Init`)](./Core/Array/Array_Init.Md) / [要素設定 (`Array_Set`)](./Core/Array/Array_Set.Md) / [要素・行取得 (`Array_Get`)](./Core/Array/Array_Get.Md) / [矩形分割 (`Array_Split`)](./Core/Array/Array_Split.Md) / [水平結合 (`Array_Join`)](./Core/Array/Array_Join.Md) / [文字列置換 (`Array_Replace`)](./Core/Array/Array_Replace.Md) / [行列転置 (`Array_Transpose`)](./Core/Array/Array_Transpose.Md)
*   **4. シーケンス分岐・エラートラップ・待機・外部起動（`Core/Sequence`：全 17 コマンド）** :
    *   [無条件ラベルジャンプ (`Jump`)](./Core/Sequence/Jump.Md) / [合否・条件判定 (`Judge`)](./Core/Sequence/Judge.Md) / [文字列等価判定 (`Equal`)](./Core/Sequence/Equal.Md) / [コール復帰 (`Return`)](./Core/Sequence/Return.Md)
    *   [エラートラップ登録 (`OnError`)](./Core/Sequence/OnError.Md) / [エラークリア (`ClearError`)](./Core/Sequence/ClearError.Md) / [任意エラー発火 (`SetError`)](./Core/Sequence/SetError.Md) / [バージョン検査 (`Version`)](./Core/Sequence/Version.Md)
    *   [ミリ秒待機 (`Wait`)](./Core/Sequence/Wait.Md) / [ミリ秒待機 (`WaitMs`)](./Core/Sequence/WaitMs.Md) / [秒数待機 (`WaitSec`)](./Core/Sequence/WaitSec.Md) / [時刻到達待機 (`WaitTime`)](./Core/Sequence/WaitTime.Md) / [条件一致待機 (`WaitUntil`)](./Core/Sequence/WaitUntil.Md) / [ファイル出現待機 (`WaitFile`)](./Core/Sequence/WaitFile.Md) / [プロセス終了待機 (`WaitProcess`)](./Core/Sequence/WaitProcess.Md)
    *   [外部プロセス呼出 (`Ext_Call`)](./Core/Sequence/Ext_Call.Md) / [メッセージ表示 (`Msg_Box`)](./Core/Sequence/Msg_Box.Md)
*   **5. データ・ワーク処理（`Core/Work`：全 5 コマンド）** :
    *   [受信データ取込 (`Set_Data`)](./Core/Work/Set_Data.Md) / [指定行抽出 (`Get_Line`)](./Core/Work/Get_Line.Md) / [行スキップ (`Skip_Line`)](./Core/Work/Skip_Line.Md) / [データスキップ (`Skip_Data`)](./Core/Work/Skip_Data.Md) / [タイムスタンプ制御・取得 (`Time_Stamp`)](./Core/Work/Time_Stamp.Md)
*   **6. ファイル制御（`Core/File`：全 5 コマンド）** :
    *   [ファイル読込 (`Read_File`)](./Core/File/Read_File.Md) / [ファイル書込 (`Write_File`)](./Core/File/Write_File.Md) / [ファイル追記 (`Append_File`)](./Core/File/Append_File.Md) / [行読込 (`Read_Line`)](./Core/File/Read_Line.Md) / [行書込 (`Write_Line`)](./Core/File/Write_Line.Md)
*   **7. 透過ポート通信制御（`Core/Port`：全 4 コマンド）** :
    *   [文字列非同期送信 (`PortSend`)](./Core/Port/PortSend.Md) / [CR終端送信 (`SendCR`)](./Core/Port/SendCR.Md) / [ポート応答受信 (`PortReceive`)](./Core/Port/PortReceive.Md) / [送受信クエリ (`PortQuery`)](./Core/Port/PortQuery.Md)
