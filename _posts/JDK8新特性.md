---
title: JDK8新特性
categories:
  - 笔记
tags:
  - JDK8
  - lambda表达式
  - stream
  - 函数式编程
date: 2018-07-25 20:12:49
---
 这是摘要
 <!-- more -->

## lambda表达式
将函数作为一个方法的参数进行传递,其实就是原来的用匿名内部类实现接口作为参数传递,换了一种写法.

## 函数式接口
函数式接口(Functional Interface)就是一个有且仅有一个抽象方法，但是可以有多个非抽象方法的接口。
```java
interface Hello{
    void sayHi(String name);

    default void sayBye(String name,String word){
        System.out.println(name + word);
    }
} 
```
函数式接口可以被隐式转换为lambda表达式。
```java
String name = "Jack";
Hello hello = name1 -> System.out.println("hi " + name1);
hello.sayHi(name); 
```

## 函数式编程

## stream

### 常用方法
1. map
```java
List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5);
// 获取对应的平方数
List<Integer> squaresList = numbers.stream().map( i -> i*i).distinct().collect(Collectors.toList());
```
2. filter
```java
List<String> strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
List<String> filtered = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.toList());
```
3. forEach
```java
Random random = new Random();
//limit 方法用于获取指定数量的流
random.ints().limit(10).forEach(System.out::println);
```
4. sorted
```java
Random random = new Random();
random.ints().limit(10).sorted().forEach(System.out::println);
```
5. 转并行
```java
List<String> strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
// 获取空字符串的数量
int count = strings.parallelStream().filter(string -> string.isEmpty()).count();
```
6. Collectors
```java
List<String>strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
List<String> filtered = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.toList());
 
System.out.println("筛选列表: " + filtered);
String mergedString = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.joining(", "));
System.out.println("合并字符串: " + mergedString);
```
7. 统计
```java
List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5);
IntSummaryStatistics stats = integers.stream().mapToInt((x) -> x).summaryStatistics();
System.out.println("列表中最大的数 : " + stats.getMax());
System.out.println("列表中最小的数 : " + stats.getMin());
System.out.println("所有数之和 : " + stats.getSum());
System.out.println("平均数 : " + stats.getAverage());
```




## FP

## 接口默认方法
首先，之前的接口是个双刃剑，好处是面向抽象而不是面向具体编程，**缺陷是，当需要修改接口时候，需要修改全部实现该接口的类**，目前的java 8之前的集合框架没有foreach方法，通常能想到的解决办法是在JDK里给相关的接口添加新的方法及实现。然而，对于已经发布的版本，是没法在给接口添加新方法的同时不影响已有的实现。所以引进的默认方法。他们的目的是为了解决接口的修改与现有的实现不兼容的问题。

## 新的日期API
1. 本地化日期API
```java
import java.time.LocalDate;
import java.time.LocalTime;
import java.time.LocalDateTime;
import java.time.Month;
 
public class Java8Tester {
   public static void main(String args[]){
      Java8Tester java8tester = new Java8Tester();
      java8tester.testLocalDateTime();
   }
    
   public void testLocalDateTime(){
    
      // 获取当前的日期时间
      LocalDateTime currentTime = LocalDateTime.now();
      System.out.println("当前时间: " + currentTime);		//当前时间: 2016-04-15T16:55:48.668
        
      LocalDate date1 = currentTime.toLocalDate();
      System.out.println("date1: " + date1);			//date1: 2016-04-15
        
      Month month = currentTime.getMonth();
      int day = currentTime.getDayOfMonth();
      int seconds = currentTime.getSecond();
        
      System.out.println("月: " + month +", 日: " + day +", 秒: " + seconds);//月: APRIL, 日: 15, 秒: 48
        
      LocalDateTime date2 = currentTime.withDayOfMonth(10).withYear(2012);
      System.out.println("date2: " + date2);			//date2: 2012-04-10T16:55:48.668
        
      // 12 december 2014
      LocalDate date3 = LocalDate.of(2014, Month.DECEMBER, 12);
      System.out.println("date3: " + date3);			//date3: 2014-12-12
        
      // 22 小时 15 分钟
      LocalTime date4 = LocalTime.of(22, 15);
      System.out.println("date4: " + date4);			//date4: 22:15
        
      // 解析字符串
      LocalTime date5 = LocalTime.parse("20:15:30");
      System.out.println("date5: " + date5);			//date5: 20:15:30
   }
}
```
2. 使用时区的日期API
```java 
import java.time.ZonedDateTime;
import java.time.ZoneId;
 
public class Java8Tester {
   public static void main(String args[]){
      Java8Tester java8tester = new Java8Tester();
      java8tester.testZonedDateTime();
   }
    
   public void testZonedDateTime(){
    
      // 获取当前时间日期
      ZonedDateTime date1 = ZonedDateTime.parse("2015-12-03T10:15:30+05:30[Asia/Shanghai]");
      System.out.println("date1: " + date1);	//date1: 2015-12-03T10:15:30+08:00[Asia/Shanghai]
        
      ZoneId id = ZoneId.of("Europe/Paris");
      System.out.println("ZoneId: " + id);		//ZoneId: Europe/Paris
        
      ZoneId currentZone = ZoneId.systemDefault();
      System.out.println("当期时区: " + currentZone);		//当期时区: Asia/Shanghai
   }
}
```