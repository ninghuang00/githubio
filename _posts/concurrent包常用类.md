---
title: concurrent包常用类
categories:
  - 笔记
tags:
  - Executors
  - ConcurrentHashmap
  - cas
  - atomic
  - 线程安全
date: 2018-08-19 14:15:54
---
 参考地址:https://my.oschina.net/yu120/blog/689204
 <!-- more -->

## ConcurrentHashmap(K/V not null)
参考地址(更完整):https://blog.csdn.net/u010412719/article/details/52145145
https://www.jianshu.com/p/e694f1e868ec
1. 1.8中的实现
{% asset_img ConcurrentHashMap1.8.png ConcurrentHashMap1.8%}
使用Node+CAS+synchronized实现

sizeCtl是控制标识符，不同的值表示不同的意义。
> * 负数代表正在进行初始化或扩容操作 ,其中-1代表正在初始化 ,-N 表示有N-1个线程正在进行扩容操作
* 正数或0代表hash表还没有被初始化，这个数值表示初始化或下一次进行扩容的大小，类似于扩容阈值。它的值始终是当前ConcurrentHashMap容量的0.75倍，这与loadfactor是对应的。实际容量>=sizeCtl，则扩容。

2. put()的实现大致流程
    1. 检查key/value是否为空，如果为空，则抛异常，否则进行2
    为什么不支持key/value为null
        1. 如果支持value为null,并发状态下,通过get(key)获取的value如果是null,则无法判断map中是没有这个key,还是key对应的value就是null;如果是单线程则可以用containsKey()判断
    2. 进入for死循环(直到插入成功)，
        1. 检查table是否初始化了，如果没有，则调用initTable()进行初始
        2. 根据key的hash值计算出其应该在table中储存的位置i，取出table[i]的节点用f表示。如果f==null(即该位置的节点为空，没有发生碰撞)
        则利用CAS操作直接存储在该位置，插入空桶不用加锁,如果CAS操作成功则退出死循环。
        3. 检查table[i]的节点的hash是否等于MOVED，如果等于，则检测到正在扩容，则帮助其扩容
        4. 如果table[i]!=null(即该位置已经有其它节点，发生碰撞)
            synchronized(f),然后判断f==table[i]
            1. 如果table[i]为链表节点，更新成功后,break
            2. 如果table[i]为树节点，更新成功后,break
    3. 如果table[i]的节点是链表节点，则检查table的第i个位置的链表是否需要转化为红黑树，如果需要则调用treeifyBin函数进行转化

2. 扩容transfer()
> 参考地址:https://juejin.im/post/5b00160151882565bd2582e0

2. 1.7中的实现
使用HashEntry和Segment(继承ReentrantLock)实现
{% asset_img ConcurrentHashMap1.7.png ConcurrentHashMap1.7%}

## ReentrantLock
1. 锁的实现:
参考地址:https://www.jianshu.com/p/fe027772e156
ReentrantLock基于`volatile int state`和cas实现,尝试获得锁的时候会判断state的值,
    1. 如果是0的话,使用cas算法修改state加1
    2. 不是0的话说明锁已经被占用,使用cas加1,进入等待队列
ReentrantLock是JDK实现的,synchronized是依赖于JVM实现的
2. 性能的区别:
JDK1.6之后(synchronized引入了偏向锁,轻量级锁(自旋锁)之后)synchronized优化的与ReentrantLock性能差不多,在两种方法都可以使用的情况下官方建议使用synchronized,都是试图在用户态就将加锁问题解决,避免进入内核态
3. 功能的区别:
synchronized的使用比较简洁,由编译器保证加锁和施放锁,而ReentrantLock需要手工声明加锁和施放锁,但是ReentrantLock在锁的细粒度和灵活性方面更强
	1. ReentrantLock可以指定公平锁(FIFO)和非公平锁(默认是这个),synchronized只能是非公平锁(一个线程想要获得锁,会先尝试插队)
	2. ReentrantLock提供了一个Condition类来实现按照满足条件分组唤醒线程,而synchronized只能是随机唤醒一个线程,要么全部唤醒
	3. ReentrantLock提供了一个可以中断等待锁的机制,通过lock.lockInterruptibly()来实现这个机制
	4. Lock实现了可轮询锁,使用if(lock.tryLock())可以尝试获得锁,如果获取不到则else
