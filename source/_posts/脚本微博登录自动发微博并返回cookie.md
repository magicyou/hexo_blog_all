---
title: 脚本微博登录自动发微博并返回cookie
date: 2018-10-23 20:12:23
updated: 2018-10-23 20:12:23
tags: [Python]
categories: ['Python']
---
研究新浪微博前端的层层加密，脚本登录，发送微博，返回cookie

<!--more-->

```python
#!/usr/bin/python
#coding=utf-8

import rsa
import json
import requests
import re
from urllib import parse
import base64
import time
from binascii import b2a_hex
import urllib3
urllib3.disable_warnings() # 取消警告


def get_timestamp():
    '''
    获取13位时间戳
    :return:
    '''
    return int(time.time()*1000)

def get_su(username):
   '''
   加密用户名
   :param username: 用户名
   :return: 加密后的用户名
   '''
   su = base64.b64encode(parse.quote(username).encode('utf-8'))
   return su


class WeiboLogin(object):

    # 请求头
    header = {
        'Content-Type': 'application/x-www-form-urlencoded',
        'Origin': 'http://my.sina.com.cn',
        'Referer': 'http://my.sina.com.cn/profile/unlogin',
        'User-Agent ': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.102 Safari/537.36'
    }
    def __init__(self, username, password):

        self.username = username
        self.password = password

        self.session = requests.session()
        self.session.verify = False  # 取消证书验证



        # 第一次请求获取参数,注备登录
        self.get_origin_param()

    def get_origin_param(self):
        '''
        第一次请求获取参数,注备登录
        '''
        url = "https://login.sina.com.cn/sso/prelogin.php?entry=weibo&callback=sinaSSOController.preloginCallBack&su=%s&rsakt=mod&checkpin=1&client=ssologin.js(v1.4.19)&_=%s" % (
        self.get_su(), get_timestamp())
        data = None
        response = self.session.post(url, data=data, headers=self.header).text
        val_keys = re.findall(r"[(](.*?)[)]", response)
        params = json.loads(val_keys[0])

        self.pubkey = params['pubkey']
        self.retcode = params['retcode']
        self.servertime = params['servertime']
        self.pcid = params['pcid']
        self.nonce = params['nonce']
        self.rsakv = params['rsakv']
        self.is_openlock = params['is_openlock']
        self.showpin = params['showpin']
        self.exectime = params['exectime']

    def get_su(self):
        '''
        获取加密用户名
        '''
        su = base64.b64encode(parse.quote(self.username).encode('utf-8'))
        return su.decode('utf-8')

    def get_sp(self):
        '''
        加密密码
        '''
        publickey = rsa.PublicKey(int(self.pubkey, 16), int('10001', 16))
        message = str(self.servertime) + '\t' + str(self.nonce) + '\n' + str(self.password)
        sp = rsa.encrypt(message.encode(), publickey)
        return b2a_hex(sp).decode('utf-8')

    def login(self):
        '''
        微博登录
        '''
        url = "https://login.sina.com.cn/sso/login.php?client=ssologin.js(v1.4.15)&_=%s" % (get_timestamp())
        data = {
            "entry": "weibo",
            "gateway": 1,
            "from": None,
            "savestate": 30,
            "qrcode_flag": 'false',
            "useticket": 1,
            "pagerefer": "http://my.sina.com.cn/",
            "vsnf": 1,
            "su": self.get_su(),
            "service": "miniblog",
            "servertime": self.servertime,
            "nonce": self.nonce,
            "pwencode": "rsa2",
            "rsakv": self.rsakv,
            "sp": self.get_sp(),
            "sr": "1600 * 900",
            "encoding": "UTF-8",
            "cdult": 3,
            "domain": "sina.com.cn",
            "prelt": 35,
            "returntype": "META",
            "url" : "https://weibo.com/ajaxlogin.php?framelogin=1&callback=parent.sinaSSOController.feedBackUrlCallBack",

        }
        response = self.session.post(url, data=data, allow_redirects=False, headers=self.header)  # 在一些post请求中，还需要用到headers部分，此处未加，在下文中会说到
        redirect_url = re.findall(r'location.replace\("(.*?)"\);', response.text)[0]
        result = self.session.get(redirect_url, allow_redirects=False, headers=self.header).text
        ticket,ssosavestate = re.findall(r'ticket=(.*?)&ssosavestate=(.*?)"',result)[0] #获取ticket和ssosavestate参数
        uid_url = 'https://passport.weibo.com/wbsso/login?ticket={}&ssosavestate={}&callback=sinaSSOController.doCrossDomainCallBack&scriptId=ssoscript0&client=ssologin.js(v1.4.19)&_={}'.format(
            ticket, ssosavestate, get_timestamp())
        data = self.session.get(uid_url, headers=self.header).text  # 请求获取uid
        uid = re.findall(r'"uniqueid":"(.*?)"', data)[0]
        print('uid:',uid)
        self.uid = uid


    def main(self):

        cookies = []
        self.login()
        cookie = self.session.cookies.get_dict()
        print('cookie:',cookie)

        cookies.append(cookie)

        return cookies

if __name__ == "__main__":
    username = "账号XXX"
    password = '密码XXX'

    wb_hander = WeiboLogin(username, password)
    wb_hander.main()
   
```




