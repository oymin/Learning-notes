# 水平分库：全局自增ID唯一性解决方案

当水平分库中的表的 ID 是自增 ID 时，向不同的表插入数据自增产生的 ID 可能会产生相同的 ID。

## 解决方案一：使用 MyCAT 全局自增 ID 功能

### 1. 在一个数据库节点中新建一个数据库 `mycat` (名称随意)

### 2. 导入 `mycat/conf/` 目录下的 `dbseq.sql` 脚本到 `mycat` 数据库中

```sql
 mysql -uroot -p mycat < dbseq.sql
```

该脚本只导入了 1 张表：`MYCAT_SEQUENCE`

```sql
mysql> select * from MYCAT_SEQUENCE;

+--------+---------------+-----------+
| name   | current_value | increment |
+--------+---------------+-----------+
| GLOBAL |             1 |         1 |
+--------+---------------+-----------+
1 row in set (0.00 sec)
```

向表中插入要使用全局自增 ID 的表名（要大写）：

```sql
# 自增 Id 初始值为 1，并以 1 递增
insert into MYCAT_SEQUENCE values ('ORDER_MASTER', 1, 1);
```

### 3. 修改 `mycat/conf/sequence_db_conf.properties` 文件

文件指定了相关表和函数指定的节点

```yml
# GLOBAL 对应 MYCAT_SEQUENCE 表中 name 字段的 GLOBAL
GLOBAL=mycat
# ORDER_MASTER 要使用全局自增id的表名，要大写
ORDER_MASTER=mycat
```

### 4. 修改 `server.xml` 文件，添加属性

```xml
<property name="sequnceHandlerType">1</property>
```

`sequnceHandlerType` 属性有 5 种取值：

    0 以本地文件方式生成序列号
    1 以数据库的方式
    2 以时间戳序列的方式
    3 分布式 zokeerper ID 生成器方式生成序列
    4 zokeerper 递增方式

### 5. 修改 `schema.xml` 配置

1、表的 `<table>` 要增加 `autoIncrement` 属性

```xml
<table name="order_master" primaryKey="order_id" dataNode="orderdb01,orderdb02,orderdb03,orderdb04" rule="order_master_rule" autoIncrement="true" />
```

2、如果 `mycat` 的数据库节点没有配置，则要增加

```xml
<dataNode name="mycat" dataHost="mysql128" database="mycat" />

<dataHost name="mysql128" maxCon="1000" minCon="10" balance="1"
            writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
    <heartbeat>select user()</heartbeat>

    <writeHost host="192.168.194.128" url="192.168.194.128:3306" user="im_mycat" password="123456" />
</dataHost>
```

3、检查 `<dataHost>` 标签中 `im_mycat` 用户是否有执行方法和存储过程的权限

```sql
mysql> show grants for im_mycat@'192.168.194.%';
+---------------------------------------------------------------------------+
| Grants for im_mycat@192.168.194.%                                         |
+---------------------------------------------------------------------------+
| GRANT SELECT, INSERT, UPDATE, DELETE ON *.* TO 'im_mycat'@'192.168.194.%' |
+---------------------------------------------------------------------------+
```

只有读写权限，需要添加

```sql
grant execute on *.* to 'im_mycat'@'192.168.194.%';
```

```sql
mysql>  show grants for im_mycat@'192.168.194.%';
+------------------------------------------------------------------------------------+
| Grants for im_mycat@192.168.194.%                                                  |
+------------------------------------------------------------------------------------+
| GRANT SELECT, INSERT, UPDATE, DELETE, EXECUTE ON *.* TO 'im_mycat'@'192.168.194.%' |
+------------------------------------------------------------------------------------+
```
