---
title: mac版本Navicat连接远程mysql
date: 2018-05-02 13:12:45
updated: 2018-05-02 14:34:12
tags: ['other']
categories: ['MacOS']
---
一步一步解决mac版本的Navicat连接远程mysql
<!--more-->

云服务器：腾讯云1核1GB1Mbps

系统：centos7.2

已安装lnmp 1.5


这时mac上使用Navicat连接远程mysql是拒绝访问，没关系，来一步步搞一下

1.设置mysql允许远程连接数据库

使用“use mysql”命令，选择要使用的数据库，修改远程连接的基本信息，保存在mysql数据库中，因此使用mysql数据库。

使用“GRANT ALL PRIVILEGES ON *.* TO ‘root’@’%’ IDENTIFIED BY ‘root’ WITH GRANT OPTION;”命令可以更改远程连接的设置。

```
mysql> use mysql;mysql> GRANT ALL PRIVILEGES ON . TO '数据库用户名（一般是root）'@'%' IDENTIFIED BY '数据库密码' WITH GRANT OPTION;
```

使用“flush privileges;”命令刷新刚才修改的权限，使其生效

```
mysql> flush privileges;
```

```
mysql> use mysql
Database changed
mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '*********' WITH GRANT OPTION;
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql> exit
Bye
```



2.开放3306端口

Centos 7使用firewalld代替了原来的iptables。下面记录如何使用firewalld开放Linux端口

```
1.systemctl start firewalld 开启防火墙

2.firewall-cmd --zone=public --add-port=3306/tcp --permanent

命令含义：

--zone #作用域

--add-port=80/tcp #添加端口，格式为：端口/通讯协议

--permanent #永久生效，没有此参数重启后失效

3.firewall-cmd --reload

```
![img1](http://blog.magicyou.cn/img/20180502/image-20180818144247986.png)
3. 测试一下，还不行的，添加出入站规则
![img2](http://blog.magicyou.cn/img/20180502/image-20180818144502300.png)
 4.可能会出现下图状况
 ![img1](http://blog.magicyou.cn/img/20180502/image-20180818144600996.png)
 解决方案：

进入mysql

更改max_connection_errors的值，即提高允许的max_connection_errors的数量。

查看该属性设置为多大：命令：show global variables like '%max_connect_errors%';

```
mysql> show global variables like '%max_connect_errors%';
+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| max_connect_errors | 100   |
+--------------------+-------+
1 row in set (0.00 sec)
```


然后修改该属性

```
mysql> set global max_connect_errors=1000;
Query OK, 0 rows affected (0.00 sec)

mysql> show global variables like '%max_connect_errors%';
+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| max_connect_errors | 1000  |
+--------------------+-------+
1 row in set (0.00 sec)

mysql> exit
```

这样子设置后会发现还会出现一样的问题，永久解决问题的话就接着操作

进入mysql配置文件，设置max_connect_errors = 1000，重启mysql，完毕

```
max_connect_errors = 1000
```