4. 使用方法
```java
class SafeSeqWithLock{  
    private long count = 0;  
    private ReentrantLock lock = new ReentrantLock();  
    public void inc()  
    {  
        lock.lock();  
  
        try{  
            count++;  
        }  
        finally{  
            lock.unlock();  
        }  
    }  
    public long get()  {  return count;  }  
}
```

## Condition
> 参考地址:https://blog.csdn.net/vernonzheng/article/details/8288251
https://blog.csdn.net/u011486491/article/details/77849326

生产者消费者模型:
```java
public class ProductQueue<T> {
    private final T[] items;
    private final Lock lock = new ReentrantLock();
    private Condition notFull = lock.newCondition();
    private Condition notEmpty = lock.newCondition();
    private int head, tail, count;

    public ProductQueue(int maxSize) {
        items = (T[]) new Object[maxSize];
    }
    public ProductQueue() {
        this(10);
    }
    public void put(T t) throws InterruptedException {
        lock.lock();
        try {
            while (count == getCapacity()) {
            	//队列满了就挂起,等待notFull.signal()唤醒
            	//此时会释放锁,等到唤醒是再次获得锁
                notFull.await();	
            }
            items[tail] = t;
            if (++tail == getCapacity()) {
                tail = 0;
            }
            ++count;
            notEmpty.signalAll();	//队列不空,唤醒之前调用take()但是因为队列空而挂起的线程
        } finally {
            lock.unlock();
        }
    }
    public T take() throws InterruptedException {
        lock.lock();
        try {
            while (count == 0) {
                notEmpty.await();	//如果队列空,挂起线程
            }
            T ret = items[head];
            items[head] = null;//GC
            //
            if (++head == getCapacity()) {
                head = 0;
            }
            --count;
            notFull.signalAll();	//唤醒挂起的线程
            return ret;
        } finally {
            lock.unlock();
        }
    }
    public int getCapacity() {
        return items.length;
    }
    public int size() {
        lock.lock();
        try {
            return count;
        } finally {
            lock.unlock();
        }
    }
}
```

## ReentrantReadWriteLock
读锁可以有很多个锁同时上锁，只要当前没有写锁
写锁是排他的，上了写锁，其他线程既不能上读锁，也不能上写锁；同样，需要上写锁的前提是既没有读锁，也没有写锁；  

## Atomic
线程安全的并且无阻塞的类
1. AtomicLong
在多线程操作的时候不需要额外的同步处理
```java
public class AtomicTest {

    public void testAtomic() {
        final int loopcount = 10000;
        int threadcount = 10;

        final NonSafeSeq seq1 = new NonSafeSeq();
        final SafeSeq seq2 = new SafeSeq();

        final CountDownLatch l = new CountDownLatch(threadcount);

        for (int i = 0; i < threadcount; ++i) {
            final int index = i;
            new Thread(() -> {
                for (int j = 0; j < loopcount; ++j) {

                    seq1.inc();
                    seq2.inc();
                }
                System.out.println("finished : " + index);
                l.countDown();
            }).start();
        }

        try {
            l.await();
        } catch (InterruptedException e) {

            e.printStackTrace();
        }

        System.out.println("both have finished....");

        System.out.println("NonSafeSeq:" + seq1.get());
        System.out.println("SafeSeq with atomic: " + seq2.get());

    }
}

class NonSafeSeq {
    private long count = 0;
    public void inc() {count++;}
    public long get() {return count;}
}

class SafeSeq {
    private AtomicLong count = new AtomicLong(0);
    public void inc() {count.incrementAndGet();}
    public long get() {return count.longValue();}
}
```


