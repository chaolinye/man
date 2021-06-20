# Netty

Netty 是一个高性能通信框架，简化了 Java NIO 网络编程，并对于网络编程中的各种问题提供了解决方案。

> 涉及 Java 高性能网络通信，Netty 是首选底层框架

Java 1.4 出现的 NIO 类库，让 Java 有了高性能网络编程的能力

但是 NIO 编程问题众多且难以排查，导致难以普及，Netty 简化了 NIO 的编程，且内置解决了众多 NIO 问题

Netty 让 Java 在高性能服务器领域有了和 C++ 的一比之力

## 5 种 IO 模型

- 阻塞 IO 模型

![](../images/bio.png)

- 非阻塞 IO 模型

![](../images/nio.png)

- IO 复用模型

![](../images/io-select.png)

- 信号驱动 IO 模型

![](../images/io-sign.png)

- 异步 IO

![](../images/aio.png)

## IO 多路复用

## 粘包和粘包

## Netty 高性能之道

- 异步非阻塞通信

- 高效的 Reactor 线程模型

    - Reactor 单线程模型
    - Reactor 多线程模型
    - 主从 Reactor 多线程模型

- 高性能的序列化框架
    - 序列化后的码流大小
    - 序列化和反序列化的性能（CPU资源占用）
    - 是否支持跨语言

- 零拷贝

- 内存池

- 无锁化的串行设计

- 高效的并发编程