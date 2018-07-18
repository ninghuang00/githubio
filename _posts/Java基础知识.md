---
title: Java基础知识
categories:
  - 笔记
tags:
  - Java
date: 2018-07-07 19:51:55
---
 Java基础知识
 <!-- more -->

# java基础 +++
## 面向对象

## 常用类及数据结构
### Object 
1. ==和equals的区别

### String
### StringBuilder
### StringBuffer
### AVL树
### B树
### 红黑树
### 大根堆

---
## 集合
### 线程*安全*的集合
#### Vector

#### HashTable

#### StringBuffer

### 线程*不安全*的集合
#### HashSet

#### LinkedList

#### ArrayList
`elementData[size++] = e;`
	从上面的源码中可以看出,ArrayList中增加一个元素的时候会分层两步进行:
1. 先在elementData[size++]的地方存放元素,
2. 然后再增大size.
	因此如果是多线程进行访问的话,有可能两个线程在同一个位置存放了元素.

#### HashMap
参考地址:http://www.importnew.com/22011.html
HashMap的底层实现原理
* HashMap的结构:是一个Node<K,V>[]的数组,Node实现了Map.Entry,初始容量16(代码中是`1<<4`),最大是2^30,当检查到插入数据时容量超出`capacity*loadFactor`,就要增大Hash表的尺寸,
* 放入规则:计算A的键的hash值,然后%数组长度就是index,放在数组的index位置,如果后面put进来B的键的hash值和A相等的,那就是`B.next=A,Node[index]=B`
* 为什么capacity的值是2的次幂:因为`index = hash & (length - 1)`,相当于取余,这样length-1的二进制就是1,和hash进行&操作之后不容易产生相同的结果,也就是尽量减少在同一个index放置多个元素,这样不但空间利用率最高,同时为了利用数组查找元素速度,我们当然是希望每一个位置只有一个元素,这样就不用遍历链表去比对key了.
* resize()死循环的问题:当HashMap进行扩容的时候,原来的链表就要进行重排,这个时候如果并发操作,由于transfer()函数,就可能形成环,然后下次get元素的时候就可能进入死循环,
jdk1.8好像解决了这个问题,进行优化之后省去了重新计算hash值的时间,而是通过判断扩容之后hash中新增的最高一位范围是1还是0(0则是原位置,1则是原位置+oldCap),均匀地将之前冲突的节点分散到新的桶上,
* 支持NULL键和NULL值,而且null值的key总是放在数组第一个位置
* 遍历的方法,取出keySet().iterator(),遍历iterator,即Map.entry,从entry中取key和value比较高效.

---
## 一致性Hash算法
参考地址:http://blog.csdn.net/cywosp/article/details/23397179
一致性hash算法提出了在动态变化的Cache环境中，判定哈希算法好坏的四个定义:
1. 平衡性
2. 单调性
3. 分散性
4. 负载

## 异常 

---
## 反射
### 访问私有变量和私有方法
1. 调用类的getDeclaredFields()和getDeclaredMethods()方法获得相应的field和method
2. 调用method和field的setAccessible()方法允许访问私有成员属性
3. field调用get(实例)方法,获得对应field的值
4. method调用invoke()方法执行私有方法

---
## 泛型
### 注意点
1. 泛型方法和泛型类的<>互不相干
2. 泛型擦除如果泛型参数没有指定上限(如<T extends String>),那类型擦除之后T就被替换成Object,否则就是上限,如String
3. 类型擦除也是为什么泛型类和泛型方法中不接受基本数据类型的原因

