# ElasticSearch批量操作

批量查询的好处:

- 就是一条一条的查询，比如说要查询100条数据，那么就要发送100次网络请求，这个开销还是很大的
- 如果进行批量查询的话，查询100条数据，就只要发送1次网络请求，网络请求的性能开销缩减100倍

## mget 批量查询

可以说 mget 是很重要的，一般来说，在进行查询的时候，如果一次性要查询多条数据的话，那么一定要用 batc h批量操作的 api

尽可能减少网络开销次数，可能可以将性能提升数倍，甚至数十倍，非常非常之重要

```bash
GET /_mget
{
   "docs" : [
      {
         "_index" : "test_index",
         "_type" :  "test_type",
         "_id" :    1
      },
      {
         "_index" : "test_index",
         "_type" :  "test_type",
         "_id" :    2
      }
   ]
}
```

**如果查询的 document 是一个 index 下的不同 type 种的话:**

```bash
GET /test_index/_mget
{
   "docs" : [
      {
         "_type" :  "test_type",
         "_id" :    1
      },
      {
         "_type" :  "test_type",
         "_id" :    2
      }
   ]
}
```

**如果查询的数据都在同一个 index 下的同一个 type 下，最简单了:**

```bash
GET /test_index/test_type/_mget
{
   "ids": [1, 2]
}
```

## bulk 批量增删改

### bulk 语法

每一个操作要两个 json 串，语法如下：

```json
{"action": {"metadata"}}
{"data"}
```

举例，比如你现在要创建一个文档，放 bulk 里面，看起来会是这样子的：

```json
{"index": {"_index": "test_index", "_type", "test_type", "_id": "1"}}
{"test_field1": "test1", "test_field2": "test2"}
```

可操作类型:

1. delete：删除一个文档，只要 1 个 json 串就可以了

2. create：PUT /index/type/id/_create，强制创建

3. index：普通的 put 操作，可以是创建文档，也可以是全量替换文档

4. update：执行的 partial update 操作

```json
{ "delete": { "_index": "test_index", "_type": "test_type", "_id": "3" }}

{ "create": { "_index": "test_index", "_type": "test_type", "_id": "12" }}
{ "test_field":    "test12" }

{ "index":  { "_index": "test_index", "_type": "test_type", "_id": "2" }}
{ "test_field":    "replaced test2" }

{ "update": { "_index": "test_index", "_type": "test_type", "_id": "1", "_retry_on_conflict" : 3} }
{ "doc" : {"test_field2" : "bulk test1"} }
```

`bulk` api 对 json 的语法，有严格的要求，每个 json 串不能换行，只能放一行，同时一个 json 串和一个 json 串之间，必须有一个换行。

这种格式最大的优势在于，不需要将 json 数组解析为一个 JSONArray 对象，形成一份大数据的拷贝，浪费内存空间，尽可能地保证性能。

bulk 操作中，任意一个操作失败，是不会影响其他的操作的，但是在返回结果里，会告诉你异常日志

### bulk size 最佳大小

bulk request 会加载到内存里，如果太大的话，性能反而会下降，因此需要反复尝试一个最佳的 bulk size。一般从 1000~5000 条数据开始，尝试逐渐增加。另外，如果看大小的话，最好是在 5~15MB 之间。















