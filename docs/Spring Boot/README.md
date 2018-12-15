## Spring Boot 目录文件结构

#### src/main/java

- 存放代码

#### src/main/resources

- static    : 存放静态文件
- templates : 存放静态页面
- config    : 存放配置文件
- resources

#### 同个文件的加载顺序，静态资源文件

Spring Boot 默认会挨个从 META/resources ? resources > static > public 里面找是否存在相应的资源，如果有则直接返回。

#### Spring Boot 默认配置加载的路径

```
spring.resources.static-locations = classpath:/META-INF/resources/,classpath:/resources/,classpath:/static/,classpath:/public/
```

#### 指定文件不进行热部署

```
spring.devtools.restart.exclude=static/**,public/**
```
