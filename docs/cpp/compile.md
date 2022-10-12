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

### 链接器的顺序依赖

GCC 中的 LD 链接库时是从左到右只遍历一遍，如果某个库依赖的定义还不存在时，会记录下来，在后续的库里寻找；但是如果某个库没有被前面的库依赖，会直接抛弃。

根据上述规则，如果 A 库依赖 B 库，那么链接库时应该先写 A 库，再写 B 库。

因为顺序导致的问题案例：

背景：使用 blade 构建工具，再构建工具的配置文件中定义了 A 依赖 B 依赖 C。但是由于代码设计混乱，其实 C 代码中也有依赖 A，但是为了避免循环依赖，不能配置 C 依赖 A；

问题现象：在正常的构建中，从 A 开始构建，没有问题；但是在写 B 的单元测试时，配置依赖 B 和 A，出现了 C 的代码中找不到 A 的定义的问题

问题分析：blade 自动识别三个库的依赖顺序，依赖顺序写成了 `B的 Test -> A -> B -> C`，由于 B 的单元测试代码中并没有依赖 A，A 就被直接抛弃了，等链接 C 时，由于没有 A 的定义导致报错

短期解决方案：在 B 的单元测试代码中引用 A 的代码。

长期解决方案：梳理好循环依赖的问题

## LLVM 和 Clang

传统的编译器通常分为三个部分，前端(frontEnd)，优化器(Optimizer)和后端(backEnd)。

在编译过程中，前端主要负责词法和语法分析，将源代码转化为抽象语法树；

优化器则是在前端的基础上，对得到的中间代码进行优化，使代码更加高效；

后端则是将已经优化的中间代码转化为针对各自平台的机器代码。

> 通过解耦成三个部分，降低了编译器的复杂性，可以让三个部分独立发展。

GCC 自身包含完整的编译器三部分，而且打包成了一个执行文件。这也导致了模块化差的原因。

### LLVM

LLVM (Low Level Virtual Machine，底层虚拟机) 提供了与编译器相关的支持，能够进行程序语言的编译期优化、链接优化、在线编译优化、代码生成。简而言之，可以作为多种编译器的后台来使用。

最初的搭配是 GCC（前端部分）+ LLVM

### Clang

Clang 是苹果推出的编译器前端， 结合 LLVM，可以完全替代掉 GCC

Glang 的优势

- 内容占用小
- 诊断信息可读性强
- 兼容性好： Clang 从一开始就被设计为一个API，允许它被源代码分析工具和 IDE 集成。GCC 被构建成一个单一的静态编译器，这使得它非常难以被作为 API 并集成到其他工具中。

GCC 的优势：

- 支持更多平台
- 更流行，广泛使用，支持完备


## make

