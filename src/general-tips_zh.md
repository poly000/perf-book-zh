# General Tips

本书的前几节已经讨论了Rust特有的技术。
本节将简要介绍一些一般的性能原则。

只要避免了明显的缺陷（例如[使用非发布的构建]）。
Rust一般都有不错的性能。尤其是你习惯于Python和Ruby等动态类型的语言时。

[使用非发布的构建]: build-configuration.md

优化后的代码往往比未优化的代码更复杂，写起来更费劲。出于这个原因，只有优化热代码才是值得的。

最大的性能提升往往来自于算法或数据结构的改变，而不是低级别的优化。
[**Example 1**](https://github.com/rust-lang/rust/pull/53383/commits/5745597e6195fe0591737f242d02350001b6c590),
[**Example 2**](https://github.com/rust-lang/rust/pull/54318/commits/154be2c98cf348de080ce951df3f73649e8bb1a6).

编写与现代硬件良好配合的代码并不总是容易的，但值得努力。例如，在可能的情况下，尽量减少缓存遗漏和分支误判。

大多数优化都会带来小幅提速。虽然没有一个小的提速是明显的，但如果你能做足够多的提速，它们真的会加起来。

不同的剖析器有不同的优势。使用多个剖析器是件好事。

当剖析表明一个函数很热的时候，有两种常见的方法来加速。
· 让函数更快
· 避免调用它。

通常，相较于聪明的提速，消除愚蠢的减速更容易。

除非必要，否则避免全部求值。惰性/按需 求值往往是一种好办法。
[**Example 1**](https://github.com/rust-lang/rust/pull/36592/commits/80a44779f7a211e075da9ed0ff2763afa00f43dc),
[**Example 2**](https://github.com/rust-lang/rust/pull/50339/commits/989815d5670826078d9984a3515eeb68235a4687).

一般复杂的情况，往往可以通过乐观地检查比较简单的常见特殊情况来避免。
[**Example 1**](https://github.com/rust-lang/rust/pull/68790/commits/d62b6f204733d255a3e943388ba99f14b053bf4a),
[**Example 2**](https://github.com/rust-lang/rust/pull/53733/commits/130e55665f8c9f078dec67a3e92467853f400250),
[**Example 3**](https://github.com/rust-lang/rust/pull/65260/commits/59e41edcc15ed07de604c61876ea091900f73649).
尤其是在小尺寸占主导地位的情况下，特别处理0、1或2个元素的集合往往是一种好办法。
[**Example 1**](https://github.com/rust-lang/rust/pull/50932/commits/2ff632484cd8c2e3b123fbf52d9dd39b54a94505),
[**Example 2**](https://github.com/rust-lang/rust/pull/64627/commits/acf7d4dcdba4046917c61aab141c1dec25669ce9),
[**Example 3**](https://github.com/rust-lang/rust/pull/64949/commits/14192607d38f5501c75abea7a4a0e46349df5b5f),
[**Example 4**](https://github.com/rust-lang/rust/pull/64949/commits/d1a7bb36ad0a5932384eac03d3fb834efc0317e5).

同样，在处理重复性数据时，通常可以使用一种简单的数据压缩形式，对常见的值使用紧凑的表示方式，然后对不常见的值进行回退到二级表。
[**Example 1**](https://github.com/rust-lang/rust/pull/54420/commits/b2f25e3c38ff29eebe6c8ce69b8c69243faa440d),
[**Example 2**](https://github.com/rust-lang/rust/pull/59693/commits/fd7f605365b27bfdd3cd6763124e81bddd61dd28),
[**Example 3**](https://github.com/rust-lang/rust/pull/65750/commits/eea6f23a0ed67fd8c6b8e1b02cda3628fee56b2f).

当代码处理多种情况时，要衡量情况频率，先处理最常见的情况。

当处理涉及高局部性的查找时，在数据结构前面放一个小的缓存是一种胜利。

优化后的代码往往具有不明显的结构，这意味着解释性注释是有价值的，特别是那些参考剖析测量的注释。像 "99%的时候这个向量都是0或1个元素，所以要先处理这些情况 "这样的评论可以起到启发作用。
