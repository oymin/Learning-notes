# RabbitMQ集群延迟队列插件应用

延迟队列可以做什么事情?

比如消息的延迟推送、定时任务（消息）的执行。包括一些消息重试策略的配置使用，以及用于业务消峰限流、降级的异步延迟消息机制，都是延迟队列的实际应用场景。

## 延迟插件的安装

### 第一步： 下载插件

下载地址：[http://www.rabbitmq.com/community-plugins.html](http://www.rabbitmq.com/community-plugins.html)

如何使用：[Github: rabbitmq/rabbitmq-rtopic-exchange](https://github.com/rabbitmq/rabbitmq-delayed-message-exchange)

### 第二步：把下载好的插件放到指定目录

```bash
/usr/lib/rabbitmq/lib/rabbitmq_server-3.7.17/plugins/
```

### 第三步：启动插件

```bash
rabbitmq-plugins enable rabbitmq_delayed_message_exchange
```

### 第四步：创建一个延迟交换机

访问地址 [http://192.168.194.151:15672/#exchanges](http://192.168.194.151:15672/#exchanges)

Exchange Type： 新增了 `x-delayed-message` 类型选项

![image](../image/25.png)

### 第五步：创建队列

![image](../image/26.png)

### 第六步：绑定队列到交换机

![image](../image/27.png)

### 第七步：发送消息

![image](../image/28.png)

**20 秒后消息发送到队列：**

![image](../image/29.png)

## 代码中的使用

声明 Exchange 交换机：

参数添加：`"x-delayed-type", "direct"`

```java
// ... elided code ...
Map<String, Object> args = new HashMap<String, Object>();
args.put("x-delayed-type", "direct");
channel.exchangeDeclare("my-exchange", "x-delayed-message", true, false, args);
// ... more code ...
```

创建 Message 消息：

Properteis 中添加 key ：`"x-delay"`

```java
// ... elided code ...
byte[] messageBodyBytes = "delayed payload".getBytes("UTF-8");
Map<String, Object> headers = new HashMap<String, Object>();
headers.put("x-delay", 5000);
AMQP.BasicProperties.Builder props = new AMQP.BasicProperties.Builder().headers(headers);
channel.basicPublish("my-exchange", "", props.build(), messageBodyBytes);

byte[] messageBodyBytes2 = "more delayed payload".getBytes("UTF-8");
Map<String, Object> headers2 = new HashMap<String, Object>();
headers2.put("x-delay", 1000);
AMQP.BasicProperties.Builder props2 = new AMQP.BasicProperties.Builder().headers(headers2);
channel.basicPublish("my-exchange", "", props2.build(), messageBodyBytes2);
// ... more code ...
```
