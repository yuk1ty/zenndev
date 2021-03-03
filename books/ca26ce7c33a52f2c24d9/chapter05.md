---
title: "独自のデータ構造を定義する"
---

Rust にはデータ構造を定義する方法が 2 つあります。`struct` と `enum` です。

## struct

`struct` というのは新しいデータ型を定義するための構文です。Go や C に登場する構造体と思ってもらって差し支えありません。

### 構造体を定義してみる

たとえば、次のように配送業者の注文情報を保持するためのデータ構造を定義できたりします。

`struct` というキーワードを使って、下記のようにデータ構造を定義できます。

```rust
struct Order {
    id: i32,
    pic: String,
    date: String,
    accepted: bool
}
```

たとえば、下記のようにデータを詰めて、いわゆるインスタンスを作成することができます。`struct` は、次のようにして各フィールドに値を詰めることで設定できます。

```rust
struct Order {
    id: i32,
    pic: String,
    date: String,
    accepted: bool
}

fn main() {
    let order = Order {
        id: 1,
        pic: "Yuki Toyoda".to_string(), // 担当者は私
        date: "2021-03-03T09:00:00Z".to_string(), // 2021/03/03の9時に受付
        accepted: false // まだ注文を受理していないので false
    };
    // ...続く
}
```

### 構造体に実装を追加してみる

`struct` に対しては、具体的にどのような値を扱うかに関する関数の実装を他の言語と同じように与えることができます。いわゆるメソッドというものです。メソッドの定義は、`impl` というキーワードを使って行えます。

ここで、Rust でよく使うイディオムとして `new` というものがあるので、それを実装してみます。`struct` にはコンストラクタのようなものは言語機能上ありませんが、コンストラクタを `new` という関数名でよく定義します。これは慣習のようなもので、非常によく使われます。

```rust
struct Order {
    id: i32,
    pic: String,
    date: String,
    accepted: bool,
}

impl Order {
    fn new(id: i32, pic: String, date: String, accepted: bool) -> Order {
        // 構造体の定義上のフィールド名と、構造体を生成する際に渡す変数名とが
        // 完全に一致する場合は、`id: id` といった繰り返しの記法が不要になる。
        Order {
            id,
            pic,
            date,
            accepted,
        }
    }
}

fn main() {
    let order = Order::new(
        1,
        "Yuki Toyoda".to_string(),
        "2021-03-03T09:00:00Z".to_string(),
        false,
    );
}
```

`Order::new` の `::new` は、それがスタティックなメソッドであることを示します。これは要するに Java でいうところの `static` メソッドと同じです。Java で書き直すとしたら、上述のコードは次のコードと対応しています。（注: Java では `new` は予約語なので、本来は関数名としては使用できない。このコードはコンパイルエラーとなる）

```java
public class Order {
    private final int id;
    private final String pic;
    private final String date;
    private final boolean accepted;

    private Order(int id, String pic, String date, boolean accepted) {
        this.id = id;
        this.pic = pic;
        this.date = date;
        this.accepted = accepted;
    }

    public static Order new(int id, String pic, String date, boolean accepted) {
        return new Order(id, pic, date, accepted);
    }
}
```

スタティックなメソッドを定義するだけでは困ってしまうので、インスタンスに紐づくメソッドも定義してみましょう。たとえば、`Order` 自身が保持する担当者と終了状況のペアを返すメソッドを定義してみましょう。

```rust
struct Order {
    id: i32,
    pic: String,
    date: String,
    accepted: bool
}

impl Order {
    // (先の続きに下記を追加する)

    fn quick_look(self) -> (String, bool) {
        (self.pic, self.accepted)
    }
}

fn main() {
    let order = Order::new(1, "Yuki Toyoda".to_string(), "2021-03-03T09:00:00Z".to_string(), false);
    let (pic, accepted) = order.quick_look();
    println!("担当者: {}, 完了: {}", pic, accepted);
}
```

メソッドの呼び出し自体は他の言語とまったく同じで変わるところはありません。`order` という変数に、`Order` のインスタンスを代入しておきます。そのインスタンスの情報から、`quick_look` というメソッドを引っ張ってきて呼び出しをします。

ここで、quick_look という関数を少し詳しく見てみましょう。まずメソッドの場合は、Python と同じように第一引数に `self` というキーワードを持ちます。まずこの状態ですと、イミュータブルな関数であることを示しています。

