# 基于 scoll+bulk+索引别名 实现零停机重建索引

一个 field 的设置是不能被修改的，如果要修改一个 Field，那么应该重新按照新的 mapping，建立一个 index，然后将数据批量查询出来，重新用 bulk api 写入 index 中。

批量查询的时候，建议采用 scroll api，并且采用多线程并发的方式来 reindex 数据，每次 scoll 就查询指定日期的一段数据，交给一个线程即可。

**1. 自动创建索引**

依靠 dynamic mapping，插入数据，2017-01-01 这种日期格式的，所以 title 这种 field 被自动映射为了 date 类型，实际上它应该是 string 类型的

```json
// 自动创建的索引 title 类型被自动转换为 date
PUT /my_index/my_type/3
{
  "title": "2017-01-03"
}

//向索引中加入string类型的title值的时候，就会报错
PUT /my_index/my_type/4
{
  "title": "my first article"
}

// 修改title的类型也报错，不支持修改
PUT /my_index/_mapping/my_type
{
  "properties": {
    "title": {
      "type": "text"
    }
  }
}
```

**2. reindex 重建索引**

此时，唯一的办法，就是进行 reindex，也就是说，重新建立一个索引，将旧索引的数据查询出来，再导入新索引。

如果说旧索引的名字，是 old_index，新索引的名字是 new_index，终端 java 应用，已经在使用 old_index 在操作了，难道还要去停止 java 应用，修改使用的 index 为 new_index，才重新启动 java 应用吗？这个过程中，就会导致 java 应用停机，可用性降低。

所以说，给 java 应用一个别名，这个别名是指向旧索引的，java 应用先用着，java 应用先用 goods_index alias 来操作，此时实际指向的是旧的 my_index。

```json
//1. 创建 my_index 的别名 goods_index
PUT /my_index/_alias/goods_index

//2. 新建一个index，调整其title的类型为string
PUT /my_index_new
{
  "mappings": {
    "my_type": {
      "properties": {
        "title": {
          "type": "text"
        }
      }
    }
  }
}

//3. 使用scroll api将数据批量查询出来
GET /my_index/_search?scroll=1m
{
    "query": {
        "match_all": {}
    },
    "sort": ["_doc"],
    "size":  1
}

//4. 采用bulk api将scoll查出来的一批数据，批量写入新索引
POST /_bulk
{ "index":  { "_index": "my_index_new", "_type": "my_type", "_id": "2" }}
{ "title":    "2017-01-02" }

//5. 反复循环步骤3~4，采取bulk api将每一批数据批量写入新索引，直到写入所有数据

//6. 将 goods_index alias 切换到 my_index_new 上去，java应用会直接通过index别名使用新的索引中的数据，java应用程序不需要停机，高可用
POST /_aliases
{
    "actions": [
        { "remove": { "index": "my_index", "alias": "goods_index" }},
        { "add":    { "index": "my_index_new", "alias": "goods_index" }}
    ]
}

//7. 直接通过 goods_index 别名来查询数据，是否ok
```






