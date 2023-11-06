---
title: "Atcoderのアップデート(2023)で入ったGoのライブラリをメモ"
emoji: "🐙"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Go","AtCoder"]
published: true
---

細かいところまでは見てないので自分で確認してください．

# installation
アップデート用の[スプレッドシート](https://docs.google.com/spreadsheets/d/1HXyOXt5bKwhKWXruzUvfMFHQtBxfZQ0047W7VVObnXI/edit#gid=408033513&range=L5)から以下が入ることを確認．

```sh
pushd /tmp                                    
wget https://go.dev/dl/go1.20.6.linux-amd64.tar.gz
sudo tar -C /opt -xf go1.20.6.linux-amd64.tar.gz
popd
export PATH=$PATH:/opt/go/bin
# create project and install libraries
go mod init atcoder.jp/golang
go get -u github.com/emirpasic/gods/...
go get -u gonum.org/v1/gonum/...
go get -u github.com/liyue201/gostl/...
go get -u golang.org/x/exp/
```
# Go ver
本体はgo1.20.6(1.21ではない)

# github.com/emirpasic/gods/...
repository:https://github.com/emirpasic/gods

GoDS (Go Data Structures)とあるように，Goでいろんなデータ構造を作ってまとめたライブラリ．

# gonum.org/v1/gonum/...
repository:https://github.com/gonum/gonum

Goの数値計算ライブラリ．行列，線形代数，統計，確率分布等．グラフをプロットしたりもできる．Pythonでいうnumpyやmatplotlib的な立ち位置のようだ．2014年にはコミットがあるのでかなり気合の入ったプロジェクトに見える．

# github.com/liyue201/gostl/...
これまたGoでデータ構造とアルゴリズムを実装したライブラリ．C++ STLを意識しているようで，今まではC++だと標準だけどGoにはないライブラリがあり，解説のプログラムを移植できないということもあったけど，これで解決するかも．

# golang.org/x/exp/
標準ライブラリから外れたり，実験的だったりする機能が入っている．
golang.org/x/exp/slices，golang.org/x/exp/mapsは1.21からは標準ライブラリとしてslices，mapsとして追加された．

# 罠
max，minがbuilt-inになるのはGo 1.21から．さらなるアップデートが来るまでは自分で実装しよう．
スライスならsilces.Max()，silces.Min()を使ってもいい．

# 個人的に競プロで使いそうな機能
* Gonumの行列
* slices.Sort[]()
    sorts.Intsより速いらしい．
* godsのstack，queue
    gostlにもあるが，どちらにせよいちいちappendではなくメソッドとしてpush，enqueueが使えるのは楽
* slices.Contains[]()
* slices.Index[]()
    for文が1個減って見やすくなるかも．

全部見きれてないので見逃した要素が結構あるはず．気が向いたら追加する．

# poem
今までデータ構造は自作していたのでこれからも続けるつもり．勉強になるし．
競プロとして本当に速さを求めるときはライブラリを利用すべきでしょうが，理解せずライブラリ使ったら予想外の計算量になるので気をつけなければ(slices.Indexは線形探索なのでO(n)かかる，とか)．


# 参考文献
https://zenn.dev/ficilcom/articles/15180e8333a45c
