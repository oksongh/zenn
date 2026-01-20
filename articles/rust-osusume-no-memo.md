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

## 変数の可変性と式指向とnull安全の相性の良さ
(以下はJavaの悪口ではない。Rustは後発のプログラミング言語だから対策がとれたというだけ。)

以下に何らかの条件によって初期値を変えるコードをJavaで書いた。
つらいことにJavaの参照型の変数(配列、クラスのインスタンスを持つ変数)はすべてnullで初期化できてしまい、安易にnullでの初期化に逃げる、という選択肢が取れてしまう。
そのため、strをnullで初期化しているのは意図したものなのかミスなのか分かりづらい。
デフォルト値で初期化するよう気を付けるなど、人間の注意力が要求されがち。
null安全がないと困る一例である。

```java
public class Main {
    public static void main(String[] args) {
        boolean flag = false;
        boolean flag2 = false;
        
        String str = null;

        if (flag){
            str = "foo";
        }
        
        if (flag2){
            str = "bar";
        } 
        
        System.out.println(str); // null
    }
}
```

Rustだと
```rust
fn foo(){

}
```


Javaでも初期化されないケースがあるときはコンパイルエラーで教えてくれる。
```java
public class Main {
    public static void main(String[] args) {
        boolean flag = false;
        boolean flag2 = true;
        
        String str;

        if (flag){
            str = "foo";
        } 
        if (flag2){
            str = "bar";
        }
        
        System.out.println(str); // strが初期化されない場合がある、というコンパイルエラー
    }
}
```
