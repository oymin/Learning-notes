# MessageConverter详解

涉及网络传输的应用序列化不可避免，发送端以某种规则将消息转成 byte 数组进行发送，接收端则以约定的规则进行 byte[] 数组的解析。

RabbitMQ 的序列化是指 Message 的 body 属性，即我们真正需要传输的内容，RabbitMQ 抽象出一个 MessageConvert 接口处理消息的序列化，其实现有 `SimpleMessageConverter`（默认）、`Jackson2JsonMessageConverter` 等。

当调用了 `convertAndSend` 方法时会使用 MessageConvert 进行消息的序列化

`SimpleMessageConverter` 对于要发送的消息体 body 为 byte[] 时不进行处理，如果是 String 则转成字节数组,如果是 Java 对象，则使用 jdk 序列化将消息转成字节数组，转出来的结果较大，含class类名，类相应方法等信息。因此性能较差

当使用 RabbitMQ 作为中间件时，数据量比较大，此时就要考虑使用类似 `Jackson2JsonMessageConverter` 等序列化形式以此提高性能

---

## 1. 消费方法中的MessageConverter

```java
@RabbitListener(queues = "order")
public void handleMessage(Message message){
    System.out.println("消费消息");
    System.out.println(new String(message.getBody()));
}
```

消息处理方法参数是由 MessageConverter 转化，若使用自定义 MessageConverter 则需要在 RabbitListenerContainerFactory 实例中去设置（默认 Spring 使用的实现是 SimpleRabbitListenerContainerFactory）

消息的 content_type 属性表示消息 body 数据以什么数据格式存储，接收消息除了使用 Message 对象接收消息（包含消息属性等信息）之外，还可直接使用对应类型接收消息 body 内容，但若方法参数类型不正确会抛异常：

> **application/octet-stream：** 二进制字节数组存储，使用 byte[]
>
> **application/x-java-serialized-object：** java 对象序列化格式存储，使用 Object、相应类型（反序列化时类型应该同包同名，否者会抛出找不到类异常）
>
> **text/plain：** 文本数据类型存储，使用 String
>
> **application/json：** JSON 格式，使用 Object、相应类型

---

## 2. Message内容对象序列化与反序列化

### 2.1 使用 Java 序列化与反序列化

默认的 SimpleMessageConverter 在发送消息时会将对象序列化成字节数组，若要反序列化对象，需要自定义 MessageConverter

```java
@Configuration
public class RabbitMQConfig {
    @Bean
    public RabbitListenerContainerFactory<?> rabbitListenerContainerFactory(ConnectionFactory connectionFactory) {
        SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
        factory.setConnectionFactory(connectionFactory);
        factory.setMessageConverter(new MessageConverter() {
            @Override
            public Message toMessage(Object object, MessageProperties messageProperties) throws MessageConversionException {
                return null;
            }

            @Override
            public Object fromMessage(Message message) throws MessageConversionException {
                try (ObjectInputStream ois = new ObjectInputStream(new ByteArrayInputStream(message.getBody()))) {
                    return (User) ois.readObject();
                } catch (Exception e) {
                    e.printStackTrace();
                    return null;
                }
            }
        });
        return factory;
    }

}
```

```java
@Component
@RabbitListener(queues = "consumer_queue")
public class Receiver {
    @RabbitHandler
    public void processMessage1(User user) {
        System.out.println(user.getName());
    }
}
```

### 2.2 使用 JSON 序列化与反序列化

RabbitMQ 提供了 Jackson2JsonMessageConverter 来支持消息内容 JSON 序列化与反序列化

消息发送者在发送消息时应设置 MessageConverter 为 Jackson2JsonMessageConverte：

```java
rabbitTemplate.setMessageConverter(new Jackson2JsonMessageConverter());
```

发送消息：

```java
User user = new User("linyuan");
rabbitTemplate.convertAndSend("topic.exchange","key.1",user);
```

消息消费者也应配置 MessageConverter 为 Jackson2JsonMessageConverter：

```java
@Configuration
public class RabbitMQConfig {
    @Bean
    public RabbitListenerContainerFactory<?> rabbitListenerContainerFactory(ConnectionFactory connectionFactory){
        SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
        factory.setConnectionFactory(connectionFactory);
        factory.setMessageConverter(new Jackson2JsonMessageConverter());
        return factory;
    }

}
```

消费消息：

```java
@Component
@RabbitListener(queues = "consumer_queue")
public class Receiver {
    @RabbitHandler
    public void processMessage1(@Payload User user) {
        System.out.println(user.getName());
    }

}
```

**注意：** 被序列化对象应提供一个无参的构造函数，否则会抛出异常
