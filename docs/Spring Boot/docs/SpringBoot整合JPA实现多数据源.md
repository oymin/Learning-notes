# SpringBoot整合JPA实现多数据源

## pom 依赖

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

## 配置 application.properties

```yml
spring:
  datasource
    #主数据源
    primary:
      jdbc-url: jdbc:mysql://127.0.0.1:3306/lc_seller?user=root&password=root&useUnicode=true&characterEncoding=utf-8
      driver-class-name: com.mysql.jdbc.Driver
    #从数据源
    backup:
      jdbc-url: jdbc:mysql://127.0.0.1:3306/seller-backup?user=root&password=root&useUnicode=true&characterEncoding=utf-8
      driver-class-name: com.mysql.jdbc.Driver
  jpa:
    show-sql: true
```

**注意：** SpringBoot2.x 默认使用的数据源是 HikariCP，它使用的连接参数地址是 `jdbc-url` 而不是 url

## 数据源配置类

```java
/**
 * JPA配置多个数据源
 */
@Configuration
public class DataSourceConfig {

    @Bean("primaryDataSource")
    @Primary //配置多个相同类型时，用@Primary 标注其中的一个，用于区分
    @ConfigurationProperties("spring.datasource.primary")
    public DataSource primaryDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean("backupDataSource")
    @ConfigurationProperties("spring.datasource.backup")
    public DataSource backupDataSource() {
        return DataSourceBuilder.create().build();
    }

}
```

必须有一个 `@Primary` 作为主数据源，生成的 `@Bean` 都将交给 Spring 容器进行管理，所以在同类型有多个注册的情况下需要在 `@Bean` 上定义别名，到时候注入的时候可以通过名称来进行注册，以免报错无法注入 bean。

## 配置数据源的JPA

```java
/**
 * JPA 主数据源配置
 */
@Configuration
@EnableTransactionManagement
@EnableJpaRepositories(
        entityManagerFactoryRef = "entityManagerFactoryMaster",//配置连接工厂 entityManagerFactory
        transactionManagerRef = "transactionManagerMaster", //配置事物管理器  transactionManager
        basePackages = {"com.koax.seller.repositores"} //该数据源的reposiotre所在的包
)
public class JpaMasterDataSourceConfig {
    @Autowired
    @Qualifier("primaryDataSource")
    private DataSource primaryDataSource;
    @Autowired
    private JpaProperties jpaProperties;

    @Bean("entityManagerMaster")
    @Primary
    public EntityManager entityManager(EntityManagerFactoryBuilder builder) {
        return entityManagerFactoryBean(builder).getObject().createEntityManager();
    }

    @Bean("entityManagerFactoryMaster")
    @Primary
    public LocalContainerEntityManagerFactoryBean entityManagerFactoryBean(EntityManagerFactoryBuilder builder) {
        return builder.dataSource(primaryDataSource)
                .properties(getVendorProperties())
                .packages("com.koax.entity") //实体类包名
                .persistenceUnit("masterPersistenceUnit") //持久化单元名称，当存在多个EntityManagerFactory时，需要制定此名称
                .build();
    }

    private Map getVendorProperties() {
        HibernateSettings hibernateSettings = new HibernateSettings();
        //  hibernateSettings.ddlAuto(ddlAuto);
        return jpaProperties.getHibernateProperties(hibernateSettings);
    }

    @Bean("transactionManagerMaster")
    @Primary
    public PlatformTransactionManager transactionManager(EntityManagerFactoryBuilder builder) {
        return new JpaTransactionManager(entityManagerFactoryBean(builder).getObject());
    }

}
```

---

```java
/**
 * JPA 副数据源配置
 */
@Configuration
@EnableTransactionManagement
@EnableJpaRepositories(
        entityManagerFactoryRef = "entityManagerFactorySlave",//配置连接工厂 entityManagerFactory
        transactionManagerRef = "transactionManagerSlave",//配置事物管理器  transactionManager
        basePackages = {"com.koax.seller.repositoresBackup"}
)
public class JpaBackupDataSourceConfig {
    @Autowired
    @Qualifier("backupDataSource")
    private DataSource backupDataSource;
    @Autowired
    private JpaProperties jpaProperties;

    @Bean("entityManagerFactorySlave")
    public LocalContainerEntityManagerFactoryBean localContainerEntityManagerFactoryBean(EntityManagerFactoryBuilder builder) {
        return builder.dataSource(backupDataSource)
                .properties(getVendorProperties())
                .packages("com.koax.entity") //实体类包名
                .build();
    }

    @Bean("entityManagerSlave")
    public EntityManager entityManager(EntityManagerFactoryBuilder builder) {
        return localContainerEntityManagerFactoryBean(builder).getObject().createEntityManager();
    }

    //JPAshi'事务
    @Bean("transactionManagerSlave")
    public PlatformTransactionManager transactionManager(EntityManagerFactoryBuilder builder) {
        return new JpaTransactionManager(localContainerEntityManagerFactoryBean(builder).getObject());
    }

    private Map getVendorProperties() {
        HibernateSettings hibernateSettings = new HibernateSettings();
        //hibernateSettings.ddlAuto(ddlAuto);
        return jpaProperties.getHibernateProperties(hibernateSettings);
    }
}
```

**注意：** 多数据源时，不同数据的源实现 `JpaRepository` 接口的 dao，放在不同的包，`basePackages` 扫描各自数据源下的包

@EnableJpaRepositories 注解的参数：

- `entityManagerFactoryRef`： 映射 `LocalContainerEntityManagerFactoryBean` 对应的 bean 名称，表示配置连接工厂。

- `transactionManagerMaster`： 也是一样，表示的是事务管理器。

- `basePackages`： 扫描的是继承 `JpaRepository<Student,Integer>` 等类所在的包

- `packages`： 扫描的是实体类所在的包。

JPA 自动配置关键的3个Bean:：

- dataSource
- entityManagerFactory
- transactionManager

---

SpringBoot JPA 自动配置的大概源码流程：

`spring-boot-autoconfigure` 自动配置依赖包

- `JpaRepositoriesAutoConfiguration.class` JPA自动配置类

  - `JpaRepositoriesAutoConfigureRegistrar.class`

    - `@EnableJpaRepositories`

  - `HibernateJpaAutoConfiguration.class` -> 父类 `JpaBaseConfiguration.class`

    - `transactionManager()` 注册到容器中

    - `entityManagerFactory()` 注册到容器中

---

## JPA 读写分离

主库包下的 `OrderRepository`

```java
/**
 * 主库: 写 repository
 */
public interface OrderRepository extends JpaRepository<Order, String>, JpaSpecificationExecutor<Order> {
}
```

从库包下的 `BackupOrderRepository`

```java
/**
 * 读库 repository
 */
public interface BackupOrderRepository extends OrderRepository {
}
```

测试：

```java
@Autowired
OrderRepository orderRepository;

@Autowired
@Qualifier("backupOrderRepository")
OrderRepository backupOrderRepository;

//测试不同的数据源查询
@Test
public void testDataSource() {
    List<Order> all = orderRepository.findAll();
    System.err.println(all.toString());

    System.err.println("--------------------------------------");

    List<Order> all1 = backupOrderRepository.findAll();
    System.err.println(all1.toString());
}
```

结果：

```yml
[orderId=1,chanId=1,chanUserId=1,orderType=1,productId=1,amount=1.00,outerOrderId=1]

-------------------------------------------------

[orderId=5,chanId=5,chanUserId=5,orderType=5,productId=5,amount=5.00,outerOrderId=5]
```
