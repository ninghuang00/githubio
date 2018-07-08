---
title: db2部署教程
categories:
  - 笔记
tags:
  - 教程
  - db2
  - 数据库
date: 2018-07-08 13:23:24
---
 这是摘要
 <!-- more -->

# Install DB2
DB2 v9.7 fp5 is recommended. Follow the command steps of installing DB2 is ok. Here is a sample script which can directly install DB2 successfully. 
One thing to make sure is set DB2 type to be Oracle compatible. To support sql like “select * from * between 1 and 100”, Oracle compatible mode is required.
During DB2 installation, one user account “db2inst1” is setup and we need set its password, for example, set it as “passw0rd”.
# login as root
```sh 
mkdir /root/db2setup
cd /root/db2setup
```
# download v9.7fp5_linuxx64_ese.tar.gz from ftp 
```sh 
wget ftp://ningbo:sterling11@9.115.47.13/DB2/licenses/v9.7fp5_linuxx64_ese.tar.gz
tar -zxf v9.7fp5_linuxx64_ese.tar.gz
cd ese
./db2_install -f NOTSAMP   # choose No when whether use different directory       choose ESE as the version of DB2
```

# Blow steps is to create instance
# add group role
```sh 
groupadd dasadm1
groupadd db2fadm1
groupadd db2iadm1
```

# add user
( the passw0rd may not correct setted by this cmd, if so, just use passwd $username to change password )
```sh 
useradd -m -d /home/db2inst1 -g db2iadm1 db2inst1 -p passw0rd
useradd -m -d /home/db2fenc1 -g db2fadm1 db2fenc1 -p passw0rd
useradd -m -d /home/dasusr1 -g dasadm1 dasusr1 -p passw0rd
passwd db2inst1
passwd db2fenc1
passwd dasusr1
```

# create db2 user & instance
```sh 
cd /opt/ibm/db2/V9.7/instance
./db2icrt -u db2fenc1 db2inst1
su - db2inst1
```

# set database connection type and port
```sh 
db2set DB2COMM=tcpip
db2set DB2CODEPAGE=1208
db2 update database manager configuration using svcename 50000

db2start
db2stop
```

# activate the license
```sh 
wget ftp://ningbo:sterling11@9.115.47.13/DB2/licenses/v9.7fp5_linuxx64_ese_c.tar.gz
tar -zxf v9.7fp5_linuxx64_ese_c.tar.gz
cd ese_c/db2/license/
db2licm -l
db2licm -a db2ese_c.lic

db2start
db2stop
```

# login to root user
# set DB2 management server
```sh 
cd /opt/ibm/db2/V9.7/instance
./dascrt -u dasusr1 
su - dasusr1
db2admin start
```
