---
title: zookeeper入门
categories:
  - 笔记
tags:
  - zookeeper
date: 2018-10-07 10:25:10
---
 参考地址:https://mp.weixin.qq.com/s/OzVlgGEaLRgnhhK9gEd9xQ
 <!-- more -->

## 什么是zookeeper
是一个开源的分布式协调服务,用于以简单而稳健的方式访问应用程序.后来Apache zookeeper成为hadoop,hbase和其他分布式框架使用的有组织服务的标准.它的设计目的就是将那些复杂且容易出错的分布式一致性服务封装起来,构成一个高效可靠的原语集(原语具有不可分割性),并向用户提供接口.

## zookeeper应用场景
zookeeper是一个典型的分布式数据一致性解决方案,分布式应用可以基于zookeeper实现:数据的发布/订阅,负载均衡,分布式协调/通知,命名服务,集群管理,master选举.分布式锁和分布式队列等
1. 担任服务生产消费者的注册中心
{% asset_img dubbo架构.png %}
服务生产者将自己提供的服务注册到zookeeper中心,服务消费者需要调用服务的时候先到注册中心查询服务信息,然后根据返回的服务详细信息去调用服务生产者的内容和数据.

2. 作为dubbo的注册中心
主要提供的功能:
	* 集群管理:容错,负载均衡
	* 配置文件集中管理
	* 集群的入口


## 集群角色
1. leader
2. follower
3. observer




## 相关概念
1. 为什么最好用奇数个服务器构成zookeeper集群
zookeeper中的leader选举算法使用的是zab协议,它的核心思想就是当多数server写成功,则任务数据写成功,也就是说不管是3台server还是4台server,都是最多只允许1台server挂掉,那么他们的可靠性是一致的.
2. zab算法和paxos算法
	* zab(zookeeper atomic broadcast原子广播)
	该协议是为zookeeper专门设计的一种支持崩溃恢复的原子广播协议,主要依赖该协议来实现分布式数据一致性,实现一种主备模式的系统架构来保持集群中各个副本之间的数据一致性
3. session
4. znode
5. watcher
6. acl(AccessControlLists)权限控制