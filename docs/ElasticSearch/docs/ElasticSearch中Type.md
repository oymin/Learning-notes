# ElasticSearch 中 Type

**Type** 是一个 Index 中用来区分类似的数据的，类似的数据，但是可能有不同的 fields，而且有不同的属性来控制索引建立、分词器。

field 的 value，在底层的 lucene 中建立索引的时候，全部是 opaque bytes 类型，不区分类型的。

lucene 是没有 Type 的概念的，在 Document 中，实际上将 Type 作为一个 Document 的 field 来存储，即 _Type，es 通过 _Type 来进行 Type 的过滤和筛选。

一个 Index 中的多个 Type，实际上是放在一起存储的，因此一个 Index 下，不能有多个 Type 重名，而类型或者其他设置不同的，因为那样是无法处理的。

**root object**

就是某个 type 对应的 mapping json，包括了 ：

- properties
- metadata（_id，_source，_type）
- settings（analyzer），其他settings（比如include_in_all）

**properties**

参数：

- type
- index
- analyzer

**_source**

好处:

1. 查询的时候，直接可以拿到完整的 document，不需要先拿 document id，再发送一次请求拿 document
2. partial update 基于 _source 实现
3. reindex 时，直接基于 _source 实现，不需要从数据库（或者其他外部存储）查询数据再修改
4. 可以基于 _source 定制返回 field
5. debug query 更容易，因为可以直接看到 _source

如果不需要上述好处，可以禁用 _source

```bash
PUT /my_index/_mapping/my_type2
{
  "_source": {"enabled": false}
}
```

**_all**

将所有 field 打包在一起，作为一个 _all field，建立索引。没指定任何 field 进行搜索时，就是使用 _all field 在搜索。

```bash
PUT /my_index/_mapping/my_type3
{
  "_all": {"enabled": false}
}
```

也可以在 field 级别设置 include_in_all field，设置是否要将 field 的值包含在 _all field 中

```bash
PUT /my_index/_mapping/my_type4
{
  "properties": {
    "my_field": {
      "type": "text",
      "include_in_all": false
    }
  }
}
```

**标识性 metadata**

- _index
- _type
- _id

