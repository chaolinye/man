# Rust 语言特性

## Attritues

Attritues 是 Rust 中给编译器的说明或者建议，可以装饰所有的 Item。

[文档](https://doc.rust-lang.org/reference/attributes.html)

语法格式:

- 内部语法：`#![MetaItem]`(作用于父元素)
- 外部语法：`#[MetaItem]`(作用于下方元素)

MetaItem 的语法：

- `MetaWord`
- `MetaWord = "str"`
- `MetaWord(param1, param2)`
- `MetaWord(namedparam1 = "str1", namedparam2 = "str2")`

```rust
// 作用于 crate，指定 crate 的类型
#![crate_type = "lib"]

// 标记该函数为测试函数，cargo test 执行
#[test]
fn test_foo() {
    /* ... */
}

// 条件编译
#[cfg(target_os = "linux")]
mod bar {
    /* ... */
}

// 消除编译器的某些警告或者错误
#[allow(non_camel_case_types)]
type int8_t = i8;
```

## Test

Rust 提供多种编写测试的方法；

- `#[test]` attribute
    > 可以在 module 中编写，获取足够的访问权限

- `#[cfg(test)]` test module

    ```rust
    #[cfg(test)] // include this module only when testing
    mod tests {
        fn roughly_equal(a: f64, b: f64) -> bool {
            (a - b).abs() < 1e-6
        }
        #[test]
        fn trig_works() {
            use std::f64::consts::PI;
            assert!(roughly_equal(PI.sin(), 0.0));
        }
    }
    ```

- `tests/xx.rs` 下的集成测试

    ```rust
    // tests/unfurl.rs - Fiddleheads unfurl in sunlight
    use fern_sim::Terrarium;
    use std::time::Duration;

    #[test]
    fn test_fiddlehead_unfurling() {
    let mut world =
    Terrarium::load("tests/unfurl_files/fiddlehead.tm");
    assert!(world.fern(0).is_furled());
    let one_hour = Duration::from_secs(60 * 60);
    world.apply_sunlight(one_hour);
    assert!(world.fern(0).is_fully_unfurled());
    }
    ```

- 文档注释的代码块

提供的断言方法:

- `assert!` 宏
- `assert_eq!` 宏
- `#[should_panic]` attribute

命令行:

```bash
# 执行所有 Tests
cargo test
# 执行名称中包含 math 的 Tests
cargo test math
# 指定测试线程,第一个 -- 是指后面的参数不是给 cargo 的参数，而是 test 的参数
cargo test -- --test-thread 1
# 显示成功的 Tests 的输出
cargo test -- --no-capture
```

> cargo build 会跳过测试代码

## Document

> 当 crate pushlish 到 `crate.io` 后，其文档会自动部署到 `docs.rs` 中

命令行:

```bash
# 生成文档，--no-deps（只生成当前 crate 的文档），--open（在浏览器打开文档）
cargo doc --no-deps --open

# 文档单元测试
# 底层执行的是 rustdoc --test
cargo test
```

文档注释 `///` 和 `//!`

也可以使用等价的 Attribute `#[doc]` 和 `#![doc]`

Rust 文档注释使用的是 Markdown 格式，除了支持 Markdown 的基础语法，还有一些自定义的语法，比如使用 Rust Item 的路径作为 link，格式:

```
[`pathOfItem`]
```

还可以使用 `#[doc(alias="route")` 给 Item 起别名用于 path link。

另外，文档注释中的代码块还可以编写单元测试

```rust
use std::ops::Range;
/// Return true if two ranges overlap.
///
/// assert_eq!(ranges::overlap(0..7, 3..10), true);
/// assert_eq!(ranges::overlap(1..5, 101..105), false);
///
/// If either range is empty, they don't count as overlapping.
///
/// assert_eq!(ranges::overlap(0..0, 0..10), false);
///
pub fn overlap(r1: Range<usize>, r2: Range<usize>) -> bool {
 r1.start < r1.end && r2.start < r2.end &&
```

本质上，每个代码块都会抽取出独立的文件，类似于

```rust
use ranges;
fn main() {
 assert_eq!(ranges::overlap(0..7, 3..10), true);
 assert_eq!(ranges::overlap(1..5, 101..105), false);
}
```

> 因此文档测试不是在模块内的，对于模块内的访问权限需要关注下

如果不想在文档中显示代码块中的某些代码，可以在每行的开头加上 `#`.

不想执行的文档测试，代码块语法标识成 `no_run`。当然如果标记成其它语法，也会被忽略，比如 `cpp` `sh` 等