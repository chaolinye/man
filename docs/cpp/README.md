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


## 类型

## 模块化

### 头文件

> 头文件的后缀没有规定，通常使用 `.h`

## IO

C++ 语言并未定义任何输入输出（IO）语句，而是通过标准库(`iostream`)来提供 IO 机制.

- 标准输出: `std:cout`
- 标准输入: `std:cin`
- 错误输出: `std:cerr` 

- 输出运算符: `<<`
- 输入运算符: `>>`

> 输入输出运算符的返回值都是左侧对象

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

