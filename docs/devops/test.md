# 测试

测试是对系统性能的测试、或者对数据的检查

测试是软件质量的重要保证

> 测试让开发人员可以对开发的软件质量拥有信心

## 单元测试 

单元测试(Unit Test) 是系统测试的基础，同时也是测试驱动开发（TDD）的基础

单元测试的要求

- 执行速度快
- 不依赖外部对象
- 覆盖主要代码

单元测试结果的检查不应该通过 `打印` 或者 `debug` 来检查，而是使用 `assert` 来检查，这样才能实现自动化

Java 领域流行的单元测试框架有：

- Jtest
- JUnit
- TestNG

其中 `JUnit` 是最为流行的

### JUnit

JUnit 是 Java 领域最流行的单元测试框架。

优点：

- 基于元数据（注解）
- 无侵入性（不需要继承任何类）

这里主要讨论的是 `JUnit4`，`JUnit3` 是有侵入性的

`JUnit4` 相比于 JUnit3 的优势：

- 测试类不再需要继承 `TestCase` 基类
- 不再需要固定 `setUp` `tearDown` 这类方法名，改成使用 `@Before`, `@After` 注解，另外还支持 `@BeforeClass` `@AfterClass` 注解
- 测试方法名不再需要以 `test` 开头，改成使用 `@Test` 注解

注解方法执行顺序

```
beforeClass
before
test1
after
before
test2
after
afterClass
```

#### 断言

`Assert` 类常用断言工具方法

- assertTrue
- assertFlase
- assertNull
- assertNotNull
- assertEquals

    > 如果是 `dobule` 类型，要使用 `assertEquals(double expected, double actual, double delta)` 或者直接使用 BigDecimal 替代

- assertSame

    > 等同于 `=`

- assertNotSame

异常断言是通过 `@Test(expected=<Exception class name>.class)` 注解参数来指定

```java
@Test(expected=RuntimeException.class)
public void test_exception_condition() {
    throw new RuntimeException();
}
```

#### Runner


## References

- [你知道 Junit 是怎么跑的吗？](https://juejin.cn/post/6977171333922684958)






