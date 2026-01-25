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

## 本当に不変(イミュータブル)

変数の変更を禁止できるとバグが減り、認知負荷が下がり処理が分かりやすくなるというメリットがある。
他のメジャーな言語だと`final`や`const`をつけても再代入しか防がず、内部要素は変更できる、という事例がある。
JavaだとUnmodifiableListがあるが、変更してもコンパイルエラーではなく実行時エラーしか出ない。(UnmodifiableListはリストの変更不可能なビューを提供するのでしょうがない)
完全に不変にするにはクラス設計をちゃんとするしかない。（JavaだとStringは不変なので本当に設計次第）

```java
public class Main {
    public static void main(String[] args) {
        final List<String> list = new ArrayList<>();
        list = new ArrayList<>(); // 再代入はコンパイルエラーだが
        // 要素の変更は防がない
        list.add("A");
        list.add("B");

        List<String> unmodifiable = Collections.unmodifiableList(list);

        unmodifiable.add("C");  // 実行時エラー

        // 参照元を変えると反映される
        list.add("C");
        System.out.println(unmodifiable); // [A,B,C] 
    }
}
```

Rustでは不変にした場合、再代入も要素の変更もコンパイルエラーで防ぐ。

<!-- 変数宣言の際に不変にする(=`mut`をつけない≃`final`をつける)　⇒  初期化処理がどこで終わるか分かる　⇒　初期化以降は値が変わらないことが保証される　⇒　脳の負荷が下がる -->
```rust
fn foo() {
    // mutをつけると可変
    let mut vec = vec![1, 2];
    vec.push(3);

    // デフォルトで不変
    let vec2 = vec![1,2];
    vec = vec![3, 4]; // 再代入でコンパイルエラー
    vec.push(3); // 変更でコンパイルエラー
}
```

## 初期化と式指向の相性がよい

`if`や`match`、ブロックが値を返却してくれるので初期化処理がシンプルに書ける。
Javaと比較する。

<!-- 個人の感覚の話になるが、Rustでは式指向のおかげで初期化処理の都合で変数を可変にする必要が減っているように思う。 -->
<!-- 初期化処理がどこで終わるかはっきりするというメリットがあると思う。←それは不変にすることで説明済み -->
個人の感覚の話になるが、Rustでは式指向のおかげで変数を不変にしやすくなっていると思う。

変数を不変に初期化するにはRustでは`let x = 式;`、Javaでは`final var x = 式;`の形にする必要がある。
Javaでは`式`の部分にはリテラルや関数呼び出し、他の変数などを置けるが、Rustでは加えて`if`や`match`、ブロック`{}`も置けて
`let x = 式;`で初期化できるバリエーションが多い。
`if`、`match`、ブロックは複数の処理が必要な場合もこの形式で初期化ができるので便利。

```rust
fn foo() {
    let x = "qux";
    let x = y;
    let x = bar();

    let x = if z < 100 { "baz" } else { "quux" };
    let x = match a {
        100 => "100",
        200 => "200",
        _ => "other",
    };
}
```

<!-- 宣言の際に不変にする=mutが消える=finalをつける　⇒  初期化処理がどこで終わるか分かる　⇒　初期化以降は値が変わらないことが保証される　⇒　脳の負荷が下がる -->

(個人的には`let x = yyy`で初期化が完了することを「一撃で初期化」などと呼んでいるのだが、逆に、式指向による「一撃で初期化」がなかったら、変数宣言のたびに`mut`をつけて回らなくてはならず、変数宣言のデフォルトが不変にならなかったのではないか？と想像している。)

### Javaでの実例

以下はJavaの例である。
不変にするためには、`ブロックで囲む`、`条件の組み替え`などがある。
個人的には良いと思うが、`ブロックで囲む`はコーディング規約で認められるのか不明。

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

#### ブロックで囲む
個人的には問題ないと思うが、コーディング規約で認められるのか分からない。

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

#### 条件の組み替え
デフォルト値が分かりにくいかもしれない。

```java
public class Main {
    public static void main(String[] args) {
        boolean flag = false;
        boolean flag2 = false;
        boolean flag3 = true;
        
        final String str;
        
        if (flag) {
            str = "foo";
        }
        if (flag2) {
            str = "bar";
        } else if (flag3) { 
            str = "qux";
        } else {
            str = "baz";
        }
        System.out.println(str); // qux
    }
}
```

