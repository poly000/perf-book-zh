# Inlining

调用和退出热函数、未内联的函数往往占执行时间的一小部分。内联这些函数可以提供小而简单的速度优势。

有四个内联标识可以用于Rust函数。
- **无**. 编译器会自己决定是否应该内联函数。这将取决于优化级别、函数的大小等。如果你没有使用链接时间优化，函数永远不会跨箱子内联。
- **`#[inline]`**. 这表明该函数应该内嵌，包括跨越crate边界。
- **`#[inline(always)]`**. 这强烈建议该函数应该内嵌，包括跨越箱子边界。
- **`#[inline(never)]`**. 这强烈表示该函数不应被内联。

内联标识并不能保证一个函数是内联或不内联的，但实际上，`#[inline(always)]`除了最特殊的情况外，都会导致内联。

## Simple Cases

最适合内联的是(a)非常小的函数，或者(b)只有一个调用点的函数。编译器通常会自己内联这些函数，即使没有内联属性。但是编译器不可能总是做出最好的选择，所以有时需要标识。
[**Example 1**](https://github.com/rust-lang/rust/pull/37083/commits/6a4bb35b70862f33ac2491ffe6c55fb210c8490d),
[**Example 2**](https://github.com/rust-lang/rust/pull/50407/commits/e740b97be699c9445b8a1a7af6348ca2d4c460ce),
[**Example 3**](https://github.com/rust-lang/rust/pull/50564/commits/77c40f8c6f8cc472f6438f7724d60bf3b7718a0c),
[**Example 4**](https://github.com/rust-lang/rust/pull/57719/commits/92fd6f9d30d0b6b4ecbcf01534809fb66393f139),
[**Example 5**](https://github.com/rust-lang/rust/pull/69256/commits/e761f3af904b3c275bdebc73bb29ffc45384945d).

Cachegrind是一个很好的判断函数是否被内联的剖析器。当查看Cachegrind的输出时，如果（也只有当）函数的第一行和最后一行没有*标记事件数*，你就可以判断该函数被内联了。
例如
```text
      .  #[inline(always)]
      .  fn inlined(x: u32, y: u32) -> u32 {
700,000      eprintln!("inlined: {} + {}", x, y);
200,000      x + y
      .  }
      .  
      .  #[inline(never)]
400,000  fn not_inlined(x: u32, y: u32) -> u32 {
700,000      eprintln!("not_inlined: {} + {}", x, y);
200,000      x + y
200,000  }
```
添加内联属性后应该再测一次，因为效果可能是不可预知的。有时它没有效果，因为附近一个之前内联的函数不再内联了。有时会拖慢代码的速度。内联也会影响编译时间，特别是跨包内联，它涉及到重复函数的内部表示。

## Harder Cases

有时候，你有一个函数很大，有多个调用点，但只有一个调用点是热调用点。你希望内联热调用点以提高速度，但不内联冷调用点以避免不必要的代码膨胀。处理的方法是将函数分成总是内联和从不内联的部分，后者调用前者。

例如，这个函数。
```rust
# fn one() {};
# fn two() {};
# fn three() {};
fn my_function() {
    one();
    two();
    three();
}
```
应该修改为如下函数
```rust
# fn one() {};
# fn two() {};
# fn three() {};
// Use this at the hot call site.
#[inline(always)]
fn inlined_my_function() {
    one();
    two();
    three();
}

// Use this at the cold call sites.
#[inline(never)]
fn uninlined_my_function() {
    inlined_my_function();
}
```
[**Example 1**](https://github.com/rust-lang/rust/pull/53513/commits/b73843f9422fb487b2d26ac2d65f79f73a4c9ae3),
[**Example 2**](https://github.com/rust-lang/rust/pull/64420/commits/a2261ad66400c3145f96ebff0d9b75e910fa89dd).

