# 垂直分库(四)：通过 MYCAT 访问 DB

## 1. 从库主机连接 mycat 验证是否配置成功：

例如在 `131` 的主机中连接：

```sql
mysql -uapp_koax -p123456 -h192.168.194.128 -P8066 -A
```

### 如果连接 mycat 逻辑数据库成功，验证是否能够访问逻辑表数据：

```sql
select * from product_info limit 10;
```

能够查询到数据，表示配置 MyCAT 成功。

---

## 2. 清除多余数据

把不属于 `129` 、`130` 、 `131` 每个自己节点的表清理掉 

首先关闭各个从库的 slvae：

```sql
stop slave;

reset slave all;
```

然后每个节点删除不需要的表，只留下 `schemal.xml` 中该节点配置的表。

---

## 3. 配置和验证全局表

登录 mycat ：

```sql
mysql -uapp_koax -p123456 -h192.168.194.128 -P8066 -A
```

执行查询语句：

```sql
SELECT supplier_name, b.region_name AS '省', c.region_name AS '市', d.region_name AS '区'
FROM product_supplier_info a
JOIN region_info b ON b.`region_id` = a.`province`
JOIN region_info c ON c.`region_id` = a.`city`
JOIN region_info d ON d.`region_id` = a.`district`;
```

会报错 `invalid route in sql` sql 路由错误：

```sql
ERROR 1064 (HY000): invalid route in sql, multi tables found but datanode has no intersection
```

这是因为查询语句中有表关联查询，关联了在 `order_db` 数据库中的表 `region_info`, 而 `product_supplier_info` 表在 `product_db` 中，这样就产生了跨分片查询。

这里的解决方法是将 `region_info` 表设置为全局表，每个节点点中都存在该表，并且该表的维护由 MyCAT 管理。

### 配置全局表

#### 1. order_db 备份表

```sql
mysqldump -uroot -p order_db region_info > region_info
```

#### 2. 其他节点导入备份表

```sql
mysql -uroot -p product_db < region_info

mysql -uroot -p customer_db < region_info
```

#### 3. 修改  schema.xml 中 region_info 表的配置

```xml
<table name="region_info" primaryKey="region_id" dataNode="ordb,custdb,prodb" type="global" />
```

#### 4. 重启 MyCAT

#### 5. 再次执行查询语句，查看效果

查询结果：

```sql
+---------------+-----------+-----------+-----------+
| supplier_name | 省        | 市        | 区        |
+---------------+-----------+-----------+-----------+
| 供应商-1      | 上海市    | 上海市    | 徐汇区    |
+---------------+-----------+-----------+-----------+
1 row in set (0.02 sec)
```

---

## 4. 总结

### 垂直切分的优点

1. 数据库的拆分简单明了，拆分规则明确

2. 应用程序模块清晰明确，整合容易

3. 数据维护方便易行，容易定位

4. 减少对数据库写的负担

### 处置切分的缺点

1. 部分表关联无法在数据库级别完成，需要在程序中完成

2. 对于访问极其频繁且数据量超大的表任然存在性能瓶颈

3. 切分达到一定程度之后，扩展性会遇到限制

### 解决跨分片关联的方式

1. 使用 MyCAT 全局表

2. 冗余部分关键数据

3. 使用 API 的方式获取数据
