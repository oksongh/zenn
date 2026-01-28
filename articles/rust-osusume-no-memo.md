---
title: "Rustの個人的に気に入っているポイントの解説"
emoji: "🎉"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: [rust]
published: true
---

内容はn番煎じだけど、自分でまとめたほうが便利なので。

## 変数がデフォルトで不変

変数がデフォルトで不変であり、不変のまま変更しようとするとコンパイルエラー発生。
```rust
fn foo(){
    let str = "abc";
    str = "def"; // 不変な変数を変更しようとしているためコンパイルエラー
    println!("{}", str);
}
```

`mut`をつけると可変になり、コンパイルが通る。
```rust
fn foo(){
    let mut str = "abc";
    str = "def"; // 可変なので正常に動作する。("abc"で初期化したの無駄だぞ、という警告は出る)
    println!("{}", str); // def
}
```
可変であってほしい変数より不変であってほしい変数のほうが使用頻度が高いため不変がデフォルトになっているとのこと。可変でも、大抵は初期化の時に変更する必要があるだけで、初期化以降は不変であってほしいことが多いので、次の式指向が役に立つ。

## 本当に不変

変数の変更を禁止できるとバグが減り、認知負荷が下がり処理が分かりやすくなるというメリットがある。他のメジャーな言語だと`final`や`const`をつけても再代入しか防がず、内部要素は変更できる、という事例がある。JavaだとUnmodifiableListがあるが、変更してもコンパイルエラーではなく実行時エラーしか出ない。(UnmodifiableListはリストの変更不可能なビューを提供するもので、変更を防ぐためのものではない)
完全に不変にするにはクラス設計を工夫するしかない。（JavaだとStringは不変なので本当に設計次第で可能）

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
```rust
fn foo() {
    // mutをつけると可変
    let mut vec = vec![1, 2];
    vec.push(3);

    // デフォルトで不変
    let vec2 = vec![1,2];
    vec2 = vec![3, 4]; // 再代入でコンパイルエラー
    vec2.push(3); // 変更でコンパイルエラー
}
```

## 式指向
変数の初期化に必要な処理を閉じ込めておくことで、外のスコープでは不要な変数が露出するのを防げる。`loop`も式だが、個人的にはあまり使わないので省略。
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

Rustのifは`if文`ではなく`if式`であり、値を返却する。
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

