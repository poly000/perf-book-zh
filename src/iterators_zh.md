# Iterators

## `collect` 与 `extend`

[`Iterator::collect`]将一个迭代器转换为一个集合，如`Vec`，它通常需要一个分配。如果该集合只是再次迭代，你应该避免调用`collect`。

[`Iterator::collect`]: https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.collect

出于这个原因，从函数中返回一个迭代器类型，比如`impl Iterator<Item=T>`，往往比`Vec<T>`更好。请注意，有时这些返回类型需要额外的生存期，正如[this post]所解释的那样。
[**Example**](https://github.com/rust-lang/rust/pull/77990/commits/660d8a6550a126797aa66a417137e39a5639451b).

[this post]: https://blog.katona.me/2019/12/29/Rust-Lifetimes-and-Iterators/

同样，你可以使用[`extend`]用迭代器扩展一个现有的集合（如`Vec`），而不是将迭代器收集到`Vec`中，然后使用[`append`]。

[`extend`]: https://doc.rust-lang.org/std/iter/trait.Extend.html#tymethod.extend
[`append`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.append

最后，当你编写一个迭代器，通常值得实现 [`Iterator::size_hint`] 或 [`ExactSizeIterator::len`] 方法，可能的话。
使用此迭代器的 `collect` 与 `extend` 调用可能将使用更少的分配，因为他们有此迭代器转出的元素数量的预先信息。

[`Iterator::size_hint`]: https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.size_hint
[`ExactSizeIterator::len`]: https://doc.rust-lang.org/std/iter/trait.ExactSizeIterator.html#method.len

## 连接

[`chain`]可以非常方便，但也可能比单个迭代器慢。如果可能的话，热迭代器可能值得避免。
[**Example**](https://github.com/rust-lang/rust/pull/64801/commits/5ca99b750e455e9b5e13e83d0d7886486231e48a).

类似地，[`filter_map`]可能比使用[`filter`]和[`map`]更快。

[`chain`]: https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.chain
[`filter_map`]: https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.filter_map
[`filter`]: https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.filter
[`map`]: https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.map

