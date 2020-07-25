---
title: 安装python的crypto模块
date: 2018-09-26 23:12:45
updated: 2018-09-26 23:12:45
tags: [Python]
categories: ['Python']
---
crypto模块安装老是出问题，这次咋就这么顺利，先记一下

<!--more-->



* 测试环境1：

  系统：centos6.8

  python版本：Python 3.6.1

* 测试环境2：

  系统：macOs

  python版本：Python 3.7.2



安装办法

1. 点击下载 [下载](https://www.dlitz.net/software/pycrypto/)

2. 手动安装

   ```
   >>>   tar zxvf pycrypto-2.6.1.tar.gz
   >>>   cd pycrypto-2.6.1
   >>>   python3 setup.py install
   ```

   测试一下

   ```
   [root@localhost testPython]# python3
   Python 3.6.1 (default, Dec 29 2018, 16:52:58)
   [GCC 4.4.7 20120313 (Red Hat 4.4.7-23)] on linux
   Type "help", "copyright", "credits" or "license" for more information.
   >>> from Crypto import Random
   >>> import binascii
   >>> from Crypto.PublicKey import RSA
   >>> from Crypto.Cipher import PKCS1_v1_5
   >>>
   ```


SUCCESS !

* 