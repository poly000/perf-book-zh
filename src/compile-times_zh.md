# 编译时间

虽然本书的主要内容是提高Rust程序的性能，但本节的内容是关于减少Rust程序的编译时间，因为这是很多人感兴趣的相关话题。

## 链接

编译时间的很大一部分其实是链接时间，尤其是在小改动后重新构建程序的时候。在Linux和Windows上，你可以选择lld作为链接器，这比默认的链接器快得多。

要从命令行指定ld，在你的编译命令前加上`RUSTFLAGS="-C link-arg=-fuse-ld=lld"`。

要从`Cargo.toml`文件中指定lld，请添加这些行。
```text
[build]
rustflags = ["-C", "link-arg=-fuse-ld=lld"]
```
或者，增加如下配置
```text
[target.x86_64-unknown-linux-gnu]
linker = "lld"
```
您可以使用[Cargo配置文件]将这些配置应用于多个项目。

[Cargo配置文件]: https://doc.rust-lang.org/cargo/reference/config.html

lld还没有完全支持在Rust中使用，但它应该可以在Linux和Windows中的大多数情况下使用。有一个[GitHub Issue]跟踪lld的完整支持。

[GitHub Issue]: https://github.com/rust-lang/rust/issues/39915#issuecomment-618726211

## 增量编译

Rust编译器支持[增量编译]，这可以避免在重新编译一个crate时重做工作。它可以大大加快编译速度，但代价是有时会使生成的可执行文件运行得更慢一些。出于这个原因，它只在调试编译时默认启用。如果你想在发布版本的编译中也启用它，请在`Cargo.toml`文件中添加以下行。
```toml
[profile.release]
incremental = true
```
关于`incremental`设置以及为不同的配置文件启用特定设置的详细信息，请参见[Cargo文档]。

[增量编译]: https://blog.rust-lang.org/2016/09/08/incremental.html
[Cargo文档]: https://doc.rust-lang.org/cargo/reference/profiles.html#incremental

## 可视化

Rust编译器有一个功能，可以让你可视化编译你的程序。用这个命令进行编译。
```text
cargo +nightly build -Ztimings
```
完成后，它将打印一个HTML文件的名称。在网络浏览器中打开该文件。它包含了一个[Gantt chart]，显示了你的程序中各个箱子之间的依赖关系。这显示了你的装箱图中有多少并行性，这可以说明是否应该将任何串行化编译的大crate应被打断。见 [此文档][Z-timings] for more
details on how to read the graphs.

[Gantt chart]: https://en.wikipedia.org/wiki/Gantt_chart
[Z-timings]: https://doc.rust-lang.org/nightly/cargo/reference/unstable.html#timings

## LLVM IR

Rust编译器的后端使用[LLVM]。LLVM的执行会占到编译时间的很大一部分，尤其是当Rust编译器的前端产生大量的[IR]时，这需要LLVM花很长的时间去优化。

[LLVM]: https://llvm.org/
[IR]: https://en.wikipedia.org/wiki/Intermediate_representation

这些问题可以用[`cargo llvm-line`]来诊断，它显示了哪些Rust函数导致了最多的LLVM IR生成。通用函数通常是最重要的函数，因为它们在大型程序中可以被实例化几十次甚至几百次。

[`cargo llvm-lines`]: https://github.com/dtolnay/cargo-llvm-lines/

如果一个通用函数导致IR膨胀，有几种方法可以解决。最简单的方法就是把函数变小。
[**Example**](https://github.com/rust-lang/rust/pull/72166/commits/5a0ac0552e05c079f252482cfcdaab3c4b39d614).

另一种方法是将函数的非通用部分移到一个单独的非通用函数中，这个函数将只被实例化一次。这是否可能取决于泛型函数的细节。非通用函数通常可以写成通用函数中的一个内部函数，以尽量减少它对代码其余部分的暴露。
[**Example**](https://github.com/rust-lang/rust/pull/72013/commits/68b75033ad78d88872450a81745cacfc11e58178).

有时，像[`Option::map`]和[`Result::map_err`]这样的常用实用函数会被实例化多次。 用等价的`match`表达式替换它们可以帮助编译时间。

[`Option::map`]: https://doc.rust-lang.org/std/option/enum.Option.html#method.map
[`Result::map_err`]: https://doc.rust-lang.org/std/result/enum.Result.html#method.map_err

这些变化对编译时间的影响通常是很小的，但偶尔也会很大。
[**Example**](https://github.com/servo/servo/issues/26585).
