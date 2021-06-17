# RocketMQ vs Kafka

## 功能比较

![](../images/why-rocketmq.png)

## 选择

Kafka 吞吐量更优，且延迟也不差，社区更为庞大，问题解决方案更多，使用领域更广(作为数据管道等)，优先选择

Rocketmq 支持延时消息、广播消息、基于 tag 和 sql 服务端过滤、可视化运维等业务应用常用功能，如果有这些功能的需要，就使用 Rocketmq。