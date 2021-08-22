# 单/多数据源、动态数据源使用场景

- 单数据源


- 多数据源


- 动态数据源


# 单数据源

使用SpringBoot时，配置单数据源很容易。Spring Boot会根据`application.yml`的spring.datasource的相关配置和`pom.xml`的数据库依赖自动为我们配置好一个DataSource。

其中，如果使用的是Springboot默认支持4种的数据源类型，会通过自动配置`org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration`生成一个DataSource Bean。
> **SpringBoot直接支持的的4种数据源连接池是**：
>- org.apache.tomcat.jdbc.pool.DataSource
>- com.zaxxer.hikari.HikariDataSource
>- org.apache.commons.dbcp.BasicDataSource
>- org.apache.commons.dbcp2.BasicDataSource
> 
> 另外，**SpringBoot在不同版本的默认数据源连接池也不同**：
>- SpringBoot 1.0时数据源连接池默认使用的是tomcat，2.0以后换成了Hikari
>

配置文件如下：

```
spring.datasource.url=jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf8
spring.datasource.driverClassName=com.mysql.jdbc.Driver
spring.datasource.username=root
spring.datasource.password=123456
# 指定数据源的类型

spring.datasource.type=com.zaxxer.hikari.HikariDataSource
```

如不想使用Springboot默认支持的4种数据源，还可以指定第三方的数据源，例如：Druid、c3p0等

```
# 指定第三方数据源连接池
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
```

无论是用官方默认数据源连接池还是指定第三方数据源连接池，上面的单数据源都是走的自动配置，不需要我们手动写一个 Config类。


# 多数据源

随着业务发展，我们会引入多个资源库，或者进行数据库拆分，这时就需要在SpringBoot中配置多个数据源。

**实现要点如下：**

1. 多数据源配置的时候，必须要有一个主数据源，使用 `@Primary` 这个注解来区分主数据源。
2. @MapperScan 扫描 Mapper 接口并注入到容器管理，要注意不同的数据源必须对应不同的包。
3. `@ConfigurationProperties(prefix = "xxx")` 这个注解可以自动扫描`.yml`、`.properties`中以`xxx`前缀开头的配置项，帮我们省去很多手动配置的麻烦。


## 实现Mybatis多数据源

下面以配置两个数据源为例子：

- 配置文件

```
spring.datasource.primary.url=jdbc:mysql://localhost:3306/test1
spring.datasource.primary.username=root
spring.datasource.primary.password=123456
spring.datasource.primary.driver-class-name=com.mysql.jdbc.Driver

spring.datasource.secondary.url=jdbc:mysql://localhost:3306/test2
spring.datasource.secondary.username=root
spring.datasource.secondary.password=123456
spring.datasource.secondary.driver-class-name=com.mysql.jdbc.Driver

```


- 主数据源的Java Config

```java
@Configuration
@MapperScan(basePackages = {"com.luckinJack.dao.primary"},        
            sqlSessionFactoryRef = "primarySqlSessionFactory")
public class PrimaryDataSourceConfiguration {

    @Bean(name = "primaryDatasource")
    @Primary
    @ConfigurationProperties(prefix = "spring.datasource.primary")
    public DataSource primaryDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean(name = "primaryTransactionManager")
    @Primary
    public DataSourceTransactionManager primaryTransactionManager() {
        return new DataSourceTransactionManager(primaryDataSource());
    }
    
    @Bean(name = "primarySqlSessionFactory")
    @Primary
    public SqlSessionFactory primarySqlSessionFactory(@Qualifier("primaryDatasource") DataSource primaryDataSource)
            throws Exception {
        final SqlSessionFactoryBean sessionFactory = new SqlSessionFactoryBean();
        sessionFactory.setDataSource(primaryDataSource);
        sessionFactory.setMapperLocations(new PathMatchingResourcePatternResolver()
                .getResources(MasterDataSourceConfig.MAPPER_LOCATION));
        return sessionFactory.getObject();
    }
    
}
```

- 从数据源的Java Config

