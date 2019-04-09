---
title: spring-boot-in-action.md
categories:
  - 笔记
tags:
  - springboot
  - 读书
date: 2019-04-08 13:08:33
---
 springboot实战学习笔记

## 总结
1. springboot带来的好处
起步依赖和自动配置

## 第二章 开发第一个应用程序
### 目录结构
```
readinglist
├── pom.xml
└── src
    ├── main
    │   ├── java
    │   │   └── com
    │   │       └── hn
    │   │           └── readinglist
    │   │               └── ReadinglistApplication.java //应用程序的启动引导类,也是主要的配置类
    │   └── resources
    │       ├── application.properties //配置应用程序和springboot的属性
    │       ├── static
    │       │   └── style.css
    │       └── templates
    │           └── register.html
    └── test
        └── java
            └── com
                └── hn
                    └── readinglist
                        └── ReadinglistApplicationTests.java //基本的集成测试类
```

### 启动引导类
```
@SpringBootApplication
public class ReadinglistApplication {
    public static void main(String[] args) {
        SpringApplication.run(ReadinglistApplication.class, args);
    }
}
```
其实注解`@SpringBootApplication`包含了三个重要的注解
1. @Configuration
表示该类使用的是基于Java的配置
2. @ComponentScan
启用自动组件扫描,让拥有Bean类型注解的类能够自动注册到Spring的Bean容器中
3. @EnableAutoConfiguration
启动springboot的自动配置

### 测试springboot应用程序
```
@RunWith(SpringRunner.class)
@SpringBootTest
public class ReadinglistApplicationTests {
    @Autowired
    ReaderRepository readerRepository;
    @Test
    public void contextLoads() {
        Reader reader = readerRepository.findByUsername("hn");
        System.out.println(reader.getFullname());
    }
}
```

### 使用H2数据库和JPA
1. 在pom文件中加入以下内容:
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>
```
2. 定义领域模型(比如说Book.java)
```
@Entity
@Data
public class Book {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    private String reader;
    private String title;
    private String author;
    private String description;

}
```

3. 定义仓库接口(比如BookRepository.java)
```
public interface BookRepository extends JpaRepository<Book, String> {
    Reader findByAuthorName(String username);
}
```

4. 然后就可以在Service中使用了

### 在spring boot中连接mysql数据库并使用JPA进行操作
1. 在pom文件中加入以下依赖:
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

2. 在配置文件application.properties中加入 
```
# mysql数据库相关信息
spring.datasource.url = jdbc:mysql://localhost:3306/webtest?useUnicode=true&characterEncoding=utf8&serverTimezone=GMT
spring.datasource.username = root
spring.datasource.password = 220316
spring.datasource.driverClassName = com.mysql.jdbc.Driver
# Specify the DBMS
spring.jpa.database = MYSQL
# Show or not log for each sql query
spring.jpa.show-sql = true
# Hibernate ddl auto (create, create-drop, update)
spring.jpa.hibernate.ddl-auto = update
# Naming strategy
spring.jpa.hibernate.naming-strategy = org.hibernate.cfg.ImprovedNamingStrategy
# stripped before adding them to the entity manager)
spring.jpa.properties.hibernate.dialect = org.hibernate.dialect.MySQL5Dialect

