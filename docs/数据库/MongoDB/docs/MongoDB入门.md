# MongoDB 入门

## 基础概念

在 MongoDB 中是通过数据库、集合、文档的方式来管理数据，下边是 MongoDB 与关系数据库的一些概念对比：

| Sql术语 | MongoDB术语 | 解释/说明 |
| --- | --- | --- |
| database  | database      | 数据库    |
| table     | collection    | 数据库表/集合|
| row       | document      | 数据库记录行/文档 |
| column    | field     | 数据字段/域 |
| index     | index     | 索引 |
| table joins |         | 表链接（MongoDB不支持）|
| primary key | primary key | 主键，MongoDB自动在每个集合中添加_id主键 |
| | | |

1、一个 MongoDB 实例可以创建多个数据库

2、一个数据库可以创建多个集合

3、一个集合可以包括多个文档。


## 连接 MongoDB

MongoDB 的使用方式是客户服务器模式，即使用一个客户端连接 MongoDB 数据库（服务端）。

1、命令格式

```bash
mongodb://[username:password@]host1[:port1][,host2[:port2],...[,hostN[:portN]]][/[database][?options]]
```

**mongodb：** // 固定前缀

**username：** 账号，可不填

**password：** 密码，可不填

**host：** 主机名或ip地址，只有host主机名为必填项。

**port：** 端口，可不填，默认27017

