# 单元测试

测试是对系统性能的测试、或者对数据的检查

测试是软件质量的重要保证

> 测试让开发人员可以对开发的软件质量拥有信心

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

## JUnit

JUnit 是 Java 领域最流行的单元测试框架。

> JUnit 虽然是为了单元测试而设计，但是也可以结合其它框架进行集成测试、功能测试等各种测试

优点：

- 基于元数据（注解）
- 无侵入性（不需要继承任何类）

这里主要讨论的是 `JUnit4`，`JUnit3` 是有侵入性的

`JUnit4` 相比于 JUnit3 的优势：

- 测试类不再需要继承 `TestCase` 基类
- 不再需要固定 `setUp` `tearDown` 这类方法名，改成使用 `@Before`, `@After` 注解，另外还支持 `@BeforeClass` `@AfterClass` 注解
- 测试方法名不再需要以 `test` 开头，改成使用 `@Test` 注解

注解方法执行顺序

![](../images/junit-before-after.png)

### 断言

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

- assertThat

    assertThat 是最灵活的断言方式，方法签名如下

    ```
    public static <T> void assertThat(T actual, Matcher<? super T> matcher)
    ```

    > Matcher 是来自于 hamcrest-core 的 `org.hamcrest.Matcher` 类，因此 junit 依赖于 hamcrest-core

### Runner

JUnit4 使用 Runner 来运行测试

Runner 自带的 Runner 主要如下:

![](../images/junit4-runner.png)

使用 @RunWith 注解指定 runner， 如果没有指定，那么JUnit将会使用默认的运行器（JUnit4）

为了能够尽可能快捷地运行测试，JUnit提供了一个façade（org.junit.runner.JUnitCore），它可以运行任何 Runner。JUnit设计这个façade来执行你的测试，并收集测试结果与统计信息。

> facade 是一种设计模式，它为子系统中的一组接口提供了一个统一的接口。façade定义了一个更高级别的接口，使得子系统更易于使用。你可以使用facade来将一些复杂的对象交互简化成一个单独的接口。

### 如何执行 JUnit 测试

JUnit 测试一般是通过 IDE 或者 mvn 命令来执行

当然也可以使用最原始的 main 方法执行

> 三种方式本质上都是调用 JUnitCore 来执行，只是 IDE 会收集结果然后渲染出来

```bash
# 编译
javac -classpath /Users/apple/.m2/repository/junit/junit/4.13.1/junit-4.13.1.jar:/Users/apple/.m2/repository/org/hamcrest/hamcrest-core/1.3/hamcrest-core-1.3.jar org/freedom/AssertTest.java
# 运行 junit 测试
java -cp .:/Users/apple/.m2/repository/junit/junit/4.13.1/junit-4.13.1.jar:/Users/apple/.m2/repository/org/hamcrest/hamcrest-core/1.3/hamcrest-core-1.3.jar org.junit.runner.JUnitCore org.freedom.AssertTest
```

通过 mvn 执行

```bash
# 执行某个测试类
mvn -Dtest=OneTest test

# 执行多个测试类
mvn -Dtest=OneTest,Some*Test test

# 执行某个测试方法
mvn -Dtest=OneTest#testMethod
```

IDE 执行过于简单，略

### 高级特性

- Suite

    JUnit 通过 Suite 的概念将多个测试类分组，可以只执行某个组的测试

- Parameterized

    通过二维数组定义多组输入输出，实现同样测试代码，验证多组输入输出

- Rule

    通过

- Theory

    TODO


### JUnit5

JUnit4 的缺点

- 非模块化
- 只能有一个 Runner
- 

JUnit5 的优势

- 模块化

JUnit4 架构

![](../images/junit4-arch.png)

- 








## References

- [你知道 Junit 是怎么跑的吗？](https://juejin.cn/post/6977171333922684958)
- [JUnit4 文档](https://junit.org/junit4/)






