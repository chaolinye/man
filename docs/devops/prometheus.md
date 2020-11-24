# Prometheus

Prometheus 是一个开源的监控和告警工具包。

Prometheus 的主要特点是：

- 一个多维数据模型，其中包含通过度量名称和键/值对标识 `时间序列数据`

- PromQL，一种灵活的查询语言，可利用此维度

- 不依赖分布式存储； 单服务器节点是自治的

- 时间序列收集通过HTTP上的拉模型进行

- 通过中间网关支持推送时间序列

- 通过服务发现或静态配置发现目标

- 支持多种图形和仪表板

# 

## 架构

![prometheus-architecture](../images/prometheus-architecture.png)

## 资料

- [Prometheus 官方文档](https://prometheus.io/docs/introduction/overview/)
- [谈谈时间序列数据库](https://blog.csdn.net/gentlezuo/article/details/103038523)
