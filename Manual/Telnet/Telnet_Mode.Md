# コマンドリファレンス：Telnet_Mode

## 概要

**Telnet_Mode** は、Telnet（および Raw TCP ソケット）通信における送信デリミタ（`TXCR` / `TXCRLF` / `NON`）、受信データモード（`ASCII` / `BIN`）、および Telnet NVT 制御コード（IAC: `0xFF`）のフィルタ・自動ネゴシエーション応答の有効／無効（`ON` / `OFF`）を設定する統合モード制御コマンドです。

> 💡 **補足** : 第 3 引数 `NvtFilter` を `ON` にすると標準 Telnet サーバー（ルーターや Linux ホスト等）とのネゴシエーションを行い、`OFF` にすると `0xFF` を含む任意のバイナリフレームを透過させる Raw TCP ソケット通信として機能します。

---

## 💡 書式・パラメータ仕様

### コマンド構文

```csv
,SEQUENCE, [ステップキー], Telnet_Mode, "[TxDelimiterOrSettingType]", "[DataModeOrValue]", "[NvtFilter: ON/OFF]"
```

※ 上記の最大 3 引数による位置指定のほか、`Telnet_Mode, "<SettingType>", "<Value>"` の 2 引数キー指定形式（`IP` / `HOST`, `PORT`, `LINEENDING` / `TXDELIMITER`, `DATAMODE`, `NVTFILTER` / `NVT`）や、別名 `Telnet_Ip` / `Analysis_Telnet_Ip` による接続先 IP アドレス設定にも対応しています。

### パラメータ（引数）一覧

| 引数位置 | パラメータ名 | 必須/任意 | 型・制約 | 説明 |
| :---: | :--- | :---: | :--- | :--- |
| 第1引数 | `TxDelimiter` | 任意 | `TXCR`（`CR`） / `TXCRLF`（`CRLF`） / `NON`（`NONE`） | `Telnet_Send` 等で自動付与する送信末改行コードを指定します。空文字 `""` または省略時は現在値を維持します。<br>※ `"TXLF"` 等の未定義文字列を指定した場合、`TermRingBufferManager.ApplyTxDelimiter` では未対応（`false`）として **`TermRingBufferManager.TxDelimiter`（および `TermForm.TXCRLF`）は変更前の設定値を維持** し、`SocketTransportCore.Shared.NewLine` のみが `"\r"` となります（`LF` 単体で送信したい場合は `Telnet_Send` 側の改行引数で `"LF"` を指定します）。 |
| 第2引数 | `DataMode` | 任意 | `ASCII` / `BIN` | 受信リングバッファおよびログの解釈モード（テキスト解析／バイナリストリーム）を指定します（未定義文字列を指定した場合は `TermRingBufferManager.ApplyDataMode` が `false` を返し、 **変更前の `DataMode` 設定値を維持** します）。 |
| 第3引数 | `NvtFilter` | 任意 | `ON` / `OFF`（既定値: `ON`） | Telnet NVT プロトコル（IAC `0xFF` 制御シーケンス）の分離・自動応答を行うか（`ON` / `TRUE` / `1`）、Raw TCP ソケットとして全バイトをそのまま透過させるか（`OFF` / `FALSE` / `0`）を指定します。 |

---

## 🔄 特殊置換規約・分岐規約の適用

本コマンドの引数における特殊記法および解釈ルールです。
* **動的置換（`$`）** : 引数内で `$PARAMキー$` のように記述すると、実行時に変数プール（PARAM）の値へ動的置換されます。（適用可能な引数: 全引数対応）
* **ジャンプ先指定（`#`）** : 本コマンドはジャンプ制御を行わないため適用されません。（適用可能な引数: 非適用）
* **共通コマンド指定（`@`）** : 本コマンドは外部マクロを呼び出さないため、共通コマンド呼び出し記法は適用されません。（適用可能な引数: 非適用）
* **カンマのエスケープ** : 通信条件文字列（例: `"9600,E,7,1"`）や複数データのカンマ区切り値を単一引数として渡す場合は、必ず `"..."` と二重引用符で囲んで記述します。
* **エイリアス（別名）** : `Telnet_Ip`, `Analysis_Telnet_Ip`

---

## 💡 動作例（正しい構文での記述例）

### スクリプト記述例

※COMMAND ブロック配下の子要素行（PARAM, SEQUENCE）は必ず行頭カンマから開始し、スペースインデントや行末コメント（`;`）は記述しないでください。

```csv
COMMAND, "SAMPLE_TELNET_MODE", ""
; 1. 標準Telnet接続向け設定（CRLF改行、ASCIIモード、NVT IACフィルタ有効）
,SEQUENCE, 010, Telnet_Mode, "TXCRLF", "ASCII", "ON"
,SEQUENCE, 020, Telnet_Open, "192.168.1.50", "23"
,SEQUENCE, 030, Telnet_Close
; 2. Raw TCPソケット機器向け設定（改行なし、BINモード、NVTフィルタ無効）
,SEQUENCE, 040, Telnet_Mode, "NON", "BIN", "OFF"
,SEQUENCE, 050, Telnet_Open, "192.168.1.50", "5000"
,SEQUENCE, 060, Telnet_Send, "*IDN?", "", "NONE"
,SEQUENCE, 070, Telnet_Close
,SEQUENCE, 080, Exit
```

### 実行時の挙動・変数変化

