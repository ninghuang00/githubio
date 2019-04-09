---
title: springboot入门
categories:
  - 笔记
tags:
  - springboot 
  -
date: 2018-11-06 11:27:51
---
 这是摘要
 <!-- more -->



## 常用注解配置
### springboot
* 默认扫描位置
  * springboot默认扫描`@SpringBootApplication`入口类所在路径,同级目录包及其子包
  * 也可以使用注解`@ComponentScan(basePackages={"com.hn"})`指定扫描路径

* `@EnableScheduling`
在启动类配置该注解,之后可以使用`@Scheduled`注解定时任务

* 静态资源路径映射
```
# 默认值为 /**
spring.mvc.static-path-pattern=
# 默认值为 classpath:/META-INF/resources/,classpath:/resources/,classpath:/static/,classpath:/public/ 
spring.resources.static-locations=这里设置要指向的路径，多个使用英文逗号隔开，
```
默认配置源码在`org.springframework.boot.autoconfigure.web包下ResourceProperties类`中
```
private static final String[] CLASSPATH_RESOURCE_LOCATIONS = {
            "classpath:/META-INF/resources/", "classpath:/resources/",
            "classpath:/static/", "classpath:/public/" };
```
	1. 类路径文件 
	把类路径下的`/static、/public、/resources和/META-INF/resources文件夹下`的静态文件直接映射为`/**`来提供访问。 
	2. webjar 
	把webjar的`/META-INF/resources/webjars`下的静态文件映射为/webjar`/**`。 




## 配置优先级
Spring Boot 所提供的配置优先级顺序比较复杂。按照优先级从高到低的顺序:
1. 命令行参数。
2. 通过 System.getProperties() 获取的 Java 系统参数。
3. 操作系统环境变量。
4. 从 java:comp/env 得到的 JNDI 属性。
5. 通过 RandomValuePropertySource 生成的`1random.*1`属性。
6. 应用 Jar 文件之外的属性文件。
7. 应用 Jar 文件内部的属性文件。
8. 在应用配置 Java 类（包含@Configuration注解的 Java 类）中通过@PropertySource注解声明的属性文件。
9. 通过SpringApplication.setDefaultProperties声明的默认属性。
SpringBoot的这个配置优先级看似复杂，其实是很合理的。比如命令行参数的优先级被设置为最高。这样的好处是可以在测试或生产环境中快速地修改配置参数值，而不需要重新打包和部署应用。


> 参考地址https://www.javazhiyin.com/16381.html

### 配置mybatis
  1. 添加mybatis和mysql驱动依赖
  2. 配置文件中添加数据库参数


### Junit注解
* `@Before`,`@After`
会在每个有`@Test`注解的方法运行之前/之后运行一遍(有几个测试方法就运行几遍)

* `@BeforeClass`,`@AfterClass`
会在所有的测试方法运行前/后执行一遍(总共就一遍),使用该注解的方法应该是`public static void`并且无参

* `@Ignore`






