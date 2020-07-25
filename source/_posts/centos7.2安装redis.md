---
title: centos7.2安装redis
date: 2018-01-03 18:23:00
updated: 2018-01-03 18:23:00
tags: ['Linux']
categories: ['Linux']
---
简单记录redis安装，以及相关端口的开放
<!--more-->

###  一、安装redis

```perl

1、检查是否有redis yum 源

yum install redis
2、下载fedora的epel仓库

yum install epel-release
3、安装redis数据库

yum install redis
4、安装完毕后，使用下面的命令启动redis服务

# 启动redis
service redis start
# 停止redis
service redis stop
# 查看redis运行状态
service redis status
# 查看redis进程
ps -ef | grep redis
5、设置redis为开机自动启动

chkconfig redis on
6、进入redis服务
# 进入本机redis
redis-cli

```



### 二、修改redis默认端口和密码

1、打开配置文件
```
vi /etc/redis.conf
```
2、修改默认端口，查找 port 6379 修改为相应端口即可
![img](https://images2017.cnblogs.com/blog/1267938/201801/1267938-20180111181355472-1782582188.png)


3、修改默认密码，查找 requirepass foobared 将 foobared 修改为你的密码
![img](https://images2017.cnblogs.com/blog/1267938/201801/1267938-20180111181531051-1862975192.png)


4、使用配置文件启动 redis

```
redis-server /etc/redis.conf &
```
5、使用端口登录
```
redis-cli -h 127.0.0.1 -p 6666
```
6、此时再输入命令则会报错

7、输入刚才输入的密码
```
auth 123456
```
8、停止redis

* 命令方式关闭redis
```
redis-cli -h 127.0.0.1 -p 6179
shutdown
```
* 进程号杀掉redis
```
ps -ef | grep redis
kill -9 XXX
```




### 三、开放端口

CentOS7安装iptables防火墙
CentOS7默认的防火墙不是iptables,而是firewalle.

1、安装iptable iptable-service

```
#先检查是否安装了iptables
service iptables status
#安装iptables
yum install -y iptables
#升级iptables
yum update iptables 
#安装iptables-services
yum install iptables-services

```

2、禁用/停止自带的firewalld服务

```
#停止firewalld服务
systemctl stop firewalld
#禁用firewalld服务
systemctl mask firewalld
```

3、设置现有规则
```
#查看iptables现有规则
iptables -L -n
#先允许所有,不然有可能会杯具
iptables -P INPUT ACCEPT
#清空所有默认规则
iptables -F
#清空所有自定义规则
iptables -X
#所有计数器归0
iptables -Z
#允许来自于lo接口的数据包(本地访问)
iptables -A INPUT -i lo -j ACCEPT
#开放22端口
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
#开放21端口(FTP)
iptables -A INPUT -p tcp --dport 21 -j ACCEPT
#开放80端口(HTTP)
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
#开放443端口(HTTPS)
iptables -A INPUT -p tcp --dport 443 -j ACCEPT
#开放6666端口
iptables -A INPUT -p tcp --dport 6666 -j ACCEPT
#允许ping
iptables -A INPUT -p icmp --icmp-type 8 -j ACCEPT
#允许接受本机请求之后的返回数据 RELATED,是为FTP设置的
iptables -A INPUT -m state --state  RELATED,ESTABLISHED -j ACCEPT
#其他入站一律丢弃
iptables -P INPUT DROP
#所有出站一律绿灯
iptables -P OUTPUT ACCEPT
#所有转发一律丢弃
iptables -P FORWARD DROP

```

4、保存规则设定
```
#保存上述规则
service iptables save
```

5、开启iptables服务 
```
#注册iptables服务
#相当于以前的chkconfig iptables on
systemctl enable iptables.service
#开启服务
systemctl start iptables.service
#查看状态
systemctl status iptables.service
```