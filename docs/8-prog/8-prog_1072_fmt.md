
# 今天看 NT 内核源码，有点小小心得，分享一下

> 原文：[`www.pediy.com/kssd/pediy11/114530.html`](https://www.pediy.com/kssd/pediy11/114530.html)

今天看 NT 内核源码，有点小小心得，分享一下

===========================

我们都知道，驱动程序的入口 `DriverEntry` 是在 `SYSTEM` 进程环境中执行的，不知大家注意过没有：

1.  `DriverEntry` 为什么要在 `SYSTEM` 中运行
2.  `DriverEntry` 是如何实现在 `SYSTEM` 中运行的呢？

要回答这两个问题，先请大家看 `NtLoadDriver` 中的一段代码：

```c
    //
    // Because drivers may wish to create a system thread and execute in
    // its context, the remainder of this service must be executed in the
    // context of the primary system process.  This is accomplished by
    // queueing a request to one of the EX worker threads and having it
    // invoke the I/O system routine to complete this work.
    //
    // Fill in a request packet and queue it to the worker thread then, so
    // that it can actually do the load.
    //

    KeInitializeEvent( &loadPacket.Event, NotificationEvent, FALSE );
    loadPacket.DriverObject = (PDRIVER_OBJECT) NULL;
    loadPacket.DriverServiceName = &driverServiceName;

    if (PsGetCurrentProcessByThread(CurrentThread) == PsInitialSystemProcess) {

        //
        // If we are already in the system process, just use this thread.
        //

        IopLoadUnloadDriver(&loadPacket);

    } else {

        ExInitializeWorkItem( &loadPacket.WorkQueueItem,
                              IopLoadUnloadDriver,
                              &loadPacket );

        ExQueueWorkItem( &loadPacket.WorkQueueItem, DelayedWorkQueue );

        KeWaitForSingleObject( &loadPacket.Event,
                               UserRequest,
                               KernelMode,
                               FALSE,
                               (PLARGE_INTEGER) NULL );

    }
```

对这两个问题，程序解释得很清楚。由于驱动程序在初始化时有可能要创建在 `SYSTEM` 进程环境中运行的系统线程，所以 `DriverEntry` 必须运行在 `SYSTEM` 进程环境中。

由于 `NtLoadDriver` 既可能在 `SYSTEM` 进程中调用，也可能在其他进程中调用，所以 `NtLoadDriver` 会判断当前进程是否是 `SYSTEM` 进程。如果不是，就将 `IopLoadUnloadDriver` 例程的调用任务封装成一个任务包，通过 `ExQueueWorkItem` 加入了一个队列 `ExWorkerQueue[DelayedWorkQueue]` 中。`SYSTEM` 进程中有一个工作线程，会周期查看该队列。如果不空，就一个一个任务包取出来处理。每处理完一个任务包会将其中的 `event` 对象设置为有效，`NtLoadDriver` 就通过任务包中的 `event` 得到 `IopLoadUnloadDriver` 处理完的通知。

内核进程通信也太容易了哈，反正大家都是相同地址空间，一个全局变量，一个同步变量搞定哈。