---
## IO/NIO
{% post_link NIO的使用 %}
### 简介
参考地址:https://blog.csdn.net/shimiso/article/details/24990499
**同步和异步说的是消息的通知机制,阻塞和非阻塞说的是线程的状态**
1. 同步阻塞io
服务器实现是一个连接创建一个线程,直到连接断开这个线程都被这个连接占用
指的就是传统的Java IO,线程发起io后会阻塞,等到io结束之后才会恢复
同步阻塞时序图
{% asset_img 同步阻塞时序图.png 同步阻塞时序图%}
2. 同步非阻塞io
服务器实现是一个请求创建一个线程,或者说多个连接对应一个线程,由于线程数量小于连接数量,所以IO操作不能够阻塞
Java中的NIO(Non-blocking IO)是这种模型,线程发起io后可以去干其他事情,但是另外起一个线程去监听或者轮询io状态(是否有可读数据,或者是否可以写入);传统io中,比如read()操作,当没有数据可读的时候线程就会阻塞;但是在NIO中,没有数据可读的时候read()就会返回立即0
    * 有一个专门的线程来处理所有的IO事件,并负责分发
    * 事件触发机制:事件到来的时候处罚,而不是同步监视事件
    * 线程通讯：线程之间通过 wait,notify 等方式通讯。保证每次上下文切换都是有意义的。减少无谓的线程切换。
{% asset_img 同步非阻塞时序图.png 同步非阻塞时序图%}
3. 异步非阻塞io
服务器实现是一个有效请求创建一个线程
Java中的AIO((Asynchronous IO))是这种模型,线程发起io后可以直接干其他事前,系统会监听io状态,IO完成后会通知线程
{% asset_img 异步非阻塞时序图.png 异步非阻塞时序图%}


### 文件
1. 文件是怎么在磁盘上存储的

# Jdk8特性 +++

---
## lambda表达式

---
## 函数式编程

---
## stream

---
## FP


# 并发
## 处理高并发
* 举例子,代码实现

# 多线程 +++
## 线程
1. 什么是线程
2. 线程和进程的区别
JVM以及程序和可执行文件,class文件等是一个静态的概念,JVM实例以及进程和线程是一个动态的概念.
JVM运行起来的java程序实例其实就是一个进程,然后这个进程下面有一个主线程,然后可以有很多子线程,所以线程是进程里的多个执行路径,是基本的调度单位.
线程不仅可以共享进程的内存,还拥有自己内存空间(线程栈),创建线程时由系统分配.
java中的多线程是一种抢占机制而不是分时机制,抢占机制指的是多个线程处于可运行状态,但是只允许一个线程处于运行状态.
3. 系统可以启动的线程数量限制
不考虑系统本身的限制,主要和以下JVM启动参数有关
    1. -Xms()初始堆栈大小,
    2. -Xmx()最大堆栈大小,
    3. -Xss()单个线程堆栈
4. 线程的状态
{% asset_img 线程的状态.jpg 线程的状态%}
    1. 新生状态（New）： 
    当一个线程的实例被创建即使用new关键字和Thread类或其子类创建一个线程对象后，此时该线程处于新生(new)状态，处于新生状态的线程有自己的内存空间，但该线程并没有运行，此时线程还不是活着的（not  alive）；
    2. 就绪状态（Runnable）： 
    通过调用线程实例的start()方法来启动线程使线程进入就绪状态(runnable)；处于就绪状态的线程已经具备了运行条件，但还没有被分配到CPU即不一定会被立即执行，此时处于线程就绪队列，等待系统为其分配CPCU，等待状态并不是执行状态； 此时线程是活着的（alive）；
    3. 运行状态（Running）： 
    一旦获取CPU(被JVM选中)，线程就进入运行(running)状态，线程的run()方法才开始被执行；在运行状态的线程执行自己的run()方法中的操作，直到调用其他的方法而终止、或者等待某种资源而阻塞、或者完成任务而死亡；如果在给定的时间片内没有执行结束，就会被系统给换下来回到线程的等待状态；此时线程是活着的（alive）；
    4. 阻塞状态（Blocked）：
    通过调用join()、sleep()、wait()或者资源被暂用使线程处于阻塞(blocked)状态；处于Blocking状态的线程仍然是活着的（alive）
    5. 死亡状态（Dead）：
    当一个线程的run()方法运行完毕或被中断或被异常退出，该线程到达死亡(dead)状态。此时可能仍然存在一个该Thread的实例对象，当该Thready已经不可能在被作为一个可被独立执行的线程对待了，线程的独立的call stack已经被dissolved。一旦某一线程进入Dead状态，他就再也不能进入一个独立线程的生命周期了。对于一个处于Dead状态的线程调用start()方法，会出现一个运行期(runtime exception)的异常；处于Dead状态的线程不是活着的（not  alive）。
5. 定义线程的方法
    1. 继承Thread类,重写run()方法,但是这种方法扩张性不高,因为不能继承其他类
    2. 实现Runnable接口,实现接口中的run()方法
