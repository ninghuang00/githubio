---
title: Javaweb25天笔记
categories:
  - 笔记
tags:
  - javaweb
date: 2018-07-08 16:55:11
---
 传智播客视频javaweb25天笔记
 <!-- more -->


# 总结：
## tomcat服务器的安装和配置，http协议
* 虚拟目录的映射，
* 虚拟目 录的配置（serve.xml,catalina\localhost\）
* http协议的请求和响应的各个组成部分
	1. 各个常见请求头的含义
	2. 各个响应头的含义
## servlet开发
* servlet的开发7个细节性的问题
* ServletConfig
* ServletContext对象
	* 域对象
	* 读取资源文件（三种方式）
		* ServletContext（资源不在类目录下的时候
		* ClassLoader（文件比较小的时候，因为资源h会被读到内存中
		* FileInputStream（
	* 转发 this.getServletContext.getRequestDispatcher()
## request和response的使用
#### 一个http响应应该包含响应状态行 响应头 响应数据 要掌握通过response发送者三部分内容
	2 response发送中文
	3 response发送动态图片
	4 response实现下载
	5 response实现请求重定向
	6 meta标签模拟http响应头

	1 http请求包含请求头 请求头 请求数据 要掌握通过request获取者三部分内容
	2 使用request获取中文数据的问题（get/post
	3 使用request获取各种不同类型的表单数据（text\checkbox\radio
	4 域对象
	5 实现请求转发
## session和Cookie（回话过程中多个web组件之间数据共享问题
	#### cookie：吧数据打给客户机
		1 显示用户上次登录的时间
		2 用户的历史浏览商品记录
	#### session：把数据存在服务器的session对象中
		1 购物案例
			1 实现关掉浏览器后，session还能为用户服务，一个session为多个窗口服务
			2 实现用户禁用Cookie后，还能实现购物
		2 用户登录
		3 防止表单重复提交
			1 md5/base64
## jsp
	1 9大内置对象
	2 jsp常见的标签
	3 el表达式和jstl标签库
## javabean开发
	1 jsp+javabean（简单计算器
	2 jsp+servlet+javabean（mvc
	3 用户登录和注册案例
	4 购物车案例
## 自定义标签
	1 传统标签（了解
		tag：是否执行标签体，以及标签余下的jsp
		IterationTag：迭代标签体
		bodyTag：修改标签体
	2 简单标签（掌握
	3 SimpleTag（可以替换上面几个传统标签
	4 标签属性
	5 打包标签库
	6 .EXE文件
## el + jstl web国际化
	1 固定文本的国际化
		properties文件，一组资源文件就是一个资源包，ResourceBundle.getBundle（）
	2 动态数据国际化
		DateFormat：日期国际化
			Format（Date date） 符合国家地区习惯的字符串
			parse（String date） 把字符串格式化成日期
	3 MessageFormat
		批量国际化

## jdbc开发
	1 入门
	2 jdbc中涉及到的每一个对象的方法解析
		DriverManager
		Connection
		Statement（PreparedStatement）
		ResultSet
	3 用jdbc改造注册登录案例
		DaoFactory
		DaoException
		sqlz注入问题
	4 PreparedStatement（PreparedStatement和Statement对象的区别
	5 jdbc案例（客户关系
## jdbc开发2
	1 分页（Page）
	2 批处理
	3 大数据
	4 存储过程的调用
	5 获取自动增长的主键
## jdbc开发3
	1 事务
		1 什么是事务 start transactioncommit rollback；
		2 事务的四大特性：ACID
		3 四大特性的隔离级别，
			脏读 一个事务读取到另一个事务已经提交的数据
			不可重复读
			虚读 
		4 子安jdbc程序中怎么控制多条sql作为整体执行，在jdbc中如何设置事务隔离级别
		5 连接池的实现原理
		6 常用的连接池产品
			1 tomcat内置（DBCP，JNDI
			2 DBCP C3P0
		7 元数据
		8 自己设计类似于DBUtils的框架
## jdbc开发4 
	1 DbUtils的使用（常见的集中处理器
	2 实际开发中如何控制事务（ThreadKLocal***
	3 jdbc多表操作（hibernate基础
		一对多（多对一） 一般建议在多的一方维护关系
		多对多 一对一
	Oracle的大数据
