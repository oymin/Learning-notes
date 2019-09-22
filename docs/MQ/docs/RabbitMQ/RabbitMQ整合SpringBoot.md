# RabbitMQ 整合 SpringBoot

## 1. 生产端整合

### 配置文件

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

```yml
spring.rabbitmq.addresses=192.168.194.151:5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
spring.rabbitmq.virtual-host=/
spring.rabbitmq.connection-timeout=15000

spring.rabbitmq.publisher-confirms=true
spring.rabbitmq.publisher-returns=true
spring.rabbitmq.template.mandatory=true
```

注意一点❗：在发送消息的时候对 template 进行配置 mandatory=true 保证监听有效

### rabbitTemplate 回调函数

```java
import org.springframework.messaging.Message;
import org.springframework.messaging.MessageHeaders;
import org.springframework.messaging.support.MessageBuilder;

@Component
public class RabbitSender {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    /**
     * 检查消息是否发送出去
     * 回调函数: confirm确认
     */
    final ConfirmCallback confirmCallback = new RabbitTemplate.ConfirmCallback() {
        @Override
        public void confirm(CorrelationData correlationData, boolean ack, String cause) {
            System.err.println("correlationData: " + correlationData);
            System.err.println("ack: " + ack);
            if (!ack) {
                System.err.println("异常处理....");
            }
        }
    };

    /**
     * 回调函数: return返回
     */
    final ReturnCallback returnCallback = new RabbitTemplate.ReturnCallback() {
        @Override
        public void returnedMessage(org.springframework.amqp.core.Message message, int replyCode, String replyText,
                                    String exchange, String routingKey) {
            System.err.println("return exchange: " + exchange + ", routingKey: "
                    + routingKey + ", replyCode: " + replyCode + ", replyText: " + replyText);
        }
    };

    /**
     * 发送消息方法调用: 构建Message消息
     * @param message
     * @param properties
     * @throws Exception
     */
    public void send(Object message, Map<String, Object> properties) throws Exception {
        MessageHeaders mhs = new MessageHeaders(properties);
        Message msg = MessageBuilder.createMessage(message, mhs);
        rabbitTemplate.setConfirmCallback(confirmCallback);
        rabbitTemplate.setReturnCallback(returnCallback);
        //id + 时间戳 全局唯一
        CorrelationData correlationData = new CorrelationData("1234567890");
        rabbitTemplate.convertAndSend("topic001", "spring.abc", msg, correlationData);
    }

    /**
     * 发送消息方法调用: 构建自定义对象消息
     * @param order
     * @throws Exception
     */
    public void sendOrder(Order order) throws Exception {
        rabbitTemplate.setConfirmCallback(confirmCallback);
        rabbitTemplate.setReturnCallback(returnCallback);
        //id + 时间戳 全局唯一
        CorrelationData correlationData = new CorrelationData("0987654321");
        //发送错误路由，以触发 ReturnCallback
        rabbitTemplate.convertAndSend("topic002", "springboot.def", order, correlationData);
    }

}
```

测试代码：

```java
@Autowired
private RabbitSender rabbitSender;

private static SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss.SSS");

@Test
public void testSender1() throws Exception {
    Map<String, Object> properties = new HashMap<>();
    properties.put("number", "12345");
    properties.put("send_time", simpleDateFormat.format(new Date()));
    rabbitSender.send("Hello RabbitMQ For Spring Boot!", properties);
}

@Test
public void testSender2() throws Exception {
    Order order = new Order("001", "第一个订单");
    rabbitSender.sendOrder(order);
}
```

`testSender1()` 测试结果：

```java
correlationData: CorrelationData [id=1234567890]
ack: true
```

`testSender2()` 测试结果：

```java
return exchange: topic002, routingKey: springboot.def, replyCode: 312, replyText: NO_ROUTE
correlationData: CorrelationData [id=0987654321]
ack: true
```

## 2. 消费端配置

### 配置文件

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

```yml
spring.rabbitmq.addresses=192.168.194.151:5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
spring.rabbitmq.virtual-host=/
spring.rabbitmq.connection-timeout=15000

spring.rabbitmq.listener.simple.acknowledge-mode=manual
spring.rabbitmq.listener.simple.concurrency=5
spring.rabbitmq.listener.simple.max-concurrency=10

spring.rabbitmq.listener.order.queue.name=queue-2
spring.rabbitmq.listener.order.queue.durable=true
spring.rabbitmq.listener.order.exchange.name=exchange-2
spring.rabbitmq.listener.order.exchange.durable=true
spring.rabbitmq.listener.order.exchange.type=topic
spring.rabbitmq.listener.order.exchange.ignoreDeclarationExceptions=true
spring.rabbitmq.listener.order.key=springboot.*
```

### @RabbitListener 注解使用

```java
@Component
public class RabbitReceiver {

    @RabbitListener(bindings = @QueueBinding(
            value = @Queue(value = "queue001", durable = "true"),
            exchange = @Exchange(value = "topic001",
                    durable = "true",
                    type = "topic",
                    ignoreDeclarationExceptions = "true"),
            key = "spring.*"))

    @RabbitHandler
    public void onMessage(Message message, Channel channel) throws Exception {
        System.err.println("--------------------------------------");
        System.err.println("消费端Payload: " + message.getPayload());
        Long deliveryTag = (Long) message.getHeaders().get(AmqpHeaders.DELIVERY_TAG);
        //手工ACK
        channel.basicAck(deliveryTag, false);
    }

    /**
     * @param order
     * @param channel
     * @param headers
     * @throws Exception
     */
    @RabbitListener(bindings = @QueueBinding(
            value = @Queue(value = "${spring.rabbitmq.listener.order.queue.name}",
                    durable = "${spring.rabbitmq.listener.order.queue.durable}"),
            exchange = @Exchange(value = "${spring.rabbitmq.listener.order.exchange.name}",
                    durable = "${spring.rabbitmq.listener.order.exchange.durable}",
                    type = "${spring.rabbitmq.listener.order.exchange.type}",
                    ignoreDeclarationExceptions = "${spring.rabbitmq.listener.order.exchange.ignoreDeclarationExceptions}"),
            key = "${spring.rabbitmq.listener.order.key}"
    )
    )

    @RabbitHandler
    public void onOrderMessage(@Payload com.mq.bootproducer.entity.Order order,
                               Channel channel,
                               @Headers Map<String, Object> headers) throws Exception {
        System.err.println("--------------------------------------");
        System.err.println("消费端order: " + order.getId());
        Long deliveryTag = (Long) headers.get(AmqpHeaders.DELIVERY_TAG);
        //手工ACK
        channel.basicAck(deliveryTag, false);
    }

}
```











