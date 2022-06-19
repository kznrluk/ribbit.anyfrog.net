---
title: "WSL2上のLinuxからホストのWindowsにアクセスするにはmDNSを使うのが一番楽"
date: 2022-06-19T14:07:26+09:00
draft: false
---

WSL2を使っているWindowsでは、 `localhost` の特定ポートへのアクセスがよしなにWSL2へ転送されます。例えば、WSL2上のUbuntuで `80`番ポートをLISTENしているWebサーバに、Windows上のブラウザから `localhost:80` でアクセスすることができます。それとは逆に、Windowsで動いているWebサーバにはWSL上のUbuntuから`localhost:80` ではアクセスできません。WindowsのIPアドレスを特定して直接アクセスする必要があります。

WSL2上から見えるホストのWindowsのIPアドレスは不定です。これを取得するために、以前から[`ip` コマンドを用いた方法](https://qiita.com/samunohito/items/019c1432161a950892be)がありましたが、mDNSを使うと幾許か簡単に、そして動的にIPアドレスを取得できます。

## 参考
[aem - Access a localhost running in Windows from inside WSL2? - Stack Overflow](https://stackoverflow.com/questions/64763147/access-a-localhost-running-in-windows-from-inside-wsl2)

## tldr
mDNSが使える環境であればWindowsのホスト名に `.local` を付けたホスト名にアクセスすることで名前解決が可能。

```shell
${hostname}.local

// Windowsのホスト名がMelaの場合
Mela.local

// 一般的にWSL2上のLinuxも同じホスト名を持っているので `hostname` コマンドも使える
$ echo `hostname`.local
Mela.local

$ nslookup `hostname`.local
nslookup Mela.local
Server:         172.25.176.1
Address:        172.25.176.1#53
```

## 使用例
Windows側に簡単なWebサーバを立て、Linuxからアクセスを試す。

**Windows側でサーバーを立てる**
```
> hostname
Mela
> python -m http.server 8000
Serving HTTP on :: port 8000 (http://[::]:8000/) ...
```

初回時にファイアウォールの警告が出るので、パブリックネットワークでも通信を許可する。

{{< figure src="/media/python-firewall.png">}}

**Linux側からアクセスする**

まずは `localhost` でアクセスできないことを確認する。
```
$ curl localhost:8000
curl: (7) Failed to connect to localhost port 8000: Connection refused
```

ホスト名を `Mela.local` に変更するとアクセスできる。
```
> curl Mela.local:8000
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
<title>Directory listing for /</title>
// ...以下略
```

**Dockerコンテナ内からアクセスする**

同一のローカルネットワーク内であれば名前解決が可能なので、当たり前だがDockerコンテナ内からもアクセスすることができる。

```
$ sudo docker run -it ubuntu:latest bash
root@18401e4b3383:/# curl Mela.local:8000
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<html>
<head>
```

以上です。