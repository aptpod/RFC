# Datagram Transport Layer Security Version 1.2

[RFC6347の原文](https://tools.ietf.org/html/rfc6347)

## Abstract

この文書はDatagram Transport Layer Security (DTLS) プロトコルのバージョン1.2を定義します。DTLSプロトコルはデータグラムプロトコルに通信の秘匿を提供するものです。このプロトコルはクライアント／サーバーアプリケーションが盗聴・改ざん・偽造を防ぐよう設計された方法で通信することを可能にします。DTLSプロトコルはTransport Layer security (TLS)プロトコルに基づくものであり、同等のセキュリティを提供します。トランスポート層の基礎であるデータグラムはDTLSプロトコルによって保護されます。この文書はTLSプロトコルのバージョン1.2とともに動作するためにDTLSプロトコルのバージョン1.0を更新するものです。

## Status of This Memo

This is an Internet Standards Track document.

This document is a product of the Internet Engineering Task Force (IETF).  It represents the consensus of the IETF community.  It has received public review and has been approved for publication by the Internet Engineering Steering Group (IESG).  Further information on Internet Standards is available in Section 2 of RFC 5741.

Information about the current status of this document, any errata, and how to provide feedback on it may be obtained at http://www.rfc-editor.org/info/rfc6347.

## Copyright Notice

Copyright (c) 2012 IETF Trust and the persons identified as the document authors.  All rights reserved.

This document is subject to BCP 78 and the IETF Trust's Legal Provisions Relating to IETF Documents (http://trustee.ietf.org/license-info) in effect on the date of publication of this document.  Please review these documents carefully, as they describe your rights and restrictions with respect to this document.  Code Components extracted from this document must include Simplified BSD License text as described in Section 4.e of the Trust Legal Provisions and are provided without warranty as described in the Simplified BSD License.

This document may contain material from IETF Documents or IETF Contributions published or made publicly available before November 10, 2008.  The person(s) controlling the copyright in some of this material may not have granted the IETF Trust the right to allow modifications of such material outside the IETF Standards Process. Without obtaining an adequate license from the person(s) controlling the copyright in such materials, this document may not be modified
outside the IETF Standards Process, and derivative works of it may not be created outside the IETF Standards Process, except to format it for publication as an RFC or to translate it into languages other than English.

## 1. Introduction

TLSはネットワークトラフィックを保護するためにもっとも広く利用されているプロトコルです。ウェブのトラフィックやIMAPやPOPなどのEメールのプロトコルを守るために広く使われています。TLSの主要な利点は透過なコネクション指向の通信路を提供することです。これにより、TLSをアプリケーション層とトランスポート層の間に入れることで、アプリケーションプロトコルを容易に保護することができます。しかしながら、TLSは信頼性のあるトランスポート層の通信路、一般的にはTCPの上で動作しなければなりません。ゆえに、信頼性のないデータグラムトラフィックを保護するためにTLSを用いることはできません。

ますます多くのアプリケーション層のプロトコルがトランスポート層にUDPを利用する設計を採用しています。特に、Session Initiation Protocol (SIP)や電子ゲームで利用されるプロトコルなどがより一般的になってきています。（SIPはTCPとUDPのどちらを用いても動作することが可能ですが、UDPがより好まれる状況があることを記しておきます。）最近では、これらのアプリケーションの設計者は不満の残る選択をすることを余儀無くされています。最初の選択肢はIPsecを利用することです。しかしながら、[WHYIPSEC]で触れられているいくつかの理由によって、この選択はいくつかの限られたアプリケーションのみに適したものです。次の選択肢はアプリケーション層の中でセキュリティを確保する設計をすることです。残念なことに、一般にアプリケーション層におけるセキュリティプロトコル（e.g., S/MIMEにおけるエンドツーエンドのセキュリティ）は優れたセキュリティ特性を持つものの、TLSの上でプロトコルを動作させるために必要な努力と比較すると、プロトコル設計において多大な努力が必要となることが少なくありません。

多くの場合、クライアント／サーバーアプリケーションを保護するためのもっとも望ましい方法はTLSを使用することです。しかしながら、データグラムを使用する場合には自動的にTLSの利用が不可能となります。この文書はこの目的のためのプロトコルであるDatagram Transport Layer Security (DTLS)を説明するものです。DTLSは、新たなセキュリティの発明を最小化し、また出来る限り多くの既存のコードインフラを再利用するために、できる限りTLSと似たものとなるよう慎重に設計されています。

DTLS 1.0は元々TLS 1.1からの差分として定義されました。この文書はDTLSの新しいバージョンであるDTLS 1.2を、TLS 1.2からの差分として定義します。DTLS 1.1は存在しません。TLSのバージョン番号と同調するために、DTLS 1.1はスキップされました。DTLS 1.2ではDTLS 1.0の仕様の中で混乱させるような点を明確にするものも含まれています。

TLS 1.2の実装がTLSの以前のバージョンと相互運用することが可能（詳細はTLS 1.2のAppendix E.1を参照）なのと同様に、DTLS 1.2とDTLS 1.0の両方に対応した実装は、DTLS 1.0だけに対応した実装と（もちろんDTLS 1.0を利用して）相互運用することが可能です。ただし、SSLv2やSSLv3に該当するDTLSのバージョンは存在しないため、これらのプロトコルとの後方互換性に関する問題は当てはまりません。

### 1.1 Requirements Terminology

この文書で使用されている "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", "OPTIONAL" の語はREC 2119で説明されているように解釈します。

## 2. Usage Model

DTLSプロトコルは通信しているアプリケーション間のデータを保護するために設計されています。アプリケーション空間で動作するよう設計され、カーネルの修正は必要ありません。

データグラムはデータの信頼性や到達順序を必要とすることも提供することもありません。DTLSプロトコルはペイロードデータについてこの特性を維持します。メディアストリーミングやインターネット電話、オンラインゲームなどのアプリケーションは、遅延に影響されやすいデータの特性のために、トランスポート層の通信にデータグラムを利用しています。このようなアプリケーションの振る舞いは、DTLSプロトコルが欠損データの再送制御や順序制御を提供することはないため、通信を保護するためにDTLSプロトコルを使用しても変化することはありません。

## 3. Overview of DTLS

DTLSの基本的な設計思想は「データグラムにおけるTLS」を構築することです。TLSを直接データグラムにおいて使用できない理由を端的に書くと、パケットが消失したりパケットの到達順序が変わってしまう可能性があることです。TLSはこれらの信頼性のなさを制御する能力を内包していません。それゆえ、TLSの実装はデータグラムに移植されると破綻します。DTLSの目的はこれらの問題を解決するために必要な最小限の変更をTLSに行うことです。出来る限り、DTLSはTLSと同一のものです。DTLSに新たな仕組みを発明する必要が発生した場合はいつも、TLSのスタイルを維持するような方法で行うことを試みます。

信頼性のないことはTLSに対して以下の2つの課題をもたらしています。

1. TLSは個々のレコードを独立して復号することができません。なぜならば整合性（完全性）チェックがシーケンス番号に依存しているからです。もしレコードNが受信されなかった場合、レコードN+1の整合性チェックは誤ったシーケンス番号に基づくものとなり、失敗します。（TLS 1.1より前のバージョンでは、明示的なIVフィールドは存在しなかったため、この場合も復号に失敗することを記しておきます）
2. TLSのハンドシェイクはハンドシェイクメッセージが確実に届くことを仮定しているため、メッセージが消失した場合には破綻します。

この章ではこれらの課題を解決するためにDTLSがどのように使われるかについて説明します。

### 3.1 Loss-Insensitive Messaging

（TLSレコード層と呼ばれる）TLSのトラフィック暗号化層において、レコードは独立ではありません。以下2種類のレコード間の依存があります。

1. 暗号の前後関係（ストリーム暗号の鍵系列）がレコード間で維持される。
2. アンチリプレイ機能とメッセージの順序保証機能はシーケンス番号を含むMACによって提供されるが、シーケンス番号はレコード内の暗黙的なものである。

DTLSは最初の課題をストリーム暗号を禁止することで解決します。また2つ目の課題を明示的なシーケンス番号を追加することで解決します。

### 3.2 Providing Reliability for Handshake

TLSのハンドシェイクは決まった方式で行う暗号化されたハンドシェイクです。メッセージは定義された順序で送受信されなければならず、他の順序で行われた場合はエラーとなります。明らかに、順序の変更やメッセージの消失とは相入れません。加えて、TLSのハンドシェイクのメッセージはデータグラムのサイズに対して大きい可能性があり、これはIPフラグメンテーションの問題を引き起こします。DTLSはこれらの課題の両方に対して解決策を提供しなければなりません。

#### 3.2.1 Packet Loss

DTLSはパケットロスを制御するために、シンプルな再送タイマーを利用します。以下の図はDTLSのハンドシェイクの最初の段階を用いて基本的な概念を説明しています。

```text
 Client                                   Server
 ------                                   ------
 ClientHello           ------>

                         X<-- HelloVerifyRequest
                                          (lost)

 [Timer Expires]

 ClientHello           ------>
 (retransmit)
```

クライアントはClientHelloメッセージを送信すると、サーバーからHelloVerifyRequestメッセージが返ってくることを期待します。しかしながら、もしサーバーからのメッセージが消失した場合には、クライアントはClientHelloかHelloVerifyRequestメッセージのどちらかが消失したことを知っていて、メッセージを再送します。サーバーが再送されたメッセージを受信したとき、サーバーはメッセージを再送することを知っています。サーバーも再送タイマーを持っており、タイムアウトしたときに再送を行います。

サーバー上に状態を作ることを必要とするために、タイムアウトと再送はHelloVerifyRequestには適用されないことを記しておきます。HelloVerifyRequestはそれ自身が断片化されないよう十分に小さくなるよう設計されており、これによって複数のHelloVerifyRequestをインターリーブする懸念を避けています。

#### 3.2.2 Reordering

DTLSにおいて、それぞれのハンドシェイクメッセージはハンドシェイク内の特定のシーケンス番号を割り当てられます。ピアがハンドシェイクメッセージを受け取ったとき、そのメッセージが次に来ることを期待されているメッセージであるかどうかを速やかに判定することができます。もし期待のものであれば、メッセージを処理します。異なる場合は、そのメッセージより前の全てのメッセージを受け取った後に処理するためにキューに登録します。

#### 3.2.3 Message Size

TLSとDTLSのハンドシェイクメッセージは（理論的には2^24-1 バイトまで、実情では数キロバイト程度に）極めて大きくなる可能性があります。対して、UDPのデータグラムはしばしばIPフラグメンテーションが望まれない位場合には1500バイト以下に制限されます。この制限を補うために、DTLSのハンドシェイクメッセージはいくつかのDTLSレコードに断片化されることがあります。DTLSのハンドシェイクメッセージはfragment offsetとfragment lengthの両方を含みます。このようにして、ハンドシェイクメッセージの全てのバイトを所有する受け取り側はもともとの断片化されていないメッセージを再構成することができます。

### 3.3 Replay Detection

DTLSはオプションとしてレコードのリプレイ検知をサポートします。使用されるテクニックはIPsec AH/ESPと同じ、受信したレコードのbitmap windowを保持しておくものです。古すぎてウィンドウに含まれないレコードやすでに受信済みのレコードは無言で捨てられます。パケットの重複は必ずしも悪意のあるものではなくルーティングエラーによっても起こりうるため、リプレイ検知機能はオプションです。アプリケーションはおそらく重複したパケットを検知し、それに応じてデータの伝送方法をを修正することができます。

## 4. Differences from TLS

3章で言及されているように、DTLSは意図的にTLSととても似ています。そのため、DTLSを新しいプロトコルとして説明する代わりに、TLS 1.2からの一連の差分として表現します。明示的に違いを記載しているところを除いて、DTLSはTLS 1.2と同じです。

### 4.1 Record Layer

DTLSのレコード層はTLS 1.2のものと極めて類似しています。唯一の変更はレコード内に明示的なシーケンス番号が含まれていることです。このシーケンス番号は受け取り側が正確にTLS MACを検証できるようにします。DTLSのレコードのフォーマットは以下の通りです。

```text
   struct {
        ContentType type;
        ProtocolVersion version;
        uint16 epoch;                                    // New field
        uint48 sequence_number;                          // New field
        uint16 length;
        opaque fragment[DTLSPlaintext.length];
      } DTLSPlaintext;
```

- type
    - TLS 1.2におけるtypeと同様。
- version
    - 使用されているプロトコルのバージョン。この文書はDTLSのバージョン1.2について説明しており、versionは { 254, 253 } を使用します。この 254.253 の値はDLTSのバージョン1.2の1の補数表現です。TLSとDTLSのバージョン番号の最大限の間隔は、2つのプロトコルのレコードを容易に識別できることを保証します。（実際のバージョン番号は数値の上で増加していくのに対して、）将来のDTLSのメッセージ上で使われるversionの値は減少していくことを記しておきます。
- epoch
    - cipher stateが変更するごとに増分されるカウンタ値。
- sequence_number
    - このレコードのシーケンス番号。
- length
    - TLS 1.2におけるlengthと同じ。TLS 1.2において、lengthは2^14 を超えない方がよい。
- fragment
    - TLS 1.2におけるfragmentと同じ。

DTLSは暗黙的なものではなく、レコード内のsequence_numberで伝達される明示的なシーケンス番号を使用します。シーケンス番号は、それぞれのepochでsequence_numberが0に初期化され、それぞれのepochで別々に維持されます。例えば、epoch 0のハンドシェイクメッセージが再送された場合、epoch 1のメッセージが先に送られたとしても、epoch 1のメッセージより後のシーケンス番号を持っている可能性があります。ハンドシェイクの間、再送されたメッセージが正しいepochやその他の重要な要素を使用できるように、注意を払う必要があることを記しておきます。

いくつかのハンドシェイクが連続して実施された場合、同じシーケンス番号で異なるcipher stateの複数のレコードが同時に伝送される可能性があります。epochの値を持つことで、受け取り側がそのようなパケットを識別することが可能になります。epochの値は0で初期化され、ChangeCipherSpecメッセージが送られるごとに増分されます。あらゆるsequence_number / epochの組み合わせがユニークであることを保証するために、実装はTCPの最大セグメント生存期間の2倍の期間のあいだに同じepochの値を再利用してはなりません（MUST NOT）。実際問題としては、TLSの実装が再度ハンドシェイクを行うことは稀であり、それゆえこれは問題とならないと思われます。

DTLSのレコードは順序逆転されうるため、epoch 2が始まった後にepoch 1のレコードを受け取る可能性があることを記しておきます。一般に、実装は以前のepochのパケットを捨てる必要がある（SHOULD）のに対し、もしパケットロスが重要な問題を引き起こす場合には、パケットの順序制御を行うために、TCPの仕様にある標準の最大セグメント生存期間 (MSL)まで、以前のエポックの重要な要素を維持することを選択してもよい（MAY）です。（この記述の意図は、実装者がシステムのTCPスタックの使用しているMSLを尋ねることを試みるのではなく、MSLに関するIETFからの現在のガイドラインを使用することです。）ハンドシェイクが完了するまで、実装は過去のepochのパケットを受け入れなければなりません（MUST）。

逆に、新たにネゴシエートされたコンテキストで保護されたレコードを、ハンドシェイクの完了前に受信することは可能です。例えば、サーバー側はFinishedメッセージの送信後にデータの送信を開始してもよいです。DTLSが信頼性のあるトランスポート（e.g., SCTP）の上で使われていたとしても、実装はそのようなパケットをバッファしても捨ててもよく（MAY）、いったんハンドシェイクが完了すればバッファし処理する必要があります（SHOULD）。パケットがいつ送られうるかのTLSの制限は依然として適用され、受け取り側がパケットを正しい順番で送信されたかのように扱うことを記しておきます。特に、最初のハンドシェイクの完了前にデータを送ることは許容されません。

すでに確立された接続における再度のハンドシェイクが、確立されたセッションを再開するか、すでに確立された接続と全く同じセキュリティパラメータを使用するようなの場合には、ChangeCipherSpecやFinishedメッセージをまだ受け取ってなくても、データパケットを即時に処理することは安全だと記しておきます。
そのほかの場合では、実装はダウングレード攻撃を防止するためにFinishedメッセージの受信を待たなくてはなりません（MUST）。

TLSと同様に、シーケンス番号が一周することを許容する前に、実装は接続を破棄するか再度のハンドシェイクを行わなければなりません（MUST）。

同様に、実装はepochが一周することを許容してはならず（MUST NOT）、代わりに4.2.8のセクションで説明されているように古い接続を終えて新たな接続を確立しなければなりません（MUST）。

#### 4.1.1 Transport Layer Mapping

DTLSのレコードは1つのデータグラム内に含まれなければなりません（MUST）。IPフラグメンテーションを避けるために、DTLSのレコード層のクライアントは、レコード層から取得したPMTUの推定値に含めるためにレコードの大きさを設定するように試みる必要があります（SHOULD）。

IPsecとは異なり、DTLSのレコードは接続の識別子を持たないことを記しておきます。アプリケーションは接続間の多重送信に備える必要があります。UDPにおいては、おそらくホスト／ポート番号によって行われます。

複数のDTLSレコードを単一のデータグラムに配置してもよいです。それらは単純に連続して暗号化されます。DTLSのレコードの構成は境界を決めるのに十分な情報を持っています。しかしながら、データグラムのペイロードの最初のバイトはレコードの冒頭でなければならないことを記しておきます。レコードは複数のデータグラムにまたがることはできません。

DCCPのようないくつかのトランスポート層のプロトコルは自身のシーケンス番号を持っています。これらのトランスポート層の上で使用される場合、DTLSとそのトランスポート層のプロトコルの両方にシーケンス番号が存在することになります。これは少量の非効率となりますが、トランスポート層とDTLSのシーケンス番号は異なる目的のために使用されるため、概念の単純さのために、両方のシーケンス番号を使用することがより良いです。将来的に、制約のある環境での利用のために、単一のシーケンス番号のみを使用するようなDLTSへの拡張が定義される可能性があります。

DCCPのようないくつかのトランスポート層のプロトコルは、輻輳制御機能を持っています。輻輳ウィンドウが十分に狭い場合、DTLSハンドシェイクの再送メッセージは即時に送られるのではなく保留される可能性があり、潜在的にタイムアウトや不要な再送（spurious retransmission）を引き起こす可能性があります。DTLSがそのようなトランスポート層の上で使用されるときには、それらしい輻輳ウィンドウを越えることのないよう注意を払う必要があります。DCCPDTLSはこれらの課題を考慮したDTLSのDCCPへのマッピングについて定義しています。

##### 4.1.1.1 PMTU Issues

一般に、DTLSの方針はアプリケーションにPMTU discoveryを任せるものです。しかしながら、DTLSは3つの理由で完全にはPMTUを無視することができません。

- DTLSのレコードの構成がデータグラムのサイズを超えることによって、アプリケーションから見た有効なPMTUを下回るため
- いくつかの実装において、アプリケーションが直接ネットワークと通信することができず、このときDTLSのスタックがICMPの"Datagram Too Big"の指摘やICMPv6の"Packet Too Big"の指摘を吸収してしまうため
- DTLSのハンドシェイクメッセージがPMTUを超えうるため

最初の2つの課題に対処するために、DLTSのレコード層は以下の説明のように振舞う必要があります（SHOULD）。

もしPMTUの推定値が下のトランスポート層のプロトコルから取得可能であれば、上の層のプロトコルに提供可能にするべきです。特に、

- DTLS over UDPの場合、上の層のプロトコルはIP層で持っているPMTUの推定値を取得可能にする必要があります（SHOULD）
- DTLS over DCCPの場合、上の層のプロトコルは現在のPMTUの推定値を取得可能にする必要があります（SHOULD）
- DTLS over TCP / SCTPの場合、データグラムの断片化と再構成は自動的に行われるため、PMTUの制限は存在しません。しかしながら、上の層のプロトコルは最大レコードサイズである2^14 バイトを超えるレコードを作成してはなりません（MUST NOT）

DTLSのレコード層は上の層のプロトコルがDTLSの処理に期待されるレコードの拡張量を分かるようにする必要があります（SHOULD）。block paddingや潜在的なDTLSの圧縮の利用のために、この値は推定値であることを記しておきます。

（ICMPまたはDCCPの14章に記載のあるデータグラム送信の拒絶によって）トランスポート層の指摘がある場合、DTLSのレコード層は上の層のプロトコルにそのエラーについて伝えなければなりません（MUST）。

RFC1191またはRFC4821の仕組みによって、DTLSのレコード層はPMTU discoveryを行う上の層のプロトコルと干渉しないほうがよいです（SHOULD）。特に、

- 下のトランスポート層のプロトコルによって許可された場合には、上の層のプロトコルは（IPv4における）DFビットの状態を設定するか（IPv6における）ローカルフラグメンテーションを禁止することを可能にする必要があります
- もしアプリケーションがPMTUプローブを要求することを下のトランスポート層のプロトコル（e.g., DCCP）が許可している場合、DTLSのレコード層はこの要求を守る必要があります

最後の課題はDTLSのハンドシェイクプロトコルです。DTLSのレコード層から見て、これは単にある別の上の層のプロトコルです。しかしながら、DTLSのハンドシェイクの頻度は低く、ハンドシェイクに含まれるラウンドトリップもほんの数回です。それゆえ、ハンドシェイクプロトコルのPMTUハンドリングは精度の高いPMTU discoveryよりも迅速な完了の優先順位を第一としています。このような状況において接続を可能とするために、DTLSの実装は以下の規定に従う必要があります（SHOULD）。

- もしDTLSのレコード層がDTLSのハンドシェイク層にメッセージが大きすぎると伝えた場合、PMTUについて既知の情報を用いて、すみやかに断片化を試みる必要があります（SHOULD）
- もし繰り返しの再送を行ってもレスポンスがなく、かつPMTUが分からない場合、その後の再送はハンドシェイクメッセージを適切に断片化したより小さなレコードサイズに引き下げられる必要があります（SHOULD）。この引き下げまでの再送の試行回数の標準は明記しませんが、2-3回が適切に思われます

#### 4.1.2 Record Payload Protection

TLSのように、DTLSはデータを一連の保護されたレコードとして伝達します。このセクションではそのフォーマットの詳細について説明します。

##### 4.1.2.1 MAC

DTLSのMACはTLS 1.2のものと同じです。しかしながら、MACを計算するために必要なシーケンス番号は、TLSの暗黙的なシーケンス番号ではなく、伝達されるメッセージに含まれるepochとシーケンス番号を結合した64bitの値が使用されます。DTLSのepochとシーケンス番号を結合したものは、TLSのシーケンス番号と同じ長さになることを記しておきます。

TLSのMACの計算はプロトコルのバージョン番号でパラメータ化されます。DTLSの場合、メッセージに含まれるバージョン番号、すなわち、DTLS 1.2では {254, 253}です。

DTLSとTLSのMACの処理の重症な違いの1つは、TLSではMACのエラーの場合には接続を終了しなければならないことです。DTLSでは、受け取り側の実装は単に問題のあったレコードを捨て、接続を続けてもよいです（MAY）。この違いは、DTLSのレコードはTLSのレコードであるようなレコード間の依存関係がないことによって可能になっています。

一般に、DTLSの実装は正しくないMACのレコードやその他の点で無効なレコードは無言で捨てる必要があります（SHOULD）。エラーのログを残してもよいです（MAY）。もしDTLSの実装が正しくないMACのメッセージを受信した際にアラートを生成することを選択した場合、fatalのレベルでbad_record_macのアラートを生成し、接続状態を終了しなければなりません（MUST）。エラーが接続の終了をもたらさないために、DTLSのスタックはTLSのスタックと比較してより効率的なerror type oraclesです。したがって、TLS 1.2の6.2.3.2のセクションに記載の勧告に従うことが特に重要です。

##### 4.1.2.2 Null or Standard Stream Cipher

DTLSのNULL暗号はTLS 1.2のNULL暗号と完全に同様に動作します。

TLS 1.2に説明されている唯一のストリーム暗号はRC4であるが、RC4はランダムアクセスすることができません。RC4はDTLSで使用してはなりません（MUST NOT）。

##### 4.1.2.3 Block Cipher

DTLSのブロック暗号による暗号化と復号はTLS 1.2と完全に同様に動作します。

##### 4.1.2.4 AEAD Ciphers

TLS 1.2は認証付き（AEAD）暗号スイートを導入した。ECCGCMとRSAGCMで定義された既存のAEAD暗号スイートはTLS 1.2と完全に同様にDTLSでも使用することができます。

##### 4.1.2.5 New Cipher Suites

登録にあたり、新たなTLSの暗号スイートはDTLSの用途でも使用可能であるかと、もしあるならば必要な改造が何であるかを示さなければなりません（MUST）（IANAの考慮については7章を参照してください）。

##### 4.1.2.6 Anti-Replay

DTLSのレコードはリプレイアタックへの防御のためにシーケンス番号を含んでいます。シーケンス番号の検証は、ESPの3.4.3のセクションから借用した以下のスライディングウィンドウの制御手順を用いて、行われる必要があります（SHOULD）。

セッションにおける受け取り側のパケットカウンタは、セッション確立時に0に初期化されなければなりません（MUST）。受け取ったそれぞれのレコードに対して、受け取り側はレコードがセッションの生存中に受信したほかのどのレコードとも重複しないシーケンス番号を含んでいることを検証しなければなりません（MUST）。重複したレコードの排除の速度を早めるために、これはセッションを確立した後のパケットに対し実施される最初の確認である必要があります（SHOULD）。

重複は受信側のスライディングウィンドウを用いて排除されます。（ウィンドウがどのように実装されているかは局所的な問題ですが、以下の文章は実装が備えているべき機能性について説明しています。）最小のウィンドウサイズである32はサポートされなければなりません（MUST）が、64がより好ましく、また64がデフォルト値として使用される必要があります（SHOULD）。（最小よりも大きい）その他のウィンドウサイズが受け取り側によって選ばれてもよいです。（受け取り側はウィンドウサイズについて送り側には通知しません。）

ウィンドウの"右"端は、セッションで受信された検証済みの最も大きなシーケンス番号の値を表します。ウインドウの"左"端よりも小さなシーケンス番号を持ったレコードは排除されます。ウィンドウの範囲内のパケットが受信されると、ウィンドウに含まれる受け取り済みのパケットのリストに対して確認が行われます。この確認を行う効率的な方法の1つは、ESPの3.4.3のセクションで記載されているビットマスクに基づくものです。

もし受信したレコードがウィンドウの範囲内で新規のものであるか、パケットがウィンドウよりも右側のものである場合、受け取り側はMACの検証に進みます。もしMACの検証に失敗した場合は、受け取り側はレコードを正しくないものとして処分しなければなりません（MUST）。受信側のウィンドウはMACの検証に成功した場合にのみ更新されます。

##### 4.1.2.7 Handling Invalid Records

TLSと違い、DTLSは正しくないレコード（e.g., 正しくないフォーマット、長さ、MACなど）に直面しても強靭です。一般に、正しくないレコードは無言で捨てられ、したがって接続が維持される必要があります（SHOULD）。しかしながら、エラーは診断目的のためにログに残されても良いです（MAY）。代わりにアラートを生成することを選択する実装は、様々な種類の誤りに対して実装がどのように反応するか確認するために攻撃者が繰り返し実装を調査するような攻撃を避けるために、fatalレベルのアラートを生成しなければなりません(MUST)。DTLS over UDPの場合、UDPの偽造が容易であるために、この挙動を行うどのような実装もDoSアタックに極端に脆いであろうことを記しておきます。したがって、この方針はそのようなトランスポート層のプロトコルに対して推奨されません（NOT RECOMMENDED）。

偽造に対して耐性のあるトランスポート層のプロトコル（e.g., SCTP with SCTP-AUTH）の上でDTLSが使用される場合、攻撃者がトランスポート層に拒絶されないようなデータグラムを偽造することは難しいため、アラートを送ることはより安全です。

### 4.2 The DTLS Handshake Protocol

DTLSは、以下の3つの主要な違いのほかは全てTLSと同じハンドシェイクメッセージとフローを利用します。

1. DoS攻撃を防ぐために、ステートレスのcookie交換が追加されている
2. メッセージの消失、順序逆転、（IPフラグメンテーションを避けるための）DTLSメッセージのフラグメンテーションを制御するためのハンドシェイクメッセージのヘッダーへの変更
3. メッセージの消失を制御するための再送タイマー

これらの例外をのぞいて、DTLSのメッセージ構造、フロー、ロジックはTLS 1.2と同じものです。

#### 4.2.1 Denial-of-Service Countermeasures

データグラムのセキュリティプロトコルは様々なDoS攻撃に対して極めて脆いです。以下2つの攻撃は特に懸念となります。

1. サーバーにステートを割り当てさせたり、重い暗号処理を潜在的に実施させるような一連のハンドシェイク開始リクエストを送信することで、攻撃者はサーバ上の過度のリソースを消費することができます。
2. 犠牲者の送信元を偽装した接続開始メッセージを送ることで、攻撃者はサーバーを増幅器として使用することができます。サーバーはその次のメッセージ（DLTSでは、とても大きなサイズとなりうるCertificateメッセージ）を犠牲者のマシンに送り、溢れさせることとなります。

これらの攻撃の両方に対抗するために、DTLSはPhoturisやIKEで使用されているステートレスなcookieの手法を取り入れています。クライアントがClientHelloメッセージをサーバーに送る時、サーバーはHelloVerifyRequestメッセージをレスポンスとしてもよいです（MAY）。このメッセージはPHOTURISの手法を用いて生成されたステートレスなcookieを含んでいます。クライアントはClientHelloメッセージにこのcookieを追加して再送しなければなりません（MUST）。サーバーはcookieを検証し、それが正しいものであった場合のみハンドシェイクを進めます。この手順は攻撃者／クラアントにcookieが受信できることを強制し、これによりなりすましのIPアドレスによるDoS攻撃を困難にします。この手順は正しいIPアドレスから行われるDoS攻撃に対しては防御となりません。

このメッセージの取り交わしは以下の通りです。

```text
   Client                                   Server
   ------                                   ------
   ClientHello           ------>

                         <----- HelloVerifyRequest
                                (contains cookie)

   ClientHello           ------>
   (with cookie)

   [Rest of handshake]
```

したがって、DTLSはClientHelloメッセージにcookieの値を追加する変更を行います。

```text
struct {
  ProtocolVersion client_version;
  Random random;
  SessionID session_id;
  opaque cookie<0..2^8-1>;                             // New field
  CipherSuite cipher_suites<2..2^16-1>;
        CompressionMethod compression_methods<1..2^8-1>;
} ClientHello;
```

最初のClientHelloを送信するとき、クライアントはまだcookieを持っていません。この場合、cookieのフィールドは空（長さ0）にしておきます。

HelloVerifyRequestの定義は以下の通りです。

```text
struct {
  ProtocolVersion server_version;
  opaque cookie<0..2^8-1>;
} HelloVerifyRequest;
```

HelloVerifyRequestのメッセージタイプはhello_verify_request(3)です。

server_versionのフィールドはTLSと同じシンタックスです。しかしながら、最初のハンドシェイクにおけるバージョンネゴシエーションの要求を回避するために、DTLS 1.2のサーバー側実装はネゴシエートされることを期待するTLSのバージョンに関わらずDTLSバージョン1.0を使用する必要があります（SHOULD）。バージョンネゴシエーションの一部としてではなく、（DTLS 1.2と1.0の両方で同じである）パケットの構造を示すために、DLTS 1.2と1.0のクライアントはバージョン1.0だけを使用しなければなりません（MUST）。特にDTLS 1.2のクライアントは、サーバーがHelloVerifyRequestメッセージの中でバージョン1.0を使っていることを理由に、サーバーがDTLS 1.2ではない、最終的にDTLS 1.2ではなくDTLS 1.0でネゴシエートされるであろうなどの想定をしてはなりません（MUST NOT）。

HelloVerifyRequestにレスポンスするとき、クライアントは元々のClientHelloで使用したのと同じパラメータの値（version, random, session_id, cipher_suites, compression_method）を使用しなければなりません（MUST）。サーバーはこれらの値をcookieの生成に使用し、また受け取ったcookieの値によってそれらが正しいことを検証する必要があります（SHOULD）。サーバーはServerHelloを送信する際に使用されるであろうものと同じバージョン番号をHelloVerifyRequest内で使用しなければなりません（MUST）。複数のHelloVerifyRequestでのシーケンス番号の重複を避けるために、サーバーはClientHelloのレコードシーケンス番号をHelloVerifyRequestのレコードシーケンス番号として使用しなければなりません（MUST）。

注釈：この仕様はcookieのサイズ制限を将来的な柔軟性を大きくするために255バイトまで拡張します。以前のバージョンのDTLSにおける制限は32バイトのままです。

DTLSサーバーはサーバー上にクライアントごとのステートを保持することなく検証できるような方法でcookieを生成する必要があります（SHOULD）。1つの方法は、ランダムに生成されたSecretを保持し、以下の方法で生成するものです。

```text
   Cookie = HMAC(Secret, Client-IP, Client-Parameters)
```

2回目のClientHelloを受信したとき、サーバーはcookieが正しいものであり、かつクライアントが所与のIPアドレスでパケットを受信できることを検証できます。複数回のcookieの取り交わしにおけるシーケンス番号の重複を避けるために、サーバーはClientHelloのレコードシーケンス番号を最初のServerHelloのレコードシーケンス番号として使用しなければなりません（MUST）。それに続くServerHelloメッセージはサーバーがステートを生成したのちに送信され、正常にシーケンス番号を増分しなければなりません（MUST）。

この仕組みにおける潜在的な攻撃方法の1つは、攻撃者がいくつかの異なるアドレスからcookieを収集し、それをサーバーを攻撃するために再利用するものです。サーバーはこの攻撃に対し、Secretの値を頻繁に変更し、これらのcookieを無効とすることで防御することが可能です。正当なクライアントがこの移り変わり（e.g., クライアントがSecret 1を用いたcookieを受け取り、サーバーがSecret 2に変更された後に2回目のClientHelloを送るようなケース）の中でもハンドシェイクできることをサーバーが望む場合には、サーバーは両方のsecretを許容するような有限のウィンドウを持つことが可能です。IKEv2はこのようなケースを検知するためにcookieにバージョン番号を追加することを提案しています。もう1つのアプローチは単純に両方のSecretで検証を試すものです。

DTLSサーバーは新たなハンドシェイクが実施されたときは常にcookieの取り交わしを行う必要があります（SHOULD）。増幅攻撃が問題とならないような環境でサーバーが利用される場合には、サーバーはcookieの取り交わしを実施しない設定としてもよいです（MAY）。しかしながら、デフォルトの設定はcookieの取り交わしが実施されるものである必要があります（SHOULD）。加えて、セッションが再開されるときにcookieの取り交わしを行わないことをサーバーが選択してもよいです（MAY）。クライアントはすべてのハンドシェイクにおいてcookieの取り交わしの準備をしなければなりません（MUST）。

HelloVerifyRequestメッセージが使用された場合、最初のClientHelloとHelloVerifyRequestは（CertificateVerifyメッセージのための）handshake_messegesと（Finishedメッセージのための）verify_dataの計算には含まれません。

サーバーが正しくないcookieを含んだClientHelloメッセージを受信した場合、cookieを含まないClientHelloメッセージと同様に取り扱う必要があります（SHOULD）。これはクライアントが何らかの方法で誤ったcookieを受け取った場合に（e.g., サーバーがcookieの署名に使用する鍵を変更するために）競合／デッドロック状態を避けることになります。

実装者への注釈：これはクライアントが異なるcookieを含んだ複数のHelloVerifyRequestメッセージを受信することとなる可能性があります。クライアントは最新のHelloVerifyRequestへのレスポンスとしてcookieを含んだ新たなClientHelloメッセージを送信することで対処する必要があります（SHOULD）。

#### 4.2.2 Handshake Message Format

メッセージの消失、順序逆転、フラグメンテーションに対応するために、DTLSはTLS 1.2のハンドシェイクのヘッダーに変更を加えています。

```text
struct {
  HandshakeType msg_type;
  uint24 length;
  uint16 message_seq;                               // New field
  uint24 fragment_offset;                           // New field
  uint24 fragment_length;                           // New field
  select (HandshakeType) {
    case hello_request: HelloRequest;
    case client_hello:  ClientHello;
    case hello_verify_request: HelloVerifyRequest;  // New type
    case server_hello:  ServerHello;
    case certificate:Certificate;
    case server_key_exchange: ServerKeyExchange;
    case certificate_request: CertificateRequest;
    case server_hello_done:ServerHelloDone;
    case certificate_verify:  CertificateVerify;
    case client_key_exchange: ClientKeyExchange;
    case finished: Finished;
  } body;
} Handshake;
```

それぞれのハンドシェイクにおいて、クライアントとサーバーの双方が送信する最初のメッセージは常にmessage_seq = 0です。新たなメッセージが生成されたときは常に、messege_seqの値は1ずつ増分されます。再度のハンドシェイクの場合には、HelloRequestのmessege_seqの値は0と、ServerHelloのmessage_seqの値は1となることを意味することを記しておきます。メッセージが再送されるときは、同じmessege_seqの値が使用されます。以下は一例です。

```text
      Client                             Server
      ------                             ------
      ClientHello (seq=0)  ------>

                              X<-- HelloVerifyRequest (seq=0)
                                              (lost)

      [Timer Expires]

      ClientHello (seq=0)  ------>
      (retransmit)

                           <------ HelloVerifyRequest (seq=0)

      ClientHello (seq=1)  ------>
      (with cookie)

                           <------        ServerHello (seq=1)
                           <------        Certificate (seq=2)
                           <------    ServerHelloDone (seq=3)

      [Rest of handshake]
```

しかしながら、DTLSのレコード層の観点からは、再送は新たなレコードであることを記しておきます。このレコードは新たなTLSPlaintext.sequence_numberの値をもつこととなります。

DTLSの実装は（少なくとも名目上は）next_receive_seqカウンタを維持します。このカウンタは最初にゼロにセットされます。メッセージを受信したとき、そのメッセージのシーケンス番号がnext_receive_seqの値と一致した場合には、next_receive_seqの値は増分され、メッセージは処理されます。シーケンス番号がnext_receive_seqの値よりも小さい場合には、メッセージは廃棄されなければなりません（MUST）。シーケンス番号がnext_receive_seqの値よりも大きい場合には、実装はメッセージを待ち行列に入れる必要があります（SHOULD）が、廃棄することもできます（MAY）。（これは単純なメモリ領域と帯域のトレードオフです。）

#### 4.2.3 Handshake Message Fragmentation and Reassembly

4.1.1のセクションで記したように、DTLSのメッセージはトランスポート層の単一のデータグラムの中に含めなければなりません（MUST）。しかしながら、ハンドシェイクメッセージは最大のレコードサイズよりも大きい可能性があります。それゆえ、DTLSはハンドシェイクメッセージをそれぞれを別々に送信できるようないくつかのレコードに断片化することでIPフラグメンテーションを回避する仕組みを提供しています。

ハンドシェイクメッセージを送信する際に、送り側はメッセージを一連のN個の連続したデータの断片の並びに分割します。これらの断片は最大のhandshake fragment sizeよりも大きくてはならず（MUST NOT）、かつ共同してハンドシェイクメッセージ全体を含んでいなければいけません（MUST）。断片間でデータの範囲は重複しないほうがよいです（SHOULD NOT）。それから、送り側は元々のハンドシェイクメッセージと同じmessage_seqの値を持ったN個のハンドシェイクメッセージを作成します。作成されたメッセージはfragment_offset（これより前の断片に含まれていたデータのバイト数）とfragment_length（この断片の長さ）の値を付与されます。lengthのフィールドの値は元々のメッセージのlengthのフィールドの値と同じです。断片化を解消したメッセージはfragment_offsetの値が0でfragment_lengthの値がlengthの値と等しい縮退ケースです。

DTLSの実装がハンドシェイクメッセージの断片を受信したとき、ハンドシェイクメッセージ全体を保有するまでそれをバッファしなければなりません（MUST）。DTLSの実装は断片の範囲が重複しているものを取り扱うことができなくてはなりません（MUST）。これにより、送り側はPMTUの推定値が変化した場合により小さなフラグメントサイズでハンドシェイクメッセージを再送できるようになります。

TLSと同様に、空間がありかつ同じflightの一部である限り、複数のハンドシェイクメッセージを同じDTLSレコードに配置することが可能であることを記しておきます。このように、2つのDTLSメッセージを同じデータグラムに詰めるための2つの許容される方法があります。1つは同じレコード、もう1つは別のレコードです。

#### 4.2.4 Timeout and Retransmission

以下の図にしたがって、DTLSメッセージは一連のmessage flightに分類されます。それぞれのflightはいくつかのメッセージを含んでいる可能性がありますが、タイムアウトや再送の目的のためにモノリシックなものとしてみなす必要があります。

```text
Client                                          Server
------                                          ------

ClientHello             -------->                           Flight 1

                        <-------    HelloVerifyRequest      Flight 2

ClientHello             -------->                           Flight 3
                                           ServerHello    \
                                          Certificate*     \
                                    ServerKeyExchange*      Flight 4
                                   CertificateRequest*     /
                        <--------      ServerHelloDone    /

Certificate*                                              \
ClientKeyExchange                                          \
CertificateVerify*                                          Flight 5
[ChangeCipherSpec]                                         /
Finished                -------->                         /

                                    [ChangeCipherSpec]    \ Flight 6
                        <--------             Finished    /

            Figure 1. Message Flights for Full Handshake
```

```text
Client                                           Server
------                                           ------

ClientHello             -------->                          Flight 1

                                           ServerHello    \
                                    [ChangeCipherSpec]     Flight 2
                         <--------             Finished    /

[ChangeCipherSpec]                                         \Flight 3
Finished                 -------->                         /

      Figure 2. Message Flights for Session-Resuming Handshake
                        (No Cookie Exchange)
```

DTLSは以下のステートマシンに従って単純なタイムアウトと再送の仕組みを使用しています。DLTSクライアントが最初のメッセージ（ClientHello）を送信するために、クライアントはRREPARING状態から開始します。DTLSサーバーはWAITING状態で始まりますが、bufferは空で再送タイマーもありません。

```text
                   +-----------+
                   | PREPARING |
             +---> |           | <--------------------+
             |     |           |                      |
             |     +-----------+                      |
             |           |                            |
             |           | Buffer next flight         |
             |           |                            |
             |          \|/                           |
             |     +-----------+                      |
             |     |           |                      |
             |     |  SENDING  |<------------------+  |
             |     |           |                   |  | Send
             |     +-----------+                   |  | HelloRequest
     Receive |           |                         |  |
        next |           | Send flight             |  | or
      flight |  +--------+                         |  |
             |  |        | Set retransmit timer    |  | Receive
             |  |       \|/                        |  | HelloRequest
             |  |  +-----------+                   |  | Send
             |  |  |           |                   |  | ClientHello
             +--)--|  WAITING  |-------------------+  |
             |  |  |           |   Timer expires   |  |
             |  |  +-----------+                   |  |
             |  |         |                        |  |
             |  |         |                        |  |
             |  |         +------------------------+  |
             |  |                Read retransmit      |
     Receive |  |                                     |
        last |  |                                     |
      flight |  |                                     |
             |  |                                     |
            \|/\|/                                    |
                                                      |
         +-----------+                                |
         |           |                                |
         | FINISHED  | -------------------------------+
         |           |
         +-----------+
              |  /|\
              |   |
              |   |
              +---+

           Read retransmit
        Retransmit last flight

       Figure 3. DTLS Timeout and Retransmission State Machine
```

ステートマシンは3つの基本的な状態を持ちます。

PREPARING状態では、実装は次のメッセージflightの準備ために必要なすべての計算を行います。それから（まずバッファを空にしてから）それらを伝送のためにバッファに入れ、SENDING状態に遷移します。

SENDING状態では、実装はバッファされたメッセージflightを伝送します。メッセージが送られると、もしハンドシェイクにおける最後のflightだった場合、実装はFINISH状態に遷移します。もしくは、もし実装がメッセージを受信することを期待している場合は、再送タイマーをセットし、WAITING状態に遷移します。

WAITING状態から抜けるためには3つの方法があります。

1. 再送タイマーがタイムアウトする：実装はSENDING状態に遷移し、flightを再送し、再送タイマーをリセットし、再度WAITING状態に遷移します。
2. 実装がピアから再送されたflightを読み取る：実装はSENDING状態に遷移し、flightを再送し、再送タイマーをリセットし、再度WAITING状態に遷移します。ここでの理論的根拠は、重複のメッセージの受信はおそらくピアのタイマーのタイムアウトの結果であり、それゆえ過去のflightの一部が消失したことを示唆するものであるということです。
3. 実装が次のメッセージflightを受け取る：もしこれが最後のメッセージflightの場合、実装はFINISHED状態に遷移します。もし実装が新たなflightを送る必要がある場合は、PREPARING状態に遷移します。（部分的なメッセージであれflight内のメッセージの一部であれ）部分的な読み込みでは状態の遷移やタイマーのリセットは生じません。

DTLSのクライアントは最初のメッセージ（ClientHello）を送信するため、クライアントはPREPARING状態から開始します。DTLSサーバーはWAITINGから開始しますが、バッファは空で再送タイマーも持っていません。

サーバーが再度のハンドシェイクを希望するとき、HelloRequestメッセージを送信するためにFINISHED状態からPREPARING状態に遷移します。クライアントがHelloRequestを受信したとき、ClientHelloメッセージを送信するためにFINISHED状態からPREPARING状態に遷移します。

加えて、FINISHED状態のとき、最後のflightを送信するノード（通常のハンドシェイクにおいてはサーバー、再開したハンドシェイクにおいてはクライアント）は、少なくともTCPで定義されたデフォルトのMSLの2倍の時間、ピアの最後のflightに最後のflightの再送でレスポンスを返さなければなりません（MUST）。これにより最後のflightが消失した場合のデッドロック状態を回避します。DTLS1では明示的ではありませんが、ステートマシンが正常に働くことは常に要求されたものであるため、この要求はDTLS 1.0にも同様に適用されます。なぜこれが必要であるか理解するために、もしサーバーのFinishedメッセージが消失した場合に通常のハンドシェイクで何が起こるか考えてみてください。サーバーはハンドシェイクが完了したと信じていますが、実際にはそうではありません。クライアントはFinishedメッセージを待っているため、クライアントの再送タイマーが発火し、クライアントのFinishedメッセージを再送します。これによりサーバーはFinishedメッセージでレスポンスを返すこととなり、ハンドシェイクを完了させます。同じロジックが再開したハンドシェイクにおけるサーバー側にも適用されます。

パケットロスのために、片側が反対側からのFinishedメッセージを受けっていないにも関わらず、アプリケーションデータを送っている可能性があることを記しておきます。実装は新しいepochのためのFinishedメッセージを両方が受け取るまで、そのepochの全てのアプリケーションデータのパケットを廃棄するかバッファに入れるかのどちらかを行わなければなりません（MUST）。実装は対応するFinishedメッセージの受信に先立った新しいepochのアプリケーションデータの受信を順序逆転やパケットロスの証拠と扱い、再送タイマーをショットカットして最後のflightを即座に再送してもよいです（MAY）。

##### 4.2.4.1 Timer Values

タイマーの値は実装の選択ではありますが、例えばもし多くのDTLSのインスタンスが早期にタイムアウトして輻輳した回線上で非常に早急に再送した場合など、タイマーの誤操作は深刻な輻輳の問題を引き起こす可能性があります。実装は最初のタイマーの値として1秒（RFC 6298で定義された最小値）を使用し、RFC 6298の最大値である60秒も超えないまで再送ごとに値を2倍にする必要があります（SHOULD）。時間に制約のあるアプリケーションに対して遅延を改善するために、RFC 6298のデフォルトである3秒よりも1秒タイマーを推奨することを記しておきます。

実装は、タイマーの値が初期値にリセットできるタイミングである伝送での消失が発生しなかったときまで、現在のタイマーの値を保持しておく必要があります（SHOULD）。現在のタイマーの値の10倍もの長期間のアイドル状態ののち、実装はタイマーを初期値にリセットすることができます。これが発生しうる状況の1つは、相当の量のデータ転送後に再度のハンドシェイクが実施されるときです。

#### 4.2.5 ChangeCipherSpec

TLSと同様に、ChangeCipherSpecメッセージは技術的にはハンドシェイクメッセージではありませんが、タイムアウトや再送の目的のために、関連したFinishedメッセージと同じflightの一部として扱われなければなりません（MUST）。メッセージ消失の場合のハンドシェイクメッセージに関して、ChangeCipherSpecの順序は明確には確証できないため、これは潜在的なあいまいさを生じさせます。

論理的にChangeCipherSpecに先立つ、期待されるハンドシェイクメッセージのセットは、残りのハンドシェイクの状態から予測できるため、これは現在のTLSのモードでは問題ではありません。しかしながら、将来のモードはあいまいさを生じさせないために注意を払わなければなりません（MUST）。

#### 4.2.6 CertificateVerify and Finished Messages

CertificateVerifyとFinishedメッセージはTLSと同じ構造です。DTLS特有のフィールドであるmessage_seq, fragment_offset, fragment_lengthを含めて、ハッシュの計算はハンドシェイクメッセージ全体を含んでいます。

しかしながら、ハンドシェイクメッセージのフラグメンテーションに対して神経質にならないために、FinishedのMACは送られたハンドシェイクメッセージのそれぞれは単一のフラグメントであるとして計算されなければなりません（MUST）。cookieの取り交わしが使用された場合、最初のClientHelloとHelloVerifyRequestはCertificateVerifyやFinishedのMACの計算に含んではならない（MUST NOT）ことを記しておきます。

#### 4.2.7 Alert Messages

ハンドシェイクの状況で起こった場合でさえも、アラートのメッセージは全く再送されないことを記しておきます。しかしながら、通常時にアラートを発するDTLSの実装は、もし問題のあるレコードを再度受信した場合（e.g., 再送されたハンドシェイクメッセージのように）には、新たなアラートメッセージを生成する必要があります。実装はピアが永続的に問題のあるメッセージを送っている場合を検知する必要があり（SHOULD）、かつそのような誤った挙動が検知された後はローカルの接続状態を切断する必要があります（SHOULD）。

#### 4.2.8 Establishing New Associations with Existing Parameters

もしDLTSのクライアント-サーバーのペアが同じホスト／ポートの四つ組で繰り返しの接続が起こるように設定された場合は、クライアントは無言で接続を切って同じパラメータでもう一度始めることが可能です（e.g., 再起動後）。このときサーバーからはepoch=0の新たなハンドシェイクが現れたかのように見えます。サーバーが既知のホスト／ポートの四つ組における既存の接続であると信じ、かつepoch=0のClientHelloを受信するような場合、新たなハンドシェイクを続行する必要があります（SHOULD）が、cookieの取り交わしを完了するか、検証済みのFinishedメッセージの伝達を含めた完全なハンドシェイクが完了することにより、クライアントが到達性を示すまで、既存の接続を破棄してはなりません（MUST NOT）。正当なFinishedメッセージを受け取ったのち、オーバーラップしたepochの2本の正当な接続のあいだの混乱を避けるために、サーバーは以前の接続を破棄しなければなりません（MUST）。到達性の要求は、off-path/blind攻撃者が単に偽造したClientHelloメッセージを送るだけで接続を破壊することを防ぎます。

### 4.3 Summary of New Syntax

このセクションはTLS 1.2とDTLS 1.2の間で変更されたデータ構造に関する仕様について記載しています。構文の定義についてはTLS 1.2を参照してください。

#### 4.3.1 Record Layer

```text
   struct {
        ContentType type;
        ProtocolVersion version;
        uint16 epoch;                                     // New field
        uint48 sequence_number;                           // New field
        uint16 length;
        opaque fragment[DTLSPlaintext.length];
      } DTLSPlaintext;

      struct {
        ContentType type;
        ProtocolVersion version;
        uint16 epoch;                                     // New field
        uint48 sequence_number;                           // New field
        uint16 length;
        opaque fragment[DTLSCompressed.length];
      } DTLSCompressed;

      struct {
        ContentType type;
        ProtocolVersion version;
        uint16 epoch;                                     // New field
        uint48 sequence_number;                           // New field
        uint16 length;
        select (CipherSpec.cipher_type) {
          case block:  GenericBlockCipher;
          case aead:   GenericAEADCipher;                 // New field
        } fragment;
      } DTLSCiphertext;
```

#### 4.3.2 Handshake Protocol

```text
   enum {
     hello_request(0), client_hello(1), server_hello(2),
     hello_verify_request(3),                          // New field
     certificate(11), server_key_exchange (12),
     certificate_request(13), server_hello_done(14),
     certificate_verify(15), client_key_exchange(16),
     finished(20), (255) } HandshakeType;

   struct {
     HandshakeType msg_type;
     uint24 length;
     uint16 message_seq;                               // New field
     uint24 fragment_offset;                           // New field
     uint24 fragment_length;                           // New field
     select (HandshakeType) {
       case hello_request: HelloRequest;
       case client_hello:  ClientHello;
       case server_hello:  ServerHello;
       case hello_verify_request: HelloVerifyRequest;  // New field
       case certificate:Certificate;
       case server_key_exchange: ServerKeyExchange;
       case certificate_request: CertificateRequest;
       case server_hello_done:ServerHelloDone;
       case certificate_verify:  CertificateVerify;
       case client_key_exchange: ClientKeyExchange;
       case finished: Finished;
     } body; } Handshake;

   struct {
     ProtocolVersion client_version;
     Random random;
     SessionID session_id;
     opaque cookie<0..2^8-1>;                             // New field
     CipherSuite cipher_suites<2..2^16-1>;
     CompressionMethod compression_methods<1..2^8-1>; } ClientHello;

   struct {
     ProtocolVersion server_version;
     opaque cookie<0..2^8-1>; } HelloVerifyRequest;
```

## 5. Security Considerations

この文書はTLS 1.2の変形を説明しているため、セキュリティに関する検討項目の大半はTLS 1.2のAppendix D, E, Fで説明されているものと同じです。

DTLSによって追加で生じた主要なセキュリティに関する検討項目はdenial of service (DoS)です。DTLSはDoSを防ぐために設計されたcookieの交換を含んでいます。しかしながら、このcookieの交換を行わない実装は依然としてDoSに対して脆弱です。特に、cookieの交換を行わないDTLSサーバーは、たとえそのサーバー自身がDoSを受けていなくても、DoS増幅攻撃に使用される可能性があります。それゆえ、DTLSサーバーは使用される環境においてその増幅が脅威ではないと確信する正当な理由がない限り、cookieの交換を使用する必要があります（SHOULD）。DTLSクライアントはすべてのハンドシェイクにおいてcookieの交換を行う準備をしなければなりません（MUST）。

TLSの実装とは異なり、DTLSの実装は正しい形式でないレコードに対してはコネクションを終了し、返答しないほうがよい（SHOULD NOT）です。この詳細に関しては4.1.2.7のセクションを参照してください。

## 6. Acknowledgments

DTLSの設計に関して議論、コメントいただいたDan Boneh, Eu-Jin Goh, Russ Housley, Constantine Sapuntzakis, Hovav Shachamの各氏に感謝申し上げます。次に、DTLSのオリジナルのNDSSの論文に対してコメントいただいたNDSSの査読者に感謝申し上げます。また、いくつかの論点を明快にするようなフィードバックをいただいたSteve Kent氏に感謝申し上げます。PMTUに関するセクションはDCCPの仕様から借用しました。Pasi Eronenはこの使用に関する詳細なレビューを提供してくれました。Peter Saint-Andreは8章の変更リストを提供してくれました。ark Allman, Jari Arkko, Mohamed Badra, Michael D'Errico, Adrian Farrell, Joel Halpern, Ted Hardie, Charlia Kaufman, Pekka Savola, Allison Mankin, Nikos Mavrogiannopoulos, Alexey Melnikov, Robin Seggelmann, Michael Tuexen, Juho Vaha-Herttua, and Florian Weimerの各氏からこのドキュメントに関する有益なコメントをいただきました。

## 7. IANA Considerations

この文書はTLSと同じ識別子を使用しているため、新たなIANAへの登録は必要ありません。TLSに新たな識別子が割り当てられたときには、それらがDTLSに適合するかどうかを明示しなければなりません（MUST）。IANAは仕様がDTLSにおいて使用されうるかどうかを示すために、登録されているすべてのTLSパラメータにDLTS-OK flagを追加するよう修正しました。この文書の公表の時点で、以下のものを除いた全てのTLSに登録されている識別子はDTLSにおいても適合しています。登録された識別子の完全な表はIANAにて取得可能です。

TLS Cipher Suite Registryより:

```text
   0x00,0x03 TLS_RSA_EXPORT_WITH_RC4_40_MD5        [RFC4346]
   0x00,0x04 TLS_RSA_WITH_RC4_128_MD5              [RFC5246]
   0x00,0x05 TLS_RSA_WITH_RC4_128_SHA              [RFC5246]
   0x00,0x17 TLS_DH_anon_EXPORT_WITH_RC4_40_MD5    [RFC4346]
   0x00,0x18 TLS_DH_anon_WITH_RC4_128_MD5          [RFC5246]
   0x00,0x20 TLS_KRB5_WITH_RC4_128_SHA             [RFC2712]
   0x00,0x24 TLS_KRB5_WITH_RC4_128_MD5             [RFC2712]
   0x00,0x28 TLS_KRB5_EXPORT_WITH_RC4_40_SHA       [RFC2712]
   0x00,0x2B TLS_KRB5_EXPORT_WITH_RC4_40_MD5       [RFC2712]
   0x00,0x8A TLS_PSK_WITH_RC4_128_SHA              [RFC4279]
   0x00,0x8E TLS_DHE_PSK_WITH_RC4_128_SHA          [RFC4279]
   0x00,0x92 TLS_RSA_PSK_WITH_RC4_128_SHA          [RFC4279]
   0xC0,0x02 TLS_ECDH_ECDSA_WITH_RC4_128_SHA       [RFC4492]
   0xC0,0x07 TLS_ECDHE_ECDSA_WITH_RC4_128_SHA      [RFC4492]
   0xC0,0x0C TLS_ECDH_RSA_WITH_RC4_128_SHA         [RFC4492]
   0xC0,0x11 TLS_ECDHE_RSA_WITH_RC4_128_SHA        [RFC4492]
   0xC0,0x16 TLS_ECDH_anon_WITH_RC4_128_SHA        [RFC4492]
   0xC0,0x33 TLS_ECDHE_PSK_WITH_RC4_128_SHA        [RFC5489]
```

TLS Exporter Label Registryより:

```text
   client EAP encryption       [RFC5216]
   ttls   keying material      [RFC5281]
   ttls   challenge            [RFC5281]
```

この文書は新たなハンドシェイクメッセージであるhello_verify_requestを定義しており、その値はTLS 1.2で定義されたTLS HandshakeType registoryによって割り当てられています。IANAによって"3"の値が割り当てられています。

## 8. Changes since DTLS 1.0

この文書はDTLS 1.0からの以下の変更を反映しています。

- TLS 1.2に調和するための更新内容
- （TLS 1.2の変更に追従する）4.1.2.3のセクションのAEAS Ciphersに関する追記
- 4.1のセクションにおけるシーケンス番号とエポックに関する明確化と、4.2.8のセクションにおけるステートロスに対する明快な対処手順について
- 4.1.1.1のセクションにおけるPath MTUの課題に関する説明と詳細な規定と、断片化文章全体の明確化について
- 4.1.2.7のセクションにおける正しい形式でないレコードの扱い方に関する説明について
- 4.2.1のセクションの末尾のの正しい形式でないcookieの扱い方に関して説明している新しい段落
- 4.2.4のセクションの末尾のハンドシェイクのデッドロック状態の回避方法に関する説明の追記
- 4.2.6のセクションのCertificateVerifyメッセージに関する追記
- 4.1のセクションのepoch wrappingの禁止について
- IANAの要求に関する明確化と、それぞれのパラメータに対するIANA registration flagの明示的な要求について
- 繰り返しのClientHelloメッセージの扱いに関するレコードのシーケンス番号のミラーリング手法に関する追記
- HelloVerifyRequestに対する決まったバージョン番号の推奨について
- いくつかの編集上の変更

## 9.  References

### 9.1.  Normative References

- [REQ]       Bradner, S., "Key words for use in RFCs to Indicate Requirement Levels", BCP 14, RFC 2119, March 1997.
- [RFC1191]   Mogul, J. and S. Deering, "Path MTU discovery", RFC 1191, November 1990.
- [RFC4301]   Kent, S. and K. Seo, "Security Architecture for the Internet Protocol", RFC 4301, December 2005.
- [RFC4443]   Conta, A., Deering, S., and M. Gupta, Ed., "Internet Control Message Protocol (ICMPv6) for the Internet Protocol Version 6 (IPv6) Specification", RFC 4443, March 2006.
- [RFC4821]   Mathis, M. and J. Heffner, "Packetization Layer Path MTU Discovery", RFC 4821, March 2007.
- [RFC6298]   Paxson, V., Allman, M., Chu, J., and M. Sargent, "Computing TCP's Retransmission Timer", RFC 6298, June 2011.
- [RSAGCM]    Salowey, J., Choudhury, A., and D. McGrew, "AES Galois Counter Mode (GCM) Cipher Suites for TLS", RFC 5288, August 2008.
- [TCP]       Postel, J., "Transmission Control Protocol", STD 7, RFC 793, September 1981.
- [TLS12]     Dierks, T. and E. Rescorla, "The Transport Layer Security (TLS) Protocol Version 1.2", RFC 5246, August 2008.

### 9.2.  Informative References

- [DCCP]      Kohler, E., Handley, M., and S. Floyd, "Datagram Congestion Control Protocol (DCCP)", RFC 4340, March 2006.
- [DCCPDTLS]  Phelan, T., "Datagram Transport Layer Security (DTLS) over the Datagram Congestion Control Protocol (DCCP)", RFC 5238, May 2008.
- [DTLS]      Modadugu, N. and E. Rescorla, "The Design and Implementation of Datagram TLS", Proceedings of ISOC NDSS 2004, February 2004.
- [DTLS1]     Rescorla, E. and N. Modadugu, "Datagram Transport Layer Security", RFC 4347, April 2006.
- [ECCGCM]    Rescorla, E., "TLS Elliptic Curve Cipher Suites with SHA-256/384 and AES Galois Counter Mode (GCM)", RFC 5289, August 2008.
- [ESP]       Kent, S., "IP Encapsulating Security Payload (ESP)", RFC 4303, December 2005.
- [IANA]      IANA, "Transport Layer Security (TLS) Parameters", http://www.iana.org/assignments/tls-parameters.
- [IKEv2]     Kaufman, C., Hoffman, P., Nir, Y., and P. Eronen, "Internet Key Exchange Protocol Version 2 (IKEv2)", RFC 5996, September 2010.
- [IMAP]      Crispin, M., "INTERNET MESSAGE ACCESS PROTOCOL - VERSION 4rev1", RFC 3501, March 2003.
- [PHOTURIS]  Karn, P. and W. Simpson, "Photuris: Session-Key Management Protocol", RFC 2522, March 1999.
- [POP]       Myers, J. and M. Rose, "Post Office Protocol - Version 3", STD 53, RFC 1939, May 1996.
- [SIP]       Rosenberg, J., Schulzrinne, H., Camarillo, G., Johnston, A., Peterson, J., Sparks, R., Handley, M., and E. Schooler, "SIP: Session Initiation Protocol", RFC 3261, June 2002.
- [TLS]       Dierks, T. and C. Allen, "The TLS Protocol Version 1.0", RFC 2246, January 1999.
- [TLS11]     Dierks, T. and E. Rescorla, "The Transport Layer Security (TLS) Protocol Version 1.1", RFC 4346, April 2006.
- [WHYIPSEC]  Bellovin, S., "Guidelines for Specifying the Use of IPsec Version 2", BCP 146, RFC 5406, February 2009.

## Authors' Addresses

Eric Rescorla  
RTFM, Inc.  
2064 Edgewood Drive  
Palo Alto, CA 94303

EMail: ekr@rtfm.com

Nagendra Modadugu  
Google, Inc.

EMail: nagendra@cs.stanford.edu

## 翻訳者のアドレス

株式会社アプトポッド  
南波 寛直  
namba@aptpod.co.jp

## 免責事項

RFCの日本語訳において、内容については一切の保証を致しかねます。RFCの内容を理解する必要のある場合、リンクされている原文をご参照ください。