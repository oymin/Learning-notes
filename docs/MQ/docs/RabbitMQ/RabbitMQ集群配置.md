# RabbitMQ 集群配置

创建如下配置文件位于：`/etc/rabbitmq` 目录下（这个目录需要自己创建）

- 环境变量配置文件：`rabbitmq-env.conf`

- 配置信息配置文件：`rabbitmq.config`（可以不创建和配置，修改）

## rabbitmq-env.conf 配置文件

### 关键参数配置

| 参数 | 参数值 |
| --- | --- |
| RABBITMQ_NODE_IP_ADDRESS | 本机IP地址 |
| RABBITMQ_NODE_PORT | 5672 |
| RABBITMQ_LOG_BASE | /var/lib/rabbitmq/log |
| RABBITMQ_MNESIA_BASE | /var/lib/rabbitmq/mnesia |

配置参考参数如下：

| 参数 | 参数值 |
| --- | --- |
| RABBITMQ_NODENAME | FZTEC-240088 节点名称 |
| RABBITMQ_NODE_IP_ADDRESS | 127.0.0.1 监听IP |
| RABBITMQ_NODE_PORT | 5672 监听端口 |
| RABBITMQ_LOG_BASE | /data/rabbitmq/log 日志目录 |
| RABBITMQ_PLUGINS_DIR | /data/rabbitmq/plugins 插件目录 |
| RABBITMQ_MNESIA_BASE | /data/rabbitmq/mnesia 后端存储目录 |

更详细的配置参见： [http://www.rabbitmq.com/configure.html#configuration-file](http://www.rabbitmq.com/configure.html#configuration-file)

## rabbitmq.config 配置文件信息修改

`/usr/lib/rabbitmq/lib/rabbitmq_server-3.6.4/ebin/rabbit.app` 和 `rabbitmq.config` 配置文件**配置任意一个**即可，我们进行配置如下：

```bash
vim /usr/lib/rabbitmq/lib/rabbitmq_server-3.6.4/ebin/rabbit.app
```

### 关键参数配置

| 参数 | 说明 |
| --- | --- |
| tcp_listerners | 设置rabbimq的监听端口，默认为[5672] |
| disk_free_limit | 磁盘低水位线，若磁盘容量低于指定值则停止接收数据，默认值为{mem_relative, 1.0},<br>即与内存相关联1：1，也可定制为多少byte |
| vm_memory_high_watermark | 设置内存低水位线，若低于该水位线，则开启流控机制，默认值是0.4，即内存总量的40% |
| hipe_compile |  将部分rabbimq代码用High Performance Erlang compiler编译，可提升性能，<br>该参数是实验性，若出现erlang vm segfaults，应关掉 |
| force_fine_statistics | 该参数属于rabbimq_management，若为 true 则进行精细化的统计，但会影响性能 |

更详细的配置参见：[http://www.rabbitmq.com/configure.html](http://www.rabbitmq.com/configure.html)

