# MyCAT基础

## 1. MyCAT关键配置文件

- `schema.xml` &emsp;用于配置逻辑库表及数据节点

- `rule.xml` &emsp;&emsp;用于配置表的分片规则

- `server.xml` &emsp;用于配置服务器权限

### 1.1 schema.xml 文件

`<schema>` 定义逻辑库和 `<table>` 定义逻辑表

```xml
<schema><tabel></tabel></sechema>
```

定义数据节点

```xml
<dataNode></dataNode>
```

定义数据主机节点的物理数据源

```xml
<dataHost></dataHost>
```

### 1.2 rule.xml 文件

定义表使用的分片规则

```xml
<tableRule name=""></tableRule>
```

定义分片算法

```xml
<function name=""></function>
```

### 1.3 server.xml 文件

用于定义系统配置

```xml
<system>
    <property name=""></property>
</system>
```

用于定义连接 MyCAT 的用户

```xml
<user></user>
```






















