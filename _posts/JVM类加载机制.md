---
title: JVM类加载机制
categories:
  - 笔记
tags:
  - JVM
  - 类加载机制
date: 2018-07-24 23:19:57
---
 虚拟机把描述类的数据从Class文件加载到内存,并对数据进行校验,转换解析和初始化,最终形成可以被虚拟机直接使用的Java类型,这就是虚拟机的类加载机制.
 <!-- more -->

# 执行顺序
1. 先父类的静态代码块,静态变量(按照源代码顺序),然后是子类的
2. 先父类的非静态成员变量,代码块(按源代码顺序),构造方法(如果构造方法中调用了被子类覆盖的方法,调用的是子类的方法),然后子类的


```java
public class Apple {
    public static int a = 1;
    public int aa = 11;
    public int c = 0;

    public Apple(){
        c = 2;
    }

    {

        aa = 22;
        bb = 22;
        c = 1;
    }

    public int bb = 11;

    static {
        a = 2;
        b = 2;
        System.out.println("a is " + Apple.a);
        System.out.println("b is " + Apple.b);
    }

    public static int b = 1;

    public static void main(String[] args) {
        Apple apple = new Apple();
        System.out.println("a is " + Apple.a);
        System.out.println("b is " + Apple.b);
        System.out.println("aa is " + apple.aa);
        System.out.println("bb is " + apple.bb);
        System.out.println("c is " + apple.c);
    }

}
/* 输出
a is 2
b is 2
a is 2
b is 1
aa is 22
bb is 11
c is 2
*/
```


# 类加载时机
{% asset_img 类加载时机.png 类加载时机 %}
## 5种必须立即对类进行初始化的情况
加载验证准备解析都发生在初始化之前
1. 使用new关键字实例化对象的时候,读取或设置一个类的静态字段(被final修饰,已经在编译期把结果放进常量池的静态字段除外)的时候,调用一个类的静态方法的时候
2. 使用java.lang.reflect包的方法对类进行反射调用的时候,如果类没有进行过初始化
3. 当初始化一个类的时候,发现其父类没有初始化,则会先触发其父类的初始化
4. 虚拟机启动的时候,会先初始化用户指定的主类
5. 使用JDK1.7的动态语言支持时


## 加载
1. 通过一个类的全限定名获取定义此类的二进制字节流。这个时候会使用到类加载器.
上面说获取二进制字节流，而没有明确的说明是class文件中的字节流，因为还有其它获取字节流的方式，例如从jar包中获取、从网络中获取、动态代理运行时生成等。
2. 将这个字节流所代表的静态存储结构转换为方法区的运行时数据结构。
3. 在内存中生成一个代表这个类的java.lang.Class对象，作为方法区这个类的各种数据结构的访问入口。

### 类加载器
1. 类加载器的分类
	1. 启动类加载器(BootstrapClassloader)
	加载`JAVA_HOME/lib`下的类库
	2. 扩展类加载器(ExtensionClassloader)
	加载`JAVA_HOME/libext`下的类库
	3. 应用程序类加载器(ApplicationClassloader)
	加载项目中classpath下的类文件,也被称为系统类加载器

2. 双亲委派模型
{% asset_img 双亲委派模型.png 双亲委派模型%}
当一个类加载器加载类的时候不会马上自己去加载类,而是先尝试使用父类来加载,这种模式比较安全,可以避免某些类被重复加载.Java虚拟机建议是不要破坏这个模型。试想一下，如果我们在自己的项目中定义了一个Object类，和JDK中Object类的包同名，我们程序中使用的是JDK的，但是你的类加载器却加载了自定义的Object，那程序就乱套了，很不安全。

3. 自定义类加载器
```java
public class MyClassLoader extends ClassLoader {
    public MyClassLoader(ClassLoader parent) {
        super(parent);//构造方法需要指定父类
    }

    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {   //重写该方法自定义加载方式
        //加载的是classpath下的文件
        String path = name.replace('.', File.separatorChar) + ".class";
        System.out.println("path is " + path);
        byte[] classData = loadClassData(path);
        if (classData == null) {
            throw new ClassNotFoundException();
        }
        return defineClass(name, classData, 0, classData.length);
    }

    private byte[] loadClassData(String name) {//读取class文件,返回字节数组
        InputStream stream = ClassLoader.getSystemResourceAsStream(name);
        ByteArrayOutputStream out = new ByteArrayOutputStream(1000);
        byte[] data = new byte[1000];
        int n;
        try {
            while ((n = stream.read(data)) != -1) {
                out.write(data, 0, n);
            }
            return out.toByteArray();
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            try {
                out.close();
                if (stream != null) {
                    stream.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return null;
    }

    public static void main(String[] args) {
        MyClassLoader myClassLoader = new MyClassLoader(ClassLoader.getSystemClassLoader().getParent());
        String name = "others.Apple";
        try {
            Object clazz = Class.forName(name, false, myClassLoader).newInstance();
            System.out.println(clazz);
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InstantiationException e) {
            e.printStackTrace();
        }

    }
}
```


