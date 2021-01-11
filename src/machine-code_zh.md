# 机器代码

当你有一小段非常热点的代码(hot code)时，可能值得检查一下生成的机器代码，看看它是否有任何效率低下的地方。在这样做的时候，[Compiler Explorer] 网站是一个很好的资源。

[Compiler Explorer]: https://godbolt.org/

与此相关的是，[`core::arch`] 模块提供了对特定架构的内置函数的访问，其中许多与SIMD指令有关。

[`core::arch`]: https://doc.rust-lang.org/core/arch/index.html

有时可以通过在索引变量的范围内添加断言来避免循环内的边界检查。这是一种先进的技术，你应该检查生成的代码以确保边界检查被实际删除。
[**Example 1**](https://github.com/rust-random/rand/pull/960/commits/de9dfdd86851032d942eb583d8d438e06085867b),
[**Example 2**](https://github.com/image-rs/jpeg-decoder/pull/167/files).

