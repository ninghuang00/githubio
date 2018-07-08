---
title: spring32天笔记
categories:
  - 笔记
tags:
  - spring
date: 2018-07-08 16:52:42
---
 这是摘要
 <!-- more -->

spring 02
1 环境搭建，如何获得beans.xml的标签提示 
2 在xml中配置bean

spring 03 自己写的实例化bean容器
1 spring是如何创建bean 自己写一个

spring 04 bean实例化的另外两种方式
1 静态工厂方法
2 实例工厂方法

spring 05 bean的作用域
1 singleton
	单例模式，在每个spring ioc 容器中一个bean定义只有一个对象实例。（默认模式）
2 prototype
	每次从容器获取bean都是新对象
指定bean的初始化方法和销毁方法
	单例模式是在容器初始化的时候初始化（可以用lazy-init="true"）延迟
	prototype是在获取bean实例的时候实例化

spring 07 依赖注入
	1 通过构造器参数注入
	2 通过属性的set方法
	3 通过内部bean（bean无法被其他的调用）
	如何注入基本类型的属性

spring 09 注入集合类型的属性
	set
	list
	map
	properties


spring 10 通过构造器注入属性

spring 11 通过Field（字段）注入（用于注解方式）
1 手工装配依赖对象
	1 xml方式 
	2 使用注解进行装配
		@Autowired 默认按类型进行装配
		@Resource 默认按名称进行装配
2 自动装配依赖对象
	

spring 11 注解方式注入 自己实现注解
spring 12 @Resource 实现在字段和setter方法上的注解
spring 13 @Autowired 注解的使用 
		默认按类型进行装配 可以用@Qualified("personDaoxxx")用名称进行装配 
	自动装配
spring 14 通过在classpath自动扫描的方式吧组件纳入spring容器中管理
	@Componet 当组件不好归类的时候用
	@Service 业务层
	@Controller	控制层
	@Repository DAO层
	@Scope("prototype") 指定bean的作用域
	@PostConstruct 指定bean的初始化方法（使用广泛）
	@Predestroy bean销毁前关闭资源
spring 15 AOP技术
	代理对象（权限管理）Proxy

spring 16 CGlib的使用 当要代理的类没有继承接口的时候用
	AOP中基本概念

spring 17 spring框架的AOP编程
	1 基于xml配置的方式
	2 基于注解方式

	基于注解方式声明切面
		//（*）任意返回值 包名加类名.（*）所有方法（..代表有无参数皆可）
		@Pointcut("excetion (* cn.hn.service.impl.PersonServiceBean.*(..))")
spring 18 声明切面的各种注解

spring 19 使用xml配置切面

spring 20 表达式的介绍

spring 21 spring+JDBC组合开发
	beans.xml引入外部的配置 文件
		如果是在src目录下的话，该位置为：classpath:文件名.后缀
		如果是在/WEB-INF/目录下的话，该位置为： /WEB-INF/文件名.后缀。
spring 23 spring中的事务管理
	什么时候回滚
		Checked异常继承java.lang.Exception类。Unchecked异常继承自java.lang.RuntimeException类。
		遇到RuntimeException的时候回滚
		Exception不会回滚
	事务传播属性（spring实现）
	数据库的隔离级别（底层数据库实现）
spring 24 使用xml配置事务

spring 25 spring+Hibernate+Struts整合开发
spring 26 先完成spring+Hibernate的集成，能够操作数据库
spring 27 然后完成struts的集成，能够通过action访问数据库
spring 28 使用spring的依赖注入来获得PersonService
	首先将action交给spring管理,在beans.xml中配置bean时要注意name属性的命名要和struts-config.xml中的action-mapping中的action的path属性命名一致。
	然后在struts-config中配置一个请求控制器，来先根据action中的path属性值到spring容器中寻找bean
spring 29 Hibernate二级缓存
	hibernate5和之前的有点不同
	 <!-- 开启二级缓存 -->
				hibernate.cache.use_second_level_cache=true
				hibernate.cache.use_query_cache=false
				<!-- 设置二级缓存插件EHCache的Provider类 -->
				hibernate.cache.region.factory_class=org.hibernate.cache.ehcache.EhCacheRegionFactory

        		<!-- hibernate3的二级缓存配置   
       			hibernate.cache.provider_class=org.hibernate.cache.EhCacheProvider
       			-->
spring 30 解决struts中文乱码问题
			解决Hibernate因为session关闭导致的延迟加载异常
spring 31 spring2.5+JPA+struts
spring 32 配置struts

	





















