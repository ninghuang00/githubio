---
title: iBase4J部署笔记
categories:
  - 笔记
tags:
  - ibase4j
  - nginx
  - redis
  - zookeeper
date: 2018-07-08 13:28:59
---
 这是摘要
 <!-- more -->

# 安装配置zookeeper


# 安装配置Nginx



# 安装配置Redis
* 安装路径 
	`/usr/local/redis`
* 各种命令
	1. 启动服务器
		进入`/usr/local/bin`,执行命令`redis-server`,或者`redis-server /etc/redis.conf`
	2. 启动客户端
		进入`/usr/local/bin`,执行命令`redis-cli`
	3. 关闭服务器
		`ps -u jim(替换成你的用户名) -o pid,rss,command | grep redis-server`,或者`kill redis`r然后按tab键会自动转换成pid.