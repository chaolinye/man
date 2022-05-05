# Elastic Search

## vs Lucene

Elastic Search 基于 Apache Lucene 构建，其主要买点有

- Robust caching
- 提供了 HTTP API。Lucene 本质上是个 Java 库，并不能被其它语言直接使用，ES 提供的 HTTP API 可以给任意语言使用
- Backward-compatibility

> ES 是中间件，很多集群相关的事情都内置处理好了；而 Lucene 只是个库，如果要基于 Lucene 构建搜索能力，还要做大量的工作


## 核心原理

- 分词
- 倒排索引

![](../images/es-core.png ":size=70%")

- 分片(sharding) -- 性能
- 副本(replicas) -- 并发度 + 高可用

![](../images/es-shard-replica.png)

## 三种架构模式

1. ![](../images/es-arch-1.png ":size=30%")

2. ![](../images/es-arch-2.png ":size=50%")

3. ![](../images/es-arch-3.png ":size=60%")

## 逻辑布局和物理布局

![](../images/es-layout.png)


## References

- [通过容器运行ELK](https://elk-docker.readthedocs.io/)