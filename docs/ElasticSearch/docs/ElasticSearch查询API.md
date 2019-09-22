# ElasticSearch 查询 API

## 1. multi-index 和 multi-type 搜索模式

告诉你如何一次性搜索多个 index 和多个 type 下的数据

- `/_search`： 所有索引，所有 type 下的所有数据都搜索出来

- `/index1/_search`： 指定一个index，搜索其下所有 type 的数据

- `/index1,index2/_search`： 同时搜索两个 index 下的数据

- `/*1,*2/_search`： 按照通配符去匹配多个索引

- `/index1/type1/_search`： 搜索一个index下指定的 type 的数据

- `/index1/type1,type2/_search`： 可以搜索一个 index 下多个 type 的数据

- `/index1,index2/type1,type2/_search`： 搜索多个 index 下的多个 type 的数据

- `/_all/type1,type2/_search`： _all 可以代表搜索所有 index 下的指定 type 的数据

```bash
GET /test_index/test_type/_search
```

## 分页搜索

关键参数：

- size
- from

```bash
GET /_search?size=10
GET /_search?size=10&from=0
GET /_search?size=10&from=20
```

## 2. query string

语法：`?q=field_name:keyword`

```bash
GET /test_index/test_type/_search?q=test_field:test

# + 号表示filed必须包含关键词
GET /test_index/test_type/_search?q=+test_field:test

# - 号表示filed不能包含关键词
GET /test_index/test_type/_search?q=-test_field:test
```

**_all metadata 的原理和作用:**

```bash
GET /test_index/test_type/_search?q=test
```

直接可以搜索所有的 field，任意一个 field 包含指定的关键字就可以搜索出来。我们在进行中搜索的时候，难道是对 document 中的每一个 field 都进行一次搜索吗？不是的。

e s中的 _all 元数据，在建立索引的时候，我们插入一条 document，它里面包含了多个 field，此时，es 会自动将多个 field 的值，全部用字符串的方式串联起来，变成一个长的字符串，作为 _all field 的值，同时建立索引

后面如果在搜索的时候，没有对某个 field 指定搜索，就默认搜索 _all field，其中是包含了所有 field 的值的

**举个例子**

```bash
{
  "name": "jack",
  "age": 26,
  "email": "jack@sina.com",
  "address": "guamgzhou"
}
```

`jack 26 jack@sina.com guangzhou`，作为这一条 document 的 _all field 的值，同时进行分词后建立对应的倒排索引。

生产环境不使用。

### 2.1 query string 分词

- query string 必须以和 index 建立时相同的 analyzer 进行分词
- query string 对 exact value 和 full text 的区别对待

## 3. Query DSL 搜索语法

```bash
# 查询 test_field 包含 test
GET /test_index/test_type/_search
{
  "query": {
    "match": {
      "test_field": "test"
    }
  }
}
```

**搜索 title 必须包含 elasticsearch，content 可以包含 elasticsearch 也可以不包含，author_id 必须不为 111 :**

```bash
GET /website/article/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "title": "elasticsearch" } }
      ],
      "should": [
        { "match": { "content": "elasticsearch" } }
      ],
      "must_not": [
        { "match": { "author_id": "111" } }
      ]
    },
    //最少匹配1条
    "minimum_should_match": 1
  }
}
```

## 4. filter 与 query 对比

- **filter：** 仅仅只是按照搜索条件过滤出需要的数据而已，不计算任何相关度分数，对相关度没有任何影响

- **query：** 会去计算每个 document 相对于搜索条件的相关度，并按照相关度进行排序

**示例:**

```bash
GET /company/employee/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "join_date": "2016-01-01"
          }
        }
      ],
      "filter": {
        "range": {
          "age": {
            "gte": 30
          }
        }
      }
    }
  }
}
```

一般来说，如果你是在进行搜索，需要将最匹配搜索条件的数据先返回，那么用 query；如果你只是要根据一些条件筛选出一部分数据，不关注其排序，那么用 filter

除非是你的这些搜索条件，你希望越符合这些搜索条件的 document 越排在前面返回，那么这些搜索条件要放在 query 中；如果你不希望一些搜索条件来影响你的 document 排序，那么就放在 filter 中即可。

**filter 与 query 性能**

- filter 不需要计算相关度分数，不需要按照相关度分数进行排序，同时还有内置的自动 cache最常使用 filter 的数据
- query 相反，要计算相关度分数，按照分数进行排序，而且无法 cache 结果

## 5. query 查询类型参数

- **match_all**

  ```bash
  GET /_search
  {
      "query": {
          "match_all": {}
      }
  }
  ```

- **match**

  ```bash
  GET /_search
  {
      "query": { "match": { "title": "my elasticsearch article" }}
  }
  ```

- **match_phrase** 匹配短语,必须完全包含有要查询的内容

- **multi match**

  ```bash
  GET /test_index/test_type/_search
  {
    "query": {
      "multi_match": {
        "query": "test",
        "fields": ["test_field", "test_field1"]
      }
    }
  }
  ```

- **range**

  ```bash
  GET /company/employee/_search
  {
    "query": {
      "range": {
        "age": {
          "gte": 30
        }
      }
    }
  }
  ```

- **term**

  ```bash
  # term 不会对 "test hello" 条件分词
  GET /test_index/test_type/_search
  {
    "query": {
      "term": {
        "test_field": "test hello"
      }
    }
  }
  ```

- **terms**

  ```bash
  GET /_search
  {
      "query": { "terms": { "tag": [ "search", "full_text", "nosql" ] }}
  }
  ```

### 组合查询

