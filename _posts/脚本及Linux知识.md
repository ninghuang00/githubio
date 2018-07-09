---
title: 脚本及Linux知识
categories:
  - 笔记
tags:
  - Linux 
  - shell
  - centos 
  - ubuntu
date: 2018-07-07 20:49:09
---
 这是摘要
 <!-- more -->

# centos常用命令
## awk命令
### 命令查找日志
* 查找访问日志如 tcp 0 0 127.0.0.1:8652 127.0.0.1:40192 TIME_WAIT格式文件,IP最多的前五个IP
awk '{print $5}' apache.log|cut -d:-f1|sort|uniq -c|sort -nr|head -n 5

## 查看进程
`ps -ef|grep tomcat`

## 查看端口情况
`netstat -pan|grep 1099`
```sh
//常用参数:
-a 	显示所有连接和监听的端口
-b 	显示包含于创建每个连接或监听端口的可执行组件。在某些情况下已知可执行组件
-e 	显示以太网统计信息。此选项可以与 -s选项组合使用。
-n  以数字形式显示地址和端口号。
-o 	显示与每个连接相关的所属进程 ID。
-p proto 	显示 proto 指定的协议的连接；proto 可以是
```
## 修改密码
`passwd username`

## 添加用户


连接远程主机其他接口
`ssh root@ip -p port`

## 查看系统版本信息
`uname -a`

## filename.zip的解压
`unzip filename.zip`

## 解压文件
`tar -zxvf filename.tar.gz`
```sh
//其中zxvf含义分别如下
-c: 建立压缩档案
-x：解压
-t：查看内容
-r：向压缩归档文件末尾追加文件
-u：更新原压缩包中的文件
//这五个是独立的命令，压缩解压都要用到其中一个，可以和别的命令连用但只能用其中一个。下面的参数是根据需要在压缩或解压档案时可选的。
-z：有gzip属性的
-j：有bz2属性的
-Z：有compress属性的
-v：显示所有过程
-O：将文件解开到标准输出
//下面的参数-f是必须的
-f: 使用档案名字，切记，这个参数是最后一个参数，后面只能接档案名。
//事实上, 从1.15版本开始tar就可以自动识别压缩的格式,故不需人为区分压缩格式就能正确解压
tar -xvf filename.tar.gz
tar -xvf filename.tar.bz2
tar -xvf filename.tar.xz
tar -xvf filename.tar.Z
```

## 解压文件到指定文件夹
`tar -zxvf jdk-8u161-linux-i586.tar.gz -C java`


# centos7配置环境

## 配置java环境
1. 先将jdk的压缩包上传到服务器`scp jdk.tar.gz root@xx.xxx.xxx.xxx:/usr/local/java/`
2. 然后使用`tar -zxvf jdk.tar.gz`解压
3. 然后配置`vim /etc/profile`,添加以下内容到底部
```sh
		JAVA_HOME=/usr/local/java/jdk
        PATH=$JAVA_HOME/bin:$PATH
        CLASSPATH=$JAVA_HOME/jre/lib/ext:$JAVA_HOME/lib/tools.jar
        export PATH JAVA_HOME CLASSPATH
```
4. 然后`source /etc/profile` 

## 查看所有端口
`netstat -ntlp`
## 查看端口的开启情况
`netstat`
## 查看指定端口的占用情况
`netstat -lnp|grep 8080`
	
## 检测远程主机端口是否打开
`telnet 120.79.202.146 8080`

## centos7启动firewalld配置防火墙
1. 确定打开firewalld`systemctl restart firewalld`
2. 查看firewalld状态,版本等`firewall-cmd --state`
3. 查看已经打开的port`firewall-cmd --zone=dmz --list-ports`
4. 开启端口`firewall-cmd --zone=public --add-port=1099/tcp --permanent`
```sh
命令含义：
–zone #作用域
–add-port=80/tcp #添加端口，格式为：端口/通讯协议
–permanent #永久生效，没有此参数重启后失效
```
5. 重启防火墙
```sh
firewall-cmd --reload 				#重启firewall
systemctl stop firewalld.service 	#停止firewall
systemctl disable firewalld.service #禁止firewall开机启动
```
6. 注意阿里云服务器还要在控制中心添加端口规则

