# Elasticsearch in Action

## Introducing Elasticsearch

### Elasticsearch as a search engine

#### Providing quick searches

Elasticsearch 使用 Lucene （一个高性能搜索引擎库）来索引数据。

Lucene 通过倒排索引来索引数据

#### Ensuring relevant results

Elasticsearch 会计算文档的相关度分数，然后根据分数排序返回文档列表

默认使用 TF-IDF（term frequency-inverse document frequency） 算法来计算文档的相关度。

- Term frequency：这个词在这个文档中出现的次数，越高，相关度分数越高
- Inverse document frequency：这个词在其他文档出现越少，相关度分数越高

#### Searching beyond exact matches

- Handling typos： 处理打字错误，可以使用 fuzzy queries 来实现，可以容忍一定编辑距离的打字错误。
- Supporting derivatives：支持衍生词，比如 bicyle 可以匹配到 bicyclist 和 cycling
- Using Statistics: 可以通过 aggregations 做数据分析
- providing suggestion: 使用 前缀、通配符，正则表达式等查询方式支持提供搜索建议。

### Typical setups using Elasticsearch

#### One-stop shop for storing, searching, and statistics

![](../images/es-setup-1.png)

#### Plugin search in a complex system 

![](../images/es-setup-2.png)

#### Use it with existing tools 

![](../images/es-setup-3.png)

### Data structure and interaction

#### Understanding indexing and search functionality 

Elasticsearch 基于 Lucene 构建，额外提供以下能力：

- Robust caching
- HTTP API
- Backward-compatibility

#### Analysis

The default analyzer first breaks text into words by looking for common word separators, such as a space or a comma. Then, it lowercases those words.

![](../images/es-analysis-1.png)

#### Structuring your data in Elasticsearch

Unlike a relational database, which stores data in records or rows, Elasticsearch stores data in 
documents.

The difference is that a document is more flexible than a record mainly because, in 
Elasticsearch at least, a document can be hierarchical.

ES 中的 Index 类似于关系数据库中的 table；Document 类似于关系数据库中的 row。

> 老版本 ES 中类似于 table 的是 mapping type，而 index 类似于 database，但是新版本已经去掉 mapping type，因为 mapping type 本质上没有隔离性，同一个 index 的不同 mapping type 的同名字段只能是相同类型，并不同于 table 的概念，有致命缺陷，故已经删除 mapping type 的概念了，index 降级对标 table

### Performance and scaling

#### SCALING, SHARDING, AND PERFORMANCE

A cluster is made up of one or more nodes, where each node is an Elasticsearch process that 
typically runs on a separate server. Elasticsearch is clustered by default。

Elasticsearch divides every index into multiple chunks called shards. By default, each index has five shards. 

Each node can receive all kinds of requests、

![](../images/es-shard-1.png)

Shards can be moved around, allowing you to expand or shrink your cluster at any time.

To increase availability, you can create one or more copies (called replicas) for each of your initial shards (called primaries).Primaries differ from replicas in that they’re the first to receive new documents. Other than that, they’re the same

![](../images/es-shard-2.png)

### Downloading and starting Elasticsearch

[Docker 按照 ElasticSearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html#_configure_and_start_the_cluster)
