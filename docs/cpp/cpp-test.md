# C++ 单元测试

## 技巧

- 测试 private 方法

    使用 `#define private public` + `#undef private` 包裹要测试的方法所在的头文件的 #include

- mock 非 virtual 方法

    在正式代码中定义 `#define CAN_MOCK `, 然后使用 `CAN_MOCK` 装饰每个要 mock 的非 virtual 方法
    在测试代码中定义 `#define CAN_MOCK virtual` 即可


## References

- [googletest](https://github.com/google/googletest)
- [应用 googletest 编写单元测试代码](https://www.cnblogs.com/fnlingnzb-learner/p/6927834.html)
- [gcov代码覆盖率测试-原理和实践总结](gcov代码覆盖率测试-原理和实践总结)
- [C++ 测试覆盖率统计轻量方案-gtest+lcov](https://blog.csdn.net/hs_err_log/article/details/78024739)
- [gcov 文档](https://gcc.gnu.org/onlinedocs/gcc/Gcov.html#Gcov)