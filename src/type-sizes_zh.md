# 类型大小

缩小常用类型有助于减少内存使用峰值。

例如，如果内存占用高，堆性能分析工具如 [DHAT] 与 [heaptrack] 可以检验热分配点以及涉及的类型。
缩小这些类型可以减少内存使用峰值，而且可能会通过减少内存流量以及缓存压力提升性能。

[DHAT]: https://www.valgrind.org/docs/manual/dh-manual.html
[heaptrack]: https://github.com/KDE/heaptrack

如果 `memcpy` 在性能测试中出现的数量不多，可以使用DHAT的 “复制分析” 模式显示 `memcpy` 热调用点和涉及的类型。
将这些类型缩小到128字节或更少，可以通过避免 `memcpy` 调用和减少内存流量提升性能。



## 验证类型大小

[`std::mem::size_of`]给出了一个类型的大小，以字节为单位，但通常你也想知道确切的布局。例如，一个枚举体可能会出乎意料的大，这可能是由一个超大的变体造成的。

[`std::mem::size_of`]: https://doc.rust-lang.org/std/mem/fn.size_of.html

`-Zprint-type-sizes'选项正是这样做的，它在rustc的发行版上没有被启用，所以你需要使用rustc的夜间版本。 下面是一个通过Cargo的可能调用
```text
RUSTFLAGS=-Zprint-type-sizes cargo +nightly build --release
```
而这里是一个rustc的可能调用
```text
rustc +nightly -Zprint-type-sizes input.rs
```
它将打印出所有使用中的类型的尺寸、布局和对齐方式的详细信息。例如，对于这种类型。
```rust
enum E {
    A,
    B(i32),
    C(u64, u8, u64, u8),
    D(Vec<u32>),
}
```
它打印以下信息，以及一些内置类型的信息。
```text
print-type-size 类型: `E`: 32 字节, 对齐: 8 字节
print-type-size     判别: 1 字节
print-type-size     变体 `D`: 31 字节
print-type-size         填充: 7 字节
print-type-size         字段 `.0`: 24 字节, 对齐: 8 字节
print-type-size     变体 `C`: 23 字节
print-type-size         字段 `.1`: 1 字节
print-type-size         字段 `.3`: 1 字节
print-type-size         填充: 5 字节
print-type-size         字段 `.0`: 8 字节, 对齐: 8 字节
print-type-size         字段 `.2`: 8 字节
print-type-size     变体 `B`: 7 字节
print-type-size         填充: 3 字节
print-type-size         字段 `.0`: 4 字节, 对齐: 4 字节
print-type-size     变体 `A`: 0 字节
```
输出显示以下内容。
- 类型的大小和排列。
- 对于enums，判别子的大小。
- 对于enums，每个变量的大小（从最大到最小排序）。
- 所有字段的大小、对齐和排序。(请注意，编译器对变体`C`的字段进行了重新排序，以最小化`E`的大小。)
- 所有填充的大小和位置。

一旦你知道了热类型的布局，就有多种方法来收缩它。

## 字段顺序

Rust编译器自动排序结构体与枚举体中的字段以最小化它们（除非指定了 `#[repr(C)]` 属性）。
所以你无需担心字段顺序，但是还有其他方法最小化热类型。

## 更小的枚举体

如果一个枚举有一个超大的变体，可以考虑将一个或多个字段装箱。例如，你可以改变这个类型。
```rust
type LargeType = [u8; 100];
enum A {
    X,
    Y(i32),
    Z(i32, LargeType),
}
```
修改为：
```rust
# type LargeType = [u8; 100];
enum A {
    X,
    Y(i32),
    Z(Box<(i32, LargeType)>),
}
```
这减少了类型大小，但代价是需要为`A::Z`变体分配一个额外的堆。如果`A::Z`变体比较少见，这更有可能成为提高性能的好方法。`Box`也会使`A::Z`的使用略微不合人体工程学，特别是在 `match` 模式中。
[**Example 1**](https://github.com/rust-lang/rust/pull/37445/commits/a920e355ea837a950b484b5791051337cd371f5d),
[**Example 2**](https://github.com/rust-lang/rust/pull/55346/commits/38d9277a77e982e49df07725b62b21c423b6428e),
[**Example 3**](https://github.com/rust-lang/rust/pull/64302/commits/b972ac818c98373b6d045956b049dc34932c41be),
[**Example 4**](https://github.com/rust-lang/rust/pull/64374/commits/2fcd870711ce267c79408ec631f7eba8e0afcdf6),
[**Example 5**](https://github.com/rust-lang/rust/pull/64394/commits/7f0637da5144c7435e88ea3805021882f077d50c),
[**Example 6**](https://github.com/rust-lang/rust/pull/71942/commits/27ae2f0d60d9201133e1f9ec7a04c05c8e55e665).

## 更小的整数

通常可以通过使用较小的整数类型来缩小类型。例如，虽然对索引使用 “usize” 是最自然的，但将索引存储为 "u32"、"u16"、甚至 "u8"，然后在使用处强制转换为 "usize"，往往是合理的。
[**Example 1**](https://github.com/rust-lang/rust/pull/49993/commits/4d34bfd00a57f8a8bdb60ec3f908c5d4256f8a9a),
[**Example 2**](https://github.com/rust-lang/rust/pull/50981/commits/8d0fad5d3832c6c1f14542ea0be038274e454524).

## 装箱的切片

Rust向量包含三个 “字”：一个长度、一个容量和一个指针。如果你有一个将来不太可能被改变的向量，你可以用[`Vec::into_boxed_slice`]把它转换为一个*boxed slice*。一个boxed slice只包含两个词，一个长度和一个指针。任何多余的元素容量都会被丢弃，这可能会导致重新分配。
```rust
# use std::mem::{size_of, size_of_val};
let v: Vec<u32> = vec![1, 2, 3];
assert_eq!(size_of_val(&v), 3 * size_of::<usize>());

let bs: Box<[u32]> = v.into_boxed_slice();
assert_eq!(size_of_val(&bs), 2 * size_of::<usize>());
```
盒状切片可以用[`slice::into_vec`]转换回一个矢量，而无需任何克隆或重新分配。

[`Vec::into_boxed_slice`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.into_boxed_slice
[`slice::into_vec`]: https://doc.rust-lang.org/std/primitive.slice.html#method.into_vec

## 避免退步

如果一个类型足够常用，它的大小会影响性能，那么最好使用静态断言来确保它不会意外地回归。下面的例子使用了[`static_assertions`]中的一个宏。
```rust,ignore
  // 这个类型被大量使用。确保它不会无意识地变大。
  #[cfg(target_arch = "x86_64")]
  static_assertions::assert_eq_size!(HotType, [u8; 64]);
```
`cfg`属性很重要，因为类型大小在不同的平台上会有所不同。将断言限制在 "x86_64"(通常是最广泛使用的平台)可能足以防止实际中的回落。

[`static_assertions`]: https://crates.io/crates/static_assertions

