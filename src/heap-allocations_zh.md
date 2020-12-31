# 堆分配

堆分配的代价不高。具体细节取决于使用的分配器，但每次分配和deallocation通常都需要获取一个全局锁，做一些非平凡的数据结构操作。并可能执行一个系统调用。小分配不一定比大分配便宜。值得了解哪些Rust数据结构和操作会导致分配，因为避免它们可以大大提高性能。

[Rust Container Cheat Sheet]有常见的Rust类型的可视化，是下面章节的绝佳配套。

[Rust Container Cheat Sheet]: https://docs.google.com/presentation/d/1q-c7UAyrUlM-eZyTo1pd8SZ0qwA_wYxmPZVOQkoDmH4/

## 性能分析

如果通用性能分析器显示 "malloc"、"free"和相关函数为热函数，那么很可能值得尝试降低分配率或使用其他分配器。

[DHAT]是降低分配率时可以使用的一个优秀的剖析器。它在Linux和一些Unix系统上可用。它能精确地识别热分配点及其分配率。确切的结果会有所不同，但使用rustc的经验表明，每减少10分配每百万指令率可以有可衡量的性能改进（例如，约1%）。

[DHAT]: https://www.valgrind.org/docs/manual/dh-manual.html

下面是DHAT的一些输出示例。
```text
AP 1.1/25 (2 children) {
  Total:     54,533,440 bytes (4.02%, 2,714.28/Minstr) in 458,839 blocks (7.72%, 22.84/Minstr), avg size 118.85 bytes, avg lifetime 1,127,259,403.64 instrs (5.61% of program duration)
  At t-gmax: 0 bytes (0%) in 0 blocks (0%), avg size 0 bytes
  At t-end:  0 bytes (0%) in 0 blocks (0%), avg size 0 bytes
  Reads:     15,993,012 bytes (0.29%, 796.02/Minstr), 0.29/byte
  Writes:    20,974,752 bytes (1.03%, 1,043.97/Minstr), 0.38/byte
  Allocated at {
    #1: 0x95CACC9: alloc (alloc.rs:72)
    #2: 0x95CACC9: alloc (alloc.rs:148)
    #3: 0x95CACC9: reserve_internal<syntax::tokenstream::TokenStream,alloc::alloc::Global> (raw_vec.rs:669)
    #4: 0x95CACC9: reserve<syntax::tokenstream::TokenStream,alloc::alloc::Global> (raw_vec.rs:492)
    #5: 0x95CACC9: reserve<syntax::tokenstream::TokenStream> (vec.rs:460)
    #6: 0x95CACC9: push<syntax::tokenstream::TokenStream> (vec.rs:989)
    #7: 0x95CACC9: parse_token_trees_until_close_delim (tokentrees.rs:27)
    #8: 0x95CACC9: syntax::parse::lexer::tokentrees::<impl syntax::parse::lexer::StringReader<'a>>::parse_token_tree (tokentrees.rs:81)
  }
}
```
在这个例子中，介绍所有的内容已经超出了本书的范围，但应该清楚的是，DHAT给出了大量关于分配的信息，比如它们发生的地点和频率，它们的规模有多大，它们的寿命有多长，以及它们被访问的频率。

## `Box`

[`Box`]是最简单的堆分配类型。`Box<T>`值是一个在堆上分配的`T`值。

[`Box`]: https://doc.rust-lang.org/std/boxed/struct.Box.html

有时值得将结构体或枚举字段中的一个或多个字段装箱，以使类型更小。(更多信息请参见 [Type Sizes](type-sizes.md) 一章)。

除此之外，`Box`是直接的，并没有提供太多优化的空间。

## `Rc`/`Arc`

[`Rc`]/[`Arc`]类似于`Box`，但堆上的值有两个引用计数。它们允许值共享，这可以是减少内存使用的有效方法。

[`Rc`]：https://doc.rust-lang.org/std/rc/struct.Rc.html
[Arc]：https://doc.rust-lang.org/std/sync/struct.Arc.html

