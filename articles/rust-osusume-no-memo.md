---
title: "Rustをおすすめするときにするはなし"
emoji: "🎉"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [rust]
published: false
---

内容はn番煎じだけど、自分でまとめたほうが便利なので。

## 変数がデフォルトで不変

変数がデフォルトで不変(イミュータブル)なので変更しようとするとコンパイルエラー発生。
```rust
fn foo(){
    let str = "abc";
    str = "def"; // 不変な変数を変更しようとしているためコンパイルエラー
    println!("{}", str);
}
```

`mut`をつけるとコンパイルが通る。
```rust
fn foo(){
    let mut str = "abc";
    str = "def"; // 可変なので正常に動作する。("abc"で初期化したの無駄だぞ、という警告は出る)
    println!("{}", str); // def
}
```
普通、可変であってほしい変数より不変であってほしい変数のほうが使用頻度が高い。
可変でも、大抵は初期化の時に変更する必要があるだけで、初期化以降は不変であってほしいことが多いので、次の式指向が役に立つ。

## 式指向
変数の初期化に必要な処理を閉じ込めておくことで、外のスコープでは不要な変数が露出するのを防げる。
`loop`も式だけど個人的にはあまり使わないので省略。

波括弧(`{}`)でブロックができる。ブロックは式で新しいスコープを作り、最後の要素を返却する。
```rust
fn foo() {
    let str = {
        let some_number = 100;
        format!("数値は{}", some_number) // 文字列の"数値は100"を返却し、strに束縛する。
    };
    println!("{}", str); // 数値は100
}
```

Rustの`if`は`if文`ではなく`if式`であり、値を返却する。
```rust
fn foo() {
    let str = if 1 < 100 {
        "foo"
    } else {
        "bar"
    };
    println!("{}", str); // foo
}
```

`match式`はswitch-caseを拡張したようなもので、数値、ブール値、文字列のような値だけでなく型(enum)でも分岐できる。
```rust
fn foo() {
    let version = 1;
    let status = match version {
        0 => "old",
        1 => "current",
        _ => "future",
    };
    println!("{}", status);
}
```

## シャドーイング
説明が難しいので以下引用。

> 前に定義した変数と同じ名前の変数を新しく宣言でき、 新しい変数は、前の変数を覆い隠します。Rustaceanはこれを最初の変数は、 2番目の変数に覆い隠されたと言い、この変数を使用した際に、2番目の変数の値が現れるということです。 以下のようにして、同じ変数名を用いて変数を覆い隠し、letキーワードの使用を繰り返します:
>
> ファイル名: src/main.rs
>
>```rust
>fn main() {
>    let x = 5;
>
>    let x = x + 1;
>
>    {
>        let x = x * 2;
>        println!("The value of x in the inner scope is: {}", x);
>    }
>
>    println!("The value of x is: {}", x);
>}
>```
>このプログラムはまず、xを5という値に束縛します。それからlet x =を繰り返すことでxを覆い隠し、 元の値に1を加えることになるので、xの値は6になります。 3番目のlet文もxを覆い隠し、以前の値に2をかけることになるので、xの最終的な値は12になります。 括弧を抜けるとシャドーイングは終了し、xの値は元の6に戻ります。

