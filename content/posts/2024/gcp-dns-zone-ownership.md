---
title: "Google CloudでDNSゾーンを作成できないときはゾーン名を変えてみよう"
date: 2024-03-07T18:00:00+09:00
draft: false
---

忘備録です。Terraformを利用してゾーンを作ったり消したりしていたところ、あるタイミングで突然「所有権を確認しろ」とエラーが出るようになりました。

```
Error: Error creating ManagedZone: googleapi: Error 400: Please verify ownership of the '*****' domain (or a parent) at http://www.google.com/webmasters/verification/ and try again, verifyManagedZoneDnsNameOwnership
```

エラーメッセージには所有権を確認するためのURLが記載されていますが、このURLで登録を行っても解決しません。試行錯誤しましたが、結局ゾーン名を変えることで解決しました。

```
resource "google_dns_managed_zone" "common" {
-  name        = "common"
+  name        = "common-dev"
  dns_name    = "example.domain."

  dnssec_config {
    state = "on"
  }
}
```

ごくごく勝手な推測ですが、Googleの数あるDNSサーバー (`ns-cloud-d1.googledomains.com.` など) に一度登録されたゾーン名と同じゾーンの作成を行おうとすると、所有権確認が必要になるのかもしれません。

お試しください。以上です。

