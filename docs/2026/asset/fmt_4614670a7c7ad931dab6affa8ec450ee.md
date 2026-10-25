
# `core.startup.py`

```python
def init_modules():
    """Initialize plugins."""
    log.debug("Imported modules...")
    categories = (
        "auxiliary", "machinery", "processing", "signatures", "reporting",
    )
    # Call the init_once() static method of each plugin/module. If an exception
    # is thrown in that initialization call, then a hard error is appropriate.
    # 下面三行代码,执行auxiliary,machinery,processing,reporting四个文件夹中的所有类的init_once函数
    # 但这些类均继承自common/abstract.py中Auxiliary,Machinery,Processing,Signature,Report,
    # Extractor. 个人觉得init_once函数是一个占坑函数, 为以后代码修改提供方便
    for category in categories:
        for module in cuckoo.plugins[category]:
            module.init_once()
    for category in categories:
        log.debug("Imported \"%s\" modules:", category)
        entries = cuckoo.plugins[category]
        for entry in entries:
            if entry == entries[-1]:
                log.debug("\t `-- %s", entry.__name__)
            else:
                log.debug("\t |-- %s", entry.__name__)
    # Initialize the RunSignatures module with all available Signatures and
    # the ExtractManager with all available Extractors.
    # 
    # RunSignatures.init_once()修改了以下两个类变量:  
    #   类变量 available_signatures: 存储所有signatures文件夹下的filename
    #   类变量 ttp_descriptions: 存储ttp_descriptions.json文件中的内容.
    RunSignatures.init_once()
    # Extractor的子类
    #   OfficeDDE1, OfficeDDE2,LnkHeader,OleStream,Powerfun,Unicorn
    # 将一些Extractor的子类添加到ExtractManager的类变量extractors中.
    # 至于为什么要添加这些子类, 先挖一个坑, 后面会填上的.
    ExtractManager.init_once()
```

## `init_tasks`

```python
# 初始化任务队列, 处理的对象：
# 1. 上次分析失败的任务
# 2. 意外退出, 未完成分析的任务

def init_tasks():
    """Check tasks and reschedule uncompleted ones."""
    db = Database()
    log.debug("Checking for locked tasks..")
    # 获取cuckoo数据库, 表tasks中, status为running的任务
    for task in db.list_tasks(status=TASK_RUNNING):
        # 检查cuckoo.conf中的reschedule选项是否修改为yes
        if config("cuckoo:cuckoo:reschedule"):
            # 如果为yes
            # 重新加入分析队列中
            task_id = db.reschedule(task.id)
            log.info(
                "Rescheduled task with ID %s and target %s: task #%s",
                task.id, task.target, task_id
            )
        # 如果为no
        else:
            # 设置为TASK_FAILED_ANALYSIS
            db.set_status(task.id, TASK_FAILED_ANALYSIS)
            log.info(
                "Updated running task ID %s status to failed_analysis",
                task.id
            )
    log.debug("Checking for pending service tasks..")
    for task in db.list_tasks(status=TASK_PENDING, category="service"):
        # 任务状态为TASK_PENDING, 说明上次分析时, 虚拟机启动失败
        # 将对应任务设置为TASK_FAILED_ANALYSIS
        db.set_status(task.id, TASK_FAILED_ANALYSIS)
```

## `init_yara`

```python
# 关键代码
# 编译yara规则为对应的yarac

def init_yara():
    """Initialize & load/compile Yara rules."""
    # yara规则所在的文件夹
    categories = (
        "binaries", "urls", "memory", "scripts", "shellcode",
        "dumpmem", "office",
    )
    for category in categories:
        dirpath = cwd("yara", category)
        if not os.path.exists(dirpath):
            log.warning("Missing Yara directory: %s?", dirpath)
        rules, indexed = {}, []
        for dirpath, dirnames, filenames in os.walk(dirpath, followlinks=True):
            for filename in filenames:
                if not filename.endswith((".yar", ".yara")):
                    continue
                filepath = os.path.join(dirpath, filename)
                # 省略一些异常处理
                rules["rule_%s_%d" % (category, len(rules))] = filepath
                indexed.append(filename)
        # Need to define each external variable that will be used in the
        # future. Otherwise Yara will complain.
        externals = {
            "filename": "",
        }
        try:
            # 编译
            File.yara_rules[category] = yara.compile(
                filepaths=rules, externals=externals
            )
        except yara.Error as e:
            raise CuckooStartupError(
                "There was a syntax error in one or more Yara rules: %s" % e
            )
        # The memory.py processing module requires a yara file with all of its
        # rules embedded in it, so create this file to remain compatible.
        if category == "memory":
            f = open(cwd("stuff", "index_memory.yar"), "wb")
            for filename in sorted(indexed):
                f.write('include "%s"\n' % cwd("yara", "memory", filename))
        indexed = sorted(indexed)
        # 省略一些log.debug
    # Store the compiled Yara rules for the "dumpmem" category in
    # $CWD/stuff/ so that we may pass it along to zer0m0n during analysis.
    # 如果分析windows程序, 则将dumpmem.yarac一同发送到windows 虚拟机中, 供zer0m0n驱动使用
    File.yara_rules["dumpmem"].save(cwd("stuff", "dumpmem.yarac"))
```

## `init_binaries`

```python
# 比较简单

def init_binaries():
    """Inform the user about the need to periodically look for new analyzer
    binaries. These include the Windows monitor etc."""
    dirpath = cwd("monitor", "latest")
    # If "latest" is a symbolic link, check that it exists.
    if os.path.islink(dirpath):
        if not os.path.exists(dirpath):
            # 抛出错误信息
            throw()
    # If "latest" is a file, check that it contains a legitimate hash.
    elif os.path.isfile(dirpath):
        monitor = os.path.basename(open(dirpath, "rb").read().strip())
        if not monitor or not os.path.isdir(cwd("monitor", monitor)):
             # 抛出错误信息
            throw()
    else:
         # 抛出错误信息
        throw()
```

### `cuckoo_main`函数

#### SIGTERM信号

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

#### 关键代码

```python
# 由于cuckoo采用的时类似cs架构, 即host通过网络传输将必要的python代码和分析工具传输给client
# client, 接受样本和python代码进行分析, 然后将结果通过网络回传给host, 完成分析.
rs = ResultServer() #运行host服务器
sched = Scheduler(max_analysis_count) # 初始化Scheduler(管理任务执行和调度的类)
sched.start() # 开始进行任务调度
```

### `ResultServer`类

> 主要用于与client主机进行数据传输.

#### 构造函数

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

#### `create_server`

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

#### `del_task`

```python
# 删除一个映射, ip和task_id
def del_task(self, task, machine):
    """Delete running task and cancel existing handlers."""
    self.instance.del_task(task.id, machine.ip)
```

#### `add_task`

```python
# 添加一个映射, ip和task_id
def add_task(self, task, machine):
    """Register a task/machine with the ResultServer."""
    self.instance.add_task(task.id, machine.ip)
```

### `Scheduler`类

> `Scheduler`类cuckoo沙箱的一个任务调度类,

#### `initialize`函数