引用元 [The Rust Programming Language 日本語版 3.1.変数と可変性](https://doc.rust-jp.rs/book-ja/ch03-01-variables-and-mutability.html#%E3%82%B7%E3%83%A3%E3%83%89%E3%83%BC%E3%82%A4%E3%83%B3%E3%82%B0)

命名はコンピューターサイエンスの2つの難しいことのうちのひとつであり、シャドーイングは一時的な変数の名前を考える必要がなくなる点で便利だと思う。

## null安全
<!-- (以下はJavaの悪口ではない。Rustは後発のプログラミング言語だから対策がとれたというだけ。多分。) -->

つらいことにJavaの参照型の変数(配列、クラスのインスタンスを持つ変数)はすべてnullで初期化できてしまい、安易にnullでの初期化に逃げる、という選択肢が取れてしまう。
そのため、nullでの初期化が意図したものなのかミスなのか分かりづらい。
nullではないデフォルト値で初期化するよう気を付けるなど、人間の注意力が要求される。

```java
public class Main {
    public static void main(String[] args) {
        boolean flag = false;
        String str = null;
        if (flag) {
            str = "foo";
        }        
        System.out.println(str); // null
    }
}
```

Rustだと安易にnull(相当のもの)で初期化できず、わざわざ`Option`を扱わないといけない。
```rust
fn foo(){
    let flag: bool = false;
    let str: String = None; // コンパイルエラー

    let str: Option<String> = None; // OK
    if flag {
        str = Some("foo".to_string());
    }
    println!("{:?}", str); // None
}
```

一応Javaでも初期化されないケースがあるときはコンパイルエラーで教えてくれる。
```java
public class Main {
    public static void main(String[] args) {
        boolean flag = false;
        String str;

        if (flag) {
            str = "foo";
        }    
        
        System.out.println(str); // strが初期化されない場合がある、というコンパイルエラー
    }
}
```

## 変数の可変性と式指向とnull安全の相性の良さ

本題。複数の条件が絡む初期化処理はどこから始まってどこで終わるのか分かりづらくなることが多い。
さらに再代入があるのでconstやfinalをつけられないので、初期化以降も変更されていることに注意が向いてしまう。

```java
public class Main {
    public static void main(String[] args) {
        boolean flag = false;
        boolean flag2 = false;
        boolean flag3 = true;
        
        String str = "baz";

        if (flag) {
            str = "foo";
        }
        if (flag2) {
            str = "bar";
        } else if (flag3) { 
            str = "qux";
        }
        
        System.out.println(str); // qux
    }
}
```

Rustでそのまま書くと以下となる。

```rust
// そのまま
fn foo() {
    let flag: bool = false;
    let flag2: bool = false;
    let flag3: bool = true;

    // 値を変更するためにmutをつける必要がある
    let mut str: String = "baz".to_string();

    if flag {
        str = "foo".to_string();
    }
    if flag2 {
        str = "bar".to_string();
    } else if flag3 {
        str = "qux".to_string();
    }

    println!("{:?}", str); // qux
}
```

不変にするだけならブロックを使うだけでできる。
同じ名前の変数を新しく宣言すると2番目の。

```rust
fn foo() {
    let flag: bool = false;
    let flag2: bool = false;
    let flag3: bool = true;

    // 不変にできた
    let str: String = {
        // ブロック内は可変
        let mut str: String = "baz".to_string();
    
        if flag {
            str = "foo".to_string();
        }
        if flag2 {
            str = "bar".to_string();
        } else if flag3 {
            str = "qux".to_string();
        }
        str
    };
    println!("{:?}", str); // qux
}
```

boolの条件が3つあるので取りうるパターンは2^3(=8)通り、ifで書くと考慮漏れが出ても気づきにくい。
パターンを網羅するなら`match`が便利だ。

```rust
fn foo() {
    let flag: bool = false;
    let flag2: bool = false;
    let flag3: bool = true;

    let str: String = {
      
        let str = match (flag,flag2,flag3) {
            (true, _ ,_) => "foo".to_string(),      // flagがtrueの場合は他のフラグに依らず"foo"
            (false,true,_) => "bar".to_string(),    // flagがfalse,flag2がtrueの場合は"bar"
            (false,false,true) => "qux".to_string(),// flagがfalse,flag2がfalse,flag3がtrueの場合は"qux"
            (_,_,_) => "baz".to_string(),           // それ以外の場合は"baz"
        };
        str
    };
    println!("{:?}", str); // qux
}
```
ここまでくるとブロックが不要になり、`match式`一本で条件を整理できる。
```rust
fn foo() {
    let flag: bool = false;
    let flag2: bool = false;
    let flag3: bool = true;

    let str: String = match (flag, flag2, flag3) {
        (true, _, _) => "foo".to_string(),          // flagがtrueの場合は他のフラグに依らず"foo"
        (false, true, _) => "bar".to_string(),      // flagがfalse,flag2がtrueの場合は"bar"
        (false, false, true) => "qux".to_string(),  // flagがfalse,flag2がfalse,flag3がtrueの場合は"qux"
        (_, _, _) => "baz".to_string(),             // それ以外の場合は"baz"
    };

    println!("{:?}", str); // qux
}
```

感覚的な話になるが、Rustでは式指向のおかげで再代入なしで初期化が完了することが多いように感じている。（個人的には「一撃で初期化する」と呼んでいる。）
そのため、デフォルト値を入れたあと条件に応じて再代入という手順がなくなり、変数を可変に(mut)にする必要が無くなると思う。
逆に、式指向による「一撃で初期化」がなかったら、変数宣言のたびにmutをつけて回らなくてはならず、Rustが流行らなかったのでは？とさえ思っている。


Javaでも同じことができるのでは？

近いことはできるが、スコープを区切るためにブロックを使うのが認められるのか分からない。(個人的には良いと思う)
Rustだとブロックが値を返却するので、Rustのほうがデータの流れが分かりやすいと思う。

```java
public class Main {
    public static void main(String[] args) {
        boolean flag = false;
        boolean flag2 = false;
        boolean flag3 = true;
        
        final String str;
        {
            String str2 = "baz";
            if (flag) {
                str2 = "foo";
            }
            if (flag2) {
                str2 = "bar";
            } else if (flag3) { 
                str2 = "qux";
            }
            str = str2;
        }
        System.out.println(str); // qux
    }
}
```