* **実行前の状態** : Term 通信エンジンが初期化され、必要な通信パラメータおよび変数プール（`PARAM`）が準備されている状態。
* **実行後の状態** : 1 回目の `Telnet_Mode` では標準 Telnet クライアントとして動作し、サーバーからの `IAC DO/WILL` ネゴシエーションを自動処理してテキストを受信バッファへ格納します。 2 回目の `Telnet_Mode` では `TxDelimiter` が `NON`、`DataMode` が `BIN`、`NvtFilter` が `OFF` になるため、受信ストリーム内の `0xFF` バイトが Telnet 制御コードとして除去されることなく、そのまま Raw TCP ストリームとして送受信されます。

---

## 🔄 動作フロー（シーケンス図）

```mermaid
sequenceDiagram
    autonumber
    participant Script as "スクリプト実行エンジン (TermSequenceDriver / AnalysisEngine)"
    participant Cmd as "Telnet_ModeCommand (TermAnalysis.Telnet)"
    participant Buf as "TermRingBufferManager (モード・NVT制御)"
    participant Sock as "SocketTransportCore (TCP設定)"

    Script->>Cmd: Func(parent, param)
    Cmd->>Cmd: ValidateParamRange(0, 3) および引数の解析 (Telnet_Ip 別名 / 2引数 SettingType 指定 / 3引数位置指定)
    opt Host / Port が指定されている場合
        Cmd->>Sock: Host / Port 設定更新
    end
    opt TxDelimiter が指定されている場合
        Cmd->>Buf: ApplyTxDelimiter(txMode, parent) (TXCR / TXCRLF / NON。未定義時は変更前値を維持)
        Cmd->>Sock: NewLine 設定同期更新
    end
    opt DataMode が指定されている場合
        Cmd->>Buf: ApplyDataMode(dataMode, parent) (ASCII / BIN。未定義時は変更前値を維持)
    end
    opt NvtFilter が指定されている場合
        Cmd->>Buf: NvtFilterEnabled 設定更新 (true: IAC処理有効 / false: Raw TCP透過)
    end
    alt 設定適用が正常完了した場合
        Cmd-->>Script: 正常完了 (IsCommandGo = true)
    else 引数範囲外 (4個以上: -10) または設定変更例外 (-1) の場合
        Cmd->>Cmd: エラーログ出力 (_log.LogError / _log.LogCritical)
        Cmd-->>Script: 安全停止 (SetErrorAndHalt(-10 / -1, ...))
    end
```

---

## 🚫 エラーハンドリング・安全設計（仕様制限）

1. **引数境界保護と未定義キーワード指定時の前回設定維持・フォールバック** :
   - 4 個以上の引数が指定された場合は `ValidateParamRange(0, 3)` によりエラーコード `-10` で安全停止します。`TxDelimiter` や `DataMode` に未定義の文字列が渡された場合は、`TermRingBufferManager.ApplyTxDelimiter` / `ApplyDataMode` が `false` を返すため `TXCR` や `ASCII` へ強制上書きせず **変更前の設定値をそのまま維持** します（なお `TxDelimiter` 未定義時は `SocketTransportCore.Shared.NewLine` のみが `"\r"` となり、`NvtFilter` に `"OFF"` / `"FALSE"` / `"0"` 以外が渡された場合は `ON`［`true`］として扱われます）。
2. **BIN モードと NVT フィルタの整合性保護** :
   - Raw TCP ソケットでバイナリフレームを送受信する際、ペイロード内の `0xFF` が IAC 制御文字として処理・欠落するのを防ぐため、`BIN` モード選択時には `NvtFilter` を `"OFF"` に設定することが推奨されます。

---

## 🧪 ユニットテスト仕様

本コマンドクラス（`Telnet_ModeCommand.cs`）に実装されている `RunUnitTest()` メソッドが検証するユニットテストの仕様です。

### テスト実行前提条件

* **実行トリガー** : `RunUnitTest()` メソッドの呼び出し
* **有効化条件** : システム全体の最小ログレベル（`ErrorLogger.MinimumLevel`）が `LogLevel.Test` であること（それ以外の場合は無効化ガードにより早期リターン）
* **検証結果出力** : `_log.LogTest` によるログ出力。検証失敗時は `_log.LogError`、予期せぬ例外発生時は `_log.LogCritical` で記録し、`finally` ブロックで `ApplyTelnetSetting` により `"TXDELIMITER"`=`"TXCR"`、`"DATAMODE"`=`"ASCII"`、`"NVTFILTER"`=`"ON"` へ初期状態を復元します。

### テストケース一覧

| No | テストカテゴリ | テスト条件（入力値） | 期待される結果（出力値） | 検証の目的 / 観点 |
| :---: | :--- | :--- | :--- | :--- |
| **1** | 正常系 | 引数 `"TXCRLF"`, `"BIN"`, `"OFF"` の 3 引数を一括指定して実行 | `!param.HasError` かつ `TxDelimiter == TermTxDelimiterMode.TXCRLF`、`DataMode == TermDataMode.BIN`、`!TermRingBufferManager.Instance.NvtFilterEnabled` となる | 送信デリミタ・バイナリ受信モード・NVT フィルタ無効化（Raw TCP 透過）の一括設定検証 |
| **2** | 異常系 | 過剰引数 4 個（`"TXCR"`, `"ASCII"`, `"ON"`, `"EXTRA_INVALID"`）を指定して `Func(null, invalidParam)` を実行（`ValidateParamRange(0, 3)` 違反） | `invalidParam.HasError == true` かつ `!invalidParam.IsCommandGo` で安全に実行拒否・停止する | 引数個数上限（最大 3 個）の境界ガード検証 |
