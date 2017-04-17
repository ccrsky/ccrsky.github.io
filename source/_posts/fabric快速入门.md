---
title: fabric快速入门
date: 2016-02-14 13:07:01
tags:
    - python
    - 运维

category: 
    - python
    
---

Fabric是一个Python库，用于简化使用SSH的应用程序部署或系统管理任务。

它提供的操作包括：执行本地或远程shell命令，上传/下载文件，以及其他辅助功能等。

### 安装

[☞点我传送到fabric官网](http://www.fabfile.org/installing.html)

安装fabric库

``` shell
pip install fabric
```
>依赖 Paramiko 、PyCrypto库

安装依赖

``` shell
# 安装 pycrypto 密码库
pip install pycrypto

# 安装 pycrypto 依赖
pip install ecdsa
```

ps:【windows7 x64 ，python2.7 】Windows下pip安装包报错：Microsoft Visual C++ 9.0 is required Unable to find vcvarsall.bat ，可以安装[Micorsoft Visual C++ Compiler for Python 2.7](https://www.microsoft.com/en-us/download/details.aspx?id=44266) 。


### fabric使用

新建py脚本：fabfile.py

``` python
def hello():
    print("Hello world!")
    
def hi():
    print("Hello world!")
```

执行fab命令:执行hello函数

``` shell
fab hello
```

 如果当前模块下没有fabfile.py文件，那么需要指定-f参数.
 
 ``` shell
 fab -f test.py hello
 ```

使用参数

``` python
def hello(name, value):
    print("%s = %s!" % (name, value))
```
 
带参执行命令
 
```shell
fab hello:name=age,value=20  
```

执行本地操作

``` python
from fabric.api import local, lcd

def lsfab():
　　with lcd('~/tmp/fab'):
   　　local('ls')
```

执行远程操作

``` python
def setting_ci():
    local('echo "add and commit settings in local"')

def update_setting_remote():
     print "remote update"
     with cd('~/temp'):   #cd用于进入某个目录
         run('ls -l | wc -l')  #远程操作用run 
```

多服务器操作

``` python
#!/usr/bin/env python
# encoding: utf-8

from fabric.api import *
env.roledefs = {
            'testserver': ['user1@host1:port1',],
            'productserver': ['user2@host2:port2', ]
            }            

@roles('testserver')
def task1():
    run('ls -l | wc -l')

@roles('productserver')
def task2():
    run('ls ~/temp/ | wc -l')

def do():
    execute(task1)
    execute(task2) 
```


用不同颜色打印

```python
from fabric.colors import *

def show():
    print green('success')
    print red('fail')
    print yellow('yellow') 
```

fabric 命令行工具

```shell
装饰器作用？
    @task   # 定义任务 @task(alias='dwm') fab --list
    @parallel # 并行处理

    命令行常用： fab --help
    fab -l             -- 显示可用的task（命令）
    fab -H             -- 指定host，支持多host逗号分开
    fab -R             -- 指定role，支持多个
    fab -P             -- 并发数，默认是串行
    fab -w             -- warn_only，默认是碰到异常直接abort退出
    fab -f             -- 指定入口文件，fab默认入口文件是：fabfile/fabfile.py
```

[☞更多参考官网教程](http://docs.fabfile.org/en/1.13/)
