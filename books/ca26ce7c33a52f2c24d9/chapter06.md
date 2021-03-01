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

## 次章

ちなみに、`Option` や `Result`、さらにはこれまで何度か登場してきた `Iterator` には、アダプタメソッドと呼ばれる便利関数がたくさん用意されています。他の言語にあるような `filter` や `map`、さらには `flat_map` なども存在しており、これらを使用することでより宣言的で簡潔に Rust を記述できるようになります。

このアダプタを次章ではいくつか紹介して、演習問題を通じて慣れていく予定です。また、アダプタの使用にはクロージャという機能を理解する必要があるので、それに関する解説も行います。

## 演習

`cat` コマンドを実装してみましょう。
