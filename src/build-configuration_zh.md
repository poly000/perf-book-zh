# 构建配置

正确的构建配置将最大限度地提高Rust程序的性能，而无需对其代码进行任何修改。

## 发布构建

最重要的一个Rust性能提示很简单，但[很容易被忽视]：当你想要高性能时，确保你使用的是发布构建而不是调试构建。这通常是通过在Cargo中指定`--release`标志来实现的。

[很容易被忽视]: https://github.com/poly000/perf-book-zh/blob/master/extern/why-my-rust-program-is-so-slow.md

发布版的运行速度通常比调试版快很多。比调试版本快10-100倍是很常见的。

默认为调试构建。如果你运行 "cargo build"、"cargo run "或 "rustc "而不添加任何额外的选项，它们就会被处理。调试构建对调试很有帮助，但不被优化。

考虑一下`cargo build`的最后一行输出。
```text
Finished dev [unoptimized + debuginfo] target(s) in 29.80s
```
`[unoptimized + debuginfo]`表示已经生成了一个调试构建，编译后的代码将放置在`target/debug/`目录下，`cargo run`将运行调试编译的代码。

发布构建比调试构建更多的优化。它们也省略了一些检查，比如调试断言和整数溢出检查。用 `cargo build --release`, `cargo run --release`, 或 `rustc -O`编译。(并且，`rustc`有多个其他优化构建的选项，比如`-C opt-level`)。由于额外的优化，这通常会比调试构建花费更多时间。

请看下面的`"cargo build --release"`运行的最后一行输出。
```text
Finished release [optimized] target(s) in 1m 01s
```
`[optimed]`表示已经生成了一个发布构建。编译后的代码将被放置在`target/release/`目录下。`cargo run --release`将运行发布版。

参见 [Cargo 配置文件]，以了解更多关于调试构建(使用`dev`配置文件)和发布构建(使用`release`配置文件)之间的区别。

[Cargo 配置文件]: https://github.com/poly000/perf-book-zh/blob/master/extern/cargo-profiles.md

## 链接时优化

链接时间优化(LTO)是一种整体程序优化技术，可以将运行时性能提高10-20%甚至更多，但代价是增加构建时间。对于任何一个单独的Rust程序，很容易看出运行时间与编译时间的权衡是否值得。

尝试LTO最简单的方法是在`Cargo.toml`文件中添加以下几行，然后进行发布构建。
```toml
[profile.release]
lto = true
```
这将导致 "fat" LTO，它对依赖图中的所有crate进行优化。

或者，在`Cargo.toml`中使用`lto = "thin"`来使用 "thin"LTO，这是一种不那么激进的LTO形式，通常和 "fat"LTO一样好用，而不会增加那么多的构建时间。

请参阅[Cargo LTO文档]了解更多关于`lto`设置的细节，以及关于为不同配置文件启用特定设置的细节。

[Cargo LTO文档]: https://doc.rust-lang.org/cargo/reference/profiles.html#lto

## 代码生成单元

Rust 编译器会将你的 crate 分割成多个 [代码生成单元]，以实现并行化（从而加快）编译。然而，这可能会导致它错过一些潜在的优化。如果你想以更大的编译时间为代价来潜在地提高运行时性能，你可以将单元数设置为一个。
```toml
[profile.release]
codegen-units = 1
```
[**Example**](https://likebike.com/posts/How_To_Write_Fast_Rust_Code.html#emit-asm).

[代码生成单元]: https://doc.rust-lang.org/rustc/codegen-options/index.html#codegen-units

要注意的是，代码生成单元数是一个启发式的，因此较小的单位数实际上会导致程序较慢。

## 使用CPU专用指令

如果你不太关心你的二进制文件在旧的（或其他类型的）处理器上的兼容性，你可以告诉编译器生成特定于[特定CPU架构]的最新（可能是最快的）指令。

[特定CPU架构]: https://doc.rust-lang.org/1.41.1/rustc/codegen-options/index.html#target-cpu

例如，如果你把`-C target-cpu=native`传给rustc，它将使用当前CPU的最佳指令。
```bash
$ RUSTFLAGS="-C target-cpu=native" cargo build --release
```

这可能会产生很大的影响，特别是当编译器在你的代码中发现矢量化的机会时。

## 在`panic!`时中止

如果你不需要捕捉或解除恐慌，你可以告诉编译器在恐慌时简单地中止。这可能会减少二进制大小，并略微提高性能。
```toml
[profile.release]
panic = "abort"
```

## Profile-guided Optimization

Profile-guided optimization(PGO)是一种编译模式，即编译程序后，在收集样本数据的同时在样本数据上运行，然后用样本数据引导程序的第二次编译。
[**Example**](https://blog.rust-lang.org/inside-rust/2020/11/11/exploring-pgo-for-the-rust-compiler.html).

这是一种先进的技术，需要花费一些精力来设置，但在某些情况下是值得的。详情请看[rustc PGO documentation]。

[rustc PGO documentation]: https://doc.rust-lang.org/rustc/profile-guided-optimization.html
