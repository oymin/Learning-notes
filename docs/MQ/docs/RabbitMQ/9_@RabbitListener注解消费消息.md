# @RabbitListener 注解消费消息

在 spring 容器中可以构建 `SimpleMessageListenerContainer` 来消费消息，我们也可以使用 `@RabbitListener` 来消费消息

@RabbitListener 注解指定目标方法来作为消费消息的方法，通过注解参数指定所监听的队列或者 Binding。使用 @RabbitListener 可以设置一个自己明确默认值的 RabbitListenerContainerFactory 对象。

可以在配置文件中设置 `RabbitListenerAnnotationBeanPostProcessor` 并通过 `<rabbit:annotation-driven/>` 来设置 @RabbitListener 的执行，当然也可以通过 @EnableRabbit 注解来启用 @RabbitListener

## 1. 示例代码

```java
@Configuration
public class ConsumerConfig {

    @Bean
    public ConnectionFactory connectionFactory(){
        CachingConnectionFactory factory = new CachingConnectionFactory();
        factory.setUri("amqp://zhihao.miao:123456@192.168.1.131:5672");
        return factory;
    }

    @Bean
    public RabbitListenerContainerFactory<?> rabbitListenerContainerFactory(ConnectionFactory connectionFactory){
        //SimpleRabbitListenerContainerFactory发现消息中有content_type有text就会默认将其转换成string类型的
        SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
        factory.setConnectionFactory(connectionFactory);
        return factory;
    }

}
```

定义消息处理器，`@RabbitListener` 注解标记的方法

```java
@Component
public class MessageHandler {

    @RabbitListener(queues = "zhihao.miao.order")
    public void handleMessage(byte[] message){
        System.out.println("消费消息");
        System.out.println(new String(message));
    }
}
```

测试:

![发送消息](../../image/01.png)

控制台打印：

```text
消费消息
你的订单已经生成。
```

如果发送的消息 `content_type` 的属性是 `text` ，那么接收的消息处理方法的参数就必须是 `String` 类型，如果是 `byte[]` 类型就会报错。

例如：

![指定content_type类型为text](../../image/02.png)

### 总结

如果消息属性中没有指定 `content_type`，则接收消息的处理方法接收类型是 `byte[]`,如果消息属性中指定 `content_type` 为 `text`，则接收消息的处理方法的参数类型是 `String` 类型。不管有没有指定 `content_type`，处理消息方法的参数类型是 `Message` 都不会报错。

```java
@Component
public class MessageHandler {

    //此时如果去掉content_type为text，那么会将消息转换成其每个字符的int类型
    //@RabbitListener(queues = "zhihao.miao.order")
    public void handleMessage(String message){
        System.out.println("消费消息");
        System.out.println(new String(message));
    }

    //此时不管属性中有没有content_type属性都能接收到数据
    @RabbitListener(queues = "zhihao.miao.order")
    public void handleMessage(Message message){
        System.out.println("====消费消息===handleMessage(message)");
        System.out.println(message.getMessageProperties());
        System.out.println(new String(message.getBody()));
    }
}
```

### 整个代码实现步骤

- 在启动入口增加@EnableRabbit注解

- 在 spring 容器中托管一个RabbitListenerContainerFactory 的 bean（默认的实现是org.springframework.amqp.rabbit.config.SimpleRabbitListenerContainerFactory）

- 写一个消息处理类托管到 spring 容器中，然后在具体的消息处理方法上增加 @RabbitListener 注解

- 具体的消息处理方法的参数是跟 MessageConverter 转换后的 java 对象有关。如果想要设置 MessageConverter，则需要在 RabbitListenerContainerFactory 的实例中去设置（setMessageConverter 方法）

---

## 2. 使用@Payload和@Headers注解

> **@Payload** 用于接收的消息体 body  
>
> **@Headers** 获取消息头 Header 的信息，其为一个 Map 键值对

示例：

