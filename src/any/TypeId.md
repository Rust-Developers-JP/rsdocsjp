# 構造体 `std::any::TypeId`

```rust,ignore
pub struct TypeId {
    // private fields
}
```
## バージョン

Rust 1.0.0 ~

## 解説

型に対するグローバルに一意な識別子を提供する構造体。
不透明なオブジェクトであるため内部を直接見ることはできないが、DebugやCloneといった基本的なトレイトは実装してある。

現在、`TypeId`は`'static`制約を満たすもののみ利用できるが、これは将来解除される可能性がある。

また、`Hash`や`Ord`、`PartialOrd`を実装しているが、ハッシュ値や順序はRustのリリースごとに違うため、これらに依存した設計は避けるべきである。

## 実装

```rust,ignore
impl TypeId {
    pub const fn of<T>() -> TypeId 
    where 
        T: 'static + ?Sized;
}
```

### `of`

ジェネリクスに渡された型のTypeIdを返す関数。

```rust
use std::any::TypeId;

fn is_i32<T: 'static + ?Sized>(val: T) -> bool {
    TypeId::of::<T>() == TypeId::of::<i32>()
}

assert!(!is_i32(1.2));
```

## トレイト実装

### `Clone`

```rust,ignore
impl Clone for TypeId {
    /* trait methods */
}
```

### `std::fmt::Debug`

```rust,ignore
impl std::fmt::Debug for TypeId {
    /* trait methods */
}
```

### `std::hash::Hash`

```rust,ignore
impl std::hash::Hash for TypeId {
    /* trait methods */
}
```

### `Ord`

```rust,ignore
impl Ord for TypeId {
    /* trait methods */
}
```

### `PartialEq`

```rust,ignore
impl PartialEq for TypeId {
    /* trait methods */
}
```

### `PartialOrd`

```rust,ignore
impl PartialOrd for TypeId {
    /* trait methods */
}
```

### `Copy`

```rust,ignore
impl Copy for TypeId {}
```

### `Eq`

```rust,ignore
impl Eq for TypeId {}
```

### `Send`

```rust,ignore
impl Send for TypeId {}
```

### `Sync`

```rust,ignore
impl Sync for TypeId {}
```

### `std::marker::Freeze`

```rust,ignore
impl std::marker::Freeze for TypeId {}
```

### `std::panic::RefUnwindSafe`

```rust,ignore
impl std::panic::RefUnwindSafe for TypeId {}
```

### `Unpin`

```rust,ignore
impl Unpin for TypeId {}
```

### `std::panic::UnwindSafe`

```rust,ignore
impl std::panic::UnwindSafe for TypeId {}
```

---

### `std::any::Any`

```rust,ignore
impl<T> std::any::Any for T
where
    T: 'static + ?Sized,
{
    /* trait methods */
}
```

### `std::borrow::Borrow`

```rust,ignore
impl<T> std::borrow::Borrow<T> for T
where
    T: ?Sized,
{
    /* trait methods */
}
```

### `std::borrow::BorrowMut`

```rust,ignore
impl<T> std::borrow::BorrowMut<T> for T
where
    T: ?Sized,
{
    /* trait methods */
}
```

### `std::clone::CloneToUninit`

```rust,ignore
impl<T> std::clone::CloneToUninit for T
where
    T: Clone,
{
    /* trait methods */
}
```

### `From`

```rust,ignore
impl<T> From<T> for T {
    /* trait methods */
}
```

### `Into`

```rust,ignore
impl<T, U> Into<U> for T
where
    U: From<T>,
{
    /* trait methods */
}
```

### `std::borrow::ToOwned`

```rust,ignore
impl<T> std::borrow::ToOwned for T
where
    T: Clone,
{
    /* trait methods */
}
```

### `TryFrom`

```rust,ignore
impl<T, U> TryFrom<U> for T
where
    U: Into<T>,
{
    /* trait methods */
}
```

### `TryInto`

```rust,ignore
impl<T, U> TryInto<U> for T
where
    U: TryFrom<T>,
{
    /* trait methods */
}
```

