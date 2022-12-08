# Rust 语言特性

Rust 的定位是一门系统编程语言，主打性能和安全，适用于资源受限场景的编程。

Rust 的优势是 `speed`，`concurrency` 和 `safety`。

分别对应着 Rust 的三个设计目标：`零抽象开销`，`可靠并发`，`内存安全`。

零抽象开销是从 C++ 中借鉴的设计。至于可靠并发和内存安全是通过所有权（ownership）、移动（move）、借用（borrow）、生命周期（lifetime）机制来实现。

对于内存管理，主要有两种方式：

1. 手动管理（c/c++）。优点：及时清理内存，性能好；缺点：无法保证内存安全，容易出现内存泄漏。
2. GC（java/python/go...）。优点：内存安全；缺点：占用内存多；GC 时 stop the world，无法用于时延要求高的场景。

> 对于手动管理的内存泄漏问题，可以用 C++ 中的 RAII 编程模式来规避。但是 dangling pointers 等内存安全问题还是无法保证。

C/C++ 中最令人害怕的就是 undefined behavior。而 Rust 中解决了这些问题，常见的 dangling pointers、double free、null pointer deferences 能在编译期发现，数组越界问题通过编译期和运行期检查一起保证。

> 在 Rust 中，如果代码能够通过编译，就不会有 undefined behavior。

## 代码风格

struct、enum、trait 用大写字母开头的驼峰式

变量、方法、函数 用小写字母+下划线


## 变量

Rust 通过 `let` 关键字声明变量，**默认是不可变变量**。声明可变变量，需要添加 `mut` 关键字。

```rust
let cv: u32 = 10;
let mut v: u32 = 20;
```

!> Rust 变量之间赋值，如果类型实现了 `Copy` Trait(特征)，就执行拷贝，否则默认执行 `move`；如果要深拷贝，需要显式调用类型的 `clone` 方法（需要实现 `Clone` Trait）。  
（和 C++ 相反，C++ 默认是 Copy/深拷贝，move 反而需要显示使用 `std::move` 函数）

基本类型基本都是 `Copy`，通过 `struct` 和 `enum` 自定义的类型不会默认实现 `Copy` Trait, 如果所有字段都是 `Copy` 类型，也可以通过 `#[derive(Copy, Clone)]` 来实现 `Copy` Trait。（如果有字段不是 `Copy` 类型，编译器会报错）

> `&T` 引用也实现了 `Copy` Trait，但是 `&mut T` 不是

> `Copy` Trait 要求实现其的类型必须要实现 `Clone` Trait

> 一个值只拥有栈上数据的，都应该是 `Copy`

## 类型

由于 Rust 是静态类型语言，所以要写出函数参数、返回值、struct 字段以及其它结构体的类型。

为了提高效率，Rust 提供了两个特性：

1. 类型推断。Rust 可以根据你写出的类型推断其余大部分值的类型。

    > Rust 的类型推断比 C++ 的 auto 更智能

    比如根据后续的代码推断 Vec 的类型

    ```rust
    // 根据返回值推断出 v 的类型
    fn build_vector() -> Vec<i16> {
        let mut v = Vec::new();
        v.push(10);
        v.push(20);
        v
    }
    ```

2. 泛型函数。

    > 大部分静态语言都有这个特性（C++ 的最强大，Java 的只是擦除式泛型），动态语言中变量就可以用于任何类型，不需要泛型。

### 数字类型

- 有符号整型: `i8` `i16` `i32` `i64` `i128`

- 无符号整型: `u8` `u16` `u32` `u64` `u128`

- 机器字(内存地址的宽度 32 or 64 bit): `isize` `usize`

    > usize 能代表内存的最大值，所以很适合用于表达集合的长度和索引，比如Rust 要求数组索引必须是 usize 类型、

- 浮点数: `f32` `f64`

数字字面量可以通过加上类型后缀表示具体的类型，比如 `42u8` `1729usize`。

> 如果没有类型后缀，Rust 会根据上下文来推断其适合的类型。如果存在多个适合的类型，其中存在 `i32` 则取 `i32`，否则编译报错。如果推断不出来，也会报错。

另外，加上前缀 `0x` `0o` `0b` 可以表示十进制（`0xcafe`）、八进制(`0o106`)、二进制(`0b0010_1010`)。还可以用下划线来分割数字，增强可读性，比如 `4_296_967_295`



