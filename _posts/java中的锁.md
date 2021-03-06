---
title: java中的锁
categories:
  - 笔记
tags:
  - java 
  - 锁
date: 2018-08-18 16:22:37
---
 CAS、乐观锁与悲观锁、数据库相关锁机制、分布式锁、偏向锁、轻量级锁、重量级锁、monitor、
 锁优化、锁消除、锁粗化、自旋锁、可重入锁、阻塞锁、死锁
 <!-- more -->

### 锁 
1. 可重入锁
在JAVA中，内置锁都是可重入的，也就是说，如果某个线程试图获取一个已经由它自己持有的锁时，那么这个请求会立刻成功，并且会将这个锁的计数值加1，而当线程退出同步代码块时，计数器将会递减，当计数值等于0时，锁释放。
2. 不可重入锁
不可重入锁,同一个线程再次请求已经获取过一次的锁时会造成死锁

#### ReentrantLock和synchronized
1. 锁的实现:
ReentrantLock是JDK实现的,synchronized是依赖于JVM实现的
2. 性能的区别:
JDK1.6之后(synchronized引入了偏向锁,轻量级锁(自旋锁)之后)synchronized优化的与ReentrantLock性能差不多,在两种方法都可以使用的情况下官方建议使用synchronized,都是试图在用户态就将加锁问题解决,避免进入内核态
3. 功能的区别:
synchronized的使用比较简洁,由编译器保证加锁和施放锁,而ReentrantLock需要手工声明加锁和施放锁,但是ReentrantLock在锁的细粒度和灵活性方面更强
	1. ReentrantLock可以指定公平锁和非公平锁,synchronized只能是非公平锁
	2. ReentrantLock提供了一个Condition类来实现按照满足条件分组唤醒线程,而synchronized只能是随机唤醒一个线程,要么全部唤醒
	3. ReentrantLock提供了一个可以中断等待锁的机制,通过lock.lockInterruptibly()来实现这个机制

### 对象头
>参考:http://www.hollischuang.com/archives/1953

{% asset_img 对象头.png 对象头%}

#### 锁优化
>参考地址:https://blog.csdn.net/sted_zxz/article/details/76854371

1. 偏向锁
第一个获得锁的线程,如果在接下来的执行过程中，只要该锁没有被其他的线程获取，则持有偏向锁的线程将就不需要再进行同步。
2. 轻量级锁
当发现会有2个线程争抢资源的时候,升级偏向锁,优先使用轻量级锁,同时使用自旋等待锁释放
3. 重量级锁
当2个以上的线程竞争锁,升级为重量级锁同步方法和同步代码块中解释的就是重量级锁。

4. 自旋锁和自适应自旋锁
	1. 就是当后一个线程请求锁的时候,不要一请求不到就马上挂起,而是等待一下(	前提是物理机器有一个以上的处理器),但不放弃处理器的执行时间,看看持有锁的线程是否会很快释放锁,为了让线程等待,只需要让线程执行一个忙循环(自旋)
	2. 自适应意味着自旋的时间不再固定了，而是由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定。 
5. 锁粗化
就是一串连续的零碎操作都对同一个对象加锁,那么就会将加锁同步的范围扩大
6. 锁消除(虚拟机优化) 
虚拟机即时编译器在运行的时候会对一些代码上要求同步但是实际上不会发生数据共享竞争的锁进行消除,根据逃逸分析的数据支持,如果堆上的数据不会逃逸出去从而被其他线程访问到,就可以把他们当做是栈上的数据对待,认为他们是线程私有的,同步加锁自然无需进行,比如说StringBuilder.append()方法.







