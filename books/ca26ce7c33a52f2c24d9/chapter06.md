---
title: "Option、Result"
---

## Option 型

`Option` 型は値がない場合があることを示す型です。Java などで `null` を使って表現するような部分にこの型を使用できます。

`Option` 型は enum で定義されていて、簡単に書くと下記のような構成になっています。

`Some` は値があるときに使用され、`None` は値がないことを表現したい際に使用されます。

```rust
// <T> というのはジェネリクスというもので、「どんな型でもここに入る」という意味になる。
// 型注釈時はたとえば、`Option<i32>`、`Option<String>` といった感じに書く。
enum Option<T> {
    Some(T),
    None
}
```

### 実際に使ってみる

この `Option` 型を実際に使ってみましょう。配列に指定した数値があればその数値を取り出し、なければないという情報を返す関数とその実行プログラムを書きます。

```rust
fn find(source: Vec<i32>, target: i32) -> Option<i32> {
    for s in source.into_iter() {
        if s == target {
            return Some(s)
        }
    }
    None
}

fn main() {
    let vec = vec![1, 2, 3, 4];
    match find(vec, 3) {
        Some(value) => println!("value: {}", value),
        None => println!("not found!")
    }
}
```

- まず、関数の返り値を `Option<i32>` とします。これにより、先ほどの `Option` 型を返せるようになります。
- 次に、値があった場合は `Some()` で囲みます。これが、値が存在したという意味になります。
- 値がない場合は `None` と書いて返しておきます。これが、値がなかったという意味になります。
- `find` 関数を実行すると `Option` が返ってくるのですが、`Option` は enum です。したがって、パターンマッチングができます。これをもれなく使用します。

### if let

ちなみに、enum には実は `if let` という構文を使用できます。これは特定の 1 つの enum に対する処理のみを書き、それ以外の処理は無視するという手を使うことができます。パターンマッチするのはいいものの、する対象が 1 つしかなく、パターンマッチが冗長になる場合に使用できます。

Rust では、`Option` を使用するとこの `if let` を使用したコードが多く登場します。コードの可読性が上がるので、積極的に使っていきましょう。

今回のケースでは、`None` の場合も一応標準出力をすることにしていたので旨味は感じにくいかもしれませんが、`if let` を使用するとたとえば下記のように書き直すことができます。

```rust
fn main() {
    let vec = vec![1, 2, 3, 4];
    if let Some(value) = find(vec, 3) {
        println!("value: {}", value);
    } else {
        println!("not found!");
    }
}
```

ちなみに、`unwrap` というメソッドを使用すると、`None` だった場合はパニック（Java でいうところのランタイム例外のように、プログラムそのものを強制的に落とす操作）するものの、素早く `Option` 型を剥がすことができます。動作確認の際や、明らかに `Some` しか入り得ない場面などで使って、余計なパターンマッチングの省略をすることができます。

```rust
fn main() {
    let vec = vec![1, 2, 3, 4];
    // None だった場合は panic する
    println!("value: {}", find(vec, 3).unwrap());
}
```

シンプルな説明になりますが、`Option` に関しては以上です。

### まとめ

- 値がない可能性がある場合（他の言語で言う `nil` や `null`）には `Option` 型を使用できる。
- `Option` 型は `enum` になっている。
- `Some` は値がある場合、`None` は値がない場合を表現できる。
- `if let` という構文を用いて、`Some` のケースを取り出して…という書き方をすることがある。

## Result 型

`Result` 型は Rust でいうところのエラーハンドリングに該当します。Rust ではプログラム内でエラーとしてみなせる処理は、この `Result` 型を使って表現します。

`Result` 型も同様に enum で定義されていて、簡単に書くと下記のような構成になっています。

```rust
enum Result<T, E> {
    Ok(T),
    Err(E)
}
```

`Ok` には成功した場合の値、`Err` には、エラー終了とみなさせたい場合の値がそれぞれ入ります。`E` 型は、ジェネリクスである以上基本的にどのような値も入りますが、Rust ではエラー定義そのものも enum で行ってしまうことが多いです。