```java
@Configuration
@MapperScan(basePackages = {"com.luckinJack.dao.secondary"},        
            sqlSessionFactoryRef = "secondarySqlSessionFactory")
public class PrimaryDataSourceConfiguration {

    @Bean(name = "secondaryDatasource")
    @ConfigurationProperties(prefix = "spring.datasource.secondary")
    public DataSource secondaryDataSource() {
        return DataSourceBuilder.create().build();
    }
    
    
    @Bean(name = "secondaryTransactionManager")
    public DataSourceTransactionManager clusterTransactionManager() {
        return new DataSourceTransactionManager(secondaryDataSource());
    }

    @Bean(name = "secondarySqlSessionFactory")
    public SqlSessionFactory secondarySqlSessionFactory(@Qualifier("secondaryDataSource") DataSource secondaryDataSource)
            throws Exception {
        final SqlSessionFactoryBean sessionFactory = new SqlSessionFactoryBean();
        sessionFactory.setDataSource(secondaryDataSource);
        sessionFactory.setMapperLocations(new PathMatchingResourcePatternResolver()
                .getResources(SecondDataSourceConfig.MAPPER_LOCATION));
        return sessionFactory.getObject();
    }
}
```

至此，不同的数据源配置就已经完成，剩下的只需要将Mybatis的xml文件和DAO层的接口写好，并在Service层注入，直接使用就行


## 实现SpringDataJPA多数据源

我们可以一个数据源指定一个配置类，每个配置类分别指定数据源对应的Entity实体和Repository接口的包地址。下面以配置两个数据源为例子：


- 配置文件

```
spring.datasource.primary.url=jdbc:mysql://localhost:3306/test1
spring.datasource.primary.username=root
spring.datasource.primary.password=123456
spring.datasource.primary.driver-class-name=com.mysql.jdbc.Driver

spring.datasource.secondary.url=jdbc:mysql://localhost:3306/test2
spring.datasource.secondary.username=root
spring.datasource.secondary.password=123456
spring.datasource.secondary.driver-class-name=com.mysql.jdbc.Driver

```

- 主数据源的JPA配置

```java
@Configuration
@EnableTransactionManagement
@EnableJpaRepositories(
        entityManagerFactoryRef="entityManagerFactoryPrimary",
        transactionManagerRef="transactionManagerPrimary",
        //配置Repository接口所在位置
        basePackages= { "com.luckinjack.repository.primary" }) 
public class PrimaryConfig {

    @Autowired @Qualifier("primaryDataSource")
    private DataSource primaryDataSource;

    @Primary
    @Bean(name = "entityManagerPrimary")
    public EntityManager entityManager(EntityManagerFactoryBuilder builder) {
        return entityManagerFactoryPrimary(builder).getObject().createEntityManager();
    }

    @Primary
    @Bean(name = "entityManagerFactoryPrimary")
    public LocalContainerEntityManagerFactoryBean entityManagerFactoryPrimary (EntityManagerFactoryBuilder builder) {
        return builder
                .dataSource(primaryDataSource)
                .properties(getVendorProperties(primaryDataSource))
                //配置Entity实体所在位置
                .packages("com.luckinjack.domain.primary") 
                .persistenceUnit("primaryPersistenceUnit")
                .build();
    }

    @Autowired
    private JpaProperties jpaProperties;

    private Map<String, String> getVendorProperties(DataSource dataSource) {
        return jpaProperties.getHibernateProperties(dataSource);
    }

    @Primary
    @Bean(name = "transactionManagerPrimary")
    public PlatformTransactionManager transactionManagerPrimary(EntityManagerFactoryBuilder builder) {
        return new JpaTransactionManager(entityManagerFactoryPrimary(builder).getObject());
    }

}

```

- 从数据源的JPA配置

```java
@Configuration
@EnableTransactionManagement
@EnableJpaRepositories(
        entityManagerFactoryRef="entityManagerFactorySecondary",
        transactionManagerRef="transactionManagerSecondary",
        //配置Repository接口所在位置
        basePackages= { "com.luckinjack.repository.secondary" }) 
public class SecondaryConfig {

    @Autowired @Qualifier("secondaryDataSource")
    private DataSource secondaryDataSource;

    @Bean(name = "entityManagerSecondary")
    public EntityManager entityManager(EntityManagerFactoryBuilder builder) {
        return entityManagerFactorySecondary(builder).getObject().createEntityManager();
    }

    @Bean(name = "entityManagerFactorySecondary")
    public LocalContainerEntityManagerFactoryBean entityManagerFactorySecondary (EntityManagerFactoryBuilder builder) {
        return builder
                .dataSource(secondaryDataSource)
                .properties(getVendorProperties(secondaryDataSource))
                //配置Entity实体所在位置
                .packages("com.luckinjack.domain.secondary") 
                .persistenceUnit("secondaryPersistenceUnit")
                .build();
    }

    @Autowired
    private JpaProperties jpaProperties;

    private Map<String, String> getVendorProperties(DataSource dataSource) {
        return jpaProperties.getHibernateProperties(dataSource);
    }

    @Bean(name = "transactionManagerSecondary")
    PlatformTransactionManager transactionManagerSecondary(EntityManagerFactoryBuilder builder) {
        return new JpaTransactionManager(entityManagerFactorySecondary(builder).getObject());
    }

}

```

