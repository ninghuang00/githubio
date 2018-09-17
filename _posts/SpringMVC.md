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
```java
<dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```
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
### 例子
```java
@Aspect
@Component
public class AopLog { //这个类就是切面,切面由若干切点和通知组成
    Logger logger = LoggerFactory.getLogger(AopLog.class);
    @Autowired
    JobService jobService;

    //这个方法定义切点,一般就是要代理的方法
    @Pointcut("execution(* com.hn.controller.JobController.delete*(..))")
    public void forDelete() {}
    @Pointcut("execution(* com.hn.controller.JobController.query*(..))")
    public void forQuery() {}

    //这个就是通知,定义切点工作的时机和工作内容
    @Before("forDelete()")
    public void deleteLog(JoinPoint joinPoint) {
        String className = joinPoint.getTarget().getClass().getName();
        String methodName = joinPoint.getSignature().getName();
        Object[] params = joinPoint.getArgs();
        logger.info("target class is {}, and target method is {}, and params are {}", className, methodName, Arrays.asList(params));
    }

    @AfterReturning(value = "forQuery()", returning = "ret")
    public void queryLog(JoinPoint joinPoint, Object ret) {
        String result = ret.toString();
        logger.info("result is {}", result);
    }
}
```

### 使用场景:
1. 比较常见的就是日志记录,如果不使用aop的话,例子中**实现日志功能的代码就是切面,而需要添加日志的类或者方法就是切点.**
	1. 就会在业务组件的核心功能之外附带了其他功能代码,会让业务代码变得混乱,业务组件应该只关心核心功能的实现
	2. 比如日志功能如果在多个组件中被使用,那么日志功能的业务逻辑发生改变时,维护修改就很麻烦
1. 日志应用:可以在日志中输出监控对象的方法调用的参数输入,返回结果等
比如要监控的对象是要监控`LoginServiceImpl.login(String name,String password)`,只需要准备一个`LogServiceImpl`,`LogServiceImpl`中实现无参日志方法,有参日志方法,还有有参有返回值的日志方法,然后通过`applicationContext.xml`中的`<aop:config>`将其中一个日志方法织入`LoginServiceImpl`中配置,然后在客户端调用login方法的时候日志中就会有相应的输出内容
参考地址:http://www.cnblogs.com/yulinfeng/p/7719128.html


## 注解
>参考地址:https://blog.csdn.net/lylwo317/article/details/52163304

