# 垂直分库(三)：配置 MYCAT 垂直分库

- 使用 schema.xml 配置逻辑库

- 使用 server.xml 配置系统变量及用户权限

- 由于没有用到水平分片所以不需要配置 rule.xml

## 1. 配置 schema.xml

### 1.1 创建用于 mycat 访问数据库的用户

```sql
create user im_mycat@'192.168.75.%' identified by '123456';
```

给用户授权：

```sql
grant select,insert,update,delete on *.* to im_mycat@'192.168.75.%';
```

### 1.2 schema.xml 文件配置

```xml
<?xml version="1.0"?>
<!DOCTYPE mycat:schema SYSTEM "schema.dtd">
<mycat:schema xmlns:mycat="http://io.mycat/">

    <schema name="master_db" checkSQLschema="false" sqlMaxLimit="100">
        <table name="order_master" primaryKey="order_id" dataNode="ordb" />
        <table name="order_detail" primaryKey="order_detail_id" dataNode="ordb" />
        <table name="order_cart" primaryKey="cart_id" dataNode="ordb" />
        <table name="order_customer_addr" primaryKey="customer_addr_id" dataNode="ordb" />
        <table name="region_info" primaryKey="region_id" dataNode="ordb" />
        <table name="shipping_info" primaryKey="ship_id" dataNode="ordb" />
        <table name="warehouse_info" primaryKey="w_id" dataNode="ordb" />
        <table name="warehouse_proudct" primaryKey="wp_id" dataNode="ordb" />

        <table name="customer_balance_log" primaryKey="balance_id" dataNode="custdb" />
        <table name="customer_inf" primaryKey="customer_inf_id" dataNode="custdb" />
        <table name="customer_level_inf" primaryKey="customer_level" dataNode="custdb" />
        <table name="customer_login" primaryKey="customer_id" dataNode="custdb" />
        <table name="customer_login_log" primaryKey="login_id" dataNode="custdb" />
        <table name="customer_point_log" primaryKey="point_id" dataNode="custdb" />

        <table name="product_brand_info" primaryKey="brand_id" dataNode="prodb" />
        <table name="product_category" primaryKey="category_id" dataNode="prodb" />
        <table name="product_comment" primaryKey="comment_id" dataNode="prodb" />
        <table name="product_info" primaryKey="product_id" dataNode="prodb" />
        <table name="product_pic_info" primaryKey="product_pic_id" dataNode="prodb" />
        <table name="product_supplier_info" primaryKey="supplier_id" dataNode="prodb" />

    </schema>

    <dataNode name="ordb" dataHost="mysql129" database="order_db" />
    <dataNode name="prodb" dataHost="mysql130" database="product_db" />
    <dataNode name="custdb" dataHost="mysql131" database="customer_db" />


    <dataHost name="mysql129" maxCon="1000" minCon="10" balance="1"
              writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
        <heartbeat>select user()</heartbeat>

        <writeHost host="192.168.194.129" url="192.168.194.129:3306" user="im_mycat" password="123456" />
    </dataHost>

    <dataHost name="mysql130" maxCon="1000" minCon="10" balance="1"
              writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
        <heartbeat>select user()</heartbeat>

        <writeHost host="192.168.194.130" url="192.168.194.130:3306" user="im_mycat" password="123456" />
    </dataHost>

    <dataHost name="mysql131" maxCon="1000" minCon="10" balance="1"
              writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
        <heartbeat>select user()</heartbeat>

        <writeHost host="192.168.194.131" url="192.168.194.131:3306" user="im_mycat" password="123456" />
    </dataHost>
</mycat:schema>
```

---

## 2. 配置 server.xml

```xml
<mycat:server xmlns:mycat="http://io.mycat/">
    <system>

        <property name="serverPort">8066</property>
        <property name="managerPort">9066</property>
        <property name="nonePasswordLogin">0</property>
        <property name="bindIp">0.0.0.0</property>
        <property name="frontWriteQueueSize">2048</property>

        <property name="charset">utf8</property>
        <property name="txIsolation">2</property>
        <property name="processors">8</property>
        <property name="idleTimeout">1800000</property>
        <property name="sqlExecuteTimeout">300</property>
        <property name="useGlobleTableCheck">0</property>
        <property name="useSqlStat">0</property>
        <property name="sequnceHandlerType">2</property>
        <property name="defaultMaxLimit">100</property>
        <property name="maxPacketSize">104857600</property>

    </system>

    <user name="app_koax" defaultAccount="true">
        <property name="password">123456</property>
        <property name="schemas">master_db</property>
    </user>

</mycat:server>
```

### 给用户密码加密

在 `mycat` 的 `lib/` 目录下执行命令：

```bash
java -cp Mycat-server-1.6.5-release.jar io.mycat.util.DecryptUtil 0:app_koax:123456
```

生成密码：

```bash
GO0bnFVWrAuFgr1JMuMZkvfDNyTpoiGU7n/Wlsa151CirHQnANVk3NzE3FErx8v6pAcO0ctX3xFecmSr+976QA==
```

在 `server.xml` 文件中 `<user>` 将 `123456` 修改成加密密码：

```xml
<!-- 添加属性：标识启用加密密码 -->
<property name="usingDecrypt">1</>

<property name="password">GO0bnFVWrAuFgr1JMuMZkvfDNyTpoiGU7n/Wlsa151CirHQnANVk3NzE3FErx8v6pAcO0ctX3xFecmSr+976QA==</property>
```
