# MyCAT：SQL拦截和SQL防火墙功能

MyCAT 可以自定义想要拦截的 sql 类型的操作记录。

## 1. SQL拦截功能

### 1.1 配置 server.xml 文件

```xml
<!-- 要拦截的 java 类 -->
<property name="sqlInterceptor">io.mycat.server.Interceptor.impl.StaticsSqlInterceptor</property>

<!-- 要拦截的 sql 语句类型 -->
<property name="sqlInterceptorType">UPDATE,DELETE,INSERT</property>

<!-- 拦截记录的文件 -->
<property name="sqlInterceptorFile">/tmp/sql.txt</property>
```

### 1.2 登录 mycat 验证拦截功能

执行一个 `update` 操作语句：

```sql
update order_master set address = '吴国' where order_id = 2;
```

执行完成后，`/tmp` 目录下生成一个 `sql20xx-xx-xx.txt` 文件，表示功能正确。

---

## 2. SQL防火墙功能

- 同意控制哪些用户可以通过哪些主机访问后端数据库。

- 统一屏蔽一些 SQL 语句，加强安全控制。

### 2.1 配置 server.xml 文件

```xml
<firewall>
    <whitehost>
        <host user="app_koax" host="127.0.0.1"></host>
        <host user="app_koax" host="192.168.194.129"></host>
    </whitehost>
    <blacklist check="true">
        <property name="noneBaseStatementAllow">true</property>
        <property name="deleteWhereNoneCheck">true</property>
    </blacklist>
</firewall>
```

**注意：** server.xml 的配置一定要注意标签位置的顺序 `<firewall>` 要配置在 `<user>` 标签的前面，否则启动报错。
