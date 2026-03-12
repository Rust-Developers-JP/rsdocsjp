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

> [!NOTE]
> nightlyでのみ使用可能

```rust
pub unsafe fn downcast_unchecked<T>(self) -> Box<T, A>
where
    T: Any,
```

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

> [!NOTE]
> nightlyでのみ使用可能

```rust
pub unsafe fn downcast_unchecked<T>(self) -> Box<T, A>
where
    T: Any,
```

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
`T`がゼロサイズ型の場合、実際にはメモリーは割り当てられない。

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

中身を初期化せずに新しいboxを作成し、中身を0バイトで埋める。
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

新しい`Pin<Box<T>>`を作成する。`T`が[`Unpin`](https://doc.rust-lang.org/std/marker/trait.Unpin.html)を実装していない場合、`x`はメモリー上に固定され動かせなくなる。
`Box`の作成と固定は2段階でも行え、`Box::pin(x)`は`Box::into_pin(Box::new(x))`と同じである。すでに`Box<T>`がある場合や、[`Box::new`](#new)とは違った方法で(固定した)`Box`を作成したい場合は[`into_pin`](https://doc.rust-lang.org/std/boxed/struct.Box.html#method.into_pin)の使用を検討するとよい。

### `try_new`

> [!NOTE]
> nightlyでのみ使用可能

```rust
pub fn try_new(x: T) -> Result<Box<T>, AllocError>
```

ヒープにメモリーを割り当ててから`x`を格納し、割り当てに失敗した場合エラーを返す。
`T`がゼロサイズ型の場合、実際にはメモリーは割り当てられない。

```rust
#![feature(allocator_api)]

let five = Box::try_new(5)?;
```

### `try_new_uninit`

> [!NOTE]
> nightlyでのみ使用可能

```rust
pub fn try_new_uninit() -> Result<Box<MaybeUninit<T>>, AllocError>
```

中身を初期化せずに新しいboxを作成し、割り当てに失敗した場合エラーを返す。

```rust
#![feature(allocator_api)]

let mut five = Box::<u32>::try_new_uninit()?;
// Deferred initialization:
five.write(5);
let five = unsafe { five.assume_init() };

assert_eq!(*five, 5);
```

### `try_new_zeroed`

> [!NOTE]
> nightlyでのみ使用可能

```rust
pub fn try_new_zeroed() -> Result<Box<MaybeUninit<T>>, AllocError>
```

中身を初期化せずに新しいboxを作成し、ヒープ上のメモリーを0バイトで埋める。割り当てに失敗した場合エラーを返す。
このメソッドの正しい使用例と誤った使用例は[`MaybeUninit::zeroed`](https://doc.rust-lang.org/std/mem/union.MaybeUninit.html#method.zeroed)を参照。

```rust
#![feature(allocator_api)]

let zero = Box::<u32>::try_new_zeroed()?;
let zero = unsafe { zero.assume_init() };

assert_eq!(*zero, 0);
```

### `map`

> [!NOTE]
> nightlyでのみ使用可能

```rust
pub fn map<U>(this: Box<T>, f: impl FnOnce(T) -> U) -> Box<U>
```

boxの中身を与えられた関数より再配置する。可能なら割り当てを再利用する。
`f`はbox内の値に対して呼び出され、返り値もbox化される。
注意: これは関連関数である、つまり`b.map(f)`の代わりに`Box::map(b, f)`と呼び出さなければならない。これは中身の型のメソッドと衝突しないようにするためである。

```rust
#![feature(smart_pointer_try_map)]

let b = Box::new(7);
let new = Box::map(b, |i| i + 7);
assert_eq!(*new, 14);
```

### `try_map`

> [!NOTE]
> nightlyでのみ使用可能

```rust
pub fn try_map<R>(
    this: Box<T>,
    f: impl FnOnce(T) -> R,
) -> <<R as Try>::Residual as Residual<Box<<R as Try>::Output>>>::TryType
where
    R: Try,
    <R as Try>::Residual: Residual<Box<<R as Try>::Output>>,
```

boxの中身を与えられた関数より再配置することを試みる。可能なら割り当てを再利用する。
`f`はbox内の値に対して呼び出され、返り値もbox化される。
注意: これは関連関数である、つまり`b.map(f)`の代わりに`Box::map(b, f)`と呼び出さなければならない。これは中身の型のメソッドと衝突しないようにするためである。

```rust
#![feature(smart_pointer_try_map)]

let b = Box::new(7);
let new = Box::try_map(b, u32::try_from).unwrap();
assert_eq!(*new, 7);
```

```rust
impl<T, A> Box<T, A>
where
    A: Allocator,
```

### `new_in`

> [!NOTE]
> nightlyでのみ使用可能

```rust
pub fn new_in(x: T, alloc: A) -> Box<T, A>
where
    A: Allocator,
```

与えられたアロケーターからメモリーを割り当て`x`を格納する。
`T`がゼロサイズ型の場合、実際にはメモリーは割り当てられない。

```rust
#![feature(allocator_api)]

use std::alloc::System;

let five = Box::new_in(5, System);
```

### `try_new_in`

> [!NOTE]
> nightlyでのみ使用可能

```rust
pub fn try_new_in(x: T, alloc: A) -> Result<Box<T, A>, AllocError>
where
    A: Allocator,
```

与えられたアロケーターからメモリーを割り当て`x`を格納し、割り当てに失敗した場合エラーを返す。
`T`がゼロサイズ型の場合、実際にはメモリーは割り当てられない。

```rust
#![feature(allocator_api)]

use std::alloc::System;

let five = Box::try_new_in(5, System)?;
```

### `new_uninit_in`

> [!NOTE]
> nightlyでのみ使用可能

```rust
pub fn new_uninit_in(alloc: A) -> Box<MaybeUninit<T>, A>
where
    A: Allocator,
```

与えられたアロケーターから中身を初期化せずに新しいboxを作成する。

```rust
#![feature(allocator_api)]

use std::alloc::System;

let mut five = Box::<u32, _>::new_uninit_in(System);
// Deferred initialization:
five.write(5);
let five = unsafe { five.assume_init() };

assert_eq!(*five, 5)
```

### `try_new_uninit_in`

> [!NOTE]
> nightlyでのみ使用可能

```rust
pub fn try_new_uninit_in(alloc: A) -> Result<Box<MaybeUninit<T>, A>, AllocError>
where
    A: Allocator,
```

与えられたアロケーターから中身を初期化せずに新しいboxを作成し、割り当てに失敗した場合エラーを返す。

```rust
#![feature(allocator_api)]

use std::alloc::System;

let mut five = Box::<u32, _>::try_new_uninit_in(System)?;
// 遅延初期化:
five.write(5);
let five = unsafe { five.assume_init() };

assert_eq!(*five, 5);
```

### `new_zeroed_in`

> [!NOTE]
> nightlyでのみ使用可能

```rust
pub fn new_zeroed_in(alloc: A) -> Box<MaybeUninit<T>, A>
where
    A: Allocator,
```

与えられたアロケーターから中身を初期化せずに新しいboxを作成し、ヒープ上のメモリーを0バイトで埋める。割り当てに失敗した場合エラーを返す。
このメソッドの正しい使用例と誤った使用例は[`MaybeUninit::zeroed`](https://doc.rust-lang.org/std/mem/union.MaybeUninit.html#method.zeroed)を参照。

```rust
#![feature(allocator_api)]

use std::alloc::System;

let zero = Box::<u32, _>::try_new_zeroed_in(System)?;
let zero = unsafe { zero.assume_init() };

assert_eq!(*zero, 0);
```

### `pin_in`

> [!NOTE]
> nightlyでのみ使用可能

```rust
pub fn pin_in(x: T, alloc: A) -> Pin<Box<T, A>>
where
    A: 'static + Allocator,
```

新しい`Pin<Box<T, A>>`を作成する。`T`が[`Unpin`](https://doc.rust-lang.org/std/marker/trait.Unpin.html)を実装していない場合、`x`はメモリー上に固定され動かせなくなる。
`Box`の作成と固定は2段階でも行え、`Box::pin_in(x, alloc)`は`Box::into_pin(Box::new_in(x, alloc))`と同じである。すでに`Box<T, A>`がある場合や、[`Box::new_in`](#new_in)とは違った方法で(固定した)`Box`を作成したい場合は[`into_pin`](https://doc.rust-lang.org/std/boxed/struct.Box.html#method.into_pin)の使用を検討するとよい。

### `into_boxed_slice`

> [!NOTE]
> nightlyでのみ使用可能

```rust
pub fn into_boxed_slice(boxed: Box<T, A>) -> Box<[T], A>
```

`Box<T>`を`Box<[T]>`へ変換する。
変換にあたってヒープの割り当てはされずその場所で行われる。

### `into_inner`

> [!NOTE]
> nightlyでのみ使用可能

```rust
pub fn into_inner(boxed: Box<T, A>) -> T
```

`Box`を消費して中身の値を返す。

```rust
#![feature(box_into_inner)]

let c = Box::new(5);

assert_eq!(Box::into_inner(c), 5);
```

### `take`

> [!NOTE]
> nightlyでのみ使用可能

```rust
pub fn take(boxed: Box<T, A>) -> (T, Box<MaybeUninit<T>, A>)
```

メモリーの割り当てを消費せずに`Box`を消費し、中身の値と値が入っていた領域が未初期化の状態の`Box`を返す。
[`write`](#write)とともに使用することでboxと割り当てを再利用することができる。


```rust
#![feature(box_take)]

let c = Box::new(5);

// boxから値を取り出す
let (value, uninit) = Box::take(c);
assert_eq!(value, 5);

// 別の値のためにboxを再利用する
let c = Box::write(uninit, 6);
assert_eq!(*c, 6);
```


```rust
impl<T> Box<T>
where
    T: CloneToUninit + ?Sized,
```

### clone_from_ref

> [!NOTE]
> nightlyでのみ使用可能

```rust
pub fn clone_from_ref(src: &T) -> Box<T>
```

ヒープ上にメモリーを割り当て複製した`src`を格納する。
`src`がゼロサイズの場合、実際にはメモリーは割り当てられない。

```rust
#![feature(clone_from_ref)]

let hello: Box<str> = Box::clone_from_ref("hello");
```