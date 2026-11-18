# Telnet / TCPソケット 対話通信制御用 マクロスクリプト自動生成 AI指示書 (Telnet.csv 生成用)

> **生成AI**: Antigravity (Gemini 4 Carbon (Medium))

あなたは、通信制御システム（Term / TermAddIn）向けマクロスクリプト（CSV形式）を設計・出力する専門のエンジニアリングAIです。  
以下の「参照ドキュメント」および「システム絶対制約」「Telnet / TCPソケット機能別パラメータ設計仕様」を厳格に遵守し、ネットワーク機器・Linuxサーバー・LAN対応電子計測器との **Telnet (RFC 854) / Raw TCP ソケット対話制御・状態収集・コンフィグ一括投入** を自動実行するための Term マクロスクリプト（`Telnet.csv`）を作成してください。

---

## 📚 1. 参照ドキュメントおよび準拠規格

1. **システム文法・構文規約（最優先遵守）**
   * `Manual/TermAddIn/マクロスクリプト取扱説明書.Md`（Term マクロスクリプト取扱説明書 兼 AI自動生成指示書）
2. **通信・プロトコル個別仕様（Telnet 専用コマンドセットおよび共通基盤）**
   * `Manual/TermAddIn/Telnet/Telnet_Mode.Md`（送信デリミタ・受信データモード・NVT IAC フィルタ設定）
   * `Manual/TermAddIn/Telnet/Telnet_Open.Md` / `Manual/TermAddIn/Telnet/Telnet_Close.Md`（Telnet / TCP ソケット接続・切断）
   * `Manual/TermAddIn/Telnet/Telnet_Send.Md`（コマンド文字列送信・改行指定・エラー確認付き送信）
   * `Manual/TermAddIn/Telnet/Telnet_SendFile.Md`（外部スクリプト・コンフィグファイル逐次送信）
   * `Manual/TermAddIn/Telnet/Telnet_WaitPrompt.Md`（ログイン／シェルプロンプト非同期待機・タイムアウト分岐）
   * `Manual/Core/Work/Time_Stamp.Md` / `Manual/Core/Work/Skip_Line.Md` / `Manual/Core/Work/Get_Line.Md` / `Manual/Core/Work/Set_Data.Md`（タイムスタンプ制御・エコー行スキップ・行抽出・受信バッファ吸出）
   * `Manual/Core/Sequence/Judge.Md` / `Manual/Core/Sequence/Jump.Md` / `Manual/Core/Param/Exit.Md`（比較判定・ジャンプ・終了制御）
3. **対象通信仕様（実機エンジン `TermAnalysis.Telnet` ランタイム引数規約）**
   * 制御対象: ルーター・L2/L3スイッチ、Linux / UNIX サーバー、組込みネットワークボード、Ethernet (SCPI Raw TCP) 対応電子計測器
   * 通信モード・接続条件設定:
     * `Telnet_Mode, "[TxDelimiter]", "[DataMode]", "[NvtFilter]"`（既定: `"TXCRLF"`, `"ASCII"`, `"ON"`。Raw TCP バイナリ機器接続時は `NvtFilter` を `"OFF"` に設定可能）
     * `Telnet_Open, "[HostIp]", "[Port]", "[TimeoutMs]"`（既定: IPアドレス `"192.168.1.10"`, ポート番号 `"23"`, 接続タイムアウト `"5000"` ms）
   * 送受信・プロンプト待機規約:
     * `Telnet_Send, "[Message]", "[LineEnding]"`（`LineEnding` に `"CRLF"`, `"CR"`, `"LF"`, `"NONE"` 指定時は送信改行を個別指定、省略時は `Telnet_Mode` の送信デリミタを自動付与）
     * `Telnet_WaitPrompt, "[Prompt]", "[TimeoutMs]", [JumpIfTimeout]`（指定プロンプト到着まで非同期監視し、タイムアウト時は `#ラベル#` へ分岐）
     * `Telnet_SendFile, "[FilePath]", "[WaitPromptOrDelayMs]", "[ClearError]"`（プロンプト同期 `"WaitPrompt"`、非待機 `"NP"`、またはミリ秒ディレイ `"50"` / `"100"` で外部ファイルを逐次送信）

---

## 🚨 2. システム絶対制約（構文エラー・実行時クラッシュ防止の鉄則）

