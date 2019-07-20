# SpringBoot整合RabbitMQ（方式二）

SpringBoot版本：`2.1.6.RELEASE`

## 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
    <version>${parent.version}</version>
</dependency>

<!-- okhttp -->
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>okhttp</artifactId>
    <version>3.1.2</version>
</dependency>

<dependency>
    <groupId>com.squareup.retrofit2</groupId>
    <artifactId>retrofit</artifactId>
    <version>2.1.0</version>
</dependency>

<dependency>
    <groupId>com.squareup.retrofit2</groupId>
    <artifactId>converter-jackson</artifactId>
    <version>2.1.0</version>
</dependency>

<!--spring-mybatis-->
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.1.1</version>
</dependency>

<!--log start-->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.13</version>
</dependency>

<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
    <version>1.7.13</version>
</dependency>

<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
<!--log end-->
```

## application配置文件

```yml
server.port=9092
server.servlet.context-path=/mq

#logging
logging.path=E:\\JavaLogs\\SpringBoot-RabbitMQ\\logs
logging.file=springboot-rabbitmq-logs

spring.mvc.view.prefix=/WEB-INF/views/
spring.mvc.view.suffix=.jsp
multipart.max-request-size=20Mb
multipart.max-file-size=10Mb

logging.level.org.springframework = INFO
logging.level.com.fasterxml.jackson = INFO
logging.level.com.debug.steadyjack = DEBUG


spring.jackson.date-format=yyyy-MM-dd HH:mm:ss
spring.jackson.time-zone=GMT+8
spring.datasource.initialization-mode=always
spring.jmx.enabled=false

#数据源配置
datasource.url=jdbc:mysql://127.0.0.1:3306/mytest?useUnicode=true&amp;characterEncoding=utf-8&zeroDateTimeBehavior=convertToNull
datasource.username=root
datasource.password=root

#mybatis
mybatis.config-location=classpath:mybatis-config.xml
mybatis.checkConfigLocation = true
mybatis.mapper-locations=classpath:mappers/*.xml

#--------rabbitmq--------
spring.rabbitmq.virtual-host=/
spring.rabbitmq.host=127.0.0.1
spring.rabbitmq.port=5672
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest

#rabbitmq开启消息回调
#spring.rabbitmq.publisher-confirms=true
#spring.rabbitmq.publisher-returns=true

#rabbitmq开启消息确认
#spring.rabbitmq.listener.simple.acknowledge-mode=manual
#spring.rabbitmq.listener.direct.acknowledge-mode=manual

#消费者数量
spring.rabbitmq.listener.concurrency=10
#最大消费者数量
spring.rabbitmq.listener.max-concurrency=20
spring.rabbitmq.listener.prefetch=10

mq.env=local

#--------rabbitmq--------
```

## 配置类配置

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.amqp.core.AcknowledgeMode;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.config.SimpleRabbitListenerContainerFactory;
import org.springframework.amqp.rabbit.connection.CachingConnectionFactory;
import org.springframework.amqp.rabbit.connection.CorrelationData;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.amqp.support.converter.Jackson2JsonMessageConverter;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.amqp.SimpleRabbitListenerContainerFactoryConfigurer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.env.Environment;

@Configuration
public class RabbitmqConfig {

    private static final Logger log = LoggerFactory.getLogger(RabbitmqConfig.class);

    @Autowired
    private Environment env;

    @Autowired
    private CachingConnectionFactory connectionFactory;

    @Autowired
    private SimpleRabbitListenerContainerFactoryConfigurer factoryConfigurer;

    /**
     * 单一消费者
     *
     * @return
     */
    @Bean(name = "singleListenerContainer")
    public SimpleRabbitListenerContainerFactory listenerContainer() {
        SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
        factory.setConnectionFactory(connectionFactory);
        factory.setMessageConverter(new Jackson2JsonMessageConverter());
        factory.setConcurrentConsumers(1);
        factory.setMaxConcurrentConsumers(1);
        factory.setPrefetchCount(1);
        factory.setTxSize(1);
        factory.setAcknowledgeMode(AcknowledgeMode.AUTO);
        return factory;
    }

    /**
     * 多个消费者
     *
     * @return
     */
    @Bean(name = "multiListenerContainer")
    public SimpleRabbitListenerContainerFactory multiListenerContainer() {
        SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
        factoryConfigurer.configure(factory, connectionFactory);
        factory.setMessageConverter(new Jackson2JsonMessageConverter());
        factory.setAcknowledgeMode(AcknowledgeMode.NONE);
        factory.setConcurrentConsumers(env.getProperty("spring.rabbitmq.listener.concurrency", int.class));
        factory.setMaxConcurrentConsumers(env.getProperty("spring.rabbitmq.listener.max-concurrency", int.class));
        factory.setPrefetchCount(env.getProperty("spring.rabbitmq.listener.prefetch", int.class));
        return factory;
    }

    @Bean
    public RabbitTemplate rabbitTemplate() {
        connectionFactory.setPublisherConfirms(true);
        connectionFactory.setPublisherReturns(true);
        RabbitTemplate rabbitTemplate = new RabbitTemplate(connectionFactory);
        rabbitTemplate.setMandatory(true);
        rabbitTemplate.setConfirmCallback(new RabbitTemplate.ConfirmCallback() {
            @Override
            public void confirm(CorrelationData correlationData, boolean ack, String cause) {
                log.info("消息发送成功:correlationData({}),ack({}),cause({})", correlationData, ack, cause);
            }
        });
        rabbitTemplate.setReturnCallback(new RabbitTemplate.ReturnCallback() {
            @Override
            public void returnedMessage(Message message, int replyCode, String replyText, String exchange, String routingKey) {
                log.info("消息丢失:exchange({}),route({}),replyCode({}),replyText({}),message:{}", exchange, routingKey, replyCode, replyText, message);
            }
        });
        return rabbitTemplate;
    }

}
```

## 主启动类

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.boot.web.servlet.support.SpringBootServletInitializer;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ImportResource;

@SpringBootApplication
@MapperScan(basePackages = "com.amqp.mqdemo.mapper")
@ImportResource(locations = {"classpath:spring/spring-jdbc.xml"})
@EnableCaching
public class MqdemoApplication extends SpringBootServletInitializer {

    public static void main(String[] args) {
        SpringApplication.run(MqdemoApplication.class, args);
    }

    //工程为jar时，不使用spring boot内嵌tomcat启动方式
    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
        return builder.sources(MqdemoApplication.class);
    }

    //jackson 配置
    @Bean
    public ObjectMapper objectMapper(){
        ObjectMapper objectMapper=new ObjectMapper();
        return objectMapper;
    }

}
```

