# yml 配置数据简介

YML文件格式是YAML (YAML Aint Markup Language)编写的文件格式，YAML是一种直观的能够被电脑识别的的数据数据序列化格式，并且容易被人类阅读，容易和脚本语言交互的，可以被支持YAML库的不同的编程语言程序导入，比如： C/C++, Ruby, Python, Java, Perl, C#, PHP等。YML文件是以数据为核心的，比传统的xml方式更加简洁。

## 配置数据

### 1. 普通数据

语法：key : value

```yaml
name: tom
```

### 2. 对象数据

```yaml
person:
    name: haohao
    age: 31
    addr: beijing

#或者

person: {name: haohao,age: 31,addr: beijing}
```

### 3. 配置Map数据

```yaml
map:
  key1: 123
  key2: abc
```

### 4.  配置数组（List、Set）数据

```yaml
city:
  - beijing
  - tianjin
  - shanghai
  - chongqing

#或者
city: [beijing,tianjin,shanghai,chongqing]

#集合中的元素是对象形式
student:
  - name: zhangsan
    age: 18
    score: 100
  - name: lisi
    age: 28
    score: 88
  - name: wangwu
    age: 38
    score: 90
```
----

## 配置文件与配置类的属性映射方式

### 1. @Value 注解映射

通过 `@Value` 注解将配置文件中的值映射到一个 Spring 管理的 Bean 的字段上

实体 Bean 中代码获取 person.name：

```java
@Controller
public class QuickStartController {
    @Value("${person.name}")
    private String name;
    @Value("${person.age}")
    private Integer age;

    @RequestMapping("/quick")
    @ResponseBody
    public String quick() {
        return "springboot 访问成功! name=" + name + ",age=" + age;
    }
}
```

### 2. @ConfigurationProperties 注解映射

通过注解 `@ConfigurationProperties` (`prefix`="配置文件中的 key 的前缀")可以将配置文件中的配置自动与实体进行映
射

实体 Bean 中代码获取 person.name：

```java
@Controller
@ConfigurationProperties(prefix = "person")
public class QuickStartController {
    private String name;
    private Integer age;

    @RequestMapping("/quick")
    @ResponseBody
    public String quick() {
        return "springboot 访问成功! name=" + name + ",age=" + age;
    }

    public void setName(String name) {
        this.name = name;
    }

    public void setAge(Integer age) {
        this.age = age;
    }
}
```

**注意：** 使用 `@ConfigurationProperties` 方式可以进行配置文件与实体字段的自动映射，但需要字段必须提供 `set` 方法才可以，而使用 @Value 注解修饰的字段不需要提供 `set` 方法
