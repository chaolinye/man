# C++ 的编译和构建

## GCC

### GCC 简介

GCC 是 GNU 编译器集合的意思, 对于用户能用的常用命令, 有 `gcc` 和 `g++`。

无论是 gcc 还是 g++, 他们的定位都是 `compiler driver`, 不负责编译代码，只负责调用真正的编译器。

`driver` 负责调用编译器(狭义), 把源码编译到汇编代码. 比如 C 语言的编译器(狭义)是 `cc1`, 而 C++ 语言的编译器(狭义)是 `cc1plus`。`driver` 再调用汇编器 `as`, 把汇编代码变成二进制代码. 最后调用链接器 `ld`, 负责把二进制代码拼在一起.

![](../images/gcc.png)

gcc 和 g++ 的区别:

- 调用的编译器不同
    - g++ 会把 .c 文件当做是 C++ 语言 (在 .c 文件前后分别加上 `-xc++` 和 `-xnone`, 强行变成 C++), 从而调用 cc1plus 进行编译。g++ 遇到 .cpp 文件也会当做是 C++, 调用 cc1plus 进行编译. 
    - gcc 会把 .c 文件当做是 C 语言. 从而调用 cc1 进行编译.gcc 遇到 .cpp 文件, 会处理成 C++ 语言. 调用 cc1plus 进行编译. 

- 传递给链接器的参数不同
    - g++ 会默认告诉链接器, 让它**链接上 C++ 标准库**(`-lstdc++`).
    - gcc 默认不会链接上 C++ 标准库.

GCC 项目中源码文件后缀与编译器的默认对应关系:

![](../images/gcc-compiler.jpg ":size=40%")

> 所以 c++ 的源文件后缀可以选用 `.cc` `.cpp` `.c++` `.CPP` `.cxx` `.cp` `.C` `.ii` (前两种最常见)

### 下载 GCC 

- Linux

    ```bash
    # Centos
    ## 下载开发包，包括 gcc、g++、make、gdb、perl、python等等
    yum group install "Development Tools"

    ## 或者独立下载 gcc,g++,gdb
    yum install gcc
    yum install gcc-c++
    yum install gdb

    ## 查看版本
    gcc --version
    g++ --version

    # Ubuntu
    ## 下载开发包，包括 gcc、g++、make等等
    apt-get install build-essential
    ```

- Windows

    Windows 上也有 GCC 的移植版本，比如 MinGW 和 Cygwin 等

    [MinGW安装教程](https://cloud.tencent.com/developer/article/1605800)

    > 如果[MinGW 官网](http://www.mingw.org/)进不去，可以去 [OSDN](https://zh.osdn.net/projects/mingw/) 下载


### 使用 GCC 

#### 单文件编译

`hello.cc`

```cpp
#include <iostream>

int main() {
  std::cout << "Hello World" << std::endl;
  return 0;
}
```

```bash
# 编译
g++ hello.cc -o hello
# 执行
./hello
```

#### 多文件编译

`hello.cc`

```cpp
#include <iostream>
#include "hello.h"

namespace custom {
    void sayHello() {
    std::cout << "Hello World" << std::endl;
    }
}
```

`hello.h`

```cpp
// 避免头文件被重复 include
#pragma once

namespace custom {
    void sayHello();
}
```

`main.cc` 

```cpp
#include "hello.h"

int main() {
  custom::sayHello();
  return 0;
}
```

```bash
# 先把源文件转成机器码（使用编译器和汇编器，但不使用链接器）
g++ -c main
g++ -c hello

# 链接成可执行文件
g++ -o main main.o hello.o

# 运行执行文件
./main
```
## Makefile

## cmake

## blade

## References 

- [gcc和g++是什么关系？](https://www.zhihu.com/question/20940822)
