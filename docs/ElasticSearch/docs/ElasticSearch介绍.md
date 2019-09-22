# ElasticSearch 介绍

**简介**

ElasticSearch 是一个基于 Lucene 的搜索服务器。它提供了一个分布式多用户能力的全文搜索引擎，基于 RESTful web 接口。Elasticsearch 是用 Java 语言开发的，并作为 Apache 许可条款下的开放源码发布，是一种流行的企业级搜索引擎。ElasticSearch 用于云计算中，能够达到实时搜索，稳定，可靠，快速，安装使用方便。官方客户端在 Java、.NET（C#）、PHP、Python、Apache Groovy、Ruby 和许多其他语言中都是可用的。根据 DB-EnginES 的排名显示，Elasticsearch 是最受欢迎的企业搜索引擎，其次是 Apache Solr，也是基于 Lucene。

ElasticSearch 是一个分布式、高扩展、高实时的搜索与数据分析引擎。它能很方便的使大量数据具有搜索、分析和探索的能力。充分利用 ElasticSearch 的水平伸缩性，能使数据在生产环境变得更有价值。ElasticSearch 的实现原理主要分为以下几个步骤，首先用户将数据提交到 Elastic Search 数据库中，再通过分词控制器去将对应的语句分词，将其权重和分词结果一并存入数据，当用户搜索数据时候，再根据权重将结果排名，打分，再将返回结果呈现给用户。

Elasticsearch 是与名为 Logstash 的数据收集和日志解析引擎以及名为 Kibana 的分析和可视化平台一起开发的。这三个产品被设计成一个集成解决方案，称为 “Elastic Stack”（以前称为“ELK stack”）。

Elasticsearch 可以用于搜索各种文档。它提供可扩展的搜索，具有接近实时的搜索，并支持多租户。Elasticsearch 是分布式的，这意味着索引可以被分成分片，每个分片可以有0个或多个副本。每个节点托管一个或多个分片，并充当协调器将操作委托给正确的分片。再平衡和路由是自动完成的。“相关数据通常存储在同一个索引中，该索引由一个或多个主分片和零个或多个复制分片组成。一旦创建了索引，就不能更改主分片的数量。

Elasticsearch 使用 Lucene，并试图通过 JSON 和 Java API 提供其所有特性。它支持 facetting 和 percolating，如果新文档与注册查询匹配，这对于通知非常有用。另一个特性称为“网关”，处理索引的长期持久性；例如，在服务器崩溃的情况下，可以从网关恢复索引。Elasticsearch 支持实时 GET 请求，适合作为 NoSQL 数据存储，但缺少分布式事务。

## Elasticsearch的功能

1. 分布式的搜索引擎和数据分析引擎
2. 全文检索，结构化检索，数据分析
3. 对海量数据进行近实时的处理
     - 分布式：ES 自动可以将海量数据分散到多台服务器上去存储和检索
     - 海量数据的处理：分布式以后，就可以采用大量的服务器去存储和检索数据，自然而然就可以实现海量数据的处理了

## Elasticsearch的特点

（1）可以作为一个大型分布式集群（数百台服务器）技术，处理PB级数据，服务大公司；也可以运行在单机上，服务小公司

（2）Elasticsearch 不是什么新技术，主要是将全文检索、数据分析以及分布式技术，合并在了一起，才形成了独一无二的ES；Lucene（全文检索），商用的数据分析软件（也是有的），分布式数据库（mycat）

（3）对用户而言，是开箱即用的，非常简单，作为中小型的应用，直接3分钟部署一下 ES，就可以作为生产环境的系统来使用了，数据量不大，操作不是太复杂

（4）数据库的功能面对很多领域是不够用的（事务，还有各种联机事务型的操作）；特殊的功能，比如全文检索，同义词处理，相关度排名，复杂数据分析，海量数据的近实时处理；Elasticsearch 作为传统数据库的一个补充，提供了数据库所不不能提供的很多功能

## ES原理与应用

### ElasticSearch的核心概念

- **Near Realtime（NRT）：** 近实时，两个意思，从写入数据到数据可以被搜索到有一个小延迟（大概1秒）；基于ES执行搜索和分析可以达到秒级

- **Cluster：** 集群，包含多个节点，每个节点属于哪个集群是通过一个配置（集群名称，默认是ElasticSearch）来决定的，对于中小型应用来说，刚开始一个集群就一个节点很正常