### Rustでの実例
Rustでそのまま書くと以下となる。
不変にする方法はいくつかあるが、`let x = 式;`の形式で初期化できるのは`ブロックで囲む`、`条件の組み替え2`、`match式`。
手頃なのは`ブロックで囲む`と`シャドーイング`、網羅性が高いのは`match式`である。
Javaと比べたRustの文法は`match式`があるのはもちろん、`ブロックで囲む`のもブロックの最後を見れば何で初期化されているのか明確になるので良いと思う。

```rust
// そのまま
fn foo() {
    let flag: bool = false;
    let flag2: bool = false;
    let flag3: bool = true;

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

#### ブロックで囲む
不変にするだけならブロックで囲むだけでできる。
初期化処理の終わりが分かりやすい。

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

:::details 条件の組み替え1(コンパイルエラー)

Rustではデフォルト値なしでの変数宣言はコンパイルエラー。

```rust
fn foo() {
    let flag: bool = false;
    let flag2: bool = false;
    let flag3: bool = true;

    let mut str: String; // コンパイルエラー

    if flag {
        str = "foo".to_string();
    }
    if flag2 {
        str = "bar".to_string();
    } else if flag3 {
        str = "qux".to_string();
    } else {
        str = "baz".to_string();
    }
    println!("{:?}", str);
}
```
:::

#### 条件の組み替え2
`if`のネストが分かりにくい。

```rust
fn foo() {
    let flag: bool = false;
    let flag2: bool = false;
    let flag3: bool = true;

    // 不変にできた
    let str: String = if flag {
        "foo".to_string()
    } else {
        if flag2 {
            "bar".to_string()
        } else if flag3 {
            "qux".to_string()
        } else {
            "baz".to_string()
        }
    };
    println!("{:?}", str); // qux
}
```

#### シャドーイング
シャドーイングで宣言しなおす。
ブロックで囲んでいない分、初期化処理の終わりが分かりにくい。

```rust
fn foo() {
    let flag: bool = false;
    let flag2: bool = false;
    let flag3: bool = true;
    // 一旦可変で宣言しておいて
    let mut str: String = "baz".to_string();

    if flag {
        str = "foo".to_string();
    }
    if flag2 {
        str = "bar".to_string();
    } else if flag3 {
        str = "qux".to_string();
    }
    // 不変にできた
    let str = str;
    println!("{:?}", str); // qux
}
```

#### `match`式
`bool`の条件が3つあるので取りうるパターンは2^3(=8)通り、`if`で書くと考慮漏れが出ても気づきにくいため、パターンの網羅ができる`match`式を使うとよく、さらに`let x = 式;`の形式で初期化できた。

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

### 余談 Javaのswitch式
Javaにも`match`式に似た、`switch`式が導入されている。が、タプル型がないためRustほど自由度は高くない。

## 没案 Javaでnullでの初期化

没理由：
初期化処理が複雑で、一旦`null`で初期化する必要がある場合、Javaでは`final`にできないが、Rust(式指向言語)なら複雑な処理をブロック式に閉じ込めて`let x = 式;`で不変に初期化できるため、変数を不変にするのと式指向とnull安全の相性は良いのでは、と考えていた。
が、「`null`で初期化して、後で条件で書き換えないといけない場合」が本当に必要な場面がなかった。

初期化処理が複雑で、安易に一旦`null`で初期化してしまっている例はありえそう。
その場合、`final`をつけられないが、ブロックで囲めばよいため、特に式指向と絡めて語ることがなかった。

```java
public class Main {
    public static void main(String[] args) {
        boolean flag = false;
        boolean flag2 = false;
        
        String str = null; // finalがつけられない
        if (flag) {
            // 複雑な条件
            str = "foo";
        }
        if (flag2) {
            str = "bar";
        }
        System.out.println(str); // null
    }
}
```
不変化。
```java
public class Main {
    public static void main(String[] args) {
        boolean flag = false;
        boolean flag2 = false;
        
        final String str;
        {
            String str2 = null;
            if (flag) {
                str2 = "foo";
            }
            if (flag2) {
                str2 = "bar";
            }
            str = str2;
        }
        System.out.println(str); // null
    }
}
```
