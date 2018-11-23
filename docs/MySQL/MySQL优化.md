# MySQL优化

## 可以从几个方面进行数据库优化

![](../../images/MySQL/MySqlOpt_1.jpg)

---

## 演示数据库

- 使用MySQL提供的 sakila 数据库  
[https://dev.mysql.com/doc/index-other.html](https://dev.mysql.com/doc/index-other.html)

- sakila数据库的表结构信息可以通过以下网站查看  
[http://dev.mysql.com/doc/sakila/en/sakila-installation.html](http://dev.mysql.com/doc/sakila/en/sakila-installation.html)

---

## 如何发现有问题的 SQL ？

**使用 mysql 慢查日志对有效率问题的 SQL 进行监控**
 
查看慢查询日志是否开启

```
# Empty set 表示没有开启
mysql> show variables like 'slow _query_log';
Empty set, 1 warning (0.00 sec)

# 设置开启，先执行下面的两个步骤在执行开启慢查询
mysql> set global slow_query_log=on;
Query OK, 0 rows affected (0.02 sec)
```

查看日志记录是否开启

```
# OFF 表示没有开启
mysql> show variables like '%log%';
log_queries_not_using_indexes  | OFF

# 设置开启
mysql> set global log_queries_not_using_indexes=on;
Query OK, 0 rows affected (0.00 sec)
```

查看 "超过查询时间" 的变量

```
# 值如果为 0.000000 表示不管什么查询都记录到慢查询日志中
mysql> show variables like 'long_query_time';
+-----------------+-----------+
| Variable_name   | Value     |
+-----------------+-----------+
| long_query_time | 10.000000 |
+-----------------+-----------+
1 row in set, 1 warning (0.00 sec)
```

**e.g.**

```
# 执行一个查询
mysql> select * from store limit 10;

# 获取日志文件的位置
mysql> show variables like 'slow%';
+---------------------+------------------------------------------------------+
| Variable_name       | Value                                                |
+---------------------+------------------------------------------------------+
| slow_launch_time    | 2                                                    |
| slow_query_log      | ON                                                   |
| slow_query_log_file | F:\mysql-5.7.22-winx64\data\DESKTOP-RALESOK-slow.log |
+---------------------+------------------------------------------------------+
```

慢查询日志的存储格式

```
执行 SQL 的主机信息
# User@Host: root[root] @ localhost [::1]  Id:    91

SQL的执行信息
# Query_time: 0.000246  Lock_time: 0.000076 Rows_sent: 2  Rows_examined: 2

SQL执行时间
SET timestamp=1539023953;

SQL的内容
select * from store limit 10;
```

---

## 慢查日志的分析工具 mysqldumpslow

- Liunx 安装 MySQL 后，默认安装此工具

- windows 下需要安装 Prel 环境，并配置好环境变量  
下载 ActivePerl 
[https://www.perl.org/get.html](https://www.perl.org/get.html)

```
# liunx 命令查看 mysqldumpslow 有哪些参数命令
mysqldumpslow.pl -h

# windows 环境要在 MySQL 的 bin 目录下执行，并且使用 perl 命令
F:\mysql-5.7.22-winx64\bin>perl mysqldumpslow.pl -h
```

## 查看日志中的记录

```
# 查看前3条记录
F:\mysql-5.7.22-winx64\bin>perl mysqldumpslow.pl -t 3 ../data/DESKTOP-RALESOK-slow.log | more

# 结果
Reading mysql slow query log from ../data/DESKTOP-RALESOK-slow.log
Count: 1  Time=0.00s (0s)  Lock=0.00s (0s)  Rows=2.0 (2), root[root]@localhost
  select * from store limit N

# Count : sql 执行的次数
# Time : sql 的执行时间
# Lock : sql 等待锁的时间
# Rows=2.0 : sql 每次返回的记录数
# (2) : 总共返回的记录数
# root[root]@localhost 执行者
# sql 的具体内容
```

## 慢查日志的分析工具 pt-query-digest

windows 下在 mysql/bin 文件夹下打开 git bash 窗口，输入命安装

```
curl -o pt-query-digest.pl https://www.percona.com/get/pt-query-digest
```

简单使用

```
# 获取使用帮助
perl pt-query-digest.pl --help

# 简单查询
perl pt-query-digest.pl ../data/mysql-slow.log
```



