可以使用 `as` 操作符，进行转换,

```rust
// 无损转换
assert_eq!(10_i8 as i16, 10_u16);
// 截断
assert_eq!(1000_i16 as u8, 232_u8);
```

!> Rust 中不会隐式转换数字类型，即使时从小到大。

和其它语言不同，Rust 的基础类型可以拥有方法。比如

```rust
assert_eq!(2_u16.pow(4), 16);
assert_eq!((-4_i32).abs(), 4);
assert_eq!(0b101101_u8.count_ones(), 4);

assert_eq!(5f32.sqrt() * 5f32.sqrt(), 5.);
assert_eq!((-1.01f64).floor(), -2.0);
```

!> Rust 有个重要的特性：可以通过 trait 给已有的类型添加方法，包括基本类型。因此我们也可以给基础类型加自定义方法。

当整数运算溢出时，debug build 会 panic，release build 会 wraps round。

如果默认的行为不行，也可以通过调用方法的方式来控制行为

```rust
// checked_xxx 方法返回 Option，溢出时是 None
assert_eq!(10_u8.checked_add(20), Some(30));
assert_eq!(10_u8.checked_add(20), Some(30));

// wrapping_xxx 溢出时 wrap
assert_eq!(500_i16.wrapping_mul(500), -12144);

// saturating_xxx 溢出时取最大值或者最小值
assert_eq!(32760_i16.saturating_add(10), 32767);
assert_eq!((-32760_i16).saturating_sub(10), -32768);

// overflowing_xx 返回 tuple，溢出时 wrap，tuple 的第二个值为 true
assert_eq!(255_u8.overflowing_sub(2), (253, false));
assert_eq!(255_u8.overflowing_add(2), (1, true));
```


### bool

和其它语言一致，bool 类型只有两个值 `true` `false`。

> bool 类型占用一个字节的内存

bool 类型能转成整型

```rust
assert_eq!(false as i32, 0);
assert_eq!(true as i32, 1);
```

!> 但是，其它所有类型都不能转成 bool 类型(隐式和显式都不行)。

!> bool 值要么是 bool 字面量，要么是布尔表达式产生。

### 字符

Rust 的 char 类型是个 Unicode 字符，占用 32 bit。

`*` `\xHH`(ASCII 字符) `\u{HHHHHH}`(任何 Unicode 字符)

!> Rust 的 char 和 string 采用的编码不一样，string 采用的是 UTF8（1-4 byte 可变编码）

> 此外，用 `b'A'` 能表示一个字节的 ASCII 字符字面量，本质上是 u8 类型。

```rust
assert_eq!('*' as i32, 42);


assert_eq!('*'.is_alphabetic(), false);
assert_eq!('β'.is_alphabetic(), true);
assert_eq!('8'.to_digit(10), Some(8));
assert_eq!('a'.len_utf8(), 1);
assert_eq!(std::char::from_digit(2, 10), Some('2'));
```

### 元组

tuple 由各种类型的值组成。

```rust
let tuple_val = ("test", 1985); // (&str, i32)
```

> 和数组一样，tuple 也是存放在栈内存中

和数组不一样，tuple 的值可以是不同的类型，其次只允许用常量作为索引。

tuple 常用于函数返回多个值。

```rust
fn split_at(&self, mid: usize) -> (&str, &str);

let text = "I see the eigenvalue in thine eye";
let temp = text.split_at(21);
let head = temp.0;
let tail = temp.1;
assert_eq!(head, "I see the eigenvalue ");
assert_eq!(tail, "in thine eye");
```

tuple 可用 pattern-matching 语法进行赋值

```rust
let text = "I see the eigenvalue in thine eye";
let (head, tail) = text.split_at(21);
assert_eq!(head, "I see the eigenvalue ");
assert_eq!(tail, "in thine eye");
```

!> 此外，如果一个函数不指定返回类型，默认就是返回 `()` 类型。即 `fn swap<T>(x: &mut T, y: &mut T);` 等价于 `fn swap<T>(x: &mut T, y: &mut T) -> ();`

### 指针类型

Rust 中由几种表示内存地址的类型（references、boxes、unsafe pointers）

#### 引用

通过 `&x` 表示式，可以获取 x 的一个引用，这个过程叫做借用(borrow)

