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

### 
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

Boxの中身をT型へ変換する関数。

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
