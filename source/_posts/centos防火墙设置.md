---
title: centos防火墙设置
date: 2018-07-06 15:23:00
updated: 2018-07-06 15:23:00
tags: ['Centos']
categories: ['Linux']
---
centos防火墙设置
<!--more-->

环境概况
```
系统：CentOS Linux release 6.8

```
安装pyinstaller
​```
# 查看防火墙状态
service iptables status
 
# 停止防火墙
service iptables stop
 
# 启动防火墙
service iptables start
 
# 重启防火墙
service iptables restart
 
# 永久关闭防火墙
chkconfig iptables off
 
# 永久关闭后重启
chkconfig iptables on

​```

3、开启80端口

 vim /etc/sysconfig/iptables
 ```
# 加入如下代码，比着两葫芦画瓢 :)
-A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT
```


保存退出后重启防火墙
```
service iptables restart
```

