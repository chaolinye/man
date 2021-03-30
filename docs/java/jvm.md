# JVM

## 运行时数据区

![](../images/jvm-data.svg)


常见错误：

- OutOfMemoryError
- StackOverflowError

### 栈

```bash
# 设置栈的大小
java -Xss1m
```

### 堆

> 线程共享的 Java 堆中可以划分出多个线程私有的分配缓冲区（Thread Local Allocation Buffer，TLAB)，以提升对象分配时的效率

对象分配的地方，垃圾收集的地方

```bash
# 设置堆的最小值和最大值
java -Xms20m -Xmx30m
```

### 方法区

方法区用于存储 `类型信息`、`常量`、`静态变量`、`即时编译器编译后的代码缓存`等数据

方法区在 JDK7 及以前通过永久代实现，JDK8 起通过元空间(`Metaspace`)实现

> JDK7 起原本存放在永久代的 `字符串常量池` 被移至 Java 堆之中

```bash
# JDK7 永久代设置初始大小和最大值
java -XX:PermSize=6M -XX:MaxPermSize=6M

# JDK8 Metaspace 限制大小，默认只受限于机器内存大小
java -XX:MetaspaceSize=6M -XX:MaxMetaspaceSize=6M
```


### 直接内存

Direct Memory 并不是虚拟机运行时数据区的一部分，主要是给 NIO 使用，大小受限于机器内存大小

## 对象探秘

### 对象的创建

![](../images/jvm-object-create.svg)

为新生对象分配内存有两种方式

- 指针碰撞
- 空间链表

> 具体看 GC 有没有整理内存，Serial 和 ParNew 就是使用指针碰撞，CMS 使用空间链表（连续空闲块内部使用指针碰撞）

解决指针碰撞的并发问题

- 同步处理，实际上是CAS
- TLAB，TLAB不需要同步处理，用完了才同步处理

## 垃圾收集

### 哪些对象可以回收？

- 引用计数算法
- 可达性分析算法

### 垃圾收集算法

