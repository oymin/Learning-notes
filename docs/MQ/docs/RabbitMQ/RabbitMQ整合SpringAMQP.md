# RabbitMQ 整合 SpringAMQP

## SpringAMQP 的核心组件

- RabbitAdmin
- Spring AMQP 声明
- RabbitTemplate 消息模板
- SimpleMessageListenerContainer 简单监听容器
- MessageListenerAdapter 消息监听适配器
- MessageConverter 消息转换器

## 1. RabbitAdmin

RabbitAdmin 类可以很好的操作 RabbitMQ，在 Spring 中直接进行注入即可。

注意❗：`autoStartup` 必须要设置为 true, 否则 Spring 容器不会加载 RabbitAdmin 类

RabbitAdmin 底层实现就是从 Spring 容器中获取 Exchange , Binding , RoutingKey 以及 Queue 的 `@Bean` 声明。

然后使用 RabbitTemplate 的 execute 方法执行对应的声明、修改、删除等一系列 RabbitMQ 基础功能操作。

例如：添加一个交换机、删除一个绑定、清空和一个队列里的消息等等。

### 代码示例

#### pom 依赖

```xml
<!-- springboot version 2.1.6.RELEASE -->
<dependency>
    <groupId>com.rabbitmq</groupId>
    <artifactId>amqp-client</artifactId>
    <version>5.4.3</version>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

#### RabbitAdmin 配置类

```java
@Configuration
@ComponentScan({"com.mq.demo.*"})
public class RabbitMqConfig {

    @Bean
    public ConnectionFactory connectionFactory() {
        CachingConnectionFactory connectionFactory = new CachingConnectionFactory();
        connectionFactory.setAddresses("192.168.194.151:5672");
        connectionFactory.setUsername("guest");
        connectionFactory.setPassword("guest");
        connectionFactory.setVirtualHost("/");
        return connectionFactory;
    }

    @Bean
    public RabbitAdmin rabbitAdmin(ConnectionFactory connectionFactory){
        RabbitAdmin rabbitAdmin = new RabbitAdmin(connectionFactory);
        rabbitAdmin.setAutoStartup(true);
        return rabbitAdmin;
    }

}
```

#### 测试代码

```java
@Autowired
private RabbitAdmin rabbitAdmin;

@Test
public void testAdmin() throws Exception {
    //声明交换机
    rabbitAdmin.declareExchange(new DirectExchange("test.direct", false, false));
    rabbitAdmin.declareExchange(new TopicExchange("test.topic", false, false));
    //声明队列
    rabbitAdmin.declareQueue(new Queue("test.direct.queue", false));
    rabbitAdmin.declareQueue(new Queue("test.topic.queue", false));
    //绑定
    rabbitAdmin.declareBinding(new Binding("test.direct.queue",
            Binding.DestinationType.QUEUE,
            "test.direct",
            "direct",
            new HashMap<>(16)));

    rabbitAdmin.declareBinding(BindingBuilder
            .bind(new Queue("test.topic.queue", false))
            .to(new TopicExchange("test.topic", false, false))
            .with("user.#"));

    //清空队列数据
    rabbitAdmin.purgeQueue("test.topic.queue", false);
}
```

## 2. Spring AMQP 声明

在 RabbitMQ 基础 API 里面声明一个 Exchang、声明一个绑定、一个队列

```java
channel.exchangeDeclare(exchangeName, BuiltinExchangeType.TOPIC, true, false, false, null);
channel.queueDeclare(queueName, true, false, false, null);
channel.queueBind(queueName, exchangeName, routingKey);
```

使用 SpringAMQP 去声明，就需要使用 SpringAMQP 的如下模式，即声明 `@Bean` 的方式

```java
@Configuration
@ComponentScan({"com.mq.demo.*"})
public class RabbitMqConfig {

    @Bean
    public TopicExchange exchange001() {
        return new TopicExchange("topic001", true, false);
    }

    @Bean
    public Queue queue001() {
        return new Queue("queue001", true); //队列持久
    }

    @Bean
    public Binding binding001() {
        return BindingBuilder.bind(queue001()).to(exchange001()).with("spring.*");
    }

}
```

## 3. RabbitTemplate 消息模板

我们在与 SpringAMQP 整合的时候进行发送消息的关键类。

该类提供了丰富的发送消息方法，包括可靠性投递消息方法、回调监听消息接口的 `ConfirmCallback`, 返回值确认接口 `ReturnCallback` 等待。同样我们需要进行注入到 Spring 容器中，然后直接使用。

在于 Spring 整合时需要实例化，但是在于 SpringBoot 整合时，在配置文件里添加配置即可。

### 代码示例

```java
@Configuration
@ComponentScan({"com.mq.demo.*"})
public class RabbitMqConfig {

