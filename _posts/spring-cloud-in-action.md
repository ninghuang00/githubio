---
title: spring-cloud-in-action
categories:
  - 笔记
tags:
  - null
date: 2019-04-11 20:28:10
---
 这是摘要
 <!-- more -->

## 碰到的问题
1. UnknownHostException异常,服务消费者无法通过服务名消费服务
是因为spring-cloud和eureka以及ribbon的版本搭配不对,spring-cloud的Finchley版本应该使用spring-cloud-starter-netfilx-eureka-server,spring-cloud-starter-netfilx-eureka-client和spring-cloud-starter-netfilx-ribbon

## 版本对应关系
### spring-boot和spring-cloud的版本关系
`spring-boot-starter-parent`的版本和`spring-cloud-dependencies`的版本是对应的,不能随便搭配
> 参考地址:https://spring.io/projects/spring-cloud#release-trains


|Release Train|	Boot Version|
|----|----|
|Greenwich| 2.1.x|
|Finchley| 2.0.x|
|Edgware| 1.5.x|
|Dalston| 1.5.x|

### spring-cloud和eureka的版本关系
```
<dependencies>
	<!--注册中心依赖-->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
    </dependency>
    <!-- 服务提供者依赖 -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <!-- 负载均衡,服务消费者依赖 -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
    </dependency>
</dependencies>
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Finchley.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

## 启动java应用是配置内存限制
`nohup java -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=128m -Xms1024m -Xmx1024m -Xmn256m -Xss256k -XX:SurvivorRatio=8 -XX:+UseConcMarkSweepGC -jar /jar包路径 `
 -XX:MetaspaceSize=128m: 元空间初始大小
 -XX:MaxMetaspaceSize=128m: 元空间最大值
 -Xms1024m :堆初始化
 -Xmx1024m :堆最大值
 -Xmn256m :新生代大小
 -Xss256k :栈大小
 -XX:SurvivorRatio=8 :新生代老年代比例
 -XX:+UseConcMarkSweepGC :

## eureka-server配置
1. 依赖
```
<parent>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-parent</artifactId>
	<version>2.0.4.RELEASE</version> <!-- lookup parent from repository -->
</parent>

