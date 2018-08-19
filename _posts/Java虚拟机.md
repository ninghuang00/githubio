---
title: Java虚拟机
categories:
  - 笔记
tags:
  - Java
  - 虚拟机
  - JVM
  - 线程安全
date: 2018-07-08 16:12:32
---
 JVM相关知识
 <!-- more -->



## 内核态
### 是什么
### 保护模式

## 类加载过程
{% asset_link JVM类加载机制.md JVM类加载机制%}
1. 加载
2. 验证
3. 准备
4. 解析
5. 初始化


---
## 内存管理
### StackOverflow
启动一个新的线程的时候,虚拟机会分配一个栈,java栈以栈帧的方式保存线程运行状态,然后线程调用方法的时候,虚拟机就会压入新的栈帧,只要方法没有返回,栈帧就会存在,当栈帧太多就会导致溢出.
### OutOfMemory
1. 栈内存溢出
	就是启动的线程太多,没有足够的空间分配java栈了.一个线程的java栈大小由-Xss参数设置
2. 堆内存溢出
	java堆中存放对象实例的空间不足,通过-Xmx设置最大值.
3. 方法区和运行时常量池溢出
	`-XX:permSize=10M -XX:MaxPermSize=10M`
4. 本机直接内存溢出
	`-XX:MaxDirectMemorySize`

### 内存模型
根据某种特定的操作协议,对特定的内存或者高速缓存进行读写访问的过程抽象
1. 内存模型
为了保证共享内存的正确性（可见性、有序性、原子性），内存模型定义了共享内存系统中多线程程序读写操作行为的规范。通过这些规则来规范对内存的读写操作，从而保证指令执行的正确性。它与处理器有关、与缓存有关、与并发有关、与编译器也有关。他解决了CPU多级缓存、处理器优化、指令重排等导致的内存访问问题，保证了并发场景下的一致性、原子性和有序性。
2. java内存模型
Java内存模型（Java Memory Model ,JMM）就是一种符合内存模型规范的，屏蔽了各种硬件和操作系统的访问差异的，保证了Java程序在各种平台下对内存的访问都能保证效果一致的机制及规范。
3. 8中原子操作:
	* lock:作用于主内存的变量,将一个变量标记为一条线程独占
	* unlock:主内存,施放线程独占的变量
	* read:主内存,将变量值从主内存传输到线程的工作内存,之后要使用load操作
	* load:作用于工作内存,将从主内存中read得到的变量值放入工作内存的变量副本中
	* use:工作内存,将工作内存中一个变量的值传递给执行引擎
	* assign:工作内存,将从执行引擎接收到的值赋值给工作内存中的变量
	* store:工作内存,将工作内存中的变量值从工作内存传送到主内存,供之后的write操作使用 
	* write:主内存,包store操作从工作内存中取得的值放入主内存的变量中



## 垃圾回收(GC)
分为堆(heap)和非堆(non-heap)
简单说,堆就是给开发人员使用的代码可及的存放类实例和数组的内存区域;非堆就是除了堆之外的留给JVM自己使用的内存区域.
### 方法区的变动
1. (JDK1.8)移除了永久带,类元信息被放到metaSpace(本地化的内存?),一定程度上解决原来运行时加载大量类信息而引起的Full GC的问题,如反射,动态代理
2. 将常量池(JDK1.7中就移了)和静态变量放到了java堆中

### 运行时数据区
{% asset_img java虚拟机运行时数据区.png java虚拟机运行时数据区%}
1. 程序计数器：线程私有。是一块较小的内存，是当前线程所执行的字节码的行号指示器。是Java虚拟机规范中唯一没有规定OOM（OutOfMemoryError）的区域。
2. Java栈：线程私有。生命周期和线程相同。是Java方法执行的内存模型。执行每个方法都会创建一个栈帧，用于存储局部变量和操作数（对象引用）。局部变量所需要的内存空间大小在编译期间完成分配。所以栈帧的大小不会改变。存在两种异常情况：若线程请求深度大于栈的深度，抛StackOverflowError。若栈在动态扩展时无法请求足够内存，抛OOM。
3. Java堆：所有线程共享。虚拟机启动时创建。存放对象实力和数组。所占内存最大。分为新生代（Young区），老年代（Old区）。新生代分Eden区，Servior区。Servior区又分为From space区和To Space区。Eden区和Servior区的内存比为8:1。 当扩展内存大于可用内存，抛OOM。
4. 方法区：所有线程共享。用于存储已被虚拟机加载的类信息、常量、静态变量等数据。又称为非堆（Non–Heap）。方法区又称“永久代”(JDK1.8中已经移除)。GC很少在这个区域进行，但不代表不会回收。这个区域回收目标主要是针对常量池的回收和对类型的卸载。当内存申请大于实际可用内存，抛OOM。
5. 本地方法栈：线程私有。与Java栈类似，但是不是为Java方法（字节码）服务，而是为本地非Java方法服务。也会抛StackOverflowError和OOM。