6. 线程的方法和属性
    1. 优先级（priority）
    每个类都有自己的优先级，一般property用1-10的整数表示，默认优先级是5，优先级最高是10；优先级高的线程并不一定比优先级低的线程执行的机会高，只是执行的机率高；默认一个线程的优先级和创建他的线程优先级相同；    
    2. Thread.sleep()/sleep(long millis)
    当前线程睡眠/millis的时间（millis指定睡眠时间是其最小的不执行时间，因为sleep(millis)休眠到达后，无法保证会被JVM立即调度）；sleep()是一个静态方法(static method) ，所以他不会停止其他的线程也处于休眠状态；线程sleep()时不会失去拥有的对象锁。 作用：保持对象锁，让出CPU，调用目的是不让当前线程独自霸占该进程所获取的CPU资源，以留一定的时间给其他线程执行的机会； 
    3. Thread.yield()
    让出CPU的使用权，给其他线程执行机会、让同等优先权的线程运行（但并不保证当前线程会被JVM再次调度、使该线程重新进入Running状态），如果没有同等优先权的线程，那么yield()方法将不会起作用。    
    4. thread.join()
    使用该方法的线程会在此之间执行完毕后再往下继续执行。    
    5. object.wait()
    当一个线程执行到wait()方法时，他就进入到一个和该对象相关的等待池(Waiting Pool)中，同时失去了对象的机锁—暂时的，wait后还要返还对象锁。当前线程必须拥有当前对象的锁，如果当前线程不是此锁的拥有者，会抛出IllegalMonitorStateException异常,所以wait()必须在synchronized block中调用。   
    6. object.notify()/notifyAll()
    唤醒在当前对象等待池中等待的第一个线程/所有线程。notify()/notifyAll()也必须拥有相同对象锁，否则也会抛出IllegalMonitorStateException异常。   
    7. Synchronizing Block
    Synchronized Block/方法控制对类成员变量的访问；Java中的每一个对象都有唯一的一个内置的锁，每个Synchronized Block/方法只有持有调用该方法被锁定对象的锁才可以访问，否则所属线程阻塞；机锁具有独占性、一旦被一个Thread持有，其他的Thread就不能再拥有（不能访问其他同步方法），方法一旦执行，就独占该锁，直到从该方法返回时才将锁释放，此后被阻塞的线程方能获得该锁，重新进入可执行状态。

## 线程池
ThreadPoolExecutor

### CountdownLatch

### CyclicBarrier

### jdk原生线程池
参考地址:http://www.importnew.com/19011.html
####构造函数中各个参数的作用(四个构造函数中前三个构造函数都调用了最后一个构造器进行初始化)
1. corePoolSize:核心池的大小,创建线程池之后默认情况下,线程池中是没有线程的,除非调用了prestartCoreThread()方法预创建线程,然后当有任务进来的时候,就会开始创建线程去执行任务,当线程池中的线程数目达到corePoolSize之后,就会把到达的任务放到缓存队列中
2. maximumPoolSize:线程池的最大线程数,
3. keepAliveTime:表示空闲线程能够存活的时间,默认情况下,只有当线程数大于corePoolSize的时候此参数才会生效,即当一个线程空闲的时间超过此参数时,线程就会终止.如果调用allowCoreThreadTimeOut(true)方法,那么线程数只要大于0,次参数就会生效.次参数的单位可以从天到纳秒
4. workQueue:一个阻塞队列,用来存储等待执行的任务,重要参数,对线程池的运行过程产生重大影响,这里的阻塞队列一般有以下几种选择:
	1. ArrayBlockingQueue;
	2. LinkedBlockingQueue;
	3. SynchronousQueue;
线程池的排队策略和阻塞队列的选择有关
5. threadFactory:线程工厂,用来创建线程
6. handler:当线程池满了的时候选取的拒绝处理任务的策略:
	1. ThreadPoolExecutor.AbortPolicy:丢弃任务并抛出RejectedExecutionException异常。 
	2. ThreadPoolExecutor.DiscardPolicy：也是丢弃任务，但是不抛出异常。 
	3. ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）
	4. ThreadPoolExecutor.CallerRunsPolicy：由调用线程处理该任务

####ThreadPoolExecutor类中几个重要的方法:
	1. execute():实现了Executor接口中的方法,通过此方法想线程池提交任务,交由线程池去执行
	2. submit():继承而来的方法,也是向线程池提交任务,不过可以返回任务执行的结果,其实内部实现还是调用了execute()方法,只是利用Future来获取任务执行结果
	3. shutdown():用来关闭线程池
	4. shutdownNow():同上