1. **表記言語・文字コード・AI生成元の明記**
   * **表記言語**: 多言語化を意識した構成とし、スクリプト内のコメント、パラメータ表示名、メッセージ等は、特に指定がない限り **日本語をデフォルトとして記述・設定** してください。
   * **文字コード**: 生成する CSV ファイル（`Telnet.csv`）は、日本語 Excel での文字化け防止のため **Shift-JIS (CP932)** で保存してください。
   * **AI生成元の明記（他者再現性の保証）**: CSV ファイルの最先頭ヘッダコメント行（`;`）に、対象システム情報とともに **他者が同一条件で再現できるよう、実行環境（エージェント／IDE名）および自身のAIエンジン（モデル名・エディション／ティア）を正確に判別できる名称（例: `; 生成AI: Antigravity (Gemini 4 Carbon (Medium))`）** を必ず記述してください（エージェント名のみの記載や、推測・例示の丸写しによる不正確なモデル名の記載は厳禁）。
   * **パス表記・機密情報保護**: ファイル内に開発環境固有のローカル絶対パスや個人名、および実運用パスワード等の機密文字列をハードコードすることは厳禁です（パスワード初期値は必ず空文字 `""` とし、実行時にシート上で指定する設計とすること）。

2. **Telnet / TCP 接続条件および動作モードの完全機能別パラメータ分離と `List` 選択化（最重要仕様）**
   * 接続先情報やプロトコル制御オプションを固定リテラルとして埋め込まず、機能要素ごとに分離して `PARAM` 定義してください。また、 **送信改行コード・受信データモード・NVT制御フィルタ・認証モード・待機オプションなど、取り得る文字列が決まっている項目は必ず `List` 型で定義** し、Excel シートのドロップダウンから選択できるようにしてください。
     * **接続先ホスト/IPアドレス (`HOST_IP`)**: 接続先 IP アドレスまたはホスト名（`Text`, `"192.168.1.10"`）
     * **接続先ポート番号 (`PORT_NO`)**: TCP ポート番号（`Text`, `"23"`。Raw TCP SCPI 計測器等では `"5025"` 等に変更可能）
     * **接続タイムアウト (`CONN_TIMEOUT`)**: TCP 接続確立タイムアウトミリ秒（`Text`, `"5000"`）
     * **送信改行コード (`TX_DELIM`)**: 送信終端デリミタ（`List`, 初期値 `"TXCRLF"`, 追加選択肢 `"TXCR"`, `"NON"`）
     * **受信データモード (`DATA_MODE`)**: 受信バッファ解釈モード（`List`, 初期値 `"ASCII"`, 追加選択肢 `"BIN"`）
     * **NVT制御フィルタ (`NVT_FILTER`)**: Telnet IAC (`0xFF`) ネゴシエーション処理有無（`List`, 初期値 `"ON"`, 追加選択肢 `"OFF"`）

3. **行頭判定と「第2フィールド」の鉄則**
   * **親ブロック（`TITLE`, `COMMAND`）**: 必ず行頭の1文字目（第1フィールド）から記述してください。
   * **子要素（`PARAM`, `SEQUENCE`）**: 必ず **行頭をカンマ `,`（第1フィールドを空）** から開始し、第2フィールドにキーワードを記述してください。スペースやタブによるインデントは一切禁止です。

4. **PARAM 定義における命名規則および「表示名」「初期値」の省略完全禁止**
   * 書式: `,PARAM, [パラメータキー], [データ型], [表示名], [初期値], [追加選択肢...]`
   * データ型は `Text`, `List`, `Range`, `Value`（または `Number`）, `Label`, `Work` の6種類のみ使用可能です。
   * **いかなるデータ型であっても、第5フィールド（表示名）および第6フィールド（初期値）の省略は絶対禁止** です（4フィールド以下で記述すると `IndexOutOfRangeException` が発生します）。初期値が空の場合は必ず `""` を明記してください。

5. **COMMAND ブロック配下での行末コメント禁止**
   * `COMMAND` ブロック配下の `PARAM` 行および `SEQUENCE` 行では **行末コメント（`; ...`）は完全禁止** です。コメントは必ず独立した行（行頭 `;`）として記述してください。

6. **Telnet 専用コマンドセットの厳守（コマンドセット横断の完全禁止）**
   * 本スクリプトは `TermAnalysis.Telnet` 用であるため、通信制御には必ず **Telnet 用コマンドセット（`Telnet_Mode`, `Telnet_Open`, `Telnet_Send`, `Telnet_SendFile`, `Telnet_WaitPrompt`, `Telnet_Close`）** および共通基盤コマンド（`Time_Stamp`, `Skip_Line`, `Get_Line`, `Set_Data`, `Judge`, `Jump`, `Exit`）のみを使用してください。
   * 他通信ドメインのコマンドセット（`Com_*`, `Modbus_*`, `Melsec_*`, `Mc_*` 等）を混在させることは排他制御違反となるため **絶対禁止** です。

