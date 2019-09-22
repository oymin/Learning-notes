# CentOS7 安装 ElcasticSearch

依赖环境：JDK 1.8 版本及以上

ElcasticSearch 下载地址：[https://www.elastic.co/cn/downloads/past-releases/elasticsearch-6-8-3](https://www.elastic.co/cn/downloads/past-releases/elasticsearch-6-8-3)

## 下载与安装

```bash
# 下载
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.8.3.tar.gz

# 解压
tar -zxvf elasticsearch-6.8.3.tar.gz -C /usr/local/

cd /usr/local/elasticsearch-6.8.3/

mkdir data

mkdir logs
```

## 配置

### 修改配置文件 elasticsearch.yml

```bash
cd /usr/local/elasticsearch-6.8.3/conf

vi elasticsearch.yml
```

`elasticsearch.yml` 配置内容：

```yml
#------------------------- Cluster -------------------------
cluster.name: es_cluster

# ------------------------- Node -------------------------
node.name: es_node1
node.master: true
node.data: true

#------------------------- Network -------------------------
network.host: 192.168.194.151
http.port: 9200
transport.tcp.port: 9300

#------------------------- Discovery -------------------------
discovery.zen.minimum_master_nodes: 1
discovery.zen.ping_timeout: 30s
#设置集群中master节点的初始列表
#discovery.zen.ping.unicast.hosts: ["0.0.0.0:9300", "0.0.0.0:9301", "0.0.0.0:9302"]

#------------------------- Memory -------------------------
#设置为true可以锁住ES使用的内存，避免内存与swap分区交换数据
bootstrap.memory_lock: false
bootstrap.system_call_filter: false

#------------------------- Paths -------------------------
path.data: /usr/local/elasticsearch-6.8.3/data
path.logs: /usr/local/elasticsearch-6.8.3/logs

#------------------------- Various -------------------------
http.cors.enabled: true
http.cors.allow-origin: "*"
#http.cors.allow-headers: Authorization
```

### 配置 limits.conf 文件

```yml
vi /etc/security/limits.conf

# 添加
* soft nofile 65536
* hard nofile 1028576
* soft nproc 65536
* hard nproc unlimited
* soft memlock unlimited
* hard memlock unlimited
```

提示：集群其它节点添加同样内容

### 配置 20-nproc.conf

```yml
vi /etc/security/limits.d/20-nproc.conf

# 添加
* soft nproc 4096
```

提示：集群其它节点添加同样内容

### 配置 sysctl.conf

```yml
vim /etc/sysctl.conf

# 添加
vm.max_map_count=262144

# 重新加载配置
sysctl -p
```

提示：集群其它节点添加同样内容

## 启动 ES

```bash
# 重新加载上面的配置
source /etc/profile

# ES不能用 root 用启动，创建用户
useradd es -g esgroup

# 设置密码
passwd es

# 设置权限
chown -R es:es /usr/local/elasticsearch-6.8.3/

# 切换到es用户
su es

cd /usr/local/elasticsearch-6.8.3/

# 启动1
# bin/elasticsearch -d -p /usr/local/elasticsearch-6.8.3/pid

# 启动2
#./bin/elasticsearch -d

# 启动3
./bin/elasticsearch
```

**查看是否启动：**

```json
curl http://192.168.194.151:9200

{
    "name": "es_node1",
    "cluster_name": "es_cluster",
    "cluster_uuid": "snOmaq43ScSyrz20VqhOHQ",
    "version": {
        "number": "6.8.3",
        "build_flavor": "default",
        "build_type": "tar",
        "build_hash": "0c48c0e",
        "build_date": "2019-08-29T19:05:24.312154Z",
        "build_snapshot": false,
        "lucene_version": "7.7.0",
        "minimum_wire_compatibility_version": "5.6.0",
        "minimum_index_compatibility_version": "5.0.0"
    },
    "tagline": "You Know, for Search"
}
```

### 关闭 ES

```bash
kill `cat /usr/local/elasticsearch-6.8.3/pid`
```

## 安装 Elasticsearch Head 插件

下载地址：[https://github.com/mobz/elasticsearch-head](https://github.com/mobz/elasticsearch-head)

### 安装 nodejs 和 rpm

```bash
yum install epel-release

yum install nodejs npm
```

### 下载并安装elasticsearch-head

```bash
wget https://github.com/mobz/elasticsearch-head/archive/master.zip

unzip master.zip

mv elasticsearch-head-master/ /usr/local/

cd /usr/local/elasticsearch-head-master/

npm install
```

出现报错：

```bash
npm ERR! phantomjs-prebuilt@2.1.16 install: `node install.js`
npm ERR! Exit status 1
npm ERR!
npm ERR! Failed at the phantomjs-prebuilt@2.1.16 install script 'node install.js'.
```

解决方案：

```bash
npm install phantomjs-prebuilt@2.1.16 --ignore-scripts

cd /usr/local/elasticsearch-head-master/

npm install
```

### 启动head插件

在 head 插件根目录下通过 `npm start` 即可启动 head 插件，实际执行的是 `package.json` 里面的 `grunt server 命令。

要想通过后台启动，得先安装 gruntcli ：

```bash
npm -g install grunt-cli

# grunt server & 命令启动，&表示后台启动
grunt server &

# Started connect web server on http://localhost:9100
```

## 安装 IK 分词器

下载地址：[https://github.com/medcl/elasticsearch-analysis-ik/releases](https://github.com/medcl/elasticsearch-analysis-ik/releases)

需要修改 `plugin-descriptor.properties` 文件，将其中的 es 版本号改为你所使用的版本号，即完成 ik 分词器的安装

```bash
[victor@hadoop102 elasticsearch]$ vi plugin-descriptor.properties

elasticsearch.version=6.0.0
#修改为
elasticsearch.version=6.8.3
```

至此，安装完成，重启ES！
