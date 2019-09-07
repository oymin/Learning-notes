# MySQL配置参数

## 1. MySQL 服务器参数

### 1.1 MySQL 获取配置信息路径

- 命令行参数

```bash
mysqld_safe --datadir=/data/sql_data
```

- 配置文件

```bash
# 查询读取文件的顺序
mysqld --help --verbose | grep -A 1 'Default options'

/etc/my.cnf
/etc/mysql/my.cnf
/usr/etc/my.cnf
~/.my.cnf
```

### 1.2 MySQL 配置参数的作用域

全局参数

```sql
set global 参数名 = 参数值;
set @@global.参数名 := 参数值;
```

会话参数

```sql
set session 参数名 = 参数值;
set @@session.参数名 := 参数值;
```

## 2. 内存配置相关参数

- 确定可以使用的内存的上限
- 确定 MySQL 的每个连接使用的内存
- 确定需要为操作系统保留多少内存
- 如何为缓存池分配缓存
  
```text
sort_buffer_size        每个线程排序缓存区的大小
join_buffer_size        每个线程连接缓冲区的大小
read_buffer_size        每个线程读缓冲区的大小(要是4k的倍数)
read_rnd_buffer_size    每个线程索引缓冲区的大小
Innodb_buffer_pool_size 定义 Innodb 缓存池的大小=[总内存-(每个线程所需的内存*连接数)-系统保留内存]
key_buffer_size         用于 myisam 引擎
```

## 3. MySQL 编码

### 3.1 查看 MySQL 编码

```sql
mysql> show variables like 'character%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | utf8                       |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | utf8                       |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
```

### 3.2 设置 MySQL 编码

```sql
# vi /etc/my.cnf

[mysqld]
character‐set‐server=utf8
collation‐server=utf8_general_ci
sql_mode='NO_ENGINE_SUBSTITUTION'

[mysql]
default‐character‐set = utf8

[mysql.server]
default‐character‐set = utf8

[mysqld_safe]
default‐character‐set = utf8

[client]
default‐character‐set = utf8

# 重启 mysql
service mysqld restart
```

## 4. mysql 的目录及配置文件

- /etc/my.cnf 这是mysql的主配置文件
- /var/lib/mysql   mysql数据库的数据库文件存放位置
- /var/log/mysql   数据库的日志输出存放位置
- Netstat –nltp 看是否能找到3306的端口，查看端口






