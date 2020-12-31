# 标准库类型

值得阅读常见标准库类型的文档--如[`Vec`]、[`Option`]、[`Result`]和[`Rc`]/[`Arc`]--以找到有趣的函数，有时可以用来提高性能。

[`Vec`]: https://doc.rust-lang.org/std/vec/struct.Vec.html
[`Option`]: https://doc.rust-lang.org/std/option/enum.Option.html
[`Result`]: https://doc.rust-lang.org/std/result/enum.Result.html
[`Rc`]: https://doc.rust-lang.org/std/rc/struct.Rc.html
[`Arc`]: https://doc.rust-lang.org/std/sync/struct.Arc.html

还值得了解标准库类型的高性能替代品，如[`Mutex`]、[`RwLock`]、[`Condvar`]和[`Once`]。

[`Mutex`]: https://doc.rust-lang.org/std/sync/struct.Mutex.html
[`RwLock`]: https://doc.rust-lang.org/std/sync/struct.RwLock.html
[`Condvar`]: https://doc.rust-lang.org/std/sync/struct.Condvar.html
[`Once`]: https://doc.rust-lang.org/std/sync/struct.Once.html

## `Vec`

[`Vec::remove`]删除某一特定索引上的一个元素，并将随后的所有元素向左移动一个，这样做是O(n)。[`Vec::swap_remove`] 用最后一个元素替换特定索引上的一个元素，虽然不保留排序，但也是O(1)。

`Vec::retain`] 有效地从一个`Vec`中删除多个项目。对于其他集合类型，如`String`、`HashSet`和`HashMap`，也有一个等价的方法。

[`Vec::remove`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.remove
[`Vec::swap_remove`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.swap_remove
[`Vec::retain`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.retain

## `Option` and `Result`

[`Option::ok_or']将`Option'转换为`Result'，并传递一个`err'参数，如果`Option'值为`None'，则使用该参数。`err`是急于计算的。如果它的计算很昂贵，你应该使用[`Option::ok_or_else`]，它通过一个闭包缓慢地计算错误值。
例如，这个。
```rust
# fn expensive() {}
# let o: Option<u32> = None;
let r = o.ok_or(expensive()); // always evaluates `expensive()`
```
should be changed to this:
```rust
# fn expensive() {}
# let o: Option<u32> = None;
let r = o.ok_or_else(|| expensive()); // evaluates `expensive()` only when needed
```
[**Example**](https://github.com/rust-lang/rust/pull/50051/commits/5070dea2366104fb0b5c344ce7f2a5cf8af176b0).

[`Option::ok_or`]: https://doc.rust-lang.org/std/option/enum.Option.html#method.ok_or
[`Option::ok_or_else`]: https://doc.rust-lang.org/std/option/enum.Option.html#method.ok_or_else

[`Option::map_or`]、[`Option::unwrap_or`]、[`Result::or`]、[`Result::map_or`]和[`Result::unwrap_or`]有类似的替代方案。

[`Option::map_or`]: https://doc.rust-lang.org/std/option/enum.Option.html#method.map_or
[`Option::unwrap_or`]: https://doc.rust-lang.org/std/option/enum.Option.html#method.unwrap_or
[`Result::or`]: https://doc.rust-lang.org/std/result/enum.Result.html#method.or
[`Result::map_or`]: https://doc.rust-lang.org/std/result/enum.Result.html#method.map_or
[`Result::unwrap_or`]: https://doc.rust-lang.org/std/result/enum.Result.html#method.unwrap_or

## `Rc`/`Arc`

[`Rc::make_mut`]/[`Arc::make_mut`] 提供了写时克隆语义。他们创造一个到
 `Rc`/`Arc` 的可变借用。如果引用计数比一大，他们
将克隆内部值以确保独立的所有权；否则，他们将修改源值。
它们不经常被需要，但他们可能极有用。
[**Example 1**](https://github.com/rust-lang/rust/pull/65198/commits/3832a634d3aa6a7c60448906e6656a22f7e35628),
[**Example 2**](https://github.com/rust-lang/rust/pull/65198/commits/75e0078a1703448a19e25eac85daaa5a4e6e68ac).

[`Rc::make_mut`]: https://doc.rust-lang.org/std/rc/struct.Rc.html#method.make_mut
[`Arc::make_mut`]: https://doc.rust-lang.org/std/sync/struct.Arc.html#method.make_mut

## `Mutex`, `RwLock`, `Condvar`, and `Once`

[`parking_lot`]提供了这些同步类型的替代实现，它们比标准库中的同步类型更小、更快、更灵活。`parking_lot`类型的API和语义与标准库中的等价类型相似，但不完全相同。

[`parking_lot`]: https://crates.io/crates/parking_lot
