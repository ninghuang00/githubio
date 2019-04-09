---
title: ss入门
categories:
  - 笔记
tags:
  - ss
date: 2018-12-10 13:02:09
---
 这是摘要
 <!-- more -->


### 启动服务
`/usr/bin/ssserver -c /etc/shadowsocks.json -d start`

### 相关参数
```
usage: ssserver [OPTION]...
A fast tunnel proxy that helps you bypass firewalls.

You can supply configurations via either config file or command line arguments.

Proxy options:
  -c CONFIG              path to config file
  -s SERVER_ADDR         server address, default: 0.0.0.0
  -p SERVER_PORT         server port, default: 8388
  -k PASSWORD            password
  -m METHOD              encryption method, default: aes-256-cfb
                         Sodium:
                            chacha20-poly1305, chacha20-ietf-poly1305,
                            xchacha20-ietf-poly1305,
                            sodium:aes-256-gcm,
                            salsa20, chacha20, chacha20-ietf.
                         Sodium 1.0.12:
                            xchacha20
                         OpenSSL:
                            aes-{128|192|256}-gcm, aes-{128|192|256}-cfb,
                            aes-{128|192|256}-ofb, aes-{128|192|256}-ctr,
                            camellia-{128|192|256}-cfb,
                            bf-cfb, cast5-cfb, des-cfb, idea-cfb,
                            rc2-cfb, seed-cfb,
                            rc4, rc4-md5, table.
                         OpenSSL 1.1:
                            aes-{128|192|256}-ocb
                         mbedTLS:
                            mbedtls:aes-{128|192|256}-cfb128,
                            mbedtls:aes-{128|192|256}-ctr,
                            mbedtls:camellia-{128|192|256}-cfb128,
                            mbedtls:aes-{128|192|256}-gcm
  -t TIMEOUT             timeout in seconds, default: 300
  -a ONE_TIME_AUTH       one time auth
  --fast-open            use TCP_FASTOPEN, requires Linux 3.7+
  --workers=WORKERS      number of workers, available on Unix/Linux
  --forbidden-ip=IPLIST  comma seperated IP list forbidden to connect
  --manager-address=ADDR optional server manager UDP address, see wiki
  --prefer-ipv6          resolve ipv6 address first
  --libopenssl=PATH      custom openssl crypto lib path
  --libmbedtls=PATH      custom mbedtls crypto lib path
  --libsodium=PATH       custom sodium crypto lib path

General options:
  -h, --help             show this help message and exit
  -d start/stop/restart  daemon mode
  --pid-file PID_FILE    pid file for daemon mode
  --log-file LOG_FILE    log file for daemon mode
  --user USER            username to run as
  -v, -vv                verbose mode
  -q, -qq                quiet mode, only show warnings/errors
  --version              show version information

Online help: <https://github.com/shadowsocks/shadowsocks>
```

### 推荐加密方式
`aes-256-gcm`

### 加密方式
1. chacha20-ietf 
安装加密库libsodium
```
yum install epel-release
yum install libsodium
```
如果上述方式失败,自己下载编译
```
yum -y groupinstall "Development Tools"
wget https://github.com/jedisct1/libsodium/releases/download/1.0.15/libsodium-1.0.15.tar.gz
tar xf libsodium-1.0.15.tar.gz && cd libsodium-1.0.15
./configure && make -j2 && make install
echo /usr/local/lib > /etc/ld.so.conf.d/usr_local_lib.conf
ldconfig
```