### 堆内存结构
{% asset_img 堆空间.jpg 堆空间%}
JVM堆内存分为2块：Permanent Space 和 Heap Space
* Permanent 即 持久代（Permanent Generation），主要存放的是Java类定义信息，与垃圾收集器要收集的Java对象关系不大。
* Heap = { Old + NEW(Eden, from, to) }，Old 即 年老代（Old Generation），New 即 年轻代（Young Generation）。年老代和年轻代的划分对垃圾收集影响比较大。
	1. 年轻代
	所有新生成的对象首先都是放在年轻代。年轻代的目标就是尽可能快速的收集掉那些生命周期短的对象。年轻代一般分3个区，1个Eden区，2个Survivor区（from 和 to）。
	大部分对象在Eden区中生成。当Eden区满时，还存活的对象将被复制到Survivor区（两个中的一个），当一个Survivor区满时，此区的存活对象将被复制到另外一个Survivor区，当另一个Survivor区也满了的时候，从前一个Survivor区复制过来的并且此时还存活的对象，将可能被复制到年老代。
	2个Survivor区是对称的，没有先后关系，所以同一个Survivor区中可能同时存在从Eden区复制过来对象，和从另一个Survivor区复制过来的对象；而复制到年老区的只有从另一个Survivor区过来的对象。而且，因为需要交换的原因，Survivor区至少有一个是空的。特殊的情况下，根据程序需要，Survivor区是可以配置为多个的（多于2个），这样可以增加对象在年轻代中的存在时间，减少被放到年老代的可能。
	针对年轻代的垃圾回收即 Young GC。
	2. 年老代
	在年轻代中经历了N次（可配置）垃圾回收后仍然存活的对象，就会被复制到年老代中。因此，可以认为年老代中存放的都是一些生命周期较长的对象。
	针对年老代的垃圾回收即 Full GC。
	3. 持久代
	用于存放静态类型数据，如 Java Class, Method 等。持久代对垃圾回收没有显著影响。但是有些应用可能动态生成或调用一些Class，例如 Hibernate CGLib 等，在这种时候往往需要设置一个比较大的持久代空间来存放这些运行过程中动态增加的类型。



### 什么时候回收
触发GC的条件,JVM卸载类的判断条件:
根据Eden区和From Space区的内存大小来决定。当内存大小不足时，则会启动GC线程并停止应用线程。
1. Minor GC
当Eden区满时，触发
2. Full GC
	1. 调用System.gc时，系统建议执行Full GC，但是不必然执行
	2. 老年代空间不足
	3. 方法去空间不足
	4. 通过Minor GC后进入老年代的平均大小大于老年代的可用内存
	5. 由Eden区、From Space区向To Space区复制时，对象大小大于To Space可用内存，则把该对象转存到老年代，且老年代的可用内存小于该对象大小

### 回收对象 
通过可达性分析法无法搜索到的对象和可以搜索到的对象。对于搜索不到的方法进行标记。
1. 该类的所有实例都已经被回收,也就是java堆中不存在该类的任何实例
2. 加载该类的ClassLoader已经被回收
3. 该类对应的java.lang.Class对象没有在任何地方呗引用,无法通过反射在任何地方访问该类的方法

### 怎么回收
对于可以搜索到的对象进行复制操作，对于搜索不到的对象，调用finalize()方法进行释放。当GC线程启动时，会通过可达性分析法把Eden区和From Space区的存活对象复制到To Space区，然后把Eden Space和From Space区的对象释放掉。当GC轮训扫描To Space区一定次数后，把依然存活的对象复制到老年代，然后释放To Space区的对象。

### 引用的分类
1. 强引用
	类似`Object obj = new Object()`这类,只要强引用在,永远不会被回收
2. 软引用
	有用但是非必须的,只有内存不够的时候会被回收,如果回收之后还是不够才会抛出内存溢出异常
3. 弱引用
	只能存活到下一次垃圾回收
4. 虚引用
	唯一目的就是在一个关联了虚引用的对象被回收的时候收到系统通知

