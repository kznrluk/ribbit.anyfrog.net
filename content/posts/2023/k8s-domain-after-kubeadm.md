---
title: "kubeadmで構築したクラスタに後からドメインを追加したときの証明書エラーを解決する"
date: 2023-11-29T12:00:00+09:00
draft: false
---

{{< figure src="/media/penguins_standing_ice_floe.png" title="" >}}

`kubeadm` で構築済みの自宅クラスタ、今までVPN or SSH経由でアクセスをしていましたが利用頻度が増えてきたこともあり、きちんとドメインを振ってアクセスしたくなりました。

ドメインの割り当てはそれほど難しいことではないのですが、 `kubectl` は独自の証明書を `.kube/config` に埋め込んでおり、それがアクセス元ドメインを検証しているため(certSANs)、割り当てた後に再生成を行う必要がありました。


```
Unable to connect to the server: x509: certificate is valid for aki0, kubernetes, kubernetes.default, kubernetes.default.svc, kubernetes.default.svc.cluster.local, not aki0.anyfrog.net
```

その手順が若干ややこしかったのでメモとして残しておきます。

## tldr

以下の手順で再生成を行う。 `kubeadm` は部分的に `kubeadm init` を行った時の作業をもう一度行うことができる。
```
# すでにある証明書の退避 証明書があると後述のコマンドで再生成されない
mkdir kube_backup && cd kube_backup
sudo cp -r /etc/kuberenetes/pki/* .
sudo rm -rf /etc/kubernetes/pki/*

# `--apiserver-cert-extra-sans` 付きで証明書を再生成
# 下記コマンドを実行した時点からkubeconfig adminを再生成するまでkubectlが使用できなくなる可能性がある
sudo kubeadm init phase certs all --apiserver-cert-extra-sans aki0.anyfrog.net

# kube-apiserverなどのStaticPodを再起動させる
# kubeadm公式ドキュメント: 手動による証明書更新を参照
mkdir manifests && cd manifests
sudo mv /etc/kubernetes/manifests/* .
sleep 20
sudo mv ./* /etc/kubernetes/manifests

# /etc/kubernetes/admin.conf を削除 古い証明書をもとに作成されているため
sudo rm -rf /etc/kubernetes/admin.conf

# 新しいadmin.confを作成
sudo kubeadm init phase kubeconfig admin

# 必要に応じてユーザーディレクトリの.kube/configも更新
rm -rf ~/.kube/config
sudo cp /etc/kubernetes/admin.conf ~/.kube/config
sudo chown yourname ~/.kube/config
```

## NG集
`history` を見ながらNG集をまとめておきます。

### configmapをeditしてcertSANsを追加する

```
kubectl edit -n kube-system configmap kubeadm-config
sudo rm -rf /etc/kubernetes/pki/*
sudo kubeadm init phase certs all
```

適応されず。

### デフォルトのconfigmapを吐き出して編集

```
sudo kubeadm config print init-defaults > data.yaml
# .apiServer.certSANSにドメインを追加
nano data.yaml
sudo kubeadm upgrade apply --config=data.yaml
```

適応されず。今考えると /etc/kubernetes/pki/* と /etc/kubernetes/admin.conf を消しておけば動いたかも。ただし、 `kubeadm config print init-defaults` はあくまでデフォルトの設定を吐き出すだけなので、きちんと設定を洗わないとクラスタの設定が変になるかもしれない。--configと一緒にupgradeするな！とWARNも出る。ハイリスクなのでやめておいたほうが良いかも。

---

いろんなところで `kubeadm config view` というコマンドを見るものの、これ使えなかったんですよね。以上です。