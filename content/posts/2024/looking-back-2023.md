---
title: "2023のふりかえりと思うこと"
date: 2024-01-01T18:00:00+09:00
summary: "年が明けてしまいましたが2023年をふりかえりと思うことを書き留めて置きたいと思います。2023年はSREとして本格的に活動を開始しながら、生成AIのキャッチアップにも時間を使った一年だったかなと思います。"
draft: false
---

年が明けてしまいましたが2023年をふりかえりと思うことを書き留めて置きたいと思います。2023年はSREとして本格的に活動を開始しながら、生成AIのキャッチアップにも時間を使った一年だったかなと思います。

## 生成AIと向き合う
2023年で一番エキサイティングだったのは間違いなく生成AI関連でしょう。仕事でも趣味でも自身の生活を大きく変えた技術だと言っても過言ではありません。

### A5000の購入
2023年の4月には生成AIをしっかり触りたくなり、ワークステーション向けのGPUであるRTX A5000を中古で購入しました。世代やCUDAコア数はRTX 3080とそれほど大きく変わりませんが、GPUメモリが24GB搭載されておりもう一枚つなげることで42GBにすることができる点が特徴です。間違いなく2023年のベストバイですね。

{{< figure src="/media/2024/nvidia-a5000.png" title="" >}}

購入してからは自宅サーバーに搭載し、LLMやStableDiffusionの推論はもちろんDreamBoothやLoRAの学習などにも使用していました。GPUメモリが24GBも搭載されていると、Stable Diffusion関連の学習はほぼほぼ問題なく、LLMでの推論も13Bくらいまでなら可能であり個人での実験には十分だったと思います。

### LLM/ChatGPT
ChatGPTにも大変お世話になりました。GPT3.5-Turboに初めて触れた時点ではへぇ結構すごいじゃんという程度でしたが、2月にGPT-4の先駆けとして出てきたBingと話した時の感動は忘れられません。コードを書いたり、計算、小説の執筆、ロールプレイ、全てをうまくこなしてくれるのです。これはある一つの時代が始まったなと思いました。

その後幾度となくBingに去勢が入りだんだん人間味がなくなっていき、やるせない気持ちになったのも覚えています。

もはやChatGPTやGitHub Copilotはコードを書くときには必須です。人間の私は全体の設計とインターフェイスの定義を行い、ChatGPTを使って詳細部分を埋める書き方が浸透してきており、特に冗長な表現がなく静的型付け & ダックタイピングなGoとの相性は最高で、私は更にGoがスキになりました。

#### CLI ChatGPTクライアント aski
GitHub Copilotがなかった時代にはChatGPTに毎回ファイルをコピーペーストして渡していましたが、いい加減手間に感じてきたためファイルをうまく扱えるCLIクライアントを作成しました。askiです。

https://github.com/kznrluk/aski

開発時点でGPT-4はAPI経由で利用できませんでしたが、まぁそのうち利用できるようになるだろうと先行して開発を進めました。GLOBパターンとプロファイルの分割機能でWeb版を超す完成度に仕上げられたかなと思っています。今でもコーディングのお供に利用しており、GPT-4のコンテキスト長が124kに拡張されてから更に利用用途が広がりました。やはりOSSは自分が使うものを作るのが大切ですね。

#### Slack ChatGPT Proxy SASA
2024年の最後の方はChatGPTを社内Slackにプロキシしながら利用統計を取るBotを作っていました。経緯については会社のアドベントカレンダーに投稿されていますのでぜひ読んでみてみてください。

