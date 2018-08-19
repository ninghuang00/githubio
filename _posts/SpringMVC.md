---
title: SpringMVC
categories:
  - 笔记
tags:
  - SpringMVC
  - 开源框架
  - spring
  - 注解
date: 2018-08-07 23:35:40
---
开发自己的SpringMVC
参考地址:https://mp.weixin.qq.com/s/Fo3ZN64qm7e2bWEKEIfBFA
 <!-- more -->



## spring框架的基本原理
>基本原理其实就是通过反射(反射的原理)解析类及其类的各种信息，包括构造器、方法及其参数，属性。然后将其封装成bean定义信息类、constructor信息类、method信息类、property信息类，最终放在一个map里，也就是所谓的container，池等等，其实就是个map。。汗。。。。当你写好配置文件，启动项目后，框架会先按照你的配置文件找到那个要scan的包(一般是哪些包?)，然后解析包里面的所有类，找到所有含有@bean，@service等注解的类，利用反射解析它们，包括解析构造器，方法，属性等等，然后封装成各种信息类放到一个map里。**每当你需要一个bean的时候，框架就会从container找是不是有这个类的定义啊？如果找到则通过构造器new出来（这就是控制反转，不用你new,框架帮你new）**，再在这个类找是不是有要注入的属性或者方法，**比如标有@autowired的属性，如果有则还是到container找对应的解析类，new出对象，并通过之前解析出来的信息类找到setter方法，然后用该方法注入对象（这就是依赖注入）**。如果其中有一个类container里没找到，则抛出异常，比如常见的spring无法找到该类定义，无法wire的异常。还有就是嵌套bean则用了一下递归，container会放到servletcontext里面，每次reQuest从servletcontext找这个container即可，不用多次解析类定义。如果bean的scope是singleton，则会重用这个bean不再重新创建，将这个bean放到一个map里，每次用都先从这个map里面找。如果scope是session，则该bean会放到session里面。仅此而已，没必要花更多精力。

### bean工厂
```java
public class BeanFactory {
       private Map<String, Object> beanMap = new HashMap<String, Object>();
       /**
       * bean工厂的初始化.
       * @param xml xml配置文件
       */
       public void init(String xml) {
              try {
                     //读取指定的配置文件
                     SAXReader reader = new SAXReader();
                     ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
                     //从class目录下获取指定的xml文件
                     InputStream ins = classLoader.getResourceAsStream(xml);
                     Document doc = reader.read(ins);
                     Element root = doc.getRootElement();  
                     Element foo;
                    
                     //遍历bean
                     for (Iterator i = root.elementIterator("bean"); i.hasNext();) {  
                            foo = (Element) i.next();
                            //获取bean的属性id和class
                            Attribute id = foo.attribute("id");  
                            Attribute cls = foo.attribute("class");
                           
                            //利用Java反射机制，通过class的名称获取Class对象
                            Class bean = Class.forName(cls.getText());
                           
                            //获取对应class的信息
                            java.beans.BeanInfo info = java.beans.Introspector.getBeanInfo(bean);
                            //获取其属性描述
                            java.beans.PropertyDescriptor pd[] = info.getPropertyDescriptors();
                            //设置值的方法
                            Method mSet = null;
                            //创建一个对象
                            Object obj = bean.newInstance();
                           
                            //遍历该bean的property属性
                            for (Iterator ite = foo.elementIterator("property"); ite.hasNext();) {  
                                   Element foo2 = (Element) ite.next();
                                   //获取该property的name属性
                                   Attribute name = foo2.attribute("name");
                                   String value = null;
                                  
                                   //获取该property的子元素value的值
                                   for(Iterator ite1 = foo2.elementIterator("value"); ite1.hasNext();) {
                                          Element node = (Element) ite1.next();
                                          value = node.getText();
                                          break;
                                   }
                                  
                                   for (int k = 0; k < pd.length; k++) {
                                          if (pd[k].getName().equalsIgnoreCase(name.getText())) {
                                                 mSet = pd[k].getWriteMethod();
                                                 //利用Java的反射机制调用对象的某个set方法，并将值设置进去
                                                 mSet.invoke(obj, value);
                                          }
                                   }
                            }
                           
                            //将对象放入beanMap中，其中key为id值，value为对象
                            beanMap.put(id.getText(), obj);
                     }
              } catch (Exception e) {
                     System.out.println(e.toString());
              }
       }
      
       //other codes
}
```

## 依赖注入()
就是通过反射bean的setter方法,set成员bean的过程
## 控制反转(IOC)
就是通过配置文件+反射生成bean
## 切面编程(AOP)
参考网址:	https://www.liaoxuefeng.com/article/0013738774263173c42eae58864423698dd40556af23bb5000
### 通知(Advice)
定义切点何时工作,以工作内容
	1. 前置通知(Before):一般是方法调用前打出传入参数
	2. 后置通知(After):是返回和异常通知的并集
	3. 返回通知(After-running):方法正常返回
	4. 异常通知(After-throwing):方法抛出异常
	5. 环绕通知(Around):可以同时定义前置和后置
