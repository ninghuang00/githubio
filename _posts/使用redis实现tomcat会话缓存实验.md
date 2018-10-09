---
title: 使用redis实现tomcat会话缓存实验
categories:
  - 笔记
tags:
  - redis
  - session
  - tomcat
date: 2018-09-19 15:20:28
---
 这是摘要
 <!-- more -->


### nginx配置
1. 修改nginx.conf实现负载均衡

### 修改tomcat配置
1. 修改server.xml设置默认虚拟主机
```xml
<Engine name="Catalina" defaultHost="localhost" jvmRoute="tomcat-1">
	...

	<Host name="localhost"  appBase="webapps"
		unpackWARs="true" autoDeploy="true">
		<Context docBase="/web/webapp1" path="" reloadable="true"/>
		...

	</Host>
	...

</Engine>
```

2. /web/webapp1/中添加index.jsp文件,方便查看效果

```java
<%@page language="java" import="java.util.*" pageEncoding="UTF-8"%>
<html>
	<head>
		<title>tomcat-1</title>
	</head>
	<body>
		<h1><font color="red">Session serviced by tomcat</font></h1>
		<table aligh="center" border="1">
			<tr>
				<td>Session ID</td>
				<td><%=session.getId() %></td>
				<% session.setAttribute("abc","abc");%>
			</tr>
			<tr>
				<td>Created on</td>
				<td><%= session.getCreationTime() %></td>
			</tr>
		</table>
	</body>
<html>
```

3. 在tomcat/lib/中添加以下jar包
```
tomcat-redis-session-manage-tomcat7.jar
jedis-2.5.2.jar
commons-pool2-2.2.jar
```

4. 修改context.xml文件
```xml
<Context>
<!-- Default set of monitored resources -->
<WatchedResource>WEB-INF/web.xml</WatchedResource>
<!-- Uncomment this to disable session persistence across Tomcat restarts -->
<!--
<Manager pathname="" />
    -->
<!-- Uncomment this to enable Comet connection tacking (provides events
on session expiration as well as webapp lifecycle) -->
<!--
<Valve className="org.apache.catalina.valves.CometConnectionManagerValve" />
    -->
	<Valve className="com.s.tomcat.redissessions.RedisSessionHandlerValve"/> 
	<Manager className="com.s.tomcat.redissessions.RedisSessionManager" 
			  host="120.79.202.146"
			  port="6380"
			  database="0" 
			  password="password"
			  maxInactiveInterval="60" /> 
</Context>
```
可以尝试新的方案:https://github.com/redisson/redisson/tree/master/redisson-tomcat

### 启动redis







