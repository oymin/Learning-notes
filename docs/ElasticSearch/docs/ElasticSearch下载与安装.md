# ElasticSearch下载与安装

## 1. ElasticSearch下载与安装

1、新版本要求至少 `jdk 1.8` 以上

2、下载 `ElasticSearch 6.4.1`：[https://www.elastic.co/cn/downloads/past-releases/elasticsearch-6-4-1](https://www.elastic.co/cn/downloads/past-releases/elasticsearch-6-4-1)

解压 `elasticsearch-6.4.1.zip` 后目录结构如下：

```text
  elasticsearch-6.4.1
    ├─bin
    |  └─────脚本目录，包括：启动、停止等可执行脚本
    ├─config
    |  └─────配置文件目录
    ├─data
    |  └─────索引目录，存放索引文件的地方
    ├─lib
    |  └─────存放依赖包目录
    ├─logs
    |  └─────日志目录
    ├─modules
    |  └─────模块目录，包括了es的功能模块
    ├─plugins
    |  └─────插件目录，es支持插件机制
    ├─LICENSE.txt
    │
    ├─NOTICE.txt
    │
    └─README.textile
```

### 1.1 ElasticSearch 配置文件

#### 三个配置文件

`elasticsearch.yml：` 用于配置 Elasticsearch 运行参数 

`jvm.options：` 用于配置 Elasticsearch JVM 设置

`log4j2.properties：` 用于配置 Elasticsearch 日志

> 注意：`elasticsearch.yml` 编码格式一定是 `utf-8`。


#### ES的配置文件的地址根据安装形式的不同而不同

使用 zip、tar 安装，配置文件的地址在安装目录的 `config` 下。

使用 RPM 安装，配置文件在 `/etc/elasticsearch` 下。

使用 MSI 安装，配置文件的地址在安装目录的 `config` 下，并且会自动将 `config` 目录地址写入环境变量 `ES_PATH_CONF`。

#### elasticsearch.yml 配置文件内容

```yml
#集群名称
cluster.name: xuecheng

#节点名称
node.name: xc_node_1

#设置绑定主机的ip地址，设置为0.0.0.0表示绑定任何ip，允许外网访问，生产环境建议设置为具体的ip
network.host: 0.0.0.0

#设置对外服务的http端口(web 管理平台窗口 9200)，默认为9200
http.port: 9200

#集群结点之间通信端口
transport.tcp.port: 9300

#指定该节点是否有资格被选举成为master结点，默认是true，如果原来的master宕机会重新选举新的master
node.master: true

#指定该节点是否存储索引数据，默认为true
node.data: true

#设置集群中master节点的初始列表
#discovery.zen.ping.unicast.hosts: ["0.0.0.0:9300", "0.0.0.0:9301", "0.0.0.0:9302"]

#设置ES自动发现节点连接超时的时间，默认为3秒，如果网络延迟高可设置大些。
#discovery.zen.ping.timeout: 9s

#主结点数量的最少值 ,此值的公式为：(master_eligible_nodes / 2) + 1 ，比如：有3个符合要求的主结点，那么这里要设置为2
discovery.zen.minimum_master_nodes: 1

#设置为true可以锁住ES使用的内存，避免内存与swap分区交换数据
bootstrap.memory_lock: false

#单机允许的最大存储结点数，通常单机启动一个结点建议设置为1，开发环境如果单机启动多个节点可设置大于1.
node.max_local_storage_nodes: 1

path.data: D:\Develop2\elasticsearch-6.4.1\data
path.logs: D:\Develop2\elasticsearch-6.4.1\logs

http.cors.enabled: true
http.cors.allow-origin: "*"
#http.cors.allow-origin: /.*/
```

#### jvm.options 文件配置

设置最小及最大的 JVM 堆内存大小：

在 jvm.options 中设置 -Xms 和 -Xmx：

1. 两个值设置为相等

2. 将 Xmx 设置为不超过物理内存的一半。

#### log4j2.properties 日志配置文件

日志文件设置，ES 使用 log4j，注意日志级别的配置。

### 1.2 启动 ES

进入 bin 目录，在 cmd 下运行：elasticsearch.bat

浏览器输入：[http://localhost:9200](http://localhost:9200)

---

## 2. 系统配置

在 linux 上根据系统资源情况，可将每个进程最多允许打开的文件数设置大些。

`su limit -n` 查询当前文件数

使用命令设置 limit:

先切换到 root，设置完成再切回 elasticsearch 用户。

```bash
sudo su

ulimit ‐n 65536

su elasticsearch
```

也可通过下边的方式修改文件进行持久设置

`/etc/security/limits.conf`

将下边的行加入此文件：

```bash
elasticsearch ‐ nofile 65536
```