## 过滤器和监听器


----
# day07
* 会话管理 cookie技术（客户端的技术） session技术（服务器端的技术）
* 用cookie写一个历史浏览记录
* session技术 用session实现购物记录
* session的原理 设置session的有效期 浏览器禁用Cookie后servlet共享数据导致的问题 url重写
* session的常见应用 
	1 登录 
	2 防止表单重复提交 前台javaScript 后台表单带随机数 
	3 实现一次性校验码
* 实现带验证码的登录


----
day17 dbutils
	01 dbutils文档 大数据不建议使用这个处理，自己写传统的方法处理
	02 dbutils的已经定义的处理器，如何进行实物处理
	03 ThreadLocal：看成一个容器，想这个容器中存放的对象，在当前线程范围内都可以取得出来


----
day08 jsp

## JSP技术介绍 
	原理
	1 web服务器是如何调用并执行一个jsp页面的 JSP实际上就是servlet
	2 jsp页面中的html排版标签是如何被发送到客户端的 out.write()
	3 jsp页面中的java代码服务器是如何执行的 原封不动
	4 web服务器在调用JSP时，会给JSP提供一些什么java对象
	jsp语法
	自定义标签 el表达式
## error-page
## jsp乱码问题 jsp以什么码表保存在硬盘上，就通知浏览器以什么码表打开 include指令（jsp指令不能冲突）
	JSP九大隐式对象
	1 Request
	2 Response
	3 Session
	4 Application servletContext
	5 Config servletConfig
	6 Page this
	7 Exception
	8 Out JSPWriter
	9 pageContext
	主要讲Out隐式对象和pageContext隐式对象（最重要）可以获得其他八大隐式对象 

## jsp标签 
## 重点 web开发涉及到的4个域对象
	1 pageContext 页面范围内都可取
	2 Request 如果客户端向服务器发请求，产生数据，客户看完就没用了（新闻）
	3 Session 如果客户端向服务器发请求，产生数据，客户看完还有用（购物，购物信息，结账，浏览记录）
	4 servletContext 如果客户端向服务器发请求，产生数据，客户看完了还要给其他用户用（聊天室）


----
day09 mvc设计模式开发注册登录

## javabean 和jsp 什么是javabean（遵循特定写法的java类）
	1 这个java类必须有一个无参构造函数
	2 属性必须私有化
	2 有getter和setter函数
	JSP提供的三个javabean组件动作元素
	1 JSP：useBean
	2 jsp：setBean
	3 JSP：setProperty
## mvc设计模式 el表达式( ${customerBean.address} ) jsp自定义标签
	el表达式如何取数据

## jsp标签 


----
day10 mvc购物车


----
day11
## 自定义标签库 移除jsp中的java代码
	1 实现Tag接口的java类
	2 编写标签库描述符（tld）文件，在tld文件中岁标签处理器类进行描述
## 自定义标签如何控制jsp页面
	1 实现tag接口，控制dostarttag方法的返回值，如果这个方法的返回值为
		`EVAL_BODY_INCLUDE`：执行标签体
		`SKIP_BODY`：不返回标签体
	2 控制doendtag的返回值，
		`EVAL_PAGE`：执行余下的JSP页面
		`SKIP_PAGE`：不执行剩下的
	3 实现IterationTag接口，控制doafterbody方法的返回值，
		`EVAL_BODY_AGAIN`：又执行一次标签体，直到返回SKIP_BODY，
	4 实现BodyTag接口，控制doStartTag接口，控制doStartTag方法
		`EVAL_BODY_BUFFERED`，web服务器会创建BodyContent对象捕获标签体，然后在doendtag方法中得到代表bodyContent，就行修改
	（以上内容听说已经淘汰，但是以前的框架在用，要能看得懂）
## 简单标签库开发 SimpleTag接口
	1 `//得到代表标签体的JSPFragment`
		`JspFragment jf = this.getJspBody();`
		`//其实默认就是输出给浏览`
		`jf.invoke(null);`
	2 控制余下的jsp不执行，
		`throw new SkipPageException();`

## 开发带属性的标签 防盗链
	1 如果标签接受的是复杂类型，如何给其赋值，用el表达式

## 开发 foreach标签 if else标签

