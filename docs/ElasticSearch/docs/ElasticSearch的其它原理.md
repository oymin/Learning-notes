# ElasticSearch的其它原理

## 写一致性原理

### consistency 一致性

属性：

- **one**（primary shard）: 要求我们这个写操作，只要有一个 primary shard 是 active 活跃可用的，就可以执行

- **all**（all shard）: 要求我们这个写操作，必须所有的 primary shard 和 replica shard 都是活跃的，才可以执行这个写操作

- **quorum**（default）: 默认的值，要求所有的 shard 中，必须是大部分的 shard 都是活跃的，可用的，才可以执行这个写操作

我们在发送任何一个增删改操作的时候，比如说 `put /index/type/id`，都可以带上一个 consistency 参数，指明我们想要的写一致性是什么？

```bash
put /index/type/id?consistency=quorum
```

### quorum 机制 

写之前必须确保大多数 shard 都可用，quroum = `int( (primary + number_of_replicas) / 2 ) + 1` ，而且不能 quroum 不能小于 1 。

**举个例子：**

    3 个 primary shard，number_of_replicas = 1，总共有 3 + 3 * 1 = 6 个 shard ：quorum = int( (3 + 1) / 2 ) + 1 = 3

    所以，要求 6 个 shard 中至少有 3 个 shard 是 active 状态的，才可以执行这个写操作。

**如果节点数少于 quorum 数量，可能导致 quorum 不齐全，进而导致无法执行任何写操作**

    3个 primary shard，replica = 1，要求至少 3 个 shard 是 active，3 个 shard 按照之前学习的 shard & replica 机制，必须在不同的节点上，如果说只有 2 台机器的话，是不是有可能出现说，3 个 shard 都没法分配齐全，此时就可能会出现写操作无法执行的情况。

    es 提供了一种特殊的处理场景，就是说当 number_of_replicas > 1 时才生效，因为假如说，你就一个 primary shard，replica = 1，此时就 2 个 shard。

    ((1 + 1) / 2) + 1 = 2，要求必须有 2 个 shard 是活跃的，但是可能就 1 个 node，此时就 1 个 shard 是活跃的，如果你不特殊处理的话，导致我们的单节点集群就无法工作。

**quorum 不齐全时，wait，默认 1 分钟，timeout，100，30s**

    等待期间，期望活跃的 shard 数量可以增加，最后实在不行，就会 timeout
    我们其实可以在写操作的时候，加一个 timeout 参数，比如说 put /index/type/id?timeout=30，这个就是说自己去设定 quorum 不齐全的时候，es 的 timeout 时长，可以缩短，也可以增长。

## 什么是分词器

切分词语 和 normalization（提升 recall 召回率）

- 分词器：给你一段句子，然后将这段句子拆分成一个一个的单个的单词，同时对每个单词进行 normalization（时态转换，单复数转换）
- recall：召回率：搜索的时候，增加能够搜索到的结果的数量

处理步骤：

- **character filter：** 在一段文本进行分词之前，先进行预处理，比如说最常见的就是，过滤 html 标签（）

    ```html
    <span>hello<span> 转换 hello

    & 转换 and

    I&you 转换 I and you
    ```

- **tokenizer：** 分词，hello you and me 转换 hello, you, and, me

- **token filter：** lowercase，stop word，synonymom，dogs 转换 dog，liked 转换 like，Tom 转换 tom，mother 转换 mom，small 转换 little

一个分词器，很重要，将一段文本进行各种处理，最后处理好的结果才会拿去建立倒排索引。

### 内置分词器的介绍

Set the shape to semi-transparent by calling set_trans(5)

- **standard analyzer：**

    set, the, shape, to, semi, transparent, by, calling, set_trans, 5（默认的是standard）

- **simple analyzer：**

    set, the, shape, to, semi, transparent, by, calling, set, trans

- **whitespace analyzer：**

    Set, the, shape, to, semi-transparent, by, calling, set_trans(5)

- **language analyzer**（特定的语言的分词器，比如说，english，英语分词器）：

     set, shape, semi, transpar, call, set_tran, 5

**默认的分词器 standard**

- standard tokenizer：以单词边界进行切分
- standard token filter：什么都不做
- lowercase token filter：将所有字母转换为小写
- stop token filer（默认被禁用）：移除停用词，比如a the it等等

**修改分词器的设置**

```bash
#启用english停用词token filter
PUT /my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "es_std": {
          "type": "standard",
          "stopwords": "_english_"
        }
      }
    }
  }
}
```

**定制化自己的分词器**

```bash
PUT /my_index
{
  "settings": {
    "analysis": {
      "char_filter": {
        "&_to_and": {
          "type": "mapping",
          "mappings": ["&=> and"]
        }
      },
      "filter": {
        "my_stopwords": {
          "type": "stop",
          "stopwords": ["the", "a"]
        }
      },
      "analyzer": {
        "my_analyzer": {
          "type": "custom",
          "char_filter": ["html_strip", "&_to_and"],
          "tokenizer": "standard",
          "filter": ["lowercase", "my_stopwords"]
        }
      }
    }
  }
}
```

测试分词器：

```bash
# 测试默认分词器的分词效果
GET /_analyze
{
  "analyzer": "standard",
  "text": "Text to analyze"
}

# 测试自定义分词器
GET /my_index/_analyze
{
  "text": "tom&jerry are a friend in the house, <a>, HAHA!!",
  "analyzer": "my_analyzer"
}
```

```bash
# type 受用指定分词器
PUT /my_index/_mapping/my_type
{
  "properties": {
    "content": {
      "type": "text",
      "analyzer": "my_analyzer"
    }
  }
}
```

## 相关度评分 TF & IDF 算法

relevance score (相关性得分) 算法，简单来说就是计算出一个索引中的文本，与搜索文本，他们之间的关联匹配程度。

Elasticsearch 使用的是 `term frequency/inverse document frequency` 算法，简称为 `TF/IDF 算法

**Term frequency：** 搜索文本中的各个词条在 field 文本中出现了多少次，出现次数越多，就越相关

**Inverse document frequency：** 搜索文本中的各个词条在整个索引的所有文档中出现了多少次，出现的次数越多，就越不相关

**Field-length norm：** field 长度，field 越长，相关度越弱

## 数据写入流程

1. 数据写入buffer缓冲和translog日志文件
2. 每隔一秒钟，buffer中的数据被写入新的segment file，并进入os cache，此时segment被打开并供search使用
3. buffer被清空
4. 重复1~3，新的segment不断添加，buffer不断被清空，而translog中的数据不断累加
5. 当translog长度达到一定程度的时候，commit操作发生
   - buffer中的所有数据写入一个新的segment，并写入os cache，打开供使用
   - buffer被清空
   - 一个commit ponit被写入磁盘，标明了所有的index segment
   - filesystem cache中的所有index segment file缓存数据，被fsync强行刷到磁盘上
   - 现有的translog被清空，创建一个新的translog

基于translog和commit point，如何进行数据恢复

fsync+清空translog，就是flush，默认每隔30分钟flush一次，或者当translog过大的时候，也会flush

`POST /my_index/_flush`，一般来说别手动flush，让它自动执行就可以了

translog 每隔5秒被 fsync 一次到磁盘上。在一次增删改操作之后，当 fsync 在 primary shard 和 replica shard 都成功之后，那次增删改操作才会成功

但是这种在一次增删改时强行 fsync translog 可能会导致部分操作比较耗时，也可以允许部分数据丢失，设置异步 fsync translog

```json
PUT /my_index/_settings
{
    "index.translog.durability": "async",
    "index.translog.sync_interval": "5s"
}
```