    @Bean
    public RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory){
        RabbitTemplate rabbitTemplate = new RabbitTemplate(connectionFactory);
        return rabbitTemplate;
    }

}
```

```java
@Test
public void testSendMessage() throws Exception {
    //1 创建消息
    MessageProperties messageProperties = new MessageProperties();
    messageProperties.getHeaders().put("desc", "信息描述..");
    messageProperties.getHeaders().put("type", "自定义消息类型..");
    Message message = new Message("Hello RabbitMQ".getBytes(), messageProperties);

    rabbitTemplate.convertAndSend("topic001", "spring.amqp", message, new MessagePostProcessor() {
        @Override
        public Message postProcessMessage(Message message) throws AmqpException {
            System.err.println("------添加额外的设置---------");
            message.getMessageProperties().getHeaders().put("desc", "额外修改的信息描述");
            message.getMessageProperties().getHeaders().put("attr", "额外新加的属性");
            return message;
        }
    });
}

@Test
public void testSendMessage2() throws Exception {
    //1 创建消息
    MessageProperties messageProperties = new MessageProperties();
    messageProperties.setContentType("text/plain");
    Message message = new Message("mq 消息1234".getBytes(), messageProperties);

    rabbitTemplate.send("topic001", "spring.abc", message);

    rabbitTemplate.convertAndSend("topic001", "spring.amqp", "hello object message send!");
    rabbitTemplate.convertAndSend("topic002", "rabbit.abc", "hello object message send!");
}
```

## 4. SimpleMessageListenerContainer 简单消息监听容器

这个类非常的强大，我们可以对他进行很多设置，对于消费者的配置项，这个类都可以满足。

### 特点

- 监听队列（多个队列）、自动启动、自动声明功能。
- 设置事务特性、事务管理器、事务属性、事务容量（并发）、是否开启事务、回滚消息等。
- 设置消费者数量、最小最大数量、批量消费。
- 设置消息确认和自动确认模式、是否重回队列、一场捕获 handler 函数。
- 设置消费者标签生产策略、是否独占模式、消费者属性等。
- 设置具体的监听器、消息转换器等待。

注意❗： `SimpleMessageListenerContainer` 可以进行动态设置，比如在运行中的应用可以动态的修改其消费数量的大小、接收消息的模式等。

很多基于 RabbitMQ 的自制定化后端管控太在进行动态设置的是否，也是根据这一特性去实现。所以可以看出 SpringAMQP 非常的强大。

```java
@Bean
public SimpleMessageListenerContainer messageListenerContainer(ConnectionFactory connectionFactory) {
    SimpleMessageListenerContainer container = new SimpleMessageListenerContainer(connectionFactory);
    container.setQueues(queue001(), queue002(), queue003(), queue_image(), queue_pdf());
    container.setConcurrentConsumers(1); //当前的消费者数量
    container.setMaxConcurrentConsumers(5); //最大的消费者数量
    container.setDefaultRequeueRejected(false); //是否重回队列
    container.setAcknowledgeMode(AcknowledgeMode.AUTO); //是否自动回复签收
    container.setExposeListenerChannel(true);
    //消费端标签策略
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
            System.err.println("消费者：" + msg);
        }
    });
    return container;
}
```

## 5. MessageListenerAdapter 消息监听适配器

### 核心属性

- `defaultListenerMethod` 默认监听方法名称：用于设置监听方法名称

- `delegate` 委托对象：实际真实的委托对象，用于处理消息

- `queueOrTagToMethodName` 队列标识与方法名称组成的集合
  - 可以进行队列与方法名称一一的匹配
  - 队列和方法名称绑定，即指定队列里的消息会被绑定的方法所接收处理

```java
@Bean
public SimpleMessageListenerContainer messageListenerContainer(ConnectionFactory connectionFactory) {
    SimpleMessageListenerContainer container = new SimpleMessageListenerContainer(connectionFactory);
    container.setQueues(queue001(), queue002(), queue003(), queue_image(), queue_pdf());
    container.setConcurrentConsumers(1); //当前的消费者数量
    container.setMaxConcurrentConsumers(5); //最大的消费者数量
    container.setDefaultRequeueRejected(false); //是否重回队列
    container.setAcknowledgeMode(AcknowledgeMode.AUTO); //是否自动回复签收
    //消费端标签策略
    container.setConsumerTagStrategy(new ConsumerTagStrategy() {
        @Override
        public String createConsumerTag(String queue) {
            return queue + "_" + UUID.randomUUID().toString();
        }
    });

    /**
     * 1 适配器方式. 默认是有自己的方法名字的：handleMessage
     * 可以自己指定一个方法的名字: consumeMessage
     * 也可以添加一个转换器: 从字节数组转换为 String
     */
    //MessageListenerAdapter adapter = new MessageListenerAdapter(new MessageDelegate());
    //adapter.setDefaultListenerMethod("consumeMessage");
    //adapter.setMessageConverter(new TextMessageConverter());
    //container.setMessageListener(adapter);


    /**
     * 2 适配器方式: 我们的队列名称 和 方法名称 也可以进行一一的匹配
     */
    MessageListenerAdapter adapter = new MessageListenerAdapter(new MessageDelegate());
    adapter.setMessageConverter(new TextMessageConverter());
    Map<String, String> queueOrTagToMethodName = new HashMap<>();
    queueOrTagToMethodName.put("queue001", "method1");
    queueOrTagToMethodName.put("queue002", "method2");
    adapter.setQueueOrTagToMethodName(queueOrTagToMethodName);
    container.setMessageListener(adapter);

    return container;
}
```

委托对象 `MessageDelegate` ：

```java
public class MessageDelegate {