## html转义标签 打包标签库 配置文件放在META-INF
	如何将java程序打包成可执行的jar包
	1 先导出jar包，然后修改META-INFO文件夹中的MANIFEST.MF文件
		Main-Class: cn.hn.demo.JarDemo

----
day12
## el表达式
	1 支持的运算符
	empty运算符` <c:if test="${!empty(list)}">`
	二元表达式 `user == null ? "对不起，没登录" : "welcome you "`
	[]和. 先用.再用[] 不行再用自定义
	2 11个隐式对象
	3 调用java的方法
		1 静态方法
		2 el函数用来移除jsp中与web开发无关的java代码 el自定义标签可以移除一切java代码
## el表达式不能相互嵌套 各种常用标签
## 国际化开发 
	1 固定文本国际化 写在properties文件中
	2 动态数据国际化
		1 日期 DateFormat
		2 货币 NumberFormat
		3 动态文本 MessageFormat 批量处理 占位符
## 国际化标签库 
## 文件的上传和下载

----
day13 01 sql
## 定义外键
	create table husband{
		id int primary key,
		name varchar(20)
	};
	create table wife{
		id int primary key,
		name varchar(20),
		id int husband,
		constraint husband_FK foreign key(husband_id) references husband(id)
	};

## 表与表之间的关系：
	1 一对多，在多的表加外键约束，如部门和员工，在员工表加外键约束
	2 多对多，采用中间表，中间表都是外键，如老师和学生，一个老师有多个学生，一个学生有多个老师
	3 一对一，分为主从，一般在从表的主键约束列加外键约束，

----
day14 JDBC
## JDBC简介 
## Connection对象常用方法
	1 createStatement（）
	2 prepareStatement（sql）
	3 prepareCall（sql）
	4 setAutoCommit（Boolean autoCommit）
	5 commit（）
	6 rollback（）

## Statement对象常用方法
	1 executeQuery（String sql）：发送查询语句
	2 executeUpdate（String sql）：发送insert，delete，update语句
	3 execute（String sql）：发送所有，返回boolean
	4 addBatch（String sql）：把多条sql语句放在一个批处理中
	5 executeBatch（）：向数据库发送一批sql语句执行

## 用jdbc改造用户登录项目
## PreparedStatement类
	1 prepareStatement（）：程序对sql语句进行预编译，减轻数据库的压力	
	2 防范SQL语句注入的问题
## customer案例

----
day15 
## 分页功能 
	1 改写dao 从数据库获得数据计算分页信息
	2 设计page对象 封装分页的页面数据
	3	改写service，对web层提供分页数据服务
## 一个页面里面显示的页码数
## 重构分页 将分页抽取出来作为一个jsp文件，然后要用的地方 <%@include file="/public/page.jsp" %>
	用自定义标签显示省略号

## 对数据修改和删除
## 使用jdbc处理大数据（大文本和二进制）
	jdbc用批处理 两种处理方式Statement和prepareStatement
	用java调用存储过程

----
day16 事务
## 事务的概念 将多条sql语句作为一个整体执行 
	Start transaction 开始事务
	rollback 回滚事务
	commit 提交事务

	事务的特性 ACID
	atomicity 原子性
	consistency 一致性
	isolation 隔离性
	durability 持久性

## 事务的隔离性
	1 脏读 一个事务读取到另一个事务未提交的数据
	2 不可重复读 第一次和第二次读的数据不一致
	3 虚读 一个事务读到另一个事务插入的数据

	解决方法 set transaction isolation level Serializable
			select @@_isolation
	serializable（级别最高）
	reapeatable read
	read committed
	read uncommitted

## 连接池
	如何改写Connection.close（）方法
	当发现一个对象的方法不够用的时候
	1、写一个子类，覆盖方法
	2、写一个包装类，增强方法
		1 写一个类实现与被增强对象相同的接口
		2 定义一个变量，指向被增强对象
		3 定义一个构造方法，接受被增强对象
		4 覆盖想增强的方法
		5 对于不想增强的方法，直接调用被增强对象的方法
	3、用动态代理，返回一个代理对象，拦截原方法的调用(拦截机制)
		1 Proxy.newProxyInstance(JdbcPool.class.getClassLoader(), conn.getClass().getInterfaces(), new InvocationHandler() {
		@override
		...
		});
	使用开源库，dbcp
