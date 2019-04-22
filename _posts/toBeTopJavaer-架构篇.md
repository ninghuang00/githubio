---
title: toBeTopJavaer-架构篇
categories:
  - 笔记
tags:
  - null
date: 2019-04-19 19:47:02
---
 这是摘要
 <!-- more -->

## 五 架构篇

### 分布式

数据一致性、服务治理、服务降级

#### 分布式事务

2PC、3PC、CAP、BASE、 可靠消息最终一致性、最大努力通知、TCC

#### Dubbo

服务注册、服务发现，服务治理

http://dubbo.apache.org/zh-cn/

#### 分布式数据库

怎样打造一个分布式数据库、什么时候需要分布式数据库、mycat、otter、HBase

#### 分布式文件系统

mfs、fastdfs

#### 分布式缓存

缓存一致性、缓存命中率、缓存冗余

#### 限流降级

Hystrix、Sentinal

#### 算法

共识算法、Raft协议、Paxos 算法与 Raft 算法、拜占庭问题与算法

2PC、3PC

### 微服务

SOA、康威定律

#### ServiceMesh

sidecar

#### Docker & Kubernets

#### Spring Boot

#### Spring Cloud

### 高并发

#### 分库分表

#### CDN技术

#### 消息队列

ActiveMQ

### 监控

#### 监控什么

CPU、内存、磁盘I/O、网络I/O等

#### 监控手段

进程监控、语义监控、机器资源监控、数据波动

#### 监控数据采集

日志、埋点

#### Dapper

### 负载均衡

tomcat负载均衡、Nginx负载均衡

四层负载均衡、七层负载均衡

### DNS

DNS原理、DNS的设计

### CDN

数据一致性