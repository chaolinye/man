# MyPerf4j

一个针对高并发、低延迟应用设计的高性能 Java 性能监控和统计工具

适用场景

- 启动时间优化

- 某个页面接口性能优化

- 采集监控指标

优势

- 无侵入性

## 基本用法

在 JVM 启动参数里加入以下两个参数

- `javaagent:/path/to/MyPerf4j-ASM.jar`

- `-DMyPerf4JPropFile=/path/to/MyPerf4J.properties`

```bash
java -javaagent:/path/to/MyPerf4J-ASM.jar -DMyPerf4JPropFile=/path/to/MyPerf4J.properties -jar yourApp.jar
```

`Myperf4j.properties`

```properties
# 配置监控应用的名称
app_name = MyApp

###############################################################################
#                           Metrics Configuration                             #
###############################################################################

# 配置 MetricsExporter 类型
#	log.stdout: 	以标准格式化结构输出到 stdout.log
#	log.standard: 	以标准格式化结构输出到磁盘
#	log.influxdb: 	以 InfluxDB LineProtocol 格式输出到磁盘
#	http.influxdb: 	以 InfluxDB LineProtocol 格式发送至 InfluxDB server
metrics.exporter = log.stdout

# 配置各项监控指标日志的文件路径
# 将方法指标写入独立的文件，以便于查看
metrics.log.method = /data/logs/MyPerf4J/method_metrics.log
metrics.log.class_loading = /data/logs/MyPerf4J/metrics.log
metrics.log.gc = /data/logs/MyPerf4J/metrics.log
metrics.log.memory = /data/logs/MyPerf4J/metrics.log
metrics.log.buff_pool = /data/logs/MyPerf4J/metrics.log
metrics.log.thread = /data/logs/MyPerf4J/metrics.log
metrics.log.file_desc = /data/logs/MyPerf4J/metrics.log
metrics.log.compilation = /data/logs/MyPerf4J/metrics.log

# 配置需要监控的package，可配置多个，用英文';'分隔
#   com.demo.p1 代表包含以 com.demo.p1 为前缀的所有包和类
#   [] 表示集合的概念：例如，com.demo.[p1,p2,p3] 代表包含以 com.demo.p1、com.demo.p2 和 com.demo.p3 为前缀的所有包和类，等价于 com.demo.p1;com.demo.p2;com.demo.p3
#   * 表示通配符：可以指代零个或多个字符，例如，com.*.demo.*
filter.packages.include = org.leaf.demo;
```

通过以下命令查看方法 metrics 报告

```bash
tail -100f /data/logs/MyPerf4J/method_metrics.log
```

## 存储到 InfluxDB

真实场景中，常见的做法是将 MyPerf4J 的监控指标存储到 InfluxDB 这类时间序列数据，然后通过 Grafana 进行展示

具体步骤如下:

- 修改 `MyPerf4J.properties` 配置文件

    - 配置 `metrics.exporter` 为 `http.influxdb`

    - 配置 `influxdb.host`、`influxdb.port`、`influxdb.database` 等配置项

    ```properties
    metrics.exporter = log.influxdb

    influxdb.host = 127.0.0.1
    influxdb.port = 12186
    influxdb.database = MyPerf4J_Test

    influxdb.username = admin
    influxdb.password = admin123

    # 配置超时时间，单位：ms
    influxdb.conn_timeout = 3000
    influxdb.read_timeout = 5000
    ```

- 安装启动 InfluxDB

    ```bash
    # 启动 InfluxDB 容器
    docker pull influxdb:1.8
    docker run -d -it --name influxdb -p 12186:8086 influxdb:1.8

    # 创建数据库
    docker exec -it influxdb bash
    influx
    create database MyPerf4J_Test
    ```

- 配置 Grafana

    [文档](https://github.com/LinShunKang/MyPerf4J/wiki/Grafana_)


## References

- [官方 github](https://github.com/LinShunKang/MyPerf4J)
- [配置模板](https://github.com/LinShunKang/Objects/blob/master/jars/MyPerf4J-3.x.properties)