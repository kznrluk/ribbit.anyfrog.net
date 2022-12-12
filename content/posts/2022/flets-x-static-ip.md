---
title: "光クロス JPNE + IPIPで接続ができない件について"
date: 2022-12-12T21:45:00+09:00
draft: false
---

大変困りました。未解決です。状況と思考の整理のためにメモとして残しておきます。

## 環境
- かもめインターネット固定IPオプション
- VNE: 日本ネットワークイネイブラー株式会社 (JPNE)
- MAP-Eは繋がる・IPIPが繋がらない
- メインルーター RTX1300
- 検証用ルーター WRC-X6000XS

## ヤマハもエレコムも繋がらない
RTX1300だけ繋がらない場合は設定ミスの可能性が往々にしてあるので、エレコムも試したが同様に繋がらず。

### RTX1300の場合
TECHINFOで繋がってるよ的表示は出るがパケットが帰ってこない。IPIPの仕様に詳しくないがWireshark的な投げっぱなしプロトコルだったら繋がってるよ表示はあんまり意味ないのでは説を感じる。

```
tunnel select 1
 tunnel encapsulation ipip
 tunnel endpoint address (BRアドレス)
 ip tunnel mtu 1000 # MTU原因か？と思って少なく設定したが変わらず
 ip tunnel secure filter in 200030 200039
 ip tunnel secure filter out 200099 dynamic 200080 200082 200083 200084 200098 200099
 ip tunnel nat descriptor 1
 tunnel enable 1
nat descriptor type 1 masquerade
nat descriptor address outer 1 27.89.54.24
```

```
show status ipip
------------------- IPIP INFORMATION -------------------
Number of IPIP tunnels: 1
TUNNEL[1]: 
  Current status is Online.
  from 2022/12/12 20:12:29.
  1 minute 9 seconds  connection.
  Received:    (IPv4) 0 packet [0 octet]
               (IPv6) 0 packet [0 octet]
  Transmitted: (IPv4) 10 packets [520 octets]
               (IPv6) 0 packet [0 octet]
  Remote endpoint address: 2404:9200:255:100::65
```

### WRC-X6000XSの場合
WAN&LANの画面で設定するが接続できず。ステータス画面で接続中→接続できませんでした→空白→接続中を繰り返す。

## ISPへネットフォレスト問い合わせ
まったく別々のルーターがどちらも接続できないことから、フレッツ網ひいてはVNEであるJPNEを疑う。

ネットフォレストへ問い合わせ→テンプレ的な接続情報の確認をメールで行い、電話連絡。結果的にルーターが悪い可能性があるのでエレコムに問い合わせを行えとのコト。

## エレコムへ問い合わせ
一発目。テンプレ的な接続設定を行い接続ができないことを確認後折り返し待ち。

折り返し二発目。テンプレ的な接続説明のあと、かもめインターネットは動作保証対象外であり説明することはできないと説明。VNE通信サービスに「v6プラス」固定IPサービスがあることを指摘すると、交換対応するかどうかという話に。

自分も相手をあまり困らせたくなかったので、いったん電話を切りISPへ話を戻すことに。

## エレコムルーター 「v6プラス」固定IPサービスがバグってる説
ここで気がついたが、サポセン相手が出してきたページにある動作確認済みIPv6(IPoE)接続サービスの欄に「v6プラス」固定IPサービスを利用しているISPがない。
普通に「v6プラス」固定IPサービスな回線につなげてテストを行っていればそのISPをこの欄に書くはずなので、ろくにデバッグしていないのでは？という気持ちになってきた。昔IIJが提供していたことを知っているので、その流れで対応したが記載忘れたとかなのか...?

## RTX1300 設定見直し
今までWebGUIをベースにすべての設定を行ってきたが、今回はすべてテキストのコンフィグで行ったが改善ならず。

後ほど追記予定