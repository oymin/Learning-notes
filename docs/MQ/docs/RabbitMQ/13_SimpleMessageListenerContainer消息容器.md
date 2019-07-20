# SimpleMessageListenerContainer消息容器

`SimpleMessageListenerContainer` 即简单消息监听容器,，用对于消费者的配置项.

`SimpleMessageListenerContainer` 可以进行动态设置，比如在运行中的应用可以动态的修改其消费者数量的大小、接收消息的模式等。

很多基于 rabbitMQ 的自制定化后端管控台在进行设置的时候，也是根据这一去实现的。所以可以看出 SpringAMQP 非常的强大。

> 它有监听单个或多个队列、自动启动、自动声明功能。
>
> 它可以设置事务特性、事务管理器、事务属性、事务并发、是否开启事务、回滚消息等。但是我们在实际生产中，很少使用事务，基本都是采用补偿机制。
>
> 它可以设置消息确认和自动确认模式、是否重回队列、异常捕获 Handler 函数。
>
> 它可以设置消费者标签生成策略、是否独占模式、消费者属性等。
>
> 它还可以设置具体的监听器、消息转换器等等。

```java
@Bean   //connectionFactory 也是要和最上面方法名保持一致
public SimpleMessageListenerContainer messageContainer(ConnectionFactory connectionFactory) {
    SimpleMessageListenerContainer container = new SimpleMessageListenerContainer(connectionFactory);
    container.setQueues(queue001(), queue002(), queue003());    //监听的队列
    container.setConcurrentConsumers(1);        //当前的消费者数量
    container.setMaxConcurrentConsumers(5);     //最大的消费者数量
    container.setDefaultRequeueRejected(false); //是否重回队列
    container.setAcknowledgeMode(AcknowledgeMode.AUTO); //签收模式
    container.setExposeListenerChannel(true);
    //消费端的标签策略
    container.setConsumerTagStrategy(new ConsumerTagStrategy() {
        @Override
        public String createConsumerTag(String queue) {
            return queue + "_" + UUID.randomUUID().toString();
        }
    });

    container.setMessageListener(new ChannelAwareMessageListener() {
        @Override
        public void onMessage(Message message, Channel channel) throws Exception {
            String msg = new String(message.getBody());
            log.info("消费者: " + msg);
        }
    });
    return container;
}
```

单元测试:

```java
@Test
public void testSendMessage2() throws Exception {
    //1 创建消息
    MessageProperties messageProperties = new MessageProperties();
    messageProperties.setContentType("text/plain");
    Message message = new Message("mq 消息1234   --spring.abc".getBytes(), messageProperties);

    rabbitTemplate.send("topic001", "spring.abc", message);

    rabbitTemplate.convertAndSend("topic001", "spring.amqp", "hello object message send!   -spring.amqp");
    rabbitTemplate.convertAndSend("topic002", "rabbit.abc", "hello object message send!  -rabbit.abc");
}
```

## 配置参考二

消费端：

```java
@Configuration
public class MQConfig {

    @Bean
    public ConnectionFactory connectionFactory(){
        CachingConnectionFactory factory = new CachingConnectionFactory();
        factory.setUri("amqp://zhihao.miao:123456@192.168.1.131:5672");
        return factory;
    }

    @Bean
    public RabbitAdmin rabbitAdmin(ConnectionFactory connectionFactory){
        RabbitAdmin rabbitAdmin = new RabbitAdmin(connectionFactory);
        return rabbitAdmin;
    }

    @Bean
    public RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory){
        RabbitTemplate rabbitTemplate = new RabbitTemplate(connectionFactory);
        return rabbitTemplate;
    }

    @Bean
    public SimpleMessageListenerContainer messageListenerContainer(ConnectionFactory connectionFactory){
        SimpleMessageListenerContainer container = new SimpleMessageListenerContainer();
        container.setConnectionFactory(connectionFactory);
        container.setQueueNames("zhihao.miao.order");

        MessageListenerAdapter adapter = new MessageListenerAdapter(new MessageHandler());
        //指定Json转换器
        adapter.setMessageConverter(new Jackson2JsonMessageConverter());
        //设置处理器的消费消息的默认方法
        adapter.setDefaultListenerMethod("onMessage");
        container.setMessageListener(adapter);

        return container;
    }
}
```

