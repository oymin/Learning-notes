# ElasticSearch 索引 API

**创建索引**

```json
PUT /my_index
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 0
  },
  "mappings": {
    "my_type1": {
      "properties": {
        "my_field": {
          "type": "text"
        }
      }
    }
  }
}
```

**修改索引**

```json
PUT /my_index/_settings
{
    "number_of_replicas": 1
}
```

**删除索引**

```bash
DELETE /my_index

DELETE /index_one,index_two

DELETE /index_*

DELETE /_all
```

**禁用删除所有索引配置**

```bash
# elasticsearch.yml
action.destructive_requires_name: true
```

## 倒排索引组成结构以及其索引可变原因

倒排索引，是适合用于进行搜索的

### 倒排索引的结构

（1）包含这个关键词的 document list
（2）包含这个关键词的所有 document 的数量：IDF（inverse document frequency）
（3）这个关键词在每个 document 中出现的次数：TF（term frequency）
（4）这个关键词在这个 document 中的次序
（5）每个 document 的长度：length norm
（6）包含这个关键词的所有 document 的平均长度

### 倒排索引不可变的好处

（1）不需要锁，提升并发能力，避免锁的问题
（2）数据不变，一直保存在 os cache 中，只要 cache 内存足够
（3）filter cache 一直驻留在内存，因为数据不变
（4）可以压缩，节省 cpu 和 io 开销

倒排索引不可变的坏处：每次都要重新构建整个索引

















