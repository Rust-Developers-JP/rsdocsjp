# 構造体 `std::vec::Drain` <!-- ここはかならずフルパスで書くこと -->

<!-- 
内部的な実装を書く。内容は公式ドキュメントの写しでよい。 
ただし、rustdocsjpのすべてのコードブロックにおいて、使用する構造体やトレイトはstd::prelude::rust_2024に列挙されているもの以外、必ずフルパスで書く。
-->
```rust,ignore
pub struct Drain<'a, T, A = std::alloc::Global>
where
    T: 'a,
    A: std::alloc::Allocator + 'a
{ /* private fields */ }
```

<!--
以降、
解説、コード例、実装、トレイト実装、関連項目の順に記載
-->
## 解説 

> [!NOTE]
> ジェネリクス`A`を明示的に使えるのはnightlyのみ

<!-- 詳細な解説を行う。公式ドキュメントおよびソースコードをよく読んでから書くこと -->
ベクター内の指定された区間を削除し、取り除かれた要素を与えるイテレータ。
`Vec`の`drain`メソッドによって作成される。

## 例

<!-- 
コード例を書く。
公式サイトのものをそのまま拝借しても構わないが、わかりやすいように必ず日本語のコメント文を挿入すること 
-->
```rust
let mut v = vec![0, 1, 2, 3];
let u = v.drain(1..).collect::<Vec<_>>();

// 前から２番目以降の要素が削除されている
assert_eq!(v, &[0]); 

// 削除された要素が返されている
assert_eq!(u, &[1, 2, 3]); 
```

## 実装

<!-- 
実装(implementation)を記入する。トレイトを使用した実装とは分離する。
また、複数の実装がある場合はコードブロックを分離し、できるだけ一つの実装につき一つのコードブロックにする
-->

```rust,ignore
impl<T, A> Drain<'_, T, A>
where
    A: std::aloc::Allocator
{ /* private fields */ }
```

<!-- nightlyでのみ使用できる場合はnightly-onlyというコメント文を挿入すること -->
```rust,ignore
impl<'a, T, A> Drain<'a, T, A>
where
    A: std::alloc::Allocator
{
    pub fn as_slice(&self) -> &[T];

    // nightly-only
    pub fn allocator(&self) -> &A;

    // nightly-only
    pub fn keep_rest(self);
}
```

<!--
実装されている関数の説明。
解説とコード例を必ず載せること。
-->

### `as_slice`

まだ削除されていない要素をスライスで返す関数。

```rust
let mut v = vec![0, 1, 2, 3];
let mut drain = v.drain(..);

// まだ何も消費していない
assert_eq!(drain.as_slice(), &[0, 1, 2, 3]); 

// １個前に進める（１個消費する）
let _ = drain.next().unwrap(); 

// １番目の要素が削除される
assert_eq!(drain.as_slice(), &[1, 2, 3]); 
```

### `allocator`

<!--
nightly-onlyの関数の場合は、解説内に

> [!NOTE]
> nightlyでのみ使用可能

が必須である。
-->

> [!NOTE]
> nightlyでのみ使用可能

内部で使用しているアロケーターを返す関数。

```rust
#![feature(allocator_api)]

use std::any::{Any, TypeId};

let v = vec![0, 1, 2, 3];
assert_eq!(
    // allocatorは参照を返すためcloneする
    v.allocator().clone().type_id(), 

    // Vecのアロケーターはデフォルトでstd::alloc::Global
    TypeId::of::<std::alloc::Global>() 
);
```

### `keep_rest`

> [!NOTE]
> nightlyでのみ使用可能

一度も消費していない要素をもとのベクターに返還する関数。

```rust
#![feature(drain_keep_rest)]

let mut v = vec![0, 1, 2, 3];
let mut drain = v.drain(..);

// ひとつ前に進める（一つ消費する）
drain.next().unwrap();

// vに消費していない要素を返還する
drain.keep_rest();
assert_eq!(v, [1, 2, 3]);
```

## トレイト実装

<!--
トレイトを使用した実装について記述する。一つのトレイトにつき一つの項目にすること。
以下は例である。

## ??? (トレイトの名前、std::prelude::rust_2024に列挙されている場合はフルパスを省略する)

```rust,ignore
(実装を記述)
```
-->

### `AsRef`

```rust,ignore
impl<'a, T, A> AsRef<[T]> for Drain<'a, T, A>
where 
    A: std::alloc::Allocator
{ /* trait methods */ }
```

### `std::fmt::Debug`

```rust,ignore
impl<T, A> std::fmt::Debug for Drain<'_, T, A>
where
    T: std::fmt::Debug,
    A: std::alloc::Allocator,
{ /* trait methods */ }
```

### `DoubleEndedIterator`

```rust,ignore
impl<T, A> DoubleEndedIterator for Drain<'_, T, A>
where
    A: std::alloc::Allocator
{ /* trait methods */ }
```

### `Drop`

```rust,ignore
impl<T, A> Drop for Drain<'_, T, A>
where
    A: std::alloc::Allocator
{ /* trait methods */ }
```

### `ExactSizeIterator`

```rust,ignore
impl<T, A> ExactSizeIterator for Drain<'_, T, A>
where
    A: std::alloc::Allocator
{ /* trait methods */ }
```

### `Interator`

```rust,ignore
impl<T, A> Iterator for Drain<'_, T, A>
where
    A: std::alloc::Allocator
{ /* trait methods */ }
```

### `std::iter::FusedIterator`

```rust,ignore
impl<T, A> std::iter::FusedIterator for Drain<'_, T, A>
where
    A: std::alloc::Allocator
{ /* trait methods */ }
```

### `Send`

```rust,ignore
impl<T, A> Send for Drain<'_, T, A>
where
    T: Send,
    A: Send + std::alloc::Allocator
{ /* trait methods */ }
```

### `Sync`

```rust,ignore
impl<T, A> Sync for Drain<'_, T, A>
where
    T: Sync,
    A: Sync + std::alloc::Allocator
{ /* trait methods */ }
```

### `std::iter::TrustedLen`

```rust,ignore
impl<T, A> std::iter::TrustedLen for Drain<'_, T, A>
where
    A: std::alloc::Allocator
{ /* trait methods */ }
```

### `std::marker::Freeze`

```rust,ignore
impl<'a, T, A> std::marker::Freeze for Drain<'a, T, A> { 
    /* trait methods */ 
}
```

### `std::panic::RefUnwindSafe`

```rust,ignore
impl<'a, T, A> std::panic::RefUnwindSafe for Drain<'a, T, A>
where
    T: std::panic::RefUnwindSafe,
    A: std::panic::RefUnwindSafe
{ /* trait methods */ }
```

### `Unpin`

```rust,ignore
impl<'a, T, A> Unpin for Drain<'a, T, A> { 
    /* trait methods */ 
}
```

### `std::panic::UnwindSafe`

```rust,ignore
impl<'a, T, A> std::panic::UnwindSafe for Drain<'a, T, A>
where
    T: std::panic::RefUnwindSafe,
    A: std::panic::RefUnwindSafe
{ /* trait methods */ }
```

<!-- 関連項目については、必要だと思ったら記載すること -->