## 将idea web项目部署到远程tomcat,以及开机启动tomcat

1. 配置`tomcat/bin/catalina.sh`文件,首行加入
```sh
		export CATALINA_OPTS="-Dcom.sun.management.jmxremote 
		-Dcom.sun.management.jmxremote.port=1099 
		-Dcom.sun.management.jmxremote.ssl=false 
		-Dcom.sun.management.jmxremote.authenticate=false 
		-Djava.rmi.server.hostname=10.82.82.248"
		export JAVA_OPTS="-Dcom.sun.management.jmxremote=
		-Dcom.sun.management.jmxremote.port=1099
		-Dcom.sun.management.jmxremote.ssl=false
		-Dcom.sun.management.jmxremote.authenticate=false"
```
2. 启动tomcat
`sh
./catalina.sh run > /dev/null 2>&1 &
`
其中`> /dev/null 2>&1 &`是Linux中的命令：把标准输出和出错处理都放到回收站，这样就免得一大堆输出占领你的屏幕。
3. 在用jps命令：

4. javahome: /usr/lib/jvm/java-8-oracle

5. 设置tomcat开机启动,将startup.sh修改
```sh
export JAVA_HOME=/usr/local/java/jdk1.8
export CLASSPATH=$CLASSPATH:$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib/dt.jar:.  
export PATH=$JAVA_HOME/bin:$PATH  
export CATALINA_HOME=/usr/local/tomcat  
/usr/local/tomcat/bin/catalina.sh start
```

6. 在/etc/rc.d/rc.local中加入`/usr/local/tomcat/bin/startup.sh`

## docker开机自动启动
`systemctl  enable docker.service`

## 使用iptables管理防火墙
`vi /etc/sysconfig/iptables`配置端口
`service iptables restart`重启防火墙
`systemctl restart iptables.service`重启防火墙
`/etc/init.d/iptables restart`重启防火墙
`systemctl enable iptables.service`设置防火墙卡机启动


# tomcat配置
## idea远程debug
1. centos环境,在startup.sh首行输入
```sh 
declare -x CATALINA_OPTS="-server -Xdebug -Xnoagent -Djava.compiler=NONE -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=5005"
```
2. 在Ubuntu环境中,startup.sh首行输入
```sh 
CATALINA_OPTS="-server -Xdebug -Xnoagent -Djava.compiler=NONE -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=5005"
```

## idea远程部署
1. 配置tomcat/bin/catalina.sh文件
```sh 
export CATALINA_OPTS="-Dcom.sun.management.jmxremote 
-Dcom.sun.management.jmxremote.port=1099 
-Dcom.sun.management.jmxremote.ssl=false 
-Dcom.sun.management.jmxremote.authenticate=false 
-Djava.rmi.server.hostname=10.82.82.248"

export JAVA_OPTS="-Dcom.sun.management.jmxremote=
-Dcom.sun.management.jmxremote.port=1099
-Dcom.sun.management.jmxremote.ssl=false
-Dcom.sun.management.jmxremote.authenticate=false"
```
2. 启动tomcat
```sh 
./catalina.sh run > /dev/null 2>&1 &
```
其中“ > /dev/null 2>&1 &”是Linux中的命令：把标准输出和出错处理都放到回收站，这样就免得一大堆输出占领你的屏幕。
3. 在用jps命令：

4. javahome: /usr/lib/jvm/java-8-oracle

5. 设置tomcat开机启动,将startup.sh修改
```sh 
export JAVA_HOME=/usr/lib/jvm/java-8-oracle  
export CLASSPATH=$CLASSPATH:$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib/dt.jar:.  
export PATH=$JAVA_HOME/bin:$PATH  
export CATALINA_HOME=/usr/local/tomcat  
/usr/local/tomcat/bin/catalina.sh start
```

6. 在/etc/rc.d/rc.local中加入`/usr/local/tomcat/bin/startup.sh`

