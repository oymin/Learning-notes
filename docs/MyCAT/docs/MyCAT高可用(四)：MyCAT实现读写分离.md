# MyCAT实现读写分离.md

## 1. 操作步骤

1. 配置各个节点的 MySQL 主从复制

2. 配置 MyCAT 对后端 DB 进行读写分离

3. 滚动应用 MyCAT 配置

## 2. 演示环境说明

| 主机名 | IP | 角色 |
| ---- | ---- | ---- |
| Node-128 | 192.168.194.128 | MyCAT, ZK, MySQL |
| Node-129 | 192.168.194.129 | ZK, MySQL, HAProxy, Keepalived |
| Node-130 | 192.168.194.130 | ZK, MySQL, HAProxy, Keepalived |
| Node-131 | 192.168.194.131 | MyCAT, MySQL |
| Node-132 | 192.168.194.132 | MySQL |

## 3. 实现读写分离

### 3.1 复制 131 数据库到 132 节点

```sql
mysqldump --master-data=2 --single-transaction -uroot -p customer_db > node131.sql

scp node131.sql root@192.168.194.132:/root
```

132 节点导入数据库

```sqll
mysql -uroot -p customer_db < node131.sql
```

### 3.2 创建专门主从复制的用户

```sql
create user 'im_repl'@'192.168.75.%' identified by '123456';

create user 'im_mycat'@'192.168.194.%' identified by '123456';
```

给用户授权：

```sql
grant replication slave on *.* to 'im_repl'@'192.168.75.%';

GRANT SELECT, INSERT, UPDATE, DELETE, EXECUTE ON *.* TO 'im_mycat'@'192.168.194.%';
```

配置自定义链路：

```sql
change master to master_host = '192.168.194.131',master_user='im_repl',master_password='123456',MASTER_LOG_FILE='mysql-bin.000009',MASTER_LOG_POS=154;

start slave;
```

### 3.5 修改 schema.xml 文件

修改 128 节点 `schema.xml` 配置文件的 `dataHost name="mysql131"` 内容：

```xml
<dataHost name="mysql131" maxCon="1000" minCon="10" balance="1" writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
    <heartbeat>select user()</heartbeat>

    <writeHost host="192.168.194.131" url="192.168.194.131:3306" user="im_mycat" password="123456">
        <readHost host="192.168.194.132" url="192.168.194.132:3306" user="im_mycat" password="123456" />
    </writeHost>

    <writeHost host="192.168.194.132" url="192.168.194.132:3306" user="im_mycat" password="123456" />
</dataHost>
```

将配置文件初始化到 zookeerpre 进行同步：

```bash
/usr/local/mycat/bin/init_zk_data.sh
```
