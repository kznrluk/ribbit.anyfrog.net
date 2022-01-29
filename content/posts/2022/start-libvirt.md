---
title: "libvirtで始めるリモート開発環境"
summary: "自宅サーバのリソースを使い、開発用の仮想環境をlibvirtで構築していきます。"
date: 2022-01-29T13:35:42+09:00
draft: false
---

私が普段何かを開発するとき、基本的に趣味の開発についてはWindowsを利用して行い、業務に関連する開発はmacOSを利用しています。

開発をWindowsで行っているのはコンプラやセキュリティの問題もありますが、一番は業務用のPCがもっさりしているからです。Apple SiliconeのmacOSなのに、ファイルIOがあるたびにセキュリティスキャンを行っているからか、いつももっさりしています。

そんな感じで、最近行っているRustの学習もWindowsで行っていたのですが、Rust的にWindowsはあまり重要視されていないようで、外部ライブラリのインストールなんかでよく詰まります。RustでDBを扱うdieselも、一筋縄ではインストールできませんでした。

そこで、今回は自宅サーバーにLinuxの仮想環境を作成して、そこで開発を行えるようにしたいと思います。

### virt-managerをWindowsから使う

virt-managerはバーチャルマシンのインストールや構成の変更などをGUIで行うことができるツールです。libvirt自体はCLIだけでも扱うことができますが、面倒くさがりなのでGUIを使います。

ちょっと前まではWindows用のバイナリがなく、WSL上で動かしてX410を設定して...とそこそこ面倒だったのですが、Windows 11上でWSL2が動いていれば適当にインストールして実行するだけで利用できます。Xが動くようになったWSL2の恩恵を感じますね。

```shell
// WSL2に入る
> bash

// virt-managerのインストール
$ sudo apt install virt-manager

// 実行
$ virt-manager
```

{{< figure src="/media/virt-manager-on-windows.png" title="コマンドをちょいちょいっと叩くだけで実行できる" >}}

### ホストマシンのUbuntuにlibvirtをインストールする

次は実際に仮想環境をホストするマシンにlibvirtをインストールしていきます。私の環境ではUbuntuを利用しているので、 `libvirt-daemon-system` というパッケージ名になります。

```shell
$ sudo apt install libvirt-daemon-system

// 私の環境ではリブートしないとvirt-managerからの接続がうまくいきませんでした
$ sudo reboot now
```

必要なのはこれだけです🐪

### virt-managerからホストマシンに接続する

#### SSH接続の準備
あらかじめWSL2上からホストマシンへSSHできる状態にしておく必要があります。

```shell
// 接続元のWindows
> bash

// ホストからWSL2内のホームディレクトリに.sshディレクトリをコピーして権限設定する
$ cp -R /mnt/c/Users/kznrl/.ssh ~/ && chmod -R 700 ~/.ssh

// 証明書にパスフレーズを設定している場合はssh-agentを使って入力なしで接続できるようにする
$ eval `ssh-agent` && ssh-add ~/.ssh/id_ed25519

// 一度接続してみる
// ここで接続できない or パスワードが求められるとvirt-managerでもうまく接続できない
$ ssh your_host
```

SSH接続できたらOKです。

#### virt-managerからUbuntu上のlibvirtに接続する

SSHで接続できることが確認できたら、virt-managerから接続してみます。

```
// 初回
$ virt-manager -c 'qemu+ssh://your_host/system?socket=/var/run/libvirt/libvirt-sock'

// 2回目以降は-cを省略してもOK
$ virt-manager
```

{{< figure src="/media/virt-manager-connected.png" title="正常に接続された状態" >}}


下記のようにエラーメッセージが出た場合はSSHの設定がうまくいっていないか証明書のパスフレーズ入力が必要な状態の可能性があります。上記手順をやり直しましょう。エラー文で書かれているように `x11-ssh-askpass` とかを入れてあげれば入力フォームを出して聞いてくれるのかもしれませんが今回は試していません。

```text
Unable to connect to libvirt qemu+ssh://your_host/system?socket=/var/run/libvirt/libvirt-sock.

Configure SSH key access for the remote host, or install an SSH askpass package locally.
```

### ゲストマシンを構築する

あとはGUIをポチポチするだけで構築を行うことができます。

## 詰まったところメモ

今回の作業で詰まったところをメモしていきます。

### 接続後、virt-managerの画面が真っ黒

WSL2上で動かすvirt-managerだと、仮想マシンを起動後の画面が真っ黒になってしまいました。仕方なく、代替としてビューア単体の `virt-viewer` をダウンロードして利用することにしました。

[Virtual Machine Manager Download](https://virt-manager.org/download/)

Windows 11でネイティブ動作する形式で配布されているのですが、接続先をいちいち入力しなければいけなかったり、SSHと組み合わせた接続がうまくいかなかったりと体感はあまり良くないです。

### GPUアクセラレーションが効かない

Spiceかつリモート接続という条件だと、GPUアクセラレーションを使うことができません。試しに適当なWebGLアプリケーションを開いてみましたが、CPU利用率が跳ね上がります。

普段の開発は可能だとは思いますが、3Dアプリの開発は無理そうですね。

## 思ったより簡単で便利

今回の構築では、GPUパススルーのような複雑な設定は行わずとりあえずホストだけ起動してみました。なにかにハマるかなと思い記事を書きつつ作業を進めていましたが、起動までは割とサクサクっとできてしまいました。

特にSpiceを用いた画面操作はLAN内であれば物理マシンで動いているかのような快適な挙動を見せてくれます。4kモニタにネイティブ解像度で表示しても、大きく遅延したりすることもなくかなり快適でした。

今後の開発はこの環境を試していきたいと思います。