```rust
fn quick_look(self) -> (String, bool) {
    (self.pic, self.accepted)
}
```

ちなみに、超強引に Java で上記を実装するなら、次のようになります。Java には `Tuple` がないので自前で実装することになります。

```java
public class Order {

    public static class Tuple<A, B> {
        private final A a;
        private final B b;
        public Tuple(A a, B b) {
            this.a = a;
            this.b = b;
        }
    }

    private final int id;
    private final String pic;
    private final String date;
    private final boolean accepted;

    private Order(int id, String pic, String date, boolean accepted) {
        this.id = id;
        this.pic = pic;
        this.date = date;
        this.accepted = accepted;
    }

    public static Order create(int id, String pic, String date, boolean accepted) {
        return new Order(id, pic, date, accepted);
    }

    public Tuple<String, Boolean> quickLook() {
        return new Tuple(this.pic, this.accepted);
    }
}
```

`quickLook` はインスタンスに紐づくメソッドなので、先ほどとは違い `static` がつきません。こういった対応関係になっています。

話がそれました。現時点ではイミュータブルなメソッドのみを紹介してきました。ミュータブルなメソッド、つまり構造体の中身を書き換えるようなメソッドは提供できるのでしょうか。もちろんできます。先ほど `self` だったものを `mut self` とすると、構造体の中身を書き換えるような操作を行うことができるようになります。

注文の例でイミュータブルな操作を考える際にわかりやすいのは、注文が完了したと設定するための関数を用意することでしょう。ということで、`get_accepted` というメソッドを用意することにしましょう。下記のように書きます。

```rust
impl Order {
    fn get_accepted(mut self) {
        self.accepted = true;
    }
}
```

とても単純ですが、自身の `accepted` フィールドを `true` に書き換えています。こうすることで、注文の処理がすべて完了したことを意味します。

呼び出す際は、通常の `self` の場合とは違い、変数宣言時に `mut` を付与する必要があります。ない場合にはコンパイルエラーになります。

```rust
fn main() {
    let mut order = Order::new(1, "Yuki Toyoda".to_string(), "2021-03-03T09:00:00Z".to_string(), false);
    order.get_accepted();
    println!("{}", order.accepted);
}
```

:::message
`to_string` という関数が登場してきて、すこしまどろっこしいと感じたかもしれません。

Rust では、ダブルクオーテーションで囲んだ文字列の型は `&str` 型になります。これはサイズ固定の UTF-8 の文字列で、文字列の追加や削除などが不可能な形式です。メモリ上に保有する文字列データに対する参照を保持しているイメージです。要するに実データは保持しておらず、ただその文字列の領域を指し示すのみです。実データを持っていないので、その文字列に対する変更などは行なえません。

一方で、`to_string` という関数を呼び出すと、ダブルクオーテーションで囲まれた `&str` 型の文字列は `String` 型に変換されます。この型はサイズ可変の UTF-8 文字列で、文字列の追加や削除などが可能な形式です。文字列自体はヒープ領域に保持されており、ヒープ領域上の実データを持っています。実データを持っているので、その文字列に対する変更などが行えるわけです。

このハンズオンでは、最初の方は混乱を避けるためにすべて `String` で統一して扱います。こうすることで、不要な借用が発生せず、高級言語出身者でも Rust の概観をつかみやすくなると筆者が考えるためです。

今回、`Order::new` の中身も `String` 型で定義していました。そのため、`to_string` という関数を毎回要求されるわけです。

これがだるいという場合には、実はちょっとしたワークアラウンドがあります。このハンズオンでは後半に紹介する予定ですが、`impl Trait` という機能を使って次のように書き換えると、`&str` でも `String` でも受け取り可能な引数をもつ関数を作成できます。

```rust
impl Order {
    fn new(id: i32, pic: impl Into<String>, date: impl Into<String>, accepted: bool) -> Order {
        ...
    }
}
```

:::

## 新しい構造体を構造体自身に追加してみる

ここで少しチャレンジングな内容に取り組んでみましょう。まず、注文した商品を保持できるように、`items` というフィールドを追加します。フィールド自体は、一旦 `struct Item` というものを用意しておきましょう。これはこの後変更します。

