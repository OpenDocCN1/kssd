
观察20~24的调用，这是一个线程池里的线程，然后周期性地调用着一个回调。

观察17~19的调用，发现是`nlohmann`的`json`类在调用构造函数。

观察05~11的调用，发现也调用了`AcLayers`垫片的`FTH`。

观察04的调用，可知7号线程也在等待一个`CS`锁，观察这个锁如下：

```
0:007> !cs 00d30258
---------------------------------------------
Critical section = 0x00d30258 (+0xD30258)
DebugInfo = 0x776dc920
LOCKED
LockCount = 0x3
WaiterWoken = No
OwningThread = 0x00004b38
RecursionCount = 0x1
LockSemaphore = 0xFFFFFFFF
SpinCount = 0x020007cf
```

这个`cs`锁的持有者是线程`0x00004b38`，这个线程就是主线程。

通过以上的分析，可以猜想是主线程和7号线程发生了死锁，然后其他线程也因执行堆的相关操作而全都卡住。

## 定位死锁的原因

回想主线程等待的`SRWLock`，通过它的地址，我们可以知道这个锁的具体名字：

```
ln 6be0c9ac
Browse module
Set bu breakpoint
(6be0c9ac)   AcLayers!NS_FaultTolerantHeap::FthDelayFreeQueueFlushLock
```

这个`SRWLock`的名字叫`FthDelayFreeQueueFlushLock`。在`IDA`中，我们可以看到`AcLayers`垫片的`FthDelayFreeQueueInsert`函数是如何尝试获取`SRWLock`的：

![image-20211025214331561](https://bbs.kanxue.com/upload/attach/202110/744796_Y6ZQ4CNADRZVGYM.jpg)

交叉引用`FthDelayFreeQueueFlushLock`，发现有多个函数会获取这个锁：

![image-20211025214725855](https://bbs.kanxue.com/upload/attach/202110/744796_PUK378Y4RWVCGC4.jpg)

再观察7号线程，发现7号线程在获取`CS`锁之前，调用了`FthDelayFreeQueueFlush`函数，通过`IDA`观察如下：

![image-20211025220130896](https://bbs.kanxue.com/upload/attach/202110/744796_3HEJTUD67HN69FV.jpg)

`FthDelayFreeQueueFlush`函数做的第一件事就是以独占的方式获取`FthDelayFreeQueueFlushLock`。

## 结论

主线程获取了`CS`锁后，因为进入了`AcLayers`垫片的`FTH`，所以尝试获取共享的`FthDelayFreeQueueFlushLock`；

而7号线程在获取了独占的`FthDelayFreeQueueFlushLock`后，又希望获取主线程持有的`CS`锁；

其他线程也因为处理堆时，进入了`AcLayers`垫片，进而尝试获取堆相关的`CS`锁，导致暂停，最后整个进程都死锁了。

这个问题是`win10`下，在调试器中启动程序的调试版本+兼容模式出现的，程序正常启动是不会造成此问题的（因为不会启用调试堆）。如果不以兼容模式运行程序，那也不会造成此问题（因为不会启用垫片引擎）。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于  2021-10-25 23:18

被`coneco`编辑

，原因：

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