处理器，定义了二个消息处理方法，参数不一样：

```java
public class MessageHandler {

    public void onMessage(byte[] message){
        System.out.println("---------onMessage----byte-------------");
        System.out.println(new String(message));
    }

    public void onMessage(String message){
        System.out.println("---------onMessage---String-------------");
        System.out.println(message);
    }
}
```

消费端应用启动类：

```java
import java.util.concurrent.TimeUnit;

@ComponentScan
public class Application {
    public static void main(String[] args) throws Exception{
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(Application.class);
        System.out.println("===start up======");
        TimeUnit.SECONDS.sleep(60);
        context.close();
    }
}
```

生产端代码：

```java
@Configuration
public class MQConfig {

    @Bean
    public ConnectionFactory connectionFactory(){
        CachingConnectionFactory factory = new CachingConnectionFactory();
        factory.setUri("amqp://zhihao.miao:123456@192.168.1.131:5672");
        return factory;
    }

    @Bean
    public RabbitAdmin rabbitAdmin(ConnectionFactory connectionFactory){
        RabbitAdmin rabbitAdmin = new RabbitAdmin(connectionFactory);
        return rabbitAdmin;
    }

    @Bean
    public RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory){
        RabbitTemplate rabbitTemplate = new RabbitTemplate(connectionFactory);
        return rabbitTemplate;
    }
}
```

应用启动类，生产端传递的消息类型是 Order 类型，并且转换成 JSON 类型发送到队列中：

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.ComponentScan;

import java.time.LocalDateTime;

@ComponentScan
public class Application {
    public static void main(String[] args) throws Exception{
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(Application.class);

        RabbitTemplate rabbitTemplate = context.getBean(RabbitTemplate.class);
        System.out.println(rabbitTemplate);

        Order order = new Order();
        order.setId(1);
        order.setUserId(1000);
        order.setAmout(88d);
        order.setTime(LocalDateTime.now().toString());

        ObjectMapper mapper = new ObjectMapper();
        String json = mapper.writeValueAsString(order);
        System.out.println(json);

        rabbitTemplate.convertAndSend("","zhihao.miao.order",json);
        context.close();
    }
}
```

消费之后的控制台打印：

```java
---------onMessage----byte-------------
org.springframework.amqp.support.converter.Jackson2JsonMessageConverter fromMessage

警告: Could not convert incoming message with content-type [text/plain]

{"id":1,"userId":1000,"amout":88.0,"time":"2017-09-08T22:03:46.015"}
```

我们发现消费端还是将其当作字节数组来消费，转换器还是将其转换成byte[]

## 改造

生产者应用启动类重新指定了相应的 contentType 类型

```java
@ComponentScan
public class Application {
    public static void main(String[] args) throws Exception{
        ...

        MessageProperties messageProperties = new MessageProperties();
        //指定了 contentType 类型
        messageProperties.setContentType("application/json");
        Message message = new Message(json.getBytes(),messageProperties);

        rabbitTemplate.send("","zhihao.miao.order",message);
        context.close();
    }
}
```

此时消费端的 Jackson2JsonMessageConverter 类型转换器将其转换成 Map 类型，指定消费的方法参数类型是 Map 即可。

```java
public class MessageHandler {

    ...

    //添加解析参数map的方法
    public void onMessage(Map order){
        System.out.println("---------onMessage---map-------------");
        System.out.println(order.toString());
    }

}
```

此时消费端控制台打印，我们知道生产者传递 JSON 类型数据，消费者将其作为 Map 类型的数据进行处理：

```java
---------onMessage---map-------------
{id=1, userId=1000, amout=88.0, time=2017-10-15T22:47:03.500}
```

后续内容参考贴: [https://www.jianshu.com/p/83861676051c](https://www.jianshu.com/p/83861676051c)
