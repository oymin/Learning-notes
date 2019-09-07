# MySQL 索引

MySQL 索引的建立对于 MySQL 的高效运行是很重要的，索引可以大大提高 MySQL 的检索速度。

## 什么是索引

索引的作用相当于图书的目录，可以根据目录中的页码快速找到所需的内容。

数据库使用索引以找到特定值，然后顺指针找到包含该值的行。在表中建立索引，然后在索引中找到符合查询条件的索引值，最后通过保存在索引中的 ROWID（相当于页码）快速找到表中对应的记录。索引的建立是表中比较有指向性的字段，相当于目录，比如说行政区域代码，同一个地域的行政区域代码都是相同的，那么给这一列加上索引，避免让它重复扫描，从而达到优化的目的！

### 索引分单列索引和组合索引

- 单列索引，即一个索引只包含单个列，一个表可以有多个单列索引，但这不是组合索引。

- 组合索引，即一个索引包含多个列。

创建索引时，你需要确保该索引是应用在 SQL 查询语句的条件(一般作为 WHERE 子句的条件)。

实际上，索引也是一张表，该表保存了主键与索引字段，并指向实体表的记录。

上面都在说使用索引的好处，但过多的使用索引将会造成滥用。因此索引也会有它的缺点：虽然索引大大提高了查询速度，同时却会降低更新表的速度，如对表进行 INSERT、UPDATE 和 DELETE。因为更新表时，MySQL 不仅要保存数据，还要保存一下索引文件。

建立索引会占用磁盘空间的索引文件。

## 什么情况下使用索引

1、表的主关键字

2、自动建立唯一索引

3、表的字段唯一约束

4、直接条件查询的字段（在SQL中用于条件约束的字段）

5、查询中与其它表关联的字段

6、查询中排序的字段（排序的字段如果通过索引去访问那将大大提高排序速度）

7、查询中统计或分组统计的字段

8、表记录太少（如果一个表只有5条记录，采用索引去访问记录的话，那首先需访问索引表，再通过索引表访问数据表，一般索引表与数据表不在同一个数据块）

9、经常插入、删除、修改的表（对一些经常处理的业务表应在查询允许的情况下尽量减少索引）

10、数据重复且分布平均的表字段（假如一个表有10万行记录，有一个字段A只有T和F两种值，且每个值的分布概率大约为50%，那么对这种表A字段建索引一般不会提高数据库的查询速度。）

11、经常和主字段一块查询但主字段索引值比较多的表字段

12、对千万级 MySQL 数据库建立索引的事项及提高性能的手段

## 如何选择合适的列建立索引

1、在 `where` 从句，`group by` 从句，`order by` 从句，`on` 从句中的列添加索引

2、索引字段越小越好（因为数据库数据存储单位是以“页”为单位的，数据存储的越多，IO 也会越大）

3、离散度大的列放到联合索引的前面

例子：

```sql
select * from payment where staff_id = 2 and customer_id = 584;
```

注意: 是 `index（staff_id，customer_id）`好，还是 `index（customer_id，staff_id）`好？那我们怎么进行验证离散度？

我们先查看以下表结构

```sql
mysql> desc payment;

+--------------+----------------------+------+-----+-------------------+-----------------------------+
| Field        | Type                 | Null | Key | Default           | Extra                       |
+--------------+----------------------+------+-----+-------------------+-----------------------------+
| payment_id   | smallint(5) unsigned | NO   | PRI | NULL              | auto_increment              |
| customer_id  | smallint(5) unsigned | NO   | MUL | NULL              |                             |
| staff_id     | tinyint(3) unsigned  | NO   | MUL | NULL              |                             |
| rental_id    | int(11)              | YES  | MUL | NULL              |                             |
| amount       | decimal(5,2)         | NO   |     | NULL              |                             |
| payment_date | datetime             | NO   | MUL | NULL              |                             |
| last_update  | timestamp            | NO   |     | CURRENT_TIMESTAMP | on update CURRENT_TIMESTAMP |
+--------------+----------------------+------+-----+-------------------+-----------------------------+
```

分别查看这两个字段中不同的 id 的数量，数量越多，则表明离散程度越大：因此可以通过下列查询看出：customer_id 离散程度大。

```sql
select count(distinct staff_id), count(distinct customer_id) from payment;

+--------------------------+-----------------------------+
| count(distinct staff_id) | count(distinct customer_id) |
+--------------------------+-----------------------------+
|                        2 |                         599 |
+--------------------------+-----------------------------+
```

结论：由于customer_id 离散程度大，使用 `index（customer_id，staff_id）`好

## 普通索引

### 创建普通索引

这是基本的索引，它没有任何限制。它有以下几种创建方式：

```sql
CREATE INDEX [indexName] ON mytable(username(length));

#创建索引
create index id on t_student(name);
```