```rust
struct Order {
    id: i32,
    pic: String,
    date: String,
    items: Vec<Item>,
    accepted: bool
}

struct Item {
    id: i32,
    name: String,
}
```

これに伴って、コンストラクタ関数も修正を加えます。また、せっかくなので `Item` にも `new` を使ってコンストラクタを生やしておきましょう。

```rust
impl Order {
    fn new(id: i32, pic: String, date: String, accepted: bool) -> Order {
        // 構造体の定義上のフィールド名と、構造体を生成する際に渡す変数名とが
        // 完全に一致する場合は、`id: id` といった繰り返しの記法が不要になる。
        Order {
            id,
            pic,
            date,
            items: vec![], // Order を用意したい際には、一旦注文された商品は空っぽであるとする。
            accepted
        }
    }
}

impl Item {
    fn new(id: i32, name: String) -> Item {
        Item {
            id,
            name,
        }
    }
}
```

### 構造体自身にミュータブルな操作をする関数を追加してみる

さて、商品の注文が行われた際に、商品を注文内容に追加できる関数を作ってみましょう。下記のようにすると作成できます。

```rust
impl Order {
    // (quick_look 関数の下に書きましょう)

    fn add_item(mut self, item: Item) {
        self.items.push(item);
    }
}
```

ミュータブルな操作をする（要するに、`Order` 構造体自身に破壊的な変更を加える操作をする）関数は、`self` の前に `mut` をつけると実現できます。

最後は下記のようにしてこのメソッドを呼び出しできます。呼び出し後は、`order` 内の `items` に値が詰まっていることになります。

```rust
fn main() {
    let order = Order::new(
        1,
        "Yuki Toyoda".to_string(),
        "2021-03-03T09:00:00Z".to_string(),
        false,
    );
    order.add_item(Item::new(1, "ハイボール".to_string()));
}
```

:::message
Rust の `self` と `mut self` について、ざっくりしたイメージを持つなら、`self` は `self` 用の呼び出し可能なメソッド群があり、`mut self` は `mut self` 用の呼び出し可能なメソッド群があると捉えられるでしょう。

たとえば、ある構造体 `A` に対して、

- self 側
  - a
  - b
  - c
- mut self 側
  - d
  - e
  - f

という設定がされていた場合、この構造体をイミュータブルに宣言する（`let a = A {}`）と、使用可能になるメソッドは `a`, `b`, `c` のみとなります。

一方で、ミュータブルに宣言する（`let mut a = A {}`）と、使用可能になるメソッドは `d`, `e`, `f` のみとなります。

この規則は、構造体が保有する別のデータにも適用されます。いい例が先ほどの `items` に使われていた `Vec` でしょう。実は、`Vec::push` は定義を確認するとわかるのですが `&mut self` です。つまり、ミュータブルなメソッドということになります。これを呼び出し可能にするためには、`Order` を `mut` にする必要があるのでした。したがって、`add_item` メソッドは、`mut self` をもっていたわけです。自身に対するミュータブルな `self` をもつことで、`Vec` に対してもミュータブルな操作を可能になる、という具合です。

ちなみに、`mut self` は `self` を含みます。つまり、`mut self` としておくと、`self` も `mut self` も呼び出し可能になります。逆はできません。
:::

### 余談: 所有権との邂逅

お気づきかもしれませんが、意図的にそらしてきた話題があります。たとえば下記のようにもう 1 つ商品を追加したくなったとしましょう。

```rust
fn main() {
    let order = Order::new(
        1,
        "Yuki Toyoda".to_string(),
        "2021-03-03T09:00:00Z".to_string(),
        false,
    );
    order.add_item(Item::new(1, "ハイボール".to_string()));
    order.add_item(Item::new(2, "ジンジャーハイボール".to_string()));
}
```

できましたね。私たちがいつも使ってきたプログラミング言語であれば、これでなんの問題もなくコンパイルが通り、処理を実行できるはずです。が、Rust はそうはいきません。実際にコンパイルしてみましょう。

```
error[E0382]: use of moved value: `order`
  --> src/main.rs:48:5
   |
46 |     let mut order = Order::new(1, "Yuki Toyoda".to_string(), "2021-03-03T09:00:00Z".to_string(), false);
   |         --------- move occurs because `order` has type `Order`, which does not implement the `Copy` trait
47 |     order.add_item(Item::new(1, "ハイボール".to_string()));
   |     ----- value moved here
48 |     order.add_item(Item::new(2, "ハイボール".to_string()));
   |     ^^^^^ value used here after move
```