<!--注册中心依赖-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Finchley.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```
2. 配置文件`application.properties`
```
# eureka server 配置
server.port=1111
eureka.instance.hostname=localhost
# 注册中心不需要注册自己
eureka.client.register-with-eureka=false
# 注册中心不需要检索服务,而是维护服务实例
eureka.client.fetch-registry=false
eureka.client.serviceUrl.defaultZone=http://${eureka.instance.hostname}:${server.port}/eureka/
# 配置注册中心用户名和密码
# eureka.client.serviceUrl.defaultZone=http://<username>:<password>@${eureka.instance.hostname}:${server.port}/eureka/
```
3. 如果使用server高可用的话,可以再配置`application-peer1.properties`和`application-peer2.properties`文件
```
spring.application.name=eureka-server
# 本注册中心的配置
server.port=1111
eureka.instance.hostname=peer1
# 指向其他注册中心
eurekaServerPort=1112
eurekaServerHost=peer2
eureka.client.serviceUrl.defaultZone=http://${eurekaServerHost}:${eurekaServerPort}/eureka/
```
并且将`application.properties`中的`eureka.client.register-with-eureka`和`eureka.client.fetch-registry`注释掉
启动应用的时候使用`java -jar eureka-server.jar --spring.profiles.active=peer1`启动,可以使用`-Xms256m -Xmx512m`参数限制一下内存使用

## eureka-client配置
1. 依赖
```
<!-- 服务提供者依赖 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```
spring-boot和spring-cloud-dependencies依赖同server
2. 配置文件`application.properties`
```
# eureka client 配置
spring.application.name=hello-service
# 注册中心的端口
eurekaServerPort1=1111
eurekaServerPort2=1112
# 注册中心的主机名
eurekaServerHost1=peer1
eurekaServerHost2=peer2
eureka.client.serviceUrl.defaultZone=http://${eurekaServerHost1}:${eurekaServerPort1}/eureka/,http://${eurekaServerHost2}:${eurekaServerPort2}/eureka/
```
3. 如果配置多个服务,可以再配置`application-service1.properties`和`application-service2.properties`文件
```
# 指定服务启动端口号
server.port=8081
# eureka client 配置
spring.application.name=hello-service
eurekaServerPort=1111
eurekaServerHost=peer1
eureka.client.serviceUrl.defaultZone=http://${eurekaServerHost}:${eurekaServerPort}/eureka/
```
如果注册中心使用高可用的方式,将服务注册分别到不同的节点,节点之间会自动将服务进行同步.
4. 在一台机器上使用同一个服务启动多个实例
```
server.port=${random.int[10000,19999]}
eureka.instance.instanceId=${spring.application.name}:${random.int}
```


## 启动两个注册中心eureka-server和两个服务器提供者eureka-client之后的效果
1. peer1
{% asset_img eureka-server-peer1.png %}
2. peer2
{% asset_img eureka-server-peer2.png %}

## 服务消费者配置,这里使用的ribbon实现
1. 依赖
```
<!-- 服务提供者依赖 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<!-- 负载均衡,服务消费者依赖 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
</dependency>
```

2. 配置文件`application.properties`
```
# 服务消费者 配置
spring.application.name=ribbon-consumer
eurekaServerPort=1111
eurekaServerHost=peer1
eureka.client.serviceUrl.defaultZone=http://${eurekaServerHost}:${eurekaServerPort}/eureka/
```
其实和服务提供一样,就是注册到服务中心

## eureka服务治理机制
{% asset_img eureka服务治理机制.png %}

### 服务提供者
1. 服务注册
eureka-client在启动的时候会通过REST请求将自己注册到服务注册中心,另外还会带上一些资自身服务的元数据信息.服务的元数据信息存储在一个双层结构中,第一层的key是服务名,第二层的key是服务的实例名.需要注意的是需要在配置文件中配置`eureka.client.register-with-eureka=true`属性,如果设置为false则不会进行注册操作
2. 服务同步
示例图中的两个服务是分别注册到两个不同的服务注册中心的,而这两个注册中心相互注册为replicate服务,最终所有注册中心的服务将会同步.服务注册时的请求会被注册中心转发到注册中心集群中,从而实现注册中心线之间的服务同步,服务消费者可以通过任意一台注册中心获取到所有服务的注册信息列表.
3. 服务续租
服务提供者注册完服务之后,会维护一个心跳用来持续告知eureka-server自己还活着,不要把我剔除,其中有两个重要的属性配置:
```
# 指定调用服务续约时间
eureka.instance.lease-renewal-interval-in-seconds=30
# 指定服务失效时间
eureka.instance.lease-expiration-duration-in-seconds=90
```

## 服务消费者
1. 获取服务
eureka-server会维护一份只读的服务清单,服务消费者可以定时获取这份清单,以此来消费服务.可以通过以下属性配置服务消费者更新服务清单缓存的时间间隔:
```
eureka.client.registry-fetch-interval-seconds=30
```
2. 服务调用
根据服务清单可以获取服务的元数据信息,ribbon中默认采用轮询的方式请求服务.eureka中包含Region和Zone的概念,一个Region可以包含多个Zone,每个服务客户端会被注册到一个Zone中,所以每个客户端对应一个Region和一个Zone.在服务进行调用的时候会有限访问同一个Zone中的服务提供者,如果访问不到在寻找其他的Zone,可以配置的属性如下:
```
eureka.clientregion=
eureka.client.availability-zones=
```
3. 服务下线
关闭服务实例的时候,会触发一个服务下线的REST请求给eureka-server,注册中心接收到请求后会将该服务实例的状态置为DOWN,并将改服务下线的消息广播出去.

## 服务注册中心
1. 失效剔除
eureka-server启动的时候会创建一个定时任务,每隔60s的时间就会检查当前清单中超时(90s)没有续约服务实例,并从清单中剔除
2. 自我保护
eureka-server运行期间会统计15分钟内心跳失败的比例,如果低于85%,eureka-server会启动保护措施,将当前的实例注册信息保存起来,尽可能保护这些实例信息,让它们不会过期.但是如果在保护期间,部分实例真的出现了问题,那么服务消费者很容易请求到不可用的服务,出现调用失败的情况,这时候就需要客户端用容错机制,如请求重发,断路器等.可以通过一下属性关闭保护机制:`eureka.server.enable-self-preservation=false`

## eureka-server和eureka-client的配置类
更多详细的配置信息可以在`EurekaClientConfigBean.java`和`EurekaServerConfigBean.java`中查看

## eureka-instance配置类
可以查看`EurekaInstanceConfigBean.java`,
```
# spring-cloud-eureka中对同一主机启动多个相同的服务实例的默认命名规则
${spring.cloud.client.hostname}:${spring.application.name}:{spring.application.instance_id}:${server.port}