match式はswitch-caseを拡張したようなパターンマッチの機能で、数値、ブール値、文字列のような値だけでなく型(enum)でも分岐でき、値を返却する。Javaでは`switch式`で同様のことができる。
```rust
fn foo() {
    let version = 1;
    let status = match version {
        0 => "old",
        1 => "current",
        _ => "future",
    };
    println!("{}", status); // current
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

シャドーイングは同じ責務の一時的な変数の名前を考える必要がなくなる点で便利。match式などパターンマッチを使うと同じ責務の変数が出てくるのでRustには合っていると思う。
```rust
fn foo() {
    let user: Option<User> = find_user(1234); // userはOption<User>型
    match user {
        Some(user) => println!("{:?} was found!", user), // userはUser型
        None => eprintln!("404 Not Found"),
    }
}
```

## null安全

つらいことにJavaでは参照型の変数(配列、クラスのインスタンスを持つ変数)はすべてnullで初期化できてしまい、安易にnullでの初期化に逃げる、という選択肢が取れてしまう。
nullでの初期化が意図したものなのかミスなのか分かりづらいため、注意力が要求される。

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

Rustだと安易にnull(相当のもの)で初期化できず、`Option`で扱うことを強制する。
```rust
fn foo(){
    let flag: bool = false;
    let str: String = None; // コンパイルエラー

    let mut str: Option<String> = None; // OK
    if flag {
        str = Some("foo".to_string());
    }
    println!("{:?}", str); // None
}
```

## 不変な初期化と式指向の相性がよい
Rustではifやmatch、ブロックが式なので初期化処理がシンプルに書ける。

変数を不変に初期化するにはRustでは`let x = 式;`、Javaなど他メジャー言語では`final var x = 式;`の形にする必要がある。
Javaでは`式`の部分にはリテラルや関数呼び出し、他の変数、switch式などを置けるが、Rustでは加えてifやブロック`{}`も置けるため、`let x = 式;`で初期化できる種類が多い。
ブロックは雑多な処理でもスコープを分けつつ`let x = 式;`で初期化できるので便利。

```rust
fn foo() {
    let x = "qux";
    let x = y;
    let x = bar();

    // 複数の処理を書ける
    let x = if z < 100 { "baz" } else { "quux" };
    let x = match a {
        100 => "100",
        200 => "200",
        _ => "other",
    };
    let x = {
        let mut x = 1 + 2 + 3*4;
        x = x * x;
        x + 10
    };
}
```

(個人的には`let x = yyy`で初期化が完了することを「一撃で初期化」などと呼んでいるのだが、逆に、式指向による「一撃で初期化」がなかったら、変数宣言のたびに`mut`をつけて回らなくてはならず、変数宣言のデフォルトが不変にならなかったのではないか？と想像している。)

### Javaでの実例
以下はJavaの例である。
不変にするためには、`ブロックで囲む`、`条件の組み替え`、`switch式`(後述)などがある。
(個人的には良いと思うが)`ブロックで囲む`はコーディング規約で認められるのか微妙で、シャドーイングはないために同じ責務だが別の変数名を付ける必要がある。`条件の組み替え`はロジックの検証コストがかかる。

#### 変更前コード
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
```java
public class Main {
    public static void main(String[] args) {
        boolean flag = false;
        boolean flag2 = false;
        boolean flag3 = true;
        
        final String str;
        {
            String tmpstr = "baz";
            if (flag) {
                tmpstr = "foo";
            }
            if (flag2) {
                tmpstr = "bar";
            } else if (flag3) { 
                tmpstr = "qux";
            }
            str = tmpstr;
        }
        System.out.println(str); // qux
    }
}
```

#### 条件の組み替え
デフォルト値もif文内に移動すると`str`への代入が一度になるため`final`を追加できるようになった。
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
不変にする方法はいくつかあるが、`let x = 式;`の形式で初期化できるのは`ブロックで囲む`、`条件の組み替え2`、`シャドーイング`、`match式`(後述)。この例で手頃なのは`ブロックで囲む`と`シャドーイング`、網羅性が高いのは`match式`。この例だと`条件の組み替え2`ではネストが深くなってしまったが、if式での初期化も`ブロックで囲む`と同様に使える場面もある。
Javaと比べたRustの文法のメリットは、`ブロックで囲む`ことがコーディング規約として認められていることだと思う。

#### 変更前コード
```rust
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
ifのネストが分かりにくい。
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

### パターンマッチ
パターンマッチ用の構文としてRustではmatch式、Javaではswitch式がある。
Javaにはタプルがないので相当するクラスを作る必要がある点だけ不便かもしれない。

上記の例では`bool`の条件が3つあるので取りうるパターンは2^3(=8)通り、ifで書くと考慮漏れが出ても気づきにくい。
条件の組み替えが許容できるなら、条件を網羅できるパターンマッチにするとよいと思う。

#### match式で書いた場合
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

## あとがき
個人的な感覚だが、Rustでは式指向のおかげで変数を不変にしやすくなっていると思う。特にブロック式が良くて、雑にスコープを分離でき、かつ返却値をみればデータの流れがすぐ分かる。

## メモ
Rustが「本当に不変」を実現するうえで所有権システムを使っているらしい？
確かに複数の箇所から参照されているときに1箇所だけ不変にしようとしても他の参照元から変更されたら不変ではなくなってしまうので参照元を制限する必要はありそう。
所有権システムがない言語で近い機能を実現するなら、あるスコープ内で破壊的変更を行わないことを示す`final`、`const`的な修飾子があればいいだろうか。

## 没案 Javaにあるnullでの一時的な初期化をRustでは一撃で初期化できるのでは

没理由：
初期化処理が複雑で、一旦`null`で初期化する必要がある場合、Javaでは`final`にできないが、Rust(式指向言語)なら複雑な処理をブロック式に閉じ込めて`let x = 式;`で不変に初期化できるため、変数を不変にするのと式指向とnull安全の相性が良い、という話ができるのではないかと考えていた。
しかしJavaでも「一旦`null`で初期化して、後で条件で置換する処理」は`条件の組み替え`や`ブロックで囲む`で結局回避可能だったので没。`switch`式があることも執筆途中で知った。

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
不変化できる。
```java
public class Main {
    public static void main(String[] args) {
        boolean flag = false;
        boolean flag2 = false;
        
        final String str;
        {
            String tmpstr = null;
            if (flag) {
                tmpstr = "foo";
            }
            if (flag2) {
                tmpstr = "bar";
            }
            str = tmpstr;
        }
        System.out.println(str); // null
    }
}
```
