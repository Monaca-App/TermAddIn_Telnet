<img src="images/Monaca_store.png" align="right" width="130" alt="Monaca Mascot" />

# TermAddIn_Telnet - Telnet / TCP ソケット通信エクステンション

👉 [← TermAddIn 統合本体に戻る](https://github.com/Monaca-App/TermAddIn)  
👉 [← Monaca アプリ工房（トップ）に戻る](https://github.com/Monaca-App/store)

---

[![Platform](https://img.shields.io/badge/Platform-Windows%2010%20%2F%2011-blue.svg)](https://www.microsoft.com/)
[![Excel](https://img.shields.io/badge/Excel-2016%20%2F%202019%20%2F%202021%20%2F%20M365-green.svg)](https://www.microsoft.com/excel)
[![Framework](https://img.shields.io/badge/.NET%20Framework-4.8-purple.svg)](https://dotnet.microsoft.com/)
[![Architecture](https://img.shields.io/badge/Architecture-TermAddIn%20Extension-orange.svg)]()
[![License](https://img.shields.io/badge/License-TermAddIn__Telnet%20License-blue.svg)]()

## 📖 概要

**TermAddIn_Telnet** は、Excel 総合通信アドイン **TermAddIn** 上で動作する **Telnet (RFC 854) / Raw TCP ソケット通信専用エクステンション（機能ドメイン単一 DLL）** です。

ネットワーク機器（ルーター、L2/L3 スイッチ、ファイアウォール）、Linux / UNIX サーバー、組込みネットワークボード、Ethernet 接続型電子計測器や各種スマートデバイスとの対話的コマンド制御・設定自動バックアップ・ログ収集を Excel ワークシート上から完全自動化します。

VBA マクロを一切記述することなく、 **標準の `.xlsx`（マクロ無効ブック）形式のままで TCP 接続、NVT ネゴシエーション、プロンプト自動待機、外部設定ファイルの一括流し込み、受信ログのセル自動展開** が安全に実行できます。

> [!NOTE]
> **利用前提とライセンス体系**  
> 本エクステンションは [TermAddIn 本体](https://github.com/Monaca-App/TermAddIn) 上で動作します。  
> TermAddIn ではプロトコル毎に個別ライセンスを設ける体系を採用しており、本機能のご利用には **`TermAddIn_Telnet` 個別ライセンス（料金未定・準備中）** が適用されます（現在は評価期間として全機能を無償でご試用いただけます）。

---

## ✨ 主な機能・特徴

1. **RFC 854 準拠の標準 Telnet と Raw TCP ソケットの完全両立**
   * 標準の Telnet 通信（ポート 23 等）に加え、任意のポート番号を指定した Raw TCP ソケット通信をシームレスにサポート。
2. **NVT ネゴシエーション＆バイナリ透過の動的切替 (`Telnet_Mode`)**
   * Telnet NVT（Network Virtual Terminal）制御シーケンス（IAC: `0xFF`）の自動フィルタリングおよびネゴシエーション応答に対応。`NvtFilter` を `OFF` に設定することで、制御コードを含む任意のバイナリフレームを透過送受信可能。
3. **プロンプト自動待機 ＆ タイムアウト分岐 (`Telnet_WaitPrompt`)**
   * 接続先ホストからのログインプロンプト（`"login:"`, `"Password:"`）やシェルプロンプト（`">"`, `"#"`, `"$"` 等）を非同期待機。指定タイムアウト（ミリ秒）到達時の自動ラベルジャンプ機能により、堅牢な対話シーケンスを実現。
4. **外部設定スクリプトの一括送信 (`Telnet_SendFile`)**
   * ネットワーク機器のコンフィグファイルや初期化スクリプトを読み込み、行単位でウェイトやプロンプト待機を挟みながら確実・迅速に一括送信。
5. **Excel 32bit / 64bit 完全動作マトリクス**
   * .NET Framework 標準のマネージドソケット基盤（`System.Net.Sockets`）を採用しており、外部の専用ネイティブ DLL に依存しないため、Excel 32bit 版および 64bit 版の双方で制限なく完全動作。

---

## 🔌 提供コマンド一覧（全 6 コマンド）

各コマンドの詳細な構文・パラメータ仕様・記述例は、以下の個別リファレンスをご参照ください。

| コマンド名 | 構文概要 | 主な役割・用途 | 詳細マニュアル |
| :--- | :--- | :--- | :---: |
| **`Telnet_Open`** | `Telnet_Open` | 設定された接続先ホスト（IP/ポート）へ TCP/Telnet 接続を開設 | [詳細説明](./Manual/Telnet/Telnet_Open.Md) |
| **`Telnet_Close`** | `Telnet_Close` | オープン中の Telnet/TCP セッションを安全に切断・リソース解放 | [詳細説明](./Manual/Telnet/Telnet_Close.Md) |
| **`Telnet_Mode`** | `Telnet_Mode, "[TxDelimiter]", "[DataMode]", "[NvtFilter]"` | 送信改行、データモード（ASCII/BIN）、NVT フィルタ（ON/OFF）の動的変更 | [詳細説明](./Manual/Telnet/Telnet_Mode.Md) |
| **`Telnet_Send`** | `Telnet_Send, "<送信文字列/ファイルパス>", "[形式]", "[エラー受信用]"` | コマンド文字列または 16 進バイナリ電文の送信 | [詳細説明](./Manual/Telnet/Telnet_Send.Md) |
| **`Telnet_SendFile`** | `Telnet_SendFile, "<ファイルパス>", "[待機条件]"` | 外部テキストファイル（コンフィグ等）を行単位で順次送信 | [詳細説明](./Manual/Telnet/Telnet_SendFile.Md) |
| **`Telnet_WaitPrompt`** | `Telnet_WaitPrompt, "<プロンプト>", "[タイムアウトms]", "[タイムアウト時ジャンプ先]"` | 応答プロンプト文字列の受信待機および分岐処理 | [詳細説明](./Manual/Telnet/Telnet_WaitPrompt.Md) |

---

## 💻 動作環境・システム要件

| 項目 | システム要件 |
| :--- | :--- |
| **OS** | Windows 10 / Windows 11 (64bit / 32bit) |
| **対応 Excel** | Microsoft Excel 2016 / 2019 / 2021 / Microsoft 365 (デスクトップ版 32bit / 64bit) |
| **必須アドイン** | [TermAddIn](https://github.com/Monaca-App/TermAddIn) (Excel VSTO アドイン) |
| **ランタイム** | .NET Framework 4.8 / Visual Studio 2010 Tools for Office Runtime |
| **インターフェース** | Ethernet (10BASE-T / 100BASE-TX / 1000BASE-T), Wi-Fi 等の TCP/IP ネットワーク回線 |
| **対応プロトコル** | Telnet (RFC 854), Raw TCP ソケット |
| **対象機器** | ネットワーク機器（ルーター、スイッチ）、Linux/UNIX サーバー、組込みボード、LAN 対応計測器 |

---

## 🔰 シーケンス記述例（Excel セル設定）

Excel シート上での典型的な Telnet ログイン・コマンド実行シーケンスの構成例です。

```csv
; 1. Telnet 接続（改行: CRLF、ASCII、NVT フィルタ有効）
,SEQUENCE, 010, Telnet_Mode, "TXCRLF", "ASCII", "ON"
,SEQUENCE, 020, Telnet_Open
; 2. ログインプロンプト "login:" を最大 10000ms 待機
,SEQUENCE, 030, Telnet_WaitPrompt, "login:", 10000, "#ERROR"
,SEQUENCE, 040, Telnet_Send, "admin"
; 3. パスワードプロンプト "Password:" を最大 5000ms 待機
,SEQUENCE, 050, Telnet_WaitPrompt, "Password:", 5000, "#ERROR"
,SEQUENCE, 060, Telnet_Send, "secret123"
; 4. シェルプロンプト ">" を待機
,SEQUENCE, 070, Telnet_WaitPrompt, ">", 5000, "#ERROR"
; 5. バージョン情報取得コマンド送信
,SEQUENCE, 080, Telnet_Send, "show version"
,SEQUENCE, 090, Telnet_WaitPrompt, ">", 5000, "#ERROR"
; 6. セッションを切断して正常終了
,SEQUENCE, 100, Telnet_Close
,SEQUENCE, 110, Exit
; エラー処理ブロック
#ERROR
,SEQUENCE, 120, Msg_Box, "Telnet 通信でタイムアウトが発生しました。"
,SEQUENCE, 130, Telnet_Close
```

---

## 💳 ライセンス・購入プラン（TermAddIn_Telnet）

> [!NOTE]
> **料金プランおよび正式販売について**  
> 本エクステンション（`TermAddIn_Telnet`）の販売料金およびライセンスプラン（買い切り・サブスクリプション）は **現在検討中（未定）** です。  
> 正式な価格およびお申込み受付開始時期が決定次第、本ドキュメントおよび [Monaca アプリ工房ポータル](https://github.com/Monaca-App/store) にてご案内いたします。  
> 現在は評価版として全機能をご試用いただけます。

| プラン | 価格 | お支払い・お申込み |
| :--- | :---: | :---: |
| **一括購入（買い切りライセンス）**<br>無期限で永続利用できるライセンスです。 | **未定** | 準備中 |
| **月額サブスクリプション**<br>月単位で利用できる定期プランです。中長期のプロジェクトにおすすめです。 | **未定** | 準備中 |
| **週額サブスクリプション**<br>週単位で手軽に利用できるプランです。短期評価や単発テストにおすすめです。 | **未定** | 準備中 |

* ※ **ライセンスファイル名**: `TermAddIn_Telnet.License`
* ※ **配置先**: `%USERPROFILE%\Desktop\Monaca\TermAddIn_Telnet.License`（設定ファイル `Setting.xml` と同階層）
* ※ **ライセンス形態**: 1 ユーザー / 1 PC（ノードロック方式：`PC名` および `ログインユーザー名` に紐づけ）

---

## 📄 ライセンス & サポート

* **開発元**: [Monaca アプリ工房 (Monaca App Store)](https://github.com/Monaca-App/store)
* **親プロジェクト**: [TermAddIn](https://github.com/Monaca-App/TermAddIn)
* **Copyright**: Copyright (C) Monaca App Store 2026 All rights reserved.
* **お問い合わせ**: 不具合報告や機能要望は、GitHub の Issue までお気軽にお寄せください。
