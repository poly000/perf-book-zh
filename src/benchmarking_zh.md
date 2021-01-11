# 基准测试

在优化一个程序时，你需要一种方法来可靠地验证这个改变是否加快了程序。这个过程有时被称为基准测试(benchmarking)。基准测试是一个复杂的话题，全面的介绍超出了本书的范围，但以下是基本的内容。

首先，你需要工作负载来衡量。理想情况下，你会有各种工作负载，代表你的程序的实际使用情况。使用真实世界输入的工作负载是最好的，但适度进行[微基准测试]和[压力测试(stress tests)]也是有用的。

[微基准测试]: https://stackoverflow.com/questions/2842695/what-is-microbenchmarking
[压力测试(stress tests)]: https://www.wanweibaike.com/wiki-%E5%A3%93%E5%8A%9B%E6%B8%AC%E8%A9%A6%20(%E8%BB%9F%E9%AB%94)

其次，你需要一种运行工作负载的方法，这也将决定使用的指标。Rust内置的[基准测试 (benchmark tests)]是一个简单的起点，但它使用不稳定特性且只在夜版 (Nightly) Rust 下可用。
[`bencher`] 包类似，但对稳定Rust可用。[Criterion] 是一个更复杂的选择。自定义基准测试框架也是可能的。例如，[rustc-perf] 就是用来对Rust编译器进行基准测试的框架。

[基准测试(benchmark tests)]: https://doc.rust-lang.org/1.7.0/book/benchmark-tests.html
[Criterion]: https://github.com/bheisler/criterion.rs
[rustc-perf]: https://github.com/rust-lang/rustc-perf/
[`bencher`]: https://crates.io/crates/bencher

当涉及到指标时，有许多选择，正确的选择将取决于被基准化的程序的性质。例如，对批处理程序有意义的指标可能对交互式程序没有意义。在许多情况下，
挂钟时间 (wall-time) 是一个显而易见的选择，因为它符合用户的感知。然而，它可能会受到高变化率的影响。特别是，内存布局的微小变化可能会导致显著但短暂的性能波动。因此，变化率较小的其他指标 \[如周期 (cycles) 或指令数 (instruction counts) \] 可能是合理的选择。

从多个工作负载中总结测量结果也是一个挑战，尽管有多种方法，但并没有一种方法是最完美的。

良好的基准测试是很难实现的。总而言之，不要过于强调拥有一个完美的基准测试设置，特别是当你刚开始优化一个程序的时候。
一个平庸的设置比没有设置要好得多。对你正在测量的东西保持开放的心态，随着时间的推移，你可以在了解你的程序的性能特点后进行基准测试改进。