:::message
`Result` は Haskell でいうところの `Either` ではないのか、なぜわざわざ型を作ったという話はかなりたくさんされてきましたが、実は Rust には昔 `Either` 型も存在していました。両方ともありました。が、最終的にユーザーが `Result` 型しか使わなくなったという理由で、`Either` 型は廃止となりました。[このあたりの経緯はこのスクラップにまとめました。](https://zenn.dev/helloyuki/scraps/e5af11fecac719)ちなみに筆者も Scala プログラマだったため、Scala のときの癖で Scala をひたすら書いてから Rust を書くと、`Result` の型を逆に書いてよくコンパイラに怒られています。
:::

### 実際に使ってみる

たとえば、2 つの数値を使って割り算をする関数を考えてみましょう。

まず、0 での除算は定義不可能なので、エラーとして返すのが正解です。割り算を実行する際に除数が 0 でなければ正常系とし、実行する際に除数が 0 であった場合は異常系であるとします。エラー型も自分で独自定義しましょう。

次にもう一つ便宜的に実行可能条件を追加しましょう。両方の数値は正の数でなければならないものとします。これらを組み込んだ関数を考えてみましょう。

下記のようにしてコードを書いています。

- `CalcError` という enum を定義し、エラー型を用意しておく。
- `division` 関数を用意する。
  - 除数が 0 であった場合、`DividedByZero` エラーを返す。
  - 被除数または除数が 0 であった場合、`DetectedNegative` エラーを返す。
  - バリデーションチェックに問題がなけれあば、除算を実行する。
- `main` 関数内で `division` 関数を呼び出す。
  - `Ok` ならそのまま結果を出力する。
  - `Err` ならエラー内容をエラー出力する。

投入する数値を変更して、挙動を確かめてみてください。

```rust
enum CalcError {
    // ゼロ除算を行った場合に返すエラー
    DividedByZero,
    // 負の数が入っていた場合に返すエラー。中にその数値を入れる。
    DetectedNegative(i32),
}

fn division(dividened: i32, divisor: i32) -> Result<i32, CalcError> {
    if divisor == 0 {
        return Err(CalcError::DividedByZero);
    }

    if dividened < 0 {
        return Err(CalcError::DetectedNegative(dividened));
    }

    if divisor < 0 {
        return Err(CalcError::DetectedNegative(divisor));
    }

    Ok(dividened / divisor)
}

fn main() {
    let answer = division(4, 2);
    match answer {
        Ok(value) => println!("answer is {}", value),
        Err(err) => match err {
            CalcError::DividedByZero => eprintln!("ゼロ除算です"), // eprintln! マクロはエラー出力をできる。
            CalcError::DetectedNegative(num) => eprintln!("{} は負の数です。負の数は入れられません。", num)
        }
    }
}
```

なお、`Result` 型にも、`Option` 型と同様に `unwrap` メソッドが用意されています。`Ok` だった場合は、`Ok` が持つ値を返し、`Err` だった場合はパニックします。`Option` と同様に、動作確認時や明らかに `Ok` しか返さない場所で使用すると記述量の低減に繋がります。

### シンタックスシュガー

エラーを伝播させて、最後の最後だけ標準出力するなりでさばきたいというユースケースがあると思います。Rust ではそのようなユースケースに対応できるように、シンタックスシュガー `?` が用意されています。

たとえば 3 つのファイルを読み込んで、1 つが正常に読み込めたら次のファイル、読み込んだ際に見つからないなどのエラーが見つかった場合はエラーメッセージを返すというような実装をするとしましょう。下記のように書くことができます。

```rust
fn file_read(path: String) -> Result<String, String> {
    // std::fs::read_to_string を使うと、指定パスのファイルを読み込み、文字列を生成します
    match std::fs::read_to_string(path.clone()) {
        Ok(content) => Ok(content),
        Err(_) => Err(format!("ファイルの読み込みに失敗しました: {}", path)), // Err(_) の `_` は、その値を使用しない際に使えるイディオムです
    }
}

fn execute(path1: String, path2: String, path3: String) -> Result<String, String> {
    let file1 = file_read(path1)?;
    let file2 = file_read(path2)?;
    let file3 = file_read(path3)?;
    Ok(format!(
        "file1: {}\nfile2: {}\nfile3: {}",
        file1, file2, file3
    ))
}

fn main() {
    match execute(
        "path1".to_string(),
        "path2".to_string(),
        "path3".to_string(),
    ) {
        Ok(content) => println!("{}", content),
        Err(err) => println!("{}", err),
    }
}
```

たとえば上記の例で、`file1` の読み込みは成功し、`file2` の読み込みに失敗したとします。すると、処理自体は `file2` で止まり、そのまま `Err` として値が返されていきます。これがシンタックスシュガー `?` のもつ機能です。

ちなみに、使用できる条件としてはエラーの型が一致している必要があります。たとえば下記はコンパイルエラーになります。

```rust
// 先ほど用意したエラーは String 型の関数
fn file_read(path: String) -> Result<String, String> {
    match std::fs::read_to_string(path) {
        Ok(content) => Ok(content),
        Err(_) => Err("ファイルの読み込みに失敗しました: {}".to_string()),
    }
}

// エラーの型を bool に変えてみた。エラーの型を bool にすることはまったくないが…
fn file_read_return_bool(path: String) -> Result<String, bool> {
    match std::fs::read_to_string(path) {
        Ok(content) => Ok(content),
        Err(_) => Err(false),
    }
}

fn execute(path1: String, path2: String, path3: String) -> Result<String, String> {
    let file1 = file_read(path1)?;
    // 変えた関数を使用するように調整した
    // コンパイルは通らない
    let file2 = file_read_return_bool(path2)?;
    let file3 = file_read(path3)?;
    Ok(format!(
        "file1: {}\nfile2: {}\nfile3: {}",
        file1, file2, file3
    ))
}
```

コンパイルエラーは下記のように出てきます。下記が意味するところは、`bool` を `String` に変換するトレイト（このハンズオンの後半で出てきます、一旦は「そういう振る舞いを与える実装」の意味だと思ってください）を実装していないから、コンパイルを通すことはできないという旨です。

```
error[E0277]: `?` couldn't convert the error to `String`
  --> src/main.rs:19:45
   |
17 | fn execute(path1: String, path2: String, path3: String) -> Result<String, String> {
   |                                                            ---------------------- expected `String` because of this
18 |     let file1 = file_read(path1)?;
19 |     let file2 = file_read_return_bool(path2)?;
   |                                             ^ the trait `From<bool>` is not implemented for `String`
   |
   = note: the question mark operation (`?`) implicitly performs a conversion on the error value using the `From` trait
   = help: the following implementations were found:
             <String as From<&String>>
             <String as From<&mut str>>
             <String as From<&str>>
             <String as From<Box<str>>>
           and 2 others
   = note: required by `from`
```

シンタックスシュガー `?` を使用することで、`unwrap` や多重パターンマッチングを使用する必要がなくなり、実装がよりクリアになるというメリットがあります。使える場面では積極的に使っていきましょう。

### まとめ

- `Result` 型はエラーハンドリングを行いたい際に使用する。
- `Result` 型も `enum` になっている。
- `Ok` なら正常系、`Err` なら異常系を示す。
- ただ、毎回パターンマッチングでエラーハンドリングをしていると辛いので、`?` というシンタックスシュガーが用意されている。それをよく使う。

## 次章

ちなみに、`Option` や `Result`、さらにはこれまで何度か登場してきた `Iterator` には、アダプタメソッドと呼ばれる便利関数がたくさん用意されています。他の言語にあるような `filter` や `map`、さらには `flat_map` なども存在しており、これらを使用することでより宣言的で簡潔に Rust を記述できるようになります。

このアダプタを次章ではいくつか紹介して、演習問題を通じて慣れていく予定です。また、アダプタの使用にはクロージャという機能を理解する必要があるので、それに関する解説も行います。

## 演習

`cat` コマンド（に近いもの）を実装してみましょう。

やることは

1. 実行時引数にファイルパスを入力する。
2. 実行する。
3. 指定されたパスにあるファイルの中身を表示する。

です。

`std::fs::read_to_string` という関数を使用すると、指定されたパスに存在するファイルを読み込み、中身を文字列で受け取って表示します。

`std::fs::read_to_string` は、Result 型を返す定義を持ちます。正常系だった場合は `Ok` の中にファイルの中身の文字列が入ってきて、異常系だった場合は `Err` の中にエラーの内容が入ってきます。

`Err` だった場合は、エラーの内容を表示してみましょう。表示自体は `Err(err) => println!("{}", err)` を使うとできます。

```rust
fn run(path: String) {
    // 処理を書いてみましょう
}

fn main() {
    run("./src/main.rs") // main.rs を表示できるようになるはずです
}
```

次に、ファイルパスを自由に与えられるようにしてみましょう。つまり、実行時引数からファイルパスを取得し、そのパスを読み込むという処理に修正していきます。

たとえば `cargo run` で実行するなら、次のようにファイルパスを与えられます。

```
$ cargo run ./src/main.rs
```

実行時引数は、`std::env::args` という関数を使用すると受け取れます。`std::env::args().nth(1)` で、今回欲しい 1 つ目の引数を取得できるはずです。

`nth` 関数は、引数を取得してみた結果を `Option` に入れて返します。`None` だった場合は、引数が必要なことをユーザーに知らせましょう。

先ほど書いた関数にさらに処理を追加し、`cat` コマンドを完成させましょう。

- std::fs::read_to_string: https://doc.rust-lang.org/std/fs/fn.read_to_string.html
- std::env::args: https://doc.rust-lang.org/beta/std/env/fn.args.html

:::details 答え
エラーメッセージの内容などはお好みで。

```rust
fn run(path: String) {
    match std::fs::read_to_string(path) {
        Ok(content) => print!("{}", content),
        Err(why) => println!("{}", why),
    }
}

fn main() {
    let args = std::env::args();
    match args.nth(1) {
        Some(path) => run(path),
        None => println!("1つ目の実行時引数にファイルパスを入れる必要があります。")
    }
}
```

:::