    public void handleMessage(byte[] messageBody) {
        System.err.println("默认方法, 消息内容:" + new String(messageBody));
    }

    public void consumeMessage(byte[] messageBody) {
        System.err.println("字节数组方法, 消息内容:" + new String(messageBody));
    }

    public void consumeMessage(String messageBody) {
        System.err.println("字符串方法, 消息内容:" + messageBody);
    }

    public void method1(String messageBody) {
        System.err.println("method1 收到消息内容:" + new String(messageBody));
    }

    public void method2(String messageBody) {
        System.err.println("method2 收到消息内容:" + new String(messageBody));
    }


    public void consumeMessage(Map messageBody) {
        System.err.println("map方法, 消息内容:" + messageBody);
    }


    public void consumeMessage(Order order) {
        System.err.println("order对象, 消息内容, id: " + order.getId() +
                ", name: " + order.getName() +
                ", content: " + order.getContent());
    }

    public void consumeMessage(Packaged pack) {
        System.err.println("package对象, 消息内容, id: " + pack.getId() +
                ", name: " + pack.getName() +
                ", content: " + pack.getDescription());
    }

    public void consumeMessage(File file) {
        System.err.println("文件对象 方法, 消息内容:" + file.getName());
    }
}
```

自定义转换器类 `TextMessageConverter` ：

```java
public class TextMessageConverter implements MessageConverter {

    /**
     * java对象 转换成 Message对象
     *
     * @param object
     * @param messageProperties
     * @return Message
     * @throws MessageConversionException
     */
    @Override
    public Message toMessage(Object object, MessageProperties messageProperties) throws MessageConversionException {
        return new Message(object.toString().getBytes(), messageProperties);
    }

    /**
     * Message对象 转换成 java对象
     *
     * @param message
     * @return Object
     * @throws MessageConversionException
     */
    @Override
    public Object fromMessage(Message message) throws MessageConversionException {
        String contentType = message.getMessageProperties().getContentType();
        if (null != contentType && contentType.contains("text")) {
            return new String(message.getBody());
        }
        return message.getBody();
    }

}
```

运行测试方法发送消息测试。

## 6. MessageConverter 消息转换器

我们在进行发哦是那个消息的时候，正常情况下消息体为二进制的数据方式进行传输，如果希望内部帮我们进行转换，或者指定自定义的转换器，就需要用到 MessageConverter 。

### 自定义常用转换器

一般都需要实现 `MessageConverter` 接口，重写下面 2 个方法：

**toMessage:** Java 对象转换为 Message

**fromMessage:** Message 对象转换为 Java 对象

### 转换器类型

**Json 转换器：** Jackson2JsonMessageConverter 可以进行 Java 对象的转换共、功能

**DefaultJacksion2JavaTypeMapper 映射器：** 可以进行 Java 对象的映射关系

**自定义二进制转换器：** 比如图片类型, PDF, PPT, 流媒体

### 转换 json 格式代码

```java
@Bean
public SimpleMessageListenerContainer messageListenerContainer(ConnectionFactory connectionFactory) {
    SimpleMessageListenerContainer container = new SimpleMessageListenerContainer(connectionFactory);
    container.setQueues(queue001(), queue002(), queue003(), queue_image(), queue_pdf());

    //省略...

    //3 支持json格式的转换器
    MessageListenerAdapter adapter = new MessageListenerAdapter(new MessageDelegate());
    adapter.setDefaultListenerMethod("consumeMessage");
    Jackson2JsonMessageConverter jackson2JsonMessageConverter = new Jackson2JsonMessageConverter();
    adapter.setMessageConverter(jackson2JsonMessageConverter);
    container.setMessageListener(adapter);

    return container;
}
```

运行测试代码：

```java
@Test
public void testSendJsonMessage() throws Exception {

    Order order = new Order();
    order.setId("001");
    order.setName("消息订单");
    order.setContent("描述信息");
    ObjectMapper objectMapper = new ObjectMapper();
    String msg = objectMapper.writeValueAsString(order);
    System.err.println("order 4 json: " + msg);

    //这里注意一定要修改 contentType 为 application/json
    Message message = MessageBuilder.withBody(msg.getBytes())
            .setContentType(MessageProperties.CONTENT_TYPE_JSON)
            .build();
    rabbitTemplate.send("topic001", "spring.order", message);
}
```

控制台输出：

```java
order 4 json: {"id":"001","name":"消息订单","content":"描述信息"}

