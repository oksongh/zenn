---
title: "備忘録: Diesel入門"
emoji: "🚂"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [rust, diesel]
published: true
---

# Getting Started
[Diesel](https://diesel.rs/)はRustのORMライブラリ。
本家[Getting Started](https://diesel.rs/guides/getting-started)に従って、Dieselを使ってみたときのメモ。


# 環境

* ubuntu 22.04
* Rust edition 2021
* postgresql 14

# インストール
diesel_cliのインストールにはcargo使うのが一番楽だと思う。（本家ではなぜか一番下にあるが）
```bash
cargo install diesel_cli
```

# 実行
show_posts.rsを実行すると以下のエラーが出た。
```bash
/usr/bin/ld: cannot find -lpq: No such file or directory
collect2: error: ld returned 1 exit status
```
本家のコラム"A note on installing diesel_cli"と同様で、libpqが見つからないというエラー。

## libpqの解決
ライブラリの検索
```bash
ldconfig -p | grep libpq
```
見つかったら、シンボリックリンクを張る
```bash
sudo ln -s /lib/x86_64-linux-gnu/libpq.so.5 /lib/x86_64-linux-gnu/libpq.so
```
見つからなかったらlibpqをインストール。

参考：https://please-sleep.cou929.nu/20080718.html

# プログラム全般
以下をファイル頭に書いておくと楽。
```rust
use self::models::*;
use diesel::prelude::*;
use diesel_demo::*;
```
# 個人的につまづいたところ
## posts, id, title, body, publishedとは

唐突に出てくるが、これはschema.rsからマクロ生成された構造体。
それぞれpostsはテーブル名と、id, title, body, publishedはカラム名と対応した**構造体**。
これらは[cargo expand](https://crates.io/crates/cargo-expand/)でschema.rsをマクロ展開してみるとなんとなくわかる。

つまりpostsの例では以下のuseは
```rust
use self::schema::posts::dsl::*;
```
以下と等価で、
```rust
use schema::posts::columns::id;
use schema::posts::columns::title;
use schema::posts::columns::body;
use schema::posts::columns::published;
use schema::posts::table as posts;
```
以下のような構造体を参照している。
```rust
pub struct table;
...
pub struct id;
...
pub struct title;
...
pub struct body;
...
pub struct published;
...
```

さらには、構造体がidといった一般的な名前なので、上記のことを知らなければ変数名と重複してしまっても気づきにくい。

cargo expandでマクロ展開してlessで見るとき
```bash
cargo expand --lib schema::posts --color always| less -R
```

## rust-analyzerがミスる
![](/images/hello-diesel/existfilter.png)
![](/images/hello-diesel/nofilter.png)

.filterの有無でinlay hintsが出なくなる。
どこのバグかわからないがrust-analyzer無しではrust書けないため困る。

追記:rust-analyzer側の既知のバグらしい。
https://github.com/rust-lang/rust-analyzer/issues/14607
