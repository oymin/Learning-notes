# Springboot整合RabbitMQ（方式一）

## 环境配置

配置使用 `3_RabbitMQ的初始环境搭建.md` 中的内容

## rabbitMQ配置类

```java
import org.springframework.amqp.core.*;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class RabbitmqConfig {

    //声明交换机
    @Bean("koms")
    public Exchange koms() {
        return ExchangeBuilder.topicExchange("koms").durable(true).build();
    }

    //声明队列
    @Bean("order")
    public Queue order() {
        return new Queue("order");
    }

    @Bean
    public Binding BINDING_QUEUE_ORDER(@Qualifier("order") Queue queue,
                                       @Qualifier("koms") Exchange exchange) {

        return BindingBuilder.bind(queue).to(exchange).with("order_key").noargs();
    }

}
```

## 生产者

```java
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

@SpringBootTest
@RunWith(SpringRunner.class)
public class PrpducerTest01 {
    @Autowired
    RabbitTemplate rabbitTemplate;

    //使用rabbitTemplate发送消息
    @Test
    public void testMq() {

        for (int i = 0; i < 10; i++) {
            /**
             * 参数1：交换机名称
             * 参数2：routingKey
             * 参数3：object消息内容
             */
            rabbitTemplate.convertAndSend("koms", "order_key", "msgResult:" + i);
        }

    }
}
```

## 消费者

```java
import com.rabbitmq.client.Channel;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

import java.io.IOException;

@Component
public class TestConsumer {

    @RabbitListener(queues = {"order"})
    public void test(String msg, Message message, Channel channel) throws IOException {
        long tag = message.getMessageProperties().getDeliveryTag();
        try {
            byte[] body = message.getBody();
            System.out.println("body is：" + new String(body, "utf-8"));
            System.out.println("message is: " + msg);

            channel.basicAck(tag, true);
        } catch (Exception e) {
            e.printStackTrace();
            //basicReject一次只能拒绝接收一个消息，
            channel.basicReject(tag, false);
            //basicNack方法可以支持一次0个或多个消息的拒收，并且也可以设置是否requeue。
            //channel.basicNack(tag, false, false);
        }
    }

}
```

---

## 生产者2

```java
//向MQ发送页面发布消息
private void sendPostPage(String pageId) {
    CmsPage cmsPage = this.getById(pageId);

    HashMap<String, String> msgMap = new HashMap<>();
    msgMap.put("pageId", pageId);
    //消息内容
    String msg = JSON.toJSONString(msgMap);
    //获取站点id作为routingKey
    String siteId = cmsPage.getSiteId();

    //发布消息
    rabbitTemplate.convertAndSend("postpage", siteId, msg);
}
```

## 消费者2

```java
/**
 * 监听MQ，接收页面发布消息
 */
@Component
public class ConsumerPostPage {

    private static final Logger LOGGER = LoggerFactory.getLogger(ConsumerPostPage.class);

    @Autowired
    CmsPageRepository cmsPageRepository;
    @Autowired
    PageService pageService;

    @RabbitListener(queues = {"${xuecheng.mq.queue}"})
    public void postPage(String msg) {
        //解析消息
        Map map = JSON.parseObject(msg, Map.class);

        //取出页面id
        String pageId = (String) map.get("pageId");
        //查询页面信息
        Optional<CmsPage> optional = cmsPageRepository.findById(pageId);
        if (!optional.isPresent()) {
            LOGGER.error("receive cms post page,cmsPage is null:{}", msg.toString());
            return;
        }
        //将页面保存到服务器物理路径
        pageService.savePageToServerPath(pageId);
    }

}
```
