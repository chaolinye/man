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

- Handling typos
- Supporting derivatives