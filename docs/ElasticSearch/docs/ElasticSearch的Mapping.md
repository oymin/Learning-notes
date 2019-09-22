# ElasticSearch 的 Mapping

自动或手动为 index 中的 type 建立的一种数据结构和相关配置，简称为 mapping

1. 往 es 里面直接插入数据，es 会自动建立索引，同时建立 type 以及对应的 mapping

2. mapping 中就自动定义了每个 field 的数据类型

3. 不同的数据类型（比如说 text 和 date），可能有的是 exact value，有的是 full text

4. exact value，在建立倒排索引的时候，分词的时候，是将整个值一起作为一个关键词建立到倒排索引中的；full text，会经历各种各样的处理，分词，normaliztion（时态转换，同义词转换，大小写转换），才会建立到倒排索引中

5. 同时呢，exact value 和 full text 类型的 field 就决定了，在一个搜索过来的时候，对 exact value field 或者是 full text field 进行搜索的行为也是不一样的，会跟建立倒排索引的行为保持一致；比如说 exact value 搜索的时候，就是直接按照整个值进行匹配，full text query string，也会进行分词和 normalization 再去倒排索引中去搜索

6. 可以用 es 的 dynamic mapping，让其自动建立 mapping，包括自动设置数据类型；也可以提前手动创建 index 和 type 的 mapping，自己对各个 field 进行设置，包括数据类型，包括索引行为，包括分词器，等等

mapping，就是 index 的 type 的元数据，每个 type 都有一个自己的 mapping，决定了数据类型，建立倒排索引的行为，还有进行搜索的行为

dynamic mapping 自动为我们建立 index，创建 type，以及 type 对应的 mapping，mapping中包含了每个 field 对应的数据类型，以及如何分词等设置。

插入几条数据，让 es 自动为我们建立一个索引：

```bash
PUT /website/article/1
{
  "post_date": "2017-01-01",
  "title": "my first article",
  "content": "this is my first article in this website",
  "author_id": 11400
}

PUT /website/article/2
{
  "post_date": "2017-01-02",
  "title": "my second article",
  "content": "this is my second article in this website",
  "author_id": 11400
}

PUT /website/article/3
{
  "post_date": "2017-01-03",
  "title": "my third article",
  "content": "this is my third article in this website",
  "author_id": 11400
}
```

尝试各种搜索:

```bash
GET /website/article/_search?q=2017         # 3条结果
GET /website/article/_search?q=2017-01-01   # 3条结果
GET /website/article/_search?q=post_date:2017-01-01     # 1条结果
GET /website/article/_search?q=post_date:2017           # 1条结果
```

搜索结果为什么不一致，因为 es 自动建立 mapping 的时候，设置了不同的 field 不同的 data type。不同的 data type 的分词、搜索等行为是不一样的。所以出现了 `_all field` 和 `post_date field` 的搜索表现完全不一样。

```bash
# 获取 mapping 关系映射
GET /website/_mapping/article

{
  "website": {
    "mappings": {
      "article": {
        "properties": {
          "author_id": {
            "type": "long"
          },
          "content": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          },
          "post_date": {
            "type": "date"
          },
          "title": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 256
              }
            }
          }
        }
      }
    }
  }
}
```

## 定制化自己的 dynamic mapping 策略

### 定制 dynamic 策略

- true：遇到陌生字段，就进行 dynamic mapping
- false：遇到陌生字段，就忽略
- strict：遇到陌生字段，就报错

```bash
PUT /my_index
{
  "mappings": {
    "my_type": {
      "dynamic": "strict", # 全局的
      "properties": {
        "title": {
          "type": "text"
        },
        "address": {
          "type": "object",
          "dynamic": "true" # field
        }
      }
    }
  }
}
```

### 定制 dynamic mapping 策略

#### date_detection

默认会按照一定格式识别 date，比如 `yyyy-MM-dd`。但是如果某个 field 先过来一个 2017-01-01 的值，就会被自动 dynamic mapping 成 date，后面如果再来一个 "hello world" 之类的值，就会报错。可以手动关闭某个type 的 date_detection，如果有需要，自己手动指定某个 field 为 date 类型。

```bash
PUT /my_index/_mapping/my_type
{
    "date_detection": false
}
```

#### 定制自己的 dynamic mapping template（type level）

```json
PUT /my_index
{
  "mappings": {
    "my_type": {
      "dynamic_templates": [
          {
            "en": {
                // 新增 document 中的 field 名字以 _en 结尾的，会使用下面的 mapping
                "match":              "*_en",
                "match_mapping_type": "string",
                "mapping": {
                    //类型设置为 string
                    "type":           "string",
                    //分词器设置为 english
                    "analyzer":       "english"
                }
            }
          }
      ]
    }
  }
}
```

测试:

```json
//添加数据
PUT /my_index/my_type/1
{
  "title": "this is my first article"
}

PUT /my_index/my_type/2
{
  "title_en": "this is my first article"
}

//查询
GET /my_index/my_type/_search
{
  "query": {
    "match": {
      "title" : "is"
    }
  }
}

GET /my_index/my_type/_search
{
  "query": {
    "match": {
      "title_en" : "is"
    }
  }
}
```

`title` 没有匹配到任何的 dynamic 模板，默认就是 standard 分词器，不会过滤停用词，is 会进入倒排索引，用i s 来搜索是可以搜索到的。

`title_en` 匹配到了 dynamic 模板，就是 english 分词器，会过滤停用词，is 这种停用词就会被过滤掉，用 is 来搜索就搜索不到了。

#### 定制自己的 default mapping template（index level）

```json
PUT /my_index
{
    "mappings": {
        "_default_": {
            "_all": { "enabled":  false }
        },
        "blog": {
            "_all": { "enabled":  true  }
        }
    }
}
```




