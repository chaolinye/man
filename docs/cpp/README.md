# C++ 语言特性

## 控制语句

### 循环

- `while(condition) { statement }`
- `do { statement } while(condition)`
- `for(initial statement;condition;after statement) { statement }`

### 判断

- `if (condition) { statement} else if (anoter condition) { statement } else { statement }`

## 注释

- 单行注释: `//`
- 多行注释: `/*   */` 


## 类型和变量

### 内置类型

内置类型分为

- 算术类型
    - 整型（包括字符和布尔类型）
    - 浮点型
- 空类型（void）

对于算术类型所占的比特会 C++ 标准只是规定了最小值（如下），真实大小会**因机器而异**

![](../images/cpp_number_size.png ":size=50%")

> 查看算数类型在本机的大小: `std::cout << sizeof(double) << std::endl;`

除去布尔型和扩展的字符型之外，其他整型可以划分为带符号的 `signed` 和无符号的 `unsigned` 两种

无符号整型用 `unsigned <type>` 表示，其中 `unsigned int` 可以缩写为 `unsigned`

### 类

## 模块化

### 头文件

> 头文件的后缀没有规定，通常使用 `.h`

> `#include` 标准库的头文件应该用尖括号 `<>` 包围，对于不属于标准库的头文件，则用双引号`" "`包围。

## IO

C++ 语言并未定义任何输入输出（IO）语句，而是通过标准库(`iostream`)来提供 IO 机制.

- 标准输出: `std:cout`
- 标准输入: `std:cin`
- 错误输出: `std:cerr` 

> Linux 中重定向标准IO： `./executable_file <infile >outfile 2>errfile`

- 输出运算符: `<<`
- 输入运算符: `>>`

> 输入输出运算符的返回值都是**左侧对象**

```cpp
#include <iostream>
int main() {
    std::cout << "Enter two number:" << std::endl;
    int v1 = 0, v2 = 0;
    std::cin >> v1 >> v2;
    std::cout << "The sum of " << v1 << " and " << v2 << "is" << v1 + v2 << std::endl;
    return 0;
}
```