- **Node：** 节点，集群中的一个节点，节点也有一个名称（默认是随机分配的），节点名称很重要（在执行运维管理操作的时候），默认节点会去加入一个名称为 “ElasticSearch” 的集群，如果直接启动一堆节点，那么它们会自动组成一个 ElasticSearch 集群，当然一个节点也可以组成一个 ElasticSearch 集群

- **Document & Field：** Document 文档，ES 中的最小数据单元，一个 Document 可以是一条客户数据、一条商品分类数据、一条订单数据、通常用 JSON 数据结构表示，每个 Index 下的 Type 中，都可以去存储多个Document。一个 Document 里面有多个 Field，每个 Field 就是一个数据字段。

    ```json
    Product Document
    {
    "product_id": "1",
    "product_name": "高露洁牙膏",
    "product_dESc": "高效美白",
    "category_id": "2",
    "category_name": "日化用品"
    }
    ```

- **Index：** 索引，包含一堆有相似结构的文档数据，比如可以有一个客户索引，商品分类索引，订单索引，索引有一个名称。一个 Index 包含很多 Document，一个 Index 就代表了一类类似的或者相同的 Document。比如说建立一个 Product Index (商品索引)，里面可能就存放了所有的商品数据，所有的商品 Document。

- **Type：** 类型，每个索引里都可以有一个或多个 Type，Type 是 Index 中的一个逻辑数据分类，一个 Type 下的 Document，都有相同的 Field，比如博客系统，有一个索引，可以定义 `用户数据Type`、`博客数据Type`、`评论数据Type`。

    ```css
    商品 Index
        ├─日化商品Type
        |    └─────  Document1: { product_id，category_name }
        |    └─────  Document2: { product_id，category_name }
        |
        ├─电器商品Type
        |    └─────  Document1: { product_id，category_name，service_period }
        |    └─────  Document2: { product_id，category_name，service_period }
        |
        └─生鲜商品Type
             └─────  Document1: { product_id，category_name，eat_period }
             └─────  Document2: { product_id，category_name，eat_period }
    ```

- **shard：** 分片，单台机器无法存储大量数据，ES 可以将一个索引中的数据切分为多个 shard，分布在多台服务器上存储。有了 shard 就可以横向扩展，存储更多数据，让搜索和分析等操作分布到多台服务器上去执行，提升吞吐量和性能。每个 shard 都是一个 Lucene Index。

- **replica：** 副本，任何一个服务器随时可能故障或宕机，此时 shard 可能就会丢失，因此可以为每个 shard 创建多个 replica 副本。replica 可以在 shard 故障时提供备用服务，保证数据不丢失，多个 replica 还可以提升搜索操作的吞吐量和性能。primary shard（建立索引时一次设置，不能修改，默认5个），replica shard（随时修改数量，默认1个），默认每个索引10个 shard，5个primary shard，5个 replica shard，最小的高可用配置，是2台服务器。

**🎈注意：** 6.0 之前的版本有 Type（类型）概念，Type 相当于关系数据库的表，ES 官方将在 ES9.0 版本中彻底删除 Type。

### Elasticsearch核心概念 vs 数据库核心概念

| Elasticsearch | 数据库 |
| ------------- | ----- |
| Field         | 列 |
| Document      | 行 |
| Type          | 表 |
| Index         | 库：如果相当于数据库，就表示一个索引库可以创建很多不同类型的文档，这在ES中也是允许的 |
| Index         | 表：如果相当于表，就表示一个索引库只能存储相同类型的文档，ES官方建议在一个索引库中只存储相同类型的文档 |

### 索引与结构

下图是 ElasticSearch 的索引结构，下边黑色部分是物理结构，上边黄色部分是逻辑结构，逻辑结构也是为了更好的描述 ElasticSearch 的工作原理及去使用物理结构中的索引文件。

![jpg](../imagES/01.jpg)

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

## 字段类型

Elasticsearch 字段类型：

