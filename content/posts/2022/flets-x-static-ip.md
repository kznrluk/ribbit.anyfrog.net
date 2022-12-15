---
title: "v6プラス固定IPサービスで接続ができない件 (Yamaha RTX1300 & Elecomルーター)"
date: 2022-12-12T21:45:00+09:00
draft: false
---

未解決です。状況と思考の整理のためにメモとして残しておきます。

## 環境
- フレッツ光 光クロス
- かもめインターネット固定IPオプション
- VNE: 日本ネットワークイネイブラー株式会社 (JPNE)
- 光クロス + ONU直刺しなのでアドレスの配布はDHCPv6-PD
- 動的IP(MAP-E)は繋がる
- 固定IP(IPIP)が繋がらない
- メインルーター RTX1300
- 検証用ルーター WRC-X6000XS

## ヤマハもエレコムも繋がらない
民生用ルーターのWRC-X6000XSが繋がり、業務用ルーターであるRTX1300だけ繋がらない場合は私の設定が悪い可能性が高いのだが、今回はどちらも繋がらない。RTX1300はパケットを通さず、WRC-X6000XSは再接続を繰り返すような状態になる。

MAP-Eは内部でIPIPトンネリングを利用しているため、スペック不足やフレッツ網の不具合の可能性は低いと予想。

### RTX1300の場合
RTX1300のWebGUIからTECHINFOを見ると、IPIPで繋がってるよ的な表示は出るが、IPv4送信パケットが少しだけ存在する以外通信をしている形跡がない状態。IPIPトンネリングの仕様に詳しくないが、WireGuard的な投げっぱなしプロトコルだとすれば繋がってるよ表示はあんまり意味ない可能性がある。

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

### WRC-X6000XSの場合
WANの設定画面から「v6プラス」固定IPサービスを選択し各情報を入力するが接続できず。ステータス画面で接続中→接続できませんでした→空白→接続中を繰り返す。

### ISPへネットフォレスト問い合わせ
まったく別々のメーカーのルーター2つがどちらも接続できないことから、ISP側の不具合を疑いかもめインターネットのネットフォレストへ問い合わせを行う。

テンプレ的な接続情報の確認をメールで行い、電話連絡。色々話をしてもらったが結果的にルーターが悪い可能性があるのでエレコムに問い合わせをしてほしいとのコト。ここで粘ってもらちが明かないのでエレコムに問い合わせる。

### エレコムへ問い合わせ
こちらもテンプレ的な接続設定を行い、接続ができないことを確認。折り返し電話するとのことなので待機。

折り返し。テンプレ的な接続説明のあと下記ページを案内され、接続サービスに名前が載っていないからかもめインターネットは動作保証対象外であり対応していないと案内を受ける。

