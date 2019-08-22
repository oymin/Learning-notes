# Slave_SQL_Running: No 错误

## 第一种解决方案

```sql
mysql> stop slave;
Query OK, 0 rows

mysql> set global sql_slave_skip_counter=1;
Query OK, 0 rows

mysql> start slave;
Query OK, 0 rows
```

## 第二种解决方案

在第一种解决不了的情况下

并且输入 `show slave status \G` 显示错误：

```sql
Last_SQL_Error : Could not execute Update_rows event on table mysql.user; Can't find record in 'user', Error_code: 1032; handler error HA_ERR_KEY_NOT_FOUND; the event's master log mysql-bin.000008, end_log_pos 1544
```

**解决方法：** 找到同步的 `binlog` 和 `POS` 点，然后重新做同步，这样就可以有新的中继日值了。

改变新的日志点：

```sql
CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000008',MASTER_LOG_POS=1544;
Query OK, 0 rows affected (0.00 sec)
```

执行成功，接着使用第一种方法，启动 `slave` 。