```java
@Component
public class MessageHandler {

    //获取所有 Header 属性
    @RabbitListener(queues = "zhihao.miao.order")
    public void handleMessage(@Payload String body,@Headers Map<String,Object> headers){
        System.out.println(token);
        for (Map.Entry<String, Object> entry : headers.entrySet()) {
            System.out.println(entry.getKey() + "---" + entry.getValue());
        }
    }

    //获取单个指定的 Header 属性
    @RabbitListener(queues = "zhihao.miao.order")
    public void handleMessage(@Payload String body,@Header String token){
        System.out.println(token);
        System.out.println(body);
    }
}
```

![测试使用@Payload和@Headers注解](../../image/03.png)

---

## 3. @RabbitListener和@RabbitHandler搭配使用

@RabbitListener 可以标注在类上面，当使用在类上面的时候，需要配合 @RabbitHandler 注解一起使用，@RabbitListener 标注在类上面表示当有收到消息的时候，就交给带有 @RabbitHandler 的方法处理，具体找哪个方法处理，需要跟进 `MessageConverter` 转换后的 java 对象。

```java
import org.springframework.amqp.rabbit.config.SimpleRabbitListenerContainerFactory;
import org.springframework.amqp.rabbit.connection.CachingConnectionFactory;
import org.springframework.amqp.rabbit.connection.ConnectionFactory;
import org.springframework.amqp.rabbit.listener.RabbitListenerContainerFactory;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class ConsumerConfig {

    @Bean
    public ConnectionFactory connectionFactory(){
        CachingConnectionFactory factory = new CachingConnectionFactory();
        factory.setUri("amqp://zhihao.miao:123456@192.168.1.131:5672");
        return factory;
    }

    @Bean
    public RabbitListenerContainerFactory<?> rabbitListenerContainerFactory(ConnectionFactory connectionFactory){
        //SimpleRabbitListenerContainerFactory发现消息中有content_type有text就会默认将其转换成string类型的
        SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
        factory.setConnectionFactory(connectionFactory);
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

消费监听处理器方法：

```java
@Component
@RabbitListener(queues ="zhihao.miao.order")
public class MessageHandler {

    @RabbitHandler
    public void handleMessage(byte[] message){
        System.out.println("====消费消息handleMessage");
        System.out.println(new String(message));
    }

    @RabbitHandler
    public void handleMessage2(String message){
        System.out.println("====消费消息===handleMessage2");
        System.out.println(message);
    }
}
```

应用启动类：

```java
import java.util.concurrent.TimeUnit;

@EnableRabbit
@ComponentScan
public class Application {
    public static void main(String[] args) throws Exception{
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(Application.class);
        System.out.println("rabbit service startup");
        TimeUnit.SECONDS.sleep(3000);
        context.close();
    }
}
```

发送不包含 content_type 属性的消息和 content_type 属性为 text 的消息，控制台打印：

```text
rabbit service startup

====消费消息handleMessage
订单已经生成，请到订单-详情页面确认。

====消费消息===handleMessage2
订单已经生成，请到订单-详情页面确认。
```

---

## 4. @RabbitListener注解的containerFactory属性

`@RabbitListener` 注解的 `containerFactory` 属性可以指定一个RabbitListenerContainerFactory 的 bean，默认是找名字为 rabbitListenerContainerFactory 的实例

```java
@RabbitListener(queues ="order",containerFactory = "rabbitListenerContainerFactory")
```

当我们将 `@Configuration` 配置类中的 RabbitListenerContainerFactory 实例的对象名改掉的时候，发现就会报错。

```java
//更改了实例的对象名
@Bean("rabbitListenerFactory2")
public RabbitListenerContainerFactory<?> rabbitListenerContainerFactory(ConnectionFactory connectionFactory){
    ...
    return factory;
}
```

此时需要显示指定 `@RabbitListener` 的 `containerFactory` 属性

```java
@RabbitListener(queues ="order",containerFactory = "rabbitListenerFactory2")
```