```
3. 其余步骤和使用H2数据库一样


### 自动配置原理
在引入的library中有一个叫做spring-boot-autoconfigure的jar文件,其中包含了非常多的xxxConfiguration类,比如`DataSourceAutoConfiguration.class`,`JsonHttpMessageConvertersConfiguration.class`等,这些配置类都在应用程序的Classpath中,都有可能会被使用到.可以使用`@Conditional`(可以注解类和方法)注解来条件化使用这些配置类.
```
public class JdbcTemplateCondition implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        try {
            context.getClassLoader().loadClass("org.springframework.jdbc.core.JdbcTemplate");
            return true;
        } catch (Exception e) {
            return false;
        }
    }
}
```
实现Condition接口中的matches()方法,用来设定使用条件,上面这个实现就是在Classpath中包含JdbcTemplate.class的时候才会返回以下这个Bean.
```
public class MyService {
	@Bean
    @Conditional(JdbcTemplateCondition.class)
    public Myservice getMyService() {
        ...
    }
}
```
比如在DataSourceAutoConfiguration配置类上就有注解`@ConditionalOnClass({DataSource.class, EmbeddedDatabaseType.class})`,只有在Classpath中包含这两个类的时候,DataSourceAutoConfiguration配置类提供的配置才会生效,不然会被忽略掉.

## 自定义配置
### 覆盖springboot的自动配置
1. 直接编写java配置类代码覆盖
通常是自动配置类需要较大的改动的时候使用,`@ConditionalOnMissingBean`注解是自定义配置类能够替代自动化配置类的关键,通常情况下,在springboot自动化配置的时候,由于有这个注解,框架会判断Classpath中有没有缺少注解指定的bean,如果缺少就是用自动化配置类,如果没哟缺少就使用自定义配置类.

2. 通过属性文件外置配置
如果只需要修改少量的配置,那么可以通过外置配置的方式进行微调.常用的方式有(优先级从高到低):
	1. 命令行参数
	使用命令行`java -jar readingList.jar --server.port 8000`,来制指定服务器监听的端口号
	2. java:comp/env中的JNDI属性
	3. JVM系统属性
	4. 操作系统环境变量
	比如使用`export spring_thymeleaf_cache=false`的环境变量来关闭模板缓存
	5. 随机生成的带`random.*`前缀的属性(在设置其他属性的时候,可以引用他们,比如${random.long})
	6. 应用程序之外的application.properties配置文件
	7. 打包在应用程序中的application.properties
	8. 通过@PropertySource标注的属性源
	9. 默认属性
其中application.properties文件可以放置的位置有四个(优先级从高到低):
	1. 外置,相对于应用程序运行目录的/config子目录里
	2. 外置,在应用程序运行的目录里
	3. 内置,在config包内
	4. 内置,在Classpath根目录
如果是要修改日志输出的配置,可以在Classpath的根目录(src/main/resources)里创建logback.xml配置文件,如果不想使用logback.xml作为日志配置文件名,可以使用以下配置`logging.config.classpath:logging-config.xml`
但是只是修改少量配置可以直接在application.properties文件中配置:
```
# 配置日志文件输出路径
logging.path = /var/logs/
# 配置日之外文件名称
logging.file = BookWorm.log 
# 根日志文件级别
logging.level.root = WARN
# spring security的日志级别
logging.level.root.org.springframework.security = DEBUG
```

### Bean的配置外置
比如从配置文件中获取属性值,可以使用`@ConfigurationProperties(prefix="amazon")`注解来注入属性值.
也可以在属性上使用`@Value("${amazon.associateId}")`
比如配置文件中配置了:
```
# bean属性配置
amazon.associateId = helloAmazon
```
以下代码中AmazonProperties类中的associateId属性就是从配置文件中获取.
```
@Component
@ConfigurationProperties(prefix = "amazon")
@Data
public class AmazonProperties {
    private String associateId;
}
```
在使用的时候首先要通过spring框架自动注入bean
```
@RunWith(SpringRunner.class)
@SpringBootTest
public class AmazonPropertiesTest {
    Logger logger = LoggerFactory.getLogger(AmazonPropertiesTest.class);
    @Autowired
    AmazonProperties amazonProperties;//注入bean的时候就会从配置文件中获取associateId属性
    @Test
    public void setAssociateId() {
        String associateId = amazonProperties.getAssociateId();
        logger.info("associate id is " + associateId);

    }
}
```

### 多环境配置文件,使用profile进行配置
Profile是一种条件化配置,基于运行时激活的Profile,会使用或者忽略不同的Bean或配置类.比如在生产环境下和开发环境下使用不同的日志配置:
`application.properties`文件 ,放置不区分环境的配置以及默认配置,并指定使还要用那个环境的配置.
```
# 指定使用的profile,也可以使用命令行指定
spring.profiles.active = development
logging.path = /var/logs/
logging.file = BookWorm.log 
logging.level.root = info
logging.level.root.org.springframework.security = DEBUG
```
`application-devlopment.properties`文件,放置开发环境下的配置.
```
logging.level.root = DEBUG
```
`application-production.properties`文件,放置生产环境下的配资.
```
logging.path = /var/logs/
logging.file = BookWorm.log 
logging.level.root = WARN
```

## 测试Web应用程序
### 使用Spring Mock MVC框架

### 直接跑tomcat或者Jetty容器

## 使用Groovy和Spring Boot CLI进行开发
### groovy语法特点
1. 不用import
2. 没有分号

### groovy工程特点
1. 没有spring配置文件(Bean怎么创建并组装?)
2. 没有构建文件(依赖库哪里来?)
3. 没有import文件(怎么解析具体使用的是哪个类)
4. 没有部署应用(Web服务器哪里来?)

## 在springboot中使用Grails
也是构建与spring框架之上的一种框架,和springboot一样,旨在简化开发工作

## 深入Actuator
可以用来监控springboot应用程序的运行状态

## 部署springboot应用程序

## 附录
### springboot开发者工具
### springboot起步依赖
### 配置属性
### springboot依赖











