---
title: "ABC323C覚書と微解説"
emoji: "📘"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["AtCoder"]
published: true
---

# 意義
[公式解説](https://atcoder.jp/contests/abc323/editorial/7355)では最低限の事柄に絞って書かれている気がする．
ちょっと気になったところを書く．

# 解法の要点
最初の状態でボーナス含めた最高得点を算出する．
プレイヤー$i$がまだ解いていない問題のうち，高い配点の問題から順に解いていき，最初に計算した最高得点より大きくなるまでの問題数をカウント．

必要な計算は`max`,`sort`(降順)ができればよいとおもう．


# 問題設定の詳細　はっきり書かれていないところ
* $A_i$は100の倍数，ボーナス点が$i$点 ($1\le i\le 100$) 入る．
    したがって，総合得点の下2桁はボーナスによって$00-99$に一意に定まり，総合得点も一意に定まる（同点は発生しない）．

* "他のプレイヤー全員の現在の総合得点を上回る"必要がある
    この条件だけだと，最大得点で同点のときも問題を解く必要がある．その場合，プレイヤー$i$を除いた場合の最大値が必要になるので問題が複雑になってしまう．
    が，得点は一意なので考える必要がなくなる．

* 最高得点のプレイヤーはすでに"他のプレイヤー全員の現在の総合得点を上回"っている

# 不明点
公式解説には"入れ替える"という操作が書かれているが，ここはよくわからず

# poem
問題設定に穴がない
作問者ってすごいね