### 判断对象是否存活的方法
1. 引用计数法
	当对象被引用的时候计数加一,不用的时候减一,缺点是无法解决对象之间相互循环引用的问题,比如两个对象中的成员变量相互引用的时候
2. 可达性分析算法
	当一个对象到GC Roots没有任何引用链相连的时候,就是该对象不可用
	GC Roots包括:
		1. 虚拟机栈(栈帧中的本地变量表)中的引用对象
		2. 方法区中的类的静态引用属性
		3. 方法区中的常量引用的对象
		4. 本地方法栈中JNI(即一般说的Native方法)引用的对象

### 回收方法区(HotSpot中的永久带)
回收性价比比较低,主要回收弃用常量和无用的类
1. 判断弃用常量
	比如字面量"abc",当没有String对象叫"abc"的时候
2. 判断无用的类(同时满足以下三个条件)
	1. 当java堆中没有该类的实例
	2. 加载该类的ClassLoader被回收
	3. 该类的java.lang.Class对象没有在任何地方被引用,也就是无法在任何地方通过反射访问该类的方法

### 垃圾回收算法
1. 标记清除算法
	不足:标记和清除的效率不高,然后就是清除之后会产生大量不连续的内存碎片
2. 复制算法
	直接将内存一分为二,当其中一块内存用完了,将其中还存活的对象复制到另一块内存上,比较浪费空间.
3. 复制收集算法(回收新生代)
	将内存分为一块Eden空间和两块Survivor空间,8:1:1,回收时将Eden和Survivor中还存活的对象复制到另一块Survivor空间中,如果Survivor空间内存不够,就需要用到老年代
4. 标记整理算法(回收老年代)
5. 分代收集算法
	一般将java堆分成新生代和老年代,再使用合适的回收算法

### 垃圾收集器(书上列举了七种)
1. Serial收集器
	单线程收集器,垃圾回收时必须暂停其他所有的工作线程
2. ParNew收集器

### 垃圾回收器


## JVM性能调优的大致步骤
参考地址:https://blog.csdn.net/rickyit/article/details/53895060
参考地址(强烈推荐):https://blog.csdn.net/kthq/article/details/8618052
1. 年轻代大小选择
	响应时间优先的应用 ：尽可能设大，直到接近系统的最低响应时间限制 （根据实际情况选择）。在此种情况下，年轻代收集发生的频率也是最小的。同时，减少到达年老代的对象。
	吞吐量优先的应用 ：尽可能的设置大，可能到达Gbit的程度。因为对响应时间没有要求，垃圾收集可以并行进行，一般适合8CPU以上的应用。
2. 年老代大小选择
	1. 响应时间优先的应用 ：
	年老代使用并发收集器，所以其大小需要小心设置，一般要考虑并发会话率 和会话持续时间 等一些参数。如果堆设置小了，可以会造成内存碎片、高回收频率以及应用暂停而使用传统的标记清除方式；如果堆大了，则需要较长的收集时间。最优化的方案，一般需要参考以下数据获得：
		* 并发垃圾收集信息
		* 持久代并发收集次数
		* 传统GC信息
		* 花在年轻代和年老代回收上的时间比例
	减少年轻代和年老代花费的时间，一般会提高应用的效率
	2. 吞吐量优先的应用 ：
	一般吞吐量优先的应用都有一个很大的年轻代和一个较小的年老代。原因是，这样可以尽可能回收掉大部分短期对象，减少中期的对象，而年老代尽存放长期存活对象。
3. 较小堆引起的碎片问题
	因为年老代的并发收集器使用标记、清除算法，所以不会对堆进行压缩。当收集器回收时，他会把相邻的空间进行合并，这样可以分配给较大的对象。但是，当堆空间较小时，运行一段时间以后，就会出现“碎片”，如果并发收集器找不到足够的空间，那么并发收集器将会停止，然后使用传统的标记、清除方式进行回收。如果出现“碎片”，可能需要进行如下配置：
		-XX:+UseCMSCompactAtFullCollection ：使用并发收集器时，开启对年老代的压缩。
		-XX:CMSFullGCsBeforeCompaction=0 ：上面配置开启的情况下，这里设置多少次Full GC后，对年老代进行压缩

