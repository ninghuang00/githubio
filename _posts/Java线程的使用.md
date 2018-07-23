---
title: Java线程的使用
categories:
  - 笔记
tags:
  - 线程
  - 进程
date: 2018-07-23 21:46:03
---
 这是摘要
 <!-- more -->
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
    当一个线程的实例被创建即使用new关键字和Thread类或其子类创建一个线程对象后，此时该线程处于新生(new)状态，处于新生状态的线程有自己的内存空间，但该线程并没有运行，此时线程还不是活着的（not alive）；
    2. 就绪状态（Runnable）： 
    通过调用线程实例的start()方法来启动线程使线程进入就绪状态(runnable)；处于就绪状态的线程已经具备了运行条件，但还没有被分配到CPU即不一定会被立即执行，此时处于线程就绪队列，等待系统为其分配CPU，等待状态并不是执行状态； 此时线程是活着的（alive）；
    3. 运行状态（Running）： 
    一旦获取CPU(被JVM选中)，线程就进入运行(running)状态，线程的run()方法才开始被执行；在运行状态的线程执行自己的run()方法中的操作，直到
        1. 调用其他的方法而终止
        2. 等待某种资源而阻塞
        3. 完成任务而死亡
        4. 如果在给定的时间片内没有执行结束，就会被系统给换下来回到线程的等待状态；
    此时线程是活着的（alive）；
    4. 阻塞状态（Blocked）：
    通过调用join()、sleep()、wait()或者资源被暂用使线程处于阻塞(blocked)状态；处于Blocking状态的线程仍然是活着的（alive）
    5. 死亡状态（Dead）：
    当一个线程的run()方法运行完毕或被中断(Thread.interrupt())或被异常退出，该线程到达死亡(dead)状态。此时可能仍然存在一个该Thread的实例对象，当该Thread已经不可能在被作为一个可被独立执行的线程对待了，线程的独立的call stack已经被dissolved。一旦某一线程进入Dead状态，他就再也不能进入一个独立线程的生命周期了。对于一个处于Dead状态的线程调用start()方法，会出现一个运行期(runtime exception)的异常；处于Dead状态的线程不是活着的（not alive）。
5. 定义线程的方法
    1. 继承Thread类,重写run()方法,但是这种方法扩张性不高,因为不能继承其他类
    2. 实现Runnable接口,实现接口中的run()方法,然后创建Thread实例时作为参数传入