## 开源库 c3p0的使用 
	使用tomcat服务器创建的连接池 JNDI容器(java naming and directory interface)
## 编写自己的jdbc框架 
	1 利用元数据，改写dao的增删改方法，使用ResultSetMetaData对象获取数据信息
	2 使用接口以及反射技术改写查方法，即处理结果集的方法做成接口对象参数，让用户传入，

----
day17 dbutil

----
day18 
## web.xml中的配置不正确可能会导致tomcat服务器启动失败
	01 servlet过滤器Filter
		filter可以对request和response进行一些预处理，比如进行字符集设置
		可以对filter设置初始化参数
## 用 `${pageContext.request.contextPath }`获得项目路径
	02 filter的常见应用
		1 阻止浏览器缓存jsp页面的filter
		2 自动登录功能
			完善自动登录功能，因为filter会拦截所有页面的请求，自动登录以后每次访问网页又执行登陆验证的代码
	03 filter的部署和注册
		可以拦截4种向服务器请求资源的方式
			request forward error include
		过滤器高级开发，对拦截的request，response增强之后进行传递
			使用包装设计模式增强BufferedReader 

----
Day19 权限过滤器
## 搭建框架
## 三个对象
    1. 用户
    2. 角色
    3. 权限
## 数据库五张表
### User表
	create table user(
	id varchar(40) primary key,
	username varchar(100) not null unique,
	password varchar(100) not null,
	description varchar(255)
	);

### role表
	     create table role(
	     id varchar(40) primary key,
	     name varchar(100) not null unique,
	     description varchar(255)
	     );

### privilege表

     create table privilege(
     id varchar(40) primary key,
     name varchar(100) not null unique,
     description varchar(255)
     );

### user_role表

		create table user_role(
		user_id varchar(40),
		role_id varchar(40),
		primary key(user_id,role_id),
		constraint user_id_FK foreign key(user_id) references user(id),
		constraint role_id_FK foreign key(role_id) references role(id)
		);

### role_privilege表

		create table role_privilege(
		role_id varchar(40),
		privilege_id varchar(40),
		primary key(role_id,privilege_id),
		constraint role_id_FK1 foreign key(role_id) references role(id),
		constraint privilege_id_FK foreign key(privilege_id) references privilege(id)
		);

## sp页面分割 head left body
	权限管理 jsp页面设计 servlet设计 

----
day20 文件的上传和下载

	01 表单文件上传
		1 解决上传文件名和上传文件数据的乱码问题
		2 表单为文件上传，request设置编码无效,只能手工转化
		value = new String(value.getBytes("iso8859-1"),"UTF-8");
	 	3 为了保证服务器安全，上传文件应该放在外界无法访问的目录
		4 为了保证上传文件的唯一性，为用户上传的文件名生成UUID
		5 保存上传文件的目录要打散，使用hash算法分配路径makePath（），防止一个目录下存放太多的文件
	02 限制文件上传最大值
		1 调用item.delete()确保删除临时文件
		2 限制上传文件类型
		3 设置上传文件进度监听器 要在upload.parseRequest(request);之前
		4 在web页面中动态添加上传输入项
	03 Servlet监听器
		1 frame窗口demo，
		2 如何为自己写的对象设计监听器,观察者设计模式（Observer）
		3 Servlet监听器
			1 ServletContext监听器，实现ServletContextListener接口，然后在web.xml中配置注册，`<listener>`标签
	04 各种监听器
		1 HttpSessionListener，session的创建和销毁（网站在线人数）
		2 HttpRequest监听器（网站请求数量）
	05 各种监听器
		1 自定义session扫面器 （增删改查用linkedList
		2 用监听器定时发送邮件
		3 监听三个域对象的属性变化
			1 ServletCOntextAttributeListener(据说没什么大用)
			2 HttpSessionAttributeListener
			3 ServletRequestAttributeListener
		4 感知Session绑定的事件监听器 
	06 显示登录用户列表，并实现踢人功能
day21 书店管理系统