---

## 📡 3. 対象システム仕様および TITLE ブロック定義

Telnet / TCPソケット対話通信制御システム向けとして、`TITLE` ブロックを以下の通り定義してください。

* **タイトル名**: `"Telnet / TCPソケット 対話通信制御"`
* **書式**:
  ```csv
  TITLE, "Telnet / TCPソケット 対話通信制御"
  ```

---

## 📐 4. 実装すべき COMMAND ブロックと完全機能別パラメータ設計仕様

以下の 6 つの独立した `COMMAND` ブロックを実装してください（`TermAnalysis.Telnet` の全 6 コマンドを網羅）。

### ① 通信モード設定・Telnet/TCP接続オープン：`COMMAND, "PORT_OPEN"`
* **目的**: Telnet 送信改行・データモード・NVT 制御フィルタを `List` パラメータから設定し、指定ホスト IP・ポート番号・接続タイムアウトで TCP/Telnet ソケットセッションを排他オープンする。
* **パラメータ定義**:
  * `HOST_IP` (`Text`, `"接続先ホスト/IPアドレス"`, `"192.168.1.10"`)
  * `PORT_NO` (`Text`, `"接続先ポート番号"`, `"23"`)
  * `CONN_TIMEOUT` (`Text`, `"接続タイムアウト(ms)"`, `"5000"`)
  * `TX_DELIM` (`List`, `"送信改行コード"`, `"TXCRLF"`, `"TXCR"`, `"NON"`)
  * `DATA_MODE` (`List`, `"受信データモード"`, `"ASCII"`, `"BIN"`)
  * `NVT_FILTER` (`List`, `"NVT制御フィルタ(IAC)"`, `"ON"`, `"OFF"`)
* **シーケンス手順**:
  1. ステップ 10: `Telnet_Mode, "$TX_DELIM$", "$DATA_MODE$", "$NVT_FILTER$"`
  2. ステップ 20: `Telnet_Open, "$HOST_IP$", "$PORT_NO$", "$CONN_TIMEOUT$"`
  3. ステップ 30: `Exit`

### ② リモートホスト認証・シェルプロンプト確立：`COMMAND, "LOGIN_AUTH", ON`
* **目的**: 標準 Telnet サーバーのユーザー名／パスワード対話認証（`USER_PASS`）と、認証不要なコンソール・Raw TCP 機器のプロンプト確認のみ（`PROMPT_ONLY`）を `AUTH_MODE` で切り替え、シェルプロンプト（既定 `"#"`）の到着を確認する。
* **パラメータ定義**:
  * `AUTH_MODE` (`List`, `"認証モード"`, `"USER_PASS"`, `"PROMPT_ONLY"`)
  * `LOGIN_PROMPT` (`Text`, `"ログインプロンプト"`, `"login:"`)
  * `USER_ID` (`Text`, `"ユーザー名"`, `"admin"`)
  * `PASS_PROMPT` (`Text`, `"パスワードプロンプト"`, `"Password:"`)
  * `PASSWORD` (`Text`, `"パスワード(実行時指定)"`, `""`)
  * `SHELL_PROMPT` (`Text`, `"シェルプロンプト"`, `"#"`)
  * `TIMEOUT_MS` (`Text`, `"待機タイムアウト(ms)"`, `"10000"`)
* **シーケンス手順**:
  1. ステップ 10: `Judge, EQU, "$AUTH_MODE$", "USER_PASS", #WAIT_SHELL#`
  2. ステップ 20: `Telnet_WaitPrompt, "$LOGIN_PROMPT$", "$TIMEOUT_MS$", #AUTH_ERR#`
  3. ステップ 30: `Telnet_Send, "$USER_ID$"`
  4. ステップ 40: `Telnet_WaitPrompt, "$PASS_PROMPT$", "$TIMEOUT_MS$", #AUTH_ERR#`
  5. ステップ 50: `Telnet_Send, "$PASSWORD$"`
  6. ステップ `#WAIT_SHELL#`: `Telnet_WaitPrompt, "$SHELL_PROMPT$", "$TIMEOUT_MS$", #AUTH_ERR#`
  7. ステップ 70: `Exit`
  8. ステップ `#AUTH_ERR#`: `Exit`

