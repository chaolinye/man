# Rust unsafe

Rust 的安全检查过强，导致某些情况无法实现，这时可以使用 `unsafe` 来解决。

> 使用 unsafe 代表你要替代编译器的部分职责对 unsafe 代码的正确性负责

unsafe 能赋予我们 5 种超能力，这些能力在安全的 Rust 代码中是无法获取的：

- 解引用裸指针
- 调用一个 unsafe 或外部的函数
- 访问或修改一个可变的静态变量
- 实现一个 unsafe 特征
- 访问 union 中的字段

## 解引用裸指针

三种类似指针的概念：引用、智能指针和裸指针。与前两者不同，裸指针：

- 可以绕过 Rust 的借用规则，可以同时拥有一个数据的可变、不可变指针，甚至还能拥有多个可变的指针
- 并不能保证指向合法的内存
- 可以是 null
- 没有实现任何自动的回收 (drop)

裸指针长这样: `*const T` 和 `*mut T`，它们分别代表了不可变和可变。

> 创建裸指针是安全的行为，而解引用裸指针才是不安全的行为

```rust
fn main() {
    let mut num = 5;

    let r1 = &num as *const i32;

    unsafe {
        println!("r1 is: {}", *r1);
    }
}
```

!> unsafe 语句块的范围一定要尽可能的小

其它获得裸指针的方式:

```rust
// 基于内存地址
let address = 0x012345usize;
let r = address as *const i32;

// 基于只能指针
let a: Box<i32> = Box::new(10);
// 需要先解引用a
let b: *const i32 = &*a;
// 使用 into_raw 来创建
let c: *const i32 = Box::into_raw(a);
```

> 裸指针的一个重要用途就是跟 C 语言的代码进行交互( FFI )

## 调用 unsafe 函数或方法

unsafe 函数是为了告诉调用者：当调用此函数时，你需要注意它的相关需求，因为 Rust 无法担保调用者在使用该函数时能满足它所需的一切需求。

强制调用者加上 unsafe 语句块。

```rust
unsafe fn dangerous() {}

fn main() {
    unsafe {
        dangerous();
    }
}
```

!> 使用 unsafe 声明的函数时，一定要看看相关的文档，确定自己没有遗漏什么。


在 unsafe 函数体中使用 unsafe 语句块是多余的行为

同时，一个函数包含了 unsafe 代码不代表我们需要将整个函数都定义为 `unsafe fn`。只要这个函数对外是 safe 的，就不要标识 `unsafe fn`。

## 访问或修改一个可变的静态变量

静态变量为了保证其多线程安全性，默认是不能修改的。如果要修改，除了使用线程安全的手段(Atomic, Mutex)，还是可以使用 `unsafe` 直接修改，由程序员来保证安全性。

## 实现 unsafe 特征

Send 特征标记为 unsafe 是因为 Rust 无法验证我们的类型是否能在线程间安全的传递，因此就需要通过 unsafe 来告诉编译器，它无需操心，剩下的交给我们自己来处理。

```rust
unsafe trait Foo {
    // 方法列表
}

unsafe impl Foo for i32 {
    // 实现相应的方法
}

fn main() {}
```

## 访问 union 中的字段

`union` 类型主要用于跟 C 代码进行交互。

访问 union 的字段是不安全的，因为 Rust 无法保证当前存储在 union 实例中的数据类型。

```rust
#[repr(C)]
union MyUnion {
    f1: u32,
    f2: f32,
}
```

## References

- [Unsafe Rust](https://course.rs/advance/unsafe/intro.html)