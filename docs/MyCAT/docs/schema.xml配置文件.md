# schema.xml 配置文件

文件用途：

- 配置逻辑库及逻辑表

- 配置逻辑表所存储的数据节点

- 配置数据节点所对应的物理数据库服务器信息

## 标签配置

```xml
<sechme>
    <table></table>
    <dataNode />
    <dataHost>
        <writeHost>
            <readHost></readHost>
        </writeHost>
    </dataHost>
</sechme>
```

### `<sechme>` 定义逻辑库

```xml
<!-- 
    name：逻辑库名，库名唯一; 
    sqlMaxLimit：限制f返回结果集的行数，-1为标识管理limit行限制; 
    checkSQLschema：判断是否检查发给MYCAT的SQL是否含库名
 -->
<schema name="TESTDB" checkSQLschema="false" sqlMaxLimit="100">
    <table name="travelrecord" dataNode="dn1,dn2,dn3" rule="auto-sharding-long" />
</schema>
```

### `<table>` 定义逻辑表

```xml
<!-- 
    name：逻辑表名，名称唯一;
    primaryKey：定义了逻辑表的主键;
    dataNode：定义表数据所存储的数据节点;
    rule：逻辑表分片规则，对应rule.xml中的<tableRule>
-->
<table name="customer" primaryKey="ID" dataNode="dn1,dn2" rule="sharding-by-intfile">
</table>
```

### `<dataNode>` 定义逻辑表存储的物理数据库

```xml
<!-- 
    name：数据节点的名称（唯一）；
    dataHost：数据库主机名称；
    database：物理数据库名称
 -->
<dataNode name="dn1" dataHost="localhost1" database="dn_db01" />
<dataNode name="dn2" dataHost="localhost1" database="dn_db02" />
```

### `<dataHost>` 定义后端数据库主机信息

```xml
<dataHost name="localhost1" maxCon="1000" minCon="10" balance="0" writeType="0"
        dbType="mysql" dbDriver="native" switchType="1" slaveThreshold="100">

    <heartbeat>select user()</heartbeat>

    <!-- can have multi write hosts -->
    <writeHost host="192.168.1.3" url="192.168.1.3:3306" user="root" password="123456">
        <!-- can have multi read hosts -->
        <readHost host="192.168.1.4" url="192.168.1.4:3306" user="root" password="123456" />
    </writeHost>

    <writeHost host="192.168.1.4" url="192.168.1.4:3316" user="root" password="123456" />

</dataHost>
```

#### dataHost 参数：

- name：这一组数据库服务器的名称（唯一）；
- maxCon：mycat和后端数据库的最大连接数
- minCon：mycat和后端数据库的最小连接数
- balance：
  - 0 表示不开启读写分离机制
  - 1 全部的 readHost 与 stand by writeHost 参与 select 语句的负载均衡
  - 2 所有的 readHost 与 writeHost 都参与 select 语句的负载均衡
  - 3 所有的 readHost 都参与 select 语句的负载均衡
- writeType：0 或 1
- dbType：数据库类型
- dbDriver：native 或 jdbc
- switchType：
  - 1 开启自动切换
  - -1 关闭自切换
- slaveThreshold：

#### \<heartbeat>select user()\</heartbeat>

表示如何检查后端数据库是否可用

#### \<writeHost> 与 \<readHost> 标签

组合定义一组读写请求访问的数据库实例

- `<writeHost>` 定义主从复制集群中的 master 服务器，负担写任务的数据库

- `<readHost>` 定义主从复制集群中的 slave 服务器，负担读任务的数据库，配置在 \<writeHost> 标签内

user 和 password 属性为真实连接后端数据库的账号和密码

