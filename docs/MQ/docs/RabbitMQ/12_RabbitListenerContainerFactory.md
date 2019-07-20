# RabbitListenerContainerFactory

这个 bean 只会在 consumer 端通过 @RabbitListener 注解的方式接收消息的时候使用。每个 @RabbitListener 注解方法都会由RabbitListenerContainerFactory 创建一个 MessageListenerContainer，负责接收消息。

```java
@Bean(name = "singleListenerContainer")
public SimpleRabbitListenerContainerFactory listenerContainerFactory() {
    SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
    factory.setConnectionFactory(connectionFactory()); //设置spring-amqp的ConnectionFactory
    factory.setMessageConverter(new Jackson2JsonMessageConverter()); //消息序列化类型
    factory.setConcurrentConsumers(1);       //设置每个MessageListenerContainer将会创建的Consumer的最小数量，默认是1个
    factory.setMaxConcurrentConsumers(1);
    factory.setPrefetchCount(1);             //设置每次请求发送给每个Consumer的消息数量。
    factory.setChannelTransacted(false);     //是否设置Channel的事务
    factory.setTxSize(1);                    //设置事务当中可以处理的消息数量
    factory.setDefaultRequeueRejected(true); //收到nack/reject确认信息时的处理方式，设为true，扔回queue头部，设为false，丢弃
    //factory.setErrorHandler();
    factory.setAcknowledgeMode(AcknowledgeMode.AUTO);
    return factory;
}
```