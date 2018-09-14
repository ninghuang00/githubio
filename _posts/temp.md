---
title:temp
categories:
  - 笔记
tags:
  - temp
date: 2018-08-18 09:46:59
---
 这是摘要
 <!-- more -->

## events块

## http块
### server块
1. 配置https
```
server {
    listen       8443;        #nginx对外提供服务的https端口
    include /etc/nginx/default.d/*.conf;
    ssl on;        #配置ssl
    ssl_certificate edu.crt;        #ssl 密钥， edu.crt放到/etc/nginx目录
    ssl_certificate_key edu.key;   #ssl 密钥， edu.key放到/etc/nginx目录
    ssl_session_timeout 5m;
    #ssl_verify_client off;
    ssl_protocols SSLv3 TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers "HIGH:!aNULL:!MD5 or HIGH:!aNULL:!MD5:!3DES";
    ssl_prefer_server_ciphers on;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }
}
```
#### location块
1. 获取用户真实ip
>参考地址:https://blog.csdn.net/bao19901210/article/details/52537279

```
location /edu-ui {
    # 携带用户真实ip 
    proxy_set_header X-Real-IP $remote_addr;
    # 携带用户真实ip 
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    # 携带域名
    proxy_set_header Host $host;
    # 
    proxy_set_header X-NginX-Proxy true;说明开启Nginx代理
}
```

### upstream块
1. 基本配置
```
upstream bakend{ #定义负载均衡设备的Ip及设备状态 
      ip_hash; 
      server 10.0.0.11:9090 down; 
      server 10.0.0.11:8080 weight=2 max_fails=3 fail_timeout=10s; 
      server 10.0.0.11:6060 weight=1 max_fails=3 fail_timeout=10s; 
      server 10.0.0.11:7070 backup; 
}
```
upstream 还能够为每一个设备设置状态值，这些状态值的含义分别例如以下：

down 表示单前的server临时不參与负载.

weight weight越大，负载的权重就越大。

max_fails 同意请求失败的次数默觉得1.当超过最大次数时，返回proxy_next_upstream 模块定义的错误.

fail_timeout max_fails次失败后。暂停的时间。

backup 其他全部的非backup机器down或者忙的时候，请求backup机器。所以这台机器压力会最轻。





