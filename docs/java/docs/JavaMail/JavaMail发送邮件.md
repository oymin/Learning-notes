# JavaMail发送邮件

## pom依赖

```xml
<dependency>
    <groupId>com.sun.mail</groupId>
    <artifactId>javax.mail</artifactId>
    <version>1.6.2</version>
</dependency>
```

## application文件配置

```yml
#邮件参数配置
spring.boot.mail.properties.host=smtp.qq.com
spring.boot.mail.properties.port=465
spring.boot.mail.properties.userName=302660844@qq.com
#授权码
spring.boot.mail.properties.password=
spring.boot.mail.properties.protocol=smtp
spring.boot.mail.properties.needAuth=true
spring.boot.mail.properties.sslClass=javax.net.ssl.SSLSocketFactory

mail.from=302660844@qq.com
mail.to=oymtofight@163.com,sasdfsf11@qq.com
#抄送者邮箱
mail.by=oymtofight@163.com
mail.subject=这是SpringBoot整合RabbitMQ邮件
mail.content=您好,这是RabbitMQ实战用于注册时异步发送邮件进行邮箱验证
```

## 参数配置文件

```java
@Configuration
@ConfigurationProperties(prefix = "spring.boot.mail.properties")
public class MailProperties {

    private String host;

    private Integer port;

    private String userName;

    private String password;

    private String protocol;

    private String needAuth;

    private String sslClass;

    public String getHost() {
        return host;
    }

    public void setHost(String host) {
        this.host = host;
    }

    .....
}
```

## controller代码

```java
import com.amqp.mqdemo.config.MailProperties;
import com.amqp.mqdemo.response.BaseResponse;
import com.amqp.mqdemo.response.StatusCode;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.env.Environment;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

import javax.mail.Message;
import javax.mail.Session;
import javax.mail.Transport;
import javax.mail.internet.MimeMessage;
import java.util.Properties;

/**
 * 发送邮件
 */
@RestController
public class MailController {
    private static final Logger log = LoggerFactory.getLogger(MailController.class);

    @Autowired
    private MailProperties mailProperties;
    @Autowired
    private Environment env;

    @RequestMapping(value = "mail/send", method = RequestMethod.GET)
    public BaseResponse sendMail() {
        try {
            Properties properties = new Properties();
            properties.setProperty("mail.host", mailProperties.getHost());
            properties.setProperty("mail.transport.protocol", mailProperties.getProtocol());
            properties.setProperty("mail.smtp.auth", mailProperties.getNeedAuth());
            properties.setProperty("mail.smtp.socketFactory.class", mailProperties.getSslClass());
            properties.setProperty("mail.smtp.port", mailProperties.getPort() + "");

            Session session = Session.getDefaultInstance(properties);
            session.setDebug(true);

            MimeMessage message = new MimeMessage(session);
            message.setFrom(env.getProperty("mail.from"));
            message.setSubject(env.getProperty("mail.subject"));
            message.setContent(env.getProperty("mail.content"), "text/html;charset=utf-8");
            //发送给多个邮箱
            String[] toArr = env.getProperty("mail.to").split(",");
            Address[] addresses = new Address[toArr.length];
            for (int i = 0; i < toArr.length; i++) {
                addresses[i] = new InternetAddress(toArr[i]);
            }
            message.addRecipients(Message.RecipientType.TO, addresses);
            //message.addRecipients(Message.RecipientType.TO, "oymtofight@163.com");
            message.addRecipients(Message.RecipientType.CC, "296117916@qq.com");//抄送人

            Transport transport = session.getTransport();
            transport.connect(mailProperties.getHost(), mailProperties.getUserName(), mailProperties.getPassword());
            transport.sendMessage(message, message.getAllRecipients());
            transport.close();
        } catch (Exception e) {
            e.printStackTrace();
            return new BaseResponse(StatusCode.Fail);
        }
        log.info("邮件发送完毕........");
        return new BaseResponse(StatusCode.Success);
    }

}
```

## 测试

访问方法请求路径测试
