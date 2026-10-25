
# CUCKOO沙箱源码分析 上篇

## 前言

> 鄙人现今是一名研究生, 研究方向为样本分析自动化, `cuckoo` 沙箱项目是一个非常不错的例子, 给大家分享一下.

## 环境搭建

```
os: Ubuntu 18.04

python: python2

link: 916K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2B7K9h3q4F1M7$3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8X3q4U0x3o6l9&6k6U0k6U0x3U0M7I4x3q4)9J5z5q4!0q4z5q4!0n7c8W2)9&6z5g2!0q4y4q4!0n7z5q4!0m8b7g2!0q4z5g2)9&6x3#2!0n7c8g2!0q4y4W2)9^5c8g2!0m8y4g2!0q4y4W2)9&6z5q4!0m8c8W2!0q4y4W2)9^5z5q4)9&6x3g2!0q4z5q4!0m8c8W2!0q4y4g2!0q4z5q4!0n7c8W2)9^5y4#2!0q4y4W2)9&6b7#2)9^5x3q4!0q4z5g2)9&6c8q4!0m8x3q4!0q4z5q4!0n7x3q4!0n7x3g2!0q4y4#2)9&6b7g2)9^5y4q4)9J5z5b7`.`.
```

## `cuckoo` 总体架构描述

`cuckoo`总体架构如下图, host接收分析任务,然后开启虚拟机client, 将样本和一些必要的分析代码通过`http`协议传输给client端. 必要的分析代码通过会包括一些hook代码和内核模块,以及一些其他与这些模块交互的Python代码, 以Windows系统为例:`monitor.dll`(hook), `zer0m0n.sys`(内核模块), `inject.exe`(控制monitor),`execsc.exe`(执行shellcode)等. `cuckoo`通过这些分析代码来获取样本的行为, 然后待分析时间耗尽, 将分析结果通过`tcp`传输给host端, 将client回复到样本执行的状态.

