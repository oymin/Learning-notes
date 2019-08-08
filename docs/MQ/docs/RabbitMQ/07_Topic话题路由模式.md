# Topic话题路由模式

话题路由模式：

1、每个消费者监听自己的队列，并且设置带统配符的 routingkey。

2、生产者将消息发给 broker，由交换机根据 routingkey 来转发消息到指定的队列。

## 生产者

声明交换机，指定 topic 类型

```java
import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

public class Producer_topic {

    private static final String QUEUE_INFORM_EMAIL = "queue_inform_email";
    private static final String QUEUE_INFORM_SMS = "queue_inform_sms";
    private static final String EXCHANGE_TOPIC_INFORM = "exchange_topic_inform";
    private static final String ROUTINGKEY_EMAIL = "amqp.#.email.#";
    private static final String ROUTINGKEY_SMS = "amqp.#.sms.#";

    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("127.0.0.1");
        factory.setPort(5672);
        factory.setUsername("guest");
        factory.setPassword("guest");
        factory.setVirtualHost("/");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        channel.exchangeDeclare(EXCHANGE_TOPIC_INFORM, BuiltinExchangeType.TOPIC);
        channel.queueDeclare(QUEUE_INFORM_EMAIL, true, false, false, null);
        channel.queueDeclare(QUEUE_INFORM_SMS, true, false, false, null);
        channel.queueBind(QUEUE_INFORM_SMS, EXCHANGE_TOPIC_INFORM, ROUTINGKEY_SMS);
        channel.queueBind(QUEUE_INFORM_EMAIL, EXCHANGE_TOPIC_INFORM, ROUTINGKEY_EMAIL);
        for (int i = 0; i < 5; i++) {
            String message = "send email inform message to suer";
            channel.basicPublish(EXCHANGE_TOPIC_INFORM, "amqp.email.sms", null, message.getBytes("utf-8"));
            System.out.println("send to mq: " + message);
        }
        connection.close();
        channel.close();
    }

}
```

## 消费者

可用 `6_Routing路由模式.md` 中的生产者代码