なぜでしょうか。ここには所有権という問題が絡んできます。先ほど定義したメソッドをもう一度眺めてみましょう。

```rust
fn add_item(mut self, item: Item) {
    self.items.push(item);
}
```

このメソッドの第一引数は `mut self` となっていました。これが意味するところは、「この関数がインスタンスの所有権を奪い使用する」という意味になります。要するに、このメソッドの処理が終了した瞬間に、このメソッドの構造体が確保していたメモリ領域は解放されてしまい、以降は使用できなくなるということを意味します。

これを回避するためにはどうしたらよいでしょうか？借用という考え方が Rust にはあり、それを利用することで回避できるようになります。つまり、先頭に `&` をつけることで実現できます。

```rust
fn add_item(&mut self, item: Item) {
    self.items.push(item);
}
```

もう一度実行してみると、今度はコンパイルが通ることを確認できるはずです。

```rust
fn main() {
    let order = Order::new(
        1,
        "Yuki Toyoda".to_string(),
        "2021-03-03T09:00:00Z".to_string(),
        false,
    );
    order.add_item(Item::new(1, "ハイボール".to_string()));
    order.add_item(Item::new(2, "ジンジャーハイボール".to_string()));
}
```

Rust で構造体に対してメソッドを生やして、構造体というまとまり単位で処理を進めていく場合にも、この所有権という仕組みが絡んできてしまいます。初心者のうちは結構つまづきやすいポイントです。先に説明してしまいましたが、所有権の章でもう一度説明するので、現時点では「こういうことが起こるんだ」という認識で大丈夫です。

## enum

商品を扱う際に、商品にはいくつか種類があるということに気づくはずです。この配送システムでは、大きくカテゴライズすると、「服」「食料品」「酒類」「本」を扱っているものとしましょう。`enum` は、こうした種類をもつ値に対して適用できるデータの概念です。

今回は `Item` を少し拡張して、`enum` を使って表現してみましょう。それをその後に、`Order` に対して組み込んでみましょう。

まずは、扱う商品の種類を定義しましょう。先ほどの `Item` 構造体は、一度削除してしまって構いません。

```rust
enum Item {
    Clothes,
    Foods,
    Booze,
    Books
}
```

他の言語ですと、`enum` は `int` のエイリアスでしかなく、そのような使い方しかできません。しかし、Rust の enum は非常に強力です。これは代数的データ型と呼ばれる概念に対応しており、後ほど紹介するようにパターンマッチングと組み合わせながらその力を発揮します。

各 `Item` にはそれぞれ特長があるはずです。id と名前は当然のこと、服なら何色かという情報をもつでしょうし、食料品なら産地の情報をもつかもしれません。お酒ならアルコール度数の情報を持つはずです。本であれば、ISBN コードを持つでしょう。

Rust の `enum` にはこうした値をもたせることができます。たとえば、`Books` に ID をもつように追加してみましょう。ここでは、id は `i32` を使用するものとします。すると、次のように記述できます。

```rust
enum Item {
    // ...
    Books(i32),
}
```

Rust の enum はこれだけではなく、内部に構造体をもたせることができます。これらを利用しながら、`Item` を実用的な形に仕上げていきましょう。下記のように書くことができます。

```rust
enum Item {
    Clothes {
        id: i32,
        name: String,
        colour: String,
    },
    Foods {
        id: i32,
        name: String,
        made_in: String,
    },
    Booze {
        id: i32,
        name: String,
        percentage: i32,
    },
    Books {
        id: i32,
        name: String,
        isbn: String,
    },
}
```

先ほどの `Item` の追加関数があったことを思い出してください。これを拡張して、バケットに入ったアイテムを一気に追加できる仕様にしてみましょう。

一旦下記のように書くと、要素を一気に追加することができます[^2]。

```rust
impl Order {
    fn new(id: i32, pic: String, date: String, accepted: bool) -> Order {
        Order {
            id,
            pic,
            date,
            items: vec![],
            accepted
        }
    }

    fn add_items(mut self, is: Vec<Item>) {
        for item in is.into_iter() {
            self.items.push(item);
        }
    }
}
```

