# Spark 环境搭建

[官方文档](https://spark.apache.org/docs/2.4.8/cluster-overview.html)

Spark 目前支持几个集群管理器：

- `Standalone` – Spark 附带的简单集群管理器，可以轻松设置集群。
- `Apache Mesos` – 一个通用集群管理器，也可以运行 Hadoop MapReduce 和服务应用程序。
- `Hadoop YARN` – Hadoop 2 中的资源管理器。
- `Kubernetes`——一个用于自动部署、扩展和管理容器化应用程序的开源系统。

## Spark 架构图

![](../images/spark-cluster-overview.png)

## Spark Standalone Mode 环境搭建

[官方文档](https://spark.apache.org/docs/2.4.8/spark-standalone.html)

`docker-compose.yml`

```yaml
version: '2'

services:
  spark-master:
    image: docker.io/bitnami/spark:3.3
    container_name: spark-master
    environment:
      - SPARK_MODE=master
      - SPARK_RPC_AUTHENTICATION_ENABLED=no
      - SPARK_RPC_ENCRYPTION_ENABLED=no
      - SPARK_LOCAL_STORAGE_ENCRYPTION_ENABLED=no
      - SPARK_SSL_ENABLED=no
    ports:
      - '8080:8080'
      - '7077:7077'
  spark-worker:
    image: docker.io/bitnami/spark:3.3
    container_name: spark-worker
    environment:
      - SPARK_MODE=worker
      - SPARK_MASTER_URL=spark://spark:7077
      - SPARK_WORKER_MEMORY=1G
      - SPARK_WORKER_CORES=1
      - SPARK_RPC_AUTHENTICATION_ENABLED=no
      - SPARK_RPC_ENCRYPTION_ENABLED=no
      - SPARK_LOCAL_STORAGE_ENCRYPTION_ENABLED=no
      - SPARK_SSL_ENABLED=no
```


## 编写和运行 Java Spark 程序

```xml
<project>
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>hello-spark</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>

    <dependencies>
        <!-- https://mvnrepository.com/artifact/org.apache.spark/spark-core -->
        <dependency>
            <groupId>org.apache.spark</groupId>
            <!-- 这里的 2.12 指的是 scala 版本 -->
            <artifactId>spark-core_2.12</artifactId>
            <version>3.3.0</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-sql_2.12</artifactId>
            <version>3.3.0</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <artifactId>maven-assembly-plugin</artifactId>
                <version>3.0.0</version>
                <configuration>
                    <descriptorRefs>
                        <descriptorRef>jar-with-dependencies</descriptorRef>
                    </descriptorRefs>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

> 通过 `assembly` 插件把相关依赖打入一个 jar 包，这样 `spark-submit` 时会更简单些， spark 相关依赖设置为 `provided`，无需打入

```java
package org.example;

import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import scala.Tuple2;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

public class HelloWorld {
    public static void main(String[] args) {
        SparkConf conf = new SparkConf();
        // 本地调试使用
        // conf.setMaster("local");
        conf.setAppName("hello");

        JavaSparkContext sc = new JavaSparkContext(conf);
        JavaRDD<String> rdd = sc.parallelize(Arrays.asList("Hello World", "Hello Spark", "Hello Hadoop"));
        List<Tuple2<String, Integer>> result = rdd.flatMapToPair(row -> {
            String[] words = row.split("\\s");
            List<Tuple2<String, Integer>> wordPairs = new ArrayList<>();
            for (String word : words) {
                wordPairs.add(new Tuple2<>(word, 1));
            }
            return wordPairs.iterator();
        }).reduceByKey(Integer::sum).collect();
        System.out.println(result);
    }
}
```

```bash
# 打包
mvn clean package
# 把包 copy 到容器中
docker cp target/hello-spark-1.0-SNAPSHOT.jar spark-master:/opt/bitnami/app/app.jar
# 执行 spark 程序
docker exec -it spark-master sh -c 'spark-submit --class org.example.HelloWorld /opt/bitnami/app.jar'
```

> IDEA 本地调试可以使用，可以把 master 设置为 local

> 使用 spark-submit 可以兼容所有集群方式

## References

- [spart submit 文档](https://spark.apache.org/docs/2.4.8/submitting-applications.html)