如果是 `CHAR`，`VARCHAR` 类型，length 可以小于字段实际长度；如果是 `BLOB` 和 `TEXT` 类型，必须指定 length。

### 修改表结构（添加索引）

`ALTER TABLE` 用来创建普通索引、UNIQUE 索引或 PRIMARY KEY 索引。

`ALTER TABLE` 允许在单个语句中更改多个表，因此可以在同时创建多个索引。

```sql
ALTER TABLE table_name ADD INDEX index_name (column_list)
```

```text
参数说明：
table_name      要增加索引的表名
column_list     对哪些列进行索引，多列时各列之间用逗号分隔
index_name      索引名(可选，缺省时，MySQL 将根据第一个索引列赋一个名称)
```

### 创建表的时候直接指定

```sql
CREATE TABLE mytable(
  ID INT NOT NULL,
  username VARCHAR(16) NOT NULL,
  INDEX [indexName](username(length))
);
```

### 删除索引的语法

```sql
DROP INDEX [indexName] ON mytable;
```

---

## 唯一索引

它与前面的普通索引类似，不同的就是：索引列的值必须唯一，但允许有空值。如果是组合索引，则列值的组合必须唯一。它有以下几种创建方式：

### 创建唯一索引

```sql
CREATE UNIQUE INDEX [indexName] ON mytable(cloumn(length))
```

### 添加唯一索引

```sql
ALTER TABLE [tableName] ADD UNIQUE [indexName](cloumn(length))
```

### 创建表时指定唯一索引

```sql
CREATE TABLE mytable(
    ID INT NOT NULL,
    username VARCHAR(16) NOT NULL,
    UNIQUE [indexName] (username(length))
);
```

## 联合索引

### 什么是联合索引

1、两个或更多个列上的索引被称作联合索引，又被称为是复合索引。

2、利用索引中的附加列，您可以缩小搜索的范围，但使用一个具有两列的索引 不同于使用两个单独的索引。

```text
复合索引的结构与电话簿类似，人名由姓和名构成，电话簿首先按姓氏对进行排序，然后按名字对有相同姓氏的人进行排序。如果您知 道姓，电话簿将非常有用；如果您知道姓和名，电话簿则更为有用，但如果您只知道名不姓，电话簿将没有用处。
```

所以说创建复合索引时，应该仔细考虑列的顺序。对索引中的所有列执行搜索或仅对前几列执行搜索时，复合索引非常有用；仅对后面的任意列执行搜索时，复合索引则没有用处。

### 命名规则：`表名_字段名`

1. 需要加索引的字段，要在where条件中
2. 数据量少的字段不需要加索引
3. 如果 where 条件中是 OR 关系，加索引不起作用
4. 符合最左原则

---

## 使用 ALTER 命令添加和删除索引

有四种方式来添加数据表的索引：

```sql
-- 该语句添加一个主键，这意味着索引 值必须是唯一的，且不能为NULL。
ALTER TABLE tbl_name ADD PRIMARY KEY (column_list)

-- 这条语句创建索引的值必须 是唯一的（除了NULL外，NULL可能会出现多次）。
ALTER TABLE tbl_name ADD UNIQUE index_name (column_list)
  
-- 添加普通索引，索引值可出现 多次。
ALTER TABLE tbl_name ADD INDEX index_name (column_list)

-- 该语句指定了索引为 FULLTEXT ，用于全文索引
ALTER TABLE tbl_name ADD FULLTEXT index_name (column_list)
```

以下实例为在表中添加索引：

```sql
ALTER TABLE testalter_tbl ADD INDEX (c);
```

在 ALTER 命令中使用 DROP 子句来删除索引:

```sql
ALTER TABLE testalter_tbl DROP INDEX c;
```

## 使用 ALTER 命令添加和删除主键

主键只能作用于一个列上，添加主键索引时，你需要确保该主键默认不为空（NOT NULL）。实例如下：

```sql
ALTER TABLE testalter_tbl MODIFY username INT NOT NULL;

ALTER TABLE testalter_tbl ADD PRIMARY KEY (username);
```

你也可以使用 ALTER 命令删除主键：

```sql
ALTER TABLE testalter_tbl DROP PRIMARY KEY;
```

删除主键时只需指定 `PRIMARY KEY`，但在删除索引时，你必须知道索引名。

---

## 显示索引信息

可以使用 `SHOW INDEX` 命令来列出表中的相关的索引信息。可以通过添加 `\G` 来格式化输出信息。

```sql
SHOW INDEX from table_name \G
```

## 索引优化

### 1. 索引优化 SQL 的方法

1、索引的维护及优化（重复及冗余索引）