[【コード公開】ゲーム開発でどう活かす？ChatGPTをみんなで使える仕組みづくり | CyberAgent Developers Blog](https://developers.cyberagent.co.jp/blog/archives/44803/)

技術的に面白いことはありませんが、社内での使われ方を見ると人それぞれプロンプトの書き方に違いがあって気づきを得ることができます。来年はこのBotと統計情報を用いて、もっと利用者を増やせるような工夫をしていきたいと思います。

### Stable Diffusion
画像生成AIも2023年4月あたりから本格的に触れ始めました。趣味でイラストを書くときの参考にしたり、社内デザイナと協力して業務効率化を模索したりしています。

#### 画像タグ付け支援ソフト Dagger
DreamBoothやLoRAの学習では、画像に対してキャプションを同名のテキストファイルでキャプションを指定する作業が発生します。一つ一つの画像を見ながらキャプションを設定していくのですが、これが地味に大変だったため効率的に作業できるようにVSCode向けの支援プラグインを作成しました。

[kznrluk/vscode-sd-caption-helper: Stable Diffusion's captioning assistance tool for VSCode](https://github.com/kznrluk/vscode-sd-caption-helper)

これは非常に簡単なプラグインで、開かれているテキストファイルと同じファイル名の画像を別タブで表示するというシンプルなものです。6月の時点でChatGPTを使えばプログラミング無限じゃん！と思って制作を開始しましたが、VSCodeのプラグインのように情報量が少なめかつAPIに変更があるものについては絶妙に利用できないコードが生成されるため、最終的には自分でドキュメントを読む羽目になった苦い思い出が蘇ります。

ただ、何十枚何百枚と画像を整理し始めると、各タグについて一斉に管理したかったり、タグをもとにフィルター・検索をしたかったりなどVSCodeのプラグインだけでは難しくなってきました。そのため、Next.jsを利用しWebブラウザで利用できるフロントエンドを作りました。

![](https://private-user-images.githubusercontent.com/29700428/253725631-9bfaae0a-382b-4e9c-bfff-12c17ec4d878.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3MDQwOTA1OTAsIm5iZiI6MTcwNDA5MDI5MCwicGF0aCI6Ii8yOTcwMDQyOC8yNTM3MjU2MzEtOWJmYWFlMGEtMzgyYi00ZTljLWJmZmYtMTJjMTdlYzRkODc4LnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNDAxMDElMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjQwMTAxVDA2MjQ1MFomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWJmZDYzMGEzYzNhMzYwYjIzZjUwZTNhMmQ3ZGRjZjMwNDg0ZmM4ODI3M2UwYjNhMjlkYmM5Y2FlNDY2MjllZDMmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0JmFjdG9yX2lkPTAma2V5X2lkPTAmcmVwb19pZD0wIn0.pr2RuzU0-vtHkrmKZaeHzymy2_oKe0RXVl7PJmDzKJ4)

[kznrluk/dagger: 🗡️Image Captioning Editor for Stable Diffusion](https://github.com/kznrluk/dagger)

久しぶりのWebフロントエンドということもあり、ダーッと数日程度で書き上げることができました。Webフロントエンドはフィードバックループが短いので開発してて楽しいですね。これが完成したことでタグ付けの負担をかなり軽減させることができました。

### AIどうなんだ？
AIについて思うことはたくさんあるのですが、自身の言葉でまとめる機会もなかったのでここに書いておこうと思います。

#### AIとコーディング
コーディング分野は言わずもがな便利になりました。Next.jsやSlack APIなど利用時にドキュメントを読み込む手間がほとんどなくなり、作りたいものができたあとの腰の重さが減りました。ドキュメント読む面倒くさいポイントを飛ばし、熱量があるうちに設計やデザインなど開発楽しいポイントまで持っていくことができるので作品の質や量がぐっと増えた印象です。

また、CLIツールと相性が良いのもありがたい点です。kubectlを特定の形式で出力したり、それを元に簡単な統計情報を出したりするのが簡単になりました。これがない状態では、まずはコマンドの出力を調べて構造を把握し、必要な情報を他のコマンドと繋げて処理する...みたいなパズルゲームが必要で、どうもやりにくい思いをしていました。

AIに仕事が取って代わられるかは不明ですが、エンジニアリングにおいては人間が代替されるのではなく、インターフェースが代替されるようになるのではないかなと思っています。IDEやクラウドコンソールとLLMが統合した未来、今まで指示の仕方がコードやUIの操作だったものが言語による指示に代替されそうです。そうすると、LLMにわかりやすいようにインターフェースを定義したり言語化できない条件を噛み砕いて説明するようなスキルが求められそうですね。

また、LLMや生成AIを自社製品に取り入れた後、どのように制御するかというのが新たな仕事になりそうです。年末にauが生成AIを用いて入力したワードから画像を生成するというキャンペーンを行っていましたが、既存の創作物をもとにしたであろう生成物が出ると一部で話題になっていました。ある程度ブラックリスト形式でNGワードリストを作成していたようですが、それだけだと制御は難しかったようです。

[MYイマジナリメーカー | au × 屋根裏のラジャー MEET YOUR IMAGINATION](https://rudger.au.com/imaginary/)

[イマジナリメーカー - 検索 / X](https://twitter.com/search?q=%E3%82%A4%E3%83%9E%E3%82%B8%E3%83%8A%E3%83%AA%E3%83%A1%E3%83%BC%E3%82%AB%E3%83%BC)

生成AIで生成された作品を人間が確認しないと安心できないのが現状ですが、このあたりの制御をうまく行うことができればもっと活用の幅が広がるのでは、と感じています。

---

AIは人間の初手を拡張する「高速道路」になっています。

エンジニアリングでは先人が苦労しながら整備し初学者が入門しやすい学習環境のことを「知の高速道路」と呼んだりします。先人が苦労し作り上げた道路を走れば、より早く先に進めます。でも、高速道路の先にたどり着き、いざ自身が高速道路を作る番になったときに、高速道路の下に埋もれた技術や基礎の作り方を知らなければ延伸することはできないでしょう。

生成AIも同じですね。生成AIが出力した内容が優れているものであっても、なぜそうなっているのかを理解しなければいつか頭打ちが来てしまいます。内容をよく観察してなぜこの形なのか？もっと良い方法はないのか？と問い続ける姿勢を忘れないでいたいものです。

## Kubernetesと自宅クラスタ
2023年はKubernetesについてもかなり理解を深めることができました。去年の後半からサーバサイドからSREに転向したことで、社で利用するインフラ関連技術をキャッチアップする必要があったのですが、コーディングとは違う脳みそを使うTerraformやKubernetesマニフェストにかなり悩まされた記憶があります。

2023年夏頃からチームの上司と集中的にキャッチアップする機会を設けていただいたこともあり、今は通常業務でほとんど困ることはないレベルまで持ってこれたかなと感じています。同時期に自宅サーバーでKubernetesクラスタを動かし始め、会社のGitOpsを模倣しいくつかのアプリケーションを運用し始めたところからモチベーションも上がりました。

マイKubernetesクラスタ自体は2月頃にGKEを使って構築していたのですが、やはりクラウドを使うと面白い部分が隠蔽されるというか、仕組みをじっくり考えるスキがないですね。やはり手元で動かして壊して治すのが自分には向いているなと思いました。

## 一人暮らし
11月に一人暮らしを始めていました。

変な物件を条件にして物件を探していたので、4階なのにエレベーターが無いとか水圧が低いとかそういう細かい問題はありますが、掃除をしたり料理をしたりそれなりに楽しんで2ヶ月を過ごしております。インターネットも拠点間VPNで実家化したりして遊んでいました。

[賃貸無料ネットを拠点間VPNで攻略！Raspberry Piルーターを作ろう](https://zenn.dev/kznrluk/articles/e37777e8e1e40a)

社会人も三年目になると生活の基盤を整えるための資金もある程度あり、ものを揃えるときに「ちょっと良いものを」と選んでいたらカードが爆発してしまいました。特に大きいトピックもないのでかわりに買ってよかったものを軽くまとめておきます。

### 1. シモンズのベッド・マットレスセット(約15万)
友人にいいベッドを買え、人生の1/3は寝てるんだぞと言われていたので買いました。東京インテリアで設置組み立て込15万円でした。今までベッドを買ったことが無いのでこんなもんなのかなと思っていましたが、ニトリなどの家具量販店で買えばだいぶ安く揃えられたみたいです。

{{< figure src="/media/2024/simmons.png" title="" >}}

ベッドなんてなんでも寝れれば同じだろうなんて思っていましたが、実際のところ寝心地はかなりよくセミダブルのサイズも相まってかなり気に入っています。

### 2. 象印オーブンレンジ エブリノ(約5万)
料理でオーブンを利用することもあるだろうという気持ちから購入しました。大多数のオーブンレンジはオーブンの際に金属製の皿を利用することからレンジで温めてからオーブンを利用するときには器を載せ替えなければいけませんが、このエブリノはオーブンの皿が陶器製のためレンジからオーブンを直接利用することができます。

そのため、食材の下準備だけしてオーブンレンジに投入するだけで、唐揚げやカレー、ハンバーグなどの調理が完了するのでIHコンロを塞がず、使用する食器の枚数も削減できるのでかなり便利に使っています。料理をするのであればこのエブリノおすすめかもしれません。

### 3. FlexiSpot 昇降デスク Q2L(約5万)
引っ越しと合わせてデスクも購入しました。L字かつそれなりの大きさが条件で昇降である必要はなかったのですが、Amazonでちょうど良さそうなのがあったのでそれを購入しました。それがQ2Lです。

{{< figure src="/media/2024/q2l.png" title="" >}}

L字かつ大きめなので、ワーキングスペースが大きく取れるので良いです。特に私の場合は液晶タブレットを利用するので、その際に左右の手を置くスペースが確保できるのが嬉しいポイントです。正直なところ、昇降機能はそれほど利用していませんが細かく高さを決められるのは嬉しいですね。

---

一人暮らしはかなり初期費用がかかるものでしたが、失敗した買い物はなかったと思います。カードは爆発してしまいましたが、そこそこのモノを買えたために失敗率も下がったのかな、と思って納得することにしたいと思います。

## 2024年
2023年は生成AIとKubernetesについて理解が深まった年でした。生成AIは1998年に生まれた私にとって体験したことがない技術のイノベーションで、PCやインターネットの誕生を体験している人と同じイノベーションを得ているのではないかなと思っていますし、2024年もこれを使ってまずは楽しんでいこうと思っています。

本業のSREとしての立ち回りもインフラ周りに積極的にコミットできるようになれたかなと思います。が、現状SREではなくインフラエンジニアなのでは？と悶々とする気持ちもあるので、2024年は幅広めな技術スタックを応用してきちんとSREと胸を張って言えるようなエンジニアリングをしていきたいですね。

では、今年も何卒よろしくお願いいたします。