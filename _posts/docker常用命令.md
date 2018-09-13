---
title: docker常用命令
categories:
  - 运维相关
tags:
  - docker
date: 2018-07-07 19:51:55
---
 docker常用命令
 <!-- more -->

1. 关于容器的使用
* 创建容器
`
docker run -d -P training/webapp python app.py
`
参数说明:
>
-d:让容器在后台运行。
-P:将容器内部使用的网络端口映射到我们使用的主机上。



* 创建并启动容器
`
docker run --name tomcat_test -p 8080:8080 -v $PWD/webapps:/usr/local/tomcat/webapps -d tomcat  
`
参数说明:
>
-p 8080:8080：将主机的8080端口映射到容器的8080端口(冒号后面的是容器的端口)
-v $PWD/test:/usr/local/tomcat/webapps/test：将主机中当前目录下的test挂载到容器的/test

* 启动容器
`
docker start containerID
`

* 停止所有容器
`
docker stop $(docker ps -q)
`

* 删除容器
`
docker rm determined_swanson 
`

* 删除镜像 删除镜像之前要先删除镜像的容器
`
docker rmi image_name
`

* 通过push备份容器
`
docker commit -p containername container-backup
docker login
`
>
username:mending
`
docker tag containername mending/container-backup
docker push mending/container-backup
`

* 通过save保存到本地
`
docker save -o ~/container-backup.tar container-backup
`

* 通过主机命令行进入master容器
`
docker exec -it mysql_test bash
[root@bogon ~]# docker exec -it mysql-master bash
root@1651d1cab219:/#
`

* 创建mysql容器
先要关闭本机上的mysql服务
stop mysql
`
docker run --restart=always --name mysql_test -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root -d mysql:latest
`
>
–restart :参数来设置容器开机启动。 
no-container：不重启 
on-failure-container：退出状态非0时重启 
always：始终重启



* 创建mysql容器
`
sudo docker run -d -p 3306:3306 --name mysql_test -e MYSQL_ROOT_PASSWORD=root -v /mysql/datadir:/var/lib/mysql -v /mysql/conf:/etc/mysql/conf.d  docker.io/mysql:latest  
`
参数说明:
>
-e MYSQL_ROOT_PASSWORD=password ：指定root密码  
-v /mysql/datadir:/var/lib/mysql ：指定数据库本地存储路径，如果系统没有关闭SELinux，会启动失败，原因是本地目录不允许挂载到容器，需要先执行chcon -Rt svirt_sandbox_file_t /mysql/datadir  
-v /mysql/conf:/etc/mysql/conf.d ：指定使用自定义的mysql配置文件启动数据库，比如在该路径下创建一个my-config.cnf  

2. docker中配置Oracle

* 创建Oracle容器
`
docker run --name oracle_test -d -p 49160:22 -p 49191:1521 -e ORACLE_ALLOW_REMOTE=true daocloud.io/ihypo/oracle-xe-11g
`

* 进入oracle容器
`
docker exec -it 806ebe7f5231  /bin/bash
`

* ssh进入oracle容器
`
ssh root@localhost -p 49160
`

* 容器中登录Oracle数据库
`
su - oracle
sqlplus system/oracle
`

* 创建用户
`
create user HZYXY_BI identified by HZYXY_BI;
`
>
by 后面是密码

* 授予权限
`
grant resource,connect to HZYXY_BI;
`

* 切换用户登录
`
conn HZYXY_BI/HZYXY_BI;
`

* 容器中存放数据库文件的目录
/u01/app/oracle/oradata/XE/

create user shyhex identified by shyhex default tablespace shyhex

* 容器开机自动启动方法
1. 如果容器没有创建
`
sudo docker run --restart=always .....
`
2. 如果容器已经创建
`
docker update --restart=always xxx
`
	* no – 容器退出时不要自动重启。这个是默认值。
	* on-failure[:max-retries] – 只在容器以非0状态码退出时重启。可选的，可以退出docker daemon尝试重启容器的次数。
	* always – 不管退出状态码是什么始终重启容器。当指定always时，docker daemon将无限次数地重启容器。容器也会在daemon启动时尝试重启，不管容器当时的状态如何。
	* unless-stopped – 不管退出状态码是什么始终重启容器，不过当daemon启动时，如果容器之前已经为停止状态，不要尝试启动它。