分别新增主/次数据源的实体、主/次数据源的repository数据访问接口，分别放在先前配置的包地址下：

- 主数据源的User实体和Repository数据访问接口

```java
package com.luckinjack.domain.primary

@Data
@Entity
public class User {

    @Id
    @GeneratedValue
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private Integer age;
    
}
```
```java
package com.luckinjack.repository.primary

public interface UserRepository extends JpaRepository<User, Long> {

}
```

- 从数据源的Article实体和Repository数据访问接口：

```java
package com.luckinjack.domain.secondary

@Data
@Entity
public class Article {

    @Id
    @GeneratedValue
    private Long id;

    @Column(nullable = false)
    private String title;

    @Column(nullable = false)
    private String content;
    
}
```
```java
package com.luckinjack.repository.secondary

public interface ArticleRepository extends JpaRepository<Article, Long> {

}
```


接下来用测试用例验证SpringDataJPA多数据源是否配置成功。

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(Application.class)
public class ApplicationTests {

    @Autowired
    private UserRepository userRepository;
    @Autowired
    private ArticleRepository articleRepository;

    @Test
    public void test() throws Exception {

        userRepository.save(new User("aaa", 10));
        userRepository.save(new User("bbb", 20));
        userRepository.save(new User("ccc", 30));

        Assert.assertEquals(3, userRepository.findAll().size());

        articleRepository.save(new Article("t1", "123456"));
        articleRepository.save(new Article("t2", "654321"));
        articleRepository.save(new Article("t3", "135246"));

        Assert.assertEquals(3, articleRepository.findAll().size());
    }

}
```

# 动态数据源

我们为什么需要动态数据源呢


实现要点如下：

1. 通过扩展Spring的`AbstractRoutingDataSource`类，并重写其`determineCurrentLookupKey()`，在该方法中嵌入路由逻辑，来实现一个特定的动态数据源。
2. 配置好了动态数据源，接着就是如何在需要的时候动态切换数据源，这里我们可以使用Aspect切面 + 注解，拦截带 被注解修饰的方法来进行数据源动态切换。


## 实现Mybatis动态数据源


-  禁用数据源默认自动配置

```java
@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

数据源默认自动配置会读取 `spring.datasource.*` 的属性来创建数据源，所以要禁用以进行定制


- 配置文件

```
server:
  port: 8080
spring:
  datasource:
    primary:
      driver-class-name: com.mysql.cj.jdbc.Driver
      type: com.zaxxer.hikari.HikariDataSource
      jdbcUrl: jdbc:mysql://127.0.0.1:3306/primary?useUnicode=true&zeroDateTimeBehavior=convertToNull&autoReconnect=true&characterEncoding=utf-8
      username: root
      password: 123456
    secondary:
      driver-class-name: com.mysql.cj.jdbc.Driver
      type: com.zaxxer.hikari.HikariDataSource
      jdbcUrl: jdbc:mysql://127.0.0.1:3306/secondary?useUnicode=true&zeroDateTimeBehavior=convertToNull&autoReconnect=true&characterEncoding=utf-8
      username: root
      password: 123456
```

- Java Config