注解本质是一个继承了Annotation的特殊接口，其具体实现类是Java运行时生成的动态代理类。通过代理对象调用自定义注解（接口）的方法，会最终调用AnnotationInvocationHandler的invoke方法。该方法会从memberValues这个Map中索引出对应的值。而memberValues的来源是Java常量池。
### 注解的使用
1. 声明作用在类上的注解,保留到运行时,默认值"default"
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface HelloAnnotation {
    String say() default "Hi";
}
```
其实注解被编译后的本质就是一个继承Annotation接口的接口,所以`@HelloAnnotation`其实就是`public interface HelloAnnotation extends Annotation`,当我们通过`AnnotationTest.class.getAnnotation(HelloAnnotation.class)`调用时，JDK会通过**动态代理**生成一个实现了`HelloAnnotation`接口的对象，并把将`RuntimeVisibleAnnotations`属性值设置进此对象中，此对象即为`HelloAnnotation`注解对象，通过它的`say()`方法就可以获取到注解值。
2. 声明作用在方法上的类,保留到运行时,
```java
@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
public @interface TestMethod {
    String value();
}
```
3. 通过class获取注解
```java
@HelloAnnotation("Do it")
public class AnnotationTest {
    @TestMethod("tomcat-method")
    public void test(){
    }
    public static void main(String[] args){
        HelloAnnotation t = AnnotationTest.class.getAnnotation(HelloAnnotation.class);
        System.out.println(t.say());//Do it
        TestMethod tm = null;
        try {
            tm = AnnotationTest.class.getDeclaredMethod("test",null).getAnnotation(TestMethod.class);
        } catch (Exception e) {
            e.printStackTrace();
        }
        System.out.println(tm.value());//tomcat-method
    }
}
```

### SpringMVC中的常用注解
> 参考地址:https://www.javazhiyin.com/12491.html

1. `@ResponseBody`
 `@ResponseBody`注解使用的返回值处理器是`RequestResponseBodyMethodProcessor`,调用 `HttpMessagConverter` 消息转换机制,使用后Controller中的方法可以直接返回json数据
2. `@RestController`
相当于使用了`@ResponseBody`和`@Controller`
3. `@RequestParam`
绑定请求和Controller中方法的参数
3. `@PathVariable`
请求URI中的模板变量部分到处理器功能处理方法的方法参数上的绑定,比如
```java
@RequestMapping(value="/get/{bookId}")
public String getBookById(@PathVariable String bookId,Model model){
  model.addAttribute("bookId", bookId);
  return "book";
}
``` 
4. `@RequestMapping`
请求路径映射
RequestMapping注解有六个属性，下面我们把她分成三类进行说明（下面有相应示例）。
  1. value， method；
  value：     指定请求的实际地址，指定的地址可以是URI Template 模式（后面将会说明）；
  method：  指定请求的method类型， GET、POST、PUT、DELETE等；
  2. consumes，produces
  consumes： 指定处理请求的提交内容类型（Content-Type），例如application/json, text/html;
  produces:    指定返回的内容类型，仅当request请求头中的(Accept)类型中包含该指定类型才返回；
  3. params，headers
  params： 指定request中必须包含某些参数值是，才让该方法处理。
  headers： 指定request中必须包含某些指定的header值，才能让该方法处理请求。
5. `@Scope(“prototype”)`
  1. 默认是`singleton`单例的作用域,声明成原型作用域每次使用都是创建新的实例
  2. 适用于Web应用的还有`request`,`session`,`application`

6. `@Bean`
使用在返回实例的方法上


## SpringBoot
### 常用配置
1. 默认扫描位置
  * springboot默认扫描`@SpringBootApplication`入口类所在路径,同级目录包及其子包
  * 也可以使用注解`@ComponentScan(basePackages={"com.hn"})`指定扫描路径
2. 配置日志

### 配置优先级
Spring Boot 所提供的配置优先级顺序比较复杂。按照优先级从高到低的顺序:
1. 命令行参数。
2. 通过 System.getProperties() 获取的 Java 系统参数。
3. 操作系统环境变量。
4. 从 java:comp/env 得到的 JNDI 属性。
5. 通过 RandomValuePropertySource 生成的1random.*1属性。
6. 应用 Jar 文件之外的属性文件。
7. 应用 Jar 文件内部的属性文件。
8. 在应用配置 Java 类（包含@Configuration注解的 Java 类）中通过@PropertySource注解声明的属性文件。
9. 通过SpringApplication.setDefaultProperties声明的默认属性。
SpringBoot的这个配置优先级看似复杂，其实是很合理的。比如命令行参数的优先级被设置为最高。这样的好处是可以在测试或生产环境中快速地修改配置参数值，而不需要重新打包和部署应用。


> 参考地址https://www.javazhiyin.com/16381.html

3. 配置mybatis
  1. 添加mybatis和mysql驱动依赖
  2. 配置文件中添加数据库参数


## Bean的实例化过程

## spring事务
{% asset_img 事务属性.png 事务属性%}
可以使用注解`@Transaction(propagation=Propagation. REQUIRED,readOnly=true)`配置
1. 传播行为
传播行为定义了客户端与被调用方法之间的事务边界，即传播规则回答了这样的一个问题，新的事务应该被启动还是挂起，或者方法是否要在事务环境中运行。
2. 隔离级别
ISOLATION_DEFAULT:使用后端数据库默认的规则
ISOLATION_READ_UNCOMMITTED:允许读取尚未提交的数据变更，可能会导致脏读，幻读或不可重复读
ISOLATION_READ_COMMITTED:允许读取并发事务已经提交的数据，可以防止脏读，但是幻读或不可重复读仍有可能发生
ISOLATION_REPEATABLE_READ:对同意字段的多次读取结果是一致的，除非数据是被本事务自己所修改，看阻止脏读和不可重复读，但幻读仍有可能发生
ISOLATIOM_SERIALIZABLE:完全服从ACID的隔离级别，确保阻止脏读，不可重复读以及幻读，这是最慢的数据隔离级别
3. 是否只读
如果事务只对后端的数据库进行读操作，数据库可以利用事务ID只读特性来进行一些特定的优化。通过将事务设置为只读，你就可以给数据库一个机会，让他应用它认为合适的优化措施。因为是否只读是在事务启动的时候由数据库实施的，所以只有对那些具备可能启动一个新事务的传播行为（PROPAGATION_REQUIRED,PROPAGATION_REQUIRED_NEW,PROPAGATION_NESTED）的方法来说，才有意义。
4. 事务超时
为了使应用程序很好地运行，事务不能运行太长时间。因为超时时钟会在事务开始时启动，所以只有对那些具备可能启动一个新事务的传播行为（PROPAGATION_REQUIRED,PROPAGATION_REQUIRED_NEW,PROPAGATION_NESTED）的方法来说，才有意义。
5. 事务回滚
事务回滚规则定义了哪些异常会导致事务回滚而哪些不会。默认情况下，事务只有在遇到运行时期异常才回滚，而在遇到检查型异常时不会回滚。

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
>参考地址:http://lgbolgger.iteye.com/blog/2105101

{% asset_img springMVC基本工作流程图.png springMVC基本工作流程图 %}

```
1.用户发起请求到前端控制器。
2.前端控制器通过处理器映射器查找handler。
3.处理器映射器返回执行链。
	a)handler对象
	b)拦截器（集合）
