# 構造体 `std::vec::IntoIter`

```rust,ignore
pub struct IntoIter<T, A = std::alloc::Global>
where
    A: std::alloc::Allocator
{ /* private fields */ }
```

## 解説

ベクターの所有権を放棄し、各要素を順に返すイテレータ。
`Vec`の`into_iter`メソッド（`IntoIterator`トレイトの実装）によって作成される。

## 例

```rust
let v = vec![0, 1, 2];
// vの所有権がiterに移動する
let iter: std::vec::IntoIter<i32> = v.into_iter();

// 所有権を持っているので、要素を直接消費できる
for x in iter {
    println!("{}", x);
}
```

## 実装

```rust,ignore
impl<T, A> IntoIter<T, A>
where
    A: std::alloc::Allocator
{
    pub fn as_slice(&self) -> &[T];

    pub fn as_mut_slice(&mut self) -> &mut [T];

    // nightly-only
    pub fn allocator(&self) -> &A;
}
```

### `as_slice`

まだ消費されていない要素をスライスとして返す関数。

```rust
let vec = vec!['a', 'b', 'c'];
let mut into_iter = vec.into_iter();

// 最初はすべての要素が見える
assert_eq!(into_iter.as_slice(), &['a', 'b', 'c']);

// 1つ消費する
let _ = into_iter.next().unwrap();

// 残りの要素が返される
assert_eq!(into_iter.as_slice(), &['b', 'c']);
```

### `as_mut_slice`

まだ消費されていない要素を可変スライスとして返す関数。

```rust
let vec = vec!['a', 'b', 'c'];
let mut into_iter = vec.into_iter();

// 消費前のスライスを書き換える
into_iter.as_mut_slice()[2] = 'z';

assert_eq!(into_iter.next().unwrap(), 'a');
assert_eq!(into_iter.next().unwrap(), 'b');
assert_eq!(into_iter.next().unwrap(), 'z'); // 書き換えた値が返る
```

### `allocator`

> [!NOTE]
> nightlyでのみ使用可能

内部で使用しているアロケーターへの参照を返す関数。

## トレイト実装

### `AsRef`

```rust,ignore
impl<T, A> AsRef<[T]> for IntoIter<T, A>
where
    A: std::alloc::Allocator
{ /* trait methods */ }
```

### `Clone`

```rust,ignore
impl<T, A> Clone for IntoIter<T, A>
where
    T: Clone,
    A: std::alloc::Allocator + Clone
{ /* trait methods */ }
```

### `std::fmt::Debug`

```rust,ignore
impl<T, A> std::fmt::Debug for IntoIter<T, A>
where
    T: std::fmt::Debug,
    A: std::alloc::Allocator
{ /* trait methods */ }
```

### `Default`

```rust,ignore
impl<T, A> Default for IntoIter<T, A>
where
    A: std::alloc::Allocator + Default
{ /* trait methods */ }
```

### `DoubleEndedIterator`

```rust,ignore
impl<T, A> DoubleEndedIterator for IntoIter<T, A>
where
    A: std::alloc::Allocator
{ /* trait methods */ }
```

### `Drop`

```rust,ignore
impl<T, A> Drop for IntoIter<T, A>
where
    A: std::alloc::Allocator
{ /* trait methods */ }
```

### `ExactSizeIterator`

```rust,ignore
impl<T, A> ExactSizeIterator for IntoIter<T, A>
where
    A: std::alloc::Allocator
{ /* trait methods */ }
```

### `Iterator`

```rust,ignore
impl<T, A> Iterator for IntoIter<T, A>
where
    A: std::alloc::Allocator
{ /* trait methods */ }
```

### `std::iter::FusedIterator`

```rust,ignore
impl<T, A> std::iter::FusedIterator for IntoIter<T, A>
where
    A: std::alloc::Allocator
{ /* trait methods */ }
```

### `Send`

```rust,ignore
impl<T, A> Send for IntoIter<T, A>
where
    T: Send,
    A: std::alloc::Allocator + Send
{ /* trait methods */ }
```

### `Sync`

```rust,ignore
impl<T, A> Sync for IntoIter<T, A>
where
    T: Sync,
    A: std::alloc::Allocator + Sync
{ /* trait methods */ }
```

### `std::iter::TrustedLen`

```rust,ignore
impl<T, A> std::iter::TrustedLen for IntoIter<T, A>
where
    A: std::alloc::Allocator
{ /* trait methods */ }
```
