# コマンドリファレンス：Telnet_Send

---

## 概要

**Telnet_Send** は、接続中の Telnet（または Raw TCP ソケット）ストリームに対して文字列メッセージ、16 進バイト列（HEX）、または指定ファイルの内容を送信するコマンドです。
第 3 引数にステータス格納先変数を指定することで、ソケット切断や送信タイムアウト等の通信エラーフラグ監視付き送信としても動作します。

> 💡 **補足**: `Telnet_Mode` で設定された送信デリミタ（`TXCR` / `TXCRLF` / `NON`）が送信データ末尾に自動付与されます。

---

## 💡 書式・パラメータ仕様

### コマンド構文

```csv
,SEQUENCE,Telnet_Send, "<Message/FilePath>", "[Format/Mode]", "[ErrorStatusTarget]"
```

### パラメータ（引数）一覧

| 引数位置 | パラメータ名 | 必須/任意 | 型・制約 | 説明 |
| :---: | :--- | :---: | :--- | :--- |
| 第1引数 | `Message/FilePath` | **必須** | 文字列 | 送信する文字列データ、16進文字列、または送信対象ファイルのパスを指定します。 |
| 第2引数 | `Format/Mode` | 任意 | `ASCII` / `HEX` / `FILE`（既定値: `ASCII`） | 送信データの解釈モードを指定します。<br>・`ASCII`: テキスト文字列として送信（デリミタ自動付与）<br>・`HEX`: 16進文字列（例: `"02 41 42 03"`）をバイナリ変換して送信<br>・`FILE`: 第1引数をファイルパスとして読み込み送信 |
| 第3引数 | `ErrorStatusTarget` | 任意 | 変数名（`$` / `@`） | 送信時の通信エラー判定結果（`"0"`: 正常 / `"1"`: エラー発生）を格納する変数名を指定します。 |

---

## 🔄 特殊置換規約・分岐規約の適用

1. **変数展開（`$` / `@` / `#`）の完全サポート**:
   - 第 1 引数に `"MEAS:VOLT? $CH"` や `"@CMD_LINE"` を指定すると、実行時に変数値へ展開されてから送信されます。
2. **カンマを含む文字列の送信**:
   - カンマを含むコマンド文字列（例: `"CONF:CURR 1.5,0.01"`）を送信する場合は、ダブルクォーテーションで囲むことで第 1 引数として保護されます。

---

## 💡 動作例（正しい構文での記述例）

### スクリプト記述例

```csv
,PARAM,TARGET_IP_CMD,show ip interface brief
,PARAM,TX_ERR,0

; 1. 変数展開を含むTelnetコマンド送信
,SEQUENCE,Telnet_Send, "$TARGET_IP_CMD"

; 2. Raw TCPソケット機器へのHEXバイナリフレーム送信とエラーフラグ確認
,SEQUENCE,Telnet_Send, "50 00 00 FF FF 03 00", "HEX", "$TX_ERR"
,SEQUENCE,Judge, "$TX_ERR", "==", "1", "SOCKET_ERR_HANDLER"
,SEQUENCE,Exit
```

### 実行時の挙動・変数変化

1. 1 回目の `Telnet_Send` では `"show ip interface brief"` に送信デリミタ（`\r\n`）が付与されて TCP ストリームへ送出されます。
2. 2 回目の `Telnet_Send` では 7 バイトのバイナリフレームが直接送出され、正常送信時は `$TX_ERR` に `"0"` が格納されます。

---

## 🔄 動作フロー（シーケンス図）

```mermaid
sequenceDiagram
    autonumber
    participant Script as "スクリプト実行エンジン (SequenceRunner)"
    participant Cmd as "Telnet_SendCommand (TermAnalysis.Telnet)"
    participant Var as "VariableManager (変数管理)"
    participant Sock as "TcpClient / NetworkStream"

    Script->>Cmd: Execute(context, args)
    Cmd->>Var: 引数の変数展開 ($ / @ / #)
    Cmd->>Cmd: Format/Mode 判定 (ASCII / HEX / FILE)
    alt Format == ASCII
        Cmd->>Cmd: Telnet_Mode設定のTxDelimiter(CR/CRLF/NON)を末尾付与
    else Format == HEX
        Cmd->>Cmd: 16進文字列をbyte[]配列へパース
    else Format == FILE
        Cmd->>Cmd: 指定ファイルから送信用バイト列を読込
    end
    Cmd->>Sock: WriteAsync(payloadBytes) / FlushAsync()
    alt 第3引数 ErrorStatusTarget 指定あり
        Sock-->>Cmd: 送信完了またはSocketException結果
        Cmd->>Var: ErrorStatusTarget へ "0"(正常) または "1"(異常) を格納
    end
    Cmd-->>Script: 正常終了 (Next Step)
```

---

## 🚫 エラーハンドリング・安全設計（仕様制限）

1. **ソケット未接続時の実行防止**:
   - `Telnet_Open` が実行されていない状態で `Telnet_Send` を呼び出した場合、未接続エラーを送出してシーケンスを停止します。
2. **不正な HEX 文字列の検証**:
   - `HEX` モード指定時に 16 進数として解釈できない文字列が指定された場合は、`FormatException` を送出して不正パケット送出を防止します。

---

## 🧪 ユニットテスト仕様

### テスト実行前提条件
- **対象クラス**: `TermAnalysis.Telnet.Telnet_SendCommand`（`Telnet_SendCommand.cs`）
- **モック環境**: 接続状態の `ITelnetClientAdapter` モックが注入されていること。

### テストケース一覧

| テストID | テスト観点 | 入力スクリプト | 期待される動作・検証項目 |
| :--- | :--- | :--- | :--- |
| `UT-TELNET-SEND-01` | ASCII送信・変数展開 | `,SEQUENCE,Telnet_Send, "STAT $CH"` | 変数展開後の文字列に現在の `TxDelimiter` が付与されて送信されること。 |
| `UT-TELNET-SEND-02` | HEXバイナリ送信 | `,SEQUENCE,Telnet_Send, "AA 55 0D", "HEX"` | バイト配列 `{0xAA, 0x55, 0x0D}` がそのまま送信されること。 |
| `UT-TELNET-SEND-03` | エラーステータス格納 | `,SEQUENCE,Telnet_Send, "PING", "ASCII", "$ERR"` | 正常送信時に `$ERR` に `"0"`、ソケットエラー時に `"1"` が格納されること。 |

