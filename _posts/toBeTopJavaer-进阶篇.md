---
title: toBeTopJavaer-进阶篇
categories:
  - 笔记
tags:
  - null
date: 2019-04-19 19:46:50
---
 这是摘要
 <!-- more -->

## 三 进阶篇

### Java底层知识

#### 字节码、class文件格式

#### CPU缓存，L1，L2，L3和伪共享

#### 缓存的算法实现
* 有哪些算法

>参考地址:https://www.cnblogs.com/dolphin0520/p/3749259.html

* FIFO
1. 概念
FIFO(First in First out)，先进先出。在操作系统的设计理念中很多地方都利用到了先进先出的思想，比如作业调度（先来先服务）,使用数据结构中的队列即可实现。
在FIFO Cache设计中，核心原则就是：如果一个数据最先进入缓存中，则应该最早淘汰掉。也就是说，当缓存满的时候，应当把最先进入缓存的数据给淘汰掉。
2. Cache中应该支持以下操作;
    * get(key)：如果Cache中存在该key，则返回对应的value值，否则，返回-1；
    * set(key,value)：如果Cache中存在该key，则重置value值；如果不存在该key，则将该key插入到到Cache中，若Cache已满，则淘汰最早进入Cache的数据。
3. 实现
使用map和队列

* LFU
1. 概念
LFU（Least Frequently Used）最近最少使用算法。它是基于“如果一个数据在最近一段时间内使用次数很少，那么在将来一段时间内被使用的可能性也很小”的思路。
LFU是基于访问次数的。
2. LFU Cache应该支持的操作为：
    * get(key)：如果Cache中存在该key，则返回对应的value值，否则，返回-1；
    * set(key,value)：如果Cache中存在该key，则重置value值；如果不存在该key，则将该key插入到到Cache中，若Cache已满，则淘汰最少访问的数据。
3. 实现

* LRU
1. 概念 
LRU全称是Least Recently Used，即最近最久未使用的意思。LRU算法的设计原则是：如果一个数据在最近一段时间没有被访问到，那么在将来它被访问的可能性也很小。也就是说，当限定的空间已存满数据时，应当把最久没有被访问到的数据淘汰。
2. 具备的操作：
    * set(key,value)：如果key在hashmap中存在，则先重置对应的value值，然后获取对应的节点cur，将cur节点从链表删除，并移动到链表的头部；若果key在hashmap不存在，则新建一个节点，并将节点放到链表的头部。当Cache存满的时候，将链表最后一个节点删除即可。
    * get(key)：如果key在hashmap中存在，则把对应的节点放到链表头部，并返回对应的value值；如果不存在，则返回-1。
3. 实现
使用双向链表和map实现

#### 尾递归

#### 位运算

用位运算实现加、减、乘、除、取余

---

### 设计模式
{% asset_link 设计模式.md 设计模式 %}

* 设计模式的六大原则：
开闭原则（Open Close Principle）、里氏代换原则（Liskov Substitution Principle）、依赖倒转原则（Dependence Inversion Principle）
接口隔离原则（Interface Segregation Principle）、迪米特法则（最少知道原则）（Demeter Principle）、合成复用原则（Composite Reuse Principle）

* 常用设计模式

#### 了解23种设计模式

创建型模式：单例模式、抽象工厂模式、建造者模式、工厂模式、原型模式。

结构型模式：适配器模式、桥接模式、装饰模式、组合模式、外观模式、享元模式、代理模式。

行为型模式：模版方法模式、命令模式、迭代器模式、观察者模式、中介者模式、备忘录模式、解释器模式（Interpreter模式）、状态模式、策略模式、职责链模式(责任链模式)、访问者模式。

#### 会使用常用设计模式

单例的七种写法：懒汉——线程不安全、懒汉——线程安全、饿汉、饿汉——变种、静态内部类、枚举、双重校验锁

工厂模式、适配器模式、策略模式、模板方法模式、观察者模式、外观模式、代理模式等必会

#### 不用synchronized和lock，实现线程安全的单例模式

#### 实现AOP

#### 实现IOC

#### nio和reactor设计模式

---

### 网络编程知识
{% asset_link 网络编程知识.md 网络编程知识 %}

#### tcp、udp、http、https等常用协议

三次握手与四次关闭、流量控制和拥塞控制、OSI七层模型、tcp粘包与拆包

#### http/1.0 http/1.1 http/2之间的区别

http中 get和post区别

常见的web请求返回的状态码

404、302、301、500分别代表什么

#### http/3

#### Java RMI，Socket，HttpClient

#### cookie 与 session

