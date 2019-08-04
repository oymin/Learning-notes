# ElasticSearch 介绍

## 1. 简介

ElasticSearch 是一个基于 Lucene 的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于 RESTful web 接口。Elasticsearch 是用 Java 语言开发的，并作为 Apache 许可条款下的开放源码发布，是一种流行的企业级搜索引擎。ElasticSearch 用于云计算中，能够达到实时搜索，稳定，可靠，快速，安装使用方便。官方客户端在 Java、.NET（C#）、PHP、Python、Apache Groovy、Ruby 和许多其他语言中都是可用的。根据 DB-Engines 的排名显示，Elasticsearch 是最受欢迎的企业搜索引擎，其次是 Apache Solr，也是基于 Lucene。

ElasticSearch 是一个分布式、高扩展、高实时的搜索与数据分析引擎。它能很方便的使大量数据具有搜索、分析和探索的能力。充分利用 ElasticSearch 的水平伸缩性，能使数据在生产环境变得更有价值。ElasticSearch 的实现原理主要分为以下几个步骤，首先用户将数据提交到 Elastic Search 数据库中，再通过分词控制器去将对应的语句分词，将其权重和分词结果一并存入数据，当用户搜索数据时候，再根据权重将结果排名，打分，再将返回结果呈现给用户。

Elasticsearch 是与名为 Logstash 的数据收集和日志解析引擎以及名为 Kibana 的分析和可视化平台一起开发的。这三个产品被设计成一个集成解决方案，称为 “Elastic Stack”（以前称为“ELK stack”）。

Elasticsearch 可以用于搜索各种文档。它提供可扩展的搜索，具有接近实时的搜索，并支持多租户。Elasticsearch 是分布式的，这意味着索引可以被分成分片，每个分片可以有0个或多个副本。每个节点托管一个或多个分片，并充当协调器将操作委托给正确的分片。再平衡和路由是自动完成的。“相关数据通常存储在同一个索引中，该索引由一个或多个主分片和零个或多个复制分片组成。一旦创建了索引，就不能更改主分片的数量。

Elasticsearch 使用 Lucene，并试图通过 JSON 和 Java API 提供其所有特性。它支持 facetting 和 percolating，如果新文档与注册查询匹配，这对于通知非常有用。另一个特性称为“网关”，处理索引的长期持久性；例如，在服务器崩溃的情况下，可以从网关恢复索引。Elasticsearch 支持实时 GET 请求，适合作为 NoSQL 数据存储，但缺少分布式事务。

## 2. ES原理与应用

### 2.1 索引与结构

下图是 ElasticSearch 的索引结构，下边黑色部分是物理结构，上边黄色部分是逻辑结构，逻辑结构也是为了更好的描述 ElasticSearch 的工作原理及去使用物理结构中的索引文件。

![jpg](..\images\01.jpg)

逻辑结构部分是一个倒排索引表：

1、将要搜索的文档内容分词，所有不重复的词组成分词列表。

2、将搜索的文档最终以 Document 方式存储起来。

3、每个词和 Docment 都有关联。

| term | Doc_1 | Doc_2 |
|------|:-----:|:-----:|
| Quick |  | x |
| the | x | |
| brown | x | x |
| dog | x | |
| dos | | x |
| in | x | x |
| jumped | x | x |
| lazy | x | x |
| leap | | x |

现在，如果我们想搜索 `quick brown` ，我们只需要查找包含每个词条的文档：

| term | Doc_1 | Doc_2 |
|:-----:|:-----:|:-----:|
| Quick |  | x |
| brown | x | x |

查找结果：

| Total | 1 | 2 |
|:-----:|:-----:|:-----:|
| &emsp;&emsp;| &emsp;&emsp;&emsp;&emsp; | &emsp;&emsp; |

两个文档都匹配，但是第 `Doc_2` 文档比 `Doc_1` 文档匹配度更高。
