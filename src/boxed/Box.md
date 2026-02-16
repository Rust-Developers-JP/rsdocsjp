# 構造体 `std::boxed::Box`

```rust
pub struct Box<T, A = std::alloc::Global>(/* private fields */)
where
    A: std::alloc::Allocator,
    T: ?Sized;
```

## バージョン

Rust 1.0.0 ~

## 解説

`T`型のヒープアロケーションを一意に有するポインター型。  
詳細は[モジュールドキュメント](./about.md)を参照。

## 実装

```rust
impl<A> Box<dyn Any, A>
where
    A: std::alloc::Allocator,
```

### `downcast`

**Rust 1.0.0 ~**

```rust
pub fn downcast<T>(self) -> Result<Box<T, A>, Box<dyn Any, A>>
where
    T: Any,
```

Boxの中身をT型へ変換することを試みる関数。

```rust
use std::any::Any;

fn print_if_string(value: Box<dyn Any>) {
    if let Ok(string) = value.downcast::<String>() {
        println!("String ({}): {}", string.len(), string);
    }
}

let my_string = "Hello World".to_string();
print_if_string(Box::new(my_string)); // String (11): Hello World
print_if_string(Box::new(0i8)); // String に変換できないため何も表示されない
```

### `downcast_unchecked`

```rust
pub unsafe fn downcast_unchecked<T>(self) -> Box<T, A>
where
    T: Any,
```
> [!NOTE]
> nightlyでのみ使用可能

Boxの中身をT型へ変換する関数。
メモリー安全な代替は[`downcast`](#downcast)を参照

```rust
#![feature(downcast_unchecked)]

use std::any::Any;

let x: Box<dyn Any + Send> = Box::new(1_usize);

unsafe {
    assert_eq!(*x.downcast_unchecked::<usize>(), 1);
}
```

**安全性**

Boxの中身はT型でなければならない。誤った型でこの関数を呼び出すと未定義動作となる。

```rust
impl<A> Box<dyn Any + Send, A>
where
    A: Allocator,
```
### `downcast`

**Rust 1.0.0 ~**

```rust
pub fn downcast<T>(self) -> Result<Box<T, A>, Box<dyn Any + Send, A>>
where
    T: Any,
```

Boxの中身をT型へ変換することを試みる関数。

```rust
use std::any::Any;

fn print_if_string(value: Box<dyn Any + Send>) {
    if let Ok(string) = value.downcast::<String>() {
        println!("String ({}): {}", string.len(), string);
    }
}

let my_string = "Hello World".to_string();
print_if_string(Box::new(my_string));
print_if_string(Box::new(0i8));
```

### `downcast_unchecked`

```rust
pub unsafe fn downcast_unchecked<T>(self) -> Box<T, A>
where
    T: Any,
```
> [!NOTE]
> nightlyでのみ使用可能

Boxの中身をT型へ変換する関数。
メモリー安全な代替は[`downcast`](#downcast-1)を参照

```rust
#![feature(downcast_unchecked)]

use std::any::Any;

let x: Box<dyn Any + Send + Sync> = Box::new(1_usize);

unsafe {
    assert_eq!(*x.downcast_unchecked::<usize>(), 1);
}
```

**安全性**

Boxの中身はT型でなければならない。誤った型でこの関数を呼び出すと未定義動作となる。

```rust
impl<T> Box<T>
```

**Rust 1.0.0 ~**

### `new`

```rust
pub fn new(x: T) -> Box<T>
```
ヒープ領域にメモリーを割り当て`x`を格納する。
`T`がゼロサイズの場合、実際にはメモリーは割り当てられない。

```rust
let five = Box::new(5);
```

### `new_uninit`

**Rust 1.82.0 ~**

```rust
pub fn new_uninit() -> Box<MaybeUninit<T>>
```

中身を初期化せずに新しいboxを作成する。

```rust
let mut five = Box::<u32>::new_uninit();
// 遅延初期化:
five.write(5);
let five = unsafe { five.assume_init() };

assert_eq!(*five, 5)
```

### `new_zeroed`

**Rust 1.92.0 ~**

```rust
pub fn new_zeroed() -> Box<MaybeUninit<T>>
```

初期化せずに新しいboxを作成し、中身を0バイトで埋める。
このメソッドの正しい使用例と誤った使用例は[`MaybeUninit::zeroed`](https://doc.rust-lang.org/std/mem/union.MaybeUninit.html#method.zeroed)を参照。

```rust
let zero = Box::<u32>::new_zeroed();
let zero = unsafe { zero.assume_init() };

assert_eq!(*zero, 0)
```

### `pin`

**Rust 1.33.0 ~**

```rust
pub fn pin(x: T) -> Pin<Box<T>>
```

新しい`Pin<Box<T>>`を作成する。`T`が[`Unpin`](https://doc.rust-lang.org/std/marker/trait.Unpin.html)を実装していない場合、`x`はメモリー内に固定され動かせなくなる。
`Box`の作成と固定は2段階でも行え、`Box::pin(x)`は`Box::into_pin(Box::new(x))`と同じである。すでに`Box<T>`がある場合や、[`Box::new`](#new)とは違った方法で(固定した)`Box`を作成したい場合は[`into_pin`](https://doc.rust-lang.org/std/boxed/struct.Box.html#method.into_pin)の使用を検討するとよい。