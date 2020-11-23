# Linting

[Clippy]是一个用来捕捉Rust代码中常见错误的lints集合。它是运行在Rust代码上的一个优秀工具。它还可以帮助提高性能，因为许多lints与可能导致次优性能的代码模式有关。
[Clippy]: https://github.com/rust-lang/rust-clippy

安装后，很容易运行它
```text 
cargo clippy
```
通过访问[lint list]并取消选择除 "Perf "以外的所有lints组，可以看到完整的性能lints列表。

[lint list]: https://rust-lang.github.io/rust-clippy/master/

除了让代码更快之外，性能上的lint建议通常会让代码更简单、更习惯，所以即使是不热门的代码也值得遵循。
