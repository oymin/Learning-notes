# Kibana的 DevTools 命令使用

## 简单的集群管理

ES 提供了一套 API，叫做 `cat` API，可以查看 ES 中各种各样的数据。

**1、快速检查集群的健康状况**

```html
GET /_cat/health?v
```

如何快速了解集群的健康状况? `green` 、 `yellow` 、 `red`?

- green：每个索引的 primary shard 和 replica shard 都是 active 状态的
- yellow：每个索引的 primary shard 都是 active 状态的，但是部分 replica shard 不是 active 状态，处于不可用的状态
- red：不是所有索引的 primary shard 都是 active 状态的，部分索引有数据丢失了

我们现在就一个笔记本电脑，就启动了一个 ES 进程，相当于就只有一个 node。现在 ES 中有一个 index，就是 kibana 自己内置建立的 index。由于默认的配置是给每个 index 分配5个 primary shard 和5个 replica shard ，而且 primary shard 和 replica shard 不能在同一台机器上（为了容错）。现在 kibana 自己建立的 index 是1个 primary shard 和1个 replica shard。当前就一个 node，所以只有1个 primary shard 被分配了和启动了，但是一个 replica shard 没有第二台机器去启动。

做一个小实验：此时只要启动第二个 ES 进程，就会在 ES 集群中有2个 node，然后那1个 replica shard 就会自动分配过去，然后 cluster status 就会变成 green 状态。

**2、快速查看集群中有哪些索引**

```html
GET /_cat/indicES?v
```

**3、简单的索引操作**

```bash
# 创建test_index索引
PUT /test_index?pretty

# 删除test_index索引
DELETE /test_index?pretty
```

## Document 的 CRUD API

**新增文档，建立索引**

语法：`PUT /\<index\>/\<type\>/\<id\>`

```json
PUT /ecommerce/product/1
{
    "name" : "gaolujie yagao",
    "desc" :  "gaoxiao meibai",
    "price" :  30,
    "producer" :      "gaolujie producer",
    "tags": [ "meibai", "fangzhu" ]
}

PUT /ecommerce/product/2
{
    "name" : "jiajieshi yagao",
    "desc" : "youxiao fangzhu",
    "price" : 25,
    "producer" : "jiajieshi producer",
    "tags" : [ "fangzhu" ]
}
```

ES 会自动建立 index 和 type，不需要提前创建，而且 es 默认会对 document 每个 field 都建立倒排索引，让其可以被搜索。

**检索文档**

语法：`GET /\<index\>/\<type\>/\<id\>`

```html
GET /ecommerce/product/1
```

**替换文档**

语法：`PUT /\<index\>/\<type\>/\<id\>`

```json
PUT /ecommerce/product/1
{
    "name" : "jiaqiangban gaolujie yagao",
    "desc" :  "gaoxiao meibai",
    "price" :  30,
    "producer" :      "gaolujie producer",
    "tags": [ "meibai", "fangzhu" ]
}
```

**更新文档**

语法：`POST /\<index\>/\<type\>/\<id\>/_update`

```json
POST /ecommerce/product/1/_update
{
  "doc": {
    "name": "jiaqiangban gaolujie yagao"
  }
}
```

**删除文档**

语法：`DELETE /\<index\>/\<type\>/\<id\>`

```html
DELETE /ecommerce/product/1
```

## 搜索

1. query string search
2. query DSL
3. query filter
4. full-text search
5. phrase search
6. highlight search

### 1、query string search

```bash
# 搜索全部商品：
GET /ecommerce/product/_search

# 搜索商品名称中包含yagao的商品，而且按照售价降序排序：
GET /ecommerce/product/_search?q=name:yagao&sort=price:desc
```

query string search 的由来，因为 search 参数都是以 http 请求的 query string 来附带的。

适用于临时的在命令行使用一些工具，比如 curl，快速的发出请求，来检索想要的信息；但是如果查询请求很复杂，是很难去构建的。

在生产环境中，几乎很少使用 query string search

### 2、query DSL

DSL：Domain Specified Language 特定领域的语言

http request body：请求体，可以用 json 的格式来构建查询语法，比较方便，可以构建各种复杂的语法，比 query string search 肯定强大多了

