---
title: "nが素数のときnCrはnで割れる" 
emoji: "🐡" 
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: ["math"] # タグ。["markdown", "rust", "aws"]のように指定する
published: true # 公開設定（falseにすると下書き）
---

$$
\begin{aligned}
r \ _n\mathrm{C}_r &= r\  \frac{n!}{(n-r)!r!} \\
&=n\  \frac{(n-1)!}{(n-r)!(r-1)!} \\
&= n \ _{n-1}\mathrm{C}_{r-1} \\
\end{aligned}
$$


$$
\begin{aligned}
r \ _n\mathrm{C}_r  &= n \ _{n-1}\mathrm{C}_{r-1}\\
                    &= nk\ (k \in \mathbb{Z})
\end{aligned}
$$
$r$,$n$が互いに素なら、$k$は$r$の倍数。

<!-- $n \in \mathbb{P}, 1 \leq r < n$ のとき、nとrは互いに素。 -->

$$
    _n\mathrm{C}_r = n \frac{k}{r}=nk'
$$
となり、$_n\mathrm{C}_r$はnの倍数。
$n \in \mathbb{P},1 \leq r < n$ のとき，$r$と$n$は互いに素。したがって$n$が素数のとき、$\ _n\mathrm{C}_r$は$n$で割れる。

# ポイント
nとrが互いに素なら、という条件を見つけること。

# reference
https://manabitimes.jp/math/588