4.前端控制器通处理器适配器包装，执行handler对象。思考：为什么要通过适配器来执行？
因为普通的servlet方法中参数都是固定的request和response,而灵活多变的handler中的方法参数不确定,因此需要handlerAdapter
5.通过模型handler处理业务逻辑。
6.处理业务完成后，返回ModeAndView对象，其中有视图名称，模型数据。
7.将视图名称和模型数据返回到前端控制器。
8.前端控制器通过视图解释器查找视图对象。
9.视图解释器返回真正的视图。
10.前端控制器通过返回的视图和数据进行渲染。
11.返回渲染完成的视图。
12.将最终的视图返回给用户，产生响应。
```
### 代码调用流程
1. 经过ApplicationFilterChain.doFilter()
2. HttpServlet.service()
3. FrameworkServlet.doGet()
4. FrameworkServlet.proecssRequest()
5. DispatcherServlet.doService()
6. DispatcherServlet.doDispatch()
在这个方法会完成获取handler以及找到合适的handlerAdapter,然后还会调用handlerAdapter.handle(request,response,handler)方法进行真正的处理返回modelAndView
  1. HandlerMapping
  通过handlerMapping来寻找HandlerExecutionChain(其中包含了拦截器和handler),其中的handler也就是平时用的XXXController类
    * BeanNameUrlHandlerMapping ：
    通过对比url和bean的name找到对应的对象 
    * SimpleUrlHandlerMapping ：
    也是直接配置url和对应bean,比BeanNameUrlHandlerMapping功能更多 
    * RequestMappingHandlerMapping : 
    主要是针对注解配置`@RequestMapping`的
  2. HandlerAdapter
  通过handlerAdapter来寻找具体的方法,也就是XXXController类中的某个方法
    * RequestMappingHandlerAdapter : 
    和上面的`RequestMappingHandlerMapping`配对使用，针对`@RequestMapping`
    * HttpRequestHandlerAdapter ： 
    要求handler实现HttpRequestHandler接口，该接口的方法为`void handleRequest(HttpServletRequest request, HttpServletResponse response)`也就是  handler必须有一个handleRequest方法 
    * SimpleControllerHandlerAdapter：
    要求handler实现Controller接口，该接口的方法为`ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response)`

### 拦截器和过滤器
{% asset_img 拦截器过滤器流程.png 拦截器过滤器流程%}
1. 拦截器(Interceptor)
  1. 使用
  实现`HandlerInterceptor`接口,创建类继承`WebMvcConfigurerAdapter`并重写`addInterceptors()`方法,添加拦截器实例就行了
  2. 应用

  2. 注意点:
    1. 基于反射实现
    1. 其中`WebMvcConfigurerAdapter`已经过时,使用`WebMvcConfigurationSupport`代替,但是据说会影响到`WebMvcConfiguration`自动配置,访问不到静态资源
    2. 拦截可以在handle之前,之后,以及在整个请求结束之后(即dispatcherServlet渲染对应视图之后),细粒度可以具体到方法
    3. 拦截器只能拦截经过dispatcherServlet转发的请求
2. 过滤器(Filter)
  1. 使用

  2. 应用
    1. 比如过拦截掉我们不需要的接口请求
    2. 修改请求（request）和响应（response）内容
    3. 完成CORS跨域请求等等
  2. 注意点
    1. 基于回调实现
    1. 过滤器属于Servlet范畴的API,和spring没什么关系,并且需要在Servlet容器中实现
    2. 只能在request前后有用
    3. 几乎所有请求,只要符合过滤规则


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
