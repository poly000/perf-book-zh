# Logging and Debugging

有时，日志代码或调试代码会大大降低程序的速度。要么是日志记录/调试代码本身很慢，要么是反馈到日志记录/调试代码的数据收集代码很慢。确保在不启用日志记录/调试时，不为日志记录/调试目的做不必要的工作。
[**Example 1**](https://github.com/rust-lang/rust/pull/50246/commits/2e4f66a86f7baa5644d18bb2adc07a8cd1c7409d),
[**Example 2**](https://github.com/rust-lang/rust/pull/75133/commits/eeb4b83289e09956e0dda174047729ca87c709fe).

请注意，[`assert!`]调用总是运行，但[`debug_assert!`]调用只在调试构建中运行。如果你有一个热的断言，但对安全来说不是必需的，可以考虑把它变成一个`debug_assert!`。
[**Example**](https://github.com/rust-lang/rust/pull/58210/commits/f7ed6e18160bc8fccf27a73c05f3935c9e8f672e).

[`assert!`]: https://doc.rust-lang.org/std/macro.assert.html
[`debug_assert!`]: https://doc.rust-lang.org/std/macro.debug_assert.html
