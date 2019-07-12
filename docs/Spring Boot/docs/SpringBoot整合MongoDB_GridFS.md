# SpringBoot整合MongoDB GridFS

## GridFS 介绍

GridFS 是 MongoDB 提供的用于持久化存储文件的模块。

它的工作原理是：在 GridFS 存储文件是将文件分块存储，文件会按照 256KB 的大小分割成多个块进行存储，GridFS 使用两个集合（collection）存储文件，一个集合是 chunks , 用于存储文件的二进制数据；一个集合是 files，用于存储文件的元数据信息（文件名称、块大小、上传时间等信息）。从 GridFS 中读取文件要对文件的各各块进行组装、合并。

详细参考：[https://docs.mongodb.com/manual/core/gridfs/](https://docs.mongodb.com/manual/core/gridfs/)

## GridFS 测试

### 首先向测试程序注入 `GridFsTemplate`

```java
@SpringBootTest
@RunWith(SpringRunner.class)
public class GridFsTest {

    @Autowired
    GridFsTemplate gridFsTemplate;

}
```

### 存文件测试

```java
@Test
//存文件
public void testStroe() throws FileNotFoundException {
    //要存储的文件
    File file = new File("d:/test1.ftl");
    //定义输入流
    FileInputStream inputStream = new FileInputStream(file);
    //向GridFS存储文件
    ObjectId objectId = gridFsTemplate.store(inputStream, "轮播图测试文件01");
    //得到文件id
    String fileId = objectId.toString();
    System.err.println(fileId);
}
```

存储原理说明：

- 文件存储成功得到一个文件 id
- 此文件 id 是 fs.files 集合中的主键。
- 以通过文件 id 查询 fs.chunks 表中的记录，得到文件的内容。


### 读取文件测试

1、创建配置类

`GridFSBucket` 用于打开下载流对象

```java
@Configuration
public class MongoConfig {

    @Value("${spring.data.mongodb.database}")
    String db;

    @Bean
    public GridFSBucket getGridFSBucket(MongoClient mongoClient) {
        MongoDatabase database = mongoClient.getDatabase(db);
        GridFSBucket bucket = GridFSBuckets.create(database);
        return bucket;
    }

}
```

2、测试代码

注入配置类中的 `GridFSBucket`：

```java
@Autowired
GridFSBucket gridFSBucket;
```

读取文件代码：

```java
@Test
public void queryFile() throws IOException {
    String fileId = "5d0f0ec6ed44e05bd4e38e84";
    //根据ID查询条件
    GridFSFile gridFSFile = gridFsTemplate.findOne(Query.query(Criteria.where("_id").is(fileId)));
    //打开下载流对象
    GridFSDownloadStream gridFSDownloadStream = gridFSBucket.openDownloadStream(gridFSFile.getObjectId());
    //创建gridFsResource，；用于获取流对象
    GridFsResource gridFsResource = new GridFsResource(gridFSFile, gridFSDownloadStream);
    //获取流中的数据
    String s = IOUtils.toString(gridFsResource.getInputStream(), "UTF-8");
    System.err.println(s);
}
```

### 删除文件测试

```java
@Test
public void testDelFile() {
    //根据文件id删除fs.files 和 fs.chunks中的记录
    gridFsTemplate.delete(Query.query(Criteria.where("_id").is("5d0f0ec6ed44e05bd4e38e84")));
}
```
