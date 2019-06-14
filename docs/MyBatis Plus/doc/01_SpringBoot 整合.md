# 初始化工程

可以使用 `Spring Initializr` 快速初始化一个 Spring Boot 工程

## 添加依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>

<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.1.0</version>
</dependency>

<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context-support</artifactId>
</dependency>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

## 配置 

在 `application.properties` 文件添加 MySQL 数据库的相关配置

```yml
#数据库连接信息
#spring.datasource.url=jdbc:oracle:thin:@//192.168.1.171:1521/orcl
spring.datasource.url=jdbc:mysql://localhost:3306/mybatis?allowMultiQueries=true&useUnicode=true&characterEncoding=UTF-8&useSSL=false&serverTimezone=GMT%2B8
spring.datasource.username=root
spring.datasource.password=root
#spring.datasource.driver-class-name=oracle.jdbc.driver.OracleDriver
spring.datasource.driver-class-name=com.mysql.jdbc.Driver

#指向mapper的xml文件位置
#mybatis.mapper-locations=classpath:mapper/*Mapper.xml
```

在 Spring Boot 启动类中添加 `@MapperScan` 注解，扫描 Mapper 文件夹：

```java
@SpringBootApplication
@MapperScan("com.baomidou.mybatisplus.samples.quickstart.mapper")
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(QuickStartApplication.class, args);
    }

}
```

## 编码

编写实体类 `User.java`（此处使用了 Lombok 简化代码）

```java
@Data
public class User {
    private Long id;
    private String name;
    private Integer age;
    private String email;
}
```

编写 Mapper 类 `UserMapper.java`

```java
public interface UserMapper extends BaseMapper<User> {

}
```

## 使用测试

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SampleTest {

    @Autowired
    private UserMapper userMapper;

    @Test
    public void testSelect() {
        System.out.println(("----- selectAll method test ------"));
        List<User> userList = userMapper.selectList(null);
        Assert.assertEquals(5, userList.size());
        userList.forEach(System.out::println);
    }

}
```
