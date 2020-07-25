---
title: yum源的代理设置及修改
date: 2019-02-03 18:23:00
updated: 2019-02-03 18:23:00
tags: ['Linux','Centos']
categories: ['Linux']
---

最近公司服务器要进行升级，所以要在升级之前系统整体迁移到新的服务器。

<!--more-->

##### 前言

系统都是内网使用，所以服务器也是内网，所幸是运维给了一个代理ip，能访问几个特定的网站，比如qq.com(本来开的腾讯云短信，后来发现qq.com域名下二级域名都能访问)。

升级的道路很坎坷，各种包要重装，依赖很多，但是没有外网，虽然原本带的yum包足够多，但还是不够。

后来后来，作死的换了yum源，阿里的，无奈不能使用。我找到所有的yum源地址，一个一个试，惊奇的发现中科大yum源居然能用代理访问。

```
curl -m 30 --retry 3 -x 10.0.248.64:3128 http://centos.ustc.edu.cn
```



#### 1. Centos系统使用代理上网时 yum的代理设置

##### 1.1 编辑yum配置文件：

```
vim /etc/yum.conf
```

##### 1.2 打开yum的配置文件之后，在文件最后加上代理服务器的协议、地址、端口，如果代理服务器需要用户认证话，同时加上认证用户的用户名和密码。

代理服务器不需要认证：加上 proxy=协议://代理服务器地址:端口 ，例：

```
proxy=http://192.168.1.1:80
```

代理服务器需要认证用户：加上 proxy=协议://代理服务器地址:端口 ，例：

```
proxy=http://192.168.1.1:80
proxy_username=代理服务器用户名
proxy_password=代理服务器密码
```



#### 2. yum源的更换

##### 2.1 首先备份CentOS-Base.repo

```
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
```

##### 2.2 下载对应版本的CentOS-Base.repo, 放入/etc/yum.repos.d/

```
Centos5
https://lug.ustc.edu.cn/wiki/_export/code/mirrors/help/centos?codeblock=1

Centos6
https://lug.ustc.edu.cn/wiki/_export/code/mirrors/help/centos?codeblock=2

Centos7
https://lug.ustc.edu.cn/wiki/_export/code/mirrors/help/centos?codeblock=3
```

放进 /etc/yum.repos.d/


##### 2.3 运行以下命令生成缓存

```
yum clean all
yum makecache
```



#### 3. yum简单使用

```
   # 安装rpm包：
   yum install RPM包

   # 删除rpm包，包括与该包有依赖性的包：
   yum remove 包名

   # 检查可更新的rpm包：
   yum check-update

   # 更新所有的rpm包：
   yum update

   # 更新指定的rpm包：
   yum update 包名

   # 大规模的升级版本：
   yum upgrade

   # 列出资源库中所有可以安装或更新的rpm包的信息：
   yum info

   # 列出资源库中特定的可以安装或更新以及已经安装的rpm包的信息：
   yum info 包名

   # 列出资源库中所有可以更新的rpm包的信息：
   yum info updates

   # 列出已经安装的所有的rpm包的信息：
   yum info installed

   # 列出已经安装的但是不包含在资源库中的rpm包信息：
   yum info 包名

   # 列出资源库中所有可以更新的rpm包：
   yum list updates

   # 列出已经安装的所有rpm包：
   yum list installed

   # 列出已经安装的但不包含在资源库中的rpm包：
   yum list extras

   # 列出资源库中所有可以安装或更新的rpm包：
   yum list

   # 列出资源库中特定的可以安装或更新以及已经安装的rpm包：
   yum list 包名

   # 搜索匹配特定字符的rpm包的详细信息：
   yum search 包名

   # 搜索包含特定文件名的rpm包：
   yum provides 包名

   # 清除暂存的rpm包文件：
   yum clean packages

   # 清除暂存的rpm头文件：
   yum clean headers

   # 清除暂存中旧的rpm旧文件：
   yum clean oldheaders

   # 清除暂存中旧的rpm头文件和包文件：
   yum clean
   或
   yum clean all
```

   