## 验证
验证是连接阶段的第一步，这一阶段的目的是为了确保Class文件的字节流中包含的信息符合虚拟机的要求，并且不会危害虚拟机自身的安全。验证阶段大致会完成以下4个阶段的校验动作：文件格式验证、元数据验证、字节码验证和符号引用验证。
1. 文件格式验证(ClassFormatError)
这一阶段目的是验证二进制字节流是否符合Class文件格式的规范，并且能被当前版本的虚拟机处理，包括以下几点:
	* 是否以魔数（0xCAFEBABY）开头。
	* 主次版本号是否在当前虚拟机处理范围之内。
	* 常量池中的常量是否有不支持的常量类型。
	* 指向常量的各种索引值中是否有指向不存在的常量或不符合类型的常量。
	* CONSTANT_Utf8_info 型常量是否有不符合UTF8数据编码的数据。
	* Class文件中各个部分及文件本身是否有被删除的或附加的信息。
这个阶段是基于二进制字节流进行的，只有通过了这个阶段的验证，字节流才会流入方法区中进行存储，后面3个阶段全是基于方法区的存储结构进行的，不会再直接操作字节流。
2. 元数据验证
这一阶段主要对字节码的描述信息进行语义分析，以保证其描述信息符合java语言规范，这阶段的验证点可能包括以下几点：
	* 这个类是否有父类
	* 这个类的父类是否继承了不被允许继承的类(被final 修饰)
	* 如果这个类不是抽象类，是否实现了父类或接口中的方法
	* 类中的字段、方法是否与父类产生矛盾（覆盖父类的final字段值等）
3. 字节码验证
这一阶段目的主要目的是确定程序语义是合法的、符合逻辑的。这个阶段主要对类的字节码进行校验分析，保证该类的方法不会在运行时做出危害虚拟机安全的事：
	* 保证任意时刻操作数栈的数据类型与指令代码序列都能配合工作，例如不会出现操作数栈上 int 类型的数据使用时按long类型加载进本地变量表中。
	* 保证跳转指令不会跳转到方法体以外的字节码指令上。
	* 保证方法体内的类型转化是有效的，可以把一个子类对象赋值给父类数据结构，这是安全的，而不能把父类赋值给子类甚至与它无关系的数据类型，这是危险和不合法的。
4. 符号引用验证
这一阶段用来将符号引用转换为直接引用的时候，这个转化将在解析阶段中发生，符号引用验证可以看做是类对自身以外（常量池中各种符号引用）的信息进行匹配性校验，通常需要校验以下内容：
	* 符号引用中能否根据字符串的权限定名找到对应的类。
	* 在指定类中是否存在符合方法的字段描述符以及简单名称描述的方法和字段。
	* 符号引用中的类、字段、方法的访问性是否可以被当前类访问。

## 准备
准备阶段是正式为**类变量**分配内存并设置初始值的阶段，这些变量所使用的内存都将在方法区分配。实例变量会在对象实例化的时候跟对象一起在java堆中分配。这里的初始值指的是通常情况下的零值。假设一个类变量的定义为:
```java
public static int a = 123;
```
那么变量a初始化的值是0而不是123。如果变量同时是final类型，那么准备阶段就会被赋值为123，不必等到初始化阶段再赋值。

