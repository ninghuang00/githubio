---
title: java中的String和常量池
categories:
  - 笔记
tags:
  - String
  - 常量池
date: 2018-08-18 16:54:29
---
 这是摘要
 <!-- more -->

### String注意点

### 什么时候会进入常量池
可以使用`javap -verbose Test.class`反编译试试
```java
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