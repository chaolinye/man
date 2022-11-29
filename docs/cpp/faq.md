# 常见问题

## 库的类型

[Linux共享库、静态库、动态库详解](https://www.cnblogs.com/sunsky303/p/7731911.html)

## 命名空间解析

[C++ 嵌套 namespace 解析](https://juejin.cn/post/6857516154799030286)
[C++ 命名空间：默认命名空间与匿名命名空间](https://blog.csdn.net/PecoHe/article/details/112505385)

## 编译器参数

[GCC FLAGS](https://developers.redhat.com/blog/2018/03/21/compiler-and-linker-flags-gcc)

## inline 函数

inline 只是个提示，是否真的内联由编译器根据函数的复杂度等因素来决定。

> 如果函数比较复杂，调用函数的成本相对于函数内部的成本可忽略不计，则不会内联

内联是由编译器执行，而不是链接器，所以要内联的函数的定义必须在编译期可见。

所以对于跨文件的 inline 函数，必须定义在头文件中，这样经过 #include 才能在编译期可见。

因此，类的方法，也是要定义在类内，才默认是 inline 函数；如果定义在类外，则不会默认 inline，需要显示标识（在声明或者定义其中一处标识即可，优先在声明处）；如果调用方是跨文件的，即使定义在类外，也需要定义在头文件中。

[C++类里面的哪些成员函数是内联函数？](https://blog.csdn.net/qq_18343569/article/details/83755202)

为了可读性，经常需要把大方法拆分成多个小方法，那么这些小方法需要定义成 inline 么？

> 一般不需要使用内联，函数调用的成本没那么高，只有频繁调用（循环中、多处调用等）的小函数才有必要。

如何识别 inline 是否被编译器应用？

```c++
// 使用 gcc 的内置函数 __builtin_return_address(LEVEL) 通过打印当前函数的返回地址判断 inline 是否生效
std::out << "current return address: " <<  __builtin_return_address(0) << std::endl;
```

另外更准确的做法是看反汇编。