但是，如果用于很少共享的值，它们可以通过堆分配本来可能不会被堆分配的值来提高分配率。
[**Example**](https://github.com/rust-lang/rust/pull/37373/commits/c440a7ae654fb641e68a9ee53b03bf3f7133c2fe).

与`Box`不同的是，在`Rc`/`Arc`值上调用`clone`并不涉及分配。相反，它只是增加一个引用计数。

## `Vec`

[`Vec`]是一种堆分配类型，在优化分配数量和/或尽量减少浪费的空间方面有很大的空间。要做到这一点，需要了解其元素的存储方式。

[`Vec`]：https://doc.rust-lang.org/std/vec/struct.Vec.html

一个 "Vec "包含三个词：一个长度、一个容量和一个指针。如果容量是非零，元素大小是非零，指针将指向堆分配的内存；否则，它将不指向分配的内存。

即使 "Vec "本身不是堆分配的，元素（如果存在且大小非零）也会是堆分配的。如果存在非零大小的元素，那么存放这些元素的内存可能会比必要的大，为未来的元素提供空间。存在的元素数就是长度，不需要重新分配就可以容纳的元素数就是容量。

当向量需要增长到超过其当前容量时，元素将被复制到一个更大的堆分配中，旧的堆分配将被释放。

### `Vec` 增长

用普通方法创建一个新的、空的`Vec`。
([`vec![]`](https://doc.rust-lang.org/std/macro.vec.html) 或 [`Vec::new`] 或 [`Vec::default`]）的长度和容量为零，不需要进行堆分配。如果你反复将单个元素推到`Vec`的末端，它将周期性地重新分配。增长策略没有被指定，但在写这篇文章的时候，它使用了一个准双倍策略，结果是以下容量。0, 4, 8, 16, 32, 64, 等等. (它直接从0跳到4，而不是通过1和2，因为这在实践中[避免了许多分配]。) 随着向量的增长，重新分配的频率将以指数形式减少，但可能浪费的多余容量将以指数形式增加。

[`Vec::new`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.new
[`Vec::default`]: https://doc.rust-lang.org/std/default/trait.Default.html#tymethod.default
[避免了许多分配]: https://github.com/rust-lang/rust/pull/72227

这种增长策略对于可增长的数据结构来说是典型的，在一般情况下是合理的，但如果你事先知道一个向量的可能长度，你可以做的往往更好。如果你有一个热向量分配站点（例如一个热的 [`Vec::push`]调用），值得使用[`eprintln!`]来打印该站点的向量长度，然后做一些后处理（例如使用[`counts`]）来确定长度分布。例如，你可能有很多短向量，也可能有较少的超长向量，优化分配站点的最佳方式也会相应变化。

[`Vec::push`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.push
[`eprintln!`]: https://doc.rust-lang.org/std/macro.eprintln.html
[`counts`]: https://github.com/nnethercote/counts/

### 短 `Vec`

如果你有很多短向量，你可以使用[`smallvec`]crate中的`SmallVec`类型。`SmallVec<[T;N]>`是`Vec`的替代物，它可以在`SmallVec`本身中存储`N`个元素，如果元素数量超过这个数量，就会切换到堆分配。(还需要注意的是，`vec![]`字元必须用`smallvec![]`字元代替。)
[**Example 1**](https://github.com/rust-lang/rust/pull/50565/commits/78262e700dc6a7b57e376742f344e80115d2d3f2),
[**Example 2**](https://github.com/rust-lang/rust/pull/55383/commits/526dc1421b48e3ee8357d58d997e7a0f4bb26915).

[`smallvec`]: https://crates.io/crates/smallvec

`SmallVec`如果使用得当，可以可靠地降低分配率，但使用它并不能保证提高性能。对于正常的操作，它比`Vec`稍慢，因为它必须总是检查元素是否被堆分配。另外，如果`N`很高或者`T`很大，那么`SmallVec<[T; N]>`本身就会比`Vec<T>`大，复制`SmallVec`值的速度会比较慢。和以往一样，需要通过基准测试来确认优化是否有效。

如果你有很多短向量，并且你精确地知道它们的最大长度，[arrayvec]箱子中的ArrayVec比SmallVec更好。它不需要回落到堆分配，这使得它更快一些。
[**Example**](https://github.com/rust-lang/rust/pull/74310/commits/c492ca40a288d8a85353ba112c4d38fe87ef453e).

[`arrayvec`]: https://crates.io/crates/arrayvec

### 更长的 `Vec`

如果你知道一个向量的最小或精确大小，你可以用[`Vec::with_capacity`]、[`Vec::reserve`]或[`Vec::reserve_exact`]来保留一个特定的容量。例如，如果你知道一个向量将成长为至少有20个元素，这些函数可以使用一次分配立即提供一个至少有20个容量的向量，而一次推送一个项目将导致四次分配（对于4、8、16和32的容量）。
[**Example**](https://github.com/rust-lang/rust/pull/77990/commits/a7f2bb634308a5f05f2af716482b67ba43701681).

[`Vec::with_capacity`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.with_capacity
[`Vec::reserve`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.reserve
[`Vec::reserve_exact`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.reserve_exact

如果你知道一个向量的最大长度，上述函数也可以让你不分配多余的不必要的空间。同样，[`Vec::shrink_to_fit`]也可以用来尽量减少浪费的空间，但要注意它可能会引起重新分配。

[`Vec::shrink_to_fit`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.shrink_to_fit

## `String`

一个[`String`]包含堆分配的字节。`String`的表示和操作与`Vec<u8>`非常相似。许多与增长和容量有关的`Vec`方法与`String`有对应关系，如[`String::with_capacity`]。

[`String`]: https://doc.rust-lang.org/std/string/struct.String.html
[`String::with_capacity`]: https://doc.rust-lang.org/std/string/struct.String.html#method.with_capacity

来自[`smallstr`]箱子的`SmallString`类型与`SmallVec`类型类似。

[`smallstr`]: https://crates.io/crates/smallstr

来自 [`smartstring`] crate的 `String` 类型是`String`的插入式实现，
避免了少于三个 “字” 的`String`进行堆分配。 在64位平台上，这通常是任意少于24字节的字符串，
包括了所有含有23个或更少ASCII字符的字符串。
[**Example**](https://github.com/djc/topfew-rs/commit/803fd566e9b889b7ba452a2a294a3e4df76e6c4c).

[`smartstring`]: https://crates.io/crates/smartstring

请注意，`format!`宏产生一个`String`，这意味着它进行了分配。如果你能通过使用字符串文字来避免`format!`的调用，就能避免这种分配。
[**Example**](https://github.com/rust-lang/rust/pull/55905/commits/c6862992d947331cd6556f765f6efbde0a709cf9).

## 散列表

[`HashSet`]和[`HashMap`]是散列表。在分配方面，它们的表示和操作与`Vec`的表示和操作相似：它们有一个单一的连续的堆分配，存放键和值，随着表的增长，必要时重新分配。许多与增长和容量有关的`Vec`方法都有与`HashSet`/`HashMap`对应的方法，如[`HashSet::with_capacity`]。

[`HashSet`]: https://doc.rust-lang.org/std/collections/struct.HashSet.html
[`HashMap`]: https://doc.rust-lang.org/std/collections/struct.HashMap.html
[`HashSet::with_capacity`]: https://doc.rust-lang.org/std/collections/struct.HashSet.html#method.with_capacity

## `Cow`

有时候你有一些借用数据，比如`&str`，大部分是只读的，但偶尔需要修改。每次都克隆数据会很浪费。相反，你可以通过[`Cow`]类型使用 "write-on-clone "语义，它既可以表示借来的数据，也可以表示拥有的数据。

[`Cow`]: https://doc.rust-lang.org/std/borrow/enum.Cow.html

通常情况下，当从一个借来的值`x`开始时，你用`Cow::Borrowed(x)`把它包在一个`Cow`中。因为`Cow`实现了[`Deref`]，所以你可以直接在它所包含的数据上调用非修改的方法。如果需要修改，[`Cow::to_mut`]将获得一个对所拥有的值的可修改引用，必要时进行克隆。

[`Deref`]: https://doc.rust-lang.org/std/ops/trait.Deref.html
[`Cow::to_mut`]: https://doc.rust-lang.org/std/borrow/enum.Cow.html#method.to_mut

`Cow`的工作可能会很麻烦，但它往往是值得的。
[**Example 1**](https://github.com/rust-lang/rust/pull/37064/commits/b043e11de2eb2c60f7bfec5e15960f537b229e20),
[**Example 2**](https://github.com/rust-lang/rust/pull/50855/commits/ad471452ba6fbbf91ad566dc4bdf1033a7281811),
[**Example 3**](https://github.com/rust-lang/rust/pull/56336/commits/787959c20d062d396b97a5566e0a766d963af022),
[**Example 4**](https://github.com/rust-lang/rust/pull/68848/commits/67da45f5084f98eeb20cc6022d68788510dc832a).

## `clone`

在一个包含堆分配内存的值上调用[clone]通常会涉及额外的分配。例如，在一个非空的`Vec`上调用`clone`，需要对元素进行新的分配（但请注意，新`Vec`的容量可能与原`Vec`的容量不同）。例外的情况是`Rc`/`Arc`，`clone`的调用只是增加引用数。

[`clone`]: https://doc.rust-lang.org/std/clone/trait.Clone.html#tymethod.clone

[`clone_from`]是`clone`的替代方法。`a.clone_from(&b)`相当于`a = b.clone()`，但可以避免不必要的分配。例如，如果你想在一个现有的`Vec`之上克隆一个`Vec`，现有`Vec`的堆分配将尽可能地被重复使用，如下例所示。
```rust
let mut v1: Vec<u32> = Vec::with_capacity(99);
let v2: Vec<u32> = vec![1, 2, 3];
v1.clone_from(&v2); // v1's allocation is reused
assert_eq!(v1.capacity(), 99);
```
虽然`clone`通常会造成分配，但在很多情况下使用它是一件很合理的事情，往往可以使代码更简单。使用剖析数据来查看哪些`clone`调用是热门的，值得花力气去避免。

[`clone_from`]: https://doc.rust-lang.org/std/clone/trait.Clone.html#method.clone_from

有时，由于(a)程序员的错误，或(b)代码中的变化，使以前必要的`clone`调用变得不必要，Rust代码最终会包含不必要的`clone`调用。如果你看到一个似乎没有必要的热`clone`调用，有时可以简单地删除它。
[**Example 1**](https://github.com/rust-lang/rust/pull/37318/commits/e382267cfb9133ef12d59b66a2935ee45b546a61),
[**Example 2**](https://github.com/rust-lang/rust/pull/37705/commits/11c1126688bab32f76dbe1a973906c7586da143f),
[**Example 3**](https://github.com/rust-lang/rust/pull/64302/commits/36b37e22de92b584b9cf4464ed1d4ad317b798be).

## `to_owned`

`ToOwned::to_owned`]是为许多常见类型实现的。它从借来的数据中创建拥有的数据，通常是通过克隆的方式，因此经常会引起堆分配。例如，它可以用来从一个`&str`创建一个`String`。

[`ToOwned::to_owned`]: https://doc.rust-lang.org/std/borrow/trait.ToOwned.html#tymethod.to_owned

有时，可以通过在结构中存储对借入数据的引用而不是自有副本来避免`to_owned`调用。这需要在结构体上做终身注解，使代码复杂化，只有在分析和基准测试表明值得时才可以这样做。
[**Example**](https://github.com/rust-lang/rust/pull/50855/commits/6872377357dbbf373cfd2aae352cb74cfcc66f34).

## 重用集合

有时你需要分阶段建立一个集合，如`Vec`。通常情况下，通过修改一个`Vec`比建立多个`Vec`然后将它们组合起来更好。

例如，如果你有一个函数 "do_stuff"，产生一个 "Vec"，可能会被多次调用。
```rust
fn do_stuff(x: u32, y: u32) -> Vec<u32> {
    vec![x, y]
}
```
It might be better to instead modify a passed-in `Vec`:
```rust
fn do_stuff(x: u32, y: u32, vec: &mut Vec<u32>) {
    vec.push(x);
    vec.push(y);
}
```
有时，值得保留一个可以重复使用的 "主力"集合。例如，如果一个循环的每次迭代都需要一个`Vec`，你可以在循环外声明`Vec`，在循环体中使用它，然后在循环体结束时调用[`clear`]（清空`Vec`而不影响它的容量）。这避免了分配，但代价是掩盖了一个事实，即每次迭代对`Vec`的使用与其他迭代无关。
[**Example 1**](https://github.com/rust-lang/rust/pull/77990/commits/45faeb43aecdc98c9e3f2b24edf2ecc71f39d323),
[**Example 2**](https://github.com/rust-lang/rust/pull/51870/commits/b0c78120e3ecae5f4043781f7a3f79e2277293e7).

[`clear`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.clear

同样，有时值得在一个结构中保留一个 "主力 "集合，以便在一个或多个被重复调用的方法中重用。

## 使用其他分配器

另一个提高分配量大的Rust程序性能的选择是用一个替代分配器代替默认的（系统）分配器。确切的效果将取决于单个程序和选择的替代分配器。在不同的平台上，它也会有所不同，因为每个平台的系统分配器都有自己的优势和弱点。使用替代分配器也会影响二进制大小。

一个流行的替代分配器是[jemalloc]，可通过 [`jemallocator`]箱子。要使用它，请在你的`Cargo.toml`文件中添加一个依赖关系。
```toml
[dependencies]
jemallocator = "0.3.2"
```
然后在你的Rust代码中添加以下内容。
```rust,ignore
#[global_allocator]
static GLOBAL: jemallocator::Jemalloc = jemallocator::Jemalloc;
```
另一个可供选择的分配器是[mimalloc]，可通过[`mimalloc`]板房使用。

[jemalloc]: https://github.com/jemalloc/jemalloc
[`jemallocator`]: https://crates.io/crates/jemallocator
[mimalloc]: https://github.com/microsoft/mimalloc
[`mimalloc`]: https://docs.rs/mimalloc/0.1.22/mimalloc/
