# ElasticSeach进行groovy脚本

## 基于 groovy 脚本进行 partial update

其实是有个内置的脚本支持的，可以基于 groovy 脚本实现各种各样的复杂操作

基于 groovy 脚本，如何执行 partial update：`es scripting module`

```bash
# 准备数据
PUT /test_index/test_type/11
{
  "num": 0,
  "tags": []
}
```

## 内置脚本

```bash
# num 的值 + 1
POST /test_index/test_type/11/_update
{
   "script" : "ctx._source.num+=1"
}
```

## 外部脚本

```bash
# 在 es 安装 /config 目录下，创建 scripts 目录
mkidr scripts

# 创建一个groovy脚本文件
vi test-add-tags.groovy

# 添加内容
ctx._source.tags+=new_tag
```

```bash
POST /test_index/test_type/11/_update
{
  "script": {
    "lang": "groovy",
    "file": "test-add-tags",
    "params": {
      "new_tag": "tag1"
    }
  }
}
```

## 用脚本删除文档

```bash
vi test-delete-document.groovy

# 添加内容
ctx.op = ctx._source.num == count ? 'delete' : 'none'
```

```bash
POST /test_index/test_type/11/_update
{
  "script": {
    "lang": "groovy",
    "file": "test-delete-document",
    "params": {
      "count": 1
    }
  }
}
```




















