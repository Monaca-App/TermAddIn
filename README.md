<img src="images/Monaca_store.png" align="right" width="130" alt="Monaca Mascot" />

# TermAddIn - Excel シリアル・ネットワーク・PLC 通信制御アドイン

👉 [← Monaca アプリ工房（トップ）に戻る](https://github.com/Monaca-App/store)

---

[![Platform](https://img.shields.io/badge/Platform-Windows%2010%20%2F%2011-blue.svg)](https://www.microsoft.com/)
[![Excel](https://img.shields.io/badge/Excel-2016%20%2F%202019%20%2F%202021%20%2F%20M365-green.svg)](https://www.microsoft.com/excel)
[![Framework](https://img.shields.io/badge/.NET%20Framework-4.8-purple.svg)](https://dotnet.microsoft.com/)
[![Architecture](https://img.shields.io/badge/Architecture-VSTO%20Add--In-orange.svg)]()
[![License](https://img.shields.io/badge/Trial-Free%20Trial%20Available-brightgreen.svg)]()

## 📖 概要

**TermAddIn** は、Microsoft Excel のワークシート上から RS-232C シリアル接続機器、TCP/IP ネットワーク機器（Telnet）、産業用 Modbus デバイス（RTU / ASCII / TCP）、ならびに三菱電機 MELSEC・オムロン SYSMAC・キーエンス KV シーケンサの通信制御シーケンスを構築・実行・データ収集できる Excel VSTO アドインです。

プログラミング言語による面倒なコーディングやマクロ（VBA）の開発を行うことなく、 **使い慣れた Excel のセルにコマンドとパラメータを入力するだけ** で、機器への電文送信、プロンプト待機、レジスタ読出・書込、測定値のセル自動展開、一括シーケンス実行がすべて Excel 上で完結します。

> [!IMPORTANT]
> ### 💡 本アドイン最大の設計思想・特長
> **「VBAマクロに一切依存せず、VSTO側からActiveXコントロールを動的バインドすることで、標準のマクロ無効ブック形式（`.xlsx`）のままでシーケンスボタンを配置・保存・動作できる」**
> 
> * **セキュリティ警告・ブロックの完全回避**: 近年の Windows / Office 環境ではマクロ有効ブック（`.xlsm`）に対するセキュリティ制限（信頼できない発行元のブロック、インターネット属性・MOTW によるマクロ強制停止等）が極めて厳格化されています。本アドインはブック側に一切のマクロ（VBA）を含まないため、 **標準の `.xlsx` ファイルとして何らセキュリティ警告を受けることなく安全に作成・保存・運用・社内共有** できます。
> * **VBE・マクロ保守の手間ゼロ**: Excel のバージョン差異による VBE 参照設定エラーやマクロコード破損のリスクがなく、長期にわたる検査・計測システムの安定稼働を実現します。

※本アドインは現在 **公開中** です。

> [!NOTE]
> **詳細マニュアルのご案内**  
> 各種機能の詳しい使い方や詳細仕様については、[Excelアドイン取扱説明書](./Manual/Excelアドイン取扱説明書.Md) をご参照ください。

---

## ✨ 主な機能・特徴

### 1. VBAマクロ完全非依存 ＆ 標準 `.xlsx` での完全動作
* 本アドイン最大の強みとして、Excel シート上に動的配置されるコマンド実行ボタン（ActiveX ボタン）の制御・イベントハンドリングはすべてアドイン（VSTO / .NET）側から動的にバインド・制御されます。
* そのため、ブックをマクロ有効形式（`.xlsm`）に変換することなく、 **普段使いの標準的な `.xlsx` 形式のままシーケンスボタンの配置・保存・実行がすべて完結** します。マクロの実行許可設定やセキュリティポリシーに縛られない、極めて高い利便性と安全性を両立しています。

### 2. プラグイン型エクステンション拡張アーキテクチャ（7大通信プロトコル標準搭載 ＆ 今後も拡張予定）
* **機能ドメイン単一 DLL（ExtAnalysis）構造**: TermAddIn は各通信機能が独立したエクステンション（DLL）として分離されており、必要な機能のみをロードして安全かつ軽量に稼働します。
* **標準搭載の7大通信エクステンション**:
  * **[RS-232C / UART シリアル通信（`Comm`）](https://github.com/Monaca-App/TermAddIn_Comm)**: ボーレート、パリティ、データ長、ストップビット、フロー制御（RTS/CTS, XON/XOFF）、終端デリミタ（CR / CRLF）、ASCII / バイナリ透過モードを柔軟に制御し、組込基板や計測器との対話通信を実現します。
  * **[TCP/IP Telnet ソケット通信（`Telnet`）](https://github.com/Monaca-App/TermAddIn_Telnet)**: ネットワーク機器や産業用コントローラに対して IP アドレス・ポート番号指定でソケット接続し、NVT（IAC コマンド）自動ネゴシエーションやプロンプト待機を行えます。
  * **[Modbus 通信制御（`Modbus`）](https://github.com/Monaca-App/TermAddIn_Modbus)**: **Modbus RTU** 、 **Modbus ASCII** 、および **Modbus TCP** の全3モードに対応。CRC-16 / LRC チェックサムや MBAP ヘッダを自動生成・検証し、保持レジスタ（FC03）・入力レジスタ（FC04）の読出および単一・複数レジスタ書込をコマンド1行で実行できます。
  * **[三菱電機 MELSEC 計算機リンク（`Melco`）](https://github.com/Monaca-App/TermAddIn_Melco)**: 形式 1 〜 形式 5 の全プロトコルフレームを完全網羅。要求・応答フレーム送受信、ENQ / ACK / NAK 制御、サムチェック自動算出・照合に加え、直感的なデバイス名（D100, M0 等）でアクセス可能な高水準コマンド群を提供します。
  * **[三菱電機 MELSEC MCプロトコル通信（`MelcoMC`）](https://github.com/Monaca-App/TermAddIn_MelcoMC)**: イーサネット経由の業界標準 **MC プロトコル（QnA 互換 3E フレーム・TCP/IP バイナリ交信）** に完全対応。シーケンサ CPU や Ethernet ユニット直結でワード・ビットデバイスの高速一括読出・書込を実行できます。
  * **[オムロン FINS / Sysmac 通信（`Omron`）](https://github.com/Monaca-App/TermAddIn_Omron)**: **FINS/UDP（ポート 9600）** 、 **FINS/TCP（自動ノード通知ハンドシェイク付き）** 、および **シリアル HostLink（FCS 自動計算）** の全3方式に対応。SYSMAC CS/CJ/CP/NJ/NX シリーズの各メモリエリア（DM, CIO, WR, HR, AR, EM 等）をコマンド1行で制御できます。
  * **[キーエンス KV 上位リンク通信（`Keyence`）](https://github.com/Monaca-App/TermAddIn_Keyence)**: **Ethernet TCP/UDP 上位リンク（標準ポート 8501）** および **RS-232C シリアル上位リンク（自動 `CR`/`CQ` 対応）** に対応。KV-8000〜KV-Nano の DM/EM/W/MR/R 等をデータ形式サフィックス（`.U`/`.S`/`.H`/`.D`/`.L`）付きで高速制御できます。
* **将来拡張予定のエクステンション・ラインナップ（順次拡張予定）**:
  * **無線・IoT 計測エクステンション（`BLE` / `Wireless`）**: Bluetooth Low Energy / SPP による無線センサロギング。

### 3. プロンプト自動待機・受信データ抽出（sscanf / sprintf 2次元展開）
* 相手機器から特定の応答プロンプト（例: `"OK"`, `">"`, `"login:"` 等）が返るまで非同期で待機し、タイムアウト時には指定ラベルへ自動分岐する堅牢な対話制御が可能です。
* 受信した電文は共通受信ワークバッファへ自動格納され、`Set_Data` コマンドの書式指定によって必要な数値やステータスだけを抽出し、Excel セルへ直接展開できます。

### 4. セル単位でテストシーケンスを簡単構築
* リボンメニューの「シーケンス配置」をクリックするだけで、コマンド実行ボタン・ウェイト・コマンド名・結果セル・パラメータ枠がシート上に自動生成されます。
* リスト型のパラメータには Excel の入力規則（ドロップダウンリスト）が自動適用され、入力ミスを防ぎます。

### 5. 単発実行 ＆ 4列制御ブロックによる一括・ステップ実行
* **個別実行**: 各コマンド行の左端にある「実行」ボタン（ActiveX ボタン）をクリックすれば、その行のコマンドだけを単体テストできます。
* **一括連続実行（Start）／ 1ステップ実行（Step）**: 「Startボタン配置」で生成される **4 列制御ブロック（`Start` ボタン・`Step` ボタン・`開始位置`・`現在位置`）** により、指定した開始位置からの連続自動実行、1 行ずつのステップデバッグ実行、および中断箇所（`現在位置`）からの途中再開を行えます。

### 6. 取得データのリアルタイム・自動セル展開
* 機器からの取得データやレジスタ読出値は、シーケンス行の結果セル（Result列）へリアルタイムに書き込まれます。
* 複数データや二次元配列データ（連続レジスタ読出値やタイムスタンプ付きログデータなど）は、指定セルを起点として縦横のセル範囲へ自動的に展開されます。

### 7. セル装飾・スタイルの自動保存＆テンプレート化
* セルの背景色、罫線（太さ・線種・色）、フォント、列幅、配置などを好みにデザインしたあと、「スタイル保存」ボタンを押すだけでテンプレートとして設定ファイルへ保存されます。
* 次回以降の「シーケンス配置」時には、保存されたスタイルが自動適用されます。

### 8. 完全なファイル互換性（ライセンス無PC作成Excelのシームレス動作）
* ライセンスの無いPC（評価版環境や出張先・テレワーク用ノートPC等）で作成・編集したExcelファイル（シーケンスシート）であっても、 **ライセンスの有るPC（本番検査PCや製造ライン等）で開けば、一切の制限なくそのまま問題なく動作・実機自動通信が可能** です。
* シーケンスデータは標準のExcelセル情報として保持されるため、ライセンスの有無によるファイルロックや破損等の心配がなく、「自席のノートPCでシーケンスを作成し、検査ラインのライセンスPCで本番実行する」という柔軟な分業・運用を実現できます。

### 9. 多言語UI対応（日本語 / 英語 / 自動判別）
* メイン画面のメニュー（`[設定]` ➔ `[設定...]`）から、画面表示言語を **「日本語」「英語」「自動判別（OS設定準拠）」** から自由に切り替え可能です。
* 言語を切り替えると、メイン画面・コマンド編集画面・プロパティ設定画面・Excelリボンメニューの表示ラベルがリアルタイムに即座に更新され、設定は次回起動時も保持されます。

### 10. 通常 Excel ファイルの高速オープン・オーバーヘッド防止（除外キャッシュ）
* `.csv` やテキスト形式等の非ブック形式は走査を一切行わず **0ms で即座にスキップ** します。
* 本アドインの標準対応形式である `.xlsx`（マクロ無効形式）や `.xlsm` であっても、シーケンスボタンを持たない無関係な通常ファイルは共通除外キャッシュ（`%USERPROFILE%\Desktop\Monaca\IgnoredWorkbooks.txt`）に自動登録され、2回目以降のオープン時の走査を完全に防止します（キャッシュファイルを削除すればいつでも手動リセット可能）。

### 11. 設定変更履歴の自動記録（安心設計・トレーサビリティ保証）
* 誤った通信設定の早期発見や検査ラインでの安心運用のため、COM ポート条件や IP アドレス等の環境設定を変更して保存した際、変更前後の内容が自動的に監査ログファイルへ記録されます。
* ログの保存先フォルダやファイル名は、メイン画面のメニュー（`[設定]` ➔ `[設定エラー履歴...]`）から自由に変更・確認が可能です（既定はデスクトップの `Monaca\info.log`）。

### 12. ドライバ不要の仮想通信中継 ＆ 仮想デバイス・PLC エミュレータ連携（`c2c` / `TermEmulator` 対応）
* 実機のシリアル機器や RS-232C / RS-485 変換ケーブルが手元にない環境でも、公式無償コンパニオンツール **[c2c](https://github.com/Monaca-App/c2c)** を起動するだけで、カーネルドライバの導入や管理者権限なしに仮想 COM ポートペア（例: `COM10` ⇔ `COM11`、8ピン仮想結線対応）や Ethernet 中継を利用できます。
* さらに総合エミュレータ **[TermEmulator](https://github.com/Monaca-App/TermEmulator)** を併用すれば、SCPI 計測器や 6 種産業用プロトコル（Modbus / MELSEC 計算機リンク / MELSEC MC 3E / Telnet / OMRON FINS・HostLink / KEYENCE KV）の仮想スレーブ自動応答、共有デバイスメモリ監視、通信ログからの自動モック再現、および遅延・無応答などの異常系テストまで PC 1 台で実行できます（お持ちの `TermAddIn_*.License` をそのまま相互共用可能）。

---

## 📊 シート上のシーケンス構成

アドインで作成されるシーケンス行は、シートの特定列（A列など）に固定されるのではなく、 **選択したセル（指定セル）を先頭として、その行に横方向へ連続配置** されます（縦2行分で1つのシーケンスコマンド行を構成）。

| セル位置 | 先頭セル（基準） | +1列 | +2列 | +3列 | +4列 | +5列以降 |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **役割** | **Label (実行)** | **Wait (待機)** | **Command** | **Result (結果)** | **Param 1** | **Param 2...** |
| **上段セル**<br>*(項目名/Help)* | — | 待機時間(ms) | コマンド名 | 取得結果 | パラメータ名 | パラメータ名 |
| **下段セル**<br>*(設定値/ボタン)* | **[ 実行 ]**<br>*(ActiveX)* | `500` | `NUM READ` | `0001` | `F90000FF00` | `04010000ZR` |

<br>

| 相対位置 | 項目 | 役割・動作 |
| :---: | :--- | :--- |
| **先頭セル（基準）** | **Label (実行ボタン)** | クリックすることで、その行に記述されたコマンドを単発実行します。 |
| **+1列** | **Wait (待機時間)** | コマンド実行前のウェイト時間（ミリ秒）を指定します。機器の応答安定待ちに利用します。 |
| **+2列** | **Command (コマンド)** | 実行対象の制御コマンド名（マクロ定義名または通信コマンド）を指定します。 |
| **+3列** | **Result (結果出力)** | 機器から返却されたレスポンスやレジスタ読出値の出力先（セル番地または名前定義）を指定します。 |
| **+4列以降** | **Parameter (パラメータ)** | コマンドに渡す設定値（送信電文、レジスタアドレス、点数等）を入力・選択します。 |

### 💡 Result（結果出力セル）の仕様と運用上の重要な注意点

コマンド実行によって機器から返却された応答データや読出値は、 **+3列目の下段セル（Result設定セル）に指定されたターゲット位置へ自動的に書き込まれます** 。運用にあたっては以下の動作仕様と注意点をご確認ください。

1. **出力先リストの自動表示（データ出力コマンド時）**:
   * コマンド定義に結果出力項目が設定されている場合、シーケンス配置時にブック内の名前定義をスキャンし、 **出力先候補となる名前付きセルの選択ダイアログ** が自動表示されます。
2. **未選択時の動作（セル番地自動設定）**:
   * ダイアログでリストから何も選択せずに [OK]（または [キャンセル]）をクリックした場合、 **そのシーケンス行の結果セル自身のアドレス（例: `D10` など）** が下段セルに自動設定されます。
3. **名前付きセルの事前準備（推奨）**:
   * Excel の標準機能（[数式] ➔ [名前の定義]）を使って測定値の記録先セルに名前（例: `レジスタ読出値_D100` 等）をあらかじめ定義しておくと、配置時のリスト上に表示され、クリックするだけで出力先として紐付けられます。
4. **後からの手動直接入力・変更も可能**:
   * シーケンスを配置した後でも、下段セル（Resultセル）の値は Excel 上で直接キーボードから編集可能です。
5. **⚠️ セル名（番地文字列）運用における注意点（名前付きセルの推奨理由）**:
   * 下段セルに `"B10"` のような通常のセル番地（文字列）を設定している場合、 **シート上で行や列の挿入・削除を行うと、セル内の文字情報自体は自動変更されず `"B10"` のまま残るため、本来意図していた出力先から位置がズレてしまう** 現象が発生します。
   * **「名前付きセル」で出力先を指定しておけば、シート上で行や列の挿入・削除・移動が行われても Excel が自動的にセルの位置を追従するため、取得データが常に正しいセルへ確実に記録されます。長期的な検査シートの保守や行追加を想定する場合は、「名前付きセル」による運用を強く推奨します。**

---

## 🛠️ リボンメニュー機能

Excel 上部に表示される専用リボンタブ（「アドイン」タブ）内の **「ツール」** および **「Term制御」** グループから、以下の機能をワンクリックで呼び出せます（`GpibAddIn` 等と同時導入時は、アドインタブ内に「ツール」「GPIB制御」「Term制御」の各グループが並んで表示されます）。

| グループ | ボタン名 | 機能説明 |
| :--- | :--- | :--- |
| **ツール (`tool`)** | **スタイル保存 (`Save_Style`)** | 現在選択されている行のセル装飾（背景色、罫線、フォント、列幅等）を読み取り、今後の新規作成時の既定テンプレートとして保存します。 |
| **ツール (`tool`)** | **Startボタン配置 (`Start`)** | シート上に **4 列制御ブロック（`Start` 一括実行ボタン・`Step` 1行実行ボタン・`開始位置` 選択セル・`現在位置` 表示セル）** を配置します。 |
| **ツール (`tool`)** | **シーケンス削除 (`Delete_Sequence`)** | 選択中のシーケンス行（2行分）または Start 制御ブロック行を、Excel内部の連携を安全に解除しながら削除します。 |
| **Term制御 (`Term`)** | **ツール起動 (`ToolOpen`)** | タスク番号選択ダイアログ（`P1`〜`P10` の最大 10 タスク）で対象タスクを選び、対応するメイン操作画面を表示します。接続状態の確認や送受信ログのリアルタイム監視が可能です。 |
| **Term制御 (`Term`)** | **シーケンス配置 (`Make_Sequence`)** | タスク番号（`P1`〜`P10`）を選択後、コマンド編集画面がモーダル表示され、選択・設定したコマンドと初期パラメータをシート基準列へ自動整列生成します（既存シーケンス行では確認後に直下行へ自動挿入）。 |

---

## 🖥️ メイン画面機能（7モード切替・ログ管理・タイムスタンプ）

Excel リボンメニューの **「ツール起動 (`ToolOpen`)」** ボタンから呼び出せるメイン画面から、対話的な通信テストやログ記録、動作モード切り替えを直接操作できます。

### 1. 通信モード切替（全 7 通信モード動的連携）
* メイン画面メニューの **`[通信モード]`** から、有効ライセンスに応じた **全 7 モード（`RS-232C` / `TELNET` / `MODBUS` / `MELSEC` / `MELSEC MC` / `OMRON FINS` / `KEYENCE KV`）** をワンクリックで切り替えられます。
* マクロスクリプト実行時は、スクリプト内のオープンコマンド（`Com_Open`, `Telnet_Open`, `Modbus_Open`, `Melsec_Open`, `Mc_Open`, `Fins_Open`, `Kv_Open`）によって該当プロトコルでの通信セッションが開始されます。

### 2. 接続時刻表示（通信ログ タイムスタンプ付加）
* メニューの **[時刻表示] ➔ [時刻有効] / [時刻無効]** から、送受信データログの各行の先頭にミリ秒精度のタイムスタンプ（`yyyy/MM/dd HH:mm:ss.fff`）を付加してリアルタイム表示・記録するかどうかを切り替えることができます。

### 3. バージョン情報確認（オフライン完全対応）
* メニューの **[ヘルプ] ➔ [バージョン情報...]** を開くことで、現在稼働しているアプリケーションの正式バージョン番号、著作権情報、および動作環境（.NET Framework / OS 情報）をダイアログで確認できます。外部インターネット接続を一切行わないため、工場や検査ラインのオフラインPCでも安心してご利用いただけます。

---

## 💻 動作環境・システム要件

| 項目 | 要件 |
| :--- | :--- |
| **OS** | Windows 10 / Windows 11 (64bit / 32bit) |
| **対応 Excel** | Microsoft Excel 2016 / 2019 / 2021 / Microsoft 365 (デスクトップ版 32bit / 64bit) |
| **ランタイム** | .NET Framework 4.8<br>Visual Studio 2010 Tools for Office Runtime (VSTO ランタイム) |
| **対応インターフェース** | ・RS-232C / RS-422 / RS-485 シリアルポート（オンボード COM ポートおよび USB-シリアル変換器）<br>・Ethernet (TCP/IP・UDP) ネットワークインターフェース（Telnet / Modbus TCP / MELSEC MC）<br>・仮想ヌルモデム・ブリッジポート（公式無償ツール [c2c](https://github.com/Monaca-App/c2c) によるドライバ不要の仮想 COM ペア対応） |
| **ディスプレイ** | 推奨解像度: **Full HD (1920 × 1080) 以上** （最小動作解像度: 1280 × 720）<br>対応スケーリング: **100%、125%、150%、175%、200%** （Per-Monitor DPI V2 対応） |

### ⚙️ Excel 32bit / 64bit 環境における動作対応マトリクス

本アドインは `.NET Framework 4.8` のマネージド通信基盤（`System.IO.Ports` / `System.Net.Sockets`）かつ `AnyCPU` でビルドされており、 **Excel 32bit 版および 64bit 版の双方で、すべての通信プロトコルが制限なく完全動作** します。

| 通信プロトコル / ドメイン | Excel 32bit 版 | Excel 64bit 版 | 詳細・備考 |
| :--- | :---: | :---: | :--- |
| **[COM シリアル通信（`Comm`）](https://github.com/Monaca-App/TermAddIn_Comm)** | **◯ 完全動作** | **◯ 完全動作** | OS 標準の COM ポートおよび [c2c](https://github.com/Monaca-App/c2c) 仮想ポートを介して 32bit / 64bit 問わず完全動作します。 |
| **[Telnet ソケット通信（`Telnet`）](https://github.com/Monaca-App/TermAddIn_Telnet)** | **◯ 完全動作** | **◯ 完全動作** | OS 標準の TCP/IP ソケットを介して 32bit / 64bit 問わず完全動作します。 |
| **[Modbus 通信（`Modbus`）](https://github.com/Monaca-App/TermAddIn_Modbus)** | **◯ 完全動作** | **◯ 完全動作** | Modbus RTU / ASCII（物理 COM・[c2c](https://github.com/Monaca-App/c2c) 仮想 COM）および TCP の全モードで完全動作します。 |
| **[MELSEC 計算機リンク（`Melco`）](https://github.com/Monaca-App/TermAddIn_Melco)** | **◯ 完全動作** | **◯ 完全動作** | 形式 1 〜 形式 5 ともに物理 COM・[c2c](https://github.com/Monaca-App/c2c) 仮想 COM 上で完全動作します。 |
| **[MELSEC MCプロトコル（`MelcoMC`）](https://github.com/Monaca-App/TermAddIn_MelcoMC)** | **◯ 完全動作** | **◯ 完全動作** | MC プロトコル QnA 互換 3E フレーム（TCP/IP バイナリ交信）にて 32bit / 64bit 問わず完全動作します。 |
| **[OMRON FINS / Sysmac（`Omron`）](https://github.com/Monaca-App/TermAddIn_Omron)** | **◯ 完全動作** | **◯ 完全動作** | FINS/UDP・FINS/TCP およびシリアル HostLink（物理 COM・[c2c](https://github.com/Monaca-App/c2c) 仮想 COM）ともに完全動作します。 |
| **[KEYENCE KV 上位リンク（`Keyence`）](https://github.com/Monaca-App/TermAddIn_Keyence)** | **◯ 完全動作** | **◯ 完全動作** | Ethernet TCP/UDP（Port 8501）および RS-232C シリアル上位リンクともに完全動作します。 |

### 🖥️ 高解像度（高DPI）モニタ環境における推奨事項・制限事項

* **UIレイアウトの最小サイズ制約**:
  * メイン画面および各種設定ダイアログは、高DPIスケーリング環境下でも各操作ボタンやステータスの視認性を確保するため、最小ウィンドウ幅を **`500px × DPIスケール比率`** に制限しています。拡大率は **100%〜200%** の範囲でのご利用を強く推奨します。
* **異なる拡大率のマルチモニタ環境での注意事項**:
  * スケーリング倍率の異なる複数ディスプレイ間を跨いでウィンドウをドラッグ移動した場合、Windows および Office の再描画タイミングにより、一時的な表示のちらつきや ActiveX ボタン描画にわずかなズレが生じる場合があります。その場合はウィンドウを一度最大化/元に戻すか、Excel の「互換性を優先する」表示設定をご利用ください。

---

## 🚀 インストール手順

<a href="https://github.com/Monaca-App/TermAddIn/raw/main/Setup/TermAddIn_Setup.zip">
  <img src="images/Monaca_app_get.jpg" width="140" alt="プログラム（入手）" />
</a>

> 上記の **「プログラム（入手）」アイコン** をクリックすると、必要なセットアッププログラム一式（[`TermAddIn_Setup.zip`](https://github.com/Monaca-App/TermAddIn/raw/main/Setup/TermAddIn_Setup.zip)：`Term/` および `TermAddIn/`）が直接ダウンロードされます。ダウンロードした ZIP ファイルを展開して以下の順に実行してください。

本製品は、 **① 本体（Term 通信アプリ・共通基盤・マニュアル）** と **② Excel アドイン（VSTO アドイン本体・Excel 自動登録）** の 2 つの MSI インストーラーで構成されています。  
さらに、使用する通信プロトコル（`Comm` / `Telnet` / `Modbus` / `Melco` / `MelcoMC` / `Omron` / `Keyence`）のエクステンション インストーラーを追加導入することで機能が拡張されます。  
いずれも共通のインストール先フォルダ（`C:\Program Files (x86)\Monacaアプリ工房\Monaca Common\`）へ統合導入されます。

### インストーラー構成（`TermAddIn_Setup.zip` 内）

| 導入順 | フォルダ | インストーラーファイル | 役割 |
| :---: | :--- | :--- | :--- |
| **① 先に実行** | `Term/` | `setup.exe` / `Term.Setup.msi` | Term 本体アプリ（`Term.exe`）・共通通信基盤・HTML 取扱説明書を導入します。 |
| **② 次に実行** | `TermAddIn/` | `setup.exe` / `TermAddIn.Setup.msi` | Excel VSTO アドイン本体（`Monaca.Term.AddIn`）を導入し、Excel へ自動登録します。 |
| **③ 必要に応じて** | 各エクステンション ZIP (`Setup/*_Setup.zip`) | `TermAnalysis.*.Setup.msi` / `setup.exe` | 使用する通信プロトコル拡張 DLL とサンプル資材を追加導入します（[Comm](https://github.com/Monaca-App/TermAddIn_Comm) / [Telnet](https://github.com/Monaca-App/TermAddIn_Telnet) / [Modbus](https://github.com/Monaca-App/TermAddIn_Modbus) / [Melco](https://github.com/Monaca-App/TermAddIn_Melco) / [MelcoMC](https://github.com/Monaca-App/TermAddIn_MelcoMC) / [Omron](https://github.com/Monaca-App/TermAddIn_Omron) / [Keyence](https://github.com/Monaca-App/TermAddIn_Keyence)）。 |

### 導入ステップ

1. **① 本体（`Term.Setup`）のインストール**:
   * ダウンロードした **`TermAddIn_Setup.zip`** を展開し、 **`Term/`** フォルダ内の **`setup.exe`** （または **`Term.Setup.msi`** ）を実行します。
   * インストールウィザードの指示に従い、インストールを完了します（既定のインストール先: `C:\Program Files (x86)\Monacaアプリ工房\Monaca Common\`）。
   * ※ 管理者権限（UAC 承認）が必要です。必要なランタイム（.NET Framework 4.8 等）が未導入の場合は、`setup.exe` から自動的に導入案内が表示されます。
2. **② Excel アドイン（`TermAddIn.Setup`）のインストール**:
   * 続いて **`TermAddIn/`** フォルダ内の **`setup.exe`** （または **`TermAddIn.Setup.msi`** ）を実行します。
   * ※ インストール時は **Microsoft Excel をすべて終了** しておいてください。
   * 完了すると、Excel VSTO アドインが自動的にレジストリ登録され、デスクトップの **`Monaca\Manual`** フォルダへ取扱説明書へのショートカットが作成されます。
3. **③ 通信プロトコル拡張（`TermAnalysis.*.Setup`）のインストール**:
   * 利用したい通信プロトコルの各エクステンション リポジトリから `TermAddIn_<Ext>_Setup.zip` をダウンロード・展開し、`TermAnalysis.<Ext>/` 内の **`setup.exe`** （または **`TermAnalysis.<Ext>.Setup.msi`** ）を実行します。
   * 拡張 DLL が共通フォルダ（`Monaca Common`）へ配置され、動作確認用サンプル資材が `Monaca Sample\<Ext>\` へ導入されます。
4. **動作確認**:
   * **Microsoft Excel** を起動し、上部リボンの **「アドイン」タブ** 内に **「ツール」** および **「Term制御」** グループが追加されていることを確認します。

> [!TIP]
> **スタンドアロン運用（Excel なし）について**  
> Excel アドインを使用せず、単体アプリケーション（`Term.exe`）のみを利用する場合は、 **① `Term/`** と必要な **③ 通信プロトコル拡張** のインストールで動作します。

### ⚠️ インストール時に認証・セキュリティメッセージが表示される場合

#### 1. 「Windows によって PC が保護されました」（Microsoft Defender SmartScreen）
* **対処方法**:
  1. 画面内の **「詳細情報」** リンクをクリックします。
  2. 画面下部に表示される **「実行」** ボタンをクリックすると、インストーラーが起動します。

#### 2. ダウンロードしたファイルがブロックされている場合
* **対処方法**:
  1. ダウンロードしたファイル（`setup.exe` / `*.msi` / ZIP 等）を右クリックして **「プロパティ」** を開きます。
  2. 「全般」タブ下部のセキュリティ欄にある **「許可する」** （または「ブロックの解除」）にチェックを入れます。
  3. **「適用」** ➔ **「OK」** をクリック後、改めてインストーラーを実行してください。

### アンインストール手順
* Windows の「設定」>「アプリ」>「インストールされているアプリ」（または「コントロールパネル」の「プログラムと機能」）を開き、一覧から不要になったコンポーネントをアンインストールしてください。
  1. 各通信プロトコル拡張（ **`TermAnalysis.*.Setup`** ）
  2. **`TermAddIn.Setup`** （Excel アドインの登録解除・削除）
  3. **`Term.Setup`** （Term 本体アプリ・共通基盤の削除）
* ※ 他の Monaca アプリ工房製品（`GPIB` / `GpibAddIn` / `c2c` / `TermEmulator` 等）を併用している場合、共通基盤（`Monaca Common`）の共有ファイルは Windows Installer の参照カウント機能により自動的に保護されます。

---

## 🔰 クイックスタート (使い方)

> [!TIP]
> **サンプルスクリプトおよび実機なし折り返し・仮想機器テストで確認する場合**  
> * 各通信プロトコル拡張のインストーラーを実行すると、全 7 通信プロトコル拡張の動作確認用サンプルマクロ・AIプロンプト群（[COM シリアル通信 `Comm`](https://github.com/Monaca-App/TermAddIn_Comm) の `Comm.csv`、[Telnet 通信 `Telnet`](https://github.com/Monaca-App/TermAddIn_Telnet) の `Telnet.csv`、[Modbus 通信 `Modbus`](https://github.com/Monaca-App/TermAddIn_Modbus) の `Modbus_RTU.csv`・`Modbus_ASCII.csv`・`Modbus_TCP.csv`、[MELSEC 計算機リンク `Melco`](https://github.com/Monaca-App/TermAddIn_Melco) の `Melco_P1.csv`〜`Melco_P5.csv`、[MELSEC MC プロトコル `MelcoMC`](https://github.com/Monaca-App/TermAddIn_MelcoMC) の `MelcoMC.csv`、[オムロン FINS `Omron`](https://github.com/Monaca-App/TermAddIn_Omron) の `Omron_FINS.csv`、[キーエンス KV `Keyence`](https://github.com/Monaca-App/TermAddIn_Keyence) の `Keyence_KV.csv`）が **`C:\Program Files (x86)\Monacaアプリ工房\Monaca Sample\`** 配下に配置され、デスクトップの `Monaca\Sample` ショートカットから参照できます。プロパティ画面から読み込ませることで、コマンド選択やシーケンス配置の動作イメージをすぐに確認できます。  
> * 実機シリアル機器や PLC が手元にない場合は、公式無償ツール **[c2c（仮想ヌルモデム＆通信モニター）](https://github.com/Monaca-App/c2c)** や仮想デバイス・PLC スレーブエミュレータ **[TermEmulator](https://github.com/Monaca-App/TermEmulator)** を併用するだけで、ドライバ導入不要で PC 単体通信テストを行えます。

### ステップ 1: Termマクロスクリプトの登録と通信設定
1. **メイン画面とプロパティ画面の起動**: Excel リボンメニューの **「ツール起動 (ToolOpen)」** をクリックし、操作対象のタスク番号（`P1`〜`P10`）を選択してメイン画面を開きます。続いてツールバーの **`[プロパティ]`** （またはメニュー）からプロパティ設定画面を開きます。
2. **スクリプトの登録**: **`ファイル` タブ** （コマンド定義ファイル管理）にて、使用する機器の Term マクロスクリプト（CSV定義ファイル、例: `Sample\TermAddIn\Comm\Comm.csv`、`Telnet\Telnet.csv`、`Modbus\Modbus_RTU.csv`、`Melco\Melco_P4.csv`、`MelcoMC\MelcoMC.csv`、`Omron\Omron_FINS.csv`、`Keyence\Keyence_KV.csv` 等）を登録・選択します。
3. **通信条件の設定**: 現在選択中の通信モード（全 7 モード）に応じて動的表示される専用設定タブ（ **`COM` タブ ／ `Telnet` タブ ／ `Modbus RTU`・`Modbus Socket` タブ ／ `MELSEC` タブ ／ `MELSEC MC` タブ ／ `OMRON FINS` タブ ／ `KEYENCE KV` タブ** ）を開き、COM ポート番号・ボーレートや接続先 IP アドレス・ポート番号を設定して **`[OK]`** をクリックします。

### ステップ 2: シーケンス行の追加
1. 新規または既存の Excel ワークシートを開きます。
2. シーケンスを配置したい開始セル（例: `A5` や任意のセル）を選択します。
3. リボンメニューの **「シーケンス配置 (Make_Sequence)」** をクリックします。
4. タスク番号（`P1`〜`P10`）を選択したのち **コマンド編集画面** がモーダル表示されます。実行したいコマンドと初期パラメータを選択して閉じると、選択したセルを先頭として「実行」ボタン（タスク識別子付き）、待機時間、コマンド名、結果、各パラメータ設定セルがその行に自動配置されます。

### ステップ 3: コマンドとパラメータの設定
1. **コマンドセル**: 実行したい通信コマンドを選択または入力します。
2. **パラメータセル**: 送信文字列やレジスタアドレス、点数などの設定値を入力またはドロップダウンから選択します。
3. **待機時間セル**: コマンド実行前にウェイトをかけたい場合は、数値をミリ秒（例: `500`）で指定します。

### ステップ 4: 単発実行テスト
* 行頭の **[ 実行 ]** ボタンをクリックします。
* コマンドが機器へ送信され、応答結果が **結果セル（Result）** に表示されます。

### ステップ 5: 一括・ステップ実行制御ブロックの配置と自動通信
1. シーケンスの上部など任意のセル（例: `A2` 付近）を選択し、リボンの **「Startボタン配置 (Start)」** をクリックします。
2. 選択セルを起点に **4 列制御ブロック（`Start` ボタン・`Step` ボタン・`開始位置` ドロップダウン・`現在位置` 表示セル）** が自動配置されます。`開始位置` セルで実行を開始したい行のボタン位置（例: `A5`）を選択します。
3. **`Start`** ボタンをクリックすると上から順に連続自動実行され、 **`Step`** ボタンをクリックすると 1 行ずつステップ実行して取得結果がシート上に記録されます。

---

## 📜 Termマクロスクリプト (CSV定義ファイル)

TermAddIn で実行される制御コマンドや通信シーケンスは、独自の階層構造を持つ **CSV 形式ファイル（Shift-JIS）** によって定義されています。  
新しい通信機器の追加や独自プロトコルシーケンスの作成も、CSV ファイルを記述・登録するだけでプログラムの再ビルドなしに Excel から利用可能になります。

### 1. 階層構造と構文ルール
解析エンジンは行頭（第1フィールド）の記述状態によって、親ブロックと子要素（サブコマンド）を自動判別します。

* **親ブロック（行頭から記述）**:
  * `TITLE, "[タイトル名称]"`: 対象システム・機器の名称宣言
  * `COMMAND, "[コマンド名]", "[結果格納先キー]"`: マクロコマンドの定義宣言
* **子要素・サブコマンド（行頭カンマ `,` から記述）**:
  * `,PARAM, [キー], [データ型], [表示名], [初期値/選択肢...]`: コマンド引数の定義（第5フィールドの表示名、第6フィールドの初期値は省略不可）
  * `,SEQUENCE, [ステップ], [コマンド名], [引数...]`: 実行する処理手順（`Com_Send`, `Com_WaitPrompt`, `Modbus_Read`, `Melsec_P4s_S`, `Set_Data`, `Judge` 等）

### 2. PARAM（パラメータ）のデータ型と Excel 連携
`PARAM` で指定されたデータ型に応じて、Excel シート上での UI 動作が自動連動します。

| データ型 | 挙動・Excel上の動作 |
| :--- | :--- |
| **`Text`** | 汎用テキスト入力。自由な文字列や送信電文を入力できます。 |
| **`List`** | ドロップダウンリスト。定義された選択肢が Excel の「データの入力規則」としてセルへ自動適用されます。 |
| **`Range`** | 結果展開用。取得したカンマ・改行区切りの配列データを、Excel セルへ縦横 2次元自動展開します。 |
| **`Value` / `Number`** | 数値・16進文字列検証付きの入力スタイル。 |
| **`Label`** | 結果出力先またはジャンプ先ラベルを指定するスタイル。 |
| **`Work`** | マクロ内部の一時メモリ・バッファとして利用される領域（Excel 上では入力セル非生成）。 |

### 3. 全130コマンド体系（Term 固有 7 ドメイン 67 コマンド ＋ 共通基盤 7 ドメイン 63 コマンド）
* **[COM ポート通信（6 コマンド）](https://github.com/Monaca-App/TermAddIn_Comm)**: `Com_Open`, `Com_Close`, `Com_Send`, `Com_SendFile`, `Com_WaitPrompt`, `Com_Mode`
* **[Telnet 通信（6 コマンド）](https://github.com/Monaca-App/TermAddIn_Telnet)**: `Telnet_Open`, `Telnet_Close`, `Telnet_Send`, `Telnet_SendFile`, `Telnet_WaitPrompt`, `Telnet_Mode`
* **[Modbus 通信（5 コマンド）](https://github.com/Monaca-App/TermAddIn_Modbus)**: `Modbus_Open`, `Modbus_Close`, `Modbus_Read`, `Modbus_Write`, `Modbus_Mode`
* **[MELSEC 計算機リンク（23 コマンド）](https://github.com/Monaca-App/TermAddIn_Melco)**: `Melsec_ReadWord`, `Melsec_WriteWord`, `Melsec_ReadBit`, `Melsec_WriteBit`, `Melsec_Loopback`, `Melsec_Open`, `Melsec_Close`, `Melsec_Mode`, `Melsec_P1s_R`, `Melsec_P1s_S`, `Melsec_P2s_R`, `Melsec_P2s_S`, `Melsec_P3s_R`, `Melsec_P3s_S`, `Melsec_P4s_R`, `Melsec_P4s_S`, `Melsec_P5s_R`, `Melsec_P5s_S`, `Melsec_Res_P1s_R`, `Melsec_Res_P2s_R`, `Melsec_Res_P3s_R`, `Melsec_Res_P4s_R`, `Melsec_Res_P5s_R`
* **[MELSEC MCプロトコル通信（9 コマンド）](https://github.com/Monaca-App/TermAddIn_MelcoMC)**: `Mc_Open`, `Mc_Close`, `Mc_Mode`, `Mc_ReadWord`, `Mc_WriteWord`, `Mc_ReadBit`, `Mc_WriteBit`, `Mc_Send`, `Mc_Receive`
* **[オムロン FINS / Sysmac 通信（9 コマンド）](https://github.com/Monaca-App/TermAddIn_Omron)**: `Fins_Open`, `Fins_Close`, `Fins_Mode`, `Fins_ReadWord`, `Fins_WriteWord`, `Fins_ReadBit`, `Fins_WriteBit`, `Fins_Send`, `Fins_Receive`
* **[キーエンス KV 上位リンク通信（9 コマンド）](https://github.com/Monaca-App/TermAddIn_Keyence)**: `Kv_Open`, `Kv_Close`, `Kv_Mode`, `Kv_ReadWord`, `Kv_WriteWord`, `Kv_ReadBit`, `Kv_WriteBit`, `Kv_Send`, `Kv_Receive`
* **共通基盤 Core（全 63 コマンド / 7 ドメイン）**: パラメータ・変数演算（`Param`：12 件）、配列操作（`Array`：7 件）、シーケンス・呼出・ループ・フロー・エラートラップ制御（`Sequence`：22 件）、Excel シート順序制御（`Excel`：8 件）、ファイル入出力（`File`：5 件）、受信ワークバッファ抽出・タイムスタンプ（`Work`：5 件）、透過ポート通信制御（`Port`：4 件）

各コマンドの詳細仕様は、[マクロスクリプト取扱説明書](./Manual/マクロスクリプト取扱説明書.Md) および [Excelアドイン取扱説明書](./Manual/Excelアドイン取扱説明書.Md) をご参照ください。

---

## ⚙️ 設定ファイル (`Setting.xml`)

アドインのスタイル設定や通信設定は、以下の設定ファイルにて一元管理されています。

* **設定ファイルパス**: `%USERPROFILE%\Desktop\Monaca\Setting.xml`（`<USER_DIR>\Desktop\Monaca\Setting.xml`）
  * ※ デスクトップ上の `Monaca` フォルダがデフォルトの配置先となります。
  * ※ 購入後にメール（送信元: `sales@monaca-app-store.com`）で届く各エクステンションのライセンスファイル（`TermAddIn_Comm.License` 等）も本設定ファイルと同じフォルダ（`%USERPROFILE%\Desktop\Monaca\`）に配置します。
* **主な設定項目**:
  * `Common` テーブル `Styles` セクション: 各セルの背景色、罫線スタイル、文字配置、列幅等の共通装飾テンプレート情報（全アドイン共通）
  * `Common` テーブル `System` (`Language`): UI 表示言語コード（`""` = 自動判別、`"ja-JP"` = 日本語、`"en-US"` = 英語）
  * `Properties` / `ModeSetting` / `Command`: 登録されたマクロスクリプト（CSV）と現在選択中の通信モード構成情報
  * `Comm` / `Telnet` / `ModbusRTU` / `ModbusSocket` / `MELSEC` / `MELSEC_MC` / `OMRON_FINS` / `KEYENCE_KV`: 各プロトコル専用タブの COM ポート番号・ボーレート、接続先 IP アドレス・ポート番号、伝送モード・局番等の通信パラメータ
  * `Log`: 通信ログ・監査ログの出力先ディレクトリおよびログレベル（`Save`, `Info`, `Error` 等）
* **共通除外キャッシュファイル (`IgnoredWorkbooks.txt`)**:
  * `Setting.xml` と同じディレクトリに全アドイン共通で自動生成され、シーケンスボタンを持たない通常 Excel ファイルのパスを保持して起動オーバーヘッドを防止します（ファイルを削除することで手動キャッシュクリアが可能）。

---

## 💳 ライセンス・購入プラン（エクステンション別個別ライセンス）

> [!NOTE]
> **試用動作およびエクステンション毎の個別ライセンス体系について**  
> 日付指定による無料評価期間は廃止しております。ライセンスファイルがない場合、または無効な場合は以下の試用モードで全通信プロトコルを動作確認できます。  
> * **Excel アドイン（TermAddIn）**: ブック再オープン時、すでにシート上に記載されているボタンは無効化されます（オープン後に新規追加したボタンのみ動作確認可能です）。  
> * **スタンドアロン起動（Term.exe）**: **3 分間の期間限定版** として動作し、3 分経過後に通知を表示して通信を停止・操作をロックします。  
> TermAddIn では、必要な通信プロトコル（機能ドメイン）のみを柔軟に選択してご利用いただけるよう、 **エクステンション毎に個別のライセンスを設ける体系** を採用しております。

用途や利用期間に合わせて、必要なエクステンション毎に一括購入（買い切り）またはサブスクリプション（月額・週額）からお選びいただけます。  
メールアドレスの誤入力によるライセンス不着を防ぐため、 **事前のメールアドレス確認** を行った上で、 **Stripe** による安全なオンライン決済へご案内いたします（表記価格はすべて税別です）。

### 📦 対象エクステンションおよびライセンスファイル一覧

| エクステンション名 | 対象通信プロトコル | 対応ライセンスファイル | 価格プラン（一括 / 月額 / 週額・税別） | お申込み |
| :--- | :--- | :--- | :---: | :---: |
| **[TermAddIn_Comm](https://github.com/Monaca-App/TermAddIn_Comm)** | COM ポート・シリアル通信 (RS-232C / RS-422 / RS-485 / UART) | `TermAddIn_Comm.License` | **¥32,000** / ¥3,000 / ¥1,000 | [👉 購入手続きへ](https://stripe-license-generator.y-nakai-5a3.workers.dev/?target=comm) |
| **[TermAddIn_Telnet](https://github.com/Monaca-App/TermAddIn_Telnet)** | TCP/IP Telnet 端末通信 (ネットワーク機器・Linux 対話) | `TermAddIn_Telnet.License` | **¥32,000** / ¥3,000 / ¥1,000 | [👉 購入手続きへ](https://stripe-license-generator.y-nakai-5a3.workers.dev/?target=telnet) |
| **[TermAddIn_Modbus](https://github.com/Monaca-App/TermAddIn_Modbus)** | 産業用 Modbus 通信 (Modbus RTU / Modbus ASCII / Modbus TCP) | `TermAddIn_Modbus.License` | **¥49,000** / ¥4,400 / ¥1,500 | [👉 購入手続きへ](https://stripe-license-generator.y-nakai-5a3.workers.dev/?target=modbus) |
| **[TermAddIn_Melco](https://github.com/Monaca-App/TermAddIn_Melco)** | 三菱電機 PLC（MELSEC）計算機リンク通信 (シリアル/Ethernet 形式1〜5) | `TermAddIn_Melco.License` | **¥32,000** / ¥3,000 / ¥1,000 | [👉 購入手続きへ](https://stripe-license-generator.y-nakai-5a3.workers.dev/?target=melco) |
| **[TermAddIn_MelcoMC](https://github.com/Monaca-App/TermAddIn_MelcoMC)** | 三菱電機 PLC（MELSEC）MC プロトコル通信 (Ethernet 3Eフレーム) | `TermAddIn_MelcoMC.License` | **¥32,000** / ¥3,000 / ¥1,000 | [👉 購入手続きへ](https://stripe-license-generator.y-nakai-5a3.workers.dev/?target=melcomc) |
| **[TermAddIn_Omron](https://github.com/Monaca-App/TermAddIn_Omron)** | オムロン PLC（SYSMAC）FINS / HostLink 通信 (UDP / TCP / Serial) | `TermAddIn_Omron.License` | **¥49,000** / ¥4,400 / ¥1,500 | [👉 購入手続きへ](https://stripe-license-generator.y-nakai-5a3.workers.dev/?target=omron) |
| **[TermAddIn_Keyence](https://github.com/Monaca-App/TermAddIn_Keyence)** | キーエンス PLC（KV シリーズ）上位リンク通信 (Ethernet / Serial) | `TermAddIn_Keyence.License` | **¥49,000** / ¥4,400 / ¥1,500 | [👉 購入手続きへ](https://stripe-license-generator.y-nakai-5a3.workers.dev/?target=keyence) |

### 📋 ご購入・ライセンス発行の流れ
1. **メールアドレス確認**: 各エクステンションのリンクから、ライセンスファイルを受信するメールアドレスを入力して送信します。
2. **決済案内メールの受信**: 入力したアドレス宛てに、送信用メールアドレス **「sales@monaca-app-store.com」** より決済リンクのご案内メールが即時届きます。
3. **Stripe 決済**: 届いたメール内のご希望プランのボタンをクリックし、カード情報とともに、ご利用PCの **「PC名」** および **「ユーザー名」** を入力して決済を完了します。
4. **ライセンス自動配信**: 決済完了後、暗号署名入りのエクステンション別ライセンスファイル（例: `TermAddIn_Comm.License` 等）が即時メール添付配信されます。
5. **ライセンス配置**: 届いたライセンスファイルを設定ファイルと同じフォルダ（`%USERPROFILE%\Desktop\Monaca\`）に配置することで、該当エクステンションの制限が解除されます。

* ※ **【迷惑メールに関するご注意】**: 送信用メールアドレス（`sales@monaca-app-store.com`）からのご案内メールが届かない場合は、迷惑メールフォルダやゴミ箱をご確認いただくか、事前に受信許可（ホワイトリスト登録等）していただくようお願いいたします。
* ※ サブスクリプションプランは自動更新しません。更新時は再度申し込んでください。
* ※ **ライセンスの無いPC（評価版・ライセンス未適用環境）での制限事項**: Excelを開いた時点で、 **すでにシート上に記載されている既存シーケンスの動作確認（実行）は行えません** （動作確認できるのは、 **Excelをオープンした以降に追加したシーケンスのみ** となります）。
* ※ **ライセンスの有るPCでの動作およびファイル互換性**: 正式ライセンスがあるPCでは、保存された既存シーケンスも含めて制限なく即座に全自動実行できます。また、ライセンスの無いPC（評価版環境やノートPC等）で作成したExcelファイルであっても、ライセンスの有るPC（本番運用PC）で開けば何ら問題なくそのまま自動通信・測定を実行できます。

### ⚠️ ライセンス形態とご購入時（お支払い画面）の必須入力事項

> [!IMPORTANT]
> **1 ユーザー / 1 PC（ノードロック）ライセンスです**  
> 本アドインのライセンスは、購入時に登録された **「PC名（PC_Name）」** および **「ログインユーザー名（User_Name）」** と暗号署名によって厳密に紐づけられます。  
> 決済画面で入力した **`PC_Name` や `User_Name` を間違えると、発行されたライセンスファイルが実際の PC 環境と一致せず認証（アクティベーション）できません** 。必ず実際にご利用になる PC の正確な情報を確認の上、お支払い手続きを行ってください。

#### 🔍 PC_Name および User_Name の確認方法

お支払い画面（Stripe）のカスタムフィールドに入力する `PC_Name` と `User_Name` は、以下の手順でご確認いただけます。 **大文字・小文字やスペルミスを防ぐため、コマンドによる確認を推奨いたします。**

##### 方法1：コマンドプロンプト / PowerShell で確認（推奨・最も確実）
1. キーボードの `Windows キー` を押し、`cmd`（または `powershell`）と入力して起動します。
2. 以下のコマンドを入力して `Enter` キーを押します：
   ```cmd
   whoami
   ```
3. 画面に **`PC名\ユーザー名`** （例: `DESKTOP-ABC1234\taro`）の形式で出力されます：
   * **`\` の手前** ➔ **`PC_Name`** （例: `DESKTOP-ABC1234`）
   * **`\` の後ろ** ➔ **`User_Name`** （例: `taro`）

> [!TIP]
> 個別に確認・コピーしたい場合は、以下のコマンドも利用可能です：
> * PC名のみ確認: `echo %COMPUTERNAME%`（または `hostname`）
> * ユーザー名のみ確認: `echo %USERNAME%`

##### 方法2：Windows の「設定」画面から確認
* **`PC_Name`**: 「スタート」ボタンを右クリック ➔「システム」を選択（または「設定」➔「システム」➔「バージョン情報」）し、画面に表示される **「デバイス名」** をご確認ください。
* **`User_Name`**: 「スタート」メニューを開いたときに表示される現在のアカウント名です。  
  *(※ Microsoft アカウントと連携している場合、表示名と Windows 内部の実際のログインユーザー名が異なる場合があります。そのため方法1の `whoami` による確認を強く推奨します)*

### 🔑 各エクステンション用ライセンスファイル（`TermAddIn_<Ext>.License`）の配置・適用手順

ご購入手続き完了後、送信元 **「sales@monaca-app-store.com」** より対象エクステンションのライセンスファイル（例: `TermAddIn_Comm.License` 等）が添付されたメールが届きます。

> [!WARNING]
> **迷惑メールフォルダへの振り分けにご注意ください**  
> メールシステムやセキュリティ対策ソフトの判定により、ライセンス配信メールが「迷惑メール」として分類されたりブロックされたりするケースがございます。  
> 決済完了後すぐにメールが届かない場合は、 **必ず「迷惑メールフォルダ」や「ごみ箱」をご確認ください** 。また、確実な受信のために、あらかじめ **`sales@monaca-app-store.com`** を受信許可アドレス（セーフリスト・ホワイトリスト）に追加いただくようお願いいたします。

届いたライセンスファイルを、本アドインの **設定ファイルパス（`Setting.xml` と同じフォルダ）** に配置してください。

1. **ライセンスファイルの配置先**:
   * **配置パス**: `%USERPROFILE%\Desktop\Monaca\TermAddIn_<Ext>.License`  
     （例: `TermAddIn_Comm.License`, `TermAddIn_Telnet.License`, `TermAddIn_Modbus.License`, `TermAddIn_Melco.License`, `TermAddIn_MelcoMC.License`, `TermAddIn_Omron.License`, `TermAddIn_Keyence.License`）  
     （エクスプローラーのアドレスバーに `%USERPROFILE%\Desktop\Monaca` と入力して開くか、デスクトップ上の `Monaca` フォルダを開きます）
   * ※ インストール時に配置パスを変更した場合は、そちらに配置してください。
2. **ライセンスの有効化（アクティベーション）**:
   * ライセンスファイルを配置後、Excel を起動（または対象のワークブックを開き直す）すると、該当エクステンションのライセンスが自動的に認識・認証され、メニューやプロパティタブ、コマンドの制限が解除されます。
   * 有効期限内であれば制限なく対象エクステンションの全シーケンス機能をご利用いただけます。

---

## 📄 ライセンス & サポート

* **開発元**: [Monaca アプリ工房 (Monaca App Store)](https://github.com/Monaca-App/store)
* **関連コンパニオン・シミュレータツール**: [c2c - ドライバ不要の仮想ヌルモデム＆通信モニター](https://github.com/Monaca-App/c2c) ／ [TermEmulator - 汎用仮想デバイス＆PLCスレーブ・エミュレータ](https://github.com/Monaca-App/TermEmulator)
* **Copyright**: Copyright (C) Monaca App Store 2026 All rights reserved.
* **お問い合わせ・不具合報告**: GitHub の Issue までご連絡ください。