----
day22 java加强 泛型
## 泛型方法和泛型类，要先定义后使用,类上声明的泛型只能用于类中的非静态方法，静态方法需要单独声明
	public class Dema{
		public <T> void aa(T t){ //泛型方法

		}

		public <T> void bb(T t){

		}
	}
	等同于
	public class Demo<T>{
		public void aa(T t){

		}

		public void bb(T t){

		}

		public static <T> void cc(T t){

		}
	}

## 泛型的使用场景，dao层的接口，结合HIbernate写非常简单，反射泛型
	以下示例代码可以直接继承，继承之后就拥有泛型类型的增删改查方法

	import org.hibernate.Session;
	public abstract class BaseDao<T>{
		private Session session;
		private Class clazz;
		//哪个子类调用这个方法就得到的class就是子类的处理类型（非常重要） 
		public BaseDao(){
		Class clazz = this.getClass(); //得到子类
		ParameterizedType  pt = (ParameterizedType)clazz.getGenericSuperclass(); //得到父类
		clazz = （Class)pt.getActualTypeArguments()[0];
		}

		public void add(T t){
			session.save(t);
		}

		public T find(String id){
			return (T)session.get(clazz,id)
		}

		public void update(T t){
			session.update(t);
		}

		public void delete(String id){
		T t = (T)session.get(clazz,id);
		session.delete(t);
		}
	}

## 注解配置
		1 annotation
			1 自定义注解
				注解可以接受的数据类型有：
					1 基本数据类型，String，Class，annotation，枚举类型，以及上述类型的一维数组
					2 元annotation，修饰注解的注解
						@Retention
						用于指定该Annotation可以保留的域，可以通过RetentionPolicy.xxx来指定
						@Target
						用于指定注解可以用于修饰的那些成员
						@Documented
						可以被提取文档
						@Inherited
						被他修饰的注解具有可继承性，就是说被修饰的注解修饰的方法被子类继承后，子类的给方法上也自动拥有该注解
					3 通过注解注入对象
						1 得到需要注入的属性
						2 得到要注入的属性需要的类型
						3 创建属性需要的对象
						4 得到属性的写方法
						5 反射出方法上声明的注解
						6 得到注解上声明的信息，填充要注入的对象
						7 将对象注入目标

			2 解析注解
				1 使用反射技术拿到方法，然后拿到方法上的注解
	04 动态代理（非常重要）
		1 newProxyInstance(ClassLoader，Class，InvacationHandler) 
		2 使用动态代理做filte可以不用重写代理对象中的所有方法，只要拦截自己感兴趣的方法就行了
		3 用动态代理修改bookstores的权限系统
			1 建立实体建立表来保存权限数据
			2 在serviceFactory中运用动态代理技术进行权限拦截，细粒度拦截到方法上
		4 类加载器
			1 负责将.class加载到内存中，并为之生成相应的java.lang.Class对象
			2 父类委托机制
				tomcat没有采用该机制，所以有可能产生classCastException异常
			3 cache机制

----
day23 邮件原理与javaMail开发
	终端直接连接互联网上的主机命令：telnet smtp.souhu.com portnumber
	QQ邮箱 POP3 和 SMTP 服务器地址设置如下：邮箱POP3服务器（端口995）SMTP服务器（端口465或587）qq.compop.qq.comsmtp.qq.comSMTP服务器需要身份验证。
	gmail pop.gmail.com 995
## SMTP服务器，处理发送邮件
		telnet smtp.sina.com 25
		smtp协议
		ehlo xxx
		auth login
	bmluZ2h1YW5nMDA= 	（经过base64编码的用户名，不需要@后面的内容）
	aG4xMjM0NTY=		（经过base64编码的密码）
	mail from:<ninghuang00@sina.com>
	rcpt to:<ninghuang00@gmail.com>

	data
	from:<ninghuang00@sina.com>
	to:<ninghuang00@gmail.com>
	subject:test

		xxxxxxxxxx
		.
		quit
## POP3服务器，处理接收邮件
		telnet pop3.sina.com 110
		user ninghaung00
		pass hn123456
		stat 
		list

## Struts开发
快乐是建立在痛苦的基础上的，你要知道这个框架为什么好，就要知道没有这个框架之前有什么不好。
	 1 之前没有struts的时候写servlet遇到的问题
	 	1 需要封装bean
	 	2 需要转发地址
	 	3 代码中的异常要自己手动捕捉
	 2 struts入门