> `&` 符号类似于 C++ 的取址运算符，但得到的不是原始指针，而是引用

引用不会为 null。

> Rust 中也没有 null，而是用 Option 来表示。Java 中也有 Option，但是 Rust 的 pattern-matching 和各种语法糖，让 Option 的使用十分的流畅

> 传递引用会导致 dangling pointers 的问题，Rust 通过引入引用的生命周期概念来让编译器发现这些问题。

引用有两种: 

- `&T`: 不可变、共享的引用。类似于 C++ 中的 `const T*`
- `&mut T`: 可变、 排他的引用。类似于 C++ 中的 `T*`


#### Boxes

Rust 的值默认分配在栈上，要想分配到堆上，最简单的方式就是使用 `Box::new`

```rust
let t = (12, "eggs");
let b = Box::new(t); // allocate a tuple in the heap
```

Box 运用了 RAII 的方式（实现了 `Drop` trait），销毁 Box 时会调用 drop 方法，释放其拥有的堆内存。

Box 默认只能 move。类似于 C++ 中的 `std::unique_ptr`

#### Raw Pointers

原始指针类型:

- `*mut T`

- `*const T`

原始指针类型可以通过引用 `as` 转换得到，但是原始指针的只能在 `unsafe` 块中解引用


### Arrays，Vectors，Slices

Rust 中有 3 种表示一系列值的类型: 

- 数组：`[T; N]`

    > 栈分配

    ```rust
    // 初始化数组
    let lazy_caterer: [u32; 6] = [1, 2, 4, 7, 11, 16];
    let taxonomy = ["Animalia", "Arthropoda", "Insecta"];

    assert_eq!(lazy_caterer[3], 7);
    assert_eq!(taxonomy.len(), 3);

    // 指定初始值和大小
    let mut sieve = [true; 10000];

    // 数组排序
    let mut chaos = [3, 5, 4, 1, 2];
    chaos.sort();
    assert_eq!(chaos, [1, 2, 3, 4, 5]);
    ```

