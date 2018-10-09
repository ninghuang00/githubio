---
title: kafka入门
categories:
  - 开源框架
tags:
  - kafka
  - 消息中间件
date: 2018-08-26 09:34:24
---
 这是摘要
 <!-- more -->


## 入门教程
入门搭建使用参考地址:https://blog.csdn.net/csolo/article/details/52389646

## 一些抽象概念
1. topic,partition,offset
{% asset_img topic.png topic%}
**topic是对一组消息的归纳,kafka对每一个topic的日志进行分区(partition)**
每个分区(partition)都由一系列有序的、不可变的消息组成，这些消息被连续的追加到分区中。分区中的每个消息都**有一个连续的序列号叫做offset,用来在分区中唯一的标识这个消息**。
	1. 在一个可配置的时间段内，Kafka集群保留所有发布的消息，不管这些消息有没有被消费。比如，如果消息的保存策略被设置为2天，那么在一个消息被发布的两天时间内，它都是可以被消费的。之后它将被丢弃以释放空间。**Kafka的性能是和数据量无关的常量级的**，所以保留太多的数据并不是问题。
	2. 实际上每个consumer唯一需要维护的数据是消息在日志中的位置，也就是offset.**这个offset有consumer来维护**：一般情况下随着consumer不断的读取消息，这offset的值不断增加，**但其实consumer可以以任意的顺序读取消息，比如它可以将offset设置成为一个旧的值来重读之前的消息**
以上特点的结合，使Kafka consumers非常的轻量级：它们可以在不对集群和其他consumer造成影响的情况下读取消息。**可以使用命令行来"tail"消息而不会对其他正在消费消息的consumer造成影响**。
将**日志分区可以达到以下目的**：
	1. 首先这使得每个日志的数量不会太大，可以在单个服务上保存。
	2. 另外每个分区可以单独发布和消费，为并发操作topic提供了一种可能。


## 实现高吞吐量原理
1. 顺序读写
kafka的消息是不断追加到文件,这个特性充分利用了磁盘的顺序读写性能,远远快于随机读写,磁盘IO效率高
2. 零拷贝
先简单了解下文件系统的操作流程，例如一个程序要把文件内容发送到网络,这个程序是工作在用户空间，文件和网络socket属于硬件资源，两者之间有一个内核空间在操作系统内部，整个过程为：
{% asset_img kafka零拷贝1.png kafka零拷贝%}
在Linux kernel2.2 之后出现了一种叫做”零拷贝(zero-copy)”系统调用机制，就是跳过“用户缓冲区”的拷贝，建立一个磁盘空间和内存的直接映射，数据不再复制到“用户态缓冲区”系统上下文切换减少为2次，可以提升一倍的性能
{% asset_img kafka零拷贝2.png kafka零拷贝%}

3. 文件分段
 kafka的队列topic被分为了多个区partition，每个partition又分为多个段segment，所以一个队列中的消息实际上是保存在N多个片段文件中通过分段的方式，每次文件操作都是对一个小文件的操作，非常轻便，同时也增加了并行处理能力
{% asset_img 文件分段.png 文件分段%}

4. 批量发送
 Kafka允许进行批量发送消息，先将消息缓存在内存中，然后一次请求批量发送出去比如可以指定缓存的消息达到某个量的时候就发出去，或者缓存了固定的时间后就发送出去如100条消息就发送，或者每5秒发送一次这种策略将大大减少服务端的I/O次数

5. 数据压缩
Kafka还支持对消息集合进行压缩，Producer可以通过GZIP或Snappy格式对消息集合进行压缩压缩的好处就是减少传输的数据量，减轻对网络传输的压力Producer压缩之后，在Consumer需进行解压，虽然增加了CPU的工作，但在对大数据处理上，瓶颈在网络上而不是CPU，所以这个成本很值得


## docker部署kafka
1. 先部署zookeeper
`docker run -d --name zookeeper -p 2181:2181 -t wurstmeister/zookeeper`
2. 部署kafka
`docker run -d --name kafka --link zookeeper -e KAFKA_ADVERTISED_HOST_NAME=120.79.202.146 -e KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181 -e KAFKA_HEAP_OPTS="-Xmx50M -Xms50M" -e KAFKA_ADVERTISED_PORT=9092 -e KAFKA_BROKER_ID=1 -e LOG_CLEANER_DEDUPE_BUFFER_SIZE="20M" -p 9092:9092 -v $PWD/temp:/temp wurstmeister/kafka`

3. 测试消息收发
	1. `docker exec -it ${CONTAINER ID} /bin/bash`
	2. `cd /opt/kafka_2.12-0.10.2.0/`
	3. 创建主题
	`bin/kafka-topics.sh --create --zookeeper zookeeper:2181 --replication-factor 1 --partitions 1 --topic mykafka  `
	4. 运行生产者
	`bin/kafka-console-producer.sh --broker-list localhost:9092 --topic mykafka`
	5. 运行消费者
	`bin/kafka-console-producer.sh --broker-list localhost:9092 --topic mykafka`
	6. 
















