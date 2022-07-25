# GDB 调试

## 调试常用命令

```bash
# 编译某个程序，-g 代表保留调试信息
gcc -g -o cmdTest cmdTest.c

# 查看文件基本信息，可以知道执行文件的格式（elf），是否包含调试信息（no stripped）
file cmdTest

# 查看依赖库
ldd cmdTest

# 查看函数或者全局变量是否存在于elf文件中，nm 是列出执行文件中的符号链接以及对应的地址
nm cmdTest | grep test

# 打印elf文件中的可打印字符串
strings cmdTest | grep someString

# 查看文件段大小, 如果有减小程序大小的需求，就可以有针对性的对elf文件进行优化处理。
# text段：正文段字节数大小, 
# data段:包含静态变量和已经初始化的全局变量的数据段字节数大小
# bss段：存放程序中未初始化的全局变量的字节数大小
size cmdTest

# 去掉符号信息
strip cmdTest

# 查看 elf 文件信息
readelf -h cmdTest

# 反汇编整个程序
objdump -d cmdTest
# 反汇编指定函数，通过 nm 获取指定函数的开始地址和结束地址
objdump -d cmdTest --start-address=0x40052d --stop-address=0x400540

# core dump文件生成配置
ulimit -c #查看core文件配置，如果结果为0，程序core dump时将不会生成core文件
ulimit -c unlimited #不限制core文件生成大小
ulimit -c 10 #设置最大生成大小为10kb
```

## gdb 调试

### 启动调试

```bash
# 调试启动无参程序
$ gdb hello
(gdb)run

# 调试启动带参程序
$ gdb hello
(gdb)run 参数1
# 或者使用 set args 和 run
(gdb)set args 参数1
(gdb)run

# 调试 core 文件
gdb 程序文件名 core文件名

# 调试已运行程序，可以需要修改 `/etc/sysctl.d/10-ptrace.conf` 文件中的 `kernel.yama.ptrace_scope = 1`
$ gdb
(gdb) attach 20829
# 或者直接指定 pid
$ gdb hello 20829
# 或者
gdb hello --pid 20829

# 如果已运行的程序没有信息，可以用同样的代码先编译出一个带调试信息的版本，然后通过 file 命令指定该版本
$ gdb
(gdb) file hello
(gdb) attach 20829
```

### 断点设置

```bash
# 查看已设置的断点
info breakpoints

# 根据行号设置断点，break 可简写为 b
b 9
# 默认是 main 函数所在的文件，也可以指定其它文件的行号
b test.c:9

# 根据函数名设置断点
b printNum

# 根据条件设置断点
break test.c:23 if b==0
# 也可以给已有断点设置条件, 1 是断点的编号
condition 1 b==0

# 规则断点, 用法：rbreak file:regex
# 所有以printNum开头的函数都设置了断点
rbreak printNum*
rbreak .    # 对所有函数设置断点
rbreak test.c:. #对test.c中的所有函数设置断点
rbreak test.c:^print #对以print开头的函数设置断点

# 设置临时断点，只生效依次
tbreak test.c:10

# 某个断点跳过多次，对断点1跳过30次
ignore 1 30

# 根据表达式值变化产生断点
watch a
# 当变量值被读时断住
rwatch a 
# 被读或者被改写时断住。
awatch a

# 禁用或启动断点
disable  #禁用所有断点
disable bnum #禁用标号为bnum的断点
enable  #启用所有断点
enable bnum #启用标号为bnum的断点
enable delete bnum  #启动标号为bnum的断点，并且在此之后删除该断点

# 清除断点
clear   #删除当前行所有breakpoints
clear function  #删除函数名为function处的断点
clear filename:function #删除文件filename中函数function处的断点
clear lineNum #删除行号为lineNum处的断点
clear f:lename：lineNum #删除文件filename中行号为lineNum处的断点
delete  #删除所有breakpoints,watchpoints和catchpoints
delete bnum #删除断点号为bnum的断点
```

### 变量查看

```bash
# 打印基本类型变量，数组，字符数组, print 可简写为 p
p a
# 如果变量存在同名，可以在前面加上函数名或者文件名来区分
p 'main'::a
p 'hello.h'::b

# 打印指针指向的内容
p *d
# 打印数组指针指向的数组,后面加上 @长度
p *d@10

# 每次进入断点自动显示变量内容
display e
# 查看 display 列表
info display
# 清除或者让 display 失效
delete display num
disable display num

# 查看寄存器内容
info registers
```

### 单步调试

```bash
# 单步执行，next 可简写为 n
next

# 单步进入，step 可简写为 s
step
# 查看当遇到没有调试信息的函数，s命令是否跳过该函数，off 是跳过
show step-mode
set step-mode on
set step-mode off

# 继续执行到下一个断点 continue 可简写为 c
continue

# 继续运行到指定位置，util 可简写为 u，本质上利用的是 tbreak 临时断点
u 29

# skip可以在step时跳过一些不想关注的函数或者某个文件的代码
skip funciton add # step 时跳过 add 函数
skip file gdbStep.c
```

### 源码查看

```bash
# 列出 mian 函数所在文件的源码, list 可简写为 l
l

# 列出指定行附近源码
l 9

# 列出指定函数附近的源码
l printNum

# 设置源码一次列出行数
set listsize 20
show listsize

# 列出指定行之间的源码
l 3,15

# 列出指定文件的源码
l test.c:1
l test.c:1,test.c:3

# 指定源码路径(默认是当前目录)
dir ./temp

# 编辑源码
edit 3  #编辑第三行
edit printNum #编辑printNum函数
edit test.c:5 #编辑test.c第五行
```

### 多窗口调试

```bash
gdb hello -tui
```

## 定位 crash 问题

有时候程序崩溃了但不幸没有生成core文件，可以通过 `dmesg` 命令获取出错的基本原因和出错位置

然后通过使用 `addr2line` 命令获取出错具体行号：

```bash
$ addr2line -e cmdTest 40053b
/home/hyb/practice/cmdTest.c:4
```

## References

- [linux常用命令--开发调试篇](https://www.yanbinghu.com/2018/09/26/61877.html)
- [GDB调试指南](https://www.yanbinghu.com/2019/04/20/41283.html)
- [GDB官方文档](https://sourceware.org/gdb/documentation/)