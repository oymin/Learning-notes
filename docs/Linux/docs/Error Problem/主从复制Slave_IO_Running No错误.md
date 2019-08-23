# 主从复制 Slave_IO_Running : No错误

查看日志报错：

```shell
cat /var/log/mysqld.log
```

报错内容：

```shell
Slave I/O for channel '': Fatal error: The slave I/O thread stops because master and slave have equal MySQL server UUIDs; these UUIDs must be different for replication to work. Error_code: 1593
```

中文意思为：因为主从拥有相同的 MySQL 服务器 uuid;这些 uuid 必须是不同的，以便复制能够工作。

## 报错原因

MySQL 数据目录中通常存在一个名为 `auto.cnf` 文件，存储了`server-uuid` 的值，如下所示：

```shell
server-uuid=f2d0efd6-6ab7-11e8-8fdd-fa163eda7360
```

由于主从两台虚拟机是复制的，`server-uuid` 的值相同所以报错，更改其中的 uuid ，解决报错