cookie被禁用，如何实现session

#### 用Java写一个简单的静态文件的HTTP服务器

#### 了解nginx和apache服务器的特性并搭建一个对应的服务器

#### 用Java实现FTP、SMTP协议

#### 进程间通讯的方式

#### 什么是CDN？如果实现？
1. cdn
内容分发网络(content delivery network)

>参考地址:https://www.jianshu.com/p/57433bc34659

#### DNS？

什么是DNS 、记录类型:A记录、CNAME记录、AAAA记录等

域名解析、根域名服务器

DNS污染、DNS劫持、公共DNS：114 DNS、Google DNS、OpenDNS

#### 反向代理

正向代理、反向代理

反向代理服务器

### 框架知识

#### Servlet
* 什么是servlet容器
{% asset_img servlet容器和webserver.png servlet容器和webserver %}
用户想web server请求的只能是静态页面,如果web server想要返回动态页面,就需要servlet来产生.所谓的servlet其实就是Java程序,servlet容器就是装这些java程序的,负责servlet程序的创建,执行和销毁,因此javax.servlet中的Servlet接口定义了是哪个方法:
    1. init()
    在Servlet生命周期初始化阶段调用,传递一个实现了javax.servlet.ServletConfig接口的对象以获取web application中的初始化参数
    2. service()
    接收一个请求都会调用一次Service()方法,每个请求的处理都在独立的线程中进行,Service()方法判断请求的类型并转发给相应的Servlet进行处理
    3. destory()
* webserver和Servlet容器处理一个请求的过程
    1. Web服务器接收到HTTP请求
    2. Web服务器将请求转发给servlet容器
    3. 如果容器中不存在所需的servlet，容器就会检索servlet，并将其加载到容器的地址空间中
    4. 容器调用servlet的init()方法对servlet进行初始化（该方法只会在servlet第一次被载入时调用）
    5. 容器调用servlet的service()方法来处理HTTP请求，即，读取请求中的数据，创建一个响应。servlet会被保留在容器的地址空间中，继续处理其他的HTTP请求
    6. Web服务器将动态生成的结果返回到正确的地址。

* 生命周期

* 线程安全问题

* filter和listener
1. filter 
CrossFilter
```java
public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletResponse response= (HttpServletResponse) servletResponse;
        String origin= servletRequest.getRemoteHost()+":"+servletRequest.getRemotePort();
        response.setHeader("Access-Control-Allow-Origin", "*");
        response.setHeader("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept");
        response.setHeader("Access-Control-Allow-Methods","POST,GET,OPTIONS,DELETE");
       /* response.setHeader("Access-Control-Allow-Methods","POST,GET,OPTIONS,DELETE");
        response.setHeader("Access-Control-Max-Age","3600");
        response.setHeader("Access-Control-Allow-Credentials","true");*/
        filterChain.doFilter(servletRequest,servletResponse);
    }
```
HtmlFilter(防止XSS攻击)

>参考地址:http://blog.csdn.net/qq924862077/article/details/62053577

    ```java 
    import javax.servlet.*;
    import javax.servlet.http.HttpServletResponse;
    @WebFilter("/HtmlFilter")
    public class HtmlFilter implements Filter {
        public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain) throws IOException, ServletException {
            HttpServletRequest request = (HttpServletRequest) req;
            HttpServletResponse response = (HttpServletResponse) resp;
            MyHtmlRequest htmlRequest = new MyHtmlRequest(request);
            chain.doFilter(htmlRequest, response);
        }
        class MyHtmlRequest extends HttpServletRequestWrapper{
            public MyHtmlRequest(HttpServletRequest request) {
                super(request);
            }
            @Override
            public String getParameter(String name) {
                
                String value = super.getParameter(name);
                if(value == null) return null;
                return filter(value);
            }
            public  String filter(String message) {
                if (message == null)
                    return (null);
                char content[] = new char[message.length()];
                message.getChars(0, message.length(), content, 0);
                StringBuilder result = new StringBuilder(content.length + 50);
                for (int i = 0; i < content.length; i++) {
                    switch (content[i]) {
                    case '<':
                        result.append("&lt;");
                        break;
                    case '>':
                        result.append("&gt;");
                        break;
                    case '&':
                        result.append("&amp;");
                        break;
                    case '"':
                        result.append("&quot;");
                        break;
                    default:
                        result.append(content[i]);
                    }
                }
                return (result.toString());
            }
        }
    }
    ```
