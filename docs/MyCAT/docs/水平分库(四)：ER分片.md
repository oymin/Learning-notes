# 水平分库(四)：ER分片

本案例场景：对 `order_master` 表水平分片后，其它表没有水平分片。所以当执行 `order_master` 表与其 `order_detail` 表的关联 sql 语句时会报错。

```sql
select * from order_master a join order_detail b on a.order_id = b.order_id;

ERROR 1064 (HY000): invalid route in sql
```

在水平分片环境中可以通过 `ER分片` 解决跨片查询问题。

## 1. 配置ER分片

### 1. 配置 schemal.xml 文件

修改 `order_master` 表的 `<table>` 标签，并删除 `order_detail` 的 `<table>` 标签

```xml
<table name="order_master" primaryKey="order_id" dataNode="orderdb01,orderdb02,orderdb03,orderdb04" rule="order_master_rule" autoIncrement="true" >
    <!-- 增加 order_detail 的 `<childTable>` 子标签 -->
    <childTable name="order_detail" primaryKey="order_detail_id" joinKey="order_id" parentKey="order_id" autoIncrement="true" />
</table>

<!-- 删除或注释掉 order_detail 的 `<table>` 标签 -->
<!-- <table name="order_detail" primaryKey="order_detail_id" dataNode="ordb" /> -->
```

### 2. 配置 order_detail 表的全局主键 ID

向 `MYCAT_SEQUENCE` 表中插入要使用全局自增 ID 的表名（要大写）：

```sql
# 自增 Id 初始值为 1，并以 1 递增
insert into MYCAT_SEQUENCE values ('ORDER_DETAIL', 1, 1);
```

修改 `mycat/conf/sequence_db_conf.properties` 文件

```yml
#增加内容
ORDER_DETAIL=mycat
```

### 3. 在分片库中创建 order_detail 表

**192.168.75.129** 节点中 `orderdb01` 和 `orderdb02` 数据库创建 `order_detail` 表

**192.168.75.130** 节点中 `orderdb03` 和 `orderdb04` 数据库创建 `order_detail` 表

---

## 2. 测试ER分片关联查询

登录 `mycat` 向 `order_detail` 插入数据后执行关联查询语句

```sql
SELECT a.`order_id`, a.`order_sn`, a.`customer_id`,b.`product_name` FROM `order_master` a JOIN `order_detail` b WHERE a.`order_id` = 2;
+----------+----------------+-------------+--------------+
| order_id | order_sn       | customer_id | product_name |
+----------+----------------+-------------+--------------+
|        2 | 20180324919235 |        6235 | 上衣         |
+----------+----------------+-------------+--------------+
1 row in set (0.01 sec)
```

与 `order_master` 关联的 `order_detail` 会存在同一个数据库下。
