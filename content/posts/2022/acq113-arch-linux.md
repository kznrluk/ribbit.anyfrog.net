---
title: "XG-C100C(AQC113)を対応ディストリビューション外のArch Linuxで利用する"
date: 2022-11-16T22:42:00+09:00
draft: false
---

家のインターネット回線の10Gbps化に向け、XG-C100Cを購入しました。Linux対応が謳われていましたが、ドライバのインストールで躓いたので解決策を忘備録として残しておきます。また6Gbpsしか出ない場合の原因もあとがきに記しておきます。

## 対象
Debian系とRedHat系以外のディストロで下記のようなAquantia(Marvell)なチップが乗ったNICを利用したいユーザー

- XG-C100C
- XG-C100C V2 (筆者所有)
- GC-AQC113C
- TX401


## 発生するエラー
そもそも、ドライバ付属のシェルスクリプトではDebian系とRedHat系以外のディストロでは動作しないような記述になっています。

```
	if [ "${DISTRO}" = "debian" ] ; then
		PACKET_MNG="${APT_GET}"
		LINUX_HEADERS="linux-headers-`uname -r`"
		TOOLS="build-essential"
		CMD="dpkg-query -l"
	elif [ "${DISTRO}" = "redhat" ] ; then
		PACKET_MNG="${YUM}"
		LINUX_HEADERS="kernel-devel-`uname -r`"
		TOOLS="gcc gcc-c++ make"
		CMD="${YUM} list installed"
	else
		echo "Sorry, your operating system ${DISTRO} is not supported."
		exit ${ERR_SCRIPT}
	fi
```

シェルスクリプトを参考に手動でDKMSを叩いていくと、 `dkms add` までは成功しますが次のビルドで落ちます。

```
Error! Bad return status for module build on kernel : 6.0.x
```

## TL;DR
いわゆる開発版のソースコードが下記リポジトリで提供されているのでそれを使いましょう。

[https://github.com/Aquantia/AQtion](https://github.com/Aquantia/AQtion)

`var.h` を参考にバージョン番号を確認したら、下記のようなコマンドでDKMSでのインストールができるはずです。筆者は後述のAURを用いたのでこの内容は未確認です。気になる人は `dkms.sh` を見てみるとよいと思います。

```
$ cd AQtion
# バージョン番号は適時置き換えてください
$ mkdir -p /usr/src/atlantic-2.5.5
$ cp -r * /usr/src/atlantic-2.5.5
```

`/usr/src/atlantic-2.5.5/dkms.conf` を作成し内容を記入します。

```
PACKAGE_NAME="atlantic"
BUILT_MODULE_NAME[0]="atlantic"
PACKAGE_VERSION="2.5.5"
DEST_MODULE_LOCATION[0]="/kernel/drivers/net/ethernet/aquantia/atlantic"
AUTOINSTALL="yes"
```

完了したら `dkms` を叩いていきます。

```
$ dkms add atlantic/2.5.5
# 依存関係が足りないとビルド時にエラーが出るかもしれません。
# カーネルヘッダとgcc gcc-c++ makeあたりが入っているか確認しましょう。
$ dkms build atlantic/2.5.5
$ dkms install atlantic/2.5.5
```

Archユーザーの場合はAURにてPKGBUILDが公開されていますのでそちらも利用できます。

[https://aur.archlinux.org/packages/atlantic-dkms](https://aur.archlinux.org/packages/atlantic-dkms)

## あとがきとXG-C100C V2の物理的制約
私が入手したXG-C100C V2ですが、PCIe x4の形をしておきながら利用できるのはPCIe x2のみです。そのため、PCIe Gen3以上に対応しているポートでないと10Gbpsフルでの動作は見込めません。私のマザーボードは一つしかないGen3のポートがグラフィックボードで埋まってしまっていたので、仕方なくPCIe Gen2のポートにNICを挿しました。この状態ではGen2x2でのリンクとなり、実測で6Gbpsちょい程度の速度となります。

Ryzen3000世代のマザーボードですらそんな感じなので、皆様はお気を付けください。

以上です。