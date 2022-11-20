# C++ 宏编程

## 常用符号

- `##`

    连接符，可将多个标识符拼接起来，组成一个完整的标识符。

    ```cpp
    // 定义宏，用来打印整型变量
    #define PRINT(x) printf("%d\n",a##x)
    
    int a1 = 1;
    int a2 = 2;

    PRINT(1);   // 等同于 printf("%d\n", a1), 输出 1
    PRINT(2);   // 等同于 printf("%d\n", a2), 输出 2
    ```

- `#`

    添加双引号，转成字符串

    ```cpp
    // 定义宏，转成字符串
    #define STR(X) #x

    // 等同于 printf("%s\n", "Hello, world!");
    printf("%s\n", STR(hello, world!));
    ```

- `#@`

    添加单引号，转成字符

    ```cpp
    #define CH(x) #@x

    char a = CH(M); // 等同于 char a= 'M'
    printf("%c\n", a);
    ```

## Reference

- [C++宏编程技巧](https://blog.csdn.net/gkzscs/article/details/82934054)