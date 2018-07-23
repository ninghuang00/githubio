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
1. Java常用的数据结构中，请描述Vector, ArrayList, LinkedList的不同场景下的性能差别
    1. 不考虑同步的情况下
        1. 随机访问数据,ArrayList对象要远优于LinkedList对象；
        2. 进行插入或者删除操作，LinkedList对象要远优于ArrayList对象；
    2. 考虑同步的时候只能使用Vector,而Vector和ArrayList相比增加了同步操作,性能上比不上ArrayList

### Object 
1. ==和equals的区别
在Object类中equals()方法其实等效于==,比较的是两个对象的内存地址是否相等
```java
public boolean equals(Object obj) {
    return (this == obj);
}
```
如果重写equals()方法,那么它和==就不一定等效了
2. hashCode()和equals()
    1. hashCode的存在主要是用于查找的快捷性，如Hashtable，HashMap等，hashCode是用来在散列存储结构中确定对象的存储地址的；
    2. 如果两个对象相同，就是适用于equals(java.lang.Object) 方法，那么这两个对象的hashCode一定要相同；
    3. 如果对象的equals方法被重写，那么对象的hashCode也尽量重写，并且产生hashCode使用的对象，一定要和equals方法中使用的一致，否则就会违反上面提到的第2点；
    4. 两个对象的hashCode相同，并不一定表示两个对象就相同，也就是不一定适用于equals(java.lang.Object) 方法，只能够说明这两个对象在散列存储结构中，如Hashtable，他们“存放在同一个篮子里”。
总结一下:hashCode()方法就是用于决定元素存放在结构中(如HashMap等)的位置,然后方便查找的,而在hash值冲突的时候,就需要用到equals()方法来判断到底要找的是哪一个.**所以重写了equals()方法,一般还要重写hashCode()方法**,

### String,StringBuilder和StringBuffer
1. String对象的每次赋值都相当于重新生成一个新对象,所以String对象频繁变化会导致内存中无引用的字符串多了以后,会引起JVM的GC
2. StringBuilder底层是基于char数组实现的,append()方法基于System.arraycopy()方法实现
3. StringBuffer就是线程安全的StringBuilder

### AVL树
十分严格的平衡二叉搜索树,树中任何一个节点的两棵子树高度差不超过1
```java 
public class AVLTree<T extends Comparable<T>> {
    private AVLTreeNode<T> mRoot;    // 根结点
    // AVL树的节点(内部类)
    class AVLTreeNode<T extends Comparable<T>> {
        T key;               // 关键字(键值)
        int height;         // 高度
        AVLTreeNode<T> left;    // 左孩子
        AVLTreeNode<T> right;    // 右孩子
    ......
}
```

### 大根堆

---
## 集合
{% asset_link 常用集合类.md 常用集合类%}


## 哈希算法
基于哈希算法在信息安全中主要应用在
1. 文件校验
2. 数字签名
3. 鉴权协议

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
{% asset_link Java线程的使用.md 线程的使用%}

## 线程池
{% asset_link 线程池的使用.md 线程池的使用}

### CountdownLatch

### CyclicBarrier



## J.U.C

## Atomic

## CAS算法
Compare and Swap，即比较再交换。
参考地址:https://www.jianshu.com/p/21be831e851e


## fork/join 


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