```

## 健康状态信息页面配置
P69
spring-cloud中默认使用了spring-boot-actuator模块提供的/info和/health端点进行状态和健康监控.
### 端点配置
1. 如果配置了上下文路径
```
management.context-path=/hello
eureka.instance.statusPageUrlPath=${management.context-path}/info
eureka.instance.healthCheckUrlPath=${management.context-path}/health
```

2. 如果修改了/info和/health的原始端点路径
```
endpoints.info.path=/appInfo
endpoints.health.path=/checkHealth

eureka.instance.statusPageUrlPath=${endpoints.info.path}
eureka.instance.healthCheckUrlPath=${endpoints.health.path}
```

3. 如果需要使用绝对路径,比如部署了Https
```
eureka.instance.statusPageUrl=https://${eureka.instance.hosthome}/info
eureka.instance.healthCheckUrl=https://${eureka.instance.hosthome}/health
eureka.instance.homePageUrl=https://${eureka.instance.hosthome}
```

### 健康监测
eureka注册中心判断服务实例健康状态的依据默认是使用客户端心跳来判断的,而不是使用spring-boot-actuator的/health,这样一般不能满足实际场景需求,修改如下:
1. 引入`spring-boot-starter-actuator`依赖
2. 配置文件中添加配置`eureka.client.healthcheck.enabled=true`
3. 正确配置`eureka.instance.healthCheckUrlPath`的访问路径,让注册中心可以正确访问到/health

## ribbon实现客户端负载均衡的原理
### RestTemplate介绍
1. get请求
主要是getForEntity()方法和getForObject()方法的使用
2. post请求
主要是postForEntity()方法和postForObject()方法的使用
3. put请求和delete请求
就是put()方法和delete()方法

### 负载均衡源码分析
1. `@LoadBalanced`注解
该注解标记RestTemplate之后,负载均衡的客户端就可以配置RestTemplate了,ribbon中的客户端是LoadBalancerClient接口.
2. LoadBalancerAutoConfiguration类
该类的使用有两个条件
```
@ConditionalOnClass(RestTemplate.class)
@ConditionalOnBean(LoadBalancerClient.class)
```
自动化配置主要完成三个任务
    1. 创建LoadBalancerInterceptor的Bean,拦截客户端发起的请求进行负载均衡
    2. 创建RestTemplateCustomizer的Bean,用来给RestTemplate增加拦截器LoadBalancerInterceptor
    3. 维护标记了@LoadBalanced注解的RestTemplate列表,并初始化,添加拦截器
3. 拦截器LoadBalancerInterceptor类
该类中首先注入了LoadBalancerClient的实现(RibbonLoadBalancerClient),然后在intercept()方法中调用client的execute()方法对拦截到的请求进行处理,execute()方法的第一步就是根据传入的serviceId获取服务实例,

### 负载均衡的策略
1. RandomRule
根据服务实例列表的大小,随机产生一个数字选取实例
2. RoundRobinRule
线性循环使用服务实例
3. RetryRule
可反复重试的策略,规定时间内如果一直获取不到可用的服务实例就返回null
4. WeightedResponseTimeRule
    1. 首先会创建一个定时任务来计算每个服务实例的权重,每个实例的权重范围根据服务实例的平均响应时间计算
    2. 生成随机数,在权重列表中中比较,选择服务实例
5. ClientConfigEnabledRoundRobinRule
实际实现和RoundRobinRule一样,一般不是直接使用,而是继承并扩展后定义高级策略
6. BestAvailableRule
选择最空闲的服务实例
7. PredicateBasedRule抽象策略
先根据子类中的Predicate逻辑对服务实例进行过滤,然后再进行轮询选取实例
8. AvailabilityFilteringRule
继承了PredicateRule,过滤条件如下:
    1. 是否故障,断路器是否生效已断开
    2. 实例的并发数是不是已超过阈值
9. ZoneAvoidanceRule
组合过滤
    1. 使用主过滤条件过滤
    2. 按照顺序使用次过滤条件过滤
    3. 每次过滤后判断以下两个条件:
        1. 过滤后的实例数量 >= 最小过滤后实例数量(默认1)
        2. 过滤后的实例比例 > 最小过滤后百分比(默认0)