これを用いて、商品を登録してみましょう。

```rust
fn main() {
    let mut order = Order::new(
        1,
        "Yuki Toyoda".to_string(),
        "2021-03-03T09:00:00Z".to_string(),
        false,
    );
    let t_shirt = Item::Clothes {
        id: 1,
        name: "小さいTシャツ".to_string(),
        colour: "赤色".to_string(),
    };
    let highball = Item::Booze {
        id: 2,
        name: "サントリーハイボール".to_string(),
        percentage: 7,
    };
    order.add_items(vec![t_shirt, highball]);
}
```

これで、「小さい T シャツ」と「サントリーハイボール」を登録できました。

:::message
注目すべき点は、他の言語で言うところの `interface` などの仕組みをとくに利用していないにも関わらず、`Vec<Item>` というひとつのシグネチャで済んでいるという点です。これは一種の多相性と呼べるもので、Rust で他言語でいうところのポリモフィズムを実現したい際には、enum が利用できるケースでないかを検討するとよいかもしれません。Rust には他にもこうした多相性の仕組みがあり、それらを使い分ける必要があります。他の言語とは少しデータの組み立て方が異なるため、まず最初に苦戦するポイントかもしれません。ここは慣れが必要です。
:::

次に、enum に実装を追加してみましょう。たとえば、各々の enum の内容を表示できたら便利ですよね。ここまで意図的ではありますが、`add_item(s)` したあとの内容を簡単には確認できずにやってきました。それを確認できるようにしていきます。

enum への実装の追加は、やはり構造体と同様に `impl` というキーワードを使って行うことができます。たとえば、赤い服を作るコンストラクタを用意したいとなったら、下記のように書くことができます。

```rust
impl Item {
    fn red_clothes(id: i32, name: String) -> Item {
        Item::Clothes {
            id,
            name,
            colour: "赤色".to_string()
        }
    }
}
```

まったく構造体と同じになるでしょう。

enum の内容を表示する便利メソッドを作りましょう。まず、シグネチャは下記のようになるでしょう。`self` というのは、`Item` 自身のことを指します。ここには、`Item` enum が入っているイメージです。

```rust
impl Item {
    fn show_detail(self) -> String {
        "何か情報を返します".to_string()
    }
}
```

ところで、`Item` というのは複数要素を持つグループでした。わたしたちは、そのグループひとつひとつに対して、詳細な情報を出力するメソッドを用意したいのでした。ひとつひとつの要素に対して個別に処理を書くためにはどうしたらよいのでしょうか？そこで登場するのがパターンマッチングです。

パターンマッチングを使って、まずは下記のように実装の骨子を用意してみましょう。`_` は、enum に保持させていたひとつひとつのフィールドを指すものの、それらを使用しないことを意味しています。

```rust
impl Item {
    fn show_detail(self) -> String {
        match self {
            Item::Clothes { _, _, _ } => "服！".to_string(),
            Item::Foods { _, _, _ } => "食料品！".to_string(),
            Item::Booze { _, _, _ } => "お酒！".to_string(),
            Item::Books { _, _, _ } => "書籍！".to_string()
        }
    }
}
```

enum の値は取り出すことができます。最終的には下記のように、1 つ 1 つの値を取り出して詳細情報を返すことができるようになるでしょう。

```rust
impl Item {
    fn show_detail(self) -> String {
        match self {
            Item::Clothes { id, name, colour } => {
                format!("服 (id={}, 名前={}, 色={})", id, name, colour)
            }
            Item::Foods { id, name, made_in } => {
                format!("食料品 (id={}, 名前={}, 産地={})", id, name, made_in)
            }
            Item::Booze {
                id,
                name,
                percentage,
            } => format!(
                "お酒 (id={}, 名前={}, アルコール度数={})",
                id, name, percentage
            ),
            Item::Books { id, name, isbn } => {
                format!("書籍 (id={}, 名前={}, ISBN={})", id, name, isbn)
            }
        }
    }
}
```

さて、ここからすべてをつなげていきましょう。最終的に保持する商品内容を表示できるように、Order の実装を修正していきます。まず現時点での実装は、下記のようになっているはずです。

