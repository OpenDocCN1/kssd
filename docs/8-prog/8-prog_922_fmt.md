
# APC 浅浅的说

> 原文：[`www.pediy.com/kssd/pediy12/134203.html`](https://www.pediy.com/kssd/pediy12/134203.html)

以下是我对 APC 的一些理解，肯定不全面也不深入，真的是浅浅的，但本着共同学习进步的想法发出来，欢迎各位发表意见多多指导多多拍砖。

APC，异步过程调用。

APC 是线程相关的，每个线程都有自己独自的 APC 链表，因此可以让一段代码在指定的线程上下文中运行。系统中有两种 APC，用户层和内核层 APC（忽略用户层）。

初始化 APC 的函数定义如下：

```c
VOID KeInitializeApc (
    __out PRKAPC Apc,
    __in PRKTHREAD Thread,
    __in KAPC_ENVIRONMENT Environment,        //APC 环境
    __in PKKERNEL_ROUTINE KernelRoutine,      //内核模式 APC 函数
    __in_opt PKRUNDOWN_ROUTINE RundownRoutine, //线程退出时且 APC 链表中非空，若此例程也非空则被调用
    __in_opt PKNORMAL_ROUTINE NormalRoutine,
    __in_opt KPROCESSOR_MODE ApcMode,
    __in_opt PVOID NormalContext
)
```

参数详细说明下面介绍。

内核 APC 又分为两种：普通 APC 和特殊 APC。这两种 APC 在线程中共用同一个 APC 链表，但特殊 APC 总是排在普通 APC 的前面，即在交付时优先交付特殊 APC。APC 对象结构中并没有标记自己是一个普通 APC 还是特殊 APC 的字段，这种区分是通过参数 `NormalRoutine` 来区分的。初始化的时候如果参数 `NormalRoutine` 为 `NULL`，则说明这是一个特殊 APC，此时忽略参数 `NormalContext` 且参数 `ApcMode` 默认为 `KernelMode`；如果参数 `NormalRoutine` 不为 `NULL`，则这是一个普通 APC，参数 `NormalContext` 作为参数 `NormalRoutine` 的参数。在交付普通 APC 时，即执行 `NormalRoutine`，也执行 `KernelRoutine`。

下面说一下参数 `Environment`，这个参数的设定，是为了解决线程 `Attach` 到其它进程以及 `Dettach` 回来的情况。

在线程的内核对象中，有两个域 `ApcState` 和 `SavedApcState`，以及一个包含两个元素的数组 `ApcStatePointer` 和保存 `ApcStatePointer` 下标的整数索引值 `ApcStateIndex`。APC 插入时，总是插入到 `ApcStatePointer[ApcStateIndex]` 代表的链表中(也即总是插入到 `ApcState` 代表的链表中，因为 `ApcStatePointer[ApcStateIndex]` 和 `ApcState` 总是指向同一块地址)。例如，当线程在自己的进程中运行时，新的 APC 被插入到 `ApcState` 所代表的链表中，`ApcStateIndex` 为 0，`ApcStatePointer[0]` 指向 `ApcState` 指向的地址。当线程附加到其它进程中时，系统先将当前 `ApcStatePointer[ApcStateIndex]` 指向的内存中的内容（即 `ApcState` 指向的内容）保存到 `SavedApcState` 中，然后，将 `ApcStateIndex` 改为 1，令 `ApcStatePointer[1]` 指向 `ApcState` 指向的内存，`ApcStatePointer[0]` 指向 `ApcState` 指向的内存。然后重新初始化 `ApcStatePointer[ApcStateIndex]` 指向的内存。为什么这么麻烦呢？直接来两个字段，一个 `ApcState`，一个 `SavedApcState` 不就完了吗？当然不行，因为我们可以指定 APC 被插入到哪个链表中(`ApcState` 还是 `SavedApcState`)，而我们正是通过 `Environment` 指定的。

先看一下 `Environment` 的结构：

```c
typedef enum _KAPC_ENVIRONMENT {
    OriginalApcEnvironment,
    AttachedApcEnvironment,
    CurrentApcEnvironment,
    InsertApcEnvironment
} KAPC_ENVIRONMENT;
```

下面是对这四个域的解释：

`OriginalApcEnvironment`：插入到目标线程 `ApcStatePointer[0]` 代表的链表中。这样可以保证这个 APC 只在目标线程在自己的进程中时运行，为什么呢？因为从上面嗦的分析中可以知道，当目标线程没有 `Attach` 时，`ApcStatePointer[0]` 指向 `ApcState`，即线程正在使用的 APC 链表；当目标线程 `Attach` 了，则 `ApcStatePointer[0]` 指向 `SavedApcState`，它所代表的链表此时是作为备份的，其中的 APC 不会被交付，只有在 `Dettach` 后将其恢复到 `ApcState` 后才会被交付。所以喽。

`AttachedApcEnvironment`：插入到目标线程 `ApcStatePointer[1]` 代表的链表中。这个的使用发生在目标线程已经 `Attach`，并且插入的 APC 也想在这个 `Attach` 的进程中运行。因为如果目标线程没有 `Attach`，则 `ApcStatePointer[1]` 代表备份，所以 APC 不会被交付；当目标线程真的 `Attach` 的时候，原来的 `SavedApcState` 就会被覆盖掉，那这个 APC 还有什么用？所以不会使用。因此只有目标线程已经 `Attach` 了，APC 被插入到 `ApcStatePointer[1]` 中，此时 `ApcStatePointer[1]` 代表正在被使用的 APC 链表，因此就被交付啦。

`CurrentApcEnvironment`：APC 被插入到初始化时目标线程的 `ApcStatePointer[ApcStateIndex]` 所代表的链表中。使用这个值时，当初始化后和插入之间这段时间没有 `Attach` 或者 `Detach` 时，APC 肯定会被执行(不能保证是在自己进程中还是 `Attach` 进程中)。如果在这之间发生了 `Attach`，则 APC 等待目标线程 `Detach` 后被执行；如果这之间发生了 `Detach`，则 APC 不会被执行。

`InsertApcEnvironment`：APC 被插入到插入时（调用 `KeInsertQueueApc` 时）目标线程的 `ApcStatePointer[ApcStateIndex]` 所代表的链表中。使用这个环境值时，APC 可以被保证肯定会马上执行（所谓的马上指不需要等待 `Dettach`），因为目标线程无论是 `Attach` 还是没有，`ApcStatePointer[ApcStateIndex]` 都代表当前使用的链表。只是不能保证 APC 是在目标线程自己的进程运行还是在 `Attach` 后的进程中运行。

知道了上面的参数后，使用起来就比较简单了，调用 `KeInitializeApc` 初始化一个 APC 后，调用 `KeInsertApc` 插入 APC 就可以了。

别外需要注意的是，APC 的函数都是运行在 `IRQL` 为 `APC_LEVEL` 级别的。

### 应用

APC 被知道的最多的应用应该是对一些异步操作的支持，如 `ReadFile` 的异步方式等。以 `ReadFile` 为例，文件系统发现这是一个异步调用，就会标记为 `Pending`，然后立刻返回。等读取操作完成的时候，在将结果拷贝到用户层的内存空间和调用用户层提供的完成函数时，就用到 APC。

另外，挂钩 APC 相关函数可以达到阻止结束目标进程(`ZwTerminateProcess`)、阻止创建远程线程(`CreateRemoteThread`)、阻止对其调用 `GetContext` `SetContext` 等。（但直接读写进程空间或者 `Attach` 操作是阻止不了的）。

### 总结

如果说其它进程或者线程间通信的方式（比如共享一个对象、一块映射内存、甚至套接字等等）是两个国家间的以合作的方式交流：我需要你的什么信息或者要送给你什么，要得到你的同意才行，那么 APC 就像是间谍：我安插在你选干部的队伍里（插入 APC），等你请我去当你的干部。哪一天我当上了（交付），我就可以做我自己想做的事了，我同国外的沟通不用得到同意（目标进程里的代码可能完全不知道我的存在）。