![](https://bbs.kanxue.com/upload/attach/202006/738427_SA6TC8CE4PGKQBH.jpeg)

## `cuckoo` 常用命令行介绍

```
cuckoo community --> 获取一些恶意样本的特征库

cuckoo init --> 初始化cucko

cuckoo web --> 开启web服务, 可以使用浏览器上传样本,查看报告

cuckoo -d --> 开启分析样本服务, 

cuckoo submit --> 提交样本

cuckoo clean --> 清空分析结果
```

## `cuckoo` 命令

> 使用`pip`安装`cuckoo`之后, 可以使用`cuckoo`命令.查看一下`cuckoo`命令的位置:

```
(venv) pwnmelife@ubuntu:~/Downloads/cuckoo$ which cuckoo 

/home/pwnmelife/Downloads/cuckoo/venv/bin/cuckoo



...



(venv) pwnmelife@ubuntu:~/Downloads/cuckoo$ cat $(which cuckoo)

#!/home/pwnmelife/Downloads/cuckoo/venv/bin/python2

# -*- coding: utf-8 -*-

import re

import sys

from cuckoo.main import main

if __name__ == '__main__':

    sys.argv[0] = re.sub(r'(-script\.pyw|\.exe)?$', '', sys.argv[0])

    sys.exit(main())
```

可以看到`cuckoo`命令其实是一个`python`脚本, 最后执行代码是`cuckoo`模块中`main()`函数

## `cuckoo.main` 函数

> `main()` 函数中使用了`click`模块, 可以很方便的构建命令行程序.

> `main()` 函数中核心就是调用了下列的两个函数:

```
cuckoo_init(level, ctx) # level --> 指明输出的级别(debug or quiet), ctx是click.core.Context的对象, 用于记录一些信息.

cuckoo_main(maxcount) # maxcount --> Maxinum number of analyses to process
```

## `cuckoo.init` 函数

> 功能: 进行`cuckoo`的初始化,包括:
>
> - 创建`CWD`文件夹
> - 检查配置文件，检查版本
> - 连接数据库
> - 初始化`cuckoo`的组件

*创建`CWD`文件夹*

> `CWD`(Current Working Directory)文件夹包含了除了运行代码以外的其余文件,非常重要.

```
├── agent(agent.py agent.sh android)

├── analyzer(android, darwin, linux, windows)

├── conf  --> (cuckoo的配置文件)

├── distributed

├── elasticsearch

├── __init__.py

├── log(日志)

├── monitor(hook程序)

├── pidfiles(正在)

├── signatures(样本特征)

├── storage(存储样本程序,及分析的结果)

├── stuff

├── supervisord

├── supervisord.conf

├── web(web登录的一些设置)

├── whitelist

└── yara(样本分析的yara规则, 如:shellcode.yar, vmdetect.yar)
```

*连接数据库*

> `cuckoo`支持三种数据库: `mysql`, `sqlite3`, `postgres`, 可以在配置文件中进行配置.其中最常用的是`mysql`, 一般用作`web`模块存储数据.

```
mysql数据库中的表

+------------------+

| Tables_in_cuckoo |

+------------------+

| alembic_version  |

| errors           |

| guests           |   --> 执行分析任务虚拟机的分析流程

| machines         |   --> 虚拟机信息

| machines_tags    |

| samples          |   --> 样本信息(sha512, md5, crc32等)

| submit           |

| tags             |

| tasks            |   --> 分析任务信息

| tasks_tags       |

+------------------+





[database]

# Specify the database connection string.

# NOTE: If you are using a custom database (different from sqlite), you have to

# use utf-8 encoding when issuing the SQL database creation statement.

# Examples, see documentation for more:

# sqlite:///foo.db

# postgresql://foo:bar@localhost:5432/mydatabase

# mysql://foo:bar@localhost/mydatabase

# If empty, defaults to a SQLite3 database at $CWD/cuckoo.db.

connection = mysql://root:toor@localhost/cuckoo 



# Database connection timeout in seconds.

# If empty, default is set to 60 seconds.

timeout = 60
```

## 初始化 `cuckoo` 组件

```
init_modules()

init_tasks()

init_yara()

init_binaries()

init_rooter() # VPN的一些初始化

init_routing() # VPN的初始化

文件: core/startup.py
```

### `init_modules()`

## 关键代码

```python
def stop():
    if sched:
        sched.running = False
    if rs:
        rs.instance.stop()
    Pidfile("cuckoo").remove()
    if sched:
        sched.stop()

def handle_sigterm(sig, f):
    stop()
    # Handle a SIGTERM, to reduce the chance of Cuckoo exiting without
    # cleaning
    # 退出时, 进行一些清理工作
    signal.signal(signal.SIGTERM, handle_sigterm)
```

**关键代码**

```python
# 由于cuckoo采用的时类似cs架构, 即host通过网络传输将必要的python代码和分析工具传输给client
# client, 接受样本和python代码进行分析, 然后将结果通过网络回传给host, 完成分析.
rs = ResultServer() #运行host服务器
sched = Scheduler(max_analysis_count) # 初始化Scheduler(管理任务执行和调度的类)
sched.start() # 开始进行任务调度
```

## `ResultServer` 类

> 主要用于与client主机进行数据传输.

**构造函数**

```python
# 获取cuckoo.conf中的配置信息
ip = config("cuckoo:resultserver:ip")
port = config("cuckoo:resultserver:port")
pool_size = config('cuckoo:resultserver:pool_size')
# 绑定端口
# gevent官方介绍：
# gevent是一个基于协程的Python网络库，它使用greenlet在libev或libuv事件循环之上提供一个高级同步API
sock = gevent.socket.socket(socket.AF_INET, socket.SOCK_STREAM)
sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
sock.bind((ip, port))
# http 服务
self.thread = threading.Thread(target=self.create_server,
                               args=(sock, pool_size))
self.thread.daemon = True
self.thread.start()
```

### `create_server()`

```python
# create_server是创建一个http服务, 与client端进行交互.
# create_server中使用了GeventResultServerWorker, 这个类继承于gevent.server.StreamServer(http服务器)
# self.instance也会在add_task和del_task中使用
def create_server(self, sock, pool_size):
    if pool_size:
        pool = gevent.pool.Pool(pool_size)
    else:
        pool = 'default'
    self.instance = GeventResultServerWorker(sock, spawn=pool)
    self.instance.do_run()
```

### `del_task()`

```python
# 删除一个映射, ip和task_id
def del_task(self, task, machine):
    """Delete running task and cancel existing handlers."""
    self.instance.del_task(task.id, machine.ip)
```

### `add_task()`

```python
# 添加一个映射, ip和task_id
def add_task(self, task, machine):
    """Register a task/machine with the ResultServer."""
    self.instance.add_task(task.id, machine.ip)
```

## `Scheduler` 类

> `Scheduler`类cuckoo沙箱的一个任务调度类,

### `initialize()` 函数

```python
# 关键代码
# 以virtualbox为例
# 获取虚拟机client类型, virtualbox
machinery_name = self.cfg.cuckoo.machinery
# 获取最大并行的虚拟机数量
max_vmstartup_count = self.cfg.cuckoo.max_vmstartup_count
# 获取virtualbox的对应的管理模块, 对应的类在machinery文件夹
# cuckoo管理虚拟机通过相应的命令, 如: virtualbox
# 开启虚拟机: vboxmanage startvm <uuid|vmname>
# 恢复快照: vboxmanage snapshot <uuid|vmname> restore <snapshot-name>
machinery = cuckoo.machinery.plugins[machinery_name]()
# 读取virtualbox.conf配置信息
# 虚拟机启动的方式(headless, gui), 虚拟网卡名称(传输文件用),虚拟机的名称, 快照名称, 平台(windows/Linux)
machinery.set_options(Config(machinery_name))
# 检查虚拟机是否存在，恢复快照
machinery.initialize(machinery_name)
```

### `start()` 函数

```python
# 关键代码
# 省略的部分代码, 包括: 错误的输出, 调试信息的输出,
# 以及使用信号量机制和锁来对虚拟机(共享资源)的运行的进行同步.
# 上面一个函数, 主要初始化虚拟机软件(virtualbox ,vmware, xenserver)
self.initialize()
# 检查host是否有足够的空间
# 目前只实现了Linux
if self.cfg.cuckoo.freespace:
    dir_path = cwd("storage", "analyses")
    # Linux ,return True
    # Windows, return False
    if hasattr(os, "statvfs"):
        dir_stats = os.statvfs(dir_path.encode("utf8"))
        # Calculate the free disk space in megabytes.
        space_available = dir_stats.f_bavail * dir_stats.f_frsize
        space_available /= 1024 * 1024
        if space_available < self.cfg.cuckoo.freespace:
            # ......
            continue
    # 为任务选择合适的虚拟机, 空闲状态, web界面可以选择合适的虚拟机进行执行样本.
    for machine in self.db.get_available_machines():
        import pdb; pdb.set_trace()
        task = self.db.fetch(machine=machine.name)
        if task:
            break
    if machine.is_analysis():
        available = True
    # 接收之后, 开始进行分析, 开启虚拟机,balablala
    analysis = AnalysisManager(task.id, errors)
    analysis.daemon = True
    analysis.start()
    self.analysis_managers.add(analysis)
```

## 待续......
