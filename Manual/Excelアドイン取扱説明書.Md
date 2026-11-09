# Excelアドイン取扱説明書 (User Manual)

本ドキュメントは、Excel 上からシリアル機器（RS-232C / UART）、ネットワーク機器（Telnet）、産業用 Modbus 機器、および三菱電機 MELSEC シーケンサのテストシーケンスを構築・実行するための **Excel アドイン（TermAddIn）** の総合取扱説明書です。リボン機能、シート上のボタン仕様、設定ファイル（`Setting.xml`）、除外キャッシュ機構、32bit/64bit 動作環境、および関連マクロコマンドリファレンスを本書 1 冊に統合して解説します。

---

# 目次

- [📖 1. 概要](#-1-概要)
- [🛠️ 2. リボンメニューの説明 (個別機能)](#️-2-リボンメニューの説明-個別機能)
  - [2.1 【ツール】グループ (`tool`：全製品共通リボン機能)](#21-ツールグループ-tool全製品共通リボン機能)
  - [2.2 【Term制御】グループ (`Term`：Term専用リボン機能)](#22-term制御グループ-termterm専用リボン機能)
- [🗂️ 3. シート上の実行ボタン説明 (個別機能)](#️-3-シート上の実行ボタン説明-個別機能)
- [💾 4. 設定の自動保存・復元機能](#-4-設定の自動保存復元機能)
- [⚙️ 5. 設定ファイル (Setting.xml) について](#️-5-設定ファイル-settingxml-について)
- [⚡ 6. 高速ファイルオープンと除外キャッシュ機能（オーバーヘッド防止）](#-6-高速ファイルオープンと除外キャッシュ機能オーバーヘッド防止)
- [💻 7. 動作環境と Excel 32bit / 64bit 運用上の留意事項](#-7-動作環境と-excel-32bit--64bit-運用上の留意事項)
- [📚 8. マクロコマンド・関連マニュアル一覧](#-8-マクロコマンド関連マニュアル一覧)

---

## 📖 1. 概要
本 Excel アドイン（TermAddIn）は、Excel のワークシート上に通信コマンドのシーケンス（実行手順）をセル単位で定義し、ボタン一つで非同期に機器制御・データ収集を実行できるアドインです。コマンド名や待機時間（ウェイト）、パラメータ、実行結果を Excel 画面上で視覚的に管理・編集できます。

### 💡 本アドイン最大の特長：VBAマクロ不要・標準 `.xlsx` 形式での完全動作
*   **マクロ無効形式（`.xlsx`）のまま動作**:
    一般的なマクロ機能とは異なり、ブック内部に VBA コードを一切保持しません。Excel シート上のコマンド実行ボタン（ActiveX ボタン）はアドイン側から動的に制御されるため、 **標準的な `.xlsx` 形式のままボタンの配置・保存・自動実行が可能** です。
*   **セキュリティ警告・ブロックの解消**:
    マクロ有効ブック（`.xlsm`）を使用しないため、企業のセキュリティポリシーによるマクロ実行ブロックやセキュリティ警告に煩わされることなく、安全にシーケンスシートを作成・運用・共有できます。
*   **VBE（マクロ）の保守不要**:
    Excel のバージョンアップによる参照設定トラブルやマクロコード破損の心配がなく、長期的なシステムの安定性を保ちます。

### 📌 共通基盤アーキテクチャの責務
1. **VBA/VBE 参照の完全非依存**:
   * マクロ有効形式（`.xlsm`）や VBE 参照を必要とせず、標準的な `.xlsx` 形式のまま動作します。
2. **ActiveX 動的生成と GC 生存管理**:
   * `Forms.CommandButton.1` を OLEObject として動的配置し、生成された COM 参照およびイベントデリゲートを `ActiveXEventManager` により GC 回収から保護します。
3. **安全な COM リソース解放**:
   * シート操作・セル走査完了後は `Marshal.ReleaseComObject` を徹底し、Excel プロセスの残存を完全に防止します。
4. **多層動的リンク解決との統合**:
   * 各種ドキュメント参照は `DocPathResolver` を経由し、開発環境・GitHub Web・ユーザー作業フォルダを動的に切り替えます。

---

## 🛠️ 2. リボンメニューの説明 (個別機能)
Excel の上部メニュー（Office のアドインタブ `TabAddIns`）に追加される専用リボン内の各ボタンの個別説明です。  
リボンは共通機能である **「ツール (`tool`)」** グループと、個別機能である **「Term制御 (`Term`)」** グループの 2 つで構成されています。  
※ `GpibAddIn` と `TermAddIn`（および将来の拡張アドイン）を同一 PC に同時導入した場合、プロセス環境変数のオーナー調停機構（`ClaimCommonToolGroupOwnership`）により、アドインタブ（`TabAddIns`）内には共通の **「ツール」** グループが 1 つだけ表示され、その横に **「GPIB制御」「Term制御」** （および将来追加される機能ごとの個別制御グループ）が並んで表示されます。

| グループID / 表示名 | ボタンID / 表示名 | 主な機能概要 |
| :--- | :--- | :--- |
| **`tool`**<br>(ツール) | **`Make_Doc`** (マニュアル配置) | 取扱説明書（`Manual`）およびサンプル資材（`ListSample`）をローカル作業フォルダへ展開 |
| | **`Save_Style`** (スタイル保存) | 選択行のセル装飾・書式スタイルを読み取り、既定テンプレートとして設定へ保存 |
| | **`Start`** (Startボタン配置) | シート上に一括連続実行用「Start」ボタンと開始位置ドロップダウンセルを動的配置 |
| | **`Delete_Sequence`** (シーケンス削除) | 選択中のシーケンス行（2行分）またはStartボタン行を安全に削除（ActiveX/COM解放） |
| **`Term`**<br>(Term制御) | **`ToolOpen`** (ツール起動) | 通信制御用ターミナルのメイン画面を起動・表示 |
| | **`Make_Sequence`** (シーケンス配置) | 選択セルを起点としてコマンド実行ボタン・待機時間・コマンド・結果・引数の2行構成を自動生成 |

---

### 2.1 【ツール】グループ (`tool`：全製品共通リボン機能)

#### 🔘 マニュアル配置 ボタン (`Make_Doc`)
*   **機能**: アドインに同梱された技術ドキュメント（取扱説明書、コマンド仕様書）およびサンプル資材群（サンプルマクロ `seq.csv` 等）をユーザーの作業ディレクトリへ安全に自動展開します。
*   **用途**: 各種コマンドの仕様や取扱説明書を手元で確認したい場合、またはマクロ作成用のサンプルファイルを流用したい場合に実行します。
*   **展開される資材一式**:
    1. **取扱説明書一式 (`Manual/`)**: 総合取扱説明書、各画面マニュアル、各通信ドメイン（Comm / Telnet / Modbus / Melco）のコマンド仕様書
    2. **サンプル資材一式 (`ListSample/`)**: 基本シーケンスのサンプルマクロスクリプト（`seq.csv`）等
*   **展開先優先順位と安全性**:
    *   **展開先ディレクトリの解決**:
        1. ユーザー設定（`Setting.xml`）で指定されたカスタムパス（例: `%USERPROFILE%\Desktop\Monaca\Doc\`）
        2. 未指定時は、標準のドキュメント／AppData フォルダ（`Monaca\Doc\` 配下）
    *   **差分マージ（上書き安全性）**: 既存ファイルを無条件に削除せず、タイムスタンプを比較して新しいファイルのみを安全に更新します。
    *   **リンク自動正規化**: Markdown ファイル内の相対リンクは、配布先環境の階層構造に合わせて自動的に正規化されます。

#### 🔘 スタイル保存 ボタン (`Save_Style`)
*   **機能**: 現在アクティブなセル（または選択行）の各種書式スタイル情報を読み取り、既定テンプレートとして設定ファイル（`Setting.xml`）に永続化します。
*   **用途**: セルの色や線の太さ、幅などのデザインをカスタマイズした後に、それを今後の新規作成時の「既定のスタイル」として登録したい場合に使用します。
*   **保存対象のスタイル要素**:
    1. **セルの装飾プロパティ**:
       * 背景色（`Interior.Color`）
       * 罫線の太さ・線種・色（上下左右・格子・斜線）
       * フォント名、フォントサイズ、太字、斜体、文字色
       * 文字配置（左右の揃え、上下の揃え、折り返して全体を表示）
       * 列の幅（`ColumnWidth`）
    2. **シーケンス用スタイルと Start ボタン用スタイルの自動判別**:
       * 選択行に通常シーケンス行が含まれる場合は、シーケンス用の基準装飾テンプレートとして保存されます。
       * 選択行に Start ボタンが含まれる場合は、Start ボタンおよび開始位置指定セル（ラベルおよびドロップダウン）専用の装飾スタイルとして個別に保存されます。

#### 🔘 Startボタン配置 ボタン (`Start`)
*   **機能**: シーケンスを一括で自動実行するための **「実行開始（Start）」ボタン** と、その右隣に開始位置を指定する **「実行開始位置」セル（ドロップダウン）** をシート上に動的配置します。
*   **用途**: 構築したテストシーケンス全体を上から順に自動で連続実行させたい場合に使用します。
*   **配置要素と動作仕様**:
    *   **Start ボタン (ActiveX コマンドボタン)**: 縦 2 行結合セル上に配置される `Forms.CommandButton.1` コントロールです。クリック時に、右隣の開始位置指定セルで指定された行から順次シーケンスを下方向へ一括連続実行します。
    *   **開始位置指定セル (ドロップダウンリスト)**: ボタンの右隣セルに配置され、シート内に存在する個別実行ボタンのアドレス（例: `A5`, `A7` 等）が入力規則リストとして自動設定されます。
    *   **二重配置防止**: 同一行にすでに Start ボタンが存在する場合は警告ダイアログを表示し、重複生成を防止します。
    *   **実行の中断**: 実行中は共通中断ダイアログ（`CancelDialog`）が表示され、ユーザーはいつでも安全にシーケンスを中断できます。

#### 🔘 シーケンス削除 ボタン (`Delete_Sequence`)
*   **機能**: アクティブセルが存在するシーケンス行（2行分）、または選択した一括実行ボタンの行をシート上から安全に削除します。
*   **用途**: 不要になったシーケンスや一括実行ボタンをシートから削除したい場合に使用します。
*   **確認ダイアログ（誤操作防止）**: ボタン押下時に削除確認メッセージが表示されます。[はい] を選択した場合にのみ削除処理が実行され、[いいえ] を選択した場合は処理を中断します。
*   **COM/ActiveX 安全解放**: Excel 内部の OLEObject およびイベントデリゲート参照を安全に解放したうえで対象 2 行を行削除（上方向へシフト）します。

---

### 2.2 【Term制御】グループ (`Term`：Term専用リボン機能)

#### 🔘 ツール起動 ボタン (`ToolOpen`)
*   **機能**: 通信制御用アプリケーションのメイン画面（[メイン画面取扱説明書](./メイン画面取扱説明書.Md)）を起動して表示します。
*   **用途**: シーケンスを編集・実行する前に、実際の機器接続状態や送受信ターミナルログを確認したい場合、またはコマンド編集画面（[コマンド編集画面取扱説明書](./コマンド編集画面取扱説明書.Md)）・プロパティ設定画面（[プロパティ設定画面取扱説明書](./プロパティ設定画面取扱説明書.Md)）を開きたい場合に使用します。
*   **多重起動・タスク管理仕様**:
    *   `TermMainForm` は **多重起動（最大 10 タスク）を許可** しています。
    *   起動時にタスク番号が指定されていない場合、空いている最小のタスク番号（1〜10）が自動的に割り当てられます。
    *   特定の起動タスク番号（`P1`〜`P10` または `P=1`〜`P=10` 等）が指定されている場合、その番号がすでに起動中であれば多重起動せず、既存の画面を最前面にアクティブ化（最小化されている場合は通常サイズに復元）して安全に終了します。

#### 🔘 シーケンス配置 ボタン (`Make_Sequence`)
*   **機能**: 選択されているアクティブセルを起点として、個別実行用ボタン（先頭列）、待機時間設定セル（+1列）、コマンド名指定セル（+2列）、結果書き込みセル（+3列）、パラメータ記述セル（+4列以降）のシーケンス構成行（2行分）を自動生成します。
*   **用途**: シート上に新しい通信制御コマンド行を追加・構築したい場合に使用します。
*   **ミニシーケンス未読み込み時の安全ガード**:
    *   通信用マクロスクリプト（ミニシーケンス）が未読み込みで、利用可能なコマンドが 1 件も登録されていない状態で本ボタンを実行した場合、「利用可能なコマンドが存在しません。先にマクロスクリプト（ミニシーケンス）を読み込んでください。」の警告メッセージを表示し、シーケンス配置および行挿入処理を安全に中止します（規定外コマンドや空行の不正配置を防止）。
*   **基準カラム（配置列）の自動統一**:
    *   シート内で最初にシーケンスを配置した列（カラム）が、そのシートにおける「基準カラム」として自動認識されます。
    *   2回目以降は、シート上のどの列のセルを選択して「シーケンス配置」を実行しても、自動的に同一の基準カラムに整然と揃えて配置されます（列のズレを防止）。
*   **既存シーケンス行への配置時の自動挿入機能**:
    *   すでにシーケンスが存在する行を選択して本ボタンを実行した場合、「すでにシーケンスがある行です。次の行に挿入してシーケンスを作成しますか？」の確認ダイアログが表示されます。
    *   **[はい]**: 既存シーケンスの直下に2行が自動挿入（行追加）され、空いた行へ即座に新規シーケンスが作成されます（手動の行挿入なしで中間への安全なコマンド割り込みが可能）。
    *   **[いいえ]**: シーケンス作成を中止（キャンセル）し、既存の行構成を変更しません。
*   **エディタ画面との連携**:
    *   本ボタンをクリックすると、設定対象のコマンドやパラメータをグラフィカルに指定できる「コマンド編集画面」が表示されます。
    *   コマンド編集画面でコマンドおよびパラメータを確定すると、選択されたコマンド名および各引数のラベル・初期値・入力規則リストがシート上へ自動展開されます。
*   **Result（結果出力セル）の動作仕様**:
    *   配置対象のコマンドにデータ出力（`ResultItem`）が定義されている場合、ブック内の名前定義を検索し、**出力先候補（名前付きセル）の選択ダイアログ** が自動表示されます。
    *   **名前付きセル選択時**: 選択した名前定義（例: `応答データ_CH1` 等）が結果セル下段に設定されます。
    *   **未選択時（または名前定義なし）**: そのシーケンス行の結果セル自身のアドレス（例: `D10`）が設定されます。
    *   **実行完了時**: シーケンス実行エンジンがコマンド処理を完了すると、指定されたセルへ直接実行ステータス（`"OK"`, `"NG"`）や測定データ文字列が自動的に書き込まれます。

---

## 🗂️ 3. シート上の実行ボタン説明 (個別機能)
シート上に配置されるボタンの個別説明です。

### 🔘 シーケンス実行ボタン (個別「実行」ボタン)
*   **配置**: 「シーケンス配置」実行時に、基準列（Label）の縦2行の結合セル上に配置されます。
*   **機能**: クリックすると、その行（+2列）に指定されたコマンドを、+4列以降のパラメータ設定値を反映したうえで単発で実行します。
*   **用途**: シーケンスの一部だけを単体テストしたい場合や、個別の通信コマンドのみを単発で動作させたい場合に使用します。

### 🔘 一括実行ボタン (Startボタン)
*   **配置**: 「Startボタン配置」を実行した任意のセル（縦2行の結合セル上）に配置されます。
*   **機能**: 右隣の「実行開始位置」ドロップダウン（セルアドレス（例: `A5`）または名前定義を選択）で指定した位置の個別実行ボタンから開始し、下に向かってシーケンスを順番に自動実行していきます。
*   **用途**: 準備した全シーケンスを一気通貫でテストラン・自動検査させたい場合に使用します。

---

## 💾 4. 設定の自動保存・復元機能
本アドインでは、ユーザーがカスタマイズしたセルの各種装飾デザインや通信設定を次回もそのまま適用できるようにするため、設定の保存と読み込みを自動的に行っています。

### ① 自動的に保存・復元される情報
*   **Excel セルの装飾テンプレート**
    *   シーケンス定義行の装飾仕様（セルの背景色、罫線の太さや種類、左右・上下の文字配置、列の幅など）を「スタイル保存」機能により設定情報として保持し、次回「シーケンス配置」ボタンをクリックして新しいシーケンス行を追加する際に、同一のデザインでセルが生成されます。
*   **各通信プロトコルおよび画面座標設定**
    *   COM ポート条件、Telnet 接続先、Modbus 設定、および各ウィンドウの表示位置は `Setting.xml` に自動保存され、次回起動時に復元されます。

---

## ⚙️ 5. 設定ファイル (Setting.xml) について
本システム（Excel アドイン、メイン画面、コマンド編集画面、プロパティ設定画面）の動作環境や装飾スタイル情報は、以下の設定ファイルに一元管理されています。

*   **保存先パス**: `%USERPROFILE%\Desktop\Monaca\Setting.xml`（`<USER_DIR>\Desktop\Monaca\Setting.xml`）
    *   ※ デスクトップ上の `Monaca` フォルダがデフォルトの配置先となります。
*   **ファイル形式**: XML形式

### ① 主な記録内容と設定詳細
設定ファイル内には、以下の設定項目が体系的に保存されています。

| 分類 | 設定セクション | 主な設定項目 (キー) | 説明 |
| :--- | :--- | :--- | :--- |
| **Excelスタイル関連** | `TermAddIn_Styles` 等 | `Style`, `Width`, `Color`, `AlignmentHorizontal`, `AlignmentVertical`, `LineStyle_Border...` | Excelシート上の各列および一括実行行について、背景色、境界線、文字の配置方法、列の幅といった装飾用テンプレート情報を保持します。 |
| **コマンド集・プロトコル設定** | `Property` / `Command` / `Comm` / `Telnet` / `Modbus` | `CommandFile`, `PortName`, `BaudRate`, `Parity`, `DataBits`, `StopBits`, `IpAddress`, `TcpPort`, `SlaveId` 等 | 読み込むマクロスクリプト（CSV）のパス、および COM ポート・Telnet・Modbus の各通信パラメータを保持します。 |
| **システムログ設定** | `Log` | `Dir`（保存フォルダ）<br>`File`（ファイル名）<br>`Level`（出力レベル種別） | ログを保存する出力先のフォルダパス、ログファイル名、および出力レベル（`Error`, `Warning`, `Info`, `Debug`, および設定変更履歴専用の `Save`）を指定します。 |
| **画面の位置・サイズ** | `Point` | 各画面の名前（メイン画面 / 編集画面 / プロパティ設定画面） | それぞれの画面を最後に閉じた時点での、画面上の表示位置（座標）やウィンドウの寸法を保持します。 |
| **表示フォント設定** | `Properties` | `Font_Name`, `Font_Size`, `Font_Bold`, `ForeColor` など | ターミナル送受信ログ表示領域で使われるテキストのフォント名、文字の大きさ、スタイル、および表示文字色を保持します。 |
| **表示言語設定** | `Setting` | `Language` | UI の表示言語コード（`""` = 自動判別/システム既定、`"ja-JP"` = 日本語、`"en-US"` = 英語）を保持します。メイン画面の環境設定ダイアログから変更できます。 |

### ② 保守・管理について
*   **設定のバックアップ**:
    PCの移行時や設定の複製を行いたい場合は、この `Setting.xml` ファイルをコピーして移行先の同じフォルダ（デスクトップの `Monaca` フォルダ）に配置することで、全く同じ通信環境やスタイル設定を引き継ぐことができます。
*   **設定変更履歴（監査ログ）の確認**:
    環境設定や通信設定が変更・保存された際は、ログファイル（既定では同フォルダ内の `info.log`）に「変更日時」「変更項目」「変更前 ➔ 変更後」の履歴が自動記録されます（保存先やファイル名は、メイン画面のメニュー [環境設定...] ➔ [エラー履歴] から確認・変更可能です）。
*   **設定の初期化**:
    設定データが破損した場合や、初期状態に戻したい場合は、このファイルを削除（または一時的に退避）してシステムを再起動することで、デフォルトの初期設定ファイルが自動的に再生成されます。

---

## ⚡ 6. 高速ファイルオープンと除外キャッシュ機能（オーバーヘッド防止）

本アドインでは、日常業務で関係のない通常の Excel ファイルを開いた際に動作が重くならないよう、 **オーバーヘッドを実質ゼロにする高速スキップおよび除外キャッシュ機構** を備えています。

### ① 拡張子による早期判定（Fast Path スキップ）
ActiveX コントロールを保持できない非ブック形式（`.csv`、`.tsv`、`.txt`、`.xml` 等）を開いた場合、シートの解析やライセンス検証などの内部処理を一切行わずに **0ms で即座に処理をスキップ** します。
※ なお、`.xlsx` 形式は本アドインの標準対応形式（VBAマクロを使わずにActiveXボタンを配置・保存可能）であるため、通常通りボタンの存在確認が行われます。

### ② ボタン非存在時の無効判定と除外キャッシュの永続化
`.xlsx` や `.xlsm` 等のファイルであっても、走査の結果 Monaca のシーケンスボタンが存在しない無関係なファイル（通常業務の帳票等）の場合は「対象外」と判定され、そのファイルパスが除外キャッシュファイルに自動保存されます。

*   **キャッシュファイルパス**: `%USERPROFILE%\Desktop\Monaca\TermAddIn.IgnoredWorkbooks.txt`（`Setting.xml` と同一フォルダ）
*   **効果**: 一度「対象外」と判定されたファイルは、次回以降 Excel を開いた際にも再走査を行わず即座にバイパス（オーバーヘッド 0ms）されます。これにより、関係のない通常の `.xlsx` ファイルも 2 回目以降は完全に高速オープンされます。

### ③ ユーザーによる手動キャッシュクリア
キャッシュ状態をリセットして全ファイルを再判定させたい場合は、 **`%USERPROFILE%\Desktop\Monaca\TermAddIn.IgnoredWorkbooks.txt` ファイルを削除するだけ** で完了します。アドインはファイルの削除を自動検知し、内部の保持状態も含めて即座にクリアされます。

### ④ ボタン操作時の自動リセット（昇格・降格連携）
*   **シーケンス作成時**: 対象外と判定されていたブックであっても、リボンから「シーケンス作成」または「Start配置」を行うと、判定キャッシュが無効化（リセット）され、有効な対象ブックとして認識されます。
*   **シーケンス削除時**: リボンから「シーケンス削除」を実行した際も判定キャッシュが無効化されます。ボタンが全削除された場合は、次回オープン時に自動で「対象外（無効）」へと降格登録されます。

---

## 💻 7. 動作環境と Excel 32bit / 64bit 運用上の留意事項

本アドイン（TermAddIn）は、`.NET Framework 4.8` の標準マネージド通信基盤（`System.IO.Ports.SerialPort` および `System.Net.Sockets`）をベースに `AnyCPU` で構築されているため、 **Excel 32bit 版および Excel 64bit 版のいずれの環境でもネイティブ DLL 制約なく完全に動作** します。

### ① Excel 32bit / 64bit 対応状況まとめ

| 通信プロトコル / 機能ドメイン | Excel 32bit 版 | Excel 64bit 版 | 備考・動作方式 |
| :--- | :---: | :---: | :--- |
| **COM シリアル通信（`TermAnalysis.Comm`）** | **◯ 完全動作** | **◯ 完全動作** | OS 標準 COM ポート（オンボード RS-232C / USB-シリアル変換器）を介して 32bit/64bit 問わず動作。 |
| **Telnet ソケット通信（`TermAnalysis.Telnet`）** | **◯ 完全動作** | **◯ 完全動作** | OS 標準 TCP/IP ソケットを介して 32bit/64bit 問わず動作。 |
| **Modbus 通信（`TermAnalysis.Modbus`）** | **◯ 完全動作** | **◯ 完全動作** | Modbus RTU / ASCII（COM）および Modbus TCP（Ethernet）ともに 32bit/64bit 問わず完全動作。 |
| **MELSEC 計算機リンク（`TermAnalysis.Melco`）** | **◯ 完全動作** | **◯ 完全動作** | プロトコル1 / プロトコル4 ともに標準シリアル回線上で 32bit/64bit 問わず完全動作。 |

### ② USB-シリアル変換ケーブル等を使用する場合の留意事項
*   市販の USB-RS232C / USB-RS485 変換ケーブル（FTDI、Prolific、Silicon Labs 製チップ搭載品等）を使用する場合は、ご使用の Windows OS（64bit / 32bit）に対応した仮想 COM ポート（VCP）ドライバーが Windows デバイスマネージャー上で正常に認識されていることを確認してください（Excel 側の 32bit / 64bit ビット数には依存しません）。

---

## 📚 8. マクロコマンド・関連マニュアル一覧

本アドインのシーケンス内で利用可能な Term 固有通信コマンド（4 ドメイン・26 コマンド）および全製品共通基盤コマンド（5 ドメイン・34 コマンド）のリファレンス案内です。

### ① 画面操作マニュアル・マクロ総合ガイド
*   [メイン画面取扱説明書 (`メイン画面取扱説明書.Md`)](./メイン画面取扱説明書.Md)
*   [コマンド編集画面取扱説明書 (`コマンド編集画面取扱説明書.Md`)](./コマンド編集画面取扱説明書.Md)
*   [プロパティ設定画面取扱説明書 (`プロパティ設定画面取扱説明書.Md`)](./プロパティ設定画面取扱説明書.Md)
*   [マクロスクリプト取扱説明書 (`マクロスクリプト取扱説明書.Md`)](./マクロスクリプト取扱説明書.Md)

### ② Term 固有通信制御コマンド（全 4 ドメイン・26 コマンド）

#### 1. COM ポート通信制御（`Comm`：全 6 コマンド）
RS-232C / UART シリアルインターフェースを介した機器制御コマンド群です（[Comm エクステンション専用マニュアル](https://github.com/Monaca-App/TermAddIn_Comm)）。
*   [`Com_Open`](https://github.com/Monaca-App/TermAddIn_Comm/blob/main/Manual/Comm/Com_Open.Md): COM ポート回線の接続オープン（自動排他切替）
*   [`Com_Close`](https://github.com/Monaca-App/TermAddIn_Comm/blob/main/Manual/Comm/Com_Close.Md): COM ポート回線の切断・解放
*   [`Com_Send`](https://github.com/Monaca-App/TermAddIn_Comm/blob/main/Manual/Comm/Com_Send.Md): 文字列電文送信・エラー確認付き送信
*   [`Com_SendFile`](https://github.com/Monaca-App/TermAddIn_Comm/blob/main/Manual/Comm/Com_SendFile.Md): テキストファイルの行単位逐次送信
*   [`Com_WaitPrompt`](https://github.com/Monaca-App/TermAddIn_Comm/blob/main/Manual/Comm/Com_WaitPrompt.Md): 指定プロンプト文字列の受信待機（タイムアウト分岐対応）
*   [`Com_Mode`](https://github.com/Monaca-App/TermAddIn_Comm/blob/main/Manual/Comm/Com_Mode.Md): 通信条件・フロー制御・終端デリミタ・ASCII/BIN モードの一括／個別設定

#### 2. Telnet 通信制御（`Telnet`：全 6 コマンド）
TCP/IP ネットワーク（Telnet / 生ソケット）を介した機器制御コマンド群です（[Telnet エクステンション専用マニュアル](https://github.com/Monaca-App/TermAddIn_Telnet)）。
*   [`Telnet_Open`](https://github.com/Monaca-App/TermAddIn_Telnet/blob/main/Manual/Telnet/Telnet_Open.Md): Telnet ソケット接続の確立（自動排他切替）
*   [`Telnet_Close`](https://github.com/Monaca-App/TermAddIn_Telnet/blob/main/Manual/Telnet/Telnet_Close.Md): Telnet ソケットセッションの切断・解放
*   [`Telnet_Send`](https://github.com/Monaca-App/TermAddIn_Telnet/blob/main/Manual/Telnet/Telnet_Send.Md): 文字列電文送信・エラー確認付き送信
*   [`Telnet_SendFile`](https://github.com/Monaca-App/TermAddIn_Telnet/blob/main/Manual/Telnet/Telnet_SendFile.Md): テキストファイルの行単位逐次送信
*   [`Telnet_WaitPrompt`](https://github.com/Monaca-App/TermAddIn_Telnet/blob/main/Manual/Telnet/Telnet_WaitPrompt.Md): 指定プロンプト文字列の受信待機（タイムアウト分岐対応）
*   [`Telnet_Mode`](https://github.com/Monaca-App/TermAddIn_Telnet/blob/main/Manual/Telnet/Telnet_Mode.Md): 送信終端デリミタ・ASCII/BIN モード・NVT（IAC）フィルタ設定

#### 3. Modbus 通信制御（`Modbus`：全 5 コマンド）
Modbus RTU / ASCII / TCP プロトコルによる産業機器・温調計・インバータ・PLC レジスタ制御コマンド群です（[Modbus エクステンション専用マニュアル](https://github.com/Monaca-App/TermAddIn_Modbus)）。
*   [`Modbus_Open`](https://github.com/Monaca-App/TermAddIn_Modbus/blob/main/Manual/Modbus/Modbus_Open.Md): Modbus 通信回線（UART または TCP）の接続オープン
*   [`Modbus_Close`](https://github.com/Monaca-App/TermAddIn_Modbus/blob/main/Manual/Modbus/Modbus_Close.Md): Modbus 通信回線の切断・解放
*   [`Modbus_Read`](https://github.com/Monaca-App/TermAddIn_Modbus/blob/main/Manual/Modbus/Modbus_Read.Md): 保持レジスタ（FC03）／入力レジスタ（FC04）の読出実行
*   [`Modbus_Write`](https://github.com/Monaca-App/TermAddIn_Modbus/blob/main/Manual/Modbus/Modbus_Write.Md): 単一／複数レジスタへの書込実行（CRC-16 / LRC / MBAP 自動付与）
*   [`Modbus_Mode`](https://github.com/Monaca-App/TermAddIn_Modbus/blob/main/Manual/Modbus/Modbus_Mode.Md): 伝送モード（`RTU` / `ASCII` / `TCP`）および通信条件の設定

#### 4. 三菱電機 MELSEC シーケンサ通信制御（`Melco`：全 9 コマンド）
MELSEC 計算機リンク（プロトコル 1 / プロトコル 4）によるシーケンサ通信コマンド群です（[Melco エクステンション専用マニュアル](https://github.com/Monaca-App/TermAddIn_Melco)）。
*   [`Melsec_Open`](https://github.com/Monaca-App/TermAddIn_Melco/blob/main/Manual/Melco/Melsec_Open.Md) / [`Melsec_Close`](https://github.com/Monaca-App/TermAddIn_Melco/blob/main/Manual/Melco/Melsec_Close.Md) / [`Melsec_Mode`](https://github.com/Monaca-App/TermAddIn_Melco/blob/main/Manual/Melco/Melsec_Mode.Md): 回線接続・切断・通信モード設定
*   [`Melsec_P1s_S`](https://github.com/Monaca-App/TermAddIn_Melco/blob/main/Manual/Melco/Melsec_P1s_S.Md) / [`Melsec_P1s_R`](https://github.com/Monaca-App/TermAddIn_Melco/blob/main/Manual/Melco/Melsec_P1s_R.Md) / [`Melsec_Res_P1s_R`](https://github.com/Monaca-App/TermAddIn_Melco/blob/main/Manual/Melco/Melsec_Res_P1s_R.Md): プロトコル 1 フレーム送受信・応答検証
*   [`Melsec_P4s_S`](https://github.com/Monaca-App/TermAddIn_Melco/blob/main/Manual/Melco/Melsec_P4s_S.Md) / [`Melsec_P4s_R`](https://github.com/Monaca-App/TermAddIn_Melco/blob/main/Manual/Melco/Melsec_P4s_R.Md) / [`Melsec_Res_P4s_R`](https://github.com/Monaca-App/TermAddIn_Melco/blob/main/Manual/Melco/Melsec_Res_P4s_R.Md): プロトコル 4 フレーム送受信・応答検証

### ③ 共通基盤マクロコマンド（[全34コマンド共通基盤マニュアル目次](../README.md)）
通信プロトコルやハードウェアの種類に依存しない、全製品共通のシーケンス制御・パラメータ演算・配列操作・ファイル・ワーク処理コマンド群です。

*   **シーケンス制御（Sequence / Param）**:
    *   [合否・条件一致判定 (`Judge` / 旧 `Equ` 統合)](./Core/Sequence/Judge.Md) / [無条件ジャンプ (`Jump`)](./Core/Sequence/Jump.Md) / [サブルーチン呼出 (`Call`)](./Core/Param/Call.Md) / [シーケンス終了・復帰 (`Exit`)](./Core/Param/Exit.Md)
    *   [回数指定ループ (`Loop`)](./Core/Param/Loop.Md) / [ループ終端 (`Next`)](./Core/Param/Next.Md) / [ループ脱出 (`Break`)](./Core/Param/Break.Md)
*   **パラメータ演算（Param）**:
    *   [パラメータ設定 (`Param_Set`)](./Core/Param/Param_Set.Md) / [パラメータ取得 (`Param_Get`)](./Core/Param/Param_Get.Md) / [パラメータ転送 (`Param_Move`)](./Core/Param/Param_Move.Md)
    *   [加算 (`Param_Add`)](./Core/Param/Param_Add.Md) / [乗算 (`Param_Mul`)](./Core/Param/Param_Mul.Md) / [除算 (`Param_Div`)](./Core/Param/Param_Div.Md) / [剰余 (`Param_Mod`)](./Core/Param/Param_Mod.Md)
*   **配列操作（Array）**:
    *   [配列初期化 (`Array_Init`)](./Core/Array/Array_Init.Md) / [要素設定 (`Array_Set`)](./Core/Array/Array_Set.Md) / [要素・行取得 (`Array_Get`)](./Core/Array/Array_Get.Md) / [矩形分割 (`Array_Split`)](./Core/Array/Array_Split.Md) / [水平結合 (`Array_Join`)](./Core/Array/Array_Join.Md) / [文字列置換 (`Array_Replace`)](./Core/Array/Array_Replace.Md) / [行列転置 (`Array_Transpose`)](./Core/Array/Array_Transpose.Md)
*   **待機・ログ・ワーク処理（Work / Sequence / File）**:
    *   [ミリ秒待機 (`Wait`)](./Core/Sequence/Wait.Md) / [外部プロセス呼出 (`Ext_Call`)](./Core/Sequence/Ext_Call.Md) / [メッセージ表示 (`Msg_Box`)](./Core/Sequence/Msg_Box.Md)
    *   [タイムスタンプ (`Time_Stamp`)](./Core/Work/Time_Stamp.Md) / [変数・セル書込 (`Set_Data`)](./Core/Work/Set_Data.Md) / [指定行抽出 (`Get_Line`)](./Core/Work/Get_Line.Md) / [データスキップ (`Skip_Data`)](./Core/Work/Skip_Data.Md) / [行スキップ (`Skip_Line`)](./Core/Work/Skip_Line.Md)
    *   [ファイル読込 (`Read_File`)](./Core/File/Read_File.Md) / [ファイル書込 (`Write_File`)](./Core/File/Write_File.Md) / [ファイル追記 (`Append_File`)](./Core/File/Append_File.Md) / [行読込 (`Read_Line`)](./Core/File/Read_Line.Md) / [行書込 (`Write_Line`)](./Core/File/Write_Line.Md)

