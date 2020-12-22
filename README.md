# perf-book—zh

Rust性能手册中文版

## 查看

英文版在线查看 [here](https://nnethercote.github.io/perf-book/).
中文版在线查看 [here](https://poly000.github.io/perf-book-zh/)

## 构建

本书使用 [`mdbook`](https://github.com/rust-lang/mdBook) 构建, mdbook可以用以下命令安装:
```
cargo install mdbook
```
运行以下命令以编译本书:
```
mdbook build
```
生成的文件将被保存在`\book`目录.

## 开发

To view the built book, run this command:
```
mdbook serve
```
This will launch a local web server to serve the book. View the built book by
navigating to `localhost:3000` in a web browser. While the web server is
running, the rendered book will automatically update if the book's files
change.

To test the code within the book, run this command:
```
mdbook test
```