## 解析
解析阶段是将虚拟机常量池内的**符号引用**替换为**直接引用**的过程。解析动作主要针对类或接口、字段、类方法、接口方法、方法类型、方法句柄和调用点限定符7类符号进行。可能大家有疑问Class文件中哪有这么多内容，其实上面也说了，是针对常量池。不管是CLass文件中的方法表还是字段表，不能直接表示的内容，基本都会直接或间接存在常量池中，因此解析过程就是针对常量池中的数据类型进行解析的。
1. 类或接口的解析
要把一个从未解析过的符号引用N解析为一个类或接口的直接引用，虚拟机需要完成以下3个步骤：
	* 如果C不是一个数组类型，那么虚拟机会把代表N的权限定名传递给D的类加载器去加载这个类C。在加载的过程当中，由于元数据、字节码验证的操作，又可能触发其它类的加载动作，一旦出险任何异常，则解析宣告失败。
	* 如果C是一个数组类型，并且数组元素为对象，描述符类似“[Ljava/lang/Integer”的形式按照第一点的规则加载数组元素类型。如果N的描述符如前面所假设的形式，需要加载的类型就是java.lang.Integer，接着由虚拟机生成一个代表次数组维度和元素的数组对象。
	* 如果上面的步骤没有任何异常，那么C在虚拟机中实际上已经称为一个有效的类或接口了。解析之前还要进行符号验证，确认D是否具有对C的访问权限，如果不具备则会抛出异常。
2. 字段解析
对字段表内class_index项中索引的CONSTANT_Class_info符号引用进行解析，也就是字段所属的类或接口的符号引用，如果解析这个类或符号引用的过程中出现任何异常，都会导致字段符号引用解析的失败。如果解析成功，这个字段对应的类或接口用C表示，接下来沿着A和它的父类/父接口寻找是否有这个字段，如果有会进行权限验证，如果不具备权限则抛出异常。如果这个过程不出错，则会在找到符合字段的时候返回这个字段的直接饮用，查找结束。
3. 类(静态)方法解析
类方法解析首先也要首先解析出类方法表class_index项中索引的方法所属的类或接口的符号引用，解析成功用C表示。
	* 类方法和接口方法符号引用的常量类型定义是分开的，如果在类方法表中索引类是个接口，直接抛出异常。
	* 如果通过了第一步，在类C中查找是否有简单名称和描述符都与目标匹配的方法，有则返回这个方法的直接引用，查找结束。
	* 否则在类的父类递归查找是否有这个方法，有则返回直接引用，查找结束。
	* 否则在类的接口列表和父接口递归查找，如果存在匹配的方法，说明类C是一个抽象类，查找结束，抛出异常。
	* 否则宣告查找失败，抛出异常。
最后如果查找成功返回了直接引用，还要对这个方法进行权限验证，如果不具备权限，则会抛出异常。
4. 接口方法解析
接口方法需要先解析出接口方法表的class_index 项中索引的方法所属的类或接口的符号引用。
	* 如果发现class_index 中的索引C是个类而不是接口，直接抛出异常。
	* 否则在接口C中查找是否有描述符和名称都匹配的方法，有则返回方法的直接引用，查找结束。
	* 否则在其父接口中递归查找，匹配就返回方法的直接引用，查找结束。
	* 否则宣告方法查找失败。

## 初始化
类初始化是类加载过程的最后一步。前面的类加载过程中，除了加载阶段可以自定义类加载器干预之外，其余动作完全由虚拟机主导。到了初始化阶段，才真正开始执行java代码。
我们知道，在前面的准备阶段，已经对类变量分配过内存并设置初始值。在初始化阶段，则是为类变量或其它资源设置程序中声明的值。注意这里仍然是类变量，不包括实例变量。或者明确的说，这一阶段，是执行static关键字修饰的变量或代码块。本质上，初始化是执行类构造器`<client>`方法的过程。
`<client>`方法是由编译器自动收集类中所有类变量的赋值动作和静态代码块中的语句合并产生的。编译器收集的顺序是有语句在资源文件中出险的顺序所决定的。
```java
public class Client {
    private static Client client = new Client();

    public static int a;
    public static int b = 0;

    private Client() {
        a++;
        b++;
    }

    public static void main(String[] args) {
        System.out.println("a= " + Client.a);
        System.out.println("b= " + Client.b);
    }

}
```
输出结果为:
```java
a= 1;
b= 0;
```
在类加载的准备阶段会给类变量分配内存和赋初始值。在外部调用Client.getInstance()时，因为之前类没有被加载过，会引发类加载，到了准备阶段就会给类变量赋初始值。赋值顺序同一个类中是按声明的顺序，也就是
```java
client=null；
a=0;
b=0;
```
然后解析完开始初始化，按程序声明的值给类变量赋值。首先执行clinet=new Client(),其实关键就是这里new的过程会调用构造函数，调用完后
```java
a=1;
b=1;
```
接着继续初始化，a只是声明没有赋值，所以没有任何操作，b声明且赋值为0，所以初始化完成后
```java
a=1;
b=0;
```














