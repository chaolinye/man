# Rust 宏编程

> Rust 的宏比 C++ 的宏更为强大，因为其对输入的解析能力足够强大。

在 Rust 中宏分为两大类：

- `声明式宏`( declarative macros ): 
    - `macro_rules!` 

- `过程宏`( procedural macros ):
    - `#[derive]`，在之前多次见到的派生宏，可以为目标结构体或枚举派生指定的代码，例如 Debug 特征
    - 类属性宏(Attribute-like macro)，用于为目标添加自定义的属性
    - 类函数宏(Function-like macro)，看上去就像是函数调用

## 宏和函数的区别

从根本上来说，宏是通过一种代码来生成另一种代码。元编程可以帮我们减少所需编写的代码，也可以一定程度上减少维护的成本。

虽然函数复用也有类似的作用，但是宏的能力更为强大。比如 Rust 的函数不支持可变参数，但宏可以实现。

函数本身是 Rust 代码，而宏是在编译之前基于字符串操作生成 Rust 代码

> Rust 的宏类似于模板引擎，自定义宏就是在写模板，然后编译器会被模板引擎转换成对应的代码

宏的缺点：可读性和可维护性差，一般只用于构建框架。

## 声明式宏 `macro_rules!` 

在 Rust 中使用最广的就是声明式宏，甚至直接称呼为宏

声明式宏允许我们写出类似 match 的代码。宏是将一个值跟对应的模式进行匹配，且该模式会与特定的代码相关联。但是与 match 不同的是，**宏里的值是一段 Rust 源代码(字面量)**，模式用于跟这段源代码的结构相比较（**比较的方式更像是正则表达式匹配**），一旦匹配，传入宏的那段源代码将被模式关联的代码所替换，最终实现宏展开。值得注意的是，**所有的这些都是在编译期发生，并没有运行期的性能损耗**。

常见的宏 `vec!` 的一个简化实现

```rust
// 类似于 pub，导出宏给其它包引入
#[macro_export]
// 定义宏名称，注意这里不用写感叹号
macro_rules! vec {
    // 类似于 match 表达式的分支语法：模式 => 代码，可以多个分支
    // 不过这里的匹配语法更像是正则表达式
    ( $( $x:expr ),* ) => {
        // 要生成的代码模板，里面有用模板的循环语法
        {
            let mut temp_vec = Vec::new();
            $(
                temp_vec.push($x);
            )*
            temp_vec
        }
    };
}
```

## 过程式宏

> 过程式宏相对于声明宏更强大，解析语法更加的灵活，其中的 Function-like macro 可以完全实现声明式宏的能力。

当创建过程宏时，它的定义必须要放入一个独立的包中，且包的类型也是特殊的。

> 过程宏放入独立包的原因在于它必须先被编译后才能使用，如果过程宏和使用它的代码在一个包，就必须先单独对过程宏的代码进行编译，然后再对我们的代码进行编译，但悲剧的是 Rust 的编译单元是包，因此你无法做到这一点。

过程宏独立包的 Cargo.toml 必要配置

```rust
# 过程宏的开关
[lib]
proc-macro = true

# 过程宏的依赖
[dependencies]
syn = "1.0"
quote = "1.0"
```

### dervice macro

derive 宏输出的代码并不会替换之前的代码，这一点与声明宏有很大的不同

定义 derive 宏

```rust
extern crate proc_macro;

use proc_macro::TokenStream;
use quote::quote;
use syn;

#[proc_macro_derive(HelloMacro)]
pub fn hello_macro_derive(input: TokenStream) -> TokenStream {
    // 基于 input 构建 AST 语法树
    let ast = syn::parse(input).unwrap();

    // 构建特征实现代码
    impl_hello_macro(&ast)
}

fn impl_hello_macro(ast: &syn::DeriveInput) -> TokenStream {
    let name = &ast.ident;
    let gen = quote! {
        impl HelloMacro for #name {
            fn hello_macro() {
                println!("Hello, Macro! My name is {}!", stringify!(#name));
            }
        }
    };
    gen.into()
}
```

使用宏：

```rust
use hello_macro::HelloMacro;
use hello_macro_derive::HelloMacro;

#[derive(HelloMacro)]
struct Sunfei;

#[derive(HelloMacro)]
struct Sunface;

fn main() {
    Sunfei::hello_macro();
    Sunface::hello_macro();
}
```

其中，`struct Sunfei` 的输入通过 AST 解析得到的 DeriveInput 结构体大概是这样子的：

```
DeriveInput {
    // --snip--

    ident: Ident {
        ident: "Sunfei",
        span: #0 bytes(95..103)
    },
    data: Struct(
        DataStruct {
            struct_token: Struct,
            fields: Unit,
            semi_token: Some(
                Semi
            )
        }
    )
}
```


### Attribute-like macro

Attribute-like macro 相对于 `dervie` 宏，可以定义自己的属性。除此之外，**derive 只能用于结构体和枚举，而类属性宏可以用于其它类型项，例如函数**。

```rust
#[proc_macro_attribute]
pub fn route(attr: TokenStream, item: TokenStream) -> TokenStream {
```

```rust
#[route(GET, "/")]
fn index() {
```

### Function-like macro

类函数宏可以让我们定义像函数那样调用的宏，从这个角度来看，它跟声明宏 macro_rules 较为类似。

```rust
// 定义类函数宏
#[proc_macro]
pub fn sql(input: TokenStream) -> TokenStream {
}
```

```rust
// 调用类函数宏
let sql = sql!(SELECT * FROM posts WHERE id=1);
```

> 过程宏的解析更加灵活，对 SQL 语句进行解析并检查其正确性，这个复杂的过程是 macro_rules 难以对付的。

## Reference

- [官方文档](https://doc.rust-lang.org/reference/macros-by-example.html)
- [Macro 宏编程](https://course.rs/advance/macro.html)
- [The Little Book of Rust Macros](https://veykril.github.io/tlborm/)