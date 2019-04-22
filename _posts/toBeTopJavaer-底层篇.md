---
title: toBeTopJavaer-底层篇
categories:
  - 笔记
tags:
  - null
date: 2019-04-19 19:46:56
---
 这是摘要
 <!-- more -->

## 二 底层篇

### 计算机基础
#### 字节序
1. 大端
java默认,适合人类阅读,低位在小内存地址,高位在大内存地址

### JVM
{% asset_link Java虚拟机.md java虚拟机 %}
#### JVM内存结构

class文件格式、运行时数据区：堆、栈、方法区、直接内存、运行时常量池、

堆和栈区别

Java中的对象一定在堆上分配吗？

#### Java内存模型

计算机内存模型、缓存一致性、MESI协议

可见性、原子性、顺序性、happens-before、

内存屏障、synchronized、volatile、final、锁

#### 垃圾回收

GC算法：标记清除、引用计数、复制、标记压缩、分代回收、增量式回收

GC参数、对象存活的判定、垃圾收集器（CMS、G1、ZGC、Epsilon）

#### JVM参数及调优

-Xmx、-Xmn、-Xms、Xss、-XX:SurvivorRatio、

-XX:PermSize、-XX:MaxPermSize、-XX:MaxTenuringThreshold

#### Java对象模型

oop-klass、对象头

#### HotSpot

即时编译器、编译优化

#### 虚拟机性能监控与故障处理工具

jps, jstack, jmap、jstat, jconsole, jinfo, jhat, javap, btrace、TProfiler

Arthas

### 类加载机制

classLoader、类加载过程、双亲委派（破坏双亲委派）、模块化（jboss modules、osgi、jigsaw）

### 编译与反编译

什么是编译（前端编译、后端编译）、什么是反编译

JIT、JIT优化（逃逸分析、栈上分配、标量替换、锁优化）

编译工具：javac

反编译工具：javap 、jad 、CRF