- 向量: `Vec<T>`

    [文档](https://doc.rust-lang.org/std/vec/struct.Vec.html)

    > 元素在堆上动态分配。栈上只有堆内存第一个元素指针、容量、当前元素个数

    ```rust
    // 使用 new 关联函数创建
    let v: Vec<u8> = Vec::new();

    // 创建指定容量的 Vec
    let v2: Vec<u8> = Vec::with_capacity(2);
    assert_eq!(v2.len(), 0);
    assert_eq!(v2.capacity(), 2);

    // vec! 宏简化向量的初始化，等同于先 Vec::new 创建，然后 push 元素
    let mut primes = vec![2, 3, 5, 7];
    // product 是元素的乘积
    assert_eq!(primes.iter().product::<i32>(), 210);
    // 动态增加元素
    primes.push(11);
    primes.push(13);
    assert_eq!(primes.iter().product::<i32>(), 30030);

    // 指定初始值和个数
    let mut primes2 = vec![0, rows * cols]

    // 通过 iterator 的 collect 方法得到向量
    let v3: Vec<i32> = (0..5).collect();
    assert_eq!(v3, [0, 1, 2, 3, 4]); 

    // for..in 遍历
    for item in &v3 {
        println!("{}", item)
    }

    for index in 0..v3.len() {
        println!("{}:{}", index, v3[index]);
    }
    ```

- 切片: `&[T]` `&mut [T]`

    切片写作 `[T]`，表示数组或向量的一个范围。由于切片可以是任意长度，因此不能直接保存在变量中，也不能作为函数参数传递。**切片永远只能按引用传递**，因此也常把切片的引用直接称为切片。

    切片的引用是个胖指针(fat pointer)，包含切片第一个元素的指针和元素的个数。

    > 常见的字符串字面量的类型 `&str` 也是个胖指针

    切片引用非常适合操作一串同类数据的函数，无论这串数据存储爱数组、向量，抑或栈还是堆上。

    !> 定义一个需要处理一串同类数据的函数，用切片引用作为入参类型是个很不错的选择

    ```rust
    let v: Vec<f64> = vec![0.0, 0.707, 1.0, 0.707];
    let a: [f64; 4] = [0.0, -0.707, -1.0, -0.707];
    // 数组和向量的引用能够自动解引用成切片的引用
    let sv: &[f64] = &v;
    let sa: &[f64] = &a;
    let sv1: &[f64] = &v[0..2]
    ```

    > 数组、向量的引用实现了 `Deref` Trait，会隐式得调用 deref 方法转换成切片引用。因此数组、向量能使用切片引用的所有方法。


### 字符串类型

Rust 的字符串使用的是 UTF8 编码。

字面量是 `&str` 类型。

> &str 是个胖指针，包含实际数据的地址及其长度

> 编译时就会把字面量存放在程序的 data segment 中，&str 存储的就是其地址

```rust
let speech = "\"Ouch!\" said the well.\n";
// 字面量可以跨多行，每行后面的换行符和第二行的空白符会保留
println!("In the room the women come and go,
    Singing of Mount Abora");

// 跨多行，不会保留换行符和第二行的空白符
println!("It was a bright, cold day in April, and \
    there were four of us—\
    more or less.");

// raw string，不存在转义字符，r后面的 # 的个数不定，也可以没有，根据文本中是否存在 "### 的内存来决定个数
println!(r###"
    This raw string started with 'r###"'.
    Therefore it does not end until we reach a quote mark ('"')
    followed immediately by three pound signs ('###'):
"###);
```


Byte 字符串就是带前缀 b 的字符串字面量，类型是 `&[u8;N]`，非 UTF8 编码。

Byte 字符串字面量和普通的字符串字面量一样，可以使用转义符，也有 raw string（前缀是 `br"`）,但是只能是 ASCII 和 `\xHH` 转义序列

```rust
let method = b"GET";
assert_eq!(method, &[b'G', b'E', b'T']);
```

本质上，Rust 中的字符串内部就是 `Vec<u8>` 类型，只是保证了存储的是格式完好的 UTF8 编码。所以 `&str` 本质上就是 `&[u8]` 切片

Rust 的字符串类型 String，和 Vec 很像，操作也很像。

[文档](https://doc.rust-lang.org/std/string/struct.String.html)

同样 String 实现了 Deref Trait，其引用可以隐式转换成 `&str`

```rust
// 根据 &str 得到
let error_message = "too many pets".to_string();
let hello = String::from("Hello, world!");

// format! 宏生成
let str = format!("{}°{:02}′{:02}″N", 24, 5, 23), "24°05′23″N".to_string()

// push
let mut hello = String::from("Hello, ");
hello.push('w');
hello.push_str("orld!");

// 拼接多个字符串
let bits = vec!["veni", "vidi", "vici"];
assert_eq!(bits.concat(), "venividivici");
assert_eq!(bits.join(", "), "veni, vidi, vici");
```

字符串支持 `==` 等关系运算符

```rust
assert!("ONE".to_lowercase() == "one");

assert!("peanut".contains("nut"));
assert_eq!(" _ ".replace(" ", "■"), "■_■");
assert_eq!(" clean\n".trim(), "clean");
for word in "veni, vidi, vici".split(", ") {
    assert!(word.starts_with("v"));
}
```

Rust 保证字符串是有效的 UTF-8。但是再与其它系统操作是，需要处理不是 UTF8 的字符串。可以使用以下的方法：

- 对于 Unicode 文本，使用 `String` 和 `&str`。
- 处理文件名时，使用 `std::path::PathBuf` 和 `&Path`。
- 处理根本不是字符数据的二进制数据时，使用 `Vec<u8>` 和 `&[u8]`。
- 处理以操作系统原生形式表示的环境变量名和命令行参数时，使用 `OsString` 和 `&OsStr`。
- 与使用空字符结尾字符串的 C 库互操作时，使用 `std::ffi::CString` 和 `&CStr`。

### 类型转换

#### 点操作符

方法调用的点操作符看起来简单，实际上非常不简单，它在调用时，会发生很多魔法般的类型转换，例如：自动引用、自动解引用，强制类型转换直到类型能匹配等。

假设有一个方法 foo，它有一个接收器(接收器就是 self、&self、&mut self 参数)。如果调用 value.foo()，编译器在调用 foo 之前，需要决定到底使用哪个 Self 类型来调用。现在假设 value 拥有类型 T。
 
1. 编译器检查是否可以直接调用 `T::foo(value)`，称之为值方法调用
2. 如果不行(例如方法类型错误或者特征没有针对 Self 进行实现，上文提到过特征不能进行强制转换)，那么编译器会尝试增加自动引用，例如会尝试以下调用： `<&T>::foo(value)` 和 `<&mut T>::foo(value)`(当然前提是 value 本身得是 mut)，称之为引用方法调用
3. 若上面两个方法依然不工作，编译器会试着解引用 T ，然后再进行尝试。这里使用了 `Deref` 特征 —— 若 `T: Deref<Target = U>` (T 可以被解引用为 U)，那么编译器会使用 U 类型进行上述两步的尝试，称之为解引用方法调用
4. 若 T 不能被解引用，且 T 是一个定长类型(在编译器类型长度是已知的)，那么编译器也会尝试将 T 从定长类型转为不定长类型，例如将 [i32; 2] 转为 [i32]
5. 若还是不行，那编译器就报错

> 另外 `[]` 操作符其实是个语法糖，本质上是调用 `Index` 特征的 `index()` 方法，即 `array[0]` 等价于 `array.index(0)`，所以其也适用于点操作符的规则

## 所用权、引用、生命周期

对于内存管理，几乎所有编程语言的处理方式可以归为两派：

1. 安全优先派使用垃圾收集器来管理内存，让 GC 来自动释放不可达对象。这种方式可以解决内存泄漏和悬空指针等内存安全问题。但是 GC 会导致 stop the world，不适用于时延要求高的场景；同时 GC 的存在让对象的释放延后了，程序会占用更大的内存；GC 本身的管理也会耗费一定的 CPU 和内存资源。

    > 几乎所有的现代语言都是这个流派。包括 Java、Go、Javascript、Python等。

1. 控制优先派让程序自己管理内存。自己管理内存，容易出现悬空指针等内存安全问题。

    > 代表语言是 C 和 C++

Rust 的目标是性能和安全。

Rust 通过给所有权+引用生命周期的方式打破了内存管理安全和性能无法兼得的僵局。

> 所有权本质上就是 RAII 思想

Rust 语言内置了所有权机制，并通过编译器检查来保证。

### 所有权原则

- Rust 中每一个值都被一个变量所拥有，该变量被称为值的所有者
- 一个值同时只能被一个变量所拥有，或者说一个值只能拥有一个所有者
- 当所有者(变量)离开作用域范围时，这个值将被丢弃(drop)

!> 可以和 C++ 的 unique_ptr +  move + RAII 类比理解

与变量拥有自己的值一样，struct 也有用自己的字段。元祖、数组和向量则拥有自己的元素。

可以通过树来描述所有权关系：你的所有者是你的父节点，你拥有的值是你的子节点，每个树的根节点则是某一个变量。当控制流超出了这个变量所在的作用域时，整个树都会被清除。

也就是说每个值都是一个所有权树的节点，而这个树的根节点就是一个变量。

对于栈上的值，默认就满足所有权原则。

对于堆上的值，Rust 提供了 `Box<T>` 智能指针来创建满足所有权原则的堆上数据.

[Box 文档](https://doc.rust-lang.org/std/boxed/struct.Box.html)

> `Box<T>` 是创建堆对象最常用的方式

> 类似于 C++ 中的 `unique_ptr`

上述规则太严厉，很多场景无法实现，为此 Rust 对规则进行了扩展：

- 可以把值从一个所有者转移到另一个所有者
- 标准库提供了基于引用计数的指针类型 Rc 和 Arc，使用它们可以在满足某些限制条件的前提下将值指定给多个所有者。
- 对一个值，可以“借用其引用”。引用是生命期有限的非所有指针。

### 转移/移动（move）

在 Rust 中，对多数类型而言，给变量赋值、给函数传值或从函数返回值这样的操作不会复制值，而是转移（move）值。
所谓转移，就是原来的所有者让渡这个值的所有权给目标所有者，并变成未初始化状态。

> 和 C++ 的 move 语义一致，大多数场景就是把转移堆内存的指针；不一样的是转移后，原来的变量在 C++ 中处于未定义状态，使用会有危险；而 Rust 直接通过编译器检查不给用了(可以赋值)。

!> Rust 中的赋值默认就是转移，而 C++ 中默认是拷贝

如果不需要值的所有权，但是又需要访问值时，可以借用其引用。

### 借用规则

- 同一时刻，你只能拥有要么一个可变引用, 要么任意多个不可变引用
- 引用必须总是有效的

!> 引用的作用域不是块作用域，但是从声明到不再使用的代码行

Rust 中的引用不会为空，取引用运算法 `&T`, 解引用运算符 `*T`。取引用的过程也叫借用

> 引用一般需要显示借用，但是`.` 运算符可以隐式借用引用

> 用法上类似于 C++ 中的引用，获取和解引用语法类似于 C++ 的原始指针。

> 和 C++ 引用一样，Rust 引用可以修改指向

引用有两种：共享引用(`&T`)、可变引用(`&mut T`)

> `&T` 是 Copy 的

可以存在引用的引用，`.` 运算法可以解开多层引用，操作真实的对象

```rust
struct Point { x: i32, y: i32 }
let point = Point { x: 1000, y: 729 };
let r: &Point = &point;
let rr: &&Point = &r;
let rrr: &&&Point = &rr;

// . 运算法可以解开多层引用，操作真实的对象
assert_eq!(rrr.y, 729);
```

同样，比较运算符也可以解开多层引用(前提是引用的层级要一致，否则编译会报类型不匹配的错误)

```rust
let x = 10;
let y = 10;
let rx = &x;
let ry = &y;
let rrx = &rx;
let rry = &ry;

assert!(rrx <= rry);
assert!(rrx == rry);
assert!(!std::ptr::eq(rrx, rry))
```

借用表达式的引用

```rust
fn factorial(n: usize) -> usize {
 (1..n+1).fold(1, |a, b| a * b)
}
// 引用声明周期和变量生命周期一致
let r = &factorial(6);
// 引用生命周期到语句结束
assert_eq!(r + &1009, 1729);
```

> 引用一般只包含简单的地址，但是切片引用和 trait object 引用是个胖指针，包括地址以及长度或者 vtbl 地址。

借用的第二条规则是为了解决悬空指针的问题，但是要怎么保证呢？

为此 Rust 引用了 lifetime 机制。

### 生命周期

Rust 给程序中的每个引用类型附加一个生命期（lifetime），生命期的长短与如何使用该引用匹配。

`生命期`是程序中可以安全使用引用的一个范围，比如一个词法块、一个语句、一个表达式、某个变量的作用域，等等。

> 生命期完全是 Rust 在编译时虚构的东西。而在运行时，引用就是一个地址，其生命期取决于自身的类型，没有运行时表示。

!> 引用的生命期从借用的代码行开始到，最后一次使用的代码行

一般，Rust 会自动推导引用生命期及其使用范围的有效期。

```rust
{
 let r;
 {
    let x = 1;
    // 变量的作用域超过了引用的生命期，编译会报错
    r = &x;
 }
 assert_eq!(*r, 1); // bad: reads memory `x` used to occupy
}
```

生命周期的标注语法:

```rust
&i32        // 一个引用
&'a i32     // 具有显式生命周期的引用
&'a mut i32 // 具有显式生命周期的可变引用

fn longest<'a>(x: &'a str, y: &'a str) -> &'a str; // 函数

// 结构体
// 结构体的生命期要满足 'a 和 'b 的最小值
struct S<'a, 'b> { 
    x: &'a i32,
    y: &'b i32
}
```

编译器使用三条消除规则来确定哪些场景不需要显式地去标注生命周期：

- 每一个引用参数都会获得独自的生命周期
- 若只有一个输入生命周期(函数参数中只有一个引用类型)，那么该生命周期会被赋给所有的输出生命周期
- 若存在多个输入生命周期，且其中一个是 &self 或 &mut self，则 &self 的生命周期被赋给所有的输出生命周期

!> 一般只有在定义函数（不满足消除规则）或者自定义类型包含引用字段的情况下，才需要显式标注引用的生命周期。


另外，为了保证引用的有效性，在共享引用生命周期内，引用的目标值都是只读的：不能重新给它赋值或转移该值

```rust
let v = vec![4, 8, 19, 27, 34, 10];
let r = &v;
let aside = v; // 将向量转移到aside
r[0]; // 不行：使用v，可是它已经变成未初始化了
```


### 共享所有权

真实场景经常会出现一个对象确定应该被多个对象管理的情况。

为了满足这种场景，Rust 提供了 `Rc`(`reference count`) 和 `Arc` 类型

> 类似于 C++ 的 `shared_ptr`

[Rc 文档](https://doc.rust-lang.org/std/rc/struct.Rc.html)

`Rc` 通过实现 `Clone` Trait 来共享所有权，当最后一个 Rc 被清除后，Rust 也会清除对应的值。

```rust
use std::rc::Rc;
// Rust可以推断所有这些类型，这里写出来是为了清楚
let s: Rc<String> = Rc::new("shirataki".to_string());
let t: Rc<String> = s.clone();
let u: Rc<String> = s.clone();
```

共享所有权，导致编译器难以检查借用规则，所以 **`Rc` 中的值默认是不可变的**。如果需要修改值，可用 `Atomic` 类或者使用 `Cell<>` `RefCell<>`(内部可变性智能指针)包装。

> 当然通过 `unsafe` 也可以修改，`Cell` `RefCell` 内部也是通过 `unsafe` 实现的，但是不建议自行使用 `unsafe`

由于 clone 和 drop 会修改引用计算，多线程会存在数据竞争问题，所有 `RC` 不能用于多线程，而是应用使用 `Arc`。

[Arc 文档](https://doc.rust-lang.org/std/sync/struct.Arc.html)

> `Arc` 通过原子语义保证并发安全问题，性能差于 `RC`

`Arc` 中的数据默认也是不可变的，除非内部数据是 `Atomic` 类或者使用 `Mutex<>` 包装

> `Cell` `RefCell` 不是并发安全的，也不能用

共享所有权，很容易出现互相引用的问题，这样会导致内存无法释放，这时可以使用 `std::rc::Weak`

[Weak 文档](https://doc.rust-lang.org/std/rc/struct.Weak.html)

> 类似于 C++ 中的 `weak_ptr`

## 表达式和语句

[文档](https://doc.rust-lang.org/reference/statements-and-expressions.html)

Rust 是所谓的表达式语言

> 表达式语言可追溯到 Lisp

C 族语言中的大多数的控制流工具是语句，而在 Rust 中，它们全是表达式。

!> C 族语言常见的语句和表达式在 Rust 大部分都是表达式

!> 表达式返回值，语句不会

### 语句

Rust 中的语句有四种：

- Item : `fn`、`struct` 或 `use` 等
- let 变量声明语句
- 表达式加分号结束或者表达式返回`()`
- 宏调用

### 表达式

### 块与分号

代码块，同样也是表达式。块产生值，其可以用于任何需要值的地方。块的值就是其内部最后一个表达式的值。

如果最后一个语句加上了分号，那么这个块会默认返回 unit type `()`

```rust
let display_name = match post.author() {
    Some(author) => author.name(),
    None => {
        let network_info = post.get_network_metadata()?;
        let ip = network_info.client_address();
        ip.to_string()
    }
};

let msg = {
    // let声明：分号是必需的
    let dandelion_control = puffball.open();
    // 表达式+分号：方法调用，返回值被清除
    dandelion_control.release_all_seeds(launch_codes);
    // 表达式不带分号：方法被调用，返回值保存于msg中
    dandelion_control.get_status()
};
```

!> let 声明不返回值，比如加上分号
!> 表达式+分号, 返回值被清除



```rust
if preferences.changed() {
    // 编译会报错
    page.compute_size() // 噢，漏掉了分号
}
```

空语句可以出现在块中

```rust
loop {
 work();
 play();
 ; // <-- 空语句
}
```

#### 声明

除了表达式和分号，块还可以包含多个任意声明。最常见的是用于声明局部变量的 let 声明：

```rust
// 类型和初始值是可选的，分号是必需的。
let name: type = expr;
```

和其它语言不一样，在同一作用域中，可以创建同名变量，前面的变量会被覆盖

```rust
for line in file.lines() {
    let line = line?;
    ...
}
```

#### 条件表达式

```rust
if condition1 {
    block1
} else if condition2 {
    block2
} else {
    block_n
}
```

> if 表达式要求每个分支都要返回相同的类型，如果没有 else block，那么 if block 也必须返回 `()`

```rust
match code {
    0 => println!("OK"),
    1 => println!("Wires Tangled"),
    2 => println!("User Asleep"),
    // 类似于 switch 的 default
    _ => println!("Unrecognized Error {}", code)
}
```

match 表达式看起来类似于 C 的 switch 语句，但其实会更灵活，因为其可以使用**模式匹配**

> 模式匹配是 Rust 内置的 mini 语言

```rust
match value {
    pattern => expr,
    ...
}

match params.get("name") {
    Some(name) => println!("Hello, {}!", name),
    None => println!("Greetings, stranger.")
}
```

!> match 的所有模式中必须至少有一个匹配，否则编译器会报错

> match 的所有分支也都必须返回相同类型的值

if 可以使用模式匹配，那就是 `if let` 语句

```rust
if let pattern = expr {
    block1
} else {
    block2
}

if let Some(cookie) = request.session_cookie {
    return restore_session(cookie);
}
```

if let 是 match 的一个子集，等价于 

```rust
match expr {
    pattern => { block1 }
    _ => { block2 }
}
```

#### 循环表达式

四种类型

```rust
while condition {
    block
}
// 类似于 if let
while let pattern = expr {
    block
}

loop {
    block
}

for pattern in iterable {
    block
}
```

Rust 中的 for 循环只提供了 range 语法，但是结合 `..` 语法糖可以很简单的模拟 for-i 语句。

`..` 操作符会产生 Range 对象，`0..20`(半开区间) 等价于 `std::ops::Range { start: 0, end: 20 }`，另外 `0..` `..20` `0..=20` 等写法也会产生对应 Range 类型的对象。这些 Range 类型都实现了 `std::iter::Iterable` Trait（for 循环只能作用于 `Iterable`，）

```rust
for index in 0..20 {
    print!({}, index);
}
```

for 循环本身也是个语法糖，等价于

```rust
while let Some(item) = iterable.into_iter().next() {
    ...
}
```

for 循环也是默认使用 move

```rust
let strings: Vec<String> = error_messages();
for s in strings { // each String is moved into s here...
    println!("{}", s);
}
// ...and dropped here
println!("{} error(s)", strings.len()); // error: use of moved value
```

如果不想被 move，可以使用 `&strings` 引用替代

循环表达式可以使用 `break` `continue` 语句，break 语句可以返回值，还可以指定跳出的 label

```rust
// Find the square root of the first perfect square
// in the series.
let sqrt = 'outer: loop {
    let n = next_number();
    for i in 1.. {
        let square = i * i;
        if square == n {
        // Found a square root.
            break 'outer i;
        }
        if square > n {
            // `n` isn't a perfect square, try the next
            break;
        }
    }
};
```

#### return 表达式

return 用来退出当前函数，并返回一个值。

函数 block 一般使用最后一个表达式作为返回值，如果要提前返回，就需要使用 return

常用于 `Option<T>` 和 `Result<T, E>` 的 `?` 操作符就是个使用 return 的语法糖

```rust
let output = File::create(filename)?;

// 等价于
let output = match File::create(filename) {
    Ok(f) => f,
    Err(err) => return Err(err)
};
```

前面说过，if 表达式的所有分支必须是相同的类型。如果把这个规则强加给以 `break` 或 `return` 表达式结尾的块、无穷 `loop`、对 `panic!()` 或 `std::process::exit()` 的调用，则是不明智的。这些表达式共有的特点是它们都不以惯常的方式结束，不返回值。

不正常结束的表达式通常被指定为特殊类型 `!`，就是说这些表达式永远不会返回。比如 `std::process::exit()` 的函数签名中返回类型就是 `!`

#### 函数和方法调用

```rust
let x = gcd(1302, 462); // function call
let room = player.location();   // method call
let mut numbers = Vec::new();   // type-associated function call
```



## 模式匹配

## 错误处理

## 空处理

## 函数

函数可以定义在 block 内，但是不能访问 block 的局部变量

```rust
use std::io;
use std::cmp::Ordering;
fn show_files() -> io::Result<()> {
    let mut v = vec![];
    ...
    fn cmp_by_timestamp_then_name(a: &FileInfo, b: &FileInfo) -> Ordering {
        ...
    }
    v.sort_by(cmp_by_timestamp_then_name);
    ...
}
```

## 结构体

## 枚举

## 泛型

## 特征（Trait）

## 迭代器

## 数据结构

## unsafe

## IO

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