6. 线程的方法和属性
    1. 优先级（priority）
    每个类都有自己的优先级，一般property用1-10的整数表示，默认优先级是5，优先级最高是10；优先级高的线程并不一定比优先级低的线程执行的机会高，只是执行的机率高；默认一个线程的优先级和创建他的线程优先级相同；    
    2. Thread.yield()
        1. 让出CPU的使用权，给其他线程执行机会、让同等优先权的线程运行（但并不保证当前线程会被JVM再次调度、使该线程重新进入Running状态)
        2. 如果没有同等优先权的线程，那么yield()方法将不会起作用。    
    3. Thread.sleep(long millis) throws InterruptedException
        1. 当前线程睡眠millis的时间（millis指定睡眠时间是其最小的不执行时间，因为sleep(millis)休眠到达后，无法保证会被JVM立即调度）；
        2. sleep()是一个静态方法(static method) ，所以他不会停止其他的线程；
        3. 线程sleep()时**不会失去拥有的对象锁**。 作用：保持对象锁，让出CPU，调用目的是不让当前线程独自霸占该进程所获取的CPU资源，以留一定的时间给其他线程执行的机会； 
        4. sleep(0)的作用是立刻进行一次CPU竞争,让其他线程有机会执行
    4. thread.join() throws InterruptedException,可以有时间参数
    ```java 
    public static void main(String[] args) {
        Thread thread0 = new Thread(()->{
            for (int i = 0; i < 3; i++) {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
        thread0.start();
        try {
            thread0.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    ```
    ```java 
    public final synchronized void join(long millis)
        throws InterruptedException {
            long base = System.currentTimeMillis();
            long now = 0;

            if (millis < 0) {
                throw new IllegalArgumentException("timeout value is negative");
            }

            if (millis == 0) {
                while (isAlive()) {
                    wait(0);    //wait()方法其实调用的就是wait(0)
                }
            } else {
                while (isAlive()) {
                    long delay = millis - now;
                    if (delay <= 0) {
                        break;
                    }
                    wait(delay);
                    now = System.currentTimeMillis() - base;
                }
            }
        }
    ```
        1. 比如在main()方法中调用了thread0.join()方法,那么main线程会阻塞,等到thread0执行完之后再执行后面的代码
        2. join()方法实际上基于wait()方法实现,join()方法中会调用join(0)方法,join(0)方法(是一个同步方法)中调用了wait(0)方法,main线程先获得thread0的对象锁,然后发现thread0还活着,于是释放锁,并进入等待直到thread0结束(至于结束后怎么通知?)
        3. 可以设置等待时间
    5. object.wait() throws InterruptedException,可以有时间参数
        1. 当一个线程执行到obj.wait()方法时，他就进入到一个和该obj对象相关的等待池(Waiting Pool)中，**同时失去了obj的锁—暂时的，wait后还要返还对象锁。**
        2. **当前线程必须拥有obj的锁才能调用obj.wait()方法**，如果当前线程不是此锁的拥有者，会抛出IllegalMonitorStateException异常,所以wait()必须在synchronized block中调用。   
        2. 比如:
        线程A
        ```java 
        synchronized(obj) {
           while(!condition) {
                     obj.wait();
             }
            obj.doSomething();
        ...
        }
        ```
        当线程A获得obj的锁,但是发现条件不满足的时候,就会调用obj.wait()方法进入等待 
        线程B
        ```java 
        synchronized(obj) {
              condition = true;
              obj.notify();
              ...
        }
        ```
        然后当线程B获得obj的锁,将条件满足,然后调用obj.notify()唤醒等待obj锁的其他线程
        要注意的是:
            1. 当obj.wait()方法返回后，线程A需要再次获得obj锁，才能继续执行。  
            2. 如果A1，A2，A3都obj.wait()，则B调用obj.notify()只能唤醒A1，A2，A3中的一个（具体哪一个由JVM决定）。  
            3. obj.notifyAll()则能全部唤醒A1，A2，A3，但是要继续执行obj.wait()的下一条语句，必须获得obj锁，因此，A1，A2，A3只有一个有机会获得锁继续执行，例如A1，其余的需要等待A1释放obj锁之后才能继续执行。
            4. 当线程B调用obj.notify/notifyAll的时候，B正持有obj锁，因此，A1，A2，A3虽被唤醒，但是仍无法获得obj锁。直到B退出synchronized块，释放obj锁后，A1，A2，A3中的一个才有机会获得锁继续执行。
    6. object.notify()/notifyAll()
        1. 唤醒在object对象等待池中等待的一个线程/所有线程。
        2. notify()/notifyAll()也必须拥有相同对象锁，否则也会抛出IllegalMonitorStateException异常。   
    7. Synchronizing Block
    Synchronized Block/方法控制对类成员变量的访问；
        1. Java中的每一个对象都有唯一的一个内置的锁，每个Synchronized Block/方法只有持有 对应的对象锁才可以执行同步方法，否则所属线程阻塞；
        2. 机锁具有独占性、一旦被一个Thread持有，其他的Thread就不能再拥有（**不能访问其他同步方法,普通方法还是可以的**），方法一旦执行，就独占该锁，直到从该方法返回时才将锁释放，此后被阻塞的线程方能获得该锁，重新进入可执行状态。
    8. 中断
    参考地址:https://www.cnblogs.com/onlywujun/p/3565082.html
        1. Thread.interrupted()方法会返回中断标志位,并将中断标志位清除
        2. Thread.isInterrupted()方法只是返回中断标志位
        2. thread0.interrupt()方法设置中断标志
        3. 线程被synchronized阻塞,即在获取锁的阻塞过程中是不会被中断的,所以中断不能处理死锁
        4. 调用wait(),sleep(),join()方法的等待过程中可以被中断,因此他们都会抛InterruptedException异常
        5. 使用中断来结束阻塞状态:
        while在try里面的时候
```java
public void run() {
    try {
        ...
        /*
         * 不管循环里是否调用过线程阻塞的方法如sleep、join、wait，这里还是需要加上
         * !Thread.currentThread().isInterrupted()条件，虽然抛出异常后退出了循环，显
         * 得用阻塞的情况下是多余的，但如果调用了阻塞方法但没有阻塞时，这样会更安全、更及时。
         */
        while (!Thread.currentThread().isInterrupted()&& more work to do) {
            do more work 
        }
    } catch (InterruptedException e) {
        //线程在wait或sleep期间被中断了
    } finally {
        //线程结束前做一些清理工作
    }
} 
```
        try在while里面的时候,应该在catch块里重新设置一下中断标示，因为抛出InterruptedException异常后，中断标示位会自动清除，此时应该这样：
```java 
public void run() {
    while (!Thread.currentThread().isInterrupted()&& more work to do) {
        try {
            ...
            sleep(delay);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();//是否重新设置中断标示,来决定是否还要循环
        }
    }
}
```
    9. setDaemon()
    设置成守护线程或者用户线程,守护线程和用户线程的区别在于：守护线程依赖于创建它的线程，而用户线程则不依赖。


## 线程进程之间的访问
1. 临界区(Critical Section)
适合一个进程内的多线程访问公共区域或代码段时使用。
2. 互斥量(Mutex)
适合不同进程内多线程访问公共区域或代码段时使用，与临界区相似。
3. 事件(Event)
通过线程间触发事件实现同步互斥。
4. 信号量(Semaphore)
与临界区和互斥量不同，可以实现多个线程同时访问公共区域数据，原理与操作系统中PV操作类似，先设置一个访问公共区域的线程最大连接数，每有一个线程访问共享区资源数就减一，直到资源数小于等于零
