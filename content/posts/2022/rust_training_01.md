---
title: "理解して覚えるRust 01 モチベーションと基本文法"
summary: "理解を怠らずにドキュメントを参照しながらRustを学習していく過程を記した忘備録です。今回はRustを学習するモチベーションと基本文法に触れていきます。"
date: 2022-01-26T00:11:15+09:00
draft: false
---
理解を怠らずにドキュメントを参照しながらRustを学習していく過程を記した忘備録です。対象読者は未来の自分です。

WordPressからHugoに乗り換えたことで、管理コストをかなり減らすことができそうです。実際は以前のブログがそのまま残っているため、管理コストは以前から変わらないのですが、Hugoは記事を書いたらGitHubにPushするだけで記事が更新されるので、サーバの維持に気をかける必要がありません。

## ユーザからのフィードバックを得たい

一方で、乗り換えで実現が難しくなったこともあります。読者からのフィードバックです。

WordPressではコメント欄や、おそらくNote的なハートボタンもプラグインで設置することができ、読者からのフィードバックを得やすくなっています。PHPでの対話的なAPIがなせる技ですね。Hugoは静的なHTMLを生成するので、そのような読者からのフィードバックを得ることができません。

それでは寂しいので、Note的なハートボタンを作っていきます。

## 技術選定の仕方

ハートボタンは下記のような方法で作成しようとしています。

```
フロントエンド: JavaScript + WebComponents
サーバサイド: Rust
インフラ: 自宅サーバ
```

フロントエンドはシンプルにJSを使います。ボタンをクリックしてAPIを叩くだけなので、静的型のような高級な機能は必要ありません。

サーバサイドには今回の主役であるRustを採用します。単純に興味があったということ、メモリを管理しやすいということから長期的に運用するサービスに向いていそうなことから採用しました。
余談ですが、私が運用している早押しボタンオンラインではメモリリークが発生している箇所があり、定期的な再起動を必要としています。Rustを採用することで、堅牢なコードを書けるように意識できるようになりたいという思惑もあります。

インフラは自宅サーバです。長期運用かつDB利用なため、コストを下げたいためです。

## 簡単な設計をする

Noteにあるようなハートボタンを作成します。WebComponentsによりボタンが表示された場合、今表示しているページのURLでどれだけハートがクリックされたかをサーバに問い合わせ、表示します。

ハートボタンがクリックされた際、クリックされたことを示すAPIに接続し、サーバ内部でインクリメントします。

## 開発していく

右も左も知らない状態なので、基礎を知るため入門記事を一つ見つけてきました。

[RustでWebアプリケーションを作る - CADDi Tech Blog](https://caddi.tech/archives/416)

日本語での丁寧な解説もあり、わかりやすい記事です。この記事をベースに、横道にそれながら学習を行っていきます。

## Hello World with HTTP
最初に軽くAPIでHelloWorldできるようなコードを書きます。

```rust
use warp::{path};

#[tokio::main]
async fn main() {
    let hello = warp::path!("hello" / String).map(|name| format!("hello, {}", name));

    warp::serve(routes).run(([127, 0, 0, 1], 3030)).await;
}
```

ほとんど参考にさせていただいたコードのままですね。これをベースに気になったところを書いていきます。

### #[tokio::main]

[アトリビュート - Rust By Example 日本語版](https://doc.rust-jp.rs/rust-by-example-ja/attribute.html)

コンパイル時などで行う条件分岐などのメタデータを表しています。何となくですが `tokio` という存在は知っていたので、ここでは深追いせずに次に進みました。

### path!

[macro_rules! - Rust By Example 日本語版](https://doc.rust-jp.rs/rust-by-example-ja/macros.html)

`!` がつくところはマクロになります。Rustには強力なマクロ機能があり、よく使うような記述をこのように定義することができます。実際、 `path!` と定義されたところにこのコードがそのまま置き換えられて入るようなイメージです。

IntelliJの機能を使い定義元に飛ぶと、下記のようなコードになっていました。

```rust
#[macro_export]
macro_rules! path {
    ($($pieces:tt)*) => ({
        $crate::__internal_path!(@start $($pieces)*)
    });
}
```

上記コードで利用されていそうな `__internal_path` も軽く見てみます。

```rust
#[doc(hidden)]
#[macro_export]
// not public API
macro_rules! __internal_path {
    (@start) => (
        $crate::path::end()
    );
    (@start ..) => ({
        compile_error!("'..' cannot be the only segment")
    });
    (@start $first:tt $(/ $tail:tt)*) => ({
        $crate::__internal_path!(@munch $crate::any(); [$first] [$(/ $tail)*])
    });

    // 以下略
}
```

パターンマッチング？のような形に見えます。 `path!` 内の `$($pieces:tt)*` を、引数そのまま `__internal_path` に渡し、引数の型にマッチするような関数が選択され実行されそうですね。

今回は `warp::path!("hello" / String)` と呼び出しているので、 `/` が見える `(@start $first:tt $(/ $tail:tt)*) =>` にマッチするのではないでしょうか。

雰囲気は掴めたのでここで探索を終了します。マクロと言うと大学生時代に学んだCを思い出しますね。

### `.map(|name| format!("Hello, {}!", name))`

[クロージャ - Rust By Example 日本語版](https://doc.rust-jp.rs/rust-by-example-ja/fn/closures.html)

何となく即時関数の印象を受けましたが、その通りでした。 `name` を引数にとる関数ですね。Rustではクロージャと呼ぶそうです。実行すると `format!` の返り値である `"Hello, {name}!"` を返します。

JavaScriptで書くところのアロー関数と似てますね。 

```js
["kznrluk"].map((name) => "Hello " + name + "!")
```

## 関数を別ファイルに分割する

[ファイルの階層構造 - Rust By Example 日本語版](https://doc.rust-jp.rs/rust-by-example-ja/mod/split.html)

個人的言語機能で序盤に知りたくなる機能第一位である関数を別ファイルに分割する方法も調べておきます。nodeの時にはやり方が複数存在し混乱しました。

序盤で登場した `use` を利用するかと思いきや、ただのファイル分割の場合は `mod` を利用するようです。

簡単に文字列を返す関数をファイル `my_mod.rs` に定義します。

```rust
// my_mod.rs
pub fn get_some_text() -> &'static str {
    return "Hi";
}
```

これを利用して、 `hi` にアクセスされたら関数を叩き、 `Hi` と返すようにするコードがこちらです。

```rust
// main.rs
use warp::{Filter, path};

mod my_mod;

#[tokio::main]
async fn main() {
    let hello = warp::path!("hello" / String).map(|name| format!("Hello, {}!", name));

    let hi = warp::path("hi").map(|| my_mod::get_some_text()); // ここ

    // 後略
}
```

特に問題なく他のファイルにあるコードを利用することができました。

## まとめ

今のところコンパイラと殴り合うこともなく、平和に学習を進められている印象です。去年学習したGoに比べかなり *堅い* 印象が入門時点のコードから感じますが、まだ言語特有の仕様に悩まされるレベルには達していないようです。

次回は `use` と `mod` の違いやライフタイムについてまとめていきます。

次回 → まだ