```rust
struct Order {
    id: i32,
    pic: String,
    date: String,
    items: Vec<Item>,
    accepted: bool,
}

enum Item {
    Clothes {
        id: i32,
        name: String,
        colour: String,
    },
    Foods {
        id: i32,
        name: String,
        made_in: String,
    },
    Booze {
        id: i32,
        name: String,
        percentage: i32,
    },
    Books {
        id: i32,
        name: String,
        isbn: String,
    },
}

impl Item {
    // red_clothes は今回は使用しないので、削除しています。

    fn show_detail(self) -> String {
        match self {
            Item::Clothes { id, name, colour } => {
                format!("服 (id={}, 名前={}, 色={})", id, name, colour)
            }
            Item::Foods { id, name, made_in } => {
                format!("食料品 (id={}, 名前={}, 産地={})", id, name, made_in)
            }
            Item::Booze {
                id,
                name,
                percentage,
            } => format!(
                "お酒 (id={}, 名前={}, アルコール度数={})",
                id, name, percentage
            ),
            Item::Books { id, name, isbn } => {
                format!("書籍 (id={}, 名前={}, ISBN={})", id, name, isbn)
            }
        }
    }
}

impl Order {
    fn new(id: i32, pic: String, date: String, accepted: bool) -> Order {
        Order {
            id,
            pic,
            date,
            items: vec![],
            accepted,
        }
    }

    // quick_look は今回は使用しないので、削除しています。

    // 商品を追加するメソッド
    fn add_items(mut self, is: Vec<Item>) {
        for item in is.into_iter() {
            self.items.push(item);
        }
    }
}

fn main() {
    let mut order = Order::new(
        1,
        "Yuki Toyoda".to_string(),
        "2021-03-03T09:00:00Z".to_string(),
        false,
    );
    let t_shirt = Item::Clothes {
        id: 1,
        name: "小さいTシャツ".to_string(),
        colour: "赤色".to_string(),
    };
    let highball = Item::Booze {
        id: 2,
        name: "サントリーハイボール".to_string(),
        percentage: 7,
    };
    order.add_items(vec![t_shirt, highball]);
}
```

まず、`add_items` したあとの挙動を修正します。このままだと Rust 特有の所有権関係の話に悩まされることになるので、悩まされないように構造体をコピーする処理を追加します[^1]。

```rust
impl Order {
    // add_items を書き換えましょう
    fn add_items(mut self, is: Vec<Item>) -> Order {
        for item in is.into_iter() {
            self.items.push(item);
        }

        Order {
            id: self.id,
            pic: self.pic,
            date: self.date,
            items: self.items,
            accepted: self.accepted
        }
    }
}
```

こうすると、新しく `Order` 構造体を作り直して返すようになります。つまり、`add_items` したあとは、新たな `Order` 構造体を使用するようにしたい、ということになります。なので、 main 関数内の処理も下記のように書き換えていきましょう。

```rust
fn main() {
    let mut order = Order::new(
        1,
        "Yuki Toyoda".to_string(),
        "2021-03-03T09:00:00Z".to_string(),
        false,
    );
    let t_shirt = Item::Clothes {
        id: 1,
        name: "小さいTシャツ".to_string(),
        colour: "赤色".to_string(),
    };
    let highball = Item::Booze {
        id: 2,
        name: "サントリーハイボール".to_string(),
        percentage: 7,
    };
    // new_order という一時変数を新たに追加し、そちらに add_items 後の Order を詰め直す。
    let new_order = order.add_items(vec![t_shirt, highball]);
}
```

まず、これで `add_items` を呼び出しああとも、引き続き `Order` に関するメソッドを呼び出せる準備が整いました。

次に、`show_items_detail` という関数を追加しましょう。下記のようになるはずです。

```rust
impl Order {
    fn show_items_detail(self) -> Vec<String> {
        let mut results = Vec::new();
        for item in self.items.into_iter() {
            results.push(item.show_detail());
        }
        results
    }
}
```

さあ準備が整いました。最後は、上のコードで生成した結果を一時変数に詰め直し、それを標準出力する関数を main に書き足すだけです。下記のように書けるはずです。

