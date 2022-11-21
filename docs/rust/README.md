# Rust

## 所有权原则

- Rust 中每一个值都被一个变量所拥有，该变量被称为值的所有者
- 一个值同时只能被一个变量所拥有，或者说一个值只能拥有一个所有者
- 当所有者(变量)离开作用域范围时，这个值将被丢弃(drop)

## 借用规则

- 同一时刻，你只能拥有要么一个可变引用, 要么任意多个不可变引用
- 引用必须总是有效的

## References

- [Rust 语言圣经](https://course.rs/about-book.html)
- [Programming Rust, 2nd Edition](https://book.douban.com/subject/34973905/)
- [Rusty Book](https://rusty.rs/about.html)