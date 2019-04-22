---
title: toBeTopJavaer-基础篇
categories:
  - 笔记
tags:
  - null
date: 2019-04-19 19:46:02
---
 整体框架参考来源:https://github.com/hollischuang/toBeTopJavaer
 hollis推荐书目:
《深入理解Java虚拟机》 
《Effective Java》 
《深入分析Java Web技术内幕》 
《大型网站技术架构》 
《代码整洁之道》 
《架构整洁之道》 
《Head First设计模式》 
《maven实战》 
《区块链原理、设计与应用》 
《Java并发编程实战》 
《鸟哥的Linux私房菜》 
《从Paxos到Zookeeper》 
《架构即未来》
 <!-- more -->

## 一 基础篇

### 面向对象

#### 什么是面向对象

* 面向对象、面向过程

* 面向对象的三大基本特征和五大基本原则

* Object 
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


#### 平台无关性

[Java如何实现的平台无关性的](/basics/java-basic/platform-independent.md)

[JVM还支持哪些语言（Kotlin、Groovy、JRuby、Jython、Scala）](/basics/java-basic/jvm-language.md)

#### 值传递
* 值调用和引用调用
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

* 为什么说Java中只有值传递

* 关于new对象的位置很有讲究
如果在循环的外面new的话,相当于add进list中的都是同一个对象
```java 
    //代码段
    String[] fieldValues ;//千万不能在这里new ,不然后面add进list的全是这个地址= new String[fieldNames.length];
    if (cursor.moveToFirst()) {
        do {
            fieldValues = new String[fieldNames.length];
            list.add(fieldValues);
        } while (cursor.moveToNext());
    }
    cursor.close();
```

#### 封装、继承、多态

[什么是多态](/basics/java-basic/polymorphism.md)、[方法重写与重载](/basics/java-basic/overloading-vs-overriding.md)

Java的继承与实现

[Java的继承与组合](/basics/java-basic/inheritance-composition.md)

[构造函数与默认构造函数](/basics/java-basic/constructor.md)

[类变量、成员变量和局部变量](/basics/java-basic/variable.md)

[成员变量和方法作用域](/basics/java-basic/scope.md)

### Java基础知识
#### 类关系
{% asset_img 类图关系.png %}

#### 基本数据类型

[7种基本数据类型：整型、浮点型、布尔型、字符型](/basics/java-basic/basic-data-types.md)

[整型中byte、short、int、long的取值范围](/basics/java-basic/integer-scope.md)

[什么是浮点型？](/basics/java-basic/float.md)

[什么是单精度和双精度？](/basics/java-basic/single-double-float.md)

[为什么不能用浮点型表示金额？](/basics/java-basic/float-amount.md)

* 细节问题
```c
// 代码段 
int i = 0;
i = i++;
// 结果i还是0  

//代码段
int i = 0;
i = ++ i;
// 结果i是1 

//代码段
int a = 9;
int b = 4;
double c = a/b;
// 结果是2.0 

// 控制sout的小数输出位数 
sout("%.2f",doubleNum);
// 结果输出的double数字会保留两位小数 
```

#### 自动拆装箱

* Integer.valueOf 

>参考地址:https://blog.csdn.net/wangshihui512/article/details/50960720

```java 
Integer i01=59; //调用了Integer.valueOf()方法,返回新的Integer对象
int i02=59;//基本类型存在栈中
Integer i03=Integer.valueOf(59);//直接返回引用
Integer i04=new Integer(59);//返回新对象
System.out.println(i01==i02);//true 比较的是值
System.out.println(i01==i03);//true 比较的是引用,如果值大于127,或者小于-128,会是false
System.out.println(i01==i04);//false
System.out.println(i02==i02);//true
```

```java 
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
} 
```

* 什么是包装类型、什么是基本类型、什么是自动拆装箱]

* Integer的缓存机制

#### String

* 字符串的不可变性

* JDK 6和JDK 7中substring的原理及区别

* replaceFirst、replaceAll、replace区别、

* String对“+”的重载

* 字符串拼接的几种方式和区别

* String.valueOf和Integer.toString的区别

* switch对String的支持

