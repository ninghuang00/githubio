---
title: SpringMVC
categories:
  - 笔记
tags:
  - SpringMVC
  - 开源框架
date: 2018-08-07 23:35:40
---
参考地址:https://mp.weixin.qq.com/s/Fo3ZN64qm7e2bWEKEIfBFA
 <!-- more -->

## 九大组件
```java
protected void initStrategies(ApplicationContext context) {  
//用于处理上传请求。处理方法是将普通的request包装成            
//MultipartHttpServletRequest，后者可以直接调用getFile方法获取File.
 initMultipartResolver(context);  
//SpringMVC主要有两个地方用到了Locale：
 //一是ViewResolver视图解析的时候；
 //二是用到国际化资源或者主题的时候。
 initLocaleResolver(context); 
//用于解析主题。SpringMVC中一个主题对应 一个properties文件，里面存放着跟当前主题相关的所有资源、
//如图片、css样式等。SpringMVC的主题也支持国际化， 
 initThemeResolver(context);  
//用来查找Handler的。
 initHandlerMappings(context);  
//从名字上看，它就是一个适配器。Servlet需要的处理方法的结构却是固定的，都是以request和response为参数的方法。
//如何让固定的Servlet处理方法调用灵活的Handler来进行处理呢？这就是HandlerAdapter要做的事情
 initHandlerAdapters(context);  
//其它组件都是用来干活的。在干活的过程中难免会出现问题，出问题后怎么办呢？
//这就需要有一个专门的角色对异常情况进行处理，在SpringMVC中就是HandlerExceptionResolver。
 initHandlerExceptionResolvers(context);  
//有的Handler处理完后并没有设置View也没有设置ViewName，这时就需要从request获取ViewName了，
//如何从request中获取ViewName就是RequestToViewNameTranslator要做的事情了。
 initRequestToViewNameTranslator(context);
//ViewResolver用来将String类型的视图名和Locale解析为View类型的视图。
//View是用来渲染页面的，也就是将程序返回的参数填入模板里，生成html（也可能是其它类型）文件。
 initViewResolvers(context);  
//用来管理FlashMap的，FlashMap主要用在redirect重定向中传递参数。
 initFlashMapManager(context); 
} 
```