####深入理解线程池的实现
1. 线程池的状态:
```java
volatile int runState;	//用来保证线程之间可见
static final int RUNNING    = 0;	//创建线程池后的初始状态
static final int SHUTDOWN   = 1;	//调用shutdown()方法,此时线程池不接受新的任务,会等待现有的任务执行完毕
static final int STOP       = 2;	//调用shutdownNow()方法,不接受新任务并且尝试终止正在执行的任务
static final int TERMINATED = 3;	//当线程池处于STOP或者SHUTDOWN状态,然后所有的工作线程都已经销毁,任务缓存队列中被清空或者执行结束,线程池被置为TERMINATED
```
2. 任务的执行
ThreadPoolExecutor类中比较重要的几个成员变量
```java 
private final BlockingQueue<Runnable> workQueue;              //任务缓存队列，用来存放等待执行的任务
private final ReentrantLock mainLock = new ReentrantLock();   //线程池的主要状态锁，对线程池状态（比如线程池大小、runState等）的改变都要使用这个锁
private final HashSet<Worker> workers = new HashSet<Worker>();  //用来存放工作集
private volatile long  keepAliveTime;    //线程存货时间   
private volatile boolean allowCoreThreadTimeOut;   //是否允许为核心线程设置存活时间
private volatile int   corePoolSize;     //核心池的大小（即线程池中的线程数目大于这个参数时，提交的任务会被放进任务缓存队列）
private volatile int   maximumPoolSize;   //线程池最大能容忍的线程数
private volatile int   poolSize;       //线程池中当前的线程数
private volatile RejectedExecutionHandler handler; //任务拒绝策略
private volatile ThreadFactory threadFactory;   //线程工厂，用来创建线程
private int largestPoolSize;   //用来记录线程池中曾经出现过的最大线程数
private long completedTaskCount;   //用来记录已经执行完毕的任务个数
```
ThreadPoolExecutor类中的execute()的实现原理:
```java 
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();
    /*
     * Proceed in 3 steps:
     */
    int c = ctl.get();
    //如果当前线程数小于核心线程数,则尝试启动新线程,调用的addWorker()方法会原子性检查runState和workCount,防止错误添加线程
    //如果无法添加则重新获取当前线程数
    if (workerCountOf(c) < corePoolSize) {
    	//addWorker()方法中使用了ReentrantLock锁来保证线程池的状态
        if (addWorker(command, true))
            return;
        c = ctl.get();
    }
    //如果当前线程处于RUNNING状态,则将新任务加入缓存队列
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
        //如果线程池变成非RUNNING状态,移除新任务,并reject()
        if (! isRunning(recheck) && remove(command))
            reject(command);
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    //如果当前线程不处于RUNNING状态,尝试新建线程,创建线程失败则reject()
    else if (!addWorker(command, false))
        reject(command);
}
```
总结一下任务提交给线程池之后到执行的过程:
	1. 弄清楚corePoolSize和maximumPoolSize的含义
	2. Worker的作用
	3. 任务交给线程池的处理策略
		* 当前线程数小于核心线程数,创建新的线程处理任务
		* 如果当前线程数大于核心线程数,那么会尝试将新任务添加到缓存队列,如果添加成功,那么新任务等待空闲线程来处理;如果添加失败,一般是队列已经满了,那么会尝试创建新的线程去处理
		* 当前线程数达到maximumPoolSize时,会采取任务拒绝策略处理
		* 如果当前线程数量大于corePoolSize,那么当有线程的空闲时间超过keepAliveTime,线程将被终止,知道当前线程数不大于corePoolSize;如果允许核心线程设置空闲存活时间的话,那核心线程的空闲时间超出也会被终止.
3. 线程池的初始化
	1. prestartCoreThread():初始化一个核心线程,
	2. prestartAllCoreThread():初始化所有核心线程.
	如何合理配置线程池的大小(N为cpu个数)
```
1. 如果是CPU密集型应用，则线程池大小设置为N+1
2. 如果是IO密集型应用，则线程池大小设置为2N+1
```
	以上公式只是最简单的情况(假设一台服务器上),一般线程池大小还要结合部署的系统中的CPU个数,内存大小,主要的任务执行的是计算还是I/O,还是混合操作
	比如I/O优化中,如下公式更合适:
