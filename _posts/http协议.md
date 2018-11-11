---
title: http协议
categories:
  - 网络
tags:
  - http 
  - https
  - CSRF 
date: 2018-07-28 17:03:53
---
 这是摘要
 <!-- more -->

### 特点
1. 简单快速
请求方法非常简单,只需要传送请求方法和路径
2. 灵活
可以传输任意类型的数据,使用content-type标记
3. 无连接
每次连接仅处理一个请求,服务器处理完客户端的请求并收到应答后即断开连接,节省传输时间
http1.1版本已经默认使用长连接,使用`Connection: close`关闭
4. 无状态
不记录请求的状态,加快应答

### http协议
#### http请求方法
1. GET(幂等,无副作用,不能带Request Body)
获取资源
2. POST(非幂等,有副作用,能带Request Body)
新建或者更新资源
RFC文档例举的使用场景:
```java
- Annotation of existing resources;
- Posting a message to a bulletin board, newsgroup, mailing list, or   
  similar group of articles;
- Providing a block of data, such as the result of submitting a
  form, to a data-handling process;
- Extending a database through an append operation.
```
3. PUT(幂等,有副作用,能带Request Body)
更新资源,如修改帖子,同一个修改不管提交几次都是一样的结果
4. DELETE(幂等,有副作用,不能带Request Body)
删除资源
5. PATCH()
更新资源的一部分
6. HEAD
HEAD方法跟GET方法相同，只不过服务器响应时不会返回消息体。一个HEAD请求的响应中，HTTP头中包含的元信息应该和一个GET请求的响应消息相同。这种方法可以用来获取请求中隐含的元信息，而不用传输实体本身。也经常用来测试超链接的有效性、可用性和最近的修改。

#### 关于GET,POST,PUT的区别
1. 很多浏览器对GET请求的URI长度有限制
2. GET请求是安全的(对服务器而言),因为不会改变什么
3. 

#### 请求和响应
http1.0规定请求头或者响应头的数据格式必须是ASCII码,后面的数据可以是任意类型
1. get请求
```java
//请求行
GET / HTTP/1.1		//第一行指定请求方法和协议版本号,后面的都是描述客户端情况

//请求头
Transfer-Encoding: chunked
Accept-Encoding: gzip, deflate 		//指定可以接受的压缩类型
Accept-Language: zh-CN,zh;q=0.8
Accept-Charset: GBK,utf-8;q=0.7,*;q=0.3
Connection: close	//明确要求服务器关闭tcp连接,在http1.0还只有短连接的时候请求头中需要声明keep-alive来保持连接
Accept: */*	 		//指定客户端可以接收的数据类型
```

2. post请求
```java
POST / HTTP1.1 //请求行
//请求头
Host:www.wrox.com
User-Agent:Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; .NET CLR 2.0.50727; .NET CLR 3.0.04506.648; .NET CLR 3.5.21022)
Content-Type:application/x-www-form-urlencoded
Content-Length:40
Connection: Keep-Alive
//数据
name=Professional%20Ajax&publisher=Wiley
```

3. 响应
```java
//响应行
HTTP/1.1 200 OK //第一行是协议版本和状态码

//响应头
Content-Type: text/plain	//说明response-body中的数据格式
Content-Length: 137582		//本次回应的数据长度,1.1中一个tcp连接可以回应多个请求
Transfer-Encoding: chunked	//分块传输,返回数据长度不定
Server: Apache 0.84			//服务器类型
Content-Encoding: gzip		//说明数据的压缩类型

//响应数据
<html>
  <body>Hello World</body>
</html>
```

## 常见http协议状态码
```
1xx：指示信息--表示请求已接收，继续处理
2xx：成功--表示请求已被成功接收、理解、接受
3xx：重定向--要完成请求必须进行更进一步的操作
4xx：客户端错误--请求有语法错误或请求无法实现
5xx：服务器端错误--服务器未能实现合法的请求
```
```java
200 OK - [GET]：服务器成功返回用户请求的数据，该操作是幂等的（Idempotent）。
201 CREATED - [POST/PUT/PATCH]：用户新建或修改数据成功。
202 Accepted - [*]：表示一个请求已经进入后台排队（异步任务）
204 NO CONTENT - [DELETE]：用户删除数据成功。
400 INVALID REQUEST - [POST/PUT/PATCH]：用户发出的请求有错误，服务器没有进行新建或修改数据的操作，该操作是幂等的。
401 Unauthorized - [*]：表示用户没有权限（令牌、用户名、密码错误）。
403 Forbidden - [*] 表示用户得到授权（与401错误相对），但是访问是被禁止的。
404 NOT FOUND - [*]：用户发出的请求针对的是不存在的记录，服务器没有进行操作，该操作是幂等的。
406 Not Acceptable - [GET]：用户请求的格式不可得（比如用户请求JSON格式，但是只有XML格式）。
410 Gone -[GET]：用户请求的资源被永久删除，且不会再得到的。
422 Unprocesable entity - [POST/PUT/PATCH] 当创建一个对象时，发生一个验证错误。
500 INTERNAL SERVER ERROR - [*]：服务器发生错误，用户将无法判断发出的请求是否成功。
```

