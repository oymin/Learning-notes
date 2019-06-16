# SpringBoot整合MybatisPlus

## 添加 pom 依赖

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.1.1</version>
</dependency>

<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.1.10</version>
</dependency>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>5.1.46</version>
    <scope>runtime</scope>
</dependency>
```

----

## 配置

### `application.yml` 数据库相关配置

```yaml
spring:
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    druid:
      driverClassName: com.mysql.jdbc.Driver
      url: jdbc:mysql://127.0.0.1:3306/mytest?characterEncoding=UTF-8&serverTimezone=GMT%2B8&useSSL=false
      username: root
      password: root
      initial-size: 10
      max-active: 100
      min-idle: 10
      max-wait: 60000
      pool-prepared-statements: true
      max-pool-prepared-statement-per-connection-size: 20
      time-between-eviction-runs-millis: 60000
      min-evictable-idle-time-millis: 300000
      #validation-query: SELECT 1 FROM DUAL
      test-while-idle: true
      test-on-borrow: false
      test-on-return: false
      stat-view-servlet:
        enabled: true
        url-pattern: /druid/*
        #login-username: admin
        #login-password: admin
      filter:
        stat:
          log-slow-sql: true
          slow-sql-millis: 1000
          merge-sql: false
        wall:
          config:
            multi-statement-allow: true
```

### `application.yml` MybatisPlus 相关配置

```yaml
mybatis-plus:
  mapper-locations: classpath:mapper/**/*.xml
  #实体扫描，多个package用逗号或者分号分隔
  typeAliasesPackage: com.koax.modules.*.entity
  typeEnumsPackage: com.koax.common.enums
  global-config:
    #字段策略 0:"忽略判断",1:"非 NULL 判断"),2:"非空判断"
    field-strategy: 2
    #驼峰下划线转换
    db-column-underline: false
    #刷新mapper 调试神器
    refresh-mapper: true
    #数据库大写下划线转换
    #capital-mode: true
    #序列接口实现类配置
    #key-generator: com.baomidou.springboot.xxx
    #逻辑删除配置
    logic-delete-value: 1
    logic-not-delete-value: 0
    #自定义填充策略接口实现
    #meta-object-handler: com.baomidou.springboot.xxx
    #自定义SQL注入器
    #sql-injector: com.baomidou.mybatisplus.extension.injector.LogicSqlInjector
  configuration:
    map-underscore-to-camel-case: false
    cache-enabled: false
    call-setters-on-nulls: true
```

### 启动类配置 `@MapperScan` 注解

```java
@SpringBootApplication
@MapperScan("com.koax.dao")
public class App {
    public static void main(String[] args) {
        SpringApplication.run(App.class);
    }
}
```

----

## 代码

编写 `Mapper` 类 `UserMapper.java`

```java
public interface UserMapper extends BaseMapper<User> {

}
```

编写 `service` 接口 `UserService.java`

```java
public interface UserService extends IService<User> {

}
```

编写 `service` 接口实现类 `UserServiceImpl.java`

```java
@Service
public class UserSerivceImpl extends ServiceImpl<UserMapper,User> implements UserService {

}
```

----

## 测试使用

### 添加测试类，进行功能测试：

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

### 控制台输出

```java
User(id=1, name=Jone, age=18, email=test1@baomidou.com)
User(id=2, name=Jack, age=20, email=test2@baomidou.com)
User(id=3, name=Tom, age=28, email=test3@baomidou.com)
User(id=4, name=Sandy, age=21, email=test4@baomidou.com)
User(id=5, name=Billie, age=24, email=test5@baomidou.com)
```