## CAS算法
参考地址:https://www.jianshu.com/p/21be831e851e
Compare and Swap，即比较再交换。就是java中一种乐观锁机制,实际上是一种无锁算法
1. 基本原理
{% asset_img cas原理.png cas原理%}
因为t1和t2线程都同时去访问同一变量56，所以他们会把主内存的值完全拷贝一份到自己的工作内存空间，所以t1和t2线程的预期值都为56。
假设t1在与t2线程竞争中线程t1能去更新变量的值，而其他线程都失败。（失败的线程并不会被挂起，而是被告知这次竞争中失败，并可以再次发起尝试）。t1线程去更新变量值改为57，然后写到内存中。此时对于t2来说，内存值变为了57，与预期值56不一致，就操作失败了（想改的值不再是原来的值）。
CPU去更新一个值，但如果想改的值不再是原来的值，操作就失败，因为很明显，有其它操作先改变了这个值。
2. ABA问题
比如一个栈中有a,b,c三个元素,现在栈顶是a,
    1. 在cas模式下,线程1打算将栈顶变成b,
    2. 这时候线程2将a,b弹出,再将a压入,
    3. 线程1发现栈顶还是a,就将栈顶变成了b
那么此时栈里面只剩下b一个元素,丢失了c
3. 解决aba问题
AtomicStampedReference类中通过时间戳解决

## Executors工具类
>参考地址:https://zhuanlan.zhihu.com/p/32867181

1. 重载实现的newFixedThreadPool()
创建一个可重用固定线程数的线程池，超出的线程会在队列中等待。因为阻塞队列无界,要慎用,避免OOM
```java
public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      //基于链表的先进先出队列，如果创建时没有指定此队列大小，则默认为Integer.MAX_VALUE
                                      new LinkedBlockingQueue<Runnable>());
    }
```
2. 重载实现的newCachedThreadPool()
创建一个可缓存线程池，如果执行新任务时没有空闲的线程，新建线程执行任务；否则调用空闲线程执行新任务。对于执行很多短期异步任务的程序而言，比较合适任务的提交速度小于任务的处理速度的场景,不然很容易OOM
```java
    public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      //这个队列比较特殊，它不会保存提交的任务，而是将直接新建一个线程来执行新来的任务
                                      new SynchronousQueue<Runnable>());
    }
```
3. 重载实现的newSingleThreadExecutor()
创建一个使用单个 worker 线程的 Executor，以无界队列方式来运行该线程，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。
```java
public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
```
4. 重载实现的newSingleThreadScheduledExecutor()
5. 重载实现的newScheduledThreadPool()
创建一个定长线程池，支持定时及周期性任务执行。
```java
  public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
        return new ScheduledThreadPoolExecutor(corePoolSize);
    }
    //延迟执行调用方式
    service.schedule(new MyThread(), 2, TimeUnit.SECONDS);
    //周期性执行调用方式
    service.scheduleAtFixedRate(new MyThread(), 2,3, TimeUnit.SECONDS);
```

## CountDownLatch
1. 原理
CountDownLatch内部维护一个计数器，计数器的值为待完成的任务数Ｎ，需要等待这Ｎ个任务完成的线程调用CountDownLatch的await()方法使自己进入休眠等待状态。
当某一个任务线程完成某一个任务后调用CountDownLatch的countDown()方法来表示自己的任务已完成，此时CountDownLatch的计数器值减１，当所有的任务完成式，计数器的值为０。当计数器值为0时，CountDownLatch将唤醒所有因await()方法进入休眠的线程。
2. 使用
```java

public static void main(String[] args){
    final CountDownLatch countDownLatch = new CountDownLatch(allGates.size());
    ExecutorService threadPool = Executors.newCachedThreadPool();

    for () {
        threadPool.submit(()-> {
            try {
                //do sth
            } catch (Exception e) {
                LOGGER.error("error", e);
            } finally {
                //每次结束一个线程计数就减一
                countDownLatch.countDown();
            }
        });
    }
    
    try {
        //主线程进入等待,等待时间结束或者计数减到0将会被唤醒
        //计数减到0返回true,超时返回false
        countDownLatch.await(MAXTIMEOUT_SECONDS, TimeUnit.SECONDS);
    } catch (InterruptedException e) {
        LOGGER.error("Interrupt exception", e);
    } finally {
        threadPool.shutdownNow();//关闭线程池
    }
}
```