[make 官方文档](https://www.gnu.org/software/make/manual/make.html)

Make 是 Linux 平台上最常用的构建工具，诞生于 1977 年，主要用于C语言的项目。但是实际上 ，任何只要某个文件有变化，就要重新构建的项目，都可以用 Make 构建。

[Make 命令教程](https://www.ruanyifeng.com/blog/2015/02/make.html)

> Java 中的 Ant 就是模仿 Make 来做的（Ant 解决了 Makefile 格式编写容易出错以及无法跨平台使用的问题）

下载 make 

```bash
# centos
yum install -y make
# ubuntu
apt-get install make
```

一个编译 C 语言的 Makefile

```makefile
edit : main.o kbd.o command.o display.o 
    cc -o edit main.o kbd.o command.o display.o

main.o : main.c defs.h
    cc -c main.c
kbd.o : kbd.c defs.h command.h
    cc -c kbd.c
command.o : command.c defs.h command.h
    cc -c command.c
display.o : display.c defs.h
    cc -c display.c

clean :
     rm edit main.o kbd.o command.o display.o

.PHONY: edit clean
```

## cmake

[cmake 官方文档](https://cmake.org/cmake/help/latest/)

> CMake 是事实上的 C++ 构建工具标准

不同平台 Make 工具的 Makefile 的格式千差万别，得写多份 Makefile 工程性很差

CMake 就是为了解决这个问题而设计的：

允许开发者编写一种平台无关的 `CMakeList.txt` 文件来定制整个编译流程，然后再根据目标用户的平台进一步生成所需的本地化 Makefile 和工程文件.

在 linux 平台下使用 CMake 生成 Makefile 并编译的流程如下：

1. 编写 CMake 配置文件 CMakeLists.txt 。
2. 执行命令 `cmake PATH` 或者 ccmake PATH 生成 Makefile（ccmake 和 cmake 的区别在于前者提供了一个交互式的界面）。其中， PATH 是 CMakeLists.txt 所在的目录。
3. 使用 `make` 命令进行编译。

安装 cmake

```bash
yum install -y cmake
```

[CMake 入门实战](https://www.hahack.com/codes/cmake/)

### C++ CMakeList 示例

[CMake 指南](https://cmake.org/cmake/help/latest/guide/tutorial/index.html)

- 编译单个源文件的 CMakeLists.txt

    ```cmake
    # CMake 最低版本号要求
    cmake_minimum_required (VERSION 2.8)

    # 项目信息
    project (Demo1)

    # 指定生成目标
    add_executable(Demo main.cc)
    ```

- 同一目录，多个源文件

    ```cpp
    # CMake 最低版本号要求
    cmake_minimum_required (VERSION 2.8)

    # 项目信息
    project (Demo2)

    # 查找当前目录下的所有源文件
    # 并将名称保存到 DIR_SRCS 变量
    aux_source_directory(. DIR_SRCS)

    # 指定生成目标
    add_executable(Demo ${DIR_SRCS})
    ```

- 多个目录，多个源文件

    根目录的 CMakeLists.txt

    ```cpp
    # CMake 最低版本号要求
    cmake_minimum_required (VERSION 2.8)

    # 项目信息
    project (Demo3)

    # 查找当前目录下的所有源文件
    # 并将名称保存到 DIR_SRCS 变量
    aux_source_directory(. DIR_SRCS)

    # 添加 math 子目录
    add_subdirectory(math)

    # 指定生成目标 
    add_executable(Demo main.cc)

    # 添加链接库
    target_link_libraries(Demo MathFunctions)
    ```

    子目录的 CMakeLists.txt

    ```cpp
    # 查找当前目录下的所有源文件
    # 并将名称保存到 DIR_LIB_SRCS 变量
    aux_source_directory(. DIR_LIB_SRCS)

    # 生成链接库
    add_library (MathFunctions ${DIR_LIB_SRCS})
    ```

### 常用编译命令

> 编译三方库也是这样

```bash
# 创建并进入构建目录
mkdir build && cd $_
# 生成 make/ninja 等构建器的配置文件
cmake ..

# 构建
cmake --build .

# 安装。默认安装目录 /usr/local/
cmake --install .

# 运行测试
ctest -VV
```

Clion 的默认编译命令参考:

```bash
# -D 指定变量，CMAKE_BUILD_TYPE=Debug（编译出带调试符号的目标），CMAKE_MAKE_PROGRAM（指定编译程序）
# -S 指定源码目录，-B 指定目标二进制目录, -G 指定构建器
cmake -DCMAKE_BUILD_TYPE=Debug -S . -B build -DCMAKE_MAKE_PROGRAM=/projector/ide/bin/ninja/linux/ninja -G Ninja

cmake --build build/

```

## blade

[官方文档](https://github.com/chen3feng/blade-build/blob/master/doc/zh_CN/README.md)

blade 是腾讯模仿 Google blaze 的构建系统

> Google blaze 的开源版本是 bazel，但功能并不如 blaze 好用

> blade 主要用 python 实现，构建依赖于 [Ninja](https://ninja-build.org/)（make 的替代物）

[安装](https://github.com/chen3feng/blade-build/blob/master/doc/zh_CN/install.md)

```bash
# 安装依赖 python
yum install -y python2
cd /usr/bin && ln -s python2 python && cd -

# 安装 Ninja(https://github.com/ninja-build/ninja/wiki)
git clone --depth 1 -b release https://github.com/ninja-build/ninja.git && cd ninja
./configure.py --bootstrap
cd /usr/bin && ln -s $OLDPWD/ninja ninja && cd -

# 安装 blade
git clone --depth 1 https://github.com/chen3feng/blade-build.git && cd blade-build
./install
blade --version
```

一个 blade 的 BUILD 例子

```python
# 生成库
cc_library(
  name = 'say',
  srcs = 'say.cc',
  hdrs = ['say.h'],
  visibility = ['PUBLIC'],
)

# 生成可执行文件
cc_binary(
  name = 'hello',
  srcs = [
    'hello.cc',
  ],
  deps = [':say']
)
```

## 包管理

- [build2](https://build2.org/): 一个开源的 (MIT)、跨平台的构建工具链，旨在为开发和打包 C/C++ 项目时提供如 Rust Cargo 一样的便利性。
- [cget](https://cget.readthedocs.io/en/latest/):	Cmake 包检索工具，可用于下载并安装 Cmake 包。
- [cmodule](https://github.com/scapix-com/cmodule):	非侵入式 CMake 依赖管理。
- [conan](https://conan.io/):	去中心化、开源 (MIT) 的 C/C++ 包管理器。
- [CPM.cmake](https://github.com/TheLartians/CPM.cmake):	一段可以为 CMake 加入依赖管理功能的 CMake 脚本。它是作为 CMake 的 FetchContent 模块的一个简单包装构建的。该模块加入了版本控制、缓存、简单 API 等功能。
- [hunter](https://hunter.readthedocs.io/en/latest/):	一个 CMake 驱动的跨平台包管理器，服务于 C/C++ 项目。
- [spack](https://spack.io/)	一个超级计算机、Linux、macOS 平台的包管理器。它使得安装科学软件变得简单。非绑定于某一特定语言。
- [teaport](https://bitbucket.org/benman/teaport): 一个受 cocoapods 启发的依赖管理器。
- [vcpkg](https://docs.microsoft.com/en-us/cpp/vcpkg): 一个 Windows、Linux、macOS 平台的 C++ 包管理器。

## References 

- [gcc和g++是什么关系？](https://www.zhihu.com/question/20940822)
- [详解三大编译器：gcc、llvm 和 clang](https://developer.51cto.com/article/630677.html)
- [Blade——一个腾讯开源的C++工程构建利器](https://juejin.cn/post/6996646512390307870)
- [寻找 Google Blaze](https://zhuanlan.zhihu.com/p/55452964)
- [Linux系统下 连接器ld链接顺序的总结](https://m.xp.cn/b.php/70857.html)
- [GCC LD 对依赖库的输入顺序敏感](https://zhiqiang.org/coding/order-is-important-in-gcc-ld.html)