```
最佳线程数目 = （（线程等待时间+线程CPU时间）/线程CPU时间 ）* CPU数目
```
因为很显然，线程等待时间所占比例越高，需要越多线程。线程CPU运行时间所占比例越高，需要越少线程。
4. 任务缓存队列及排队策略
在前面我们多次提到了任务缓存队列，即workQueue，它用来存放等待执行的任务。
workQueue的类型为BlockingQueue<Runnable>，通常可以取下面三种类型：
	1. ArrayBlockingQueue：基于数组的先进先出队列，此队列创建时必须指定大小；
	2. LinkedBlockingQueue：基于链表的先进先出队列，如果创建时没有指定此队列大小，则默认为Integer.MAX_VALUE；
	3. synchronousQueue：这个队列比较特殊，它不会保存提交的任务，而是将直接新建一个线程来执行新来的任务。
5. 任务拒绝策略:上面提到的4中
6. 线程池的关闭:上面提到的2种方法
7. 线程池容量的动态调整
ThreadPoolExecutor提供了动态调整线程池容量大小的方法：setCorePoolSize()和setMaximumPoolSize()，
当上述参数从小变大时，ThreadPoolExecutor进行线程赋值，还可能立即创建新的线程来执行任务。

## J.U.C

## Atomic

## CAS算法
Compare and Swap，即比较再交换。
参考地址:https://www.jianshu.com/p/21be831e851e


## fork/join 

## 线程的几种状态


# 设计模式
{% asset_link 设计模式.md 设计模式%}

# Servlet容器 ++
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

## tomcat
### tomcat Servlet
1. 是不是线程安全的:
单例模式,当客户端第一次请求该servlet的时候会根据web.xml实例化servlet,之后客户端再访问该servlet使用的就是同一实例,是不是线程安全看servlet的实现,如果servlet的内部属性会被多线程改变,那就是不安全的.

## jetty 



# 调优技巧 ++
## 实践积累

# 坑(容易犯错的细节)
## 细节
```java 
<!-- 代码段 -->
int i = 0;
i = i++;
<!-- 结果i还是0  -->

/*代码段*/
int i = 0;
i = ++ i;
<!-- 结果i是1 -->

<!-- 基本数值类型转换问题 -->
int a = 9;
int b = 4;
double c = a/b;
<!-- 结果是2.0 -->

<!-- 控制sout的小数输出位数 -->
sout("%.2f",doubleNum);
<!-- 结果输出的double数字会保留两位小数 -->
```

## 值调用和引用调用
关于java中的值调用和引用调用(java核心技术p120)
引用调用是引用地址的拷贝,虽然能够实际影响到引用对象的状态,但是一个方法无法让一个对象引用一个新的对象,比如:
```java 
public static void swap(Person p1,Person p2){
	Person temp = p1;
	p1 = p2;
	p2 = temp;
}
```
	以上代码是无法实现交换功能的.
	
##关于new对象的位置很有讲究,如果在循环的外面new的话,相当于add进list中的都是同一个对象
```java 
List<String[]> list = new ArrayList<>();    //关于课程信息的一行数据
    //关于课程的所有字段
    String[] fieldNames = {DbHelper.COURSE_NAME, DbHelper.START_DATE, DbHelper.END_DATE, DbHelper.DAY_OF_WEEK, DbHelper.START_TIME, DbHelper.END_TIME, DbHelper.TEACHER_NAME, DbHelper.ADDRESS,DbHelper.COURSE_ID};
    //从数据库取出的一行数据
    String[] fieldValues ;//千万不能在这里new ,不然后面add进list的全是这个地址= new String[fieldNames.length];
    Cursor cursor = db.query(DbHelper.TABLE_NAME_COURSE, null, null, null, null, null, null);
    if (cursor.moveToFirst()) {
        do {
            fieldValues = new String[fieldNames.length];
            for(int i = 0; i < fieldNames.length; i ++) {
                fieldValues[i] = cursor.getString(cursor.getColumnIndex(fieldNames[i]));
            }
            Log.d("hn", fieldValues.toString());
            System.out.println(fieldValues.toString());
            list.add(fieldValues);
        } while (cursor.moveToNext());
    }
    cursor.close();
```