# 常用命令

## 主题相关

```bash
# 创建主题
bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --topic quickstart-events --partitions 2 --replication-factor 2

# 查看所有主题列表
bin/kafka-topics.sh --bootstrap-server localhost:9092 --list

# 查看所有主题详情，分区、复制等信息
bin/kafka-topics.sh --bootstrap-server localhost:9092 --describe
# 查看某个主题详情
bin/kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic quickstart-events

# 修改主题
bin/kafka-topics.sh --bootstrap-server localhost:9092 --alter --topic quickstart-events --partitions 3

# 删除主题
bin/kafka-topics.sh --bootstrap-server localhost:9092 --delete --topic quickstart-events
```

## 消费者相关

```bash
# 查看所有消费者组的详情
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --describe --all-groups

# 创建消费者
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --group first-group --topic quickstart-events
```

## 生产者相关

```bash
# 创建生产者
bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic quickstart-events
```