```rust
fn main() {
    let mut order = Order::new(
        1,
        "Yuki Toyoda".to_string(),
        "2021-03-03T09:00:00Z".to_string(),
        false,
    );
    let t_shirt = Item::Clothes {
        id: 1,
        name: "小さいTシャツ".to_string(),
        colour: "赤色".to_string(),
    };
    let highball = Item::Booze {
        id: 2,
        name: "サントリーハイボール".to_string(),
        percentage: 7,
    };
    let new_order = order.add_items(vec![t_shirt, highball]);

    // ここから書き足した
    // new_order の show_items_detail メソッドを呼ぶ。
    let items_detail = new_order.show_items_detail();

    // 商品詳細の記述が入ったベクタをループして、結果を出力する
    for detail in items_detail.into_iter() {
        println!("{}", detail);
    }
}
```

標準出力は下記のようになりました！

```
服 (id=1, 名前=小さいTシャツ, 色=赤色)
お酒 (id=2, 名前=サントリーハイボール, アルコール度数=7)
```

最終的な成果物は下記のとおりとなりました。

:::details 最終的なソースコード

```rust
struct Order {
    id: i32,
    pic: String,
    date: String,
    items: Vec<Item>,
    accepted: bool,
}

enum Item {
    Clothes {
        id: i32,
        name: String,
        colour: String,
    },
    Foods {
        id: i32,
        name: String,
        made_in: String,
    },
    Booze {
        id: i32,
        name: String,
        percentage: i32,
    },
    Books {
        id: i32,
        name: String,
        isbn: String,
    },
}

impl Item {
    fn show_detail(self) -> String {
        match self {
            Item::Clothes { id, name, colour } => {
                format!("服 (id={}, 名前={}, 色={})", id, name, colour)
            }
            Item::Foods { id, name, made_in } => {
                format!("食料品 (id={}, 名前={}, 産地={})", id, name, made_in)
            }
            Item::Booze {
                id,
                name,
                percentage,
            } => format!(
                "お酒 (id={}, 名前={}, アルコール度数={})",
                id, name, percentage
            ),
            Item::Books { id, name, isbn } => {
                format!("書籍 (id={}, 名前={}, ISBN={})", id, name, isbn)
            }
        }
    }
}

impl Order {
    fn new(id: i32, pic: String, date: String, accepted: bool) -> Order {
        Order {
            id,
            pic,
            date,
            items: vec![],
            accepted,
        }
    }

    fn add_items(mut self, is: Vec<Item>) -> Order {
        for item in is.into_iter() {
            self.items.push(item);
        }

        Order {
            id: self.id,
            pic: self.pic,
            date: self.date,
            items: self.items,
            accepted: self.accepted,
        }
    }

    fn show_items_detail(self) -> Vec<String> {
        let mut results = Vec::new();
        for item in self.items.into_iter() {
            results.push(item.show_detail());
        }
        results
    }
}

fn main() {
    let mut order = Order::new(
        1,
        "Yuki Toyoda".to_string(),
        "2021-03-03T09:00:00Z".to_string(),
        false,
    );
    let t_shirt = Item::Clothes {
        id: 1,
        name: "小さいTシャツ".to_string(),
        colour: "赤色".to_string(),
    };
    let highball = Item::Booze {
        id: 2,
        name: "サントリーハイボール".to_string(),
        percentage: 7,
    };

    let new_order = order.add_items(vec![t_shirt, highball]);
    let items_detail = new_order.show_items_detail();

    for detail in items_detail.into_iter() {
        println!("{}", detail);
    }
}

```

:::

## まとめ

- `struct` や `enum` を使用すると、独自のデータ型を定義できる。
- `struct` は、単純にデータを作るのに使える。
- `enum` は、いくつか種類のあるデータを 1 つのまとまりのなかにまとめることができる。
- それぞれには `impl` キーワードを使って実装を生やすことができる。

## 演習

### 色に関する追加

- たとえば「色」は、新しい `Colours` という enum を定義できるはずです。`CoralPink`（コーラルピンク）, `Sand`（サンド）, `Sacks`（サックス）という 3 種類の色を追加しましょう。
- `Clothes` にある `colour` は現在文字列型ですが、それを `Colours` enum を使用するように置き換えてみましょう。
- それぞれの色の名前を出力できるようにし、最終的に `items_detail` で色情報も日本語（カタカナ）で出力できるようにしましょう。

:::details 答え

- enum を追加した。
- 追加した enum に対して、`colour_name` というメソッドを生やした。中でカタカナ名を呼び出しした。
- `show_detail` 内で `colour_name` を呼び出すようにした。
- main 関数内で、文字列で色を指定していた箇所を `Colours::CoralPink` を使用するように修正した。

