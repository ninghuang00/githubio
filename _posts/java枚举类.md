---
title: java枚举类
categories:
  - 笔记
tags:
  - 枚举
date: 2018-08-04 16:25:17
---
 参考地址:http://www.hollischuang.com/archives/195
 <!-- more -->

## 使用方式
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

## enum反编译
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