# RabbitMQ的初始环境搭建

RabbitMQ 的生产者和消费者都属于客户端，创建两个maven工程，生产者工程和消费者工程，分别加入 RabbitMQ java client 的依赖。

## pom文件添加依赖

```xml
<dependency>
    <groupId>com.rabbitmq</groupId>
    <artifactId>amqp‐client</artifactId>
    <!‐‐此版本与 springboot 2.1.3.RELEASE版本匹配‐‐>
    <version>5.4.3</version>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring‐boot‐starter‐logging</artifactId>
</dependency>
```

## application配置文件

```yml
spring:
  application:
    name: test‐rabbitmq‐producer
  rabbitmq:
    host: 127.0.0.1
    username: guest
    password: guest
    virtualHost: /
```

```yml
spring:
  application:
    name: test‐rabbitmq‐consumer
  rabbitmq:
    host: 127.0.0.1
    username: guest
    password: guest
    virtualHost: /
```

## 扩展:设置QueueEnum队列枚举配置

```java
import lombok.Getter;

/**
 * 消息队列枚举配置
 */
@Getter
public enum QueueEnum {
    /**
     * 消息通知队列
     */
    MESSAGE_QUEUE("message.center.direct", "message.center.create", "message.center.create"),
    /**
     * 消息通知ttl队列
     */
    MESSAGE_TTL_QUEUE("message.center.topic.ttl", "message.center.create.ttl", "message.center.create.ttl");
    /**
     * 交换名称
     */
    private String exchange;
    /**
     * 队列名称
     */
    private String name;
    /**
     * 路由键
     */
    private String routeKey;

    QueueEnum(String exchange, String name, String routeKey) {
        this.exchange = exchange;
        this.name = name;
        this.routeKey = routeKey;
    }
}
```

### 使用枚举中的内容

来源链接：[https://www.jianshu.com/p/b74a14c7f31d](https://www.jianshu.com/p/b74a14c7f31d)

```java
/**
 * 消息通知 - 消息队列配置信息
 */
@Configuration
public class MessageRabbitMqConfiguration {
    /**
     * 消息中心实际消费队列交换配置
     *
     * @return
     */
    @Bean
    DirectExchange messageDirect() {
        return (DirectExchange) ExchangeBuilder
                .directExchange(QueueEnum.MESSAGE_QUEUE.getExchange())
                .durable(true)
                .build();
    }

    /**
     * 消息中心延迟消费交换配置
     *
     * @return
     */
    @Bean
    DirectExchange messageTtlDirect() {
        return (DirectExchange) ExchangeBuilder
                .directExchange(QueueEnum.MESSAGE_TTL_QUEUE.getExchange())
                .durable(true)
                .build();
    }

    /**
     * 消息中心实际消费队列配置
     *
     * @return
     */
    @Bean
    public Queue messageQueue() {
        return new Queue(QueueEnum.MESSAGE_QUEUE.getName());
    }
}
```