* 字符串池、常量池（运行时常量池、Class常量池）、intern
String什么时候会进入常量池,可以使用`javap -verbose Test.class`反编译试试
```c
public class Test{
    public static String a = "a";//放入常量池中
    public static String str = "ni" + "hap";//将结果nihao放入常量池中
    public static void main(){
        String b = "b";//放入常量池中

        String string1 = "ni";  
        String string2 = "hao";  
        String string3 = string1+string2;  //将ni和hao分别放入常量池中,而不是结果nihao
        String string4 = string1+"C";   //将C放入常量池中

        String str = "hao";//放入常量池,是一个字符串常量
        String str2 = new String("ni");
        String str3 = new String("hao");//放入堆中,
        System.out.println(str==str3);// 运行后结果为false

		String str4 = str3.intern();
        System.out.println(str==str4);// 运行后结果为true,通过intern方法返回的引用指向常量池中的hao
    }
}
```

* 关于String的引用
```java 
public static void main(String[] args) {
    String x = new String("ab");
    //String x = "ab";
    change(x);
    System.out.println(x);//输出ab
}
 
public static void change(String x) {
    x = "cd";
} 
```
解释:
1. x保存的是"ab"的地址,x的有自己的地址
2. change(String x)方法中的x也有自己的地址,和1中的x不同,但是他们保存的引用地址相同,只不过方法中将这个引用地址的值换成了"cd"的地址
3. 而1中的x中保存的引用地址不变
{% asset_img 值传递.jpeg %}


	```c
void change(string &x) {
    x = "cd";
}
int main(){
    string x = "ab";
    change(x);
    cout << x << endl;//输出cd
}
	```

解释:这里的`&x`直接将变量x的地址传递,而不是传递x保存的地址引用
{% asset_img 引用传递.jpeg %}

	```java
	class Apple {
	    String name;
	    public Apple(String name) {
	        this.name = name;
	    }
	    public static void main(String[] args) {
	        Apple apple = new Apple("6s");
	        change(apple);
	        System.out.println(apple.name);//输出xr
	    }
	    public static void change(Apple apple) {
	        apple.name = "xr";
	        //apple = new Apple("xr");//如果是这样输出的还是6s
	    }
	}
	```
* String,StringBuilder和StringBuffer
1. String对象的每次赋值都相当于重新生成一个新对象,所以String对象频繁变化会导致内存中无引用的字符串多了以后,会引起JVM的GC
2. StringBuilder底层是基于char数组实现的,append()方法基于System.arraycopy()方法实现
3. StringBuffer就是线程安全的StringBuilder


#### 熟悉Java中各种关键字

transient、instanceof、volatile、synchronized、final、static、const 原理及用法。


* 常用集合类
{% asset_link 常用集合类.md 常用集合类%}

* ArrayList和LinkedList和Vector的区别
1. 不考虑同步的情况下
    1. 随机访问数据,ArrayList对象要远优于LinkedList对象；
    2. 进行插入或者删除操作，LinkedList对象要远优于ArrayList对象；
2. 考虑同步的时候只能使用Vector,而Vector和ArrayList相比增加了同步操作,性能上比不上ArrayList

* SynchronizedList和Vector的区别、

* HashMap、HashTable、ConcurrentHashMap区别、

* Set和List区别,Set如何保证元素不重复？

* Java 8中stream相关用法

* apache集合处理工具类的使用

* 不同版本的JDK中HashMap的实现的区别以及原因

* Collection和Collections区别

* Arrays.asList获得的List使用时需要注意什么

* Enumeration和Iterator区别

* fail-fast 和 fail-safe

* CopyOnWriteArrayList、ConcurrentSkipListMap

#### 枚举

>参考地址:http://www.hollischuang.com/archives/195

