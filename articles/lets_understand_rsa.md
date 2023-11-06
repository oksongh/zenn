---
title: "そろそろRSA暗号を理解しよう(途中)"
emoji: "🐼" 
type: "idea" # tech: 技術記事 / idea: アイデア記事
topics: ["math"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

## 使う定理
1. $a\equiv b \pmod n$なら$ac\equiv bc\pmod n$
2. $p \in \mathbb{P}$，$x$が$p$の倍数でないとき、$x^{p-1}\equiv 1  \pmod p$
3. $a,b$が互いに素なら$ax+by=1$は整数解を持つ

## 証明
1. $a\equiv b \pmod n \to ac \equiv bc \pmod n$
    仮定は以下．

    $$
    \begin{gather}
        q_a,q_b,r \in \mathbb{Z}\\
        0<q_a,0<q_b,0 \leq r<n \\
        a = q_an + r \\
        b = q_bn + r \\
    \end{gather}
    $$
    <!-- $a-b=(q_a-q_b)n$ -->
    式(3),(4)の両辺に$c$をかけて

    $$
    \begin{gather*}
        ac = q_acn + rc \\
        bc = q_bcn + rc \\
    \end{gather*}    
    $$

    1. $rc < n$のとき

        $$
            \begin{equation*}
                (acの剰余)=(bcの剰余)
            \end{equation*}
        $$

    2. $rc \geq n$ のとき

        $$
        \begin{gather*}
            rc := q_rn+r_r,r_r<n\\
            ac = (q_ac + q_r)n + r_r \\
            bc = (q_bc  +q_r)n + r_r \\
            \therefore (acの剰余)=(bcの剰余)
        \end{gather*}    
        $$
        
    $\therefore a+c \equiv b+c$

2. $p \in \mathbb{P}$，$x$が$p$の倍数でないとき、$x^{p-1}\equiv 1  \pmod p$[fel]
    使う定理
    * nが素数のときnCrはnで割れる．
        証明 https://zenn.dev/oksongh/articles/ncr_is_dividable_by_n
    
    証明　帰納法を使う．
    1. $x=1$のとき
        $1=1$なので題意は成立．
    2. $x=m$で題意が成立するとする．($x^p\equiv x \pmod p$)
    2項定理を使う．

    $$
    \begin{align*}
        (m+1)^p &=\sum_{k=0}^p {}_pC_k m^k1^{p-k}\\
        k=0,pの場合を総和から出して\\
        &= m^p + 1^p + \sum_{k=1}^{p-1} {}_pC_k m^k1^{p-k}\\
                &= m^p + 1 + \sum_{k=1}^{p-1} {}_pC_k m^k\\
            仮定(m^{p-1}\equiv 1)よりm^p\equiv m\\
                &\equiv m + 1 + \sum_{k=1}^{p-1} {}_pC_k m^k \pmod p \\
            k\in [0,p-1]を満たすので\\            
            定理"nが素数のとき_n\mathrm{C}_rはnで割れる"を用いて\\
                &\equiv m + 1 \pmod p \\
            となるので題意成立.
    \end{align*}
    $$
    $\therefore x^{p-1}\equiv 1  \pmod p$


3. $a,b$が互いに素なら$ax+by=1$は整数解を持つ[so]
    * 補題
        a,bが互いに素な正数とする．
        整数kに対し$r(k)$は$ak\equiv r(k) \pmod b$ を満たすとする．
        整数 $k,l(0<k,l\leq b-1,k \neq l)$ に対し $r(k)\neq r(l)$．

    * 証明
        $r(k)\equiv ak,r(l) \equiv al$ なので
        
        $$    
        \begin{align*}
            r(k)-r(l) &\equiv ak - al\\
                    &=a(k-l)\\
                    a\neq 0 ,k\neq l なので&\neq 0\\
            
        \end{align*} 
        $$
        $\therefore r(k)\not \equiv r(l)$
    * 証明
        $a<b$ としておく．
        $a,b$は互いに素なので $a\cdot1, a\cdot 2,a\cdot3, ...,a\cdot (b-1)$ は$b$の倍数ではない．
        よってこれらを$b$で割った剰余$r(i)(i=1,2,3,...,b-1)$は，$r(i) \neq 0$ ，つまり $1 \leq r(i) \leq b-1$ が必要．
        また，$r(i) \neq r(j)(i\neq j)$ なので $r(i)$ は $[1,b-1]$のいずれかに割り当てられる．（鳩の巣原理）
        $r(s)=1$となるように$s$を選ぶと，(asをbで割った剰余がr(s))
        
        $$
            as = bq + r(s) = bq + 1
        $$
        変形して $as+b(-q)=1$ となり，解 $(x,y)=(s,-q)$ が存在する．



 

# reference
[fel]https://manabitimes.jp/math/680
[so]https://mathscience-teach.com/koukoumath-seisuu4-5/
