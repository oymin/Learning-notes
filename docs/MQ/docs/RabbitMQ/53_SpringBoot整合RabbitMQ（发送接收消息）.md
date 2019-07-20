# SpringBoot整合RabbitMQ（发送接收消息）

配置使用：`SpringBoot整合RabbitMQ（方式二）`中内容

## Queue、Exhcange配置

```java
//TODO:基本消息模型构造
@Bean("basicExchange")
public DirectExchange basicExchange() {
    return new DirectExchange(env.getProperty("basic.info.mq.exchange.name"), true, false);
}

@Bean(name = "basicQueue")
public Queue basicQueue() {
    return new Queue(env.getProperty("basic.info.mq.queue.name"), true);
}

@Bean
public Binding basicBinding() {
    return BindingBuilder.bind(basicQueue()).to(basicExchange()).with(env.getProperty("basic.info.mq.routing.key.name"));
}
```

## 消费端

```java
@Autowired
private ObjectMapper objectMapper;

@RabbitListener(queues = "${basic.info.mq.queue.name}", containerFactory = "singleListenerContainer")
public void consumeMessage(@Payload byte[] message) {
    try {
        //接收String
        //String msg = new String(message, "utf-8");
        //log.info("接收到消息：{}", msg);

        //接收对象
        //User user = objectMapper.readValue(message, User.class);
        //log.info("接收到消息：{}", user);

        //接收Map
        Map dataMap = objectMapper.readValue(message, Map.class);
        log.info("接收到消息：{}", dataMap);
    } catch (Exception e) {
        log.error("监听消费消息，发生异常：{}", e.fillInStackTrace());
    }
}
```

## 生产端

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import com.google.common.collect.Maps;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.core.MessageBuilder;
import org.springframework.amqp.core.MessageDeliveryMode;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.amqp.support.converter.Jackson2JsonMessageConverter;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.env.Environment;
import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.*;
import java.util.HashMap;

@RestController
public class RabbitController {

    private static final Logger log = LoggerFactory.getLogger(RabbitController.class);
    private static final String Prefix = "rabbit";

    @Autowired
    private Environment env;
    @Autowired
    private RabbitTemplate rabbitTemplate;
    @Autowired
    private ObjectMapper objectMapper;

    //发送String
    @GetMapping(value = Prefix + "/simple/message/send")
    public BaseResponse rabbitmq(@RequestParam(required = false) String message) {
        BaseResponse response = new BaseResponse(StatusCode.Success);
        try {
            log.info("待发送消息：{}", message);
            rabbitTemplate.setExchange(env.getProperty("basic.info.mq.exchange.name"));
            rabbitTemplate.setRoutingKey(env.getProperty("basic.info.mq.routing.key.name"));
            //第一种消息方式(字节流格式)
            //Message msg = MessageBuilder.withBody(message.getBytes("utf-8")).build();
            //rabbitTemplate.send(msg);

            //第二种消息方式（Json格式）
            rabbitTemplate.setMessageConverter(new Jackson2JsonMessageConverter());
            Message msg = MessageBuilder.withBody(objectMapper.writeValueAsBytes(message)).build();
            rabbitTemplate.convertAndSend(msg);

        } catch (Exception e) {
            log.error("发送简单消息发生异常：", e.fillInStackTrace());
        }

        return response;
    }

    //发送对象消息
    @RequestMapping(value = Prefix + "/object/message/send", method = RequestMethod.POST, consumes = MediaType.APPLICATION_JSON_UTF8_VALUE)
    public BaseResponse sendObjectMessage(@RequestBody User user) {
        BaseResponse response = new BaseResponse(StatusCode.Success);
        try {
            log.info("待发送对象消息：{}", user);
            rabbitTemplate.setExchange(env.getProperty("basic.info.mq.exchange.name"));
            rabbitTemplate.setRoutingKey(env.getProperty("basic.info.mq.routing.key.name"));
            rabbitTemplate.setMessageConverter(new Jackson2JsonMessageConverter());

            Message msg = MessageBuilder.withBody(objectMapper.writeValueAsBytes(user)).setDeliveryMode(MessageDeliveryMode.NON_PERSISTENT).build();

            rabbitTemplate.convertAndSend(msg);

        } catch (Exception e) {
            log.error("发送对象消息发生异常：", e.fillInStackTrace());
        }

        return response;
    }

    //发送多类型字段消息
    @RequestMapping(value = Prefix + "/multitype/message/send", method = RequestMethod.POST, consumes = MediaType.APPLICATION_JSON_UTF8_VALUE)
    public BaseResponse sendMultiTypeMessage() {
        BaseResponse response = new BaseResponse(StatusCode.Success);
        try {
            Integer id = 22;
            String name = "阿修罗";
            Long longId = 1200L;
            HashMap<String, Object> dataMap = Maps.newHashMap();
            dataMap.put("id",id);
            dataMap.put("name",name);
            dataMap.put("longId",longId);
            log.info("待发送多类型消息：{}", dataMap);
            rabbitTemplate.setExchange(env.getProperty("basic.info.mq.exchange.name"));
            rabbitTemplate.setRoutingKey(env.getProperty("basic.info.mq.routing.key.name"));
            rabbitTemplate.setMessageConverter(new Jackson2JsonMessageConverter());

            Message msg = MessageBuilder.withBody(objectMapper.writeValueAsBytes(dataMap)).setDeliveryMode(MessageDeliveryMode.NON_PERSISTENT).build();

            rabbitTemplate.convertAndSend(msg);

        } catch (Exception e) {
            log.error("发送对象消息发生异常：", e.fillInStackTrace());
        }

        return response;
    }

}
```