map方法, 消息内容:{id=001, name=消息订单, content=描述信息}
```

也支持 Java 对象转换：

```java
@Test
public void testSendJavaMessage() throws Exception {

    Order order = new Order();
    order.setId("001");
    order.setName("消息订单");
    order.setContent("描述信息");
    ObjectMapper objectMapper = new ObjectMapper();
    String msg = objectMapper.writeValueAsString(order);
    System.err.println("order 4 json: " + msg);

    Message message = MessageBuilder.withBody(msg.getBytes())
            .setContentType(MessageProperties.CONTENT_TYPE_JSON)
            .setHeader(AbstractJavaTypeMapper.DEFAULT_CLASSID_FIELD_NAME, "com.mq.demo.entity.Order")
            .build();
    rabbitTemplate.send("topic001", "spring.order", message);
}
```

控制台输出：

```java
order 4 json: {"id":"001","name":"消息订单","content":"描述信息"}

order 对象, 消息内容, id: 001, name: 消息订单, content: 描述信息
```

### 支持 Java 对象多映射转换

```java
@Bean
public SimpleMessageListenerContainer messageListenerContainer(ConnectionFactory connectionFactory) {
    SimpleMessageListenerContainer container = new SimpleMessageListenerContainer(connectionFactory);
    container.setQueues(queue001(), queue002(), queue003(), queue_image(), queue_pdf());

    //省略...

    //5 DefaultJackson2JavaTypeMapper & Jackson2JsonMessageConverter 支持java对象多映射转换
    MessageListenerAdapter adapter = new MessageListenerAdapter(new MessageDelegate());
    adapter.setDefaultListenerMethod("consumeMessage");
    Jackson2JsonMessageConverter jackson2JsonMessageConverter = new Jackson2JsonMessageConverter();
    DefaultJackson2JavaTypeMapper javaTypeMapper = new DefaultJackson2JavaTypeMapper();

    Map<String, Class<?>> idClassMapping = new HashMap<String, Class<?>>();
    idClassMapping.put("order", com.mq.demo.entity.Order.class);
    idClassMapping.put("packaged", com.mq.demo.entity.Packaged.class);

    javaTypeMapper.setIdClassMapping(idClassMapping);

    jackson2JsonMessageConverter.setJavaTypeMapper(javaTypeMapper);
    adapter.setMessageConverter(jackson2JsonMessageConverter);
    container.setMessageListener(adapter);

    return container;
}
```

```java
@Test
public void testSendMappingMessage() throws Exception {

    ObjectMapper mapper = new ObjectMapper();

    Order order = new Order();
    order.setId("001");
    order.setName("订单消息");
    order.setContent("订单描述信息");

    String json1 = mapper.writeValueAsString(order);
    System.err.println("order 4 json: " + json1);

    MessageProperties messageProperties1 = new MessageProperties();
    //这里注意一定要修改contentType为 application/json
    messageProperties1.setContentType("application/json");
    messageProperties1.getHeaders().put("__TypeId__", "order");
    Message message1 = new Message(json1.getBytes(), messageProperties1);
    rabbitTemplate.send("topic001", "spring.order", message1);

    Packaged pack = new Packaged();
    pack.setId("002");
    pack.setName("包裹消息");
    pack.setDescription("包裹描述信息");

    String json2 = mapper.writeValueAsString(pack);
    System.err.println("pack 4 json: " + json2);

    MessageProperties messageProperties2 = new MessageProperties();
    //这里注意一定要修改contentType为 application/json
    messageProperties2.setContentType("application/json");
    messageProperties2.getHeaders().put("__TypeId__", "packaged");
    Message message2 = new Message(json2.getBytes(), messageProperties2);
    rabbitTemplate.send("topic001", "spring.pack", message2);
}
```

控制台输出：

```java
order 4 json: {"id":"001","name":"订单消息","content":"订单描述信息"}
pack 4 json: {"id":"002","name":"包裹消息","description":"包裹描述信息"}

