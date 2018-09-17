---
title: nginx配置详解
categories:
  - 笔记
tags:
  - null
date: 2018-09-10 22:55:18
---
 这是摘要
 <!-- more -->


# 配置文件结构
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
2. upstream 
还能够为每一个设备设置状态值，这些状态值的含义分别例如以下：
  * down 
  表示单前的server临时不參与负载.
  * weight 
  weight越大，负载的权重就越大。
  * max_fails 
  同意请求失败的次数默觉得1.当超过最大次数时，返回proxy_next_upstream 模块定义的错误.
  * fail_timeout 
  max_fails次失败后。暂停的时间。
  * backup 
  其他全部的非backup机器down或者忙的时候，请求backup机器。所以这台机器压力会最轻。

# 配置文件模板
```sh

#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
    	# 声明服务器监听端口
        listen       80;
        # 就是http://后面的域名
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        # 静态页面目录
        root /Users/huangning/Desktop/IJProject/iBase4J/iBase4J-UI;
        # 默认首页
        index index.html

        # 相当于http://127.0.0.1:8090/education/xxxx.jpg
		location ~ \.(html|js|css|png|jpg|gif) {
    		    # 静态页面目录
	        root /usr/local/tomcat/webapps/education;
        }

		# 相当于http://127.0.0.1:8090/education
		location /education {
			# 动态页面,交给tomcat处理
            proxy_pass http://127.0.0.1:8090;
		}
		
		# 相当于http://127.0.0.1:8090/education/login.action
		location ~ \.(jsp|do|action) {
			# 动态页面,交给tomcat处理
            proxy_pass http://127.0.0.1:8090;
		}

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        # 如果被代理服务器返回的状态码为400或者大于400，设置的error_page配置起作用。默认为off。
        # proxy_intercept_errors on;
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apaches document root
        # concurs with nginxs one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}
```