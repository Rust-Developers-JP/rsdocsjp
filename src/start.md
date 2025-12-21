# rsdocsjp - Rust言語非公式日本語解説

rsdocsjpは、Rust標準ライブラリおよび関連仕様について、公式ドキュメントの単純な翻訳ではなく、各ライターの理解と解釈に基づいて日本語で解説する非公式プロジェクトです。

正確性には最大限配慮していますが、内容の完全性・公式性を保証するものではありません。
原典については必ず公式ドキュメントを参照してください。

## 方針

本サイトは常にRust言語の最新リファレンスを掲載し続けることを目標にしています。
また、Rust言語の入門として使えるよう、わかりやすさも重視しております。
基本的には公式ドキュメントを参考にしながら、編集者の独自の理解と解釈に基づいて解説を書きます。

## 本サイトを閲覧するにあたって

本サイトはよりわかりやすくするために、コードブロックではフルパスを書くようにしています。
ただし、以下の構造体・トレイトなどは除外されます。

- `std::marker::Copy`
- `std::marker::Copy` (deriveマクロ)
- `std::marker::Send`
- `std::marker::Sized`
- `std::marker::Sync`
- `std::marker::Unpin`
- `std::ops::Drop`
- `std::ops::Fn`
- `std::ops::FnMut`
- `std::ops::FnOnce`
- `std::ops::AsyncFn`
- `std::ops::AsyncFnMut`
- `std::ops::AsyncFnOnce`
- `std::mem::drop`
- `std::mem::align_of`
- `std::mem::align_of_val`
- `std::mem::size_of`
- `std::mem::size_of_val`
- `std::clone::Clone`
- `std::clone::Clone` (deriveマクロ)
- `std::cmp::Eq`
- `std::cmp::Eq` (deriveマクロ)
- `std::cmp::Ord`
- `std::cmp::Ord` (deriveマクロ)
- `std::cmp::PartialEq`
- `std::cmp::PartialEq` (deriveマクロ)
- `std::cmp::PartialOrd`
- `std::cmp::PartialOrd` (deriveマクロ)
- `std::convert::AsMut`
- `std::convert::AsRef`
- `std::convert::From`
- `std::convert::Into`
- `std::convert::TryFrom`
- `std::convert::TryInto`
- `std::default::Default`
- `std::default::Default` (deriveマクロ)
- `std::iter::DoubleEndedIterator`
- `std::iter::ExactSizeIterator`
- `std::iter::Extend`
- `std::iter::IntoIter`
- `std::iter::Iterator`
- `std::iter::FromIterator`
- `std::option::Option`
- `std::option::Option::None`
- `std::option::Option::Some`
- `std::result::Result`
- `std::result::Result::Err`
- `std::result::Result::Ok`
- `std::fmt::macros::Debug`  (deriveマクロ)
- `std::hash::macros::Hash`  (deriveマクロ)
- `std::assert`
- `std::cfg`
- `std::column`
- `std::compile_error`
- `std::concat`
- `std::env`
- `std::file`
- `std::format_args`
- `std::include`
- `std::include_bytes`
- `std::include_str`
- `std::line`
- `std::log_syntax` (nightlyのみ)
- `std::module_path`
- `std::option_env`
- `std::stringify`
- `std::trace_macros` (nightlyのみ)
- `std::concat_bytes` (nightlyのみ)
- `std::future::Future`
- `std::future::IntoFuture`

フルを書かないパスは`std::prelude::rust_2024`に基づいています。

## ライセンス
Rustの慣習に従い、明記されてない限りはMIT OR Apache License 2.0で管理されています。
