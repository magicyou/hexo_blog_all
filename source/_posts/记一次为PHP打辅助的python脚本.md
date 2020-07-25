---
title: 记一次python为PHP打辅助
date: 2018-05-05 13:12:45
updated: 2018-05-05 13:12:45
tags: [Python]
categories: ['Python']
---
* 项目结构：B/S结构，web端，自然是PHP主控；

* 问题场景：根据相应的条件，实时生成csv文件并下载。文件很大的时候，客户点击导出就一直在等，看着加载一直在打圈圈，大一点的文件三分钟也是常见，时间长了干脆请求超时，下载失败，体验极差；

* 解决办法：做成离线任务，用户点击下载之后，不再理会，在另一个面板查看文件是否已经生成，生成就可以下载。

<!--more-->

###  研究一下php怎样做离线任务

找到两个相对有代表性的博文：

1. [PHP工作笔记：离线执行](https://www.cnblogs.com/jianqingwang/p/6420602.html)

2. [离线下载文件（或者是离线执行任务）](http://www.thinkphp.cn/topic/21969.html)


简单研究了一下，感觉不是很好用，理由如下：

* 不可控，文件大了还是容易断

* 占用php资源，其他的页面请求开始慢下来

这样的话不如想想用如何把这个处理过程交给后台，让它几自个儿玩。

学Python有一段时间，刚好试试"手艺"如何。

### 规划流程

1. 用户点击导出，php拿到用户请求，整理用户要导出的相关数据（还好这些数据都是一条sql语句查出来的），存入数据库一条记录，状态status为0，表示未处理。顺便展示一下这个表结构，python和php的互帮互助全靠这张表；

|     列名     |   类型   | 长度 | 是否可为空 | 备注                           |
| :----------: | :------: | :--: | :--------: | ------------------------------ |
|      id      |  int  |  11  |     否     | 主键，非空且唯一               |
| name | varchar | 100 | 否 | 任务名称 |
| title | varchar | 255 | 否 | 导出字段标题 |
| query | varchar | 255 | 否 | 数据库查询sql |
| creater_name | varchar | 100 | 否 | 创建人 |
| status | int |  | 否 | 任务状态(0:未处理,1:处理中,2,成功,3:失败)，默认为0 |
| msg | text |  | 是 | 错误信息记录 |
| filepath | varchar | 255 | 是 | 文件地址 |

2. Python脚本一直在服务器运行，实时检测这张表是否有需要处理的任务（就是检测这张表是否有状态值status==1的记录）；

* 2.1 有，就将这条记录的状态status改为1（这时前端页面看到的任务状态是"处理中"）

* 2.2 查询数据库生成一个csv文件，成功后将文件路径放入条记录的 字段"filepath" 

* 2.3 没有报错，就将status改为2； 有报错，就将status改为3，并将失败原因记录在msg

3. 用户看到最新状态是"成功"，就可以点击文件下载了；状态是 "失败"，则需要进一步查看失败原因进行重新生成相关csv文件。

### 上python脚本

```python
#!/usr/bin/python3
# -*- coding: utf-8 -*-
import pymysql
import psycopg2
import psycopg2.extras
import time
import csv
import random
import sys
from Crypto import Random
import binascii
from Crypto.PublicKey import RSA
from Crypto.Cipher import PKCS1_v1_5
import threading

# 线程数，限定数
thread = 6

# 本地测试环境
hostName = 'http://test-web'
mysql = {'host':'localhost','user':'root','pwd':'','db':'test'}
pgsql = {'host':'127.0.0.1','database':'sys','user':'root','password':'root','port':'5432'}

def conMysql(type='dict'):
    '''
    mysql连接
    :param type: 获取数据类型 默认为字典类型
    :return:游标
    '''
    if type=='dict':
        conn = pymysql.connect(mysql['host'],mysql['user'],mysql['pwd'],mysql['db'],charset='utf8',cursorclass=pymysql.cursors.DictCursor)
    else:
        conn = pymysql.connect(mysql['host'],mysql['user'],mysql['pwd'],mysql['db'],charset='utf8')
    return conn

def conPgsql():
    '''
    pgsql数据库连接
    :return: 游标
    '''
    conn = psycopg2.connect(database=pgsql['database'],user=pgsql['user'],password=pgsql['password'],host=pgsql['host'],port=pgsql['port'])
    return conn

def exportCsv(title,data):
    '''
    生成csv文件
    :param title: 表头
    :param data: 数据
    :return: {'status':任务执行状况, 'fileName':文件名}
    '''
    # 数据处理
    for i in range(len(data)):
        dataList = list(data[i].values())
        dataList.insert(0, i+1)
        data[i] = dataList

    fileName = '/Public/Temp/' + time.strftime('%Y%m%d%H%M',time.localtime(time.time())) + '_'+ str(int(round((time.time()) * 1000))) + ".csv"
    title = title.split(',')
    with open(fileName,"w",newline='',encoding='gb18030') as csvfile:
        writer = csv.writer(csvfile)
        # 先写入columns_name
        try:
            writer.writerow(title)
            writer.writerows(data)
        except:
            return {'status':False, 'msg':'文件写入失败！'}
    return {'status':True, 'fileName':hostName + fileName}

def getCsvDate(sql):
    '''
    获取csv数据
    :param sql: 获取csv数据的sql语句
    :return:{'status':sql执行状况, 'data':数据}
    '''
    conn = conPgsql()
    cur = conn.cursor(cursor_factory=psycopg2.extras.RealDictCursor)

    try:
        cur.execute(sql)
        rows = cur.fetchall()
    except:
        cur.close()
        conn.close()
        return {'status':False, 'msg':'sql执行失败！'}

    conn.commit()
    cur.close()
    conn.close()
    return {'status':True, 'data':rows}

def dealWork():
    '''
    任务处理
    :return: None
    '''
    conn = conMysql()
    # 看有没有正在进行任务是否超过限定数,超过限定数就跳跳出
    cursor = conn.cursor()
    sql = "select count(*) count from box_download_task where status='1';"
    cursor.execute(sql)
    re = cursor.fetchone()
    rowsNum = re['count']
    if rowsNum > thread:
        return

    # 看有没有未开始的任务，有的话继续进行，没有则跳出
    sql = "select count(*) count from box_download_task where status='0' and type<>'4' order by id asc;"
    cursor.execute(sql)
    re = cursor.fetchone()
    if re['count']==0:
        return
    else:
        # 根据降序选择最早加入的并且未处理的任务进行处理
        sql = "select * from box_download_task where status='0' order by id asc;"
        cursor.execute(sql)
        re = cursor.fetchone()
        workId = re['id']
        # 修改记录status为1，处理中，防止其他线程重复处理
        sql = 'update box_download_task set status="%s" where id=%d;' % ('1',workId)
        cursor.execute(sql)
        conn.commit()

        sql = re['query']
        title = re['title']
        # 获取csv的数据
        res = getCsvDate(sql)
        if res['status']:
            # 导出csv
            file = exportCsv(title, res['data'])
            # 写入文件是否成功，成功则修改status为2，失败则status为3
            if file['status']:
                sql = 'update box_download_task set status="%s", filepath="%s" where id=%d;' % (
                '2', file['fileName'], workId)
                cursor.execute(sql)
                conn.commit()
            else:
                sql = 'update box_download_task set status="%s", msg="%s" where id=%d;' % ('3', file['msg'], workId)
                cursor.execute(sql)
                conn.commit()
        else:
            sql = 'update box_download_task set status="%s", msg="%s" where id=%d;' % ('3', res['msg'], workId)
            cursor.execute(sql)
            conn.commit()
            return False

    conn.close()
    return False

if __name__ == "__main__":

    # 引入多线程并行快速处理
    while True:
        for i in range(thread):
            t = threading.Thread(target=dealWork, args=())
            t.start()
            time.sleep(1)

        t.join()
        print('执行了')
        time.sleep(5)
```

小结：

* python引入多线程，并行处理多个任务，效率不言而喻；

* 引用队列思想，先进先出，一个一个处理，一个不落；

* python独立于php进程，不占用php运行资源；

* 比起以往点击下载后进行查询生成文件再下载，虽然步骤多了，但是用户体验提高了不止一点，属于优化（想想不该掉原先php的导出文件，数据大迟早要出问题）

完结！