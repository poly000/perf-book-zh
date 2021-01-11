# 散列

`HashSet` 和 `HashMap` 是两种广泛使用的类型。默认的散列算法没有指定，但在编写本文时，默认的是一种叫做 [SipHash 1-3] 的算法。这种算法质量很高————它提供了很高的防碰撞保护————但速度相对较慢，特别是对于短键，如整数。

[SipHash 1-3]: https://en.wikipedia.org/wiki/SipHash

如果测试显示散列是关键部分，而[散列拒绝服务攻击]并不是你的应用所关心的问题，那么使用具有更快的散列算法的散列表可以提供很大的速度优势。
- [`rustc-hash`] 提供了 "FxHashSet "和 "FxHashMap "类型，它们是 "HashSet "和 "HashMap "的替代物。它的散列算法质量不高，但速度非常快，特别是对整数键而言，并且发现它的性能优于rustc内的所有其他散列算法。
- [`fnv`] 提供了 `FnvHashSet` 和 `FnvHashMap` 类型。其散列算法比 `fxhash` 的质量高，但速度稍慢。
- [`ahash`] 提供 `AHashSet` 和 `AHashMap`。它的哈希算法可以采取一些处理器上的AES指令支持的优势。

[散列拒绝服务攻击]: https://en.wikipedia.org/wiki/Collision_attack
[`rustc-hash`]: https://crates.io/crates/rustc-hash
[`fnv`]: https://crates.io/crates/fnv
[`ahash`]: https://crates.io/crates/ahash

如果散列性能在你的程序中很重要，那么值得尝试以上几种选择。例如，在rustc中看到以下结果。
- 从 `fnv` 切换到 `rustc-hash` 的结果是[速度提高了6%]。
- 试图从 `rustc-hash` 切换到 `ahash` 的结果是[减速1-4%]。
- 试图从 `rustc-hash` 切换回默认的哈希，结果是[速度减慢了4-84%]!

[速度提高了6%]: https://github.com/rust-lang/rust/pull/37229/commits/00e48affde2d349e3b3bfbd3d0f6afb5d76282a7
[减速1-4%]: https://github.com/rust-lang/rust/issues/69153#issuecomment-589504301
[速度减慢了4-84%]: https://github.com/rust-lang/rust/issues/69153#issuecomment-589338446

如果你决定普遍使用 `FxHashSet`/`FxHashMap` 等备选方案之一，很容易在某些地方偶尔使用 `HashSet`/`HashMap`。在配置文件中出现 `SipHasher13` 代码就是一个很明显的指标。

散列函数的设计是一个复杂的话题，不在本书的讨论范围之内，[`ahash` 文档]有很好的讨论。

[`ahash` 文档]: https://github.com/tkaitchuck/aHash/blob/master/compare/readme.md