* 回收器选择
JVM给了三种选择：*串行收集器*、*并行收集器*、*并发收集器* ，但是串行收集器只适用于小数据量的情况，所以这里的选择主要针对并行收集器和并发收集器。默认情况下，JDK5.0以前都是使用串行收集器，如果想使用其他收集器需要在启动时加入相应参数。JDK5.0以后，JVM会根据当前系统配置进行判断。
	1. 吞吐量优先的 并行收集器
	如上文所述，并行收集器主要以到达一定的吞吐量为目标，适用于科学技术和后台处理等。
	典型配置 ：`java -Xmx3800m -Xms3800m -Xmn2g -Xss128k -XX:+UseParallelGC -XX:ParallelGCThreads=20`
	-XX:+UseParallelGC ：选择垃圾收集器为并行收集器。 此配置仅对年轻代有效。即上述配置下，年轻代使用并发收集，而年老代仍旧使用串行收集。
	-XX:ParallelGCThreads=20 ：配置并行收集器的线程数，即：同时多少个线程一起进行垃圾回收。此值最好配置与处理器数目相等。
	java -Xmx3550m -Xms3550m -Xmn2g -Xss128k -XX:+UseParallelGC -XX:ParallelGCThreads=20 -XX:+UseParallelOldGC
	-XX:+UseParallelOldGC ：配置年老代垃圾收集方式为并行收集。JDK6.0支持对年老代并行收集。
	java -Xmx3550m -Xms3550m -Xmn2g -Xss128k -XX:+UseParallelGC -XX:MaxGCPauseMillis=100
	-XX:MaxGCPauseMillis=100 : 设置每次年轻代垃圾回收的最长时间，如果无法满足此时间，JVM会自动调整年轻代大小，以满足此值。
	java -Xmx3550m -Xms3550m -Xmn2g -Xss128k -XX:+UseParallelGC -XX:MaxGCPauseMillis=100 -XX:+UseAdaptiveSizePolicy
	-XX:+UseAdaptiveSizePolicy ：设置此选项后，并行收集器会自动选择年轻代区大小和相应的Survivor区比例，以达到目标系统规定的最低相应时间或者收集频率等，此值建议使用并行收集器时，一直打开。
	2. 响应时间优先的 并发收集器
	如上文所述，并发收集器主要是保证系统的响应时间，减少垃圾收集时的停顿时间。适用于应用服务器、电信领域等。
	典型配置 ：`java -Xmx3550m -Xms3550m -Xmn2g -Xss128k -XX:ParallelGCThreads=20 -XX:+UseConcMarkSweepGC -XX:+UseParNewGC`
	-XX:+UseConcMarkSweepGC ：设置年老代为并发收集。测试中配置这个以后，-XX:NewRatio=4的配置失效了，原因不明。所以，此时年轻代大小最好用-Xmn设置。
	-XX:+UseParNewGC :设置年轻代为并行收集。可与CMS收集同时使用。JDK5.0以上，JVM会根据系统配置自行设置，所以无需再设置此值。
	java -Xmx3550m -Xms3550m -Xmn2g -Xss128k -XX:+UseConcMarkSweepGC -XX:CMSFullGCsBeforeCompaction=5 -XX:+UseCMSCompactAtFullCollection
	-XX:CMSFullGCsBeforeCompaction ：由于并发收集器不对内存空间进行压缩、整理，所以运行一段时间以后会产生“碎片”，使得运行效率降低。此值设置运行多少次GC以后对内存空间进行压缩、整理。
	-XX:+UseCMSCompactAtFullCollection ：打开对年老代的压缩。可能会影响性能，但是可以消除碎片

* 堆大小的典型设置：
        1. `java -Xmx3550m -Xms3550m -Xmn2g -Xss128k`
```
-Xmx3550m ：
设置JVM最大可用内存为3550M。
-Xms3550m ：
设置JVM初始内存为3550m。此值可以设置与-Xmx相同，以避免每次垃圾回收完成后JVM重新分配内存。
-Xmn2g ：
设置年轻代大小为2G。整个堆大小 = 年轻代大小 + 年老代大小 + 持久代大小 。持久代一般固定大小为64m，所以增大年轻代后，将会减小年老代大小。此值对系统性能影响较大，Sun官方推荐配置为整个堆的3/8。
-Xss128k ：
设置每个线程的堆栈大小。JDK5.0以后每个线程堆栈大小为1M，以前每个线程堆栈大小为256K。更具应用的线程所需内存大小进行调整。在相同物理内存下，减小这个值能生成更多的线程。但是操作系统对一个进程内的线程数还是有限制的，不能无限生成，经验值在3000~5000左右。
```
        2. `java -Xmx3550m -Xms3550m -Xss128k -XX:NewRatio=4 -XX:SurvivorRatio=4 -XX:MaxPermSize=16m`
```
-XX:MaxTenuringThreshold=0
-XX:NewRatio=4 :
设置年轻代（包括Eden和两个Survivor区）与年老代的比值（除去持久代）。设置为4，则年轻代与年老代所占比值为1：4，年轻代占整个堆栈的1/5
-XX:SurvivorRatio=4 ：
设置年轻代中Eden区与Survivor区的大小比值。设置为4，则两个Survivor区与一个Eden区的比值为2:4，一个Survivor区占整个年轻代的1/6
-XX:MaxPermSize=16m :
设置持久代大小为16m。
-XX:MaxTenuringThreshold=0 ：
设置垃圾最大年龄。如果设置为0的话，则年轻代对象不经过Survivor区，直接进入年老代 。对于年老代比较多的应用，可以提高效率。如果将此值设置为一个较大值，则年轻代对象会在Survivor区进行多次复制，这样可以增加对象再年轻代的存活时间 ，增加在年轻代即被回收的概论。
```


## 逃逸分析
参考地址:http://www.hollischuang.com/archives/2398
类实例和数组不一定都是分配在堆上了

### 概念
逃逸分析的基本行为就是分析对象动态作用域：当一个对象在方法中被定义后，它可能被外部方法所引用，例如作为调用参数传递到其他地方中，称为方法逃逸。
```java
public static StringBuffer craeteStringBuffer(String s1, String s2) {
    StringBuffer sb = new StringBuffer();
    sb.append(s1);
    sb.append(s2);
    return sb;
}
```
StringBuffer sb是一个方法内部变量，上述代码中直接将sb返回，这样这个StringBuffer有可能被其他方法所改变，这样它的作用域就不只是在方法内部，虽然它是一个局部变量，称其逃逸到了方法外部。甚至还有可能被外部线程访问到，譬如赋值给类变量或可以在其他线程中访问的实例变量，称为线程逃逸。
使用逃逸分析，编译器可以对代码做如下优化：
1. 同步省略。如果一个对象被发现只能从一个线程被访问到，那么对于这个对象的操作可以不考虑同步。
2. 将堆分配转化为栈分配。如果一个对象在子程序中被分配，要使指向该对象的指针永远不会逃逸，对象可能是栈分配的候选，而不是堆分配。
3. 分离对象或标量替换。有的对象可能不需要作为一个连续的内存结构存在也可以被访问到，那么对象的部分（或全部）可以不存储在内存，而是存储在CPU寄存器中。
`-XX:+DoEscapeAnalysis ： 表示开启逃逸分析 -XX:-DoEscapeAnalysis ： 表示关闭逃逸分析 从jdk 1.7开始已经默认开始逃逸分析，如需关闭，需要指定-XX:-DoEscapeAnalysis`

### 实验
看以下代码:
```java
public static void main(String[] args) {
    long a1 = System.currentTimeMillis();
    for (int i = 0; i < 1000000; i++) {
        alloc();
    }
    // 查看执行时间
    long a2 = System.currentTimeMillis();
    System.out.println("cost " + (a2 - a1) + " ms");
    // 为了方便查看堆内存中对象个数，线程sleep
    try {
        Thread.sleep(100000);
    } catch (InterruptedException e1) {
        e1.printStackTrace();
    }
}

private static void alloc() {
    User user = new User();
}

static class User {

}
```
1. 关闭JIT编译器逃逸分析
`-Xmx4G -Xms4G -XX:-DoEscapeAnalysis -XX:+PrintGCDetails -XX:+HeapDumpOnOutOfMemoryError `
2. 然后在终端中查看堆中的对象数量:100万个
```java
➜  ~ jps
2809 StackAllocTest
2810 Jps
➜  ~ jmap -histo 2809

 num     #instances         #bytes  class name
----------------------------------------------
   1:           524       87282184  [I
   2:       1000000       16000000  StackAllocTest$User
   3:          6806        2093136  [B
   4:          8006        1320872  [C
   5:          4188         100512  java.lang.String
   6:           581          66304  java.lang.Class
```
3. 打开逃逸分析,只有8万多
`-Xmx4G -Xms4G -XX:+DoEscapeAnalysis -XX:+PrintGCDetails -XX:+HeapDumpOnOutOfMemoryError `
```java 
➜  ~ jps
709
2858 Launcher
2859 StackAllocTest
2860 Jps
➜  ~ jmap -histo 2859

 num     #instances         #bytes  class name
----------------------------------------------
   1:           524      101944280  [I
   2:          6806        2093136  [B
   3:         83619        1337904  StackAllocTest$User
   4:          8006        1320872  [C
   5:          4188         100512  java.lang.String
   6:           581          66304  java.lang.Class
```



