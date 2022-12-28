# C++ 标准库

## 标准库简介

1. C++98 是第一份 C++ 标准规格
2. C++03 是对 C++98 bug 的修正
3. TR1（Technical Report on C++ Libray Extensions） 内含大幅度的标准库扩充，位于 namespace std::tr1 内
4. C++11 是第二份 C++ 标准

从 C++11 开始每 3 年发布一次新标准

> C++11 对 C++ 98 的向后兼容性仅适用于源码，不保证二进制兼容

C++ 标准的源头各种各样，也导致标准库中不同组件的设计思想不一样。比如 `String` 被设计为一个安全而便利的组件；而 STL 被设计为“将不同的数据结构与算法结合起来，产生最佳效能”，并不便利也不安全。

## 标准库的基本概念

### 命令空间 std

和 class 不同，namespace 具有扩展开放性，可发生于任何源码上。

C++ 标准库中的所有标识符都被定义于一个名为 `std` 的 namespace 内（或者子 namespace 内）。

使用标准库标识符的三种选择

- 直接指定标识符. `std::ostream`
- 使用 using declaration. `using std::ostream;`
- 使用 using directive. `using namespace std;` 这种方式容易导致意外的名称冲突，应该避免使用

### 头文件

C++ 标准库的头文件是没有后缀名的

> 真实文件系统中的头文件是否有扩展名由编译器决定

```c++
#include <iostream>
#include <string>
```

这种写法也适用于 C 标准头文件，但必须采用前缀字符 c

```c++
#include <cstdlib> // was: <stdlib.h>
#include <string> // was: <string.h>
```

在这些头文件中，每个标识符都被声明于 namespace std。

### Error 和 Exception 的处理

C++ 标准库不同组件对于错误处理差异很大，例如 string class，支持具体的差错处理；而 STL 效率重于安全，因此几乎不校验逻辑差错。

![](../images/cpp_stdexcept_2.png)

逻辑错误通常可以避免

运行期异常则是由一个位于程序作用域之外的原因触发，例如资源不足。

异常类的头文件

```c++
#include <exception>    // for classes exception an bad_exception
#include <stdexcept>    // for most logic and runtime error classes
#include <system_error> // for system errors(since c++11)
#include <new>          // for out-of-memory exceptions
#include <ios>          // for I/O exceptions
#include <future>       // for errors with async() and futures(since c++11)
#include <typeinfo>     // for bad_cast and bad_typeid
```

所有标准异常都提供 `what()`,某些异常类还提供了 `code()`


### 可调用对象

### 并发与多线程

### 分配器