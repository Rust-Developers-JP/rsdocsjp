# モジュール `std::boxed`

## バージョン

Rust 1.0.0 ~

## 概要

`Box<T>`はヒープアロケーションのための型である。

[`Box<T>`](https://doc.rust-lang.org/std/boxed/struct.Box.html)(単に ‘Box’ と呼ばれることもある)はRustにおいて一番単純なヒープアロケーションである。Boxはこのアロケーションの所有権を有し、スコープから外れる時に中身を破棄する。加えて、ヒープの割り当てが`isize::MAX`バイトを超えないことも保証する。

## 例

[`Box`](https://doc.rust-lang.org/std/boxed/struct.Box.html)を作成してスタックの値をヒープへ移動する：

```rust
let val: u8 = 5;
let boxed: Box<u8> = Box::new(val);
```

[参照外し](https://doc.rust-lang.org/std/ops/trait.Deref.html)によって[`Box`](https://doc.rust-lang.org/std/boxed/struct.Box.html)からスタックへ値を移動する：

```rust
let boxed: Box<u8> = Box::new(5);
let val: u8 = *boxed;
```

再帰データ構造の作成：

```rust
#[derive(Debug)]
enum List<T> {
    Cons(T, Box<List<T>>),
    Nil,
}

let list: List<i32> = List::Cons(1, Box::new(List::Cons(2, Box::new(List::Nil))));
println!("{list:?}");
```

上記のプログラムの結果は`Cons(1, Cons(2, Nil))`となる。
再帰データ構造にはBox化が必要である。例えば`Cons`を下記のように定義した場合コンパイルエラーとなる：

```rust
Cons(T, List<T>),
```

なぜなら`List`のサイズはリスト内にどれだけのリストがあるかにより、このままでは`Cons`に対するスタックの割り当て量を把握できないためである。サイズが決まっている[`Box<T>`](https://doc.rust-lang.org/std/boxed/struct.Box.html)の導入により、`Cons`に必要なスタックサイズが把握できるようになる。

## メモリーレイアウト

サイズが0でない値では、[`Box`](https://doc.rust-lang.org/std/boxed/struct.Box.html)はメモリー割り当てのため[`Global`](https://doc.rust-lang.org/std/alloc/struct.Global.html)アロケーターを使用する。アロケーターの引数として使用される[`Layout`](https://doc.rust-lang.org/std/alloc/struct.Layout.html)が適切な型であり、生ポインターがその型の有効な値を示す場合に、[`Box`](https://doc.rust-lang.org/std/boxed/struct.Box.html)と[`Global`](https://doc.rust-lang.org/std/alloc/struct.Global.html)アロケーターに割り当てられた生ポインターは相互に変換できる。より詳しく言うと、[`Layout::for_value(&*value)`](https://doc.rust-lang.org/std/alloc/struct.Layout.html#method.for_value)とともに使用される[`Global`](https://doc.rust-lang.org/std/alloc/struct.Global.html)アロケーターに割り当てられた`value: *mut T`は[`Box::<T>::from_raw(value)`](https://doc.rust-lang.org/std/boxed/struct.Box.html#method.from_raw)を使用してBoxへと変換できる。逆に[`Box::<T>::into_raw`](https://doc.rust-lang.org/std/boxed/struct.Box.html#method.into_raw)から得られた`value: *mut T`は[`Layout::for_value(&*value)`](https://doc.rust-lang.org/std/alloc/struct.Layout.html#method.for_value)と[`Global`](https://doc.rust-lang.org/std/alloc/struct.Global.html)アロケーターによって解放できる。

サイズが0の値では、`Box`は非ヌルでありアライメントされている必要がある。`Box::new`が使用できない場合にZST(ゼロサイズ型)のBoxを作成する推奨方法は[`ptr::NonNull::dangling`](https://doc.rust-lang.org/std/ptr/struct.NonNull.html#method.dangling)を使用することである。

これらの基本的なLayoutの要件に加えて、`Box<T>`は有効な値`T`を示さなければならない。

`T: Sized`である限り`Box<T>`は単一のポインターであることを保証するとともにCのポインター(CのT*型)とのABI互換性を持つ。これはRustでCから呼び出す関数を作成する際に`Box<T>`で定義したものがCでは`T*`になるという意味である。例として`Foo`の作成と破棄を行う関数を宣言するCのヘッダーを示す：

```C
/* C ヘッダー */

/* 呼び出し元への所有権の返却 */
struct Foo* foo_new(void);

/* 呼び出し元から所有権を取得; ヌルの場合何もしない */
void foo_delete(struct Foo*);
```

この2つの関数はRustで下記のように実装できる。ここでは`struct Foo*`型に変換された`Box<Foo>`は所有権の制約を補足する。`Box<Foo>`が非ヌルであるため、引数がヌルになる可能性のある`foo_delete`の引数はRustでは`Option<Box<Foo>>`になることに注意しなければならない。

```rust
#[repr(C)]
pub struct Foo;

#[unsafe(no_mangle)]
pub extern "C" fn foo_new() -> Box<Foo> {
    Box::new(Foo)
}

#[unsafe(no_mangle)]
pub extern "C" fn foo_delete(_: Option<Box<Foo>>) {}
```

`Box<T>`はCのABIにおいてCのポインターと同じ表現を持つが、これはいつでも`T*`を`Box<T>`へ変換したり変換したものが上手く機能するという意味ではない。`Box<T>`の値は常にメモリーアライメントされた非ヌルのポインターであり、デストラクターはグローバルアロケーターでの値の開放を試みる。一般的なベストプラクティスは`Box<T>`をグローバルアロケーターから生成されるポインターとしてのみ使用することである。

**重要** 少なくとも現時点ではRustから呼び出されるCで定義された関数の型に`Box<T>`を使用すべきではない。この場合ではCの型に可能な限り近い型を直接使用すべきである。[`rust-lang/unsafe-code-guidelines#198`](https://github.com/rust-lang/unsafe-code-guidelines/issues/198)に記載されているように、`Box<T>`をCの定義で単に`T*`のように使用すると未定義動作につながることがある。

## アンセーフなコードへに対する考慮

**注意：このセクションは規範的でなく変更の可能性を含んでおり、将来的に緩和される可能性がある。これは現在コンパイラーに実装されていルールの要約である。**

`Box<T>`へのエイリアスのルールは`&mut T`と同じものである。`Box<T>`は保有する中身の一意性を持つ。`&mut T`のような移動や借用によって変更されたBoxから取得した生ポインターの使用は許可されていない。[`rust-lang/unsafe-code-guidelines#326`](https://github.com/rust-lang/unsafe-code-guidelines/issues/326)にアンセーフなコードでのBoxの使用に関する更なる説明がある。

## エディション

[このドキュメント](https://doc.rust-lang.org/std/primitive.array.html)にある通り、Rust 2021までのエディションの配列には`IntoIterator`の実装の特別なケースが存在する。後にBox化したスライスにも同様の回避策を追加する必要が判明したため、残念ながら回避策の適用は2024エディションになった。

具体的には、`IntoIterator`はすべてのエディションの`Box<[T]>`に実装されているが、Box化したスライスに対する`into_iter()`の適用は2024以前のエディションの実装に従う。

```rust
// Rust 2015, 2018, 2021:

let boxed_slice: Box<[i32]> = vec![0; 3].into_boxed_slice();

// スライスから各々の値を参照した iterator を作成する。
for item in boxed_slice.into_iter().enumerate() {
    let (i, x): (usize, &i32) = item;
    println!("boxed_slice[{i}] = {x}");
}

//`boxed_slice_into_iter`は将来の互換性のため、リンターに下記への変更を提案される:
for item in boxed_slice.iter().enumerate() {
    let (i, x): (usize, &i32) = item;
    println!("boxed_slice[{i}] = {x}");
}

// Box 化したスライスから値の iterator を作成する場合、`IntoIterator::into_iter`を使用して明示的に宣言できる。
for item in IntoIterator::into_iter(boxed_slice).enumerate() {
    let (i, x): (usize, i32) = item;
    println!("boxed_slice[{i}] = {x}");
}
```

配列の実装と同様に、このオーバーライドは将来修正される可能性があり、後方互換性を維持したい場合はこれらのエディション間の違いに頼った挙動を避けるべきである。

## 構造体

| 名前                                                                              | 説明                                     |
| --------------------------------------------------------------------------------- | ---------------------------------------- |
| [Box](https://doc.rust-lang.org/std/boxed/struct.Box.html)                        | ヒープに配置された T 型を一意に所有するポインター型 |
| [ThinBox](https://doc.rust-lang.org/std/boxed/struct.ThinBox.html) (Experimental) | 薄い Box                                 |
