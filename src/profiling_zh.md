# 性能分析

在优化程序时，你还需要一种方法来确定程序的哪些部分是 “热” 的（执行频率足以影响运行时间），值得修改。这一点最好通过性能优化来完成。

## 性能分析工具

有许多不同的性能分析工具，每个性能分析工具都有自己的优点和缺点。以下是已经成功用于Rust程序的性能分析工具。
- [perf]是一个使用硬件性能计数器的通用性能分析工具。[Hotspot]和[Firefox Profiler]是好的perf记录的数据的查看工具。它在Linux上可用。
- [AMD μProf] 是一个通用型性能分析工具。它在Linux和Windows上可用。
- [flamegraph]是一个Cargo命令，它使用`perf`/`DTrace`对你的代码进行性能分析，然后用火焰图显示结果。
它在Linux及所有支持Dtrace的平台可用 (macOS, FreeBSD, NetBSD, Windows也有可能)。
- [Cachegrind]和[Callgrind]给出了全局的、每个函数的、每个源线的指令数以及模拟的缓存和分支预测数据。他们在Linux及一些Unix系统上可用。
- [DHAT] 可以很好的找到代码中哪些部分会造成大量的分配，并对内存使用峰值进行深入了解。
  它也可被用于验证 `memcpy` 的调用热点，它在Linux以及一些Unix系统可用。
  [dhat-rs] 是一个替代，稍微不那么强大，并需要对你的Rust程序进行小的改动，但在全平台可用。
- [heaptrack]是另一个堆分析工具。它在linux上可用。
- [`counts`]支持特别性能分析，它将`eprintln！`语句的使用与基于频率的后处理结合起来，这对于了解代码中特定领域的部分内容很有帮助。它在全平台可用。
- [Coz]执行 *因果分析* 以衡量优化潜力。它通过[coz-rs]支持Rust。

[perf]: https://perf.wiki.kernel.org/index.php/Main_Page
[Hotspot]: https://github.com/KDAB/hotspot
[Firefox Profiler]: https://profiler.firefox.com/
[AMD μProf]: https://developer.amd.com/amd-uprof/
[flamegraph]: https://github.com/flamegraph-rs/flamegraph
[Cachegrind]: https://www.valgrind.org/docs/manual/cg-manual.html
[Callgrind]: https://www.valgrind.org/docs/manual/cl-manual.html
[DHAT]: https://www.valgrind.org/docs/manual/dh-manual.html
[dhat-rs]: https://github.com/nnethercote/dhat-rs/
[heaptrack]: https://github.com/KDE/heaptrack
[`counts`]: https://github.com/nnethercote/counts/
[Coz]: https://github.com/plasma-umass/coz
[coz-rs]: https://github.com/plasma-umass/coz/tree/master/rust

## 调试信息

为了有效地性能分析一个发布构建，你可能需要启用源代码行调试信息。要做到这一点，请在你的 `Cargo.toml`文件中添加以下行。
```toml
[profile.release]
debug = 1
```
关于`debug`设置的更多细节，请参见[Cargo文档]。

[Cargo文档]: https://doc.rust-lang.org/cargo/reference/profiles.html#debug

## 符号反混淆

Rust在编译代码中使用一个混淆机制编码函数名。如果一个性能分析器不知道这个机制，它的输出可能会包含像这样的符号名
`_ZN3foo3barE`或`_ZN28_$u7b$$u7b$closure$u7d$$u7d$E`或`_ZN28_$u7b$$$u7d$E`或
`_ZN88_$LT$core.result.Result$LT$$u21$$C$$u20$E$GT$u20$as$u20$std.process.Termination$GT$6report17hfc41d0da4a40b3e8E`。
像这样的名字，可以用[`rustfilt`]手动反混淆。

[`rustfilt`]: https://crates.io/crates/rustfilt
