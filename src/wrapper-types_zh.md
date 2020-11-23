# Wrapper Types

Rust有多种 "封装 "类型，如[`RefCell`]和[`Mutex`]，它们为值提供了特殊行为。访问这些值可能会耗费大量的时间。如果多个这样的值通常是一起访问的，那么最好将它们放在一个包装器中。

[`RefCell`]: https://doc.rust-lang.org/std/cell/struct.RefCell.html
[`Mutex`]: https://doc.rust-lang.org/std/sync/struct.Mutex.html

例如，这样的结构。
```rust
# use std::sync::{Arc, Mutex};
struct S {
    x: Arc<Mutex<u32>>,
    y: Arc<Mutex<u32>>,
}
```
也许这样更典型。
```rust
# use std::sync::{Arc, Mutex};
struct S {
    xy: Arc<Mutex<(u32, u32)>>,
}
```
这是否有助于性能，将取决于值的具体访问模式。
[**Example**](https://github.com/rust-lang/rust/pull/68694/commits/7426853ba255940b880f2e7f8026d60b94b42404).
