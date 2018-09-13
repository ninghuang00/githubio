---
title: nginx入门
categories:
  - 服务器
tags:
  - nginx
  - 反向代理
  - 负载均衡
date: 2018-09-09 17:06:42
---
 这是摘要
 <!-- more -->

## 配置详解
{% link_post nginx配置详解 %}


## 反向代理
简单说就是通过代理服务器,对客户端的访问隐藏了真正的服务器.真实的服务器不能直接被外部网络访问，所以需要一台代理服务器，而代理服务器能被外部网络访问的同时又跟真实服务器在同一个网络环境，当然也可能是同一台服务器，端口不同而已。 
### nginx简单配置
```java
server {  
        listen       80;                                                         
        server_name  localhost;                                               
        client_max_body_size 1024M;

        location / {
            proxy_pass http://localhost:8080;
            proxy_set_header Host $host:$server_port;
        }
    }
```
以上配置,访问localhost相当于访问localhost:8080

## 负载均衡
### 自带的三种策略
1. RR(默认)
按照时间顺序(轮询)将请求分配到后端服务器,如果后端服务器down掉了,会自动剔除
```java
    upstream test {
        server localhost:8080;
        server localhost:8081;
    }
    server {
        listen       81;                                                         
        server_name  localhost;                                               
        client_max_body_size 1024M;

        location / {
            proxy_pass http://test;
            proxy_set_header Host $host:$server_port;
        }
    }
```
这里配置了2台服务器，当然实际上是一台，只是端口不一样而已，而8081的服务器是不存在的,也就是说访问不到，但是我们访问http://localhost 的时候,也不会有问题，会默认跳转到http://localhost:8080 具体是因为Nginx会自动判断服务器的状态，如果服务器处于不能访问（服务器挂了），就不会跳转到这台服务器，所以也避免了一台服务器挂了影响使用的情况，由于Nginx默认是RR策略，所以我们不需要其他更多的设置。

### 参考配置
nginx实现请求的负载均衡 + keepalived实现nginx的高可用
>https://www.cnblogs.com/youzhibing/p/7327342.html 

LVS + keepalived + nginx + tomcat 实现主从热备 + 负载均衡
>http://www.cnblogs.com/youzhibing/p/5061786.html

主从热备+负载均衡（LVS + keepalived）
>http://www.cnblogs.com/youzhibing/p/5021224.html

2. 权重
按照分配的权重转发请求
```java
 upstream test {
        server localhost:8080 weight=9;
        server localhost:8081 weight=1;
    }
```

3. ip_hash
```java
    upstream test {
        ip_hash;
        server localhost:8080;
        server localhost:8081;
    }
```

4. 热备
当localhos:8080发生故障时,才会用8081
```java
    upstream test {
        ip_hash;
        server localhost:8080;
        server localhost:8081 backup;
    }
```

使用默认和权重方式会有一个问题,就是应用程序有时候不是无状态的,比如使用session保存登录信息,就需要连续访问同样的服务器才行,这个时候可以是使用ip_hash转发请求

### 第三方策略
1. fair
```java
upstream backend { 
        fair; 
        server localhost:8080;
        server localhost:8081;
    }
```
根据后端服务器响应的时间选择

2. url_hash
按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，后端服务器为缓存时比较有效。
在upstream中加入hash语句，server语句中不能写入weight等其他的参数，hash_method是使用的hash算法
```java
    upstream backend { 
        hash $request_uri; 
        hash_method crc32; 
        server localhost:8080;
        server localhost:8081;
    }
```

## 动静分离
动静分离是让动态网站里的动态网页根据一定规则把不变的资源和经常变的资源区分开来，动静资源做好了拆分以后，我们就可以根据静态资源的特点将其做缓存操作，这就是网站静态化处理的核心思路
```java
upstream test{  
       server localhost:8080;  
       server localhost:8081;  
    }   

    server {  
        listen       80;  
        server_name  localhost;  

        location / {  
            root   e:\wwwroot;  
            index  index.html;  
        }  

        # 所有静态请求都由nginx处理
        location ~ \.(gif|jpg|jpeg|png|bmp|swf|css|js)$ {  
            root    e:\wwwroot;  
        }  

        # 所有动态请求都转发给tomcat处理  
        location ~ \.(jsp|do)$ {  
            proxy_pass  http://test;  
        }  

        error_page   500 502 503 504  /50x.html;  
        location = /50x.html {  
            root   e:\wwwroot;  
        }  
    }

```
这样我们就可以吧HTML以及图片和css以及js放到wwwroot目录下，而tomcat只负责处理jsp和请求，例如当我们后缀为gif的时候，Nginx默认会从wwwroot获取到当前请求的动态图文件返回，当然这里的静态文件跟Nginx是同一台服务器，我们也可以在另外一台服务器，然后通过反向代理和负载均衡配置过去就好了，只要搞清楚了最基本的流程，很多配置就很简单了，另外localtion后面其实是一个正则表达式，所以非常灵活





## nginx命令
```sh
/etc/init.d/nginx start/restart # 启动/重启Nginx服务

/etc/init.d/nginx stop # 停止Nginx服务

/etc/nginx/nginx.conf # Nginx配置文件位置

nginx -s reload # 重新加载配置文件

```

