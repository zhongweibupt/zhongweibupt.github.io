---
layout:     post
title:      �ֲ�ʽ����ʼ�-1. ��������
category: blog
description: Scrapy-Redis-MongoDB�ܹ�֮1.��������
---

#�ֲ�ʽ����ʼ�-1. ��������

- Author:  Zhao
- Date:     2016-4-13
- E-mail:  zhongwei_bupt@163.com

----------------------

>д��ǰ�棺��ƪϵͳ����ΪCentOS 7.0 -������������CentOS 6.5 -��Ѷ��������

[TOC]


##�汾˵��

###Master����

* CentOS 7.0 -����������
* Python 2.7.5
* Scrapy 1.0.5
* pip 8.1.1
* redis-py 2.10.5
* Redis-3.0.7
* pymongo-3.2.2
* MongoDB-3.2.2
* scrapy-redis-0.6.0
* requests-2.10.0

###Slaver ����
* CentOS 6.5 -��Ѷ������
* Python 2.7.11
* Scrapy 1.0.5
* redis-py 2.10.5
* pymongo-3.2.2
* redis-2.10.5-py2.py3-none-any.whl
* pip 8.1.1
* requests-2.10.0

##Master������������

###Requests��װ

``` bash
pip install requests
```

###scrapy-redis��װ

``` bash
pip install scrapy-redis
```

###redis-py��װ

``` bash
pip install redis
pip install hiredis
```

###Redis��װ

``` bash
wget http://download.redis.io/releases/redis-3.0.7.tar.gz
tar xzf redis-3.0.7.tar.gz
cd redis-3.0.7
make
```

``` bash
$ cd ./src
$ ./redis-cli
127.0.0.1:6379> PING
PONG
127.0.0.1:6379> INFO

# Server
redis_version:3.0.7
redis_git_sha1:00000000
redis_git_dirty:0
redis_build_id:22c4bf2abbb59838
redis_mode:standalone
os:Linux 3.10.0-123.9.3.el7.x86_64 x86_64
```

��̨���У�
``` bash
./redis-server &  
```

`vi /etc/sysconfig/iptable`����`6379`�˿ڣ�

``` bash
# redis
-A INPUT -p tcp -m state --state NEW -m tcp --dport 6379 -j ACCEPT
```

��������iptables��

``` bash
/etc/init.d/iptables restart
#or 
service iptables restart
```

###pymongo��װ

PyMongo is a Python distribution containing tools for working with MongoDB, and is the recommended way to work with MongoDB from Python.

``` bash
pip install pymongo
```

###MongoDB��װ

Create a `/etc/yum.repos.d/mongodb-org-3.2.repo` file :

``` bash
[mongodb-org-3.2]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.2/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-3.2.asc
```

``` bash
yum install -y mongodb-org
yum install -y mongodb-org-3.2.4 mongodb-org-server-3.2.4 mongodb-org-shell-3.2.4 mongodb-org-mongos-3.2.4 mongodb-org-tools-3.2.4
```

``` bash
service mongod start
chkconfig mongod on
```

`vi /etc/sysconfig/iptable`����`27017`�˿ڣ�

``` bash
# mongodb
-A INPUT -p tcp -m state --state NEW -m tcp --dport 27017 -j ACCEPT
```

��������iptables��

``` bash
/etc/init.d/iptables restart
#or 
service iptables restart
```

##Slaver������������

ֻ�谲װ`Python 2.7`��`requests`��`redis-py`��`pymongo`��