```java
@Configuration
@MapperScan(
        // 配置需要被扫描的DAO接口所在包地址
        value = "com.luckinJack.dao",
        sqlSessionFactoryRef = "commSessionFactory",
        sqlSessionTemplateRef = "commSessionTemplate")
public class DataSourceConfig {

    /**
     * 将动态数据源注入Spring IOC容器
     *
     * @return javax.sql.DataSource
     * @author LuckinJack
     * @date 2020/11/20 12:54
     */
    @Primary
    @Bean("dynamicDataSource")
    public DataSource dynamicDataSource(@Qualifier("primaryDataSource") DataSource primary,
                                        @Qualifier("secondaryDataSource") DataSource secondary) {
        Map<Object, Object> targetDataSources = new HashMap<>(3);
        targetDataSources.put(DynamicDataSource.DatabaseType.PRIMARY, primary);
        targetDataSources.put(DynamicDataSource.DatabaseType.SECONDARY, secondary);

        DynamicDataSource dataSource = new DynamicDataSource();
        dataSource.setTargetDataSources(targetDataSources);
        // 设置默认数据源
        dataSource.setDefaultTargetDataSource(primary);

        return dataSource;
    }

    /**
     * 创建primary数据源对象
     *
     * @return javax.sql.DataSource
     * @author LuckinJack
     * @date 2020/11/20 12:55
     */
    @Bean("primaryDataSource")
    @ConfigurationProperties(prefix = "spring.datasource.primary")
    public DataSource primary() {
        return new BasicDataSource();
    }

    /**
     * 创建secondary数据源对象
     *
     * @return javax.sql.DataSource
     * @author LuckinJack
     * @date 2020/11/20 12:55
     */
    @Bean("secondaryDataSource")
    @ConfigurationProperties(prefix = "spring.datasource.secondary")
    public DataSource secondary() {
        return new BasicDataSource();
    }

}
```

- 动态数据源实现类

```java
@Slf4j
public class DynamicDataSource extends AbstractRoutingDataSource {

    private static final ThreadLocal<Stack<DatabaseType>> DB_CONTEXT_HOLDER_STACK = ThreadLocal.withInitial(Stack::new);


    public enum DatabaseType {
        PRIMARY, SECONDARY
    }


    @Override
    protected Object determineCurrentLookupKey() {
        return getType();
    }

    /**
     * 将当前线程数据源类型设为PRIMARY
     */
    public static void supervision() {
        DB_CONTEXT_HOLDER_STACK.get().push(DatabaseType.PRIMARY);
    }

    /**
     * 将当前线程数据源类型设为SECONDARY
     */
    public static void workflow() {
        DB_CONTEXT_HOLDER_STACK.get().push(DatabaseType.SECONDARY);
    }

    /**
     * 设置当前线程数据源类型
     */
    public static void setDatabaseType(DatabaseType type) {
        DB_CONTEXT_HOLDER_STACK.get().push(type);
    }

    /**
     * 获取当前线程数据源类型
     */
    public static DatabaseType getType() {
        if (DB_CONTEXT_HOLDER_STACK.get().empty()) {
            return DatabaseType.PRIMARY;
        }
        DatabaseType current = DB_CONTEXT_HOLDER_STACK.get().peek();
        return current;
    }

    /**
     * 移除
     */
    public static void remove() {
        if (DB_CONTEXT_HOLDER_STACK.get() != null) {
            DB_CONTEXT_HOLDER_STACK.get().pop();
        }
    }

}
```

- 注解式数据源

创建一个注解 `@DataSource`，我们后面可以使用`@DataSource(value = '你想切换到的数据源') `，标记当前想使用的数据源。

```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface DataSource {
    
    /**
     * 数据源的值
     *
     * @author LuckinJack
     * @date 2020/11/20 12:57
     */
    DynamicDataSource.DatabaseType value() defalut DynamicDataSource.DatabaseType.PRIMARY;
    
}
```

- Spring AOP实现多数据源切换

拦截带 `@DataSource` 注解的方法，在方法执行前切换至目标数据源，执行完成后恢复到默认数据源。

```java
/**
 * 数据源切换切面
 *
 * @author LuckinJack
 * @date 2020/11/20
 */
@Aspect
@Component
@Order(Ordered.LOWEST_PRECEDENCE)
public class DataSourceAspect {

    @Pointcut("@annotation(com.luckinjack.annotation.DataSource)")
    public void switchDatasourcePointcut(JoinPoint point, DataSource dataSource) {
    }


    @Before("switchDatasourcePointcut()")
    public void setDataSource(JoinPoint point, DataSource dataSource) {
        DynamicDataSource.setDatabaseType(XXXX);
    }


    @After("switchDatasourcePointcut()")
    public void releaseSDataSource(JoinPoint point, DataSource dataSource) {
        DynamicDataSource.remove();
    }

}
```



# 参考

- [SpringBoot多数据源&动态数据源（主从）](https://www.cnblogs.com/nxzblogs/p/11849797.html)

- [Spring AOP实现多数据源切换](https://blog.csdn.net/qq_31142553/article/details/84661355)
