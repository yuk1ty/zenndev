---
title: "制御構文"
---

この章では Rust の制御構文を見ていきます。基本的に他の言語と同じ形で利用可能ですが、`loop` 構文などは初見の方もいるかもしれません。また、Rust におけるこれらの制御構文はすべて式となっており、式であることを利用すると、変数のスコープを小さく実装できる場合があります。

## if

他の言語にある `if` と同じですが、条件を書く部分に `()` が必要なくなっているのが注意ポイントです。

```rust
fn main() {
    let number = 3;

    if number < 5 {
        println!("5より小さい = true");
    } else {
        println!("5より小さい = false");
    }
}
```

ちなみに `if` は式であり、少々大げさですが上の内容は次のように書き換えることができます。`if` が式であるため、Rust には 3 項演算子は存在しません。不要なためです。

```rust
fn main() {
    let number = 3;
    let text = if number < 5 {
        "5より小さい = true"
    } else {
        "5より小さい = false")
    };
    println!("{}", text);
}
```

## for

Rust では `for in` という構文を使うことで、イテレータを走査することができます。

たとえば先ほど紹介した `Vec` 型からイテレータを取り出したい場合は、`iter()` あるいは `into_iter()` を使います。初心者のうちは、参照を取り出すと混乱すると思うので、値を取り出せる `into_iter()` を素直に使用しましょう。

```rust
fn main() {
    let nums = vec![1, 2, 3, 4, 5, 6, 7];

    for n in nums.into_iter() {
        println!("{}", n);
    }
}
```

:::message
（iter と into_iter の話を書く予定）
:::

また、数値を羅列したい場合は `0..99` のような書き方をできます。これは `range` と呼ばれるもので、`[start, end)` な値を生成してくれます。これを使用すると、たとえば下記のような Java コードは、

```java
// 何か適当な関数の中で。nums は何か数値が入った配列であるとする。
for (int = 0; i < nums.size(); i++) {
    System.out.println(nums[i]);
}
```

下記のように Rust では書くことができます。

```rust
for i in 0..nums.len() {
    println!("{}", nums[i]);
}
```

## loop

同じコードを永遠に実行します。他の言語で `while true {}` としていた箇所は、この `loop` という構文を使用します。

```rust
fn main() {
    loop {
        println!("永遠に Hello, World!");
    }
}
```

もしループを途中でやめる処理をはさみたい場合には、`break` というキーワードを使用して終了させることができます。

```rust
fn main() {
    let mut counter = 1;

    loop {
        if counter > 15 {
            break; // counter が15より大きくなるとここで処理を止める
        }
        println!("15回だけHello, World!: {} 回目", counter);
        counter += 1;
    }
}
```

ちなみに `loop` も式です。

```rust
fn main() {
    let mut counter = 0;

    let result = loop {
        counter += 1;

        if counter == 10 {
            break counter * 2;
        }
    };

    println!("The result is {}", result);
}
```

:::message
`loop` というキーワードがなぜ存在しているかについてですが、これは Rust の哲学のひとつである「したいことを素直に示す文法」を反映したものです。従来のプログラミング言語では、無限ループに関するワークアラウンドとしての `for (;;) {}` や `while true {}` が存在していましたが、これはパッと見で何をしたい記法かわからないという問題があります。もちろん、私たちプログラマはこのような記法にはもう慣れてしまっているので、何をしたいかは明確に把握することができます。が、プログラミング初心者だったころを思い出してみてください。この手の文の意味を、頭の中ですぐに把握することができたでしょうか？おそらく一瞬では難しかったはずです。しかし、`loop` と書かれていれば、そのキーワードのしたいことをすぐに把握できたはずです。

また、コンパイラ側の重要性という意味もあります。`loop` キーワードはいわゆる値を返さない式として解析の対象になります。Rust では、こうした決して値を返さない型を `!` と書くことができますが、まさにそれが使われています。先ほどの最後のサンプルではたしかに値を返しているように見えましたが、実際のところは `!` という型が type coercion という操作を起こしているだけで、`loop` の型はあくまで `!` です。[この話に関する解説はこちらのブログ記事に書きました。](https://blog-dry.com/entry/2020/11/02/000313)

値を返さないブロックは、コンパイラの「そこまで処理が到達するか」の解析に用いられます。この解析によって、コードの安全性をより高めています。
:::

## while

もちろん `while` も存在します。先ほどの例で出した break 付きのループは、`while` で書き直したほうがわかりやすいかもしれません。

```rust
fn main() {
    let mut counter = 1;

    while counter <= 15 {
        println!("15回だけHello, World!: {} 回目", counter);
        counter += 1;
    }
}
```

## match

後ほどさらに詳しく解説することになりますが、Rust にはパターンマッチングという記法が存在します。これは、他の言語でいうところの switch 文をより強力にしたもので、Rust を書く上では欠かせない記法です。一旦もっともシンプルな例のみ紹介しますが、後に登場する enum という構文でさらに詳しく解説する予定です。

```rust
fn main() {
    let num = 0;
    let msg = match num {
        0 => "zero",
        1 => "one",
        2 => "two",
        _ => "other"
    };
    println!("{}", msg); // zero
}
```

## 演習

配列から指定した数値を取り出す関数を作ってみましょう。1 つでも見つかった場合には「found!」、そうでない場合には「not found!」という文字列を返すものとします。

関数のシグネチャは下記のようになります。

```rust
fn find(source: Vec<i32>, target: i32) -> String
```

:::details 答え

このように `for` ループの途中で値を返したい場合は、`return` が使用できます。もし return がない場合、ここは文として認識され、ユニット型が返ることになります。セミコロンセンシティブな文法、少し難しいですね。慣れが必要な部分ではあります。

```rust
fn find(source: Vec<i32>, target: i32) -> String {
    for s in source.into_iter() {
        if s == target {
            return "found!".to_string()
        }
    }
    "not found".to_string()
}

fn main() {
    let vec = vec![1, 2, 3, 4];
    let msg = find(vec, 3);
    println!("{}", msg);
}
```

:::
