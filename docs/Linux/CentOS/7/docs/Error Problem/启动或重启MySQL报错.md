# 启动MySQL报错

输入重启命令：

```shell
systemctl start mysqld.service
```

报错内容：

```shell
Job for mysqld.service failed because the control process exited with error code. See "systemctl status mysqld.service" and "journalctl -xe" for details.
```

解决办法：

在 `/etc/my.cnf` 添加 `server_id=xx` 值随便写

```shell
vi /etc/my.cnf

server_id=1
```

错误原因：

使用 MySQL 备份命令，需要在 `/etc/my.cnf` 文件中加入配置内容 `log_bin=mysql-bin`，负责会报错。

但是在设置 `log_bin` 日志的时候，没有设置 `server_id` 参数。

`server-id` 参数用于在复制中，为主库和备库提供一个独立的 `ID` ，以区分主库和备库；开启二进制文件的时候，需要设置这个参数。

所以 `log_bin` 和 `server_id` 做好同时都设置
