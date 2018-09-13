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