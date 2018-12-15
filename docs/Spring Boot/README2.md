## Spring Boot 参考指南

---

## Spring Boot 文档

1. ##### 关于文档

2. ##### 获取帮助

3. ##### 第一步

4. ##### 使用 Spring Boot

5. ##### 了解 Spring Boot 功能

6. ##### 转向生产

7. ##### 高级主题

---

## 入门

1. ##### 介绍 Spring Boot

2. ##### 系统要求  
  2.1  Servle t容器

3. ##### 安装 Spring Boot  
  1. 关于 java 开发的安装说明

    - Maven 安装  
    - Gradle 安装

  2. 安装 Spring Boot CLI
    - 手动安装
    - 使用SDKMAN安装！
    - OSX Homebrew安装
    - MacPorts安装
    - 命令行完成
    - Windows Scoop安装
    - 快速启动Spring CLI示例

  3. 从早期版本的 Spring Boot 升级

4. ##### 开发您的第一个 Spring Boot 应用程序
  1. 创建 POM 文件

  2. 添加 Classpath 依赖项

  3. 编写代码
    - @RestController 和 @RequestMapping 注解
    - @EnableAutoConfiguration 注解
    - main 方法

  4. 运行示例

  5. 创建一个可执行的Jar

5.  ##### 接下来要阅读的内容

---

## 使用 Spring Boot

1. ##### 构建系统

  1. 依赖管理
  2. Maven
    - 继承Starter Parent
    - 在没有父POM的情况下使用Spring Boot
    - 使用Spring Boot Maven插件
  3. Gradle
  4. Ant
  5. Startes


2. ##### 构建您的代码

  - 使用“默认”包
  - 找到主应用程序类

3. ##### 配置类 Configuration Classes

  - 导入其他配置类
  - 导入 XML 配置

4. #####  自动配置 Auto-configuration
  - 逐步更换自动配置
  - 禁用特定的自动配置类

5. ##### Spring Beans 和依赖注入
  -  使用 @SpringBootApplication 注解

6. ##### 运行您的应用程序
  - IDE 中运行
  - 运行打包的应用程序运行
  - 使用 Maven 插件
  - 使用 Gradle Plugin
  - 热插拔

7. ##### 自动刷新（LiveReload）

8. ##### 全局设置

9. ##### 远程应用

10. ##### 远程应用
  - 运行远程客户端应用程序
  - 远程更新

11. ##### 打包您的生产应用程序

12. ##### 接下来要阅读的内容

---

## Spring Boot 功能

1. #### SpringApplication

  - 启动失败
  - 自定义Banner
  - 自定义SpringApplication
  - Fluent Builder API
  - 应用程序事件和监听器
  - 网络环境
  - 访问应用程序参数
  - 使用ApplicationRunner或CommandLineRunner
  - 申请退出
  - 管理功能

2. ##### 外部化配置 （Externalized Configuration）
  - 配置随机值
  - 访问命令行属性
  - 应用程序属性文件
  - 特定于配置文件的属性
  - 占位符属于
  - 加密属性
  - 使用YAML而不是属性
    1. 加载 YAML
    2. 在 Spring 环境中公开 YAML 作为属性
    3. 多页 YAML 文件
    4. YAML 缺点
  - 类型安全的配置属性
    1. 第三方配置
    2. 轻松绑定
    3. 合并复杂类型
    4. 属性转换
      - 转换持续时间
      - 转换数据大小
    5. @ConfigurationProperties 验证
    6. @ConfigurationProperties vs @Value

3. ##### 配置文件 Profiles
  - 添加活动配置文件
  - 以编程方式设置配置文件
  - 配置文件特定的配置文件

4. ##### 日志 Logging
  - 日志格式
  - 控制台输出
    1. 彩色编码输出
  - 文件输出
  - Log Levels
  - 日志组
  - 自定义日志配置
  - Logback 扩展
    1. 特定于配置文件的配置
    2. 环境属性

5. ##### JSON
  1. Jackson
  2. Gson
  3. JSON-B

6. ##### 开发 Web 应用程序
  - Spring Web MVC 框架
    1. Spring MVC Auto-configuration
    2. HttpMessageConverters
    3. 自定义 JSON 序列化器和反序列化器
    4. MessageCodesResolver
    5. 静态内容
    6. 欢迎页面
    7. 自定义 Favicon
    8. 路径匹配和内容协商
    9. ConfigurableWebBindingInitializer
    10. 模板引擎
    11. 错误处理
      - 自定义错误页面
      - 映射 Spring MVC 之外的错误页面
    12. Spring HATEOAS
    13. CORS支持
  - Spring WebFlux 框架
    1. Spring WebFlux自动配置
    2. 带有 HttpMessageReaders 和 HttpMessageWriters 的 HTTP 编解码器
    3. 静态内容
    4. 模板引擎
    5. 错误处理
      - 自定义错误页面
    6. 网络过滤器

7. ##### JAX-RS and Jersey

8. ##### 嵌入式Servlet容器支持
  - Servlet，过滤器和监听器  
    将Servlet，过滤器和监听器注册为Spring Beans

  - Servlet上下文初始化  
    扫描Servlet，过滤器和监听器

  - ServletWebServerApplicationContext

  - 自定义嵌入式Servlet容器
    Programmatic Customization
    直接自定义ConfigurableServletWebServerFactory

  - JSP 限制

  - 嵌入式Reactive Server支持

  - Reactive Server资源配置

9. ##### 安全（Security）
  - MVC Security
  - WebFlux Security
  - OAuth2
    1. Client  
      `OAuth2客户端注册常见提供`
    2. 资源服务器（Resource Server）
    3. 授权服务器（Authorization Server）
  - 执行器安全（Actuator Security）
    1. 跨站点请求伪造保护

30. Working with SQL Databases
