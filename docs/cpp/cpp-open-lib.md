# C++ 常用三方库

## 引用三方库

### 方法一：在编译自己的项目时添加 -L 和 -I 编译选项

1. 添加头文件路径: `-I/dir` 添加头文件的搜索路径

2. 添加库文件路径：
`-L`    #指定目录。link的时候，去找的目录。gcc会先从-L指定的目录去找，然后才查找默认路径。（告诉gcc,-l库名最可能在这个目录下）。
`-l`     #指定文件（库名），linking options

> `-l` 紧接着就是库名，这里的库名不是真正的库文件名。比如说数学库，它的库名是 `m`，他的库文件名是 `libm.so`。再比如说matlab eigen库，它的库名是 `eng`，它的库文件名是 `libeng.so`。即把库文件名的头 lib 和尾 .so 去掉就是库名了。在使用时，`-leng` 就告诉 gcc 在链接阶段引用共享函数库 `libeng.so`。


```bash
// 指定头文件路径（/opt/MATLAB/R2012a/extern/include），指定链接时的库路径（/opt/MATLAB/R2012a/bin/glnxa64），指定要链接的库
g++ matlab_eigen.cpp -o matlab_eigen -I/opt/MATLAB/R2012a/extern/include -L/opt/MATLAB/R2012a/bin/glnxa64 -leng -lmx
```

### 方法二： 将库路径添加到环境变量

1. 添加头文件路径：

在 `/etc/profile` 中添加（根据语言不同，任选其一）：

- `c`: `export C_INCLUDE_PATH=C_INCLUDE_PATH`: 头文件路径
- `c++`: `export CPLUS_INCLUDE_PATH=CPLUS_INCLUDE_PATH`:头文件路径

需执行一次 `source`。

另有一种方法：在 `/etc/ld.so.conf` 文件中加入自定义的 lib 库的路径，然后执行 `sudo /sbin/ldconfig`，这个方法对所有终端有效。

2. 添加库文件路径：
- `LIBRARY_PATH`: 链接时使用的静态库路径 
- `LD_LIBRARY_PATH`: 运行时的动态库路径


```bash
# 设置环境变量
MATLAB=/opt/MATLAB/R2012a
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$MATLAB/bin/glnxa64
export LIBRARY_PATH=$LIBRARY_PATH:$MATLAB/bin/glnxa64
export CPLUS_INCLUDE_PATH=CPLUS_INCLUDE_PATH:$MATLAB/extern/include

# 编译
g++ matlab_eigen.cpp -o matlab_eigen -leng -lmx
```

## Boost

[Boost](https://www.boost.org/) 是 C++ 最有名的第三库，也是 C++ 准标准库。

## 命令行参数解析

[google flags](https://github.com/gflags/gflags)

## 倒排表

[RoaringBitmap](https://github.com/RoaringBitmap/CRoaring)

## References

- [awesome-cpp](https://github.com/fffaraz/awesome-cpp)
- [gcc/g++使用第三方库时添加头文件路径和库文件路径的方法](https://blog.csdn.net/arackethis/article/details/43342655)