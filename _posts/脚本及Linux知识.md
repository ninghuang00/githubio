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
 
 # shell脚本
 ## wget使用
 1. 从有数字规律的网址抓取网页并保存在当前目录？假设网址为 http://test/0.xml，其中这个数字可以递增到100。(网易笔试题)
 ```shell
for((i=0;i<100;++i));
do
wget http://test/$i.xml; 
done
```
 
 ## shell加法
 {% asset_img shell加法.png shell加法%}
 ## source命令
 {% asset_img source命令.png source命令%}
 ## 替换文本文件
 {% asset_img 替换文本文件.png 替换文本文件%}
 ## 换行符格式
 {% asset_img 换行格式.png 换行格式%}
 ## 管道
 那么，一种思路就是把你tail输出的东西再做一次包装处理，这个很符合linux管道处理的思想。以高亮Log中的ERROR为例，你可以这样：
 `tail -f xxx.log | perl -pe 's/(ERROR)/\e[1;31m$1\e[0m/g'`
 其中，xxx.log是你要跟踪的文件。这里假设了你的Linux的PATH中有perl。perl在这里干的事情，
 就是通过命令行的方式进行动态的替换ERROR字符串的操作，替换过程中，主要使用了Linux的console_codes的语法结构。
 （具体关于console_codes的细节，可以通过man console_codes进行了解）这里，\e主要进行转移说明。
 
 
 # 正则表达式
 ## 匹配数字
 1. 匹配1~1000
 `^([1-9][0-9]{0,4}|1000)$`
 

# centos常用命令
## 查找文件
### whereis：
whereis命令用来定位指令的二进制程序、源代码文件和man手册页等相关文件的路径。
whereis命令只能用于程序名的搜索，而且只搜索二进制文件（参数-b）、man说明文件（参数-m）和源代码文件（参数-s）。如果省略参数，则返回所有信息。

### locate：
locate命令和slocate命令都用来查找文件或目录。
locate命令其实是find -name的另一种写法，但是要比后者快得多，原因在于它不搜索具体目录，而是搜索一个数据库/var/lib/locatedb，这个数据库中含有本地所有文件信息。Linux系统自动创建这个数据库，并且每天自动更新一次，所以使用locate命令查不到最新变动过的文件。为了避免这种情况，可以在使用locate之前，先使用updatedb命令，手动更新数据库。

### which：
which命令用于查找并显示给定命令的绝对路径，环境变量PATH中保存了查找命令时需要遍历的目录。which指令会在环境变量$PATH设置的目录里查找符合条件的文件。也就是说，使用which命令，就可以看到某个系统命令是否存在，以及执行的到底是哪一个位置的命令。

### type：
type命令用来显示指定命令的类型，判断给出的指令是内部指令还是外部指令。
命令类型：
```
alias：别名。
keyword：关键字，Shell保留字。
function：函数，Shell函数。
builtin：内建命令，Shell内建命令。
file：文件，磁盘文件，外部命令。
unfound：没有找到。
-t：输出“file”、“alias”或者“builtin”，分别表示给定的指令为“外部指令”、“命令别名”或者“内部指令”；
-p：如果给出的指令为外部指令，则显示其绝对路径；
-a：在环境变量“PATH”指定的路径中，显示给定指令的信息，包括命令别名。
[root@localhost ~]# type date date is /bin/date [root@localhost ~]# type mysql mysql is /usr/bin/mysql
因为显示的是路径，可以理解为找到了这个文件（个人理解）。
```
### find：
参考地址:https://www.cnblogs.com/ay-a/p/8017419.html
find命令用来在指定目录下查找文件。任何位于参数之前的字符串都将被视为欲查找的目录名。如果使用该命令时，不设置任何参数，则find命令将在当前目录下查找子目录与文件。并且将查找到的子目录和文件全部进行显示。
`find -name "dxt-core" -type d`
**find基本用法**
```java
find 如不加任何参数，表示查找当前路径下的所有文件和目录
find  -print    将结果打印到标准输出
find /data/log   指定路劲查找
find   /   -name  "abc.txt"   在系统中查找 abc.txt 如果执行完毕没有找到，则说明系统中不存在该文件
find 还支持正则表达式查找
find /data/logs -mame "*.log"  -type f -printf    查找符合指定字符串的文件
find . -name "[0-9]" -type f   查找以数字开头的文件
find / -mtime -1 |head  查找系统内最近24小时修改过的文件
find / -mmin  -15|head   查找系统内最近15 分钟修改过的文件
find 使用 type 选项可以查找特定的文件类型，常见类型如下
　　b 块设备文件
　　d 目录
　　c 字符设备文件
　　p 管道文件
　　l 符号链接文件
　　f 普通文件
find  . -type d  查找当前路径中的所有目录
find  . -type f  查找当前路径中的所有文件
find  . -type l   查找当前路径中的所有符号链接文件
```
## 查看内存使用
`free -mh`

## 查看硬盘使用
可以查看一级文件夹大小、使用比例、档案系统及其挂入点，但对文件却无能为力
`df -h`

## 查看文件夹大小
可以查看文件及文件夹的大小
`du -h`
`du -h --max-depth 1 bin/Mdroid`

## 强制杀死进程
`kill -s 9 PID`

## 修改文件权限
可执行文件
`chmod 755 filename`
```java
sudo chmod -（代表类型）×××（所有者）×××（组用户）×××（其他用户）
rwx-rwx-rwx
sudo chmod 600 ××× （只有所有者有读和写的权限）
sudo chmod 644 ××× （所有者有读和写的权限，组用户只有读的权限）
sudo chmod 700 ××× （只有所有者有读和写以及执行的权限）
sudo chmod 666 ××× （每个人都有读和写的权限）
sudo chmod 777 ××× （每个人都有读和写以及执行的权限）
```

## grep命令
### 获取本机内网ip
`ifconfig eth0|grep "inet addr"|awk '{print $2}'|awk -F : '{print $2}'`
参考地址:https://www.cnblogs.com/kerrycode/archive/2015/06/16/4581030.html
### 常用参数
```java
-d skip 忽略子目录
-l pattern files 查询多文件时只列出匹配的文件名
-v 显示不包括匹配文本的所有行
-c 只输出匹配行的计数
-i 不区分大小写(只适用于单字符)
-h 查询多文件时不显示文件名
-n 显示匹配行及行号
-s 不显示不存在或无匹配文本的错误信息
-v 显示不包括匹配文件的所有行

```

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