## 会话跟踪
1. URL重写
通过URL带上会话ID
2. 隐藏表单域
将会话ID添加到HTML表单提交到服务器
3. cookie
客户端可以保存在内存中(临时cookie),也可以保存到硬盘上(有效期cookie)
4. session
session依赖于cookie

# Https
## aes与sra加密
1. aes是对称加密,秘钥只有一个,一般是256bit,
优点是加密速度快,
缺点是在不安全的网络中传输容易别窃取
2. sra是非对称加密,分为公钥和私钥,公钥在网络中传输,私钥安全保存,
优点是网络传输更安全,
缺点是加密速度相对较慢
3. 结合两者:
将公钥给对方,让对方用公钥加密aes的秘钥,然后发回给自己,自己通过私钥解密获得aes秘钥后加密数据发给对方,对方就能用公用的aes秘钥解密,
这样既能保证加密速度,又能保证秘钥传输安全

## 非对称加密
1. 公钥加密,私钥解密
2. 私钥签名,公钥认证
3. 其实公钥私钥可以相互解密

## SSL层
实现安全连接

## Socket通信
1. java封装类
2. 阻塞和非阻塞
	1. Java服务器网络开发时， 请说明通讯中阻塞（blocking）/非阻塞（non-blocking 与 同步/异步IO的区别。
	    1. 同步／异步主要针对客户端：
	        同步：就是当客户端发出一个功能调用时，在没有得到结果之前，该调用就不返回。也就是说必须一件一件的事情去做，等一件做完了才能去做下一件。
	        异步：就是当客户端发出一个功能调用时，调用者不用等接收方发出响应。实际处理这个调用的部件在完成后，会通过状态，通知和回调来通知调用者。客户端可以接着去做 后面的事情。
	        虽然主要是针对客户端，但是服务器端不是完全没有关系的，同步／异步必须配合服务器端才能实现。同步／异步是由客户端自己控制，但是服务器端是否阻塞/非阻塞，客户端完全不需要关心。
	    2. 阻塞／非阻塞主要是针对服务器端：
	        阻塞：阻塞调用是指服务器端被调用者调用结果返回之前，当前线程会被挂起。调用线程只有在得到结果之后才会返回。
	        非阻塞：指在不能立即得到结果之前，该调用不会阻塞当前线程。
	        
{% post_link NIO的使用 %}

## HTTPS 握手机制：
1. 客户端 向服务器发起请求。
2. 服务端 取出公有密钥及证书并发送给客户端。
3. 客户端判断公有密钥是否有效，无效则显示警告。有效则生成一个随机数串，并以此生成客户端的对称密钥。
4. 用步骤 2 到的公有密钥对该随机数串加密，发送到服务器。
5. 服务器得到加密报文，用私有密钥解密报文，得到随机数串，并以此生成服务器端的对称密钥。此时客户端和服务端拥有相同的对称密钥，可以用该对称密钥进行安全通信。
6. 服务器对响应进行加密，客户端对报文进行解密。

## 跨站请求伪造
CSRF:Cross—Site Request Forgery
参考地址:http://blog.csdn.net/stpeace/article/details/53512283
过程:
1. me登录网易邮箱
2. hack制作了钓鱼网站
3. me在没有登出邮箱的时候访问了钓鱼网站
4. 于是钓鱼网站通过me的浏览器访问了网易邮箱,带着me的cookie(此时me的浏览器与网易邮箱网站服务器之间的session尚未过期)进行了一些操作

### 防御CSRF攻击
1. 验证HTTP referer字段
HTTP referer指向前一个网页的URL,而hack要发起跨站请求伪造的话只能通过他的钓鱼网站发起,合法的HTTP referer一般都是网易邮箱的页面的URL.但是这种方法依赖于浏览器的安全性和设置,浏览器可以设置成不带上referer值
2. 在请求地址中添加token并验证
服务器生成token发给客户端,然后客户端发起请求的时候带上token
GET将token放在URL后面
POST则将请求放在form标签最后加上`<input type="hidden" name="token" value="tokenValue">` 
但是一些论坛网站中token很难保证安全,以为用户可以自己发帖子,然后帖子中的链接就会带上token
3. 在HTTP头中增加自定义属性并验证

## SSL证书 
1. 域名型SSL证书(DVSSL Domain Validation SSL)
	审核内容:域名 管理权限
	证书详情:使用者身份只显示某某域名
	一般用途:个人站点 IOS应用分发下载 登录等纯HTTPS加密需求的链接
2. 企业型SSL(OVSSL Organization Validation SSL)
	审核内容:域名管理权限 企业名称,地址,电话等信息的真实性
	证书详情:使用证身份显示公司名称
	一般用途:企业站点
3. 增强型SSL(EVSSL extend Validation SSL)
	审核内容:域名管理权限 企业名称地址电话等信息的真实性 第三方数据库的审查,例如邓白氏,114查号台,律师证明等
	证书详情:显示使用者显示公司名称
	一般用途:企业官网,电商,P2P等互联网金融网站



