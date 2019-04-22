---
title: docker入门
categories:
  - 笔记
tags:
  - docker
date: 2018-09-25 22:04:39
---
 这是摘要
 <!-- more -->


## docker和虚拟机的区别
>参考地址:https://www.cnblogs.com/pangguoping/articles/5515286.html

## 部署tomcat
1. 搜索tomcat镜像
`docker search tomcat`
2. 获取镜像
`docker pull tomcat`
3. 启动容器
`docker run --name tomcat8091 -p 8092:8080 -v $PWD/temp:/usr/local/tomcat/temp -d docker.io/tomcat`
4. 访问端口号对应的ip,验证服务是否启动


## 部署redis
1. 搜索拉取
2. 启动容器
`docker run --name redis6380 -p 6380:6379 -v $PWD/data:/data -v $PWD/data/redis.config:/data/redis.config -d docker.io/redis redis-server /data/redis.config`
参数说明:
redis-server redis.config:在容器执行redis-server启动命令，以redis.config为配置文件,最好通过自定义的配置文件启动,因为默认配置文件是找不到的,无法修改
3. 进入容器查看redis状态
`docker exec -it redis redis-cli`


## 关于容器的使用

### 查看docker日志
`docker logs container_name`

### 创建并启动容器
`
docker run --name tomcat_test -p 8090:8080 -v $PWD/webapps:/usr/local/tomcat/webapps -d tomcat  
`
参数说明:
-name 为容器命名
-p 8090:8080：将宿主机的8090端口映射到容器的8080端口(冒号后面的是容器的端口)
-v $PWD/test:/usr/local/tomcat/webapps/test：将主机中当前目录下的test挂载到容器的/test

### 修改已经创建的容器的端口映射
>参考地址:https://blog.csdn.net/wesleyflagon/article/details/78961990

### 启动容器
`
docker start containerID
`

### 停止所有容器
`
docker stop $(docker ps -q)
`

### 删除容器
`
docker rm determined_swanson 
`

### 删除镜像(删除镜像之前要先删除镜像的容器)
`
docker rmi image_name
`

### 通过push备份容器
```
docker commit -p container_name container-backup
docker login
```
>
username:mending

```
docker tag container_name mending/container-backup
docker push mending/container-backup
```

### 通过save保存到本地
`
docker save -o ~/container-backup.tar container-backup
`

### 通过主机命令行进入master容器
```
docker exec -it mysql_test bash
[root@bogon ~]# docker exec -it mysql-master bash
root@1651d1cab219:/#
```

### 容器开机自动启动方法
1. 如果容器没有创建
`
sudo docker run --restart=always .....
`
2. 如果容器已经创建
`
docker update --restart=always container_name
`
	* no – 容器退出时不要自动重启。这个是默认值。
	* on-failure[:max-retries] – 只在容器以非0状态码退出时重启。可选的，可以退出docker daemon尝试重启容器的次数。
	* always – 不管退出状态码是什么始终重启容器。当指定always时，docker daemon将无限次数地重启容器。容器也会在daemon启动时尝试重启，不管容器当时的状态如何。
	* unless-stopped – 不管退出状态码是什么始终重启容器，不过当daemon启动时，如果容器之前已经为停止状态，不要尝试启动它。

## 创建mysql容器
`
sudo docker run -d -p 3306:3306 --name mysql_test -e MYSQL_ROOT_PASSWORD=root -v /mysql/datadir:/var/lib/mysql -v /mysql/conf:/etc/mysql/conf.d  docker.io/mysql:latest  
`
参数说明:
>
-e MYSQL_ROOT_PASSWORD=password ：指定root密码  
-v /mysql/datadir:/var/lib/mysql ：指定数据库本地存储路径，如果系统没有关闭SELinux，会启动失败，原因是本地目录不允许挂载到容器，需要先执行chcon -Rt svirt_sandbox_file_t /mysql/datadir  
-v /mysql/conf:/etc/mysql/conf.d ：指定使用自定义的mysql配置文件启动数据库，比如在该路径下创建一个my-config.cnf  

## docker中配置Oracle

1. 创建Oracle容器
`
docker run --name oracle_test -d -p 49160:22 -p 49191:1521 -e ORACLE_ALLOW_REMOTE=true daocloud.io/ihypo/oracle-xe-11g
`

2. 进入oracle容器
`
docker exec -it 806ebe7f5231  /bin/bash
`

3. ssh进入oracle容器
`
ssh root@localhost -p 49160
`

4. 容器中登录Oracle数据库
`
su - oracle
sqlplus system/oracle
`

5. 创建用户
`
create user HZYXY_BI identified by HZYXY_BI;
`
说明:
by 后面是密码

6. 授予权限
`
grant resource,connect to HZYXY_BI;
`

7. 切换用户登录
`
conn HZYXY_BI/HZYXY_BI;
`

8. 容器中存放数据库文件的目录
/u01/app/oracle/oradata/XE/
create user shyhex identified by shyhex default tablespace shyhex