----
day 24 struts
## Struts配置文件的常用属性
		<action
			path="/register"	//指定action处理的地址
			type="cn.hn.web.RegisterAction"	//指定处理请求的Action的完整类名
			name="User"	//指定封装数据的bean的名字
			scope="request"	//指定封装之后的bean存在什么域中，不指定默认是session
			attribute="myformbean"	//封装好的bean的key，不指定默认是name属性的值
			input="/request.jsp"	//指定接受数据的页面，校验失败后跳转回去
			forword="message,jsp"	//指定 收到请求时跳转的jsp页面，配置后action不再被调用
			include="welcome"	//指定搜到请求时进行页面包含
			unknow="true"	//默认值false，处理用户机的无效请求
			parameter="method"	//配置action参数，在action中通过调用actionMapping.getParameter方法可以获取在这里配置的参数参数
			classname=""	//<action>标签和所有的配资信息通过哪个对象进行封装，默认是actionMapping对象
			validate="true">	//默认是true，是否在封装信息时进行校验
			<forword name="hello" path="/success.jsp"/>
		</action>

以后转发请求在struts中主要在struts-config.xml中配置就行，这样就可以直接将jsp保护起来
		`<acition path="/RegisterUI" forword="/WEB-INF/jsp/register.jsp"></action>`

## 使用struts的 `<html:link> `标签构建url额好处
		1 使用 `<html:link> `标签不用关心web应用程序的名称
		2 自动附加Jsessionid进行url重写
		3 可以对参数信息进行url编码
## 使用struts完成表单校验
		1 在继承了ActionForm类的bean中重写函数validate方法完成校验，如果有错误跳转也由struts完成
		2 在properties中配置文件信息实现国际化，在file encoding中设置勾选可以自动将中文转换成ASCII编码
## struts中解决乱码问题
		1 可以自己写一个processor，不过中级解决方案还是用过滤器filter
## struts中防止表单重复提交
		1 点击注册按钮不直接Forword到注册页面，而是先通过一个action产生随机数，然后再转发到注册jsp页面
## 使用beanUtils转换表单数据
			UserFromBean formbean = （UserFormBean）form;
			User ueer = new User();
			BeanUtils.copyProperties(user,formbean);
## 在表单校验的时候进行封装，就是在UserFormBean中维护一个User，后面直接取出来就好了，这样可以避免转换两次。
## 不想进行表单校验的时候可以直接在UserFormBean中进行封装

----
day25 
	1 表单数据的业务逻辑校验
		1 用户名
	2 struts中的文件上传
		1 jsp页面中表单使用post方式提交
		2 enctype="multipart/form-data"属性
		3 ActionForm中使用org.apache.struts.upload.FormFile类型定义文件字段
		4 fileupload.jar和io.jar
		5 Action中的execute方法中使用InputStream is = file.getInputStream();方法得到文件的输入流
		6 使用io的方法对文件进行上传

## 多文件上传  
在formbean中搞一个ArrayList
Action中的execute方法:

		UpFileFormBean formbean = (UpFileFormBean) form;
        /*String username = formbean.getUsername();
        FormFile file = formbean.getUpfile();

        String filename = file.getFileName();
        InputStream is = file.getInputStream();
        FileOutputStream out = new FileOutputStream("/Users/huangning/Desktop/" + filename);
        int len = 0;
        byte[] buffer = new byte[1024];
        while((len = is.read(buffer))>0){
            out.write(buffer,0,len);
        }
        out.close();
        is.close();*/

        List<FormFile> list = ((UpFileFormBean) form).getAll();
        for (FormFile file : list) {
            String filename = file.getFileName();
            InputStream is = file.getInputStream();
            FileOutputStream out = new FileOutputStream("/Users/huangning/Desktop/" + filename);
            int len = 0;
            byte[] buffer = new byte[1024];
            while ((len = is.read(buffer)) > 0) {
                out.write(buffer, 0, len);
            }
            out.close();
            is.close();
        }
formbean的写法:

    public class UpFileFormBean extends ActionForm {
    private String username;
    //private FormFile upfile;
    List<FormFile> list = new ArrayList<>();

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public FormFile getList(int index) {
        return list.get(index);
    }

    public void setList(FormFile formFile,int index) {
        this.list.add(formFile);
    }

    public List<FormFile> getAll(){
        return this.list;
    }
	}

