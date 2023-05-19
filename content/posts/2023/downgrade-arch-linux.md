---
title: "Arch Linux カーネル更新語のDKMSエラーとダウングレード"
date: 2023-05-11T01:00:00+09:00
draft: false
---

雑に `pacman -Syu` した後に、DKMSのインストールで祈る事は、私みたいな適当Arch Linuxユーザーは一度や二度は経験したことがあると思います。今回は祈りが届かずZFSのインストールに失敗したのでリカバリ方法をメモしておきます。

## エラー内容
DKMSは、カーネルのアップグレードに応じてカーネルモジュールを再ビルドするための仕組みです。カーネルのアップグレードに伴ってカーネルモジュールを再ビルドする必要があるソフトウェアは、DKMSを利用することでカーネルのアップグレードを気にせずに使えるようになります。

今回はシステムアップデートのため `pacman -Syu` を実行した後に、ZFSのインストールに失敗しました。エラーメッセージは以下の通りです。
```
==> dkms install --no-depmod zfs/2.1.11 -k 6.3.1-arch2-1
configure: error:
    *** None of the expected "capability" interfaces were detected.
    *** This may be because your kernel version is newer than what is
    *** supported, or you are using a patched custom kernel with
    *** incompatible modifications.
    ***
    *** ZFS Version: zfs-2.1.11-1
    *** Compatible Kernels: 3.10 - 6.2

Error! Bad return status for module build on kernel: 6.3.1-arch2-1 (x86_64)
Consult /var/lib/dkms/zfs/2.1.11/build/make.log for more information.
==> WARNING: `dkms install --no-depmod zfs/2.1.11 -k 6.3.1-arch2-1' exited 10
```

DKMSを利用するようなソフトウェアが動作しなくて困らないなんてことはないと思うので、Linuxカーネルのアップグレードをするときはきちんと依存が問題ないか確認しておくのが楽です。今回の `zfs-dkms` も、カーネル6.3で動作せんよとAURのコメントで掛かれていました。 ~~でもアップデート前にいちいちAURのページ見ないよなぁ。~~

まだ再起動してないなら `dmesg` で今のカーネルを確認しておきましょう。一番上の行にカーネルのバージョンが表示されます。今回の場合は `6.2.12-arch1-1` で起動していますね。
```
$ sudo dmesg | head -n 1
[    0.000000] Linux version 6.2.12-arch1-1 (linux@archlinux) (gcc (GCC) 12.2.1 20230201, GNU ld (GNU Binutils) 2.40) #1 SMP PREEMPT_DYNAMIC Thu, 20 Apr 2023 16:11:55 +0000
```

## カーネルを戻す

pacmanのWikiページに散々書かれているように、部分的なアップグレードは推奨されません。今回はダウングレードですが、システムの一貫性が損なわれるという点では同じです。カーネルに強く依存するソフトウェアがあったり、長い間 `pacman -Syu` してこなかったシステムでダウングレードするときはリスクになります。

[pacman - ArchWiki](https://wiki.archlinux.jp/index.php/Pacman#.E9.83.A8.E5.88.86.E7.9A.84.E3.81.AA.E3.82.A2.E3.83.83.E3.83.97.E3.82.B0.E3.83.AC.E3.83.BC.E3.83.89.E3.81.AF.E3.82.B5.E3.83.9D.E3.83.BC.E3.83.88.E3.81.95.E3.82.8C.E3.81.A6.E3.81.84.E3.81.BE.E3.81.9B.E3.82.93)

**きちんと** ダウングレードしたいときはArch Linux Archiveの仕組みを使って特定日時まで戻すべきです。

[Arch Linux Archive - ArchWiki](https://wiki.archlinux.jp/index.php/Arch_Linux_Archive#.E7.89.B9.E5.AE.9A.E3.81.AE.E6.97.A5.E6.99.82.E3.81.BE.E3.81.A7.E5.85.A8.E3.81.A6.E3.81.AE.E3.83.91.E3.83.83.E3.82.B1.E3.83.BC.E3.82.B8.E3.82.92.E3.83.AA.E3.82.B9.E3.83.88.E3.82.A2.E3.81.99.E3.82.8B.E6.96.B9.E6.B3.95)

とりあえず今回はカーネルだけ元に戻します。前回 `pacman -Syu` してからそれほど時間が経っていないことと、私は適当Arch Linuxユーザーだからです。

キャッシュを削除していないなら簡単です。キャッシュから問題なかったパッケージを拾ってきて、インストールするだけです。

```
sudo pacman -U /var/cache/pacman/pkg/linux-6.2.12-arch1-1.pkg.tar.xz
sudo pacman -U /var/cache/pacman/pkg/linux-headers-6.2.12-arch1-1.pkg.tar.xz
```

キャッシュを削除した場合でも、上記のようなtarファイルがあれば復旧できます。ALAやarchive.orgを探してみましょう。先ほどのリンクが役に立つはずです。

私のようにDKMSで失敗したユーザーは `linux-headers` も元に戻すべきでしょう。元に戻すと、pacmanのフックによりDKMSのビルドも走ります。逆に元に戻さないで `dkms install` しようとしても、ヘッダーが見つからないためエラーになります。よくできてますね。

