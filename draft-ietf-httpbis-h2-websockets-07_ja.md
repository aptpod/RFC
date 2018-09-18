# Bootstrapping WebSockets with HTTP/2 (version 07)

## 翻訳について
この文書は、 Internet-Drafts 「RFC6455 Bootstrapping WebSockets with HTTP/2 (version 07)」の日本語訳です。

原文の最新版は、日本語訳の作成日から更新されている可能性があります。また、翻訳過程において誤りが混入している可能性があるため、内容について一切の保証を致しかねます。正確な情報を必要とする場合は、原文をご参照ください。

- 公開日: 2018-09-18
- 更新日: 2018-09-18
- 翻訳者: Ryosuke Iwata <<iwata@aptpod.co.jp>>
- 原文: [Bootstrapping WebSockets with HTTP/2](https://tools.ietf.org/html/draft-ietf-httpbis-h2-websockets-07)

## 概要
この文書は、 HTTP/2 コネクションの単一ストリーム上において WebSocket プロトコル ([RFC 6455](https://tools.ietf.org/html/rfc6455)) を実行するためのメカニズムを定義します。

## このメモの状態
この Internet-Drafts は、[[BCP 78](https://tools.ietf.org/html/bcp78)] および [[BCP 79](https://tools.ietf.org/html/bcp79)] の規定に完全に準拠して提出されています。

Internet-Drafts は、Internet Engineering Task Force（IETF）の作業文書です。 他のグループが Internet-Drafts として作業文書を配布する可能性もあることにご注意ください。現在の Internet-Drafts の一覧は、 [https://datatracker.ietf.org/drafts/current/](https://datatracker.ietf.org/drafts/current/) をご参照ください。

Internet-Drafts は、最大6か月間有効なドラフト文書であり、いつでも他の文書によって更新、置き換え、廃止される可能性があります。参考文献として Internet-Drafts を使用したり、作業文書以外のものとして引用したりすることは不適切です。

この Internet-Drafts は、2018年12月20日に失効します。

## 著作権表示
Copyright (c) 2018 IETF Trust and the persons identified as the document authors.  All rights reserved.

This document is subject to BCP 78 and the IETF Trust's Legal Provisions Relating to IETF Documents [https://trustee.ietf.org/license-info](https://trustee.ietf.org/license-info) in effect on the date of publication of this document.  Please review these documents carefully, as they describe your rights and restrictions with respect to this document.  Code Components extracted from this document must include Simplified BSD License text as described in Section 4.e of the Trust Legal Provisions and are provided without warranty as described in the Simplified BSD License.

## 目次
1. [はじめに](#1-はじめに)
2. [用語](#2-用語)
3. [SETTINGS_ENABLE_CONNECT_PROTOCOL SETTINGS パラメータ](#3-settings_enable_connect_protocol-settings-パラメータ)
4. [拡張 CONNECT メソッド](#4-拡張-connect-メソッド)
5. [拡張 CONNECT を使用して WebSocket をブートストラップする](#5-拡張-connect-を使用して-websocket-をブートストラップする)
    - 5.1. [例](#51-例)
6. [設計上の考慮事項](#6-設計上の考慮事項)
7. [中間者について](#7-中間者について)
8. [セキュリティ上の考慮事項](#8-セキュリティ上の考慮事項)
9. [IANAに関する考慮事項](#9-ianaに関する考慮事項)
10. [参考文献](#10-参考文献)
- [謝辞](#謝辞)
- [著者のメールアドレス](#著者のメールアドレス)


## 1. はじめに
HTTP（Hypertext Transfer Protocol）[[RFC7230](https://tools.ietf.org/html/rfc7230)] は、リソースレベルでは異なるバージョン間で互換性のあるセマンティクスを提供しますが、コネクション管理レベルでは互換性を提供しません。 HTTP のコネクション管理に依存する WebSocket などのプロトコルは、HTTPの新しいバージョンに対応するために定義を更新する必要があります。

WebSocketプロトコル \[[RFC6455](https://tools.ietf.org/html/rfc6455)] は、 HTTP/1.1 `Upgrade` メカニズム（[Section 6.7 of [RFC7230]](https://tools.ietf.org/html/rfc7230#section-6.7)）を使用して、 TCP 接続を HTTP から WebSocket に移行します。 HTTP/2 [[RFC7540](https://tools.ietf.org/html/rfc7540)]では、別のアプローチが必要となります。

HTTP/2 では、ストリームを多重化する性質上、`Upgrade` 、`Connection` リクエストヘッダーフィールドや、 `101 (Switching Protocol)` 応答コードなど、コネクション全体で使用するヘッダーフィールドやステータスコードが許可されません。これらは全て、 [[RFC6455](https://tools.ietf.org/html/rfc6455)] で定義されている opning handshake において要求されます。

HTTP/2 から WebSocket をブートストラップすることが可能になると、TCPコネクションを HTTP/2 と WebSocket 双方で共有することができるようになり、HTTP/2 の効率的なネットワーク利用を WebSocket に拡張できます。

この文書は、 HTTP CONNECT メソッド（HTTP/2 では [Section 8.3 of [RFC7540]](https://tools.ietf.org/html/rfc7540#section-8.3) で定義されています）を拡張します。 この拡張により、 CONNECT メソッドが通常使用する外部ホスト名の指定ではなく、新しい接続に使用するプロトコル名の指定が可能になります。その結果、WebSocket（または他のプロトコル）にデータを運ぶことができる単一の HTTP/2 ストリーム上のトンネルが得られます。同一コネクション上の他のストリームは、拡張 CONNECT によって開かれた別のトンネル、従来の HTTP/2 データ、またはそれらの両方を運ぶことができます。

このトンネリングされたストリームは、コネクション上の他の通常のストリームと多重化され、HTTP/2 による通常の優先制御、キャンセル機構、およびフロー制御機能を利用することができます。

この文書で定義されている、トンネリングされたストリームおよび修正された opening handshake を使用して WebSocket 接続を確立したストリームは、そのストリームを TCP コネクションであるかのように取扱い、ストリーム内で 従来の WebSocket プロトコルを使用して通信します。

## 2. 用語
この文書のキーワード `MUST`、`MUST NOT`、`REQUIRED`、`SHALL`、`SHALL NOT`、`SHOULD`、`SHOULD NOT`、`RECOMMENDED`、`NOT RECOMMENDED`、`MAY`、そして `OPTIONAL` は、前記のように全て大文字で記載された場合のみ、 [BCP 14](https://tools.ietf.org/html/bcp14) [[RFC2119](https://tools.ietf.org/html/rfc2119)] [[RFC8174](https://tools.ietf.org/html/rfc8174)] で記述されたとおりに解釈されるべきです。


## 3. SETTINGS_ENABLE_CONNECT_PROTOCOL SETTINGS パラメータ

この文書は、 [[RFC7540], Section 6.5.2](https://tools.ietf.org/html/rfc7540#section-6.5.2) によって定義された SETTINGS にパラメータを加えます。

新しいパラメータの名称は、 SETTINGS_ENABLE_CONNECT_PROTOCOL です。 パラメータの値は `0` または `1` でなければなりません。（ `MUST` ）

クライアントにおいて、値が `1` に設定された SETTINGS_ENABLE_CONNECT_PROTOCOL を受け取った場合、新しいストリームの生成に、この文書に記載された 拡張 CONNECT の定義を使用することができます。（ `MAY` ）サーバーがこのパラメータを受け取った場合は、動作に何も影響を与えません。

送信者は、値を `1` に設定した SETTINGS_ENABLE_CONNECT_PROTOCOL を送信した後、値を `0` に設定した同パラメータを送信してはなりません。（ `MUST NOT` ）


互換性のないプロトコル変更へオプトインするための SETTINGS パラメータの使用方法は、[Section 5.5 of [RFC7540]](https://tools.ietf.org/html/rfc7540#section-5.5) で定義されている Extending HTTP/2 を使用します。

具体的には、 `:protocol` 疑似ヘッダーフィールド の新規追加と、[Section 4](https://tools.ietf.org/html/draft-ietf-httpbis-h2-websockets-07#section-4) で定義されている `:authority` 疑似ヘッダーフィールドの意味変更のために、オプトインネゴシエーションを必要とします。

もしクライアントが、最初に SETTINGS_ENABLE_CONNECT_PROTOCOL パラメータを受信せずに、本書で定義されている拡張 CONNECT メソッドの規定を使用する場合、本書の規定をサポートしない通信相手は、不正な要求を検出し、ストリームエラーを生成します。（[Section 8.1.2.6 of [RFC7540]](https://tools.ietf.org/html/rfc7540#section-8.1.2.6)）


## 4. 拡張 CONNECT メソッド
HTTP/2 での CONNECT メソッドの使用方法は、[Section 8.3 of [RFC7540]](https://tools.ietf.org/html/rfc7540#section-8.3) で定義されています。 本書で定義されている拡張は、以下の方法でメソッドを変更します。

- 新しい擬似ヘッダーフィールド `:protocol` は、CONNECT によって生成されるトンネルで使用されるべき所望のプロトコルを示すために、要求 HEADERS に含めることができます。（ `MAY` ）擬似ヘッダーフィールドは単一の値で、https://www.iana.org/assignments/http-upgrade-tokens/http-upgrade-tokens.xhtml に定義されている HTTP Upgrade Token Registry の値を含みます。

- `:protocol` 擬似ヘッダーフィールドを含むリクエストでは、ターゲットURI（[Section 5](https://tools.ietf.org/html/draft-ietf-httpbis-h2-websockets-07#section-5) を参照）の `:scheme` および `:path` 擬似ヘッダーフィールドも含める必要があります。（ `MUST NOT` ）


- `:protocol` 擬似ヘッダーフィールドを含むリクエストでは、 `:authority` 擬似ヘッダーフィールドは [Section 8.3 of [RFC7540]](https://tools.ietf.org/html/rfc7540#section-8.3) ではなく [Section 8.1.2.3 of [RFC7540]](https://tools.ietf.org/html/rfc7540#section-8.1.2.3) に従って解釈されます。 特にサーバーは、従来の CONNECT メソッド要求と同様に、`:authority` で示されるホストへのトンネルを作成してはいけません。（ `MUST NOT` ）


`:protocol` 疑似ヘッダーフィールドを含む CONNECT 要求を受信すると、サーバーは疑似ヘッダフィールドによって示されるプロトコルタイプの別のサービスへのトンネルを確立します。 このサービスは、サーバーと同じ場所にある場合とない場合があります。


## 5. 拡張 CONNECT を使用して WebSocket をブートストラップする
`:protocol` 疑似ヘッダフィールドは、HTTP/2ストリーム上でWebSocket接続を開始するために、 CONNECT 要求に含まれなければならず（ `MUST` ）、`websocket` の値を持たなければなりません（ `MUST` ）。クッキー操作などのための、その他のHTTPリクエストヘッダーフィールドおよびレスポンスヘッダーフィールドは、通常通り、 CONNECT メソッドとともに HEADERS に含めることができます。 この要求は、 [[RFC6455](https://tools.ietf.org/html/rfc6455)] の GET ベースの要求に取って代わり、 WebSocket opening handshake 実行するために使用されます。


ターゲットURIのスキーム （[Section 5.1 of [RFC7230]](https://tools.ietf.org/html/rfc7230#section-5.1)）は、 `wss` スキームの WebSocket では `https` 、 `ws` スキームでは `http` でなければなりません（ `MUST` ）。ターゲットURIの残りの部分は、 websocket URI と同じになります。 websocket URI は、引き続きプロキシの自動設定に使用されます。 この仕様で使用される HTTP/2 コネクションのセキュリティ要件は、 https 要求の場合は[[RFC7540](https://tools.ietf.org/html/rfc7540)]、 http 要求の場合は[[RFC8164](https://tools.ietf.org/html/rfc8164)]によって確立されます。



[[RFC6455](https://tools.ietf.org/html/rfc6455)]では、 HTTP/2 の一部ではない `Connection` および `Upgrade` ヘッダーフィールドの使用を必要とします。ここで定義された CONNECT 要求には、それらを含めてはなりません（ `MUST NOT` ）。

[[RFC6455](https://tools.ietf.org/html/rfc6455)]は、 HTTP/2 の一部ではない Host ヘッダーフィールドの使用を必要とします。ホスト情報は、すべての HTTP/2 トランザクションで要求される `:authority` 擬似ヘッダーフィールドの一部として伝達されます。

この拡張 CONNECT を使用して WebSocket をブートする実装は、[[RFC6455](https://tools.ietf.org/html/rfc6455)] の `Sec-WebSocket-Key` および `Sec-WebSocket-Accept` ヘッダーフィールドの処理を行いません。その機能は `:protocol` 疑似ヘッダーフィールドによって置き換えられています。

[RFC6454]における `Sec-WebSocket-Version` 、 `Sec-WebSocket-Protocol` 、および `Sec-WebSocket-Extensions` ヘッダーフィールドは、[[RFC6455](https://tools.ietf.org/html/rfc6455)] で定義されているのと同じ方法で、CONNECTリクエストおよびレスポンスヘッダーフィールドで使用されます。HTTP/1ヘッダーフィールド名は大文字小文字を区別せませんが、HTTP/2 では小文字としてエンコードする必要があることに注意してください。

opening handshake が成功した後、送受信者は、CONNECT トランザクションによって生成した HTTP/2 ストリームを [[RFC6455](https://tools.ietf.org/html/rfc6455)] で言及されているTCPコネクションであるかのように使用して、 WebSocket プロトコルを開始します。この時点での WebSocket コネクションの状態は、 [[RFC6455], Section 4.1](https://tools.ietf.org/html/rfc6455#section-4.1) で定義されているように `OPEN` となります。

HTTP/2 ストリームの切断は、[[RFC6455](https://tools.ietf.org/html/rfc6455)] のTCPコネクション切断に類似しています。 規則的な TCP レベルの切断 は `END_STREAM` （[[RFC7540], Section 6.1]）フラグとして表され、`RST` 例外は エラーコード `CANCEL` （[[RFC7540], Section 7]）の `RST_STREAM` （[[RFC7540], Section 6.4]）フレームで表されます。





### 5.1 例

```
[[ From Client ]]                       [[ From Server ]]

                                        SETTINGS
                                        SETTINGS_ENABLE_CONNECT_[..] = 1

HEADERS + END_HEADERS
:method = CONNECT
:protocol = websocket
:scheme = https
:path = /chat
:authority = server.example.com
sec-websocket-protocol = chat, superchat
sec-websocket-extensions = permessage-deflate
sec-websocket-version = 13
origin = http://www.example.com

                                        HEADERS + END_HEADERS
                                        :status = 200
                                        sec-websocket-protocol = chat

DATA
WebSocket Data

                                        DATA + END_STREAM
                                        WebSocket Data

DATA + END_STREAM
WebSocket Data
```

## 6. 設計上の考慮事項
確かに、 HTTP/2 へより多くを追加することにより、ネイティブに HTTP/2 と WebSocket を統合することは可能です。この設計は、ソリューションの複雑さを最小限に抑えながら、 HTTP/2 と WebSocket を並行して実行するという主たる目的に対処するために選択されました。

## 7. 中間者について
このドキュメントは、 WebSocket と HTTPフォワードプロキシ との対話方法には変更を与えません。 WebSocket を使用したいクライアントが HTTP/2 を介して HTTPプロキシ に接続する場合、従来の（ `:protocol` 疑似ヘッダーフィールドを使用しない）CONNECT を使用して、そのプロキシをHTTP経由でWebSocketサーバにトンネリングする必要があります。

そのトンネル上のHTTPのバージョンによって、 WebSocket が直接開始されるか、またはこの文書で説明されている変更された CONNECT 要求を介して開始されるかが決定されます。


## 8. セキュリティ上の考慮事項
[[RFC6455](https://tools.ietf.org/html/rfc6455)] は、非 WebSocket クライアント、特に XMLHttpRequest ベースのクライアントが WebSocket 接続を確立できないことを保証します。 その主な仕組みは、 XMLHttpRequest ベースのクライアントでは作成できない、`Sec-` プレフィックスのついたリクエストヘッダーフィールドの使用です。本仕様は、以下の2つの方法で懸念事項に対処します。

- XMLHttpRequest は、`Spec-` プレフィックス付きのリクエストヘッダーフィールドに加えて、 CONNECT メソッドの使用も禁止しています。

- 擬似ヘッダーフィールドの使用は、コネクション固有のものであり、かつ、 HTTP/2 はプロトコルスタックの外部では作成できません。

[[RFC6455] section 10](https://tools.ietf.org/html/rfc6455#section-10) におけるセキュリティ上の考慮事項は、本仕様を使用した WebSocket プロトコル利用に対しても、Section 10.8 の例外を除いて引き続き適用されます。このセクション (Section 10.8) は、本書によって変更されたブートストラップ用ハンドシェイクに固有のものであるため、関係ありません。


## 9. IANA に関する考慮事項
この文書は、 [Section 11.3 of [RFC7540]](https://tools.ietf.org/html/rfc7540#section-11.3) によって定められた HTTP/2 Settings Registry にエントリーを追加します。

- Name: `SETTINGS_ENABLE_CONNECT_PROTOCOL`
- Code: `0x8`
- Initial Value: `0`
- Specification: This document


## 10. 参考文献
- [[RFC2119](https://www.rfc-editor.org/info/rfc2119)]
  -  Bradner, S., "Key words for use in RFCs to Indicate Requirement Levels",
  - BCP 14, RFC 2119,
  - DOI 10.17487/RFC2119, March 1997.
- [[RFC6454](https://www.rfc-editor.org/info/rfc6454)]
  - Barth, A., "The Web Origin Concept",
  - RFC 6454,
  - DOI 10.17487/RFC6454, December 2011.
- [[RFC6455](https://www.rfc-editor.org/info/rfc6455)]
  - Fette, I. and A. Melnikov, "The WebSocket Protocol",
  - RFC 6455,
  - DOI 10.17487/RFC6455, December 2011.
- [[RFC7230](https://www.rfc-editor.org/info/rfc7230)]
  - Fielding, R., Ed. and J. Reschke, Ed., "Hypertext Transfer Protocol (HTTP/1.1): Message Syntax and Routing",
  - RFC 7230,
  - DOI 10.17487/RFC7230, June 2014.
- [[RFC7540](https://www.rfc-editor.org/info/rfc7540)]
  - Belshe, M., Peon, R., and M. Thomson, Ed., "Hypertext Transfer Protocol Version 2 (HTTP/2)",
  - RFC 7540,
  - DOI 10.17487/RFC7540, May 2015.
- [[RFC8164](https://www.rfc-editor.org/info/rfc8164)]
  - Nottingham, M. and M. Thomson, "Opportunistic Security for HTTP/2",
  - RFC 8164,
  - DOI 10.17487/RFC8164, May 2017.
- [[RFC8174](https://www.rfc-editor.org/info/rfc8174)]
  - Leiba, B., "Ambiguity of Uppercase vs Lowercase in RFC 2119 Key Words",
  - BCP 14, RFC 8174,
  - DOI 10.17487/RFC8174, May 2017.


## 謝辞
The 2017 HTTP Workshop had a very productive discussion that helped determine the key problem and acceptable level of solution complexity.

## 著者のメールアドレス
Patrick McManus
Mozilla

Email: mcmanus@ducksong.com