[動作確認済みIPv6(IPoE)接続サービス対応表 | 対応表・動作検証 エレコム株式会社 ELECOM](https://www.elecom.co.jp/support/list/network/ipv6/)

ISPはVNEの回線をローミングしているだけなので、 `「v6プラス」固定IPサービス` が載っている以上対応しているのでは？と投げると、当社でできることは交換対応くらいになる、JPNEの回線を利用しているかもしれないが、かもめインターネットだけ特殊なプロトコルを利用している可能性があると案内を受けた。

自分も相手をあまり困らせたくなかったので、いったん電話を切りISPへその特殊なプロトコルとやらが利用されているか聞くことにする。

### エレコムルーターの「v6プラス」固定IPサービスに不具合がある説
電話を置いて少し考えてみると、エレコムのサポートセンターが出してきた上記のページ内に、 `「v6プラス」固定IPサービス` を利用しているISPがないことに気がついた。

ルーターの機能を開発するときに、 `「v6プラス」固定IPサービス` を追加するのであれば、開発時に使ったそのISPを動作確認済みとしてここに追記するはずだが、記載されていないということは使わずに対応を謳っていたり、過去対応していたが今は対応していないとか、そういった背景があるのではないかと感じた。

そのため、RTX1300でもう一度試行錯誤してみることに。

### RTX1300 設定見直し
今までWebGUIをベースに設定を行ってきたが、今回はマニュアルを参考にすべてテキストで行ったが改善できず。

この時に試行錯誤したコンフィグは付録として載せてあります。

## 2022/12/14 追記ここから

ヤマハネットワークエンジニア会、Twitterで多くの皆様に助言をいただいております。ありがとうございます。

いただいたご指摘を簡単に下記でまとめさせていただきます。結論から言うとまだ疎通しておりませんが、設定が怪しい部分もありましたので非常に参考になります。

- Twitter: ルーター自身にIPv6のデフォルトゲートウェイが無いので追加してみては？
- YNE: 21ipの固定IPサービスを利用しているが、MTUの設定以外同じであった。MTUを外してみては？
- YNE: IPフィルターのLAN側IPの指定が誤っている。
- YNE: フィルターをいったんすべて外して切り分け
- YNE: RTX1300の新機能であるフレキシブルポートを外して見る

付録のコンフィグファイルとTECHINFOについても上記ご指摘に合わせて更新済みです。

## 2022/12/15 追記ここから

ONUとルーターにスイッチを挟んでパケットキャプチャをしたところ、BRアドレスが一文字間違っていることに気がつきました...。紙で届いた認証情報は届いたそのうちにパスワード管理ソフトに入力して原本はしまってしまうのですが、最初の入力時に誤っていたみたいです。

IPIPはUDPのような投げっぱなしのプロトコルなので、接続時のネゴシエーション等がなく送信先を間違えるとパケットが虚無に消えていきます。相手から応答が帰ってこない場合、接続がうまくいっているかうまくいっていないかの判定ができないため、RTXではトンネルUPの表示が出ても通信出来なかったようです。

うまく通信ができない方がいらっしゃいましたら、今一度手元の設定資料が正常に入力されているか確認してみてください。お騒がせしてすみませんでした🙇‍♂️

## 付録

### RTX1300のコンフィグファイル 認証情報以外そのまま
```
# RTX1300 Rev.23.00.04 (Wed Aug 10 11:40:27 2022)
# MAC Address : ac:44:f2:b6:86:00 - ac:44:f2:b6:86:07
# Memory 1024Mbytes, 8LAN
# main:  RTX1300 ver=00 serial=S78000000 MAC-Address=ac:44:f2:b6:86:00 - ac:44:f2:b6:86:07
# Reporting Date: Dec 12 18:19:32 2022
login user admin *
user attribute admin administrator=2
console character ja.utf8
login timer clear

ip route default gateway tunnel 1
ip lan1 address 192.168.182.1/24

# FLEX PORT
# ↓外したが変わらず
lan flexible-port lan1=1-6,8,10 lan2=9 lan3=7

# NGN
ngn type lan2 ntt
 ipv6 prefix 1 dhcp-prefix@lan2::/64
 # ↓transixの設定例では存在するがv6plusの設定例では存在しない
 # 2022/12/15追記 Yamahaに問い合わせ不要であることを確認済み
 # ipv6 route default gateway dhcp lan2
 ipv6 lan1 address dhcp-prefix@lan2::(インターフェースID)/64
 ipv6 lan1 rtadv send 1 o_flag=on
 ipv6 lan1 dhcp service server
 # ↓transixの設定例では存在するがv6plusの設定例では存在しない
 # 2022/12/15追記 Yamahaに問い合わせ不要であることを確認済み
 # ipv6 lan2 address dhcp-prefix@lan2::/64
 ipv6 lan2 dhcp service client

# TUNNEL
tunnel select 1
 tunnel encapsulation ipip
 tunnel endpoint address (BRアドレス)
 # ↓v6plusの設定例では存在するがtransixの設定例では存在しない
 # 2022/12/15追記 Yamahaに問い合わせ必要であることを確認済み
 ip tunnel mtu 1460
 ip tunnel nat descriptor 1
 ip tunnel tcp mss limit auto
 tunnel enable 1

# NAT
nat descriptor type 1 masquerade
nat descriptor address outer 1 (固定IP)

# SWITCH
switch control use lan1 on terminal=on

# DHCP
dhcp service server
dhcp server rfc2131 compliant except remain-silent
dhcp scope 1 192.168.182.2-192.168.182.255/24 expire 42:00 maxexpire 42:00
dhcp scope bind 1 192.168.182.100 ff 6b 84 6a ce 00 02 00 00 ab 11 04 7f eb 92 af 0e 50 a6
dhcp scope bind 1 192.168.182.110 ethernet 24:5e:be:30:4d:04
dhcp scope bind 1 192.168.182.111 01 74 da 38 e8 8c d4
dhcp scope bind 1 192.168.182.120 01 dc a6 32 ad 32 18
dhcp scope bind 1 192.168.182.121 01 dc a6 32 ad 31 c4
dhcp scope bind 1 192.168.182.122 01 dc a6 32 ad 2d d7
dhcp scope bind 1 192.168.182.200 ethernet a0:36:bc:83:e5:4c

# DNS
dns host lan1
dns service fallback on
dns server dhcp lan2
dns server select 500000 dhcp lan2 any .

# FILTER IPv4
tunnel select 1
 ip tunnel secure filter in 200030 200039
 ip tunnel secure filter out 200099 dynamic 200080 200082 200083 200084 200098 200099
 tunnel enable 1
ip filter 200030 pass * 192.168.182.0/24 icmp * *
ip filter 200039 reject *
ip filter 200099 pass * * * * *
ip filter dynamic 200080 * * ftp
ip filter dynamic 200082 * * www
ip filter dynamic 200083 * * smtp
ip filter dynamic 200084 * * pop3
ip filter dynamic 200098 * * tcp
ip filter dynamic 200099 * * udp

# FILTER IPv6
ipv6 lan2 secure filter in 200030 200031 200038 200039
ipv6 lan2 secure filter out 200099 dynamic 200080 200081 200082 200083 200084 200098 200099
ipv6 filter 200030 pass * * icmp6 * *
ipv6 filter 200031 pass * * 4
ipv6 filter 200038 pass * * udp * 546
ipv6 filter 200039 reject *
ipv6 filter 200099 pass * * * * *
ipv6 filter dynamic 200080 * * ftp
ipv6 filter dynamic 200081 * * domain
ipv6 filter dynamic 200082 * * www
ipv6 filter dynamic 200083 * * smtp
ipv6 filter dynamic 200084 * * pop3
ipv6 filter dynamic 200098 * * tcp
ipv6 filter dynamic 200099 * * udp

# V6PLUS NOTIFY
ipv6 lan1 prefix change log on
lan linkup send-wait-time lan2 5
schedule at 1 startup * lua emfs:/v6plus_address_notification.lua
embedded file v6plus_address_notification.lua <<EOF
UPD_SV = "http://fcs.enabler.ne.jp/update"
USERNAME = "(USER_NAME)"
PASSWORD = "(PASSWORD)"
IPv6_IF = "LAN1"
LOG_PTN = "Add%s+IPv6%s+prefix.+%(Lifetime%:%s+%d+%)%s+via%s+" .. IPv6_IF .. "%s+by"

RETRY_INTVL = 10
RETRY_NUM = 3

LOG_LEVEL = "info"
LOG_PFX = "[v6plus]"
FAIL_MSG = "Failed to notify IPv6 address to the update server. (remaining retry: %d time(s))"

function logger(msg)
  rt.syslog(LOG_LEVEL, string.format("%s %s", LOG_PFX, msg))
end

local rtn, count, log, result
local req_t = {}
local res_t

req_t.url = string.format("%s?user=%s&pass=%s", UPD_SV,
                          USERNAME, PASSWORD)
req_t.method = "GET"

while true do
  rtn = rt.syslogwatch(LOG_PTN)

  if rtn then
    count = RETRY_NUM

    while true do
      res_t = rt.httprequest(req_t)
      
      if res_t.rtn1 then
        logger("Notified IPv6 address to the update server.")

        if res_t.code == 200 then
          result = "OK"
        else
          result = "NG"
        end

        log = string.format("%s to update IPv6 address. (code=%d, body=%s",
                            result, res_t.code, res_t.body)
        logger(log)

        break
      end

      count = count - 1
      if count > 0 then
        logger(string.format(FAIL_MSG, count))
        rt.sleep(RETRY_INTVL)
      else
        logger("Failed to notify IPv6 address to the update server")
        break
      end

    end
  end
end
EOF

```

### TECH LOG 参考になりそうな場所抜粋
DHCPで降ってきた？IPv6アドレスは240b:から始まるJPNEのもの。
```
show status ipv6 dhcp
DHCPv6 status

  LAN1 [server]
    state: reply
    state: reply

  LAN2 [client]
    state: established
    server:
      address: ::
      preference: 0
      prefix: 240b:11:b901:1800::/56
      duration: 1844
      T1: 922
      T2: 1475
      preferred lifetime: 1613
      valid lifetime: 1844
      DNS server[1]: 2404:1a8:7f01:b::3
      DNS server[2]: 2404:1a8:7f01:a::3
      Domain name[1]: flets-east.jp
      Domain name[2]: iptvf.jp
      SNTP server[1]: 2404:1a8:1102::b
      SNTP server[2]: 2404:1a8:1102::a

---
show status ipip
------------------- IPIP INFORMATION -------------------
Number of IPIP tunnels: 1
TUNNEL[1]: 
  Current status is Online.
  from 2022/12/14 12:41:24.
  58 seconds  connection.
  Received:    (IPv4) 0 packet [0 octet]
               (IPv6) 0 packet [0 octet]
  Transmitted: (IPv4) 986 packets [70235 octets]
               (IPv6) 0 packet [0 octet]
  Remote endpoint address: (BRアドレスが表示されている)

---
show ip route
Destination         Gateway          Interface       Kind  Additional Info.
default             -                 TUNNEL[1]    static  
192.168.182.0/24    192.168.182.1          LAN1  implicit  

---
show ip route summary
Protocol   Active    Hidden
-----------------------------
Static          1         0
Implicit        1         0
Temporary       0         0
Redirect        0         0
RIP             0         0
OSPF            0         0
BGP             0         0
-----------------------------
Total           2         0

---
show ipv6 route
Destination              Gateway                  Interface  Type
default                  fe80::fa0f:6fff:fe4b:714b 
                                                  LAN2       temporary
240b:11:b901:1800::/64   -                        LAN1       implicit
---
show ipv6 route summary
Protocol    Active   Hidden
-----------------------------
static           0         0
Implicit         1         0
Temporary        1         0
ICMP redirect    0         0
RA               0         0
RIPng            0         0
OSPFv3           0         0
-----------------------------
Total            2         0

---
show nat descriptor address
NAT/IP masquerade compatibility type : 2
Reference Descriptor : 1, Assigned Interface : TUNNEL[1](1)
Masquerade Table
    Outer address: 27.89.54.24
    Port range: 60000-64095, 49152-59999, 44096-49151   169 session.
  -*-    -*-    -*-    -*-    -*-    -*-    -*-    -*-    -*-    -*-    -*-
      No.              Inner   Session Count           Limit         Type
       1     192.168.182.100              71          250000         dynamic
       2     192.168.182.200              26          250000         dynamic
       3       192.168.182.2              19          250000         dynamic
       4       192.168.182.5              18          250000         dynamic
       5       192.168.182.3              12          250000         dynamic
       6       192.168.182.4              10          250000         dynamic
       7       192.168.182.7               4          250000         dynamic
       8      192.168.182.27               4          250000         dynamic
       9       192.168.182.6               3          250000         dynamic
      10       192.168.182.8               2          250000         dynamic
```

### ログ
見たところv6plus更新用のスクリプトも動いている

```
2022/12/14 13:05:10: Restart by restart command
2022/12/14 13:05:10: RTX1300 Rev.23.00.04 (Wed Aug 10 11:40:27 2022) starts
2022/12/14 13:05:10: main:  RTX1300 ver=00 serial=S78003058 MAC-Address=ac:44:f2:b6:86:00 - ac:44:f2:b6:86:07
2022/12/14 13:05:11: IP Tunnel[1] Up
2022/12/14 13:05:11: PORT1-8: PHY is Marvell 88E6193X.
2022/12/14 13:05:11: PORT9: PHY is Marvell 88X3310.
2022/12/14 13:05:11: PORT10: PHY is Marvell 88X3310.
2022/12/14 13:05:11: Add IPv6 prefix ff02::/64 (Lifetime: infinity) via LAN1 by Static
2022/12/14 13:05:13: [SCHEDULE] Startup: lua emfs:/v6plus_address_notification.lua
2022/12/14 13:05:14: LAN1: PORT4 link up (1000BASE-T Full Duplex)
2022/12/14 13:05:14: LAN1: link up
2022/12/14 13:05:15: LAN2: PORT9 link up (10GBASE-T Full Duplex)
2022/12/14 13:05:16: LAN2: link up
2022/12/14 13:05:19: [DHCPD] LAN1(port4) Allocates 192.168.182.4: 38:56:10:c6:f3:39
2022/12/14 13:05:22: [DHCPD] LAN1(port4) Extends 192.168.182.100: a0:36:bc:83:e4:05
2022/12/14 13:05:28: Add IPv6 prefix (DHCPで降ってきたアドレス)0::/64 (Lifetime: 1972) via LAN1 by DHCPv6
2022/12/14 13:05:33: [v6plus] Notified IPv6 address to the update server.
2022/12/14 13:05:33: [v6plus] OK to update IPv6 address. (code=200, body=OK
2022/12/14 13:05:39: Login succeeded for HTTP: 192.168.182.200 admin
2022/12/14 13:05:39: 'administrator' succeeded for HTTP: 192.168.182.200 admin
2022/12/14 13:05:39: [DHCPD] LAN1(port4) Allocates 192.168.182.5: f8:0f:f9:90:a8:3f
2022/12/14 13:05:40: [DHCPD] LAN1(port4) Allocates 192.168.182.7: f0:72:ea:f3:93:3d
```