/**database：** 连接某一个数据库

?**options：** 连接参数，key/value对

例子：

```bash
# 连接本地数据库27017端口
mongodb://localhost

# 使用用户名root密码为root连接本地数据库27017端口
mongodb://root:root@localhost

# 连接三台主从服务器，端口为27017、27018、27019
mongodb://localhost,localhost:27018,localhost:27019
```

2、使用 MongoDB 自带的 javascript shell (mongo.exe) 连接

windows 版本的 MongoDB 安装成功，在安装目录下的 bin 目录有 mongo.exe 客户端程序

```bash
# cmd状态执行 mongo.exe
mongo.exe
```

3、使用 studio3T 连接

4、使用 java 程序连接

详细参数：[http://mongodb.github.io/mongo-java-driver/3.4/driver/tutorials/connect-to-mongodb/](http://mongodb.github.io/mongo-java-driver/3.4/driver/tutorials/connect-to-mongodb/)

添加依赖：

```xml
<dependency>
    <groupId>org.mongodb</groupId>
    <artifactId>mongo‐java‐driver</artifactId>
    <version>3.4.3</version>
</dependency>
```

测试程序：

```java
@Test
public void testConnection(){
    //创建mongodb 客户端
    MongoClient mongoClient = new MongoClient( "localhost" , 27017 );

    //或者采用连接字符串
    //MongoClientURI connectionString = new
    MongoClientURI("mongodb://root:root@localhost:27017");

    //MongoClient mongoClient = new MongoClient(connectionString);
    //连接数据库
    MongoDatabase database = mongoClient.getDatabase("test");

    // 连接collection
    MongoCollection<Document> collection = database.getCollection("student");

    //查询第一个文档
    Document myDoc = collection.find().first();

    //得到文件内容 json串
    String json = myDoc.toJson();
    System.out.println(json);
}
```

## 数据库

### 1. 查询数据库

```bash
# 查询全部数据库
show dbs
```

```bash
# 显示当前数据库
db
```

### 2. 创建数据库

```bash
# 创建数据库
use [database name]
```

该命令：已有该名称数据库则切换到此数据库，没有则创建。

### 3. 删除数据库

```bash
# 首先进入要删除的数据库
db.dropDatabase()
```

## 集合

集合相当于关系数据库中的表，一个数据库可以创建多个集合，一个集合是将相同类型的文档管理起来。

### 1. 创建集合

```bash
# name: 新创建的集合名称
# options: 创建参数
db.createCollection(name, iptions)
```

### 2. 删除集合

```bash
db.[coleection_name].drop()
```

## 文档

### 1. 插入文档

mongodb 中文档的格式是 json 格式，下边就是一个文档，包括两个 key：_id 主键和 name

```bash
{
    "_id" : ObjectId("5b2cc4bfa6a44812707739b5"),
    "name" : "黑马程序员"
}
```

插入命令：

```bash
db.COLLECTION_NAME.insert(document)
```

每个文档默认以 `_id` 作为主键，主键默认类型为 ObjectId（对象类型），mongodb 会自动生成主键值。

例子：

```bash
db.student.insert({"name":"黑马程序员","age":10})
```

注意：同一个集合中的文档的 key 可以不相同！但是建议设置为相同的。

### 2. 更新文档

命令格式：

```bash
# query: 查询条件，相当于sql语句的where
# update：更新文档内容
# options：选项
db.collection.update(
    <query>,
    <update>,
    <options>
)
```

替换文档:

将符合条件 `"name":"程序"` 的第一个文档替换为 `{"name":"程序员","age":10}` 。

```bash
db.student.update({"name":"程序"},{"name":"马程序员","age":10})
```

`$set` 修改器
使用 $set 修改器指定要更新的 key，key 不存在则创建，存在则更新。

将符合条件 `"name":"程序"` 的所有文档更新 name 和 age 的值。

```bash
db.student.update({"name":"程序"},{$set:{"name":"程序员","age":10}},{multi:true})
```

**multi：** `false` 表示更新第一个匹配的文档，`true` 表示更新所有匹配的文档

### 3. 删除文档

命令格式：

```bash
# query：删除条件，相当于sql语句中的where
db.student.remove(<query>)
```

删除所有文档

```bash
db.student.remove({})
```

删除符合条件的文档

```bash
db.student.remove({"name":"程序员"})
```

### 4. 查询文档

命令格式：

```bash
# query：查询条件，可不填
# projection：投影查询key，可不填
db.collection.find(query, projection)
```

查询全部

```bash
db.student.find()
```

查询符合条件的记录

```bash
# 查询 name 等为"程序员"的文档
db.student.find({"name":"程序员"})
```

投影查询

```bash
# 只显示 name 和 age两个 key，_id 主键不显示
db.student.find({"name":"程序员"},{name:1,age:1,_id:0})
```

## 用户

### 1. 创建用户

语法格式：

```bash
mongo>db.createUser(
    {   user: "<name>",
        pwd: "<cleartext password>",
        customData: { <any information> },
        roles: [
            { role: "<role>", db: "<database>" } | "<role>",
            ...
        ]
    }
)
```

例子：

```bash
# 创建root用户，角色为root
use admin
db.createUser(
    {
        user:"root",
        pwd:"root",
        roles:[{role:"root",db:"admin"}]
    }
)
```

内置角色如下：

1. 数据库用户角色：read、readWrite;

2. 数据库管理角色：dbAdmin、dbOwner、userAdmin；

3. 集群管理角色：clusterAdmin、clusterManager、clusterMonitor、hostManager；

4. 备份恢复角色：backup、restore；

5. 所有数据库角色：readAnyDatabase、readWriteAnyDatabase、userAdminAnyDatabase、
dbAdminAnyDatabase

6. 超级用户角色：root

### 2. 查询用户

```bash
# 查询当前库下的所有用户：
show users
```

### 3. 删除用户

```bash
# 删除 root1 用户
db.dropUser("root1")
```

### 4. 修改用户

语法格式：

```bash
db.updateUser(
    "<username>",
    {
        customData : { <any information> },
        roles : [
        { role: "<role>", db: "<database>" } | "<role>",
        ...
        ],
        pwd: "<cleartext password>"
    },
    writeConcern: { <write concern> }
)
```

例子：

```bash
# 修改 root 用户的角色为 readWriteAnyDatabase
use admin
db.updateUser("root",{roles:[{role:"readWriteAnyDatabase",db:"admin"}]})
```

### 5. 修改密码

```bash
#修改 root 用户的密码为 123
use admin
db.changeUserPassword("root","123")
```
