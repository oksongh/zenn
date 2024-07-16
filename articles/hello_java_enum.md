---
title: "「入門Javaのenum」のAppendix"
emoji: "🌊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [java]
published: true
---

JavaのEnumを友人に説明してくれと言われたので。
この記事は[入門Javaのenum][reference]を補完する目的で書いたため、まずは元の記事を軽く読んでからこの記事を読むとよい。

# Enumとは
JavaのEnum(列挙型)は特殊なクラスとして表現される。
メンタルモデルの説明をするため、実際にはどのように処理されているか僕は知らない。

# Class to Enum
まずはクラスからEnumっぽいものを構成していき最後にEnumに変換する方針で説明する。

## メインで完結
まずはクラス型変数をメインに定義にする。

```java
class Rank {
    double discountRate;
    Rank(double discountRate) {
        this.discountRate = discountRate;
    }
}
public class Main {
    public static void main(String[] args) {
        Rank bronze = new Rank(0.95);
        Rank silver = new Rank(0.9);
        Rank gold = new Rank(0.85);
        System.out.println(BRONZE);
    }
}
```

## static定数に変更する。
```java
class Rank {
    double discountRate;
    Rank(double discountRate) {
        this.discountRate = discountRate;
    }
}
public class Main {
    static final Rank BRONZE = new Rank(0.95);
    static final Rank SILVER = new Rank(0.9);
    static final Rank GOLD = new Rank(0.85);
    
    public static void main(String[] args) {
        System.out.println(BRONZE);
    }
}
```

## 定数の移動
Rankに関連した定数なのでRankクラスに移動する。
```java
class Rank {
    double discountRate;
    Rank(double discountRate) {
        this.discountRate = discountRate;
    }
    public static final Rank BRONZE = new Rank(0.95);
    public static final Rank SILVER = new Rank(0.9);
    public static final Rank GOLD = new Rank(0.85);
}
public class Main {
    public static void main(String[] args) {
        System.out.println(Rank.BRONZE.discountRate);
    }
}
```
この書き方が許されるのはいいだろうか？BRONZE等はstaticなのでクラスに帰属する変数で、普通のメンバー変数はdiscountRateだけ。
一応確認しておくと`Rank.BRONZE`とは、Rankクラスの定数Blockにアクセスする、の意味。
`BRONZE.discountRate`とは、Rankクラスのインスタンスのメンバー変数discountRateにアクセスする、の意味。

## カプセル化
Enumは定数を列挙するためのクラスなので勝手にインスタンスを作られては困る。そのためコンストラクタやメンバー変数をprivateにする。この状態でも`Rank.BRONZE`はアクセスできる。

```java
class Rank {
    private double discountRate;
    private Rank(double discountRate) {
        this.discountRate = discountRate;
    }
    public static final Rank BRONZE = new Rank(0.95);
    public static final Rank SILVER = new Rank(0.9);
    public static final Rank GOLD = new Rank(0.85);
}
public class Main {
    public static void main(String[] args) {
        System.out.println(Rank.BRONZE);
    }
}
```
## Enum化
このRankはEnumを使うと省略して書ける。BRONZEはpublic static final Rank型の定数として定義されたのが省略して書かれていることがわかったと思う。
改めて[入門Javaのenum][reference]を読んでほしい。


```java
Enum Rank {
    BRONZE(0.95),// public static final Rank BRONZE = new Rank(0.95);
    SILVER(0.9), // public static final Rank SILVER = new Rank(0.9);
    GOLD(0.85); // public static final Rank GOLD = new Rank(0.85);

    double discountRate;
    Rank(double discountRate) { // privateが省略されたコンストラクタ
        this.discountRate = discountRate;
    }
}
public class Main {
    public static void main(String[] args) {
        System.out.println(Rank.BRONZE.discountRate);
    }
}
```


[reference]:https://qiita.com/suke_masa/items/bd242617a2b9bb773cbd
