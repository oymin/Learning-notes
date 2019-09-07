# MySQL主从配置步骤

- 配置主从数据库服务器参数

- 在 Master 服务器上创建用于复制的数据库账号

- 备份 Master 服务器上的数据并初始化 Slave 服务器

- 启动复制链路

## 1. 配置主从数据库服务器参数

### 主数据库配置

`my.cnf` 配置：

```yml
#启动 binlong 日志
long_bin=/data/mysql/sql_log/mysql-bin

#配置 server_id
server_id=123
```

### 从数据库配置

`my.cnf` 配置：

```sql
long_bin=/data/mysql/sql_log/mysql-bin

server_id=124

relay_log=/data/mysql/sql_log/relay-bin

super_read_only=on

skip_slave_start=on

master_info_repository=TABLE
relay_log_info_repository=TABLE
```

## 2. 在 Master 服务器上创建用于复制的数据库账号

用于 IO 进程连接 Master 服务器获取 binlog 日志，需要 `REPLICATION SLAVE` 权限

```sql
#创建用户
create user 'repl'@'ip段' identified by 'password';

#授权
grant replication slave on *.* to 'repl'@'ip段';

flush privileges;
```

对主数据库全备份：

```sql
mysqldump -uroot -p --master-data=2 --single-transaction --routines --triggers --events 数据库名称 > 备份sql.sql
```

## 3. 初始化 slave 数据库

```sql
#恢复主库备份文件
mysql -uroot -p < 备份sql.sql
```

## 4. 启动复制链路

登录从库 MySQL：

启动基于日志点的复制链路

```sql
change master to 
master_host='master_host_ip',
master_user = 'replname',
master_password='password',
master_log_file='mysql_log_file_name',
master_log_pos=xxxx;
```

```sql
#启动
start slave ;

#查看启动状态
show slave status \G
```

```sql
#启动成功
Relay_Master_Log_File: mysql-bin.000001
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
```

---

## 2. 基于GTID的复制链路

GTID：也就是全局事务ID

配置参数：

```yml
# 启用gtid
gtid_mode=on

# 强制gtid事务的一致性
enforce-gtid-consistency

# mysql5.7中可以不启动
log-slave-updates=on
```

GTID启动复制链路的命令：

```sql
change master to 
master_host='master_host_ip',
master_user = 'replname',
master_password='password',
master_auto_position=1;
```

### GTID复制的限制

- 无法使用 Create table ... select 建立表

- 无法在事务中使用 create temporary table 建立临时表

- 无法使用关联更新同时更新事务表和非事务表

















