---
title: "Option、Result"
---

## Option 型

`Option` 型は値がない場合があることを示す型です。Java などで `null` を使って表現するような部分にこの型を使用できます。

`Option` 型は enum で定義されていて、簡単に書くと下記のような構成になっています。

```rust
// <T> というのはジェネリクスというもので、「どんな型でもここに入る」という意味になる。
// 型注釈時はたとえば、`Option<i32>`、`Option<String>` といった感じに書く。
enum Option<T> {
    Some(T),
    None
}
```

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

シンプルな説明になりますが、`Option` に関しては以上です。
