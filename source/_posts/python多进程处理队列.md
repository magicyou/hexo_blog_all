---
title: python3多进程处理队列
date: 2018-09-23 20:12:45
updated: 2018-09-23 20:12:45
tags: [Python]
categories: ['Python']
---
使用过python多进程和多线程模块的都知道，Python下多线程是鸡肋，推荐使用多进程。至于原因，这里不做深究，本篇主要解决Python多进程处理队列的问题。

<!--more-->

###  简单学习一下 多进程 Multiprocessing模块

#### Process 类
Process 类用来描述一个进程对象。创建子进程的时候，只需要传入一个执行函数和函数的参数即可完成 Process 示例的创建。
* star() 方法启动进程，
* join() 方法实现进程间的同步，等待所有进程退出。
* close() 用来阻止多余的进程涌入进程池 Pool 造成进程阻塞。
#### 使用方法
multiprocessing.Process(group=None, target=None, name=None, args=(), kwargs={}, *, daemon=None)
* target 是函数名字，需要调用的函数
* args 函数需要的参数，以 tuple 的形式传入

#### 例子

```
import multiprocessing
def run(name):
    print('我是子进程%d ' % name)
if __name__ == '__main__':
    print('我是主进程')
    ser = 0
    for i in range(3):
        ser += 1
        p = multiprocessing.Process(target=run, args=(ser,))
        print('子进程%d启动' % ser)
        p.start()
    p.join()
    print('进程关闭')

```

#### Pool
Pool 可以提供指定数量的进程供用户使用，默认是 CPU 核数。当有新的请求提交到 Poll 的时候，如果池子没有满，会创建一个进程来执行，否则就会让该请求等待。 
- Pool 对象调用 join 方法会等待所有的子进程执行完毕 
* 调用 join 方法之前，必须调用 close 
* 调用 close 之后就不能继续添加新的 Process 了

#### pool.apply_async
apply_async 方法用来同步执行进程，允许多个进程同时进入池子。
#### 例子
```
from multiprocessing import Pool
def run(name):
    print('我是子进程%d ' % name)
if __name__ == '__main__':
    print('我是主进程')
    po = Pool(processes=3)
    ser = 0
    for i in range(3):
        ser += 1
        po.apply_async(run, args=(ser,))
        print('子进程%d启动' % ser)
    po.close()
    po.join()
    print('进程关闭')
```
#### multiprocessing.Manager().Queue()
多进程中的队列，可以实现多进程之间的通信，用法和Python自带queue类似

### 发现问题

* 需求：主进程有一堆数据，需要一个一个进行处理。现在要做到主进程提供数据，让多个子进程进行协同处理这些数据，并且不能重复处理。
问题代码如下：
```
#!/usr/bin/python3
# -*- coding: utf-8 -*-
from multiprocessing import Manager,Pool,freeze_support
import os
# 进程数
NUM_PROCESS = 3

def read(q):
    pid = os.getpid()
    print('子进程(%s) 启动' % pid)
    while True:
        data = q.get()
        if q.empty():
            print("列表为空！")
            break
        else:
            print('read从write中获取：', data)
    return

def main():
    print('主进程(%s) start'%os.getpid())
    queue=Manager().Queue() #Manager中的Queue才能配合Pool
    po = Pool(processes=NUM_PROCESS)
    # 模拟数据
    print('write启动')
    for i in range(100):
        queue.put(i)

    results = []
    for i in range(NUM_PROCESS):
        print("进程%d" % (i))
        result = po.apply_async(read,args=(queue,))
        results.append(result)

    po.close() #不允许进程池再加新的请求了
    po.join()

if __name__ == '__main__':
   main()

```
运行结果：
![img1](http://blog.magicyou.cn/img/20180923/python3多进程处理队列01.jpg)

处理快要结束的的时候，进程卡住了。第二次运行还是卡到，子进程不结束，主进程也无法完成。
1. 猜测是子进程的死循环无法跳出导致，修改read()方法进行测试
```
def read(q):
    pid = os.getpid()
    print('子进程(%s) 启动' % pid)
    while not q.empty():
        data = q.get()
        print('read从write中获取：', data)
    return
```
运行结果：
![img1](http://blog.magicyou.cn/img/20180923/python3多进程处理队列02.jpg)
问题依旧没有改善。
2. 四处搜索网上有没有前辈遇到这个类似的问题，有个说队列末尾加上一个标识，用这个标识判断队列中的数据是否处理完毕
修改源代码，进行测试:
```
def read(q):
    pid = os.getpid()
    print('子进程(%s) 启动' % pid)
    while True:
        data = q.get()
        if q.empty() or data == 'end':  # 使用end标识判断队列中的数据是否处理完毕
            print("列表为空！")
            break
        else:
            print('read从write中获取：', data)
    return

def main():
    print('主进程(%s) start'%os.getpid())
    queue=Manager().Queue() #Manager中的Queue才能配合Pool
    po = Pool(processes=NUM_PROCESS)
    # 模拟数据
    print('write启动')
    for i in range(100):
        queue.put(i)

    queue.put('end')  # 队列末尾添加 "end" 结束标识
    results = []
    for i in range(NUM_PROCESS):
        print("进程%d" % (i))
        result = po.apply_async(read,args=(queue,))
        results.append(result)

    po.close() #不允许进程池再加新的请求了
    po.join() #不允许进程池再加新的请求了
```
问题依旧没有改善。
多次运行脚本，发现这个问题像是具有随机性，有时候能顺利退出脚本，有时候会卡死。
不如让每个进程都多进行一次循环，判断是否有结束标志，再退出子进程呢。
```
def read(q):
    pid = os.getpid()
    print('子进程(%s) 启动' % pid)
    while True:
        data = q.get()
        if q.empty() or data == 'end':
            print("列表为空！")
            break
        else:
            print('read从write中获取：', data)
    return

def main():
    print('主进程(%s) start'%os.getpid())
    queue=Manager().Queue() #Manager中的Queue才能配合Pool
    po = Pool(processes=NUM_PROCESS)
    # 模拟数据
    print('write启动')
    for i in range(101):
        queue.put(i)

    for i in range(NUM_PROCESS):
        queue.put('end')

    results = []
    for i in range(NUM_PROCESS):
        print("进程%d" % (i))
        result = po.apply_async(read,args=(queue,))
        results.append(result)

    po.close() #不允许进程池再加新的请求了
    po.join()
```
测试运行多次，进程每次都能正常退出。算是解决了问题。

### 完整代码如下：
```
#!/usr/bin/python3
# -*- coding: utf-8 -*-
__author__ = 'magicYou'

from multiprocessing import Manager,Pool
import os

# 进程数
NUM_PROCESS = 3
def read(q):
    pid = os.getpid()
    print('子进程(%s) 启动' % pid)

    while True:
        data = q.get()
        if q.empty() or data == 'end':
            print("列表为空！")
            break
        else:
            print('read从write中获取：', data)
    return

def main():
    print('主进程(%s) start'%os.getpid())
    queue=Manager().Queue() #Manager中的Queue才能配合Pool
    po = Pool(processes=NUM_PROCESS)
    # 模拟数据
    print('write启动')
    for i in range(101):
        queue.put(i)

    for i in range(NUM_PROCESS):
        queue.put('end')

    results = []
    for i in range(NUM_PROCESS):
        print("进程%d" % (i))
        result = po.apply_async(read,args=(queue,))
        results.append(result)

    po.close() #不允许进程池再加新的请求了
    po.join()

if __name__ == '__main__':
   main()

```








