---
title: tcp和udp
categories:
  - 网络 
tags:
  - tcp 
  - udp
  - 流量控制
date: 2018-07-28 17:04:08
---
 这是摘要
 <!-- more -->


### tcp和udp比较
{% asset_img tcp和udp比较.png tcp和udp比较%}

### tcp协议
1. TCP支持的应用协议主要有：Telnet、FTP、SMTP,http,pop3等； 
2. 流量控制
参考地址:https://blog.csdn.net/yechaodechuntian/article/details/25429143
使用滑动窗口来控制流量,限制发送端可以发送的数据量
{% asset_img 滑动窗口控制流量.jpg 滑动窗口控制流量 %}

3. 拥塞控制
发送方控制拥塞窗口的原则是：只要网络没有出现拥塞，拥塞窗口就再增大一些，以便把更多的分组发送出去。但只要网络出现拥塞，拥塞窗口就减小一些，以减少注入到网络中的分组数。
    1. 慢开始和拥塞避免
        1. 当 cwnd < ssthresh 时，使用上述的慢开始算法。
        2. 当 cwnd > ssthresh 时，停止使用慢开始算法而改用拥塞避免算法。
        3. 当 cwnd = ssthresh 时，既可使用慢开始算法，也可使用拥塞控制避免算法。
{% asset_img 慢开始和拥塞避免.jpg 慢开始和拥塞避免 %}

    2. 快重传和快恢复
{% asset_img 快重传和快恢复.jpg 快重传和快恢复 %}

4. 应用场景
优点:可靠稳定,通过三次握手建立连接,有流量控制,差错控制等机制(握手 确认 窗口 拥塞控制 重传等)
缺点:传输速度慢,效率低,占用系统资源高,容易被攻击(DOS,DDOS,CC,通过大量建立无效连接来耗光服务器资源)
	1. 文件传输:FTP
	2. 邮件:SMTP
	3. putty:ssh,TELNET
	4. 浏览器:http
DOS攻击:denial of service
DDOS攻击:distributed denial of service	
参考地址:https://www.jianshu.com/p/dff5a0d537d8
5. 建立连接过程(3次握手)
	1. C发送一个请求连接的位码SYN和一个随机产生的序列号给Seq，然后S收到了这些数据。
    2. S收到了这个请求连接的位码，啊呀，有人向我发出请求了么，那我要不要接受他的请求，得实现确认一下，于是，发送了一个确认码 ACN（seq+1），和SYN，Seq给C，然后C收到了，这个是第二次连接。
    3. C收到了确认的码和之前发送的SYN一比较，偶哟，对上了么，于是他又发送了一个ACN（SEQ+1）给S，S收到以后就确定建立连接，至此，TCP连接建立完成。


### udp协议
1. UDP支持的应用层协议主要有：NFS（网络文件系统）、SNMP（简单网络管理协议）、DNS（主域名称系统）、TFTP（通用文件传输协议）等。
2. 应用场景
优点:因为是无状态传输协议,所以传输速度很快;因为没有各种控制机制,容易被利用的漏洞较少(但是也有UDP FLOOD攻击)
缺点:不稳定,不可靠,网络条件差的情况下容易丢包
	1. 在线视频:
	2. 语音通话
	3. TFTP
