# CentOS7 安装 Kibana

安装方式：`tar.gz` 格式压缩包

下载与 ElasticSearch 对应的版本：[https://www.elastic.co/downloads/kibana](https://www.elastic.co/downloads/kibana)

## 安装

```bash
tar -zxvf kibana-6.8.3-linux-x86_64.tar.gz -C /usr/local/
```

**配置 kibana.yml 文件：**

```yml
server.port: 5601

elasticsearch.url: "http://192.168.194.151:9200"

server.host: "0.0.0.0"

kibana.index: ".kibana"
```

**启动：**

```bash
cd /usr/local/kibana683/bin

./kibana &
```

**登录界面管理控制台：**

访问地址：[http://192.168.194.151:5601](http://192.168.194.151:5601)

进入 **Dev Tools 选项界面**