* 枚举的用法、枚举的实现、枚举与单例、Enum类
1. 用法

	```java
	public enum Color {  
	    RED("红色", 1), GREEN("绿色", 2), BLANK("白色", 3), YELLO("黄色", 4);  
	    // 成员变量  
	    private String name;  
	    private int index;  
	    // 构造方法  
	    private Color(String name, int index) {  
	        this.name = name;  
	        this.index = index;  
	    }  
	    // 普通方法  
	    public static String getName(int index) {  
	        for (Color c : Color.values()) {  
	            if (c.getIndex() == index) {  
	                return c.name;  
	            }  
	        }  
	        return null;  
	    }  
	    // get set 方法  
	    public String getName() {  
	        return name;  
	    }  
	    public void setName(String name) {  
	        this.name = name;  
	    }  
	    public int getIndex() {  
	        return index;  
	    }  
	    public void setIndex(int index) {  
	        this.index = index;  
	    }  
	}  
	```
2. 反编译结果

	```java
	public enum Season {
	    SPRING,SUMMER,AUTUMN,WINTER;
	} 
	```
	```java 
	public final class Season extends Enum
	{
	    //省略部分内容
	    public static final Season SPRING;
	    public static final Season SUMMER;
	    public static final Season AUTUMN;
	    public static final Season WINTER;
	    private static final Season ENUM$VALUES[];
	    static
	    {
	        SPRING = new Season("SPRING", 0);
	        SUMMER = new Season("SUMMER", 1);
	        AUTUMN = new Season("AUTUMN", 2);
	        WINTER = new Season("WINTER", 3);
	        ENUM$VALUES = (new Season[] {
	            SPRING, SUMMER, AUTUMN, WINTER
	        });
	    }
	}
	```


* Java枚举如何比较

* switch对枚举的支持

* 枚举的序列化如何实现

* 枚举的线程安全性问题

#### IO

* 字符流、字节流、输入流、输出流、

* 同步、异步、阻塞、非阻塞、Linux 5种IO模型
**同步和异步说的是消息的通知机制,阻塞和非阻塞说的是线程的状态**

> 参考地址:https://blog.csdn.net/shimiso/article/details/24990499

1. 同步阻塞io
服务器实现是一个连接创建一个线程,直到连接断开这个线程都被这个连接占用
指的就是传统的Java IO,线程发起io后会阻塞,等到io结束之后才会恢复
同步阻塞时序图
{% asset_img 同步阻塞时序图.png 同步阻塞时序图%}
2. 同步非阻塞io(消息机制会阻塞,io不会阻塞)
服务器实现是一个请求创建一个线程,或者说多个连接对应一个线程,由于线程数量小于连接数量,所以IO操作不能够阻塞
Java中的NIO(Non-blocking IO)是这种模型,线程发起io后可以去干其他事情,但是另外起一个线程去监听或者轮询io状态(是否有可读数据,或者是否可以写入);传统io中,比如read()操作,当没有数据可读的时候线程就会阻塞;但是在NIO中,没有数据可读的时候read()就会返回立即0
    1. 有一个专门的线程来处理所有的IO事件,并负责分发
    2. 事件触发机制:事件到来的时候触发,而不是同步监视事件
    3. 线程通讯：线程之间通过 wait,notify 等方式通讯。保证每次上下文切换都是有意义的。减少无谓的线程切换。
{% asset_img 同步非阻塞时序图.png 同步非阻塞时序图%}

Java中的NIO(Non-blocking IO)是这种模型,线程发起io后可以去干其他事情,但是另外起一个线程去监听或者轮询io状态(是否有可读数据,或者是否可以写入)
```java 
//创建Selector监听IO事件,
Selector selector = Selector.open();  

//for循环中,为每一个监听的port打开一个channel,并设置成非阻塞
ServerSocketChannel ssc = ServerSocketChannel.open();
ssc.configureBlocking( false );
 
ServerSocket ss = ssc.socket();
InetSocketAddress address = new InetSocketAddress( ports[i] );
ss.bind( address );

SelectionKey key = ssc.register( selector, SelectionKey.OP_ACCEPT );//将channel注册到selector

//while循环
while(ture){
	//该方法会阻塞,直到至少有一个已经注册的事件发生时,会返回发生事件的数量
	int num = selector.select();
 	//事件发生后就可以获得SelectionKey集合,然后进行迭代处理
	Set selectedKeys = selector.selectedKeys();
	Iterator it = selectedKeys.iterator();
	 
	while (it.hasNext()) {
	     SelectionKey key = (SelectionKey)it.next();
	     // ... deal with I/O event ...
	     if ((key.readyOps() & SelectionKey.OP_ACCEPT)
                        == SelectionKey.OP_ACCEPT) {
            // Accept the new connection
         } else if ((key.readyOps() & SelectionKey.OP_READ)
                == SelectionKey.OP_READ) {
            // Read the data
		 }
	}
}
```

