# SpringBoot 整合 Freemarker

## 添加pom依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-freemarker</artifactId>
</dependency>
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>okhttp</artifactId>
</dependency>
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-io</artifactId>
    <version>RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

## 配置文件

application.yml 和 logback-spring.xml

```yml
server:
  port: 8088 #服务端口

spring:
  application:
    name: test-freemarker
  freemarker:
    cache: false #关闭缓存，方便测试
    settings:
      template_update_delay: 0 #检查模板更新延迟时间，设置为0表示立即检查，如果时间大于0会有缓存不方便进行模板测试
```

## 创建模板.ftl文件

在 `src/main/resources` 下创建 templates文件夹，此目录为 freemarker 的默认模板存放目录。

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf‐8">
    <title>Hello World!</title>
</head>
<body>
Hello ${name}!
<br/>

遍历list<br>
<table>
    <tr>
        <td>序号</td>
        <td>姓名</td>
        <td>年龄</td>
        <td>生日</td>
    </tr>
<#if stus??>
    <#list stus as stu>
    <tr>
        <td>${stu_index + 1}</td>
        <td <#if stu.name=='小明'>style="background: coral" </#if>>${stu.name}</td>
        <#--<td <#if stu.age gte 19>style="background: coral" </#if>>${stu.age}</td>-->
        <td <#if (stu.age >= 19)>style="background: aqua" </#if>>${stu.age}</td>
        <#--<td>${stu.birthday?date}</td>-->
        <td>${stu.birthday?string('yyyy年MM月dd日')}</td>
    </tr>
    </#list>
</#if>
</table>
<br>

遍历数据模型中的map数据<br>
第一种方法：<br>
姓名：${stuMap['stu1'].name}<br>
年龄：${(stuMap['stu1'].age)!''}<br>
姓名：${(stuMap.stu1.name)!''}<br>
年龄：${stuMap.stu1.age}<br>

第二种方法：遍历map中的key<br>
<#list stuMap?keys as k>
姓名：${stuMap[k].name}<br>
年龄：${stuMap[k].age}<br>
</#list>
<br>

<#assign text="{'bank':'工商银行','account':'10101920201920212'}" />
<#assign data=text?eval />
开户行：${data.bank} 账号：${data.account}

</body>
</html>
```

## 静态化测试

在 test 下创建测试类，并且将 main 下的`resource/templates` 拷贝到 test 下，本次测试使用之前我们在 main 下创建的模板文件。

```java
@SpringBootTest
@RunWith(SpringRunner.class)
public class FreemarkerTest {

    //测试静态化，基于ftl模板生成html文件
    @Test
    public void testGenerateHtml() throws IOException, TemplateException {
        //创建配置类
        Configuration configuration = new Configuration(Configuration.getVersion());
        //设置模板路径
        String classpath = this.getClass().getResource("/").getPath();
        configuration.setDirectoryForTemplateLoading(new File(classpath + "/templates/"));
        //设置字符集
        configuration.setDefaultEncoding("utf-8");
        //加载模板
        Template template = configuration.getTemplate("test1.ftl");
        //数据模型
        Map<String, Object> map = getMap();
        //静态化
        String content = FreeMarkerTemplateUtils.processTemplateIntoString(template, map);
        //静态化内容
        System.out.println(content);
        InputStream inputStream = IOUtils.toInputStream(content);
        //输出文件
        FileOutputStream fileOutputStream = new FileOutputStream(new File("d:/test1.html"));
        int copy = IOUtils.copy(inputStream, fileOutputStream);
        inputStream.close();
        fileOutputStream.close();
    }

    public Map<String, Object> getMap() {
        HashMap<String, Object> map = new HashMap<>();
        //向数据模型放数据
        map.put("name", "大督导");
        Student stu1 = new Student();
        stu1.setName("小明");
        stu1.setAge(18);
        stu1.setBirthday(new Date());
        Student stu2 = new Student();
        stu2.setName("小红");
        stu2.setAge(19);
        stu2.setBirthday(new Date());
        List<Student> friends = new ArrayList<>();
        friends.add(stu1);
        stu2.setFriends(friends);
        stu2.setBestFriend(stu1);
        List<Student> stus = new ArrayList<>();
        stus.add(stu1);
        stus.add(stu2);
        //向数据模型放数据
        map.put("stus", stus);
        //准备map数据
        HashMap<String, Student> stuMap = new HashMap<>();
        stuMap.put("stu1", stu1);
        stuMap.put("stu2", stu2);
        //向数据模型放数据
        map.put("stu1", stu1);
        //向数据模型放数据
        map.put("stuMap", stuMap);
        //返回模板文件名称
        return map;
    }

}
```

## 基于模板文件的内容（字符串）生成html文件

```java
//基于模板文件的内容（字符串）生成html文件
    @Test
    public void testGenerateHtmlByString() throws IOException, TemplateException {
        //创建配置类
        Configuration configuration = new Configuration(Configuration.getVersion());
        //模板内容，这里测试的使用简单的字符串作为模板
        String templateString = "" +
                "<html>\n" +
                " <head></head>\n" +
                " <body>\n" +
                " 名称：${name}\n" +
                " </body>\n" +
                "</html>";
        //模板加载器
        StringTemplateLoader stringTemplateLoader = new StringTemplateLoader();
        stringTemplateLoader.putTemplate("template",templateString);
        //在中配置中设置模板加载器
        configuration.setTemplateLoader(stringTemplateLoader);
        //获取模板的内容
        Template template = configuration.getTemplate("template", "utf-8");
        //数据模型
        Map<String, Object> map = getMap();
        //静态化
        String content = FreeMarkerTemplateUtils.processTemplateIntoString(template, map);
        //静态化内容
        System.out.println(content);
        InputStream inputStream = IOUtils.toInputStream(content);
        //输出文件
        FileOutputStream fileOutputStream = new FileOutputStream(new File("d:/test1.html"));
        int copy = IOUtils.copy(inputStream, fileOutputStream);
        inputStream.close();
        fileOutputStream.close();
    }
```