```bash
#查询所有的商品
GET /ecommerce/product/_search
{
  "query": { "match_all": {} }
}

#查询名称包含yagao的商品，同时按照价格降序排序
GET /ecommerce/product/_search
{
    "query" : {
        "match" : {
            "name" : "yagao"
        }
    },
    "sort": [
        { "price": "desc" }
    ]
}

#分页查询商品，总共3条商品，假设每页就显示1条商品，现在显示第2页，所以就查出来第2个商品
GET /ecommerce/product/_search
{
  "query": { "match_all": {} },
  "from": 1,
  "size": 1
}

#指定要查询出来商品的名称和价格就可以
GET /ecommerce/product/_search
{
  "query": { "match_all": {} },
  "_source": ["name", "price"]
}
```

更加适合生产环境的使用，可以构建复杂的查询。

### 3、query filter

query filter：查询条件过滤器 

```bash
# 搜索商品名称包含yagao，而且售价大于25元的商品
GET /ecommerce/product/_search
{
    "query" : {
        "bool" : {
            "must" : {
                "match" : {
                    "name" : "yagao"
                }
            },
            "filter" : {
                "range" : {
                    "price" : { "gt" : 25 }
                }
            }
        }
    }
}
```

### 4、full-text search（全文检索）

```bash
GET /ecommerce/product/_search
{
    "query" : {
        "match" : {
            "producer" : "yagao zhonghua"
        }
    }
}
```

### 5、phrase search（短语搜索）

跟全文检索相对应，相反，全文检索会将输入的搜索串拆解开来，去倒排索引里面去一一匹配，只要能匹配上任意一个拆解后的单词，就可以作为结果返回。

phrase search 要求输入的搜索串，必须在指定的字段文本中，完全包含一模一样的，才可以算匹配，才能作为结果返回

```bash
GET /ecommerce/product/_search
{
    "query" : {
        "match_phrase" : {
            "producer" : "yagao producer"
        }
    }
}
```

### 6、highlight search（高亮搜索结果）

```bash
GET /ecommerce/product/_search
{
    "query" : {
        "match" : {
            "producer" : "producer"
        }
    },
    "highlight": {
        "fields" : {
            "producer" : {}
        }
    }
}
```

## 聚合操作

首先需要将文本 field 的 fielddata 属性设置为 true

```ash
PUT /ecommerce/_mapping/product
{
  "properties": {
    "tags": {
      "type": "text",
      "fielddata": true
    }
  }
}
```

第一个分析需求：计算每个 `tag` 下的商品数量

```bash
GET /ecommerce/product/_search
{
  "aggs": {
    "group_by_tags": {
      "terms": { "field": "tags" }
    }
  }
}
```

第二个聚合分析的需求：对名称中包含 yagao 的商品，计算每个 tag 下的商品数量

```bash
GET /ecommerce/product/_search
{
  "size": 0,
  "query": {
    "match": {
      "name": "yagao"
    }
  },
  "aggs": {
    "all_tags": {
      "terms": {
        "field": "tags"
      }
    }
  }
}
```

第三个聚合分析的需求：先分组，再算每组的平均值，计算每个 tag 下的商品的平均价格

```bash
GET /ecommerce/product/_search
{
    "size": 0,
    "aggs" : {
        "group_by_tags" : {
            "terms" : { "field" : "tags" },
            "aggs" : {
                "avg_price" : {
                    "avg" : { "field" : "price" }
                }
            }
        }
    }
}
```

第四个数据分析需求：计算每个 tag 下的商品的平均价格，并且按照平均价格降序排序

```bash
GET /ecommerce/product/_search
{
    "size": 0,
    "aggs" : {
        "all_tags" : {
            "terms" : { "field" : "tags", "order": { "avg_price": "desc" } },
            "aggs" : {
                "avg_price" : {
                    "avg" : { "field" : "price" }
                }
            }
        }
    }
}
```

第五个数据分析需求：按照指定的价格范围区间进行分组，然后在每组内再按照 tag 进行分组，最后再计算每组的平均价格

```bash
GET /ecommerce/product/_search
{
  "size": 0,
  "aggs": {
    "group_by_price": {
      "range": {
        "field": "price",
        "ranges": [
          {
            "from": 0,
            "to": 20
          },
          {
            "from": 20,
            "to": 40
          },
          {
            "from": 40,
            "to": 50
          }
        ]
      },
      "aggs": {
        "group_by_tags": {
          "terms": {
            "field": "tags"
          },
          "aggs": {
            "average_price": {
              "avg": {
                "field": "price"
              }
            }
          }
        }
      }
    }
  }
}
```


