```bash
GET /website/article/_search
{
  "query": {
    "bool": {
      "must":     { "match": { "title": "how to make millions" }},
      "must_not": { "match": { "tag":   "spam" }},
      "should": [
          { "match": { "tag": "starred" }}
      ],
      "filter": {
        "bool": {
            "must": [
                { "range": { "date": { "gte": "2014-01-01" }}},
                { "range": { "price": { "lte": 29.99 }}}
            ],
            "must_not": [
                { "term": { "category": "ebooks" }}
            ]
        }
      }
    }
  }
}
```

每个子查询都会计算一个 document 针对它的相关度分数，然后 bool 综合所有分数，合并为一个分数，当然 filter 是不会计算分数的

## 6. 如何定位不合法的搜索以及其原因

使用 explain 验证

```bash
GET /test_index/test_type/_validate/query?explain
{
  "query": {
    "match": {
      "test_field": "test"
    }
  }
}

#---------------- result ------------------
{
  "valid": true,
  "_shards": {
    "total": 1,
    "successful": 1,
    "failed": 0
  },
  "explanations": [
    {
      "index": "test_index",
      "valid": true,
      "explanation": "+test_field:test #(#_type:test_type)"
    }
  ]
}
```

一般用在那种特别复杂庞大的搜索下，比如你一下子写了上百行的搜索，这个时候可以先用 validate api 去验证一下，搜索是否合法。

## 7. 排序

### 默认排序规则

默认情况下, 是按照 _score 降序排序的, 然而某些情况下, 可能没有有用的 _score, 比如说 filter 。

```bash
GET /_search
{
    "query" : {
        "bool" : {
            "filter" : {
                "term" : {
                    "author_id" : 1
                }
            }
        }
    }
}

# 当然，也可以是constant_score

GET /_search
{
    "query" : {
        "constant_score" : {
            "filter" : {
                "term" : {
                    "author_id" : 1
                }
            }
        }
    }
}
```

### 定制排序规则

```bash
GET /company/employee/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "range": {
          "age": {
            "gte": 30
          }
        }
      }
    }
  },
  "sort": [
    {
      "join_date": {
        "order": "asc"
      }
    }
  ]
}
```

### 如何将一个 field 索引两次来解决字符串排序问题

如果对一个 string field 进行排序，结果往往不准确，因为分词后是多个单词，再排序就不是我们想要的结果了

通常解决方案是，将一个 string field 建立两次索引，一个分词，用来进行搜索；一个不分词，用来进行排序

```bash
# 添加索引
PUT /website
{
  "mappings": {
    "article": {
      "properties": {
        "title": {
          "type": "text",
          "fields": {
            "raw": {
              "type": "keyword",
              "index": false
            }
          },
          "fielddata": true
        },
        "content": {
          "type": "text"
        },
        "post_date": {
          "type": "date"
        },
        "author_id": {
          "type": "long"
        }
      }
    }
  }
}

# 添加文档
PUT /website/article/3
{
  "title": "thrid article",
  "content": "this is my second article",
  "post_date": "2017-01-01",
  "author_id": 110
}

PUT /website/article/2
{
  "title": "second article",
  "content": "this is my second article",
  "post_date": "2017-01-01",
  "author_id": 110
}
```

**查询：**

```bash
GET /website/article/_search
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "title.raw": {
        "order": "desc"
      }
    }
  ]
}

# 结果中的排序字段没有被分词
"sort" : [
  "second article"
]
```

## 8. scroll 技术滚动搜索大量数据

如果一次性要查出来比如10万条数据，那么性能会很差，此时一般会采取用 scroll 滚动查询，一批一批的查，直到所有数据都查询完处理完

使用 scroll 滚动搜索，可以先搜索一批数据返回 scroll_id，然后使用 scroll_ud 再搜索下一批数据，以此类推，直到搜索出全部的数据来

scroll 搜索会在第一次搜索的时候，保存一个当时的视图快照，之后只会基于该旧的视图快照提供数据搜索，如果这个期间数据变更，是不会让用户看到的

采用基于 _doc 进行排序的方式性能较高,每次发送 scroll 请求，我们还需要指定一个 scroll 参数，指定一个时间窗口，每次搜索请求只要在这个时间窗口内能完成就可以了。

```json
GET /test_index/test_type/_search?scroll=1m
{
  "query": {
    "match_all": {}
  },
  "sort": [ "_doc" ],
  "size": 3
}

# 获得的结果会有一个 scoll_id，下一次再发送scoll请求的时候，必须带上这个scoll_id
{
  "_scroll_id" : "DnF1ZXJ5VGhlbkZldGNoBQAAAAAAACcbFlh2VW9mNG91UV9TQlV4X216ZzhsY0EAAAAAAAAnHBZYdlVvZjRvdVFfU0JVeF9temc4bGNBAAAAAAAAJx0WWHZVb2Y0b3VRX1NCVXhfbXpnOGxjQQAAAAAAACceFlh2VW9mNG91UV9TQlV4X216ZzhsY0EAAAAAAAAnHxZYdlVvZjRvdVFfU0JVeF9temc4bGNB",
}

GET /_search/scroll
{
    "scroll": "1m",
    "scroll_id" : "DnF1ZXJ5VGhlbkZldGNoBQAAAAAAACcbFlh2VW9mNG91UV9TQlV4X216ZzhsY0EAAAAAAAAnHBZYdlVvZjRvdVFfU0JVeF9temc4bGNBAAAAAAAAJx0WWHZVb2Y0b3VRX1NCVXhfbXpnOGxjQQAAAAAAACceFlh2VW9mNG91UV9TQlV4X216ZzhsY0EAAAAAAAAnHxZYdlVvZjRvdVFfU0JVeF9temc4bGNB"
}
```
