CharactorFilter

    ```java
    import javax.servlet.*;
    import javax.servlet.http.HttpServletResponse;
    /**
     * Servlet Filter implementation class CharacterEncodingFilter
     */
    @WebFilter("/CharacterEncodingFilter")
    public class CharacterEncodingFilter implements Filter {
        private String charset ;
        private String defaultCharset = "UTF-8";
        public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain) throws IOException, ServletException {
            HttpServletRequest request = (HttpServletRequest) req;
            HttpServletResponse response = (HttpServletResponse) resp;
            request.setCharacterEncoding(charset);
            response.setCharacterEncoding(charset);
            response.setContentType("text/html;charset=" + charset);
            request.setAttribute("charset", charset);   
            chain.doFilter(new MyRequest(request), response);
        }
        public void init(FilterConfig fConfig) throws ServletException {
            String charset = fConfig.getInitParameter("charset");
            if(charset == null){
                charset = defaultCharset;
            }
            this.charset = charset;
        }
    }
    class MyRequest extends HttpServletRequestWrapper{
        private HttpServletRequest request;
        public MyRequest(HttpServletRequest request) {
            super(request);
            this.request = request;
        }
        @Override
        public String getParameter(String name) {
            String value = request.getParameter(name);
            if(value == null) return null;
            if(!request.getMethod().equals("get")){
                return value;
            }
            try {
                value = new String(value.getBytes("ios8859-1"),request.getParameter("charset"));
            } catch (UnsupportedEncodingException e) {
                throw new RuntimeException(e);
            }
            return value;
        }
    }
    ```

2. listener

* web.xml中常用配置及作用

#### Hibernate

什么是OR Mapping

Hibernate的缓存机制

Hibernate的懒加载

Hibernate/Ibatis/MyBatis之间的区别

#### Spring 

Bean的初始化

AOP原理

实现Spring的IOC

spring四种依赖注入方式

#### Spring MVC
{% asset_link SpringMVC.md Spring-MVC %}

* 什么是MVC

* Spring mvc与Struts mvc的区别

#### Spring Boot
{% asset_link spring-boot-in-action.md Spring-boot %}

* Spring Boot 2.0、起步依赖、自动配置、

* Spring Boot的starter原理，自己实现一个starter

#### Spring Security

### Spring Cloud

服务发现与注册：Eureka、Zookeeper、Consul

负载均衡：Feign、Spring Cloud Loadbalance

服务配置：Spring Cloud Config

服务限流与熔断：Hystrix

服务链路追踪：Dapper

服务网关、安全、消息

### 应用服务器知识

#### JBoss

#### tomcat
* 是不是线程安全的:
单例模式,当客户端第一次请求该servlet的时候会根据web.xml实例化servlet,之后客户端再访问该servlet使用的就是同一实例,是不是线程安全看servlet的实现,如果servlet的内部属性会被多线程改变,那就是不安全的.

#### jetty

#### Weblogic

### 工具

#### git & svn

#### maven & gradle

#### Intellij IDEA

常用插件：Maven Helper 、FindBugs-IDEA、阿里巴巴代码规约检测、GsonFormat

* Lombok plugin
常用注解
```
* @Builder:
* @EqualsAndHashCode：实现equals()方法和hashCode()方法 
* @ToString：实现toString()方法 
* @Data ：注解在类上；提供类所有属性的 getting 和 setting 方法，此外还提供了equals、canEqual、hashCode、toString 方法 
* @Setter：注解在属性上；为属性提供 setting 方法 
* @Getter：注解在属性上；为属性提供 getting 方法 
* @Slf4j: 注解在类上,提供一个log对象实例
* @Log4j ：注解在类上；为类提供一个 属性名为log 的 log4j 日志对象 
* @NoArgsConstructor：注解在类上；为类提供一个无参的构造方法 
* @AllArgsConstructor：注解在类上；为类提供一个全参的构造方法 
* @Cleanup：关闭流
* @Synchronized：对象同步
* @SneakyThrows：抛出异常
```
* Swagger2
常用注解
```
* @API
表示一个开放的API，可以通过description简明的描述API的功能，produce指定API的mime类型
* @ApiOperation
一个@API下，可以有多个@ApiOperation,表示针对该API的CRUD操作，在ApiOperation Annotation中还可以通过value，notes描述该操作的作用，response描述正常情况下该请求返回的对象类型
在一个@ApiOperation下，可以通过@ApiResponses描述API操作可能出现的异常情况
* @ApiParam
用于描述该API操作接收的参数类型，value用于描述参数，required指明参数是否为必须。
* @ApiModelProperty
value指定描述，required指明是否为必须
* web访问地址
`localhost:8080/swagger-ui.html`
```

.ignore

Mybatis plugin