
# [原创]`QueueUserAPC`执行成功条件-逆向总结

## 1. 背景：

使用 Win32 API `QueueUserAPC` 做注入时总是遇到明明插入成功了，但就是得不到执行的情况。为了得到原因，在 Win7 x86 PAE 模式下跟踪了一下 `QueueUserAPC` 的整个处理过程（初始化->插入->执行），详细分析过程我会用附件的形式给出，下面是得到的一些结论，有错误欢迎指出，一起交流。

## 2. 过程简要分析：

### 2.1 `KiServiceExit2`：

在系统调用、中断、异常等操作发生后，会从 r3 进入内核来处理，在内核处理完准备返回 r3 时，会调用这个函数，在函数内部会做出一些判断来决定是否要交付用户态 APC，判断条件就是当前线程的 `ApcState.UserApcPending`，这个域为 1 时，表示有要处理的用户态 APC 存在，才会继续调用 `KiDeliverApc`，这一点很关键。

### 2.2 `KiDeliverApc`：

这个函数进去第一件事就是判断当前线程的 `SpecialApcDisable` 域是否为 1，这个域相当于一个总开关，置 1 表示，内核和用户态 APC 都不能交付，函数直接返回。如果为 0，判断内核 APC 队列是否有内容，没有就去交付用户 APC，如果有内核 APC，一个循环全部交付完，但在交付内核 APC 过程中，一旦检查发现 `KernelApcDiable` 域为 1，`KiDeliverApc` 函数直接返回，不会向下处理用户 APC 了。这里可以得出结论，用户态 APC 必须在内核 APC 交付完才有机会交付。但通过 `QueueUserAPC` 插入的 APC 是不存在内核 APC 的，直接相当于内核 APC 交付完了。

而内核 APC 交付完，会开始交付用户 APC，这里需要注意一下，使用 `QueueUserApc` 插入的 APC，真正的回调函数是保存在 `NormalContext` 里，而 `NormalRoutine` 只是保存了一个 r3 执行时候的派发入口（`ntdll.dll` 中的 8 号导出函数），而要执行回调函数，需要从内核返回到 r3 由派发函数统一派发。Windows 这里采用的是劫持 `TrapFrame` 实现的回 r3。

### 2.3 小总结：

从上面的执行过程分析得出，要想让由 `QueueUserApc` 插入的用户态 APC 交付成功，需要满足以下条件：

- `CurrentThread.ApcState.UserApcPending` 值为 1
- `ApcList[1]` 队列不为空
- `CurrentThread.SpecialApcDisable` 值为 0

第三点暂时不用考虑，一般都是 0，但 `UserApcPending` 什么时候才能是 1 以及队列什么时候被插入 APC？这就需要去分析 `QueueUserApc` 的初始化和插入过程了。

### 2.4 `NtQueueApcThreadEx`：

这个函数是系统调用进来先执行的内核函数，函数内部首先判断目标线程是否是系统线程，即 `CrossThreadFlags.SystemThread` 位是否为 1，如果为 1 函数直接返回，为 0 才分配 `KAPC` 结构内存并初始化。

### 2.5 `KeInsertQueueApc`：

这个函数中判断插入的目标线程的 `ThreadFlags.ApcQueue` 位，如果这个位为 0，不插入队列，直接返回。不为 1，继续调用 `KiInsertQueueApc`。

### 2.6 `KiInsertQueueApc`：

将 APC 插入 `ApcList[1]` 队列。这个函数里面有个点，因为调用 `QueueUserApc` 环境固定为 0（始终插入原始环境），也就是 `KAPC.ApcStateIndex` 始终为 0，而这个函数里会把当前线程的 `ApcStateIndex` 与 `KAPC.ApcStateIndex` 做对比，因为线程在挂靠的时候 `ApcStateIndex` 为 1，这样就说明无法插入一个正在挂靠的线程。接下来会去判断目标线程是不是当前线程，如果是当前线程并且是用户态 APC，函数也返回了。前面条件都满足了，会判断目标线程是否处于可以被唤醒的等待状态，如果不是，函数在这里也返回了。如果是，将目标线程唤醒（从等待链表->延时就绪链表），唤醒成功后将 `UserApcPending` 置 1。到这里，条件全部满足。

## 3. 总结：

从检查顺序排列，要想 `QueueUserApc` 执行成功，目标线程需要满足以下条件：

- 目标线程不能为系统线程
- 目标线程 `ThreadFlags.ApcQueue` 位为 1
- 插入 APC 时目标线程不能为挂靠状态
- 目标线程不能是自身线程
- 目标线程必须为可以唤醒的等待状态(如：`SleepEx(1000, TRUE)`)

[#系统内核](https://bbs.kanxue.com/forum-41-1-131.htm)

[\[招生\]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [`QueueUserAPC完全逆向-下.pdf`](<javascript:void(0);>)（169.26kb，79次下载）
- [`QueueUserAPC完全逆向-上.pdf`](<javascript:void(0);>)（200.18kb，80次下载）