增加索引会有利于查询效率，但会降低 insert，update，delete 的效率，但实际上往往不是这样的，过多的索引会不但会影响使用效率，同时会影响查询效率，这是由于数据库进行查询分析时，首先要选择使用哪一个索引进行查询，如果索引过多，分析过程就会越慢，这样同样的减少查询的效率，因此我们要知道如何增加，有时候要知道维护和删除不需要的索引

2、如何找到重复和冗余的索引

重复索引：重复索引是指相同的列以相同的顺序建立的同类型的索引，如下表中的 primary key 和 ID 列上的索引就是重复索引

```sql
CREATE TABLE test (
  id INT NOT NULL PRIMARY KEY,
  NAME VARCHAR (10) NOT NULL,
  UNIQUE (id)
) ENGINE = INNODB;
```

冗余索引：冗余索引是指多个索引的前缀列相同，或是在联合索引中包含了主键的索引，下面这个例子中 key（name，id）就是一个冗余索引。

```sql
CREATE TABLE test (
  id INT NOT NULL PRIMARY KEY,
  NAME VARCHAR (10) NOT NULL,
  KEY (NAME, id)
) ENGINE = INNODB;
```

说明：对于 innodb 来说，每一个索引后面，实际上都会包含主键，这时候我们建立的联合索引，又人为的把主键包含进去，那么这个时候就是一个冗余索引。

3、如何查找重复索引

工具：使用 pt-duplicate-key-checker 工具检查重复及冗余索引

```sql
[root@~]# pt-duplicate-key-checker -uroot -p123456 -h 127.0.0.1

# ########################################################################
# Summary of indexes
# ########################################################################

# Total Indexes  123
```

4、索引维护的方法

由于业务变更，某些索引是后续不需要使用的，就要进行删除。

在 mysql 中，目前只能通过慢查询日志配合 pt-index-usage 工具来进行索引使用情况的分析；

```sql
[root@~]# pt-index-usage -uroot -p123456 /var/lib/mysql/localhost151-slow.log 

ALTER TABLE `sakila`.`actor` DROP KEY `idx_actor_last_name`; -- type:non-unique

ALTER TABLE `sakila`.`film` DROP KEY `idx_fk_language_id`, DROP KEY `idx_fk_original_language_id`, DROP KEY `idx_title`; -- type:non-unique
```

附：[https://www.percona.com/downloads/](https://www.percona.com/downloads/)

### 2. 注意事项

设计好 MySql 的索引可以让你的数据库飞起来，大大的提高数据库效率。设计 MySql 索引的时候有一下几点注意：

1、创建索引

对于查询占主要的应用来说，索引显得尤为重要。很多时候性能问题很简单的就是因为我们忘了添加索引而造成的，或者说没有添加更为有效的索引导致。如果不加索引的话，那么查找任何哪怕只是一条特定的数据都会进行一次全表扫描，如果一张表的数据量很大而符合条件的结果又很少，那么不加索引会引起致命的性能下降。

2、复合索引

比如有一条语句是这样的：`select * from users where area='beijing' and age=22;`

如果我们是在 area 和 age 上分别创建单个索引的话，由于mysql查询每次只能使用一个索引，所以虽然这样已经相对不做索引时全表扫描提高了很多效率，但是如果在 area、age 两列上创建复合索引的话将带来更高的效率。如果我们创建了 `(area, age,salary)` 的复合索引，那么其实相当于创建了 `(area,age,salary)` 、`(area,age)` 、`(area)` 三个索引，这被称为最佳左前缀特性。

因此我们在创建复合索引时应该将最常用作限制条件的列放在最左边，依次递减。

3、索引不会包含有 NULL 值的列

只要列中包含有 NULL 值都将不会被包含在索引中，复合索引中只要有一列含有 NULL 值，那么这一列对于此复合索引就是无效的。所以我们在数据库设计时不要让字段的默认值为 NULL。

4、使用短索引

对字符串列进行索引，如果可能应该指定一个前缀长度。例如，如果有一个 CHAR(255) 的列，如果在前 10 个或 20 个字符内，多数值是惟一的，那么就不要对整个列进行索引。短索引不仅可以提高查询速度而且可以节省磁盘空间和 I/O 操作。

5、排序的索引问题

mysql 查询只使用一个索引，因此如果 where 子句中已经使用了索引的话，那么 order by 中的列是不会使用索引的。因此数据库默认排序可以符合要求的情况下不要使用排序操作；尽量不要包含多个列的排序，如果需要最好给这些列创建复合索引。

6、like 语句操作

一般情况下不鼓励使用 like 操作，如果非使用不可，如何使用也是一个问题。`like '%aaa%'` 不会使用索引而 `'like aaa%'` 可以使用索引。

7、不要在列上进行运算

```sql
SELECT * FROM users WHERE YEAR(ADDDATE) ;
```

8、不使用 NOT IN 操作

`NOT IN` 操作都不会使用索引将进行全表扫描。`NOT IN` 可以 `NOT EXISTS` 代替