### ③ リモートコマンド実行・応答行データ抽出：`COMMAND, "STATUS_READ", "RESULT_DATA"`
* **目的**: タイムスタンプ自動付与を一時 OFF にした上で状態取得コマンド（既定 `"show version"`）を送信し、シェルプロンプト受信後にエコーバック行のスキップ（`ECHO_SKIP` が 1 以上の場合のみ `Skip_Line` 実行）および指定行抽出（`Get_Line`）を行って結果セル（`RESULT_DATA`）へ展開する。初回タイムアウト時は 1 回再送し、再タイムアウト時は `"ERROR_TIMEOUT"` を格納してタイムスタンプ設定を復帰する。
* **パラメータ定義**:
  * `QUERY_CMD` (`Text`, `"状態取得コマンド"`, `"show version"`)
  * `SHELL_PROMPT` (`Text`, `"応答プロンプト"`, `"#"`)
  * `TIMEOUT_MS` (`Text`, `"待機タイムアウト(ms)"`, `"5000"`)
  * `ECHO_SKIP` (`List`, `"エコー行スキップ数"`, `"1"`, `"0"`, `"2"`)
  * `TARGET_LINE` (`Text`, `"抽出対象行番号"`, `"1"`)
  * `RESULT_DATA` (`Range`, `"読出データ結果"`, `""`)
* **シーケンス手順**:
  1. ステップ 10: `Time_Stamp, "OFF"`
  2. ステップ 20: `Telnet_Send, "$QUERY_CMD$"`
  3. ステップ 30: `Telnet_WaitPrompt, "$SHELL_PROMPT$", "$TIMEOUT_MS$", #RETRY_QUERY#`
  4. ステップ `#PARSE_RES#`: `Judge, GT, "$ECHO_SKIP$", "0", #EXTRACT_RES#`
  5. ステップ 50: `Skip_Line, "$ECHO_SKIP$"`
  6. ステップ `#EXTRACT_RES#`: `Get_Line, "RESULT_DATA", "$TARGET_LINE$"`
  7. ステップ 70: `Time_Stamp, "ON"`
  8. ステップ 80: `Exit`
  9. ステップ `#RETRY_QUERY#`: `Telnet_Send, "$QUERY_CMD$"`
  10. ステップ 100: `Telnet_WaitPrompt, "$SHELL_PROMPT$", "$TIMEOUT_MS$", #QUERY_TIMEOUT#`
  11. ステップ 110: `Jump, #PARSE_RES#`
  12. ステップ `#QUERY_TIMEOUT#`: `Set_Data, "RESULT_DATA", "ERROR_TIMEOUT"`
  13. ステップ 130: `Time_Stamp, "ON"`
  14. ステップ 140: `Exit`

### ④ 任意コマンド送信・全応答ログ取得：`COMMAND, "CMD_EXEC", "RESULT_DATA"`
* **目的**: 任意のリモートコマンド文字列（既定 `"uname -a"`）を指定の行末改行コード（`CRLF` / `CR` / `LF` / `NONE`）で送信し、シェルプロンプト待機後に受信リングバッファの応答ログ全文を `RESULT_DATA` へ格納する。
* **パラメータ定義**:
  * `SEND_CMD` (`Text`, `"送信コマンド文字列"`, `"uname -a"`)
  * `LINE_END` (`List`, `"行末改行指定"`, `"CRLF"`, `"CR"`, `"LF"`, `"NONE"`)
  * `SHELL_PROMPT` (`Text`, `"応答プロンプト"`, `"#"`)
  * `TIMEOUT_MS` (`Text`, `"待機タイムアウト(ms)"`, `"5000"`)
  * `RESULT_DATA` (`Range`, `"応答ログ結果"`, `""`)
* **シーケンス手順**:
  1. ステップ 10: `Telnet_Send, "$SEND_CMD$", "$LINE_END$"`
  2. ステップ 20: `Telnet_WaitPrompt, "$SHELL_PROMPT$", "$TIMEOUT_MS$", #EXEC_TIMEOUT#`
  3. ステップ 30: `Set_Data, "RESULT_DATA"`
  4. ステップ 40: `Exit`
  5. ステップ `#EXEC_TIMEOUT#`: `Set_Data, "RESULT_DATA", "ERROR_TIMEOUT"`
  6. ステップ 60: `Exit`

