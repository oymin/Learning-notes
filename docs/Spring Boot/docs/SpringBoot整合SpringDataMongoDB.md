# SpringBoot整合Spring Data MongoDB

## 添加 pom 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

## 添加数据源配置

application.yml

```yml
# 方式一
spring:
  data:
    mongodb:
      host: localhost
      port: 27017
      database: test
      username: admin
      password: admin

# 方式二
spring:
  application:
    name: xc‐service‐manage‐cms
  data:
    mongodb:
      uri: mongodb://root:root@localhost:27017
      database: xc_cms
```

## 创建 Dao

1、创建 Dao，继承 `MongoRepository`，并指定实体类型和主键类型

```java
public interface UserRepository extends MongoRepository<User,String> {
}
```

2、自定义 Dao 方法

同 Spring Data JPA 一样 Spring Data mongodb 也提供自定义方法的规则，如下：

按照 `findByXXX`、`findByXXXAndYYY`、`countByXXXAndYYY` 等规则定义方法，实现查询操作。

```java
public interface UserRepository extends MongoRepository<User,String> {
    //根据名称查询
    User findByName(String name);

    //根据名称和性别查询
    User findByNameAndSex(String name,String sex);

    //根据名称和性别查询记录数
    int countByNameAndSex(String name,String sex);

    //根据名称和性别查询分页查询
    Page<User> findBySiteIdAndPageType(String name,String sex, Pageable pageable);
}
```

## 编写测试类

```java
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.domain.*;
import org.springframework.test.context.junit4.SpringRunner;

import java.util.List;
import java.util.Optional;

@SpringBootTest
@RunWith(SpringRunner.class)
public class UserRepositoryTest {

    @Autowired
    UserRepository userRepository;

    @Test
    public void testFindAll() {
        List<User> all = userRepository.findAll();
        System.out.println(all);
    }

    //分页查询
    @Test
    public void testFindPage() {
        int page = 1;
        int size = 10;
        Pageable pageable = PageRequest.of(page, size);
        Page<User> all = userRepository.findAll(pageable);
        System.out.println(all);
    }

    //修改
    @Test
    public void testUpdate() {
        Optional<User> optional = userRepository.findById("11");
        if (optional.isPresent()) {
            User user = optional.get();
            user.setName("aa");
            User save = userRepository.save(user);
            System.out.println(save);
        }
    }

    //根据名称查询
    @Test
    public void testFindByName() {
        User user = userRepository.findByName("测试页面");
        System.out.println(user);
    }

    //自定义条件查询测试
    @Test
    public void testFindAllByExample() {
        int page = 0;
        int size = 10;
        Pageable pageable = PageRequest.of(page, size);
        User user = new User();
        user.setId("111");
        user.setAge(22);
        user.setName("王")
        ExampleMatcher exampleMatcher = ExampleMatcher.matchingAll();
        //模糊查询名称，包含关键字
        exampleMatcher = exampleMatcher.withMatcher("name", ExampleMatcher.GenericPropertyMatchers.contains());
        Example<User> example = Example.of(user, exampleMatcher);
        Page<User> all = userRepository.findAll(example, pageable);
        List<User> content = all.getContent();
        System.err.println(content);
    }
```