### 切点(PointCut)
定义在何处工作,也就是对哪个方法应用通知

### 使用场景:
1. 比较常见的就是日志记录,如果不使用aop的话,例子中**实现日志功能的代码就是切面,而需要添加日志的类或者方法就是切点.**
	1. 就会在业务组件的核心功能之外附带了其他功能代码,会让业务代码变得混乱,业务组件应该只关心核心功能的实现
	2. 比如日志功能如果在多个组件中被使用,那么日志功能的业务逻辑发生改变时,维护修改就很麻烦
1. 日志应用:可以在日志中输出监控对象的方法调用的参数输入,返回结果等
比如要监控的对象是要监控`LoginServiceImpl.login(String name,String password)`,只需要准备一个`LogServiceImpl`,`LogServiceImpl`中实现无参日志方法,有参日志方法,还有有参有返回值的日志方法,然后通过`applicationContext.xml`中的`<aop:config>`将其中一个日志方法织入`LoginServiceImpl`中配置,然后在客户端调用login方法的时候日志中就会有相应的输出内容
参考地址:http://www.cnblogs.com/yulinfeng/p/7719128.html

```java 
<bean id="logService" class="cn.com.spring.service.impl.LogServiceImpl"></bean> 
<bean id="loginService" class="cn.com.spring.service.impl.LoginServiceImpl"></bean> 
<aop:config> 
    <!-- 切入点 --> 
    <aop:pointcut 
        expression="execution(* cn.com.spring.service.impl.LoginServiceImpl.*(..))" 
        id="myPointcut" /> 
    <!-- 切面： 将哪个对象中的哪个方法，织入到哪个切入点 --> 
    <aop:aspect id="dd" ref="logService"> 
        <!-- 前置通知 
        <aop:before method="log" pointcut-ref="myPointcut" /> 
        <aop:after method="logArg" pointcut-ref="myPointcut"> 
		--> 
        <aop:after-returning method="logArgAndReturn" returning="returnObj" pointcut-ref="myPointcut"/> 
    </aop:aspect> 
</aop:config> 
```

## 注解

### SpringBoot

### SpringMVC中的注解
1. @ResponseBody 
 `@ResponseBody`注解使用的返回值处理器是`RequestResponseBodyMethodProcessor`，调用 `HttpMessagConverter` 消息转换机制。

## Bean的实例化过程

## spring事务
### 传播行为
### 隔离级别

## spring和springMVC
1. spring是父容器,springMVC是子容器,spring容器中有Service层的bean,springMVC容器中有Controller层的bean,子容器中找不到的bean可以去父容器中找,但是父容器不能在子容器中查找bean,这就是为什么由SpringMVC容器创建的Controller可以自动注入Spring容器创建的Service的bean了

## 传统springMVC的不足
1. 传统MVC模式的不足
	1. 每次请求都必须经过"控制器-模型-视图"才能最终在浏览器上呈现,过程略显复杂
	2. 视图依赖于模型,没有模型,浏览器就无法展示页面,客户体验不好
	3. 视图的渲染是在服务器端进行的,最后返回给浏览器的是带有模型的视图,渲染性能不能得到很好的优化
2. 前后端分离的方式
就是浏览器发送AJAX请求,然后收到JSON数据返回,然后浏览器来渲染页面


## SpringMVC HTTP请求响应的过程
{% asset_img springMVC基本工作流程图.jpeg springMVC基本工作流程图 %}

{% asset_img SpringMVC中HTTP请求处理流程时序图.jpeg SpringMVC中HTTP请求处理流程时序图 %}
```
1.用户发起请求到前端控制器。
2.前端控制器通过处理器映射器查找handler。
3.处理器映射器返回执行链。
	a)handler对象
	b)拦截器（集合）
4.前端控制器通处理器适配器包装，执行handler对象。思考：为什么要通过适配器来执行？
5.通过模型handler处理业务逻辑。
6.处理业务完成后，返回ModeAndView对象，其中有视图名称，模型数据。
7.将视图名称和模型数据返回到前端控制器。
8.前端控制器通过视图解释器查找视图对象。
9.视图解释器返回真正的视图。
10.前端控制器通过返回的视图和数据进行渲染。
11.返回渲染完成的视图。
12.将最终的视图返回给用户，产生响应。
```

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

##Json问题
### 返回json数据中文乱码的问题
1. 方法1:在springMVC.xml中配置
```xml
<!-- 处理请求返回json字符串的中文乱码问题 -->  
    <mvc:annotation-driven>  
        <mvc:message-converters>  
            <bean class="org.springframework.http.converter.StringHttpMessageConverter">  
                <property name="supportedMediaTypes">  
                    <list>  
                        <value>application/json;charset=UTF-8</value>  
                    </list>  
                </property>  
            </bean>  
        </mvc:message-converters>  
    </mvc:annotation-driven>  
```
2. 方法二:在url映射注解中添加一段
	`@RequestMapping(value="/getUsersByPage",produces = "application/json; charset=utf-8")`