order对象, 消息内容, id: 001, name: 订单消息, content: 订单描述信息
package对象, 消息内容, id: 002, name: 包裹消息, content: 包裹描述信息
```

### 自定义全局转换器

```java
@Bean
public SimpleMessageListenerContainer messageListenerContainer(ConnectionFactory connectionFactory) {
    SimpleMessageListenerContainer container = new SimpleMessageListenerContainer(connectionFactory);
    container.setQueues(queue001(), queue002(), queue003(), queue_image(), queue_pdf());

    //省略...

    //6 全局的转换器:
    MessageListenerAdapter adapter = new MessageListenerAdapter(new MessageDelegate());
    adapter.setDefaultListenerMethod("consumeMessage");
    ContentTypeDelegatingMessageConverter convert = new ContentTypeDelegatingMessageConverter();

    TextMessageConverter textConvert = new TextMessageConverter();
    convert.addDelegate("text", textConvert);
    convert.addDelegate("html/text", textConvert);
    convert.addDelegate("xml/text", textConvert);
    convert.addDelegate("text/plain", textConvert);

    Jackson2JsonMessageConverter jsonConvert = new Jackson2JsonMessageConverter();
    convert.addDelegate("json", jsonConvert);
    convert.addDelegate("application/json", jsonConvert);

    ImageMessageConverter imageConverter = new ImageMessageConverter();
    convert.addDelegate("image/png", imageConverter);
    convert.addDelegate("image", imageConverter);

    PDFMessageConverter pdfConverter = new PDFMessageConverter();
    convert.addDelegate("application/pdf", pdfConverter);

    adapter.setMessageConverter(convert);
    container.setMessageListener(adapter);
    return container;
}
```

```java
@Test
public void testSendExtConverterMessage() throws Exception {
    //byte[] body = Files.readAllBytes(Paths.get("d:/002_books", "picture.png"));
    //MessageProperties messageProperties = new MessageProperties();
    //messageProperties.setContentType("image/png");
    //messageProperties.getHeaders().put("extName", "png");
    //Message message = new Message(body, messageProperties);
    //rabbitTemplate.send("", "image_queue", message);

    byte[] body = Files.readAllBytes(Paths.get("d:/", "mysql.pdf"));
    MessageProperties messageProperties = new MessageProperties();
    messageProperties.setContentType("application/pdf");
    Message message = new Message(body, messageProperties);
    rabbitTemplate.send("", "pdf_queue", message);
}
```

控制台输出：

```java
-----------PDF MessageConverter----------
文件对象 方法, 消息内容:b4b06083-57f0-4d15-a61c-d921a2ecab05.pdf
```

`ImageMessageConverter` 转换类：

```java
public class ImageMessageConverter implements MessageConverter {

    @Override
    public Message toMessage(Object object, MessageProperties messageProperties) throws MessageConversionException {
        throw new MessageConversionException(" convert error ! ");
    }

    @Override
    public Object fromMessage(Message message) throws MessageConversionException {
        System.err.println("-----------Image MessageConverter----------");

        Object _extName = message.getMessageProperties().getHeaders().get("extName");
        String extName = _extName == null ? "png" : _extName.toString();

        byte[] body = message.getBody();
        String fileName = UUID.randomUUID().toString();
        String path = "d:/010_test/" + fileName + "." + extName;
        File f = new File(path);
        try {
            Files.copy(new ByteArrayInputStream(body), f.toPath());
        } catch (IOException e) {
            e.printStackTrace();
        }
        return f;
    }

}
```

`PDFMessageConverter` 转换类：

```java
public class PDFMessageConverter implements MessageConverter {

    @Override
    public Message toMessage(Object object, MessageProperties messageProperties) throws MessageConversionException {
        throw new MessageConversionException(" convert error ! ");
    }

    @Override
    public Object fromMessage(Message message) throws MessageConversionException {
        System.err.println("-----------PDF MessageConverter----------");

        byte[] body = message.getBody();
        String fileName = UUID.randomUUID().toString();
        String path = "d:/010_test/" + fileName + ".pdf";
        File f = new File(path);
        try {
            Files.copy(new ByteArrayInputStream(body), f.toPath());
        } catch (IOException e) {
            e.printStackTrace();
        }
        return f;
    }

}
```


































