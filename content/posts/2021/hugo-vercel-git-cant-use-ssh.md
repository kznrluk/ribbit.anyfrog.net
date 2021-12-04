---
title: "VercelにHugoサイトをデプロイするとテーマが読み込まれない問題"
date: 2021-12-04T18:42:40+09:00
draft: false
---

{{< figure src="/media/hugo-vercel-congrats.png" title="いったいこれのどこがCongratulations!だというのか" >}}

### tldr
VercelはSSHでのGit Cloneに対応していない。GitのConfigを確認し、URLがSSHでないか確認する。

```plain
[remote "origin"]
        url = git@github.com:/kznrluk/hugo-PaperMod.git
        fetch = +refs/heads/*:refs/remotes/origin/*
```

```plain
[remote "origin"]
        url = https://github.com/kznrluk/hugo-PaperMod.git
        fetch = +refs/heads/*:refs/remotes/origin/*
```

### 詳細
Hugoで作成した本ページをVercelにデプロイすると画像のようにXMLが直接表示されている状態になってしまった。

{{< figure src="/media/hugo-vercel-page.png" title="RSSが送られてきている様子" >}}

VercelのDeployment Statusを見ると、下記のようなエラー文が出ていることを確認。
クローン時からWarningに ` Failed to fetch one or more git submodules` とあるように、SubmodulesのクローンでコケていてHugoがレイアウトファイルを見つけることができていないようだ。

```
Cloning github.com/kznrluk/ribbit.anyfrog.net (Branch: master, Commit: 2c3a733)
Warning: Failed to fetch one or more git submodules
Cloning completed: 425.703ms
Analyzing source code...
Installing build runtime...
Build runtime installed: 2.733s
Looking up build cache...
Build Cache not found
Installing Hugo version 0.89.4
Start building sites … 
hugo v0.89.4-AB01BA6E+extended linux/amd64 BuildDate=2021-11-17T08:24:09Z VendorInfo=gohugoio
WARN 2021/12/04 09:28:12 found no layout file for "HTML" for kind "page": You should create a template file which matches Hugo Layouts Lookup Rules for this combination.
WARN 2021/12/04 09:28:12 found no layout file for "HTML" for kind "home": You should create a template file which matches Hugo Layouts Lookup Rules for this combination.
WARN 2021/12/04 09:28:12 found no layout file for "HTML" for kind "taxonomy": You should create a template file which matches Hugo Layouts Lookup Rules for this combination.
WARN 2021/12/04 09:28:12 found no layout file for "HTML" for kind "section": You should create a template file which matches Hugo Layouts Lookup Rules for this combination.
WARN 2021/12/04 09:28:12 found no layout file for "HTML" for kind "taxonomy": You should create a template file which matches Hugo Layouts Lookup Rules for this combination.
```

### 解決

検索して解決。

https://github.com/vercel/vercel/discussions/4566#discussioncomment-479622
> Vercel supports Git submodules but only cloning them via HTTP or HTTPS, not SSH which is the default.

このエラー文じゃわからんだろ...😇 と思いつつ、GitのConfigを参照して下記のように修正。

```plain
[remote "origin"]
-       url = git@github.com:/kznrluk/hugo-PaperMod.git
+       url = https://github.com/kznrluk/hugo-PaperMod.git
        fetch = +refs/heads/*:refs/remotes/origin/*
```

見えるようになった？
