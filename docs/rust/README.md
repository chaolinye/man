# Rust

## 所有权原则

- Rust 中每一个值都被一个变量所拥有，该变量被称为值的所有者
- 一个值同时只能被一个变量所拥有，或者说一个值只能拥有一个所有者
- 当所有者(变量)离开作用域范围时，这个值将被丢弃(drop)

## 借用规则

- 同一时刻，你只能拥有要么一个可变引用, 要么任意多个不可变引用
- 引用必须总是有效的

## 安装

[文档](https://course.rs/first-try/installation.html)

核心命令行工具：

- `rustup`: rust 版本管理器。
- `cargo`: 包管理器（等同于 npm）。最常用的命令行工具
- `rustc`: rust 编译器。一般不直接使用，而是通过 cargo 调用。
- `rustdoc`: 文档构建工具。一般也是通过 cargo 调用。

常用命令：

```bash
# 构建新工程
cargo new hello_world

# 构建+运行
cargo run

# 清理生成物
cargo clean

# 构建 debug 版本可执行文件，包含调试信息和边界检查，包比较大
cargo build
# 构建 release 版本可执行文件，没有多余的调试信息和边界检查，包小性能高，但是编译时间比较常
cargo build --release

# 构建库
cargo build --crate-type lib
# 显示构建可执行文件
cargo build --crate-type bin

# 查看构建过程的详细信息，可以看到调用 rustc 的参数
cargo build -v

# 升级当前 crate 的 rust edition 版本到最新版本
cargo fix
# 升级依赖版本
cargo update

# 离线查看标准库文档
rustup doc --std

# 打包 crate，会包含所有的源码、Cargo.toml 文件，生成 .crate 文件
cargo package
# 登录 cargo 中央仓库，需要先注册账号
cargo login <apikey>
# 推送 .crate 文件到中央仓库
cargo publish
```

## References

- [Rust 语言圣经](https://course.rs/about-book.html)
- [Rust 官方文档](https://www.rust-lang.org/learn)
- [Programming Rust, 2nd Edition](https://book.douban.com/subject/34973905/)
- [Rusty Book](https://rusty.rs/about.html)
- [Rust Reference](https://doc.rust-lang.org/reference/)
- [中央仓库](https://crates.io/)
- [包搜索](https://lib.rs/)
- [文档](https://docs.rs/)
- [Cheat Sheet](https://cheats.rs/)