### ⑤ 外部コンフィグ・スクリプト一括送信：`COMMAND, "SCRIPT_SEND"`
* **目的**: 外部テキストファイルに記述されたコンフィグ・初期化コマンド群を `Telnet_SendFile` で逐次送信し、完了プロンプトおよび通信エラーフラグ（`COM_ERR_FLG`）を確認する。
* **パラメータ定義**:
  * `FILE_PATH` (`Text`, `"送信スクリプトパス"`, `"./scripts/device_setup.txt"`)
  * `WAIT_OPTION` (`List`, `"行間待機モード"`, `"WaitPrompt"`, `"NP"`, `"50"`, `"100"`)
  * `CLEAR_ERR` (`List`, `"送信前エラークリア"`, `"true"`, `"false"`)
  * `SHELL_PROMPT` (`Text`, `"完了確認プロンプト"`, `"#"`)
  * `TIMEOUT_MS` (`Text`, `"待機タイムアウト(ms)"`, `"10000"`)
  * `ERR_STATUS` (`Work`, `"通信エラー状態"`, `""`)
* **シーケンス手順**:
  1. ステップ 10: `Telnet_SendFile, "$FILE_PATH$", "$WAIT_OPTION$", "$CLEAR_ERR$"`
  2. ステップ 20: `Telnet_WaitPrompt, "$SHELL_PROMPT$", "$TIMEOUT_MS$", #SCRIPT_END#`
  3. ステップ 30: `Set_Data, "ERR_STATUS", "COM_ERR_FLG"`
  4. ステップ 40: `Judge, EQU, "$ERR_STATUS$", "CLR", #SCRIPT_END#`
  5. ステップ `#SCRIPT_END#`: `Exit`

### ⑥ ログアウト送出・Telnetソケット切断：`COMMAND, "PORT_CLOSE"`
* **目的**: 必要に応じてログアウトコマンド（既定 `"exit"`）を送出した上で、オープン中の Telnet / TCP ソケットセッションを安全に切断・解放する。
* **パラメータ定義**:
  * `SEND_LOGOUT` (`List`, `"切断前ログアウト送信"`, `"OFF"`, `"ON"`)
  * `LOGOUT_CMD` (`Text`, `"ログアウトコマンド"`, `"exit"`)
* **シーケンス手順**:
  1. ステップ 10: `Judge, EQU, "$SEND_LOGOUT$", "ON", #DO_CLOSE#`
  2. ステップ 20: `Telnet_Send, "$LOGOUT_CMD$"`
  3. ステップ `#DO_CLOSE#`: `Telnet_Close`
  4. ステップ 40: `Exit`

---

## ✅ 5. 出力前セルフチェックリスト

- [ ] 他通信ドメインのコマンド（`Com_*`, `Modbus_*`, `Melsec_*`, `Mc_*` 等）を一切使用せず、Telnet 用コマンドセット全 6 種（`Telnet_Mode`, `Telnet_Open`, `Telnet_Send`, `Telnet_SendFile`, `Telnet_WaitPrompt`, `Telnet_Close`）と共通基盤コマンドのみで完結しているか？
- [ ] 接続条件・モード選択項目（`TX_DELIM`, `DATA_MODE`, `NVT_FILTER`, `AUTH_MODE`, `ECHO_SKIP`, `LINE_END`, `WAIT_OPTION`, `CLEAR_ERR`, `SEND_LOGOUT`）がすべて `List` 型でドロップダウン選択可能になっているか？
- [ ] パスワード項目（`PASSWORD`）に固定の平文パスワードをハードコードせず、初期値が空文字 `""` に設定されているか？
- [ ] `ECHO_SKIP` が `"0"` の場合に `Skip_Line, "0"` を呼び出さないよう、事前に `Judge, GT, "$ECHO_SKIP$", "0", #EXTRACT_RES#` で安全に分岐しているか？
- [ ] すべての `TITLE`, `COMMAND` 行が行頭（第1フィールド）から始まっているか？
- [ ] すべての `PARAM`, `SEQUENCE` 行が行頭カンマ（`,`）から始まっているか？
- [ ] すべての `PARAM` 行において、第5フィールド（表示名）および第6フィールド（初期値）が省略・欠落せずに明記されているか？
- [ ] `COMMAND` ブロック配下の `PARAM` 行・`SEQUENCE` 行に行末コメント（`;`）が一切含まれていないか？
- [ ] ファイル先頭ヘッダに正確な AI エンジン識別名（`; 生成AI: Antigravity (Gemini 4 Carbon (Medium))`）が記載され、ローカル絶対パスが含まれていないか？
- [ ] 生成される CSV ファイルは Shift-JIS (CP932) + CRLF で保存される前提となっているか？

---

## 📋 6. 出力フォーマット

余計な自然言語による前置きや解説を省き、すぐに CSV ファイル（`Telnet.csv`）として保存できるように、 **マークダウンのコードブロック（`csv` 形式）に作成した CSV コードのみ** を出力してください。
