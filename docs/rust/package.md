# 包管理和模块

对于一个现代语言，模块管理机制和依赖管理工具的设计对语言的易用性和效率影响巨大。

## crate

Rust 的模块管理通过 `crate` 和 `module` 两个概念来组织。并提供了强大的包管理器 `cargo`。

> 和 npm 十分类似

crate 是独立的编译单元，编译后得到库(`--crate-type lib`)或者可执行文件(`--crate-type bin`)

同时 Rust 提供了一个[中央仓库](https://crates.io/)用来存放分享的 crates

> 这也是对于 Go（没有中央仓库） 的一个优势

同时 Cargo 通过 `Cargo.toml` 文件来管理当前 crate 依赖的其它 crate

> `Cargo.tom` 类似于 npm 的 package.json

```toml
[package]
# 当前 crate 名称
name = "fern_sim"
# 当前 crate 版本
version = "0.1.0"
# 作者
authors = ["You <you@example.com>"]
# 编译使用 rust 版本
edition = "2018"

[dependencies]
byteorder = "1.0.0"
num-iter = "0.1.32"
num-rational = "0.1.32"
num-traits = "0.1.32"
enum_primitive = "0.1.0"
# 通过 git 依赖 crate
image = { git = "https://github.com/Piston/image.git", rev =
"528f19c" }
# 通过相对路径依赖 crate
image = { path = "vendor/image" }
```

可以通过 `cargo build --verbose` 查看依赖的下载和编译信息

构建时 cargo 会去中央仓库下载依赖的 crate 的对应版本的源码包

> 另外也可以和 Go 一样指定一个 git 地址或者一个本地路径。依赖于 github 仓库，建议先 fork，然后依赖于自己的 fork 库，以避免主库出现删分支、tag 之类的行为导致构建失败

Rust 的版本语义：

- `0.0.x` 版本代表不稳定，会精确下载对应版本
- `0.x.x` 版本，会优先下载 0.x 下的最新版本
- `x.x.x` 版本，会优先下载 x. 下的最新版本
- `*` 会优先下载 crate 的最新版本

类似于 npm 的 package-lock.json，cargo 也提供了 `Cargo.lock` 文档用于锁定版本

> 使用 git 分支依赖时也会 lock commit version

> 根据经验，executable 建议提交 Cargo.lock 文件到 git 仓库，library 不建议提交 Cargo.lock 文件

Cargo 构建的入口文件默认是基于约定的，在构建可执行程序时，入口文件是 `src/main.rs`；在构建库时，入口文件是 `src/lib.rs`。

> cargo build 会根据存在哪个入口文件来决定构建的类型

[Cargo.toml 格式文档](https://doc.rust-lang.org/cargo/reference/manifest.html)

此外，在构建库时，也可以同时构建额外的可执行程序（使用库且 crate 名称相同），放在 `src/bin` 目录下(文件或者目录)，以下就是附带了两个可执行程序，

> cargo build 会先构建库，然后构建额外的可执行程序

> 当然给可执行程序使用独立的工程也可以

```
fern_sim/
├── Cargo.toml
└── src/
 └── bin/
    ├── efern.rs
    └── draw_fern/
        ├── main.rs
        └── draw.rs
```

## module

如果说 crates 是不同项目之间的代码共享机制，那么 modules 就是项目内的代码组织机制。

```rust
mod module1 {
    pub fn say_hello() {
        //..
    }

    pub mod module2 {
        pub struct Item {
            a: i32,
            b: i32
        }
    }
}

// 调用模块的方法
module1::say_hello();
```

一个文件可以定义多个 module，也可以把多个 module 分离成独立的文件。比如下面就是一个 module 独立的文件结构：

```
fern_sim/
├── Cargo.toml
└── src/
 ├── main.rs
 ├── spores.rs
 └── plant_structures/
 ├── mod.rs
 ├── leaves.rs
 ├── roots.rs
 └── stems.rs
```

通过 `mod module1;` 告诉编译器 module1 模块分离在独立的文件，叫做 `module1.rs` 或者 `module1/mod.rs`

```rust
/// main.rs
// 引用 module，类似于 c++ 中的 include，
mod module1;
module1::say_hello();

/// spores.rs
pub fn say_hello() {
    //..
}
```

> 一个分离的 module 可能是个同名的文件，或者是个同名的目录下的 `mod.rs` 文件

> 分离后的 module 中不再需要显示用 mod 声明包裹，文件本身就是 module 声明

> Rust 是以 crate 作为编译单元，即使把 module 分离成独立的文件，编译时还是会重新编译 crate 所有 module 文件

和其它语言一样，rust 提供了 `use` 关键字来简化 import module 或者 module 中的 item

!> Item 包括函数、类型、变量、nested module

```rust
// 绝对路径
// import 其它 crate 的模块
use std::collections::HashMap
// import 当前 crate 的模块
use crate::moduleName::submoduleName;

// 相对路径
// import 兄弟模块
use super::AminoAcid
// import 子模块
use submodule;
// 或者
use self::AminoAcid;

// 通过 {} 和 * 简写
// import 模块下多个 item
use std::collections::{HashMap, HashSet};
// import 模块和模块下的 item
use std::fs::{self, File}; 
// import 模块下所有 item
use std::io::prelude::*;
// import 父模块下所有 item
use super::*;

// 使用别名
// 别名 item
use std::io::Result as IOResult;
// 别名 module
use std::io as ioUtil;

// 存在子模块和外部crate同名的情况
// 通过 :: 开头引用外部模块
use ::image::Pixels;
// 通过 self 指定应用子模块
use self::image::Pixels;
```

> use 语法中的 `super` 和 `self` 类似于 `.` `..`，而 `crate` 类似于 `~`

父 module 的所有 Item 默认对子 module 可访问，但是子 module 不会自动继承父 module 的名称，还是需要使用 `use super::ItemName` 的方式

为了简化使用，Rust 默认会 import 常用的模块和 item，本质上就是自动在每个 module 中使用 `use std::preclude::v1::*;`，而 `std::preclude::v1` 模块中通过使用 `pub use crate::module` 的方式导出一系列的其它核心模块和 Item。preclude 具体看参考[文档](https://doc.rust-lang.org/std/prelude/v1/index.html)

## 访问控制

Rust 通过 module 来实现访问控制，一个元素要么在该模块可访问，要么不可访问。

当前模块的 Item 默认在当前模块和子模块可访问。

要让父模块或者其它模块可访问，需要使用 `pub` 标记 module 和 Item

`pub` 表示能够被 module 外部访问

`pub(crate)` 表示能够被当前 crate 的所有 module 访问

`pub(super)` 表示能够被父 module 访问

`pub(in <path>)` 表示能够被祖先 module 访问，path 必须是当前模块的祖先模块路径

struct 类型的字段默认是私有的，即本模块和子模块可见。要想其它模块可见，除了 struct 需要标记 pub，外部可见的字段也需要标记 pub

> 内部可见可以减少大量的 getter、setter 代码

> 体现 的思想，访问权限都是针对 module 设置的，而非其它语言的 `private` `packaged` `protected` `public` 几种访问范围

```rust
pub struct Fern {
 pub roots: RootSet,
 pub stems: StemSet
}
```

## Cargo

```bash
# 构建 debug 版本可执行文件，包含调试信息和边界检查，包比较大
cargo build
# 构建 release 版本可执行文件，没有多余的调试信息和边界检查，包小性能高，但是编译时间比较常
cargo build --release
# 构建库
cargo build --crate-type lib
# 显示构建可执行文件
cargo build --crate-type bin
# 查看构建过程的详细信息，可以看到调用 rustc 的参数
cargo build --verbose
# 升级当前 crate 的 rust edition 版本到最新版本
cargo fix
# 升级依赖版本
cargo update

# 打包 crate，会包含所有的源码、Cargo.toml 文件，生成 .crate 文件
cargo package
# 登录 cargo 中央仓库，需要先注册账号
cargo login <apikey>
# 推送 .crate 文件到中央仓库
cargo publish
```

可以在 `Cargo.toml` 文件 profile section 中对于不同场景的构建指定参数

```toml
[profile.release]
debug = true # 让 cargo build --release 构建保留调试信息

[profile.debug]
# 针对 cargo build 的配置

[profile.test]
# 针对 cargo test 的配置
```

## Workspace

Cargo 也提供了类似于 Maven 的多模块机制，叫做 Workspace

Workspace 包含了多个 crates，这个 crate 共享 workspace 的 build 目录(`target/`)和 cargo.lock 文件。

> 即使在 crate 目录下执行 cargo build 构建也是共享 workspace 的 build 目录的

```rust
fernsoft/
├── .git/...
├── Cargo.toml
├── Cargo.lock
├── fern_sim/
│ ├── Cargo.toml
│ ├── Cargo.lock
│ ├── src/...
│ └── target/...
├── fern_img/
│ ├── Cargo.toml
│ ├── Cargo.lock
│ ├── src/...
│ └── target/...
└── fern_video/
 ├── Cargo.toml
 ├── Cargo.lock
 ├── src/...
 └── target/...
```

Workspace 的 Cargo.toml:

```toml
[workspace]
members = ["fern_sim", "fern_img", "fern_video"]
```

命令行：

```bash
# 构建 workspace 下所有 crate
cargo build --workspace
```


## References

- [中央仓库](https://crates.io/)
- [包搜索](https://lib.rs/)
- [Cargo 文档](https://doc.rust-lang.org/stable/cargo/)