<table align="left" border="2" style="width:410pt;">
    <tbody>
        <tr>
            <td style="width:109.35pt;">
                <p><span style="color:#3399ea;">一级分类</span></p>
            </td>
            <td style="width:106.25pt;">
                <p><span style="color:#3399ea;">二级分类</span></p>
            </td>
            <td style="width:194.4pt;">
                <p><span style="color:#3399ea;">具体类型</span></p>
            </td>
        </tr>
        <tr>
            <td rowspan="6" style="width:109.35pt;">
            <p><span style="color:#3399ea;">核心类型</span></p>
        </td>
        <td style="width:106.25pt;">
            <p><span style="color:#3399ea;">字符串类型</span></p>
        </td>
        <td style="width:194.4pt;">
            <p><span style="color:#3399ea;">string text keyword</span></p>
        </td>
        </tr>
        <tr>
            <td style="width:106.25pt;">
            <p><span style="color:#3399ea;">数字类型</span></p>
        </td>
        <td style="width:194.4pt;">
            <p><span style="color:#3399ea;">long integer short byte double float half_float scaled _float</span></p>
        </td>
        </tr>
        <tr>
            <td style="width:106.25pt;">
            <p><span style="color:#3399ea;">日期类型</span></p>
        </td>
        <td style="width:194.4pt;">
            <p><span style="color:#3399ea;">date</span></p>
        </td>
        </tr>
        <tr>
            <td style="width:106.25pt;">
            <p><span style="color:#3399ea;">布尔类型</span></p>
        </td>
        <td style="width:194.4pt;">
            <p><span style="color:#3399ea;">boolean</span></p>
        </td>
        </tr>
        <tr>
            <td style="width:106.25pt;">
            <p><span style="color:#3399ea;">二进制类型</span></p>
        </td>
        <td style="width:194.4pt;">
            <p><span style="color:#3399ea;">binary</span></p>
        </td>
        </tr>
        <tr>
            <td style="width:106.25pt;">
            <p><span style="color:#3399ea;">范围类型</span></p>
        </td>
        <td style="width:194.4pt;">
            <p><span style="color:#3399ea;">range</span></p>
        </td>
        </tr>
        <tr>
            <td rowspan="3" style="width:109.35pt;">
        <p><span style="color:#3399ea;">复合类型</span></p>
        </td>
        <td style="width:106.25pt;">
            <p><span style="color:#3399ea;">数组类型</span></p>
        </td>
        <td style="width:194.4pt;">
            <p><span style="color:#3399ea;">array</span></p>
        </td>
        </tr>
        <tr>
            <td style="width:106.25pt;">
            <p><span style="color:#3399ea;">对象类型</span></p>
        </td>
        <td style="width:194.4pt;">
            <p><span style="color:#3399ea;">object</span></p>
        </td>
        </tr>
        <tr>
            <td style="width:106.25pt;">
            <p><span style="color:#3399ea;">嵌套类型</span></p>
        </td>
        <td style="width:194.4pt;">
            <p><span style="color:#3399ea;">nested</span></p>
        </td>
        </tr>
        <tr>
            <td rowspan="2" style="width:109.35pt;">
        <p><span style="color:#3399ea;">地理类型</span></p>
        </td>
        <td style="width:106.25pt;">
            <p><span style="color:#3399ea;">地理坐标</span></p>
        </td>
        <td style="width:194.4pt;">
            <p><span style="color:#3399ea;">geo_point</span></p>
        </td>
        </tr>
        <tr>
            <td style="width:106.25pt;">
            <p><span style="color:#3399ea;">地理图形</span></p>
        </td>
        <td style="width:194.4pt;">
            <p><span style="color:#3399ea;">geo_shape</span></p>
        </td>
        </tr>
        <tr>
            <td rowspan="5" style="width:109.35pt;">
        <p><span style="color:#3399ea;">特殊类型</span></p>
        </td>
        <td style="width:106.25pt;">
            <p><span style="color:#3399ea;">IP类型</span></p>
        </td>
        <td style="width:194.4pt;">
            <p><span style="color:#3399ea;">ip</span></p>
        </td>
        </tr>
        <tr>
            <td style="width:106.25pt;">
            <p><span style="color:#3399ea;">范围类型</span></p>
        </td>
        <td style="width:194.4pt;">
            <p><span style="color:#3399ea;">completion</span></p>
        </td>
        </tr>
        <tr>
            <td style="width:106.25pt;">
            <p><span style="color:#3399ea;">令牌计数类型</span></p>
        </td>
        <td style="width:194.4pt;">
            <p><span style="color:#3399ea;">token_count</span></p>
        </td>
        </tr>
        <tr>
            <td style="width:106.25pt;">
            <p><span style="color:#3399ea;">附件类型</span></p>
        </td>
        <td style="width:194.4pt;">
            <p><span style="color:#3399ea;">attachment</span></p>
        </td>
        </tr>
        <tr>
            <td style="width:106.25pt;">
            <p><span style="color:#3399ea;">抽取类型</span></p>
        </td>
        <td style="width:194.4pt;">
            <p><span style="color:#3399ea;">percolator</span></p>
        </td>
        </tr>
    </tbody>
</table>