## log4j.properties

```yml
#Console Log
log4j.rootLogger=INFO,console,debug,info,warn,error

#LOG_PATH=d:/logs/erp
#LOG_FILE=erp

LOG_PATTERN=[%d{yyyy-MM-dd HH:mm:ss.SSS}] boot%X{context} - %5p [%t] --- %c{1}: %m%n

#A1--Print log to Console
log4j.appender.console=org.apache.log4j.ConsoleAppender
log4j.appender.console.Threshold=DEBUG
log4j.appender.console.layout=org.apache.log4j.PatternLayout
log4j.appender.console.layout.ConversionPattern=${LOG_PATTERN}

log4j.appender.info=org.apache.log4j.DailyRollingFileAppender
log4j.appender.info.Threshold=INFO
log4j.appender.info.File=${LOG_PATH}/${LOG_FILE}_info.log
log4j.appender.info.DatePattern='.'yyyy-MM-dd
log4j.appender.info.layout = org.apache.log4j.PatternLayout
log4j.appender.info.layout.ConversionPattern=${LOG_PATTERN}

log4j.appender.error=org.apache.log4j.DailyRollingFileAppender
log4j.appender.error.Threshold=ERROR
log4j.appender.error.File=${LOG_PATH}/${LOG_FILE}_error.log
log4j.appender.error.DatePattern='.'yyyy-MM-dd
log4j.appender.error.layout = org.apache.log4j.PatternLayout
log4j.appender.error.layout.ConversionPattern=${LOG_PATTERN}


log4j.appender.debug=org.apache.log4j.DailyRollingFileAppender
log4j.appender.debug.Threshold=DEBUG
log4j.appender.debug.File=${LOG_PATH}/${LOG_FILE}_debug.log
log4j.appender.debug.DatePattern='.'yyyy-MM-dd
log4j.appender.debug.layout = org.apache.log4j.PatternLayout
log4j.appender.debug.layout.ConversionPattern=${LOG_PATTERN}

log4j.appender.warn=org.apache.log4j.DailyRollingFileAppender
log4j.appender.warn.Threshold=WARN
log4j.appender.warn.File=${LOG_PATH}/${LOG_FILE}_warn.log
log4j.appender.warn.DatePattern='.'yyyy-MM-dd
log4j.appender.warn.layout = org.apache.log4j.PatternLayout
log4j.appender.warn.layout.ConversionPattern=${LOG_PATTERN}

#MQ发送记录
log4j.logger.mqLog=INFO,mqLog
log4j.additivity.mqLog = false
log4j.appender.mqLog=org.apache.log4j.DailyRollingFileAppender
log4j.appender.mqLog.File=${LOG_PATH}/${LOG_FILE}_mqLog.log
log4j.appender.mqLog.DatePattern='.'yyyy-MM-dd
log4j.appender.mqLog.layout = org.apache.log4j.PatternLayout
log4j.appender.mqLog.layout.ConversionPattern=${LOG_PATTERN}
```
