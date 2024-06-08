---
title: "整数問題定理集(フェルマーの小定理、ax+by=1の整数解)"
emoji: "🐼" 
type: "idea" # tech: 技術記事 / idea: アイデア記事
topics: ["math"] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---


もとは"そろそろRSA暗号を理解しよう(途中)"というタイトルで始めて，RSA暗号の理解に必要な数学的な知識をまとめていたらボリュームが増えたため途中で放出．
## 定理
1. 積に対してmod
    $a\equiv b \pmod n$なら$ac\equiv bc\pmod n$
2. フェルマーの小定理
    $p \in \mathbb{P}$，$x$が$p$の倍数でないとき、$x^{p-1}\equiv 1  \pmod p$
3. $a,b$が互いに素なら$ax+by=1$は整数解を持つ

## 証明
1. $a\equiv b \pmod n \to ac \equiv bc \pmod n$
    仮定は以下．

    $$
    \begin{gather}
        q_a,q_b,r \in \mathbb{Z}\\
        0<q_a,\ 0<q_b,\ 0 \leq r<n \\
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
            rc =: q_rn+r_r\  (r_r<n)\\
            ac = (q_ac + q_r)n + r_r \\
            bc = (q_bc  +q_r)n + r_r \\
            \therefore (acの剰余)=(bcの剰余)
        \end{gather*}    
        $$
        
    $\therefore a+c \equiv b+c$

2. フェルマーの小定理
    $p \in \mathbb{P}$，$x$が$p$の倍数でないとき、$x^{p-1}\equiv 1  \pmod p$ ,または$x^{p}\equiv x  \pmod p$ が成り立つ．

    [fel]
    使う定理
    * nが素数のときnCrはnで割れる．
        証明 https://zenn.dev/oksongh/articles/ncr_is_dividable_by_n
    
    証明　帰納法．$x^{p}\equiv x  \pmod p$を示す．
    1. $x=1$のとき
        $1=1$なので成立．
    2. $x=m$で成立するとする．($x^p\equiv x \pmod p$)
    2項定理を使う．

    $$
    \begin{align*}
        (m+1)^p &=\sum_{k=0}^p {}_pC_k m^k1^{p-k}\\
        k=0,pの場合を総和から出して\\
        &= m^p + 1^p + \sum_{k=1}^{p-1} {}_pC_k m^k1^{p-k}\\
                &= m^p + 1 + \sum_{k=1}^{p-1} {}_pC_k m^k\\
            仮定(m^{p-1}\equiv 1)よりm^p\equiv m\\
                &\equiv m + 1 + \sum_{k=1}^{p-1} {}_pC_k m^k \pmod p \\
            1 \leq k < pを満たすので\\            
            定理"nが素数のとき_n\mathrm{C}_rはnで割れる"を用いて\\
                &\equiv m + 1 \pmod p \\
            となるので成立.
    \end{align*}
    $$
    $\therefore x^{p}\equiv x  \pmod p$ 


3. $a,b$が互いに素なら$ax+by=1$は整数解を持つ[so]
    * 補題（$r(k)$の単射性）
        $a,b$を互いに素な正整数とする．
        整数kに対し$r(k)$は$ak\equiv r(k) \pmod b$ を満たすとする．
        整数 $k,l(0<k,l\leq b-1,k \neq l)$ に対し $r(k)\neq r(l)$が成立．

    * 補題証明
        $r(k)\equiv ak,r(l) \equiv al$ より
        
        $$    
        \begin{align*}
            r(k)-r(l) &\equiv ak - al\\
                    &=a(k-l)\\
                    a\neq 0 ,k\neq l なので&\neq 0\\
            
        \end{align*} 
        $$
        $\therefore r(k)\not \equiv r(l)$
    * 本命証明
        $a<b$ とする．
        $a,b$は互いに素なので $a\cdot1, a\cdot 2,a\cdot3, ...,a\cdot (b-1)\ (< ab)$ は$b$の倍数ではない．
        $ai(i=1,2,3,...,b-1)$を$b$で割った剰余は補題の$r(i)$ なため，$r(i) \neq 0$．
        $r(i) \equiv ai \pmod b$ より $r(i) \leq b-1$．
        まとめると$1 \leq r(1),r(2),...,r(b-1) \leq b-1$ であり，$r(i)$は$[1,b-1]$のいずれかに割り当てられる．（鳩の巣原理）
        $r(s)=1$となるように$s$を選ぶと，($as$を$b$で割った剰余が$r(s)$)
        
        $$
            as = bq + r(s) = bq + 1
        $$
        変形して $as+b(-q)=1$ となり，解 $(x,y)=(s,-q)$ が存在する．

# reference
[fel]https://manabitimes.jp/math/680
[so]https://mathscience-teach.com/koukoumath-seisuu4-5/