jsp中的:

	<form action="${pageContext.request.contextPath}/UpFile.do" method="post" enctype="multipart/form-data">

	    用户名:<input type="text" name="username"><br>
	    上传文件1:<input type="file" name="list[0]"><br>
	    上传文件2:<input type="file" name="list[1]"><br>
	    <input type="submit" value="upload">

	 </form>

## 使用struts下载文件
    	1 继承DownloadAction类,实现getStreamInfo方法
    	2 得到下载文件路径,然后返回FileStreamInfo对象就行了
## DispatchAction,使用一个Action处理多个请求
## 动态的Formbean
   		在配置文件中配合就行
## truts-Validate框架,一个struts插件,使用声明式的方式对表单进行校验,因此不需要写java代码
   		1 维护方便
   		2 校验实现标准化
   		3 自动客户端校验
## 使用方法:
   		 1 把数据封装到org.Apache.struts.validator.DynaValidatorForm
   		 2 DynaValidatorForm的validate方法被调用时,他会去调用validate框架,需要在struts中配置插件,配置步骤查看struts-core-xxx.jar下的validation-rules.xml中的<plugin...
   		 3 为了只指挥表单校验,需要在WEB-INF目录下创建validation.xml文件
   		 4 为了在校验失败后提示错误消息,在src目录下创建资源文件,参照struts-core-xxx.jar下的validation-rules.xml
## struts中的权限管理
1. 在要求权限访问的方法上使用@Permission注解,
2. 然后继承Struts的RequestProcessor类,
	* 重写processActionPerform方法
	* 写一个方法得到struts调用的方法

			Method getInvokeMethod(HTTPServletRequest request,HTTPServletResponse response, Action action,ActionForm form,ActionMapping mapping){
				//判断Struts调用的是是不是继承了DispatchAction还
				String methodName = null;
				if(!DispatchAction.class.isAssignableFrom(action.getClass())){
					methodName = "execute";
				}else{
					methodName = request.getParameter(mapping.getParameter());
				}
				Class[] args = {ActionMapping.class,ActionForm.class,HTTPServletRequest.class,HTTPServletResponse.class};
				try{
				Method method = action.getClass().getMethod(methodName, args);
				return method;
				}catch(Exception e){
					throw new RuntimeException(e);
				}
			}

	* 判断调用的方法上有没有注解

			Permission p = method.getAnnotation(Permission.class);
			if(p == null){
			return super.....;
			}else{
			//检查用户是否登录,是否有权限...

			}
	 		

3. 在配置文件总进行配置controller

## log4j的使用
1. 导入jar包
	然后将log4j.properties放在src下.可以配置log的输出级别,相互出样式,路径,输出的追加方法
2. 得到Logger对象
	`Logger log = Logger.getLogger(Demo.class);`

## struts数据库管理模块
1. 设计一个数据库类Database

		public class Database{
			private String id;
			private String name;
			private String path;
			private Date backuptime;
			...
		}

2. 然后编写DatabaseDaoImpl
3. 创建数据库
4. 编写Service层中的方法backupDatabase()方法
5. 调用java中使用Windows cmd的方法备份数据库
	
		String filename = path + "\\" + "bookstore" + number + ".sql";//得到数据库保存文件路径
		String cmd = "cmd /c mysqldump -uroot -phn123456 bookstore>" + filename;
		//导入命令是 "cmd /c mysql -uroot -phn123456 bookstore<" + path + "\\" + filename;
		Runtime.getRuntime().exec(cmd);

6. 事务管理
	* 在JdbcUtils中添加事务管理的方法

			public void deleteDatabase(String id){
				//删除数据库还要删除文件
				try{
					JdbcUtils.startTransaction();
					Database db = dao.find(id);
					ddao.delete(id);

					String filename = ...
					File file = new File(filename);
					if(file.exists()){
						file.delete();
					}
					JdbcUtils.commit();
				}catch(Exception e){
					JdbcUtils.rollback();
				}finally{	//很重要,解除线程上绑定的连接
					JdbcUtils.close();
				}
			}

	



