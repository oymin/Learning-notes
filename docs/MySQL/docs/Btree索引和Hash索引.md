# Btree索引和Hash索引

## B-tree索引

特点：

- B-tree 以 B + 树的结构存储数据
- 能够加快数据的查询速度
- 更适合进行范围查找

在什么情况下可以用到B树索引：

- 全职匹配的查询
- 匹配最左前缀的查询
- 匹配列前缀查询
- 匹配范围值的查询
- 精确匹配左前列并范围匹配另外一列
- 只访问索引的查询

Btree索引的使用限制：

- 如果不是按照索引最左列开始查找，则无法使用索引
- 使用索引时不能跳过索引中的列
- Not in 和 < ，> 操作无法只用索引
- 如果查询中有某个列的范围查询，则其右边所有列都无法使用索引

## Hash索引

特点：

- Hash索引时基于 Hash 表实现的，只有查询条件精确匹配 Hash 索引中的所有列时，才能够使用到 Hash 索引。
- 对于 Hash 索引中的所有列，存储引擎都会为每一行计算一个 Hash 码，Hash 索引中存储的就是 Hash 码。
  
Hash 索引的限制：

- Hash 索引必须进行二次查找
- Hash 索引无法用于排序
- Hash 索引不支持部分索引查找也不支持范围查找
- Hash 索引中 Hash 码的计算可能存在 Hash 冲突

## 为什么要使用索引

- 索引大大减少了存储引擎需要扫描的数据量
- 索引可以帮我进行排序以避免使用临时表
- 索引可以把随机 I/O 变为顺序 I/O

## 索引是不是越多越好

- 索引会增加写操作的成本
- 太多的索引会增加查询优化器的选择时间

## 安装 Mysql 演示数据库

下载网址：[https://downloads.mysql.com/docs/sakila-db.tar.gz](https://downloads.mysql.com/docs/sakila-db.tar.gz)

```bash
mysql -uroot -p < sakila-schema.sql
mysql -uroot -p < sakila-data.sql
```
