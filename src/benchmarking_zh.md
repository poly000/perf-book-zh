# Benchmarking

在优化一个程序时，你需要一种方法来可靠地回答 "这个改变是否加快了程序？"这个问题。这个过程有时被称为基准测试(benchmarking)。基准测试是一个复杂的话题，彻底的覆盖范围超出了本书的范围，但以下是基本的内容。

首先，你需要工作负载来衡量。理想情况下，你会有各种工作负载，代表你的程序的实际使用情况。使用真实世界输入的工作负载是最好的，但[microbenchmarks]和[压力测试(stress tests)]在适度的情况下也是有用的。

[microbenchmarks]: https://stackoverflow.com/questions/2842695/what-is-microbenchmarking
[stress tests]: https://en.wikipedia.org/wiki/Stress_testing_(software)

其次，你需要一种运行工作负载的方法，这也将决定使用的指标。Rust内置的[基准测试(benchmark tests)]是一个简单的起点。[Criterion]是一个更复杂的选择。自定义benchmarking harnesses也是可能的。例如，[rustc-perf]就是用来对Rust编译器进行基准测试的harnesses。

[benchmark tests]: https://doc.rust-lang.org/1.7.0/book/benchmark-tests.html
[Criterion]: https://github.com/bheisler/criterion.rs
[rustc-perf]: https://github.com/rust-lang/rustc-perf/

当涉及到度量时，有许多选择，正确的选择将取决于被基准化的程序的性质。例如，对批处理程序有意义的指标可能对交互式程序没有意义。在许多情况下，Wall-time是一个显而易见的选择，因为它与用户的感知相对应。然而，它可能会受到大幅变化的影响。特别是，内存布局的微小变化可能会导致显著但短暂的性能波动。因此，方差较小的其他指标（如周期或指令数）可能是一个合理的选择。

从多个工作负载中总结测量结果也是一个挑战，尽管有多种方法，但并没有一种方法是最完美的。

良好好的基准测试是很难实现的。总而言之，不要过于强调拥有一个完美的基准测试设置，特别是当你开始优化一个程序的时候。一个平庸的设置比没有设置要好得多。对你正在测量的东西保持开放的心态，随着时间的推移，你可以在了解你的程序的性能特点后进行基准测试改进。

