# C++

从 2011 年发布的 `C++11` 开始，C++ 标准委员会承诺每 3 年发布一个新的 C++ 标准。截止到目前，已经发布了 `C++14`、`C++17`、`C++20` 标准以及若干个扩展 C++ 标准的技术规范。


## 语言心得

标准库定义的类名都是小写加下划线风格。

对象分配默认是在栈上，可通过 `new` 或者智能指针分配到堆上。

变量之间的赋值默认都是深拷贝，所以尽量使用引用、指针、move 来传递值。

可通过 `#define` 宏实现编译时代码生成，简化代码编写。

编程时要特别留意内存安全问题：

- 内存泄漏（RAII，坚持用智能指针）
- 孤儿指针、double free（注意指针的生命周期，坚持用智能指针）
- 数组越界（多考虑边界问题）

工程性体验最差的是缺乏包管理工具，头文件的模块机制也很差，特别是私有函数要在头文件声明。

最难的部分是模板元编程（多看标准库）。



## References

- [cpp reference](https://en.cppreference.com/w/)
- [C++ Primer]()
- [Effective Modern C++](https://cntransgroup.github.io/EffectiveModernCppChinese/Introduction)
- [学会查看类型推导结果](https://cntransgroup.github.io/EffectiveModernCppChinese/1.DeducingTypes/item4.html)
- [GCC STL 源码](https://github.com/gcc-mirror/gcc/tree/master/libstdc++-v3/src)
- [C++ 参考手册](https://www.apiref.com/cpp-zh/cpp.html)
- [C++ 11 vs C++ 14 vs C++ 17](https://www.geeksforgeeks.org/c-11-vs-c-14-vs-c-17/)