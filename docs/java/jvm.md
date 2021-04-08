# JVM

## 常用命令

```bash
# 查看 jvm 默认参数
java -XX:+PrintFlagsFinal -version
# 查看默认使用的 GC
java -XX:+PrintFlagsFinal -version | grep -E 'Use.+GC'
# 查看 jvm 进程指定的参数
jinfo -flags <pid>
```

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

## 对象的创建

![](../images/jvm-object-create.svg)

为新生对象分配内存有两种方式

- 指针碰撞
- 空间链表

> 具体看 GC 有没有整理内存，Serial 和 ParNew 就是使用指针碰撞，CMS 使用空间链表（连续空闲块内部使用指针碰撞）

解决指针碰撞的并发问题

- 同步处理，实际上是CAS
- TLAB，TLAB不需要同步处理，用完了才同步处理

## 垃圾收集

[官方文档](https://docs.oracle.com/en/java/javase/11/gctuning/index.html)

### 哪些对象可以回收？

#### 引用计数算法

> 很难解决对象之间循环引用的问题

#### 可达性分析算法

关键是 GC Roots 的选取, 常见的 GC Roots 主要有 
    - 栈中的本地变量表引用的对象
    - 常量引用的对象
    - 静态变量引用的对象

#### 不可达的对象并不一定被回收

![](../images/object-finalize.svg)

#### 回收方法区

条件苛刻，回收收益不高

### 垃圾收集算法

#### 分代收集理论

1. 弱分代假说：绝大多数对象都是朝生夕灭
2. 强分代假说：熬过越多次垃圾收集过程的对象就越难以消亡

根据这两个假说，大多数垃圾收集器都把堆分为新生代和老年代两块区域，分别使用不同的收集算法

> 分代出现了跨代之间的引用，比如老年代引用新生代

3. 跨代引用假说：跨代引用相对于同代引用来说仅占极少数

根据这一假说，就不用为了少量的跨代引用区去扫描这个老年代，只需要建立一个全局的数据机构（记忆集 RememberedSet），这个结构把老年代划分为若干小块，标识出老年代哪一块内存会存在跨代引用，当发生 Minor GC，只有包含跨代引用的小块内存里的对象才会被加入到 GC Roots 中

> 记忆集的标识主要是通过赋值时的 “写屏障” 来实现的

分代导致的 GC 定义

- 部分收集（Partial GC）
    - 新生代收集（Minor GC/Young GC)
    - 老年代收集（Major GC/Old GC)：目前只有 CMS 会单独收集老年代，一般都是 Full GC
    - 混合收集(Mixed GC)： 收集整个新生代和部分老年代，目前只有 G1 有
- 整堆收集（FullGC）： 收集整个 Java 堆和方法区的垃圾收集 

相关命令:

### 标记-清除算法

适用场景：CMS 的老年代收集

缺点：

- 大量对象需要回收时效率低
- 会导致空间碎片化问题

### 标记-复制算法

适用场景：新生代的收集

新生代划分为 Eden 区、两个 Survivor 区，比例默认是 8:1:1

新对象在 Eden 区分配内存，垃圾回收 Eden 区和其中一个 Survivor 区，存活对象放入另一个 Survivor 区，如果放不下，则直接升入老年代

相关参数

- `-Xmm`: 指定新生代大小
- `-XX:SurvivorRatio`: 指定 suvivor 区比例（ratio=Eden大小/Suvivor大小）
- `-XX:PretenureSizeThreshold`: 直接晋升老年代的对象大小

### 标记-整理算法

特点：适用大量对象存活的场景、没有内存碎片化问题

缺点：整理内存时需要 “Stop The World”

> CMS 大多数情况下使用标记-清除算法，如果内存碎片过多，会采用标记-整理算法收集一次


## Hotspot 垃圾收集算法难点

### 准确式 GC 中的 OopMap 及 Safepoint

[准确式 GC 和 OopMap](https://blog.csdn.net/u014028317/article/details/107435049)

如果能够区分内存上某个位置是整型还是指针的 GC 叫 `准确式 GC`，不能区分的叫 `保守式 GC`

#### 保守式 GC

保守式 GC 的缺点：

- 已死的对象由于有疑似指针(可能是个整型)指向它，无法被回收
- 无法移动对象，会出现大量的内存碎片

无法移动对象的问题，可以通过添加一个中间层 `句柄` 来解决，有个句柄表来保证对象的指针，所有引用先指向句柄，再从句柄表找到实际对象地址，这样移动对象，只需要修改句柄表即可，但是这样引用对象访问效率会降低

#### 半保守式 GC

在栈上不记录类型信息，在对象上记录类型信息，叫做半保守式 GC

为了支持半保守式GC，运行时需要在对象上带有足够的元数据。

如果是JVM的话，这些数据可能在类加载器或者对象模型的模块里计算得到，但不需要JIT编译器的特别支持。

#### 准确式 GC

准确式 GC 就是说给定某个位置上的某块数据，要能知道它的准确类型是什么；GC所关心的含义就是“这块数据是不是指针”。 

判断出一个数据的类型：

1. 让数据自身带上标记（tag）。比如，栈上对每个slot都配对一个字长的tag来说明它的类型
2. 让编译器为每个方法生成特别的扫描代码。很少见
3. 从外部记录下类型信息，存成映射表。

目前三种主流的高性能JVM实现，HotSpot、JRockit和J9都是用的第三种方式。

其中，HotSpot把这样的数据结构叫做 `OopMap`，JRockit里叫做 `livemap`，J9里叫做 `GC map`。

要实现这种功能，需要虚拟机里的解释器和JIT编译器都有相应的支持，由它们来生成足够的元数据提供给GC。 

使用这样的映射表一般有两种方式： 

1. 每次都遍历原始的映射表，循环的一个个偏移量扫描过去；这种用法也叫“解释式”； 
2. 为每个映射表生成一块定制的扫描代码（想像扫描映射表的循环被展开的样子），以后每次要用映射表就直接执行生成的扫描代码；这种用法也叫“编译式”。

> HotSpot是用“解释式”的方式来使用OopMap的，每次都循环变量里面的项来扫描对应的偏移量。

#### OopMap

在HotSpot中，对象的类型信息里有记录自己的OopMap，这个 OopMap 是在类加载过程中计算得到的。

类的 OopMap `记录了在该类型的对象内什么偏移量上是什么类型的数据`。可以理解为一个表格，第一个列是偏移量，第二列是对应的类型

对于栈，OopMap 就是记录栈上每个位置是什么类型，但在代码执行到不同的地方，变量的位置也不一样，因此代码执行到不同的地方时对应的 OopMap 不一样

如果每执行一个指令，都生成当时的 OopMap，效率太低，所以一般会选择代码中的某些位置才生成 OopMap，这些位置叫着 `Safepoint`

这些特定的位置主要在： 
1. 循环的末尾 
2. 方法临返回前 / 调用方法的call指令后 
3. 可能抛异常的位置

准确式 GC 就是等到用户线程都在 `Safepoint` 挂起时，扫描 OopMap 获取 GCRoots 的

> 对于 JNI 本地方法，无法给栈生成 OopMap，一般是通过让 JNI栈 通过句柄表来引用对象，这样扫描句柄表即可
> 由于引用了句柄表，这也是 JNI 本地方法性能不佳的原因之一

### 根节点枚举

可达性分析的根节点枚举需要 Stop the World，所有用户线程必须在 `Safepoint` 生成 OopMap 后挂起

一般是 GC 在全局设置一个 GC 标志位，每个线程到达 `Safepoint` 会检查该标志位，发现 GC 标志位后挂起

> 根节点枚举的耗时不会随着堆的变大而变长

### 并发的可达性分析

可达性分析的标记阶段，会随着堆的变大而变长，如果 Stop the world，会导致停顿时间不可控，所以这个过程一般尽量并发进行

[并发的可达性分析过程及问题](https://www.huaweicloud.com/articles/addaf3aaa40f0d0c22327f2ee51f9b8d.html)

主要的问题是并发标记漏标，导致活对象被误回收的致命问题，

解决方法：

- 增量更新(CMS)
- 原始快照(G1)

## 经典垃圾收集器

### Serial 和 Serial Old 收集器

最基础、最老的一对垃圾收集器，简单粗暴，缺点是停顿时间过长

> 至今，Hotspot 虚拟机客户端模式下的默认新生代收集器依然是 Serial 收集器

![](../images/gc-serial.svg)

### ParNew 收集器

ParNew 收集器本质上是Serial收集器的多线程版本，并没有太多的创新之处

> 在单核机器上，ParNew 并不比 Serial 好，多核 CPU ParNew 才有优势

> 在 JDK 7之前，`ParNew + CMS` 是 JVM 服务端模式首选的收集器

![](../images/gc-parnew.svg)

相关参数 

- `-XX:+UseParNewGC`：使用 ParNew 收集器
- `-XX:ParallelGCThreads=xx`: 限制垃圾收集的多线程数

### Parallel Scavenge 收集器

> 新生代收集器

和 ParNew 不一样，Parallel Scavenge 收集器关注的是吞吐量，被称为 `吞吐量有限收集器`

> 吞吐量 = 运行用户代码时间 / (运行用户代码时间 + 运行垃圾收集时间)

相关参数

- `-XX:MaxGCPauseMillis`: 控制垃圾收集最大停顿时间
- `-XX:GCTimeRatio`: 设置吞吐量大小（ratio = 用户代码时间/垃圾收集事假）
- `-XX:+UseAdaptiveSizePolicy`: 不用人工指定新生代大小、SurvivorRatio等等参数，由收集器根据策略动态调整

### Parallel Old 收集器

Parallel Old 是 Parallel Scavenge 收集器的老年代版本

> 在 Parallel Old 出现之前，Parallel Scavenge 由于无法和 CMS 配套，只能会 Serial Old 配置，导致整体性能不佳

> Parallel Old 出现后，Parallel scavenge + Parallel Old 是 JDK9 之前 Hotspot 服务器模式下的默认垃圾收集器 是

![](./images/gc-parallel.svg)

### CMS 收集器

> 老年代收集器

CMS（Concurrent Mark Sweep）收集器是一种以获取最短回收停顿时间为目标的收集器，被称为 `并发低停顿收集器`

![](../images/gc-cms.svg)

由于 CMS 收集过程中，用户线程是并发的，如果等老年代空间满了才开始收集，用户线程新创建的对象将无法分配内存，因此需要在老年代使用到一定程度时触发收集

CMS 使用的是标记-清除算法，会导致内存碎片化问题，如果出现一个大对象无法分配内存，就不得不提前触发 Full GC，使用 Serial Old 来整理老年代空间，为了解决这个问题，

相关参数

- `-XX:+UseConcMarkSweepGC`: 指定使用 CMS 收集器
- `-XX:CMSInitiatingOccupancyFraction`: 指定触发 CMS 收集的内存使用阈值
- `-XX:+UseCMSCompactAtFullCollection`: FULL GC 时进行内存整理
- `-XX:CMSFullGCsBeforeCompaction`: 指定执行几次FULL GC后进行内存整理

### Garbage First 收集器

JDK9 后的默认垃圾收集器

G1 把连续的 Java 堆划分为多个相等的 Region，每个 Region 根据需要扮演新生代的 Eden、Survivor 空间，或者老年代空间

大小超过 Region 大小的一半即可判定为大对象，存储大对象的 Region 区域，称为 Humongous 区域

G1 每次只选择回收收益最高的一批 Region，不需要回收整个新生代或者老年代，因此停顿时间是可控的，不会被堆的大小所影响

> 停顿时间不会被堆的大小所影响，这就是 G1 的最大优势

为了解决跨 Region 引用，G1 在每个 Region 都预留了一个记忆集的结构，记录哪些 Region 引用了该 Region

> 每个 Region 都有记忆集，大概占用了堆容量的 10% - 20%，内存占用是 G1 的最大缺点

由于 G1 每次只回收部分的 Region，这就要求回收的速度要高于内存分配的速度，否则就只能 Stop the world 来解决了

![](../images/gc-g1.svg)

相关参数：

- `-XX:G1HeapRegionSize`: 指定 Region 大小
- `-XX:MaxGCPauseMillis`: 指定最大停顿时间


> 从 G1 开始，最先进的垃圾收集器的设计导向都不约而同地变为追求能够应付应用的内存分配速率，而不追求一次把整个Java堆全部清理干净

### 低延迟垃圾收集器

> 衡量垃圾收集器的三项最重要的指标：内存占用、吞吐量和延迟，三者共同构成了一个 `不可能三角`

延迟日益称为最重要的指标

CMS 使用的是标记清除算法，随着内存空间的增多，终将触发严重的 Stop the world 进行内存整理，这个 Stop the world 时间和堆的大小相关

G1 使用的是标记整理算法，局部看来是标记复制算法，在复制的过程中也是需要 Stop the world 的，随着可以通过控制 Region 的数量控制停顿时间

> 也就是说 G1 及之前的垃圾收集器都无法做到内存整理的并发，低延迟垃圾收集器就是要解决内存整理的并发问题

#### Shenandoah 收集器

Shenandoah 采用和 G1 一样的 Region 布局，通过在对象头存储了转发指针，然后读屏障来将访问转移到新对象来实现内存整理的并发

> 有个和句柄一样的缺点，每次对象访问都会带来一次额外的开销，Shenandoah 是目前第一款使用到读屏障的收集器

对于并发整理的时候，用户进程和 GC 进程存在多线程问题，Shenandoah 是通过 CAS 来保证并发是对象的访问正确性的

#### ZGC 收集器

> 号称停顿时间在 10ms 以下

ZGC 也是采用了 Region 布局，但是其中的 Region 不是一样的，会分为大、中、小三类，暂时也没有给 Region 分成新生代、老年代等角色。

对于并发整理，ZGC 巧妙地使用了 64 位指针暂未使用的几个高位来标记当前指针的指向

> 染色指针是一种直接将少量额外信息存储在指针上的技术

![](../images/gc-pointer.svg)

通过这些标记位，虚拟机可以直接从指针中看到其引用对象的三色标记状态、是否进入了重分配（即被移动过）、是否只能通过finalize()方法才能被访问到

> G1 和 Shenandoah 是使用一个 1/64 堆大小的 Bitmap 来记录的

> ZGC 使用了多重映射技术，将这些标记位不同的同一指针映射到同一物理内存中

> Shenandoah 通过标记位可以知道是否需要转发指针，然后根据 Region 的转发表访问到复制后的对象，，并同时修正更新引用的值，这样只有第一次访问旧对象会慢，这被称为指针的自愈

> ZGC 在并发标记阶段，并没有像之前的垃圾收集器一样，通过维护记忆集，只扫描标记部分的堆内存，反而是扫描整个堆内存来进行标记，这种选择让 ZGC 在内存占用方面也比较少，但也限制了它能承受的对象分配效率不会太高

### 选择合适的垃圾收集

1. Faas（函数即服务）-- Epsilon 收集器（无操作收集器）

2. JDK7及之前延迟优先的选 CMS + ParNew，吞吐量优先的选 Parallel scavenge + Parallel Old

3. JDK8 - JDK11 选 G1

4. JDK12 以后延迟优先选 ZGC

> 主要要考虑堆的大小，延迟优先或吞吐量优先

### 垃圾收集器日志

JDK 9 之前

```bash
# 查看 GC 基本信息
java -XX:+PrintGC
# 查看 GC 详细信息
java -XX:+PrintGCDetails
# 查看 GC 前后堆、方法区可用容量变化
java -XX:+PrintHeapAtGC
# 查看 GC 过程中用户线程并发及停顿时间
java -XX:+PrintGCApplicationConcurrentTime -XX:+PrintGCApplicationStoppedTime
# 查看熬过收集后剩余对象的年龄分布信息
java -XX:+PrintTenuringDistribution
```

JDK 9 之后

```bash
# 查看 GC 基本信息
java -Xlog:gc
# 查看 GC 详细信息
java -Xlog:gc*
# 查看 GC 前后堆、方法区可用容量变化
java -Xlog:gc+heap=debug
# 查看 GC 过程中用户线程并发及停顿时间
java -Xlog:safepoint
# 查看熬过收集后剩余对象的年龄分布信息
java -Xlog:gc+age=trace
```

## 虚拟机性能监控、故障处理工具

定位 JVM 问题，知识、经验是关键基础，数据是依据，工具是运用知识处理数据的手段

数据包括：

- 异常堆栈

> 在应用层面打印

- 虚拟机运行日志

- 垃圾收集器日志

> 上一章已经列出了常见的日志打印参数

- 线程快照（threaddump/javacore 文件）

- 堆转储快照（headdump/hprof 文件）


### 基础命令

> JCMD 和 JHSDB 是集成式的多功能工具类，JHSDB 

| 基础工具 | JCMD | JHSDB | 用途|
| :--: | :--: | :--: | :--: |
| jps -lm | jcmd | N/A | 查看 jvm 进程列表 |
| jmap -dump <pid> | jcmd <pid> GC.heap_dump | jhsdb jmap --binaryheap | 生成堆快照 |
| jmap -histo <pid> | jcmd <pid> GC.class_histogram | jhsdb jmap --histo | 打印当前堆中对象直方图 |
| jstack <pid> | jcmd <pid> Thread.print | jhsdb jstack ---locks | 打印线程快照 |
| jinfo -sysprops <pid> | jcmd <pid> VM.system_properties | jhsdb info --sysprorps | 查看 jvm 系统属性 |
| jinfo -flags <pid> | jcmd <pid> VM.flags | jhsdb jfino -flags | 查看 jvm 参数 |
| jstat -gc <pid> | |  |  | 查看 gc 信息 |

### 图形化工具

- jconsole

> 可以监控内存、线程、类、MBean

- jvisualvm

> 查看进程的配置、环境信息，垃圾收集、线程信息，堆快照分析等等

- jhsdb

- java mission control

## 类文件结构

Class 文件是一组以 8 个字节为基础单位的二进制流，中间没有添加任何分隔符

Class 文件格式本质上只有两种数据类型 `无符号数` 和 `表`

- `无符号数`。以 u1、u2、u4、u8 来分别表示 1、2、4、8 个字节的无符号数，可以用来描述数字、索引引用、数量值或者按照 UTF-8 编码构成字符串值

- `表`。由多个无符号数或者其他表构成的复合数据类型，所有表的命名都习惯性地以 `_info` 结尾。用于描述有层次关系的复合结构的数据， 整个 Class 文件本质上也可以视为一张表

### class 文件结构

![](../images/class.png ":size=40%")

> class 文件的魔数是 `0xCAFEBABE`, 主版本号 JDK 1.1 是 `45`，以此类推

### 常量池表

常量池表首先是一个 u1 字段表示常量类型，然后就是根据常量类型定义的字段了，比如常见的

![](../images/constant-utf8.png ":size=50%")

### 字段表

![](../images/class-field.png ":size=50%")

> 字段描述符是字段类型的字符串描述，比如 `String[] abc` 的描述符是 `[java.lang.String`, 

![](../images/class-descriptor.png ":size=50%")

### 方法表

![](../images/class-method.png ":size=50%")

> 结构和字段表相似，主要的差异是最后的 attribute_info

> 方法描述符： `int indexOf(char[] source, int sourceOffset, int sourceCount, char[]target)` 的描述符是 `([CII[C]])I`

### 属性表

存放类，字段，方法的额外属性，比如方法的执行字节码存在 Code 的属性中，方法抛出的异常存在 Exceptions 属性中，泛型的参数化类型信息存储在 Signature 属性中等

> 属性表是 class 文件最容易扩展的部分，很多额外的信息都可以存储在这里，以便于运行期使用，比如用 Signature 记录下泛型类型，这样就是在运行期通过反射获取真实的类型，在一定程度上解决泛型类型擦除的问题

![](../images/class-attribute.png ":size=50%")

### 字节码指令

#### 加载和存储指令

- 将一个局部变量加载到操作栈:iload、iload_<n>、lload、lload_<n>、fload、fload_<n>、dload、
dload_<n>、aload、aload_<n> 
- 将一个数值从操作数栈存储到局部变量表:istore、istore_<n>、lstore、lstore_<n>、fstore、
fstore_<n>、dstore、dstore_<n>、astore、astore_<n> 
- 将一个常量加载到操作数栈:bipush、sipush、ldc、ldc_w、ldc2_w、aconst_null、iconst_m1、
iconst_<i>、lconst_<l>、fconst_<f>、dconst_<d>
- 扩充局部变量表的访问索引的指令: wide

#### 运算指令

- 加法指令:iadd、ladd、fadd、dadd 
- 减法指令:isub、lsub、fsub、dsub 
- 乘法指令:imul、lmul、fmul、dmul 
- 除法指令:idiv、ldiv、fdiv、ddiv 
- 求余指令:irem、lrem、frem、drem 
- 取反指令:ineg、lneg、fneg、dneg 
- 位移指令:ishl、ishr、iushr、lshl、lshr、lushr 
- 按位或指令:ior、lor 
- 按位与指令:iand、land 
- 按位异或指令:ixor、lxor 
- 局部变量自增指令:iinc
- 比较指令:dcmp g、dcmp l、fcmp g、fcmpl、lcmp

### 同步指令

方法级的同步是隐式的，无须通过字节码指令来控制，它实现在方法调用和返回操作之中。
当方法调用时，调用指令将会检查方法的 `ACC_SYNCHRONIZED` 访问标志是否被设置，如果设置了，执行线程就要求先成功持有管程，然后才能执行方法，最后当方法完成(无论是正常完成 还是非正常完成)时释放管程。

同步一段指令集序列通常是由Java语言中的 synchronized 语句块来表示的，Java虚拟机的指令集中有 `monitorenter` 和 `monitorexit` 两条指令来支持synchronized 关键字的语义

```java
void onlyMe(Foo f) { 
    synchronized(f) {
        doSomething(); 
    }
}
```

字节码

![](../images/class-synchronized.png)

> 为了保证在方法异常完成时 monitorenter 和 monitorexit 指令依然可以正确配对执行，编译器会自动产生一个异常处理程序，这个异常处理程序声明可处理所有的异常，它的目的就是用来执行 monitorexit 指令。

## 类的加载

![](../images/class-load.png)

### 6 种触发类初始化的场景

- 调用 new、getstatic、putstatic、invokestatic 指令时

- 使用 java.lang.reflect 包对类进行反射调用时

- 子类初始化时

- 主类

- 动态语言的方法句柄对应的类

- 有默认方法的接口，在实现类被初始化时

### 类加载器

![](../images/class-loader.png ":size=50%")

![](../images/class-loader-jdk9.png ":size=50%")

![](../images/class-loader-osgi.png ":size=50%")


## Java 的编译器

- 前端编译器: JDK 的 Javac
- 即时编译器: HotSpot 虚拟机的 C1、C2 编译器，Graal编译器。
- 提前编译器: JDK的 Jaotc

### 前端编译器

Java 源文件  -> class 文件

> 前端编译器的优化，支撑着程序员的编码效率和语言使用者的幸福感的提高

语法糖就是前端编译器的提高编码效率的一种手段

- 泛型

    > 类型擦除式泛型，只是语法糖，运行期无法获取到具体类型（后续 class 文件引入了 Signature、LocalVariableTypeTable 等新的属性用于解决运行期泛型的类型识别问题）

    > 现在而已，所谓的擦除，仅仅是对方法的 Code 属性中的字节码进行擦除，实际上元数据中还是保留了泛型信息，这也是我们在编码时能通过反射手段取得参数化类型的根本依据

- 自动装箱、拆箱

- foreach

用户层面可以通过编译时的注解处理器来优化开发效率和体验，比如 lombok

### 后端编译

#### 即时编译器

![](../images/jit.png)

> 解释器和编译器一起工作叫做混合模式(mixed mode)

> 在分层编译出现之前，解释器通常只能其中一个编译器搭配工作

java 相关参数

- `-client`: 指定虚拟机运行于客户端模式

- `-server`: 指定虚拟机运行于服务器模式

- `-Xint`: 强制虚拟机运行于解释模式

- `-XComp`: 运行于编译模式，优化采用编译，无法编译时才用解释

![](../images/compiler.png)

编译对象（热点代码）

- 被多次调用的方法

- 被多次执行的循环体(栈上替换)

热点代码探测方法

- 采用

- 计数器（方法调用计数器、回边计数器）

> Hotspot 用的是 计数器

参数

- `-XX:CompileThreshold`: 方法调用计数器阈值

- `-XX:OnStackReplacePercentage`: OSR 比率

#### 提前编译器

#### 常见的编译优化技术

- 方法内联(最重要)

- 公共子表达式消除

- 数据边界检查消除

- 逃逸分析（最前沿）

> 逃逸程度: 不逃逸、方法逃逸、线程逃逸

> 逃逸优化: 栈上分配，标量替换、同步消除

## Java 内存模型

![](../images/memory-model.png)

8 中操作: lock、unlock、read、load、use、assign、store、write

> read和load 以及 store和write 是一起出现，不可分割的

> lock 操作会清空工作内存此变量的值

> 执行 unlock 操作之前，必须先执行 store、write 操作

### volatile 变量

volatile 是用来解决变量的可见性问题的

volatile 变量的 read、load、use 是一起出现的， assign、store、write 也是一起出现的

volatile 变量只能保证可见性，对于多线程环境，要满足两个规则，才能

- 运行结果并不依赖变量的当前值，或者能够确保只有单一的线程修改变量的值

- 变量不需要与其他的状态变量共同参与不变约束

> volatile 还禁止指令重排序

### 并发的原子性、可见性和有序性

#### 原子性

read、load、user、assign、store、write 这六个都是原子操作

大范围的原子性保证，通过 lock 和 unlock，对应的字节码指令就是 monitorenter 和 monitorexit, 对应的关键字就是 synchronized

### 可见性

volatile 变量可以保证可见性

synchronized 关键字，利用 lock 会清除工作内存变量值及 unlock 之前会 store+write，也可以保证可见性

final 变量，只有初始化完成，就不能修改

### 有序性

Java 只能保证单线程内的执行是有序的，这种有序只是结果的有序，实际执行，不一定是严格按指令顺序来的

多线程的时候，这种有序就会被打破

保证有序性的方法：volatile 和 synchronized

### 线程

#### 线程的实现

- 内核线程

> 实现简单、性能低

- 用户线程

> 实现困难、复制，性能高

- 内核线程+用户线程混合实现

Hotspot 采用的是 内核线程

#### 线程的状态转换

![](../images/thread-state.png)


### 线程安全

共享的数据可分为

- 不可变 (String)

- 绝对线程安全

- 相对线程安全 （Vector， ConcurrentHashMap）

- 线程兼容（ArrayList HashMap）

### 线程安全的实现方法

- 互斥同步（synchronized，ReenterLock）

- 非阻塞同步（CAS）

- 无同步(ThreadLocal)


### 锁的优化

- 自旋锁和自适应自旋锁

> 自适应自旋锁的自选次数是根据策略来，而不是指定的

- 锁消除

- 锁粗化

- 轻量级锁

- 偏向锁