```rust
struct Order {
    id: i32,
    pic: String,
    date: String,
    items: Vec<Item>,
    accepted: bool,
}

enum Item {
    Clothes {
        id: i32,
        name: String,
        colour: Colours,
    },
    Foods {
        id: i32,
        name: String,
        made_in: String,
    },
    Booze {
        id: i32,
        name: String,
        percentage: i32,
    },
    Books {
        id: i32,
        name: String,
        isbn: String,
    },
}

enum Colours {
    CoralPink,
    Sand,
    Sacks
}

impl Colours {
    fn colour_name(self) -> String {
        match self {
            CoralPink => "コーラルピンク".to_string(),
            Sand => "サンド".to_string(),
            Sacks => "サックス".to_string(),
        }
    }
}

impl Item {
    fn show_detail(self) -> String {
        match self {
            Item::Clothes { id, name, colour } => {
                format!("服 (id={}, 名前={}, 色={})", id, name, colour.colour_name())
            }
            Item::Foods { id, name, made_in } => {
                format!("食料品 (id={}, 名前={}, 産地={})", id, name, made_in)
            }
            Item::Booze {
                id,
                name,
                percentage,
            } => format!(
                "お酒 (id={}, 名前={}, アルコール度数={})",
                id, name, percentage
            ),
            Item::Books { id, name, isbn } => {
                format!("書籍 (id={}, 名前={}, ISBN={})", id, name, isbn)
            }
        }
    }
}

impl Order {
    fn new(id: i32, pic: String, date: String, accepted: bool) -> Order {
        Order {
            id,
            pic,
            date,
            items: vec![],
            accepted,
        }
    }

    fn add_items(mut self, is: Vec<Item>) -> Order {
        for item in is.into_iter() {
            self.items.push(item);
        }

        Order {
            id: self.id,
            pic: self.pic,
            date: self.date,
            items: self.items,
            accepted: self.accepted
        }
    }

    fn show_items_detail(self) -> Vec<String> {
        let mut results = Vec::new();
        for item in self.items.into_iter() {
            results.push(item.show_detail());
        }
        results
    }
}

fn main() {
    let mut order = Order::new(
        1,
        "Yuki Toyoda".to_string(),
        "2021-03-03T09:00:00Z".to_string(),
        false,
    );
    let t_shirt = Item::Clothes {
        id: 1,
        name: "小さいTシャツ".to_string(),
        colour: Colours::CoralPink,
    };
    let highball = Item::Booze {
        id: 2,
        name: "サントリーハイボール".to_string(),
        percentage: 7,
    };

    let new_order = order.add_items(vec![t_shirt, highball]);
    let items_detail = new_order.show_items_detail();

    for detail in items_detail.into_iter() {
        println!("{}", detail);
    }
}

```

:::

### その他も enum にできないだろうか？

- 産地が仮に国別コードだとすると、`Country` という enum が考えられますね。
- ISBN は 10 桁のものと 13 桁のものが存在します。`Isbn` という enum に、`Isbn::Ten` と `Isbn::Thirteen` が作れるかもしれません。また、これらの enum に Isbn コードそのものの情報をもたせられるはずです。
- これらを同様に標準出力で表示させてみるのもおもしろいかもしれません。
- enum のサンプルは下記に載せておきます。

```rust
enum Countries {
    Usa,
    Uk,
    Canada,
    Japan,
    China,
    SouthKorea,
}
```

```rust
enum Isbn {
    Ten(String),
    Thirteen(String)
}
```

[^1]: みなさんが所有権をマスターした暁には、このメソッドには `&mut self` を使用することで、上述のような所有権問題を回避できるようになります。が、今はまだ所有権を意識しないプログラミングをしてもらいたいので、わざとこのような実装を行っています。
[^2]: イテレータをひとつひとつ回しながら `push` する部分の処理は、ベクタ同士の結合を行う処理です。実はこのケースは `Vec::append` を使用すれば[1 行で終了です](https://doc.rust-lang.org/std/vec/struct.Vec.html#method.append)。が、まだ説明していない `&mut` を実引数につける必要が出てくるため、今回は採用しませんでした。一般には `append` を素直に使用することが多いはずです。
