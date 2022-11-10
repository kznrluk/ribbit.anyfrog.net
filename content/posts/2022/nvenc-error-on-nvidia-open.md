---
title: "Linux上のFFMpegでNVENCがNo capable devices foundと言われ失敗する"
date: 2022-11-10T20:50:00+09:00
draft: false
---

最近自宅サーバーを整理し、Arch Linuxで再構築しました。GPUを使い、Docker上でハードウェアエンコードを行おうとしたときに下記エラーに遭遇したので記録しておきます。

## 発生するエラー

```
Press [q] to stop, [?] for help
[hevc_nvenc @ 0x55808ed05c80] OpenEncodeSessionEx failed: out of memory (10): (no details)
[hevc_nvenc @ 0x55808ed05c80] No capable devices found
Error initializing output stream 0:0 -- Error while opening encoder for output stream #0:0 - maybe incorrect parameters such as bit_rate, rate, width or height
Conversion failed!
```

## TL;DR
下記項目を確認する。

- 制限数以上で並列エンコードしようとしていないか？
GTXやRTXシリーズのGPUを利用する場合、ドライバ側で並列でエンコードできる最大数が決められています。私の使っているGTX1660 SUPERでは最大2並列です。上限を超えてエンコードを行おうとすると、上記のエラーが発生します。

- プロプライエタリなドライバを使用しているか？
オープンソースで公開されているいわゆる `nvidia-open` なドライバを利用すると、上記のエラーが発生してしまうことがあるようです。私の環境では一度 `nvidia-open` を削除し、代わりにプロプライエタリな `nvidia` ドライバーを導入することでエラーが解消しました。

私の環境では `nvidia-open` パッケージを導入していたのでこの不具合に遭遇しました。

## 環境

- Arch Linux
- NVIDIA Driver Version: 520.56.06
- Docker 20.10.21
- nvidia-container-toolkit 1.11.0-1

私の環境では `nvidia-container-toolkit` を使用してDocker上でGPUを使用していますが、恐らくホストPC上でも同じようになるのではないかなと予想しています。

## 少し調査
`nvidia-open` のリポジトリに下記のIssueを発見しました。

[nvenc not available · Issue #104 · NVIDIA/open-gpu-kernel-modules · GitHub](https://github.com/NVIDIA/open-gpu-kernel-modules/issues/104)

原因の特定はされているみたいですが、修正はまだみたいですね。ドライバーにパッチを当てれば動作するという情報もありますが、今回はおとなしくプロプライエタリなドライバを利用することにします。

今回はたまたま `nvidia-open` なドライバを導入した直後だったことから簡単に気が付くことができましたが、このエラーメッセージからオープンソースなドライバが原因であることに気が付くのは少々無理があるような気がします。改善されるといいですね。

以上です。