3. 异步非阻塞io
服务器实现是一个有效请求创建一个线程
Java中的AIO((Asynchronous IO))是这种模型,线程发起io后可以直接干其他事前,系统会监听io状态,IO完成后会通知线程
{% asset_img 异步非阻塞时序图.png 异步非阻塞时序图%}

* BIO、NIO和AIO的区别、三种IO的用法与原理、netty
1. BIO
2. NIO 

>参考地址:参考地址:https://www.ibm.com/developerworks/cn/education/java/j-nio/j-nio.html

	```java
	public class FastCopyFile {
	    static public void main(String args[]) throws Exception {
	        if (args.length < 2) {
	            System.err.println("Usage: java FastCopyFile infile outfile");
	            System.exit(1);
	        }
	        String infile = args[0];
	        String outfile = args[1];
	        FileInputStream fin = new FileInputStream(infile);
	        FileOutputStream fout = new FileOutputStream(outfile);
	        FileChannel fcin = fin.getChannel();
	        FileChannel fcout = fout.getChannel();
	        ByteBuffer buffer = ByteBuffer.allocateDirect(1024);
	        while (true) {
	            //设置缓存区,使之可以读入新数据
	            //position = 0;limit = capacity;mark = -1;
	            buffer.clear();
	            //将输入通道中的数据读到缓存区
	            int r = fcin.read(buffer);
	            //判断输入通道的数据是不是读完了
	            if (r == -1) {
	                break;
	            }
	            //设置缓存区,然缓存区中的数据可以写入新的通道
	            //limit = position;position = 0;mark = -1;
	            buffer.flip();
	            //将数据从缓存区写入输出通道
	            fcout.write(buffer);
	        }
	    }
	}
	```
3. AIO



* 文件读写经典代码
```java 
public void read() {
    BufferedInputStream inputStream = null;
    String path = "/Users/huangning/Desktop/temp/haha";
    int len;
    try {
        inputStream = new BufferedInputStream(new FileInputStream(new File(path)));
        byte[] buffer = new byte[1024];
        while((len = inputStream.read(buffer)) !=- 1){
            System.out.println(new String(buffer, 0, len));
        }
    } catch (IOException e) {
        e.printStackTrace();
    }finally {
        try {
            if (inputStream != null) {
                inputStream.close();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

public void write() {
    BufferedOutputStream outputStream = null;
    try {
        outputStream = new BufferedOutputStream(new FileOutputStream(new File("/Users/huangning/Desktop/temp/haha")));
        String content = "just try it";
        outputStream.write(content.getBytes());
    } catch (IOException e) {
        e.printStackTrace();
    }finally {
        if (outputStream != null) {
            try {
                outputStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

* 读取classpath下的配置文件
```java 
Yaml yaml = new Yaml();
Map map = (Map) yaml.load(APIRequest.class.getClassLoader().getResourceAsStream("application.yml"));
```

#### Java反射与javassist
{% asset_link 反射.md 反射 %}
* 反射与工厂模式

* 反射有什么作用

* Class类

* `java.lang.reflect.*`

#### 动态代理

* 静态代理、动态代理

* 动态代理和反射的关系

* 动态代理的几种实现方式

* AOP

#### 序列化

什么是序列化与反序列化、为什么序列化、序列化底层原理、序列化与单例模式、protobuf、为什么说序列化并不安全

#### 注解
1. 元注解、自定义注解、Java中常用注解使用、注解与反射的结合


1. 注解定义

> 参考地址:https://blog.csdn.net/javazejian/article/details/71860633

```
//注解String类型的字段
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface SQLString {
    //对应数据库表的列名
    String name() default "";
    //列类型分配的长度，如varchar(30)的30
    int value() default 0;
    //嵌套注解
    Constraints constraint() default @Constraints;
}


//约束注解
@Target(ElementType.FIELD)//只能应用在字段上
@Retention(RetentionPolicy.RUNTIME)
public @interface Constraints {
    //判断是否作为主键约束
    boolean primaryKey() default false;
    //判断是否允许为null
    boolean allowNull() default false;
    //判断是否唯一
    boolean unique() default false;
}


```
注解中方法可以看成是注解的属性值,在使用注解的时候传入,等到注解处理器中可以根据注解属性的不同值做不同的处理
2. 注解使用
```
@DBTable(name = "MEMBER")
public class Member {
    //主键ID
    @SQLString(name = "ID",value = 50, constraint = @Constraints(primaryKey = true))
    private String id;

    @SQLString(name = "NAME" , value = 30)
    private String name;

    @SQLInteger(name = "AGE")
    private int age;

    @SQLString(name = "DESCRIPTION" ,value = 150 , constraint = @Constraints(allowNull = true))
    private String description;//个人描述
}
```
在使用注解的时候,根据注解的属性传入相应的值.
3. 注解处理器
```
public class TableCreator {
  public static String createTableSql(String className) throws ClassNotFoundException {
    Class<?> cl = Class.forName(className);
    DBTable dbTable = cl.getAnnotation(DBTable.class);
    //如果没有表注解，直接返回
    if(dbTable == null) {
      System.out.println(
              "No DBTable annotations in class " + className);
      return null;
    }
    String tableName = dbTable.name();
    // If the name is empty, use the Class name:
    if(tableName.length() < 1)
      tableName = cl.getName().toUpperCase();
    List<String> columnDefs = new ArrayList<String>();
    //通过Class类API获取到所有成员字段
    for(Field field : cl.getDeclaredFields()) {
      String columnName = null;
      //获取字段上的注解
      Annotation[] anns = field.getDeclaredAnnotations();
      if(anns.length < 1)
        continue; // Not a db table column

      //判断注解类型
      if(anns[0] instanceof SQLInteger) {
        SQLInteger sInt = (SQLInteger) anns[0];
        //获取字段对应列名称，如果没有就是使用字段名称替代
        if(sInt.name().length() < 1)
          columnName = field.getName().toUpperCase();
        else
          columnName = sInt.name();
        //构建语句
        columnDefs.add(columnName + " INT" +
                getConstraints(sInt.constraint()));
      }
      ...
    }
}

```
注解处理器的基本原理就是通过反射机制,获取类,字段,方法上的注解,然后根据不同的注解信息作出不同的处理,实际编写自定义注解的时候需要继承AbstractProcessor类,然后重写里头几个重要的方法,最后还需要注册到META-INF/services/javax.annotation.processing.Processor文件(自己创建)中.
```
init():对一些工具进行初始化。
process():就是真正生成java代码的地方。
getSupportedAnnotationTypes():表示该注解处理器可以出来那些注解。
getSupportedSourceVersion():可以出来java版本
```

5. Spring常用注解


#### JMS

什么是Java消息服务、JMS消息传送模型

#### JMX

`java.lang.management.*`、 `javax.management.*`

#### 泛型
* 泛型
```java
/** 参数T
 * 第一个:占位符,表示这个T是一个泛型,提示编译器要注意
 * 第二个:表示要返回T类型的数据
 * 第三个:表示限制参数类型为T
 */
private <T> T getListFisrt(List<T> data) {
        if (data == null || data.size() == 0) {
            return null;
        }
        return data.get(0);
    }
```
注意点:
1. 泛型方法和泛型类的<>互不相干
2. 泛型擦除如果泛型参数没有指定上限(如<T extends String>),那类型擦除之后T就被替换成Object,否则就是上限,如String
3. 类型擦除也是为什么泛型类和泛型方法中不接受基本数据类型的原因


* 泛型与继承、类型擦除、泛型中K T V E ？ [object等的含义](/basics/java-basic/k-t-v-e.md)、泛型各种用法

* 限定通配符和非限定通配符、上下界限定符extends 和 super

* List<Object>和原始类型List之间的区别? 

* List<?>和List<Object>之间的区别是什么?

#### 单元测试

junit、mock、mockito、内存数据库（h2）

#### 正则表达式

`java.lang.util.regex.*`

#### 常用的Java工具库

`commons.lang`, `commons.*...` `guava-libraries` `netty`

#### API&SPI

API、API和SPI的关系和区别

如何定义SPI、SPI的实现原理

#### 异常

异常类型、正确处理异常、自定义异常

Error和Exception

异常链、try-with-resources

finally和return的执行顺序

#### 时间处理

时区、冬令时和夏令时、时间戳、Java中时间API

格林威治时间、CET,UTC,GMT,CST几种常见时间的含义和关系

SimpleDateFormat的线程安全性问题

Java 8中的时间处理

如何在东八区的计算机上获取美国时间

#### 编码方式

Unicode、有了Unicode为啥还需要UTF-8

GBK、GB2312、GB18030之间的区别

UTF8、UTF16、UTF32区别

URL编解码、Big Endian和Little Endian

如何解决乱码问题

#### 哈希算法
* 哈希算法
基于哈希算法在信息安全中主要应用在
1. 文件校验
2. 数字签名
3. 鉴权协议

* 一致性Hash算法

> 参考地址:http://blog.csdn.net/cywosp/article/details/23397179

一致性hash算法提出了在动态变化的Cache环境中，判定哈希算法好坏的四个定义:
1. 平衡性
2. 单调性
3. 分散性
4. 负载

#### 语法糖

Java中语法糖原理、解语法糖

语法糖：switch 支持 String 与枚举、泛型、自动装箱与拆箱、方法变长参数、枚举、内部类、条件编译、 断言、数值字面量、for-each、try-with-resource、Lambda表达式、

### 阅读源代码

String、Integer、Long、Enum、BigDecimal、ThreadLocal、ClassLoader & URLClassLoader、ArrayList & LinkedList、 HashMap & LinkedHashMap & TreeMap & CouncurrentHashMap、HashSet & LinkedHashSet & TreeSet

### Java并发编程

#### 并发与并行

什么是并发

什么是并行

并发与并行的区别

#### 线程
{% asset_link Java线程的使用.md 线程的使用%}

* 线程的实现、线程的状态、优先级、线程调度、创建线程的多种方式、守护线程

* 线程与进程的区别

#### 线程池
{% asset_link 线程池的使用.md 线程池的使用}

* 自己设计线程池、submit() 和 execute()、线程池原理

* 为什么不允许使用Executors创建线程池

#### 线程安全
* 线程安全的定义
当多个线程访问某个类时，不管运行时环境采用何种调度方式或者这些线程将如何交替进行，并且在主调代码中不需要任何额外的同步或协同，这个类都能表现出正确的行为，那么称这个类是线程安全的

* 死锁？、死锁如何排查、线程安全和内存模型的关系

* concurrent包
{% post_link concurrent包常用类.md concurrent常用包 %}

#### 锁
{% post_link java中的锁.md java中的锁 %}

CAS、乐观锁与悲观锁、数据库相关锁机制、分布式锁、偏向锁、轻量级锁、重量级锁、monitor、

锁优化、锁消除、锁粗化、自旋锁、可重入锁、阻塞锁、死锁

#### 死锁

死锁的原因

死锁的解决办法

#### synchronized

[synchronized是如何实现的？](/basics/java-basic/synchronized.md)

synchronized和lock之间关系、不使用synchronized如何实现一个线程安全的单例

synchronized和原子性、可见性和有序性之间的关系

#### volatile

happens-before、内存屏障、编译器指令重排和CPU指令重

volatile的实现原理

volatile和原子性、可见性和有序性之间的关系

有了symchronized为什么还需要volatile

#### sleep 和 wait

#### wait 和 notify

#### notify 和 notifyAll

#### ThreadLocal

#### 写一个死锁的程序

#### 写代码来解决生产者消费者问题

### 并发包

#### 阅读源代码，并学会使用

Thread、Runnable、Callable、ReentrantLock、ReentrantReadWriteLock、Atomic*、Semaphore、CountDownLatch、、ConcurrentHashMap、Executors