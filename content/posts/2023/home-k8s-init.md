---
title: "自宅Kubernetesクラスタ始めました - 構築編"
date: 2023-08-24T19:30:00+09:00
draft: false
---

{{< figure src="/media/home-k8s-init-cover.png" title="" >}}

自宅クラスタを動かし始めたので、次回構築時に困りそうなところをメモしておきます。

## ラズパイで失敗
元々Raspberry Pi4を3台重ねてクラスタを作ろうと思っていましたが、うまくいかなかったので仮想マシンで試してみることにしました。

`kubelet` に `etcdserver` に接続できねーぞ！というエラーが出たり、Raspbianではcontainerdではうまく動作しない、等。
```
etcdserver: request timed out' (will not retry!)
```

今考えるとこのエラーはディスクIOが遅すぎて起動に時間がかかっていただけかも。それでも重いものは重いし、ちゃんと運用していきたいのでリソースに余裕のある仮想マシンで動かすことにしました。

## ブリッジの設定


仮想マシンで動作させるために、まずブリッジの設定をしていきます。今まで仮想マシンのネットワーク接続は手軽なmacvtapを利用してきましたが、これだと仮想マシン間での通信が出来ず後で困ります。

ブリッジの作成方法は各環境で異なります。私の環境では `systemd-networkd` を利用しているので、 `/etc/systemd/networkd/` に下記ファイルを作成してリロードして完了です。

```
- 25-bridge.netdev
[NetDev]
Name=br0
Kind=bridge

- 30-bridge.network
[Match]
Name=br0

[Network]
DHCP=ipv4

- bind.network
[Match]
Name=enp5s0

[Network]
Bridge=br0
```

ここで重大な勘違いをしていましたが、 `br0` に対してIPアドレスを割り当てないとゲストも通信が出来ません。サーバー用PCにはNICが二つあり、 `br0` と紐付けた方での通信は行わないつもりでしたので当初はIPを振っておらず、通信が出来なくて困りました。

仮想マシンはこれに直接接続しても良いですが、デバイス名が変わったときに楽なのできちんと `virsh net-define ./define.xml` しておきます。

```xml
<!-- define.xml -->
<network>
  <name>vxlan10-bridge</name>
  <forward mode='bridge'/>
  <bridge name='br0'/>
</network>
```

ネットワークデバイスの利用は `virt-install` 利用であれば `--network newtwork:vxlan10-bridge` で出来ます。後から追加する場合は後述する `virsh attach-interface` が便利です。

```
sudo virt-install
  --name aki2
  --memory 8192
  --vcpus=4
  --osinfo ubuntu22.04
  -c /var/lib/libvirt/images/ubuntu-22.04.2-live-server-amd64.iso
  --disk size=64
  --network network:vxlan10-bridge --boot uefi
```

適当に起動して適当に設定します。少なくともコントロールプレーンになる仮想マシンのIPアドレスは固定しておいたほうが良いでしょう。

## ファイアウォールを無効化する
`br0` を作成後うまく通信が出来ず、この間にファイアウォールを無効化する処理をいれました。実際には必要ありませんが、後ほど有効化するので書いておきます。

ArchWikiに書いてあるように、この設定はカーネルモジュールがきちんと読み込まれている状態で行う必要があります。 `br_netfilter` が読み込まれる前に実行されると、ひっそりと適応されないことになります。対策として、明示的に `modules-load.d` にロードするように記載することが出来ます。

```
- /etc/modules-load.d/bridge.conf
br_netfilter
```

```
- /etc/sysctl.d/99-sysctl.conf
net.ipv4.ip_forward = 1 # これは必須

# 下記は実際には不要だった 後で1に変更する
net.bridge.bridge-nf-call-arptables = 0
net.bridge.bridge-nf-call-ip6tables = 0
net.bridge.bridge-nf-call-iptables = 0
```

## kubeadm join

詳細な設定は省きますがうまくいくはずです。

## ファイアウォール復活 & NIC統合
とりあえず動作することは確認が取れましたが、現在の状態では実質ファイアウォールが設定されていないのと同様です。IPv6を使い倒したいことからルーターのフィルタリングは多少甘めに設定していること、NFSを利用するためUFWできちんと接続元を絞りたいことから、このままの状態で利用することは避けたいです。

そこで、現在メインのNICである `enp4s0` にbr0をバインドし、その上でファイアウォールをしっかり設定することにしました。

### NICの統合
`enp5s0` と接続されている `br0` を `enp4s0` に変更し、ホストの通信もすべてそれ経由で通信を行うようにします。NICが減ることで考えることが減ります。

```
- 25-bridge.netdev
[NetDev]
Name=br0
Kind=bridge

- 30-bridge.network
[Match]
Name=br0

[Network]
Address=192.168.182.100/24
Gateway=192.168.182.1
DNS=1.1.1.1

- bind.network
[Match]
Name=enp4s0

[Network]
Bridge=br0
```

### ファイアウォール復活

```
- /etc/sysctl.d/99-sysctl.conf
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-arptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
```

Forwardingは全許可します。折衷案です。

```
- /etc/default/ufw
# Set the default forward policy to ACCEPT, DROP or REJECT.  Please note that
# if you change this you will most likely want to adjust your rules
DEFAULT_FORWARD_POLICY="ACCEPT"
```

これで `br0` の通信をUFWが管理出来るようになりました。

## WireGuardとの競合
さて、先ほどの設定を完了するとCalicoがエラー落ちするようになってしまいました。ホストのIPアドレスから謎のコネクションが飛んできています。

```
bird: BGP: Unexpected connect from unknown address 192.168.182.100 (port 58043)
```

結論から言うと、ホストでWireGuardを動かしており、それで `iptables -t nat -A POSTROUTING -o br0 -j MASQUERADE` を行っていたため、 `br0` を通るBGPパケットがNATされたために通信が出来なくなっていました。

これを解決するために、ノードのみが属する分離(isolated)されたブリッジを作成します。

## isolatedなブリッジを作成する
ネットワークから分離されたブリッジを作成します。

```
$ vim define.xml

<network>
  <name>kube-isolated</name>
  <bridge name='kube-isolated' stp='on' delay='0'/>
</network>

$ sudo virsh net-define ./define.xml
$ sudo virsh net-autostart kube-isolated
$ sudo virsh net-create kube-isolated
```

この時点で薄々コントロールプレーンとの通信もこのブリッジを通すように設定しなければいけないだろう考えが頭の中にあり、DHCPを利用せず手動でIPアドレスを設定しましたが、問題なのはCalicoの通信のみだったので杞憂でした。利用した方がIPアドレスの設定は簡単になると思いますが、自由です。

ゲストの設定も編集します。`virsh edit` を利用しても良いですが、 `attach-interface` の方がUUIDの生成の設定をやってくれるので便利です。

```
$ sudo virsh attach-interface aki0 network kube-isolated --model virtio --config
$ sudo virsh attach-interface aki1 network kube-isolated --model virtio --config
$ sudo virsh attach-interface aki2 network kube-isolated --model virtio --config
```

これでブリッジインターフェースが接続されました。適当にゲストを再起動して新しい設定ファイルを読み込ませます。

次に、ゲストのネットワークを更新します。ゲストはUbuntuなので `netplan` を利用します。 `ip ad` でデバイス名を確認しておきましょう。場合によっては `enpXs0` 等の名前になっていることがあります。

```
$ ip ad
...
3: ens1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:09:02:78 brd ff:ff:ff:ff:ff:ff
    altname enp8s1
    inet 192.168.190.100/24 brd 192.168.190.255 scope global ens1
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:ff:fe09:278/64 scope link
       valid_lft forever preferred_lft forever

$ sudo nano /etc/netplan/00-installer-config.yaml
```

今回は `ens1` だったので、デバイスにIPを割り当てます。
```yaml
network:
  ethernets:
    enp1s0:
      addresses:
      - 192.168.182.130/24
      nameservers:
        addresses:
        - 1.1.1.1
        - 1.0.0.1
        search: []
      routes:
      - to: default
        via: 192.168.182.1
    ens1:
      addresses:
      - 192.168.190.100/24
      # DHCPを利用出来るようにした場合は `dhcp4: true` だけでOK
      # dhcp4: true
  version: 2
```

ゲストの台数分この作業を行います。手動でIP設定する場合は被らないようにしてください。

次にKubernetesとCalicoの設定...と言いたいところですが、この時点ですでにうまく動作しているはずです。コントロールプレーンのAPIエンドポイントは変更されておらず、引き続き `br0` を通して通信が出来ています。Calicoは特に指定をしていなければ、最初に利用可能になった方法で通信を行います。

https://docs.tigera.io/calico/latest/networking/ipam/ip-autodetection#autodetection-methods


Cilliumチャレンジに続きます。