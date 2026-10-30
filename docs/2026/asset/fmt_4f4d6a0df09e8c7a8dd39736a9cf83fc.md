
# [原创]Windows内核CVE-2019-1215分析与复现

## 前言

该漏洞源于 `ws2ifsl.sys` 驱动程序在处理内存对象时的缺陷，可能导致释放重引用错误。攻击者可以利用该错误，通过精心构造的输入，访问已释放的内存区域，从而执行恶意代码，提升权限。

本文测试环境：

- 虚拟机：Window10 19H1 x64 1709
- 物理机：Windows11 24H2 x64

## 补丁

![](upload/attach/202508/951494_NFNW3FAGNA7GJCS.webp)

## 漏洞分析

进入驱动的主函数`DriverEntry`中可以看到，驱动程序首先创建了一个名为`\Device\WS2IFSL`的设备供用户态访问：

![](upload/attach/202508/951494_95WPEV79KB3M62B.webp)

驱动程序中实现了多个与用户态交互的函数，如下图所示：

![](upload/attach/202508/951494_QA3R9BCRNZQMXYE.webp)

而`MajorFunction`的下标，实际上对应了Windows内核中的宏定义，对应的字段如下：

| 下标 | 宏定义 | 含义 | 代码里的对应 |
| ---- | ----------------------- | ---------------------------------------- | ----------------------- |
| 0 | `IRP_MJ_CREATE` | 打开设备/文件 | `DispatchCreate` |
| 2 | `IRP_MJ_CLOSE` | 关闭设备/文件 | `DispatchClose` |
| 3 | `IRP_MJ_READ` | 读请求 | `DispatchReadWrite` |
| 4 | `IRP_MJ_WRITE` | 写请求 | `DispatchReadWrite` |
| 14 | `IRP_MJ_DEVICE_CONTROL` | `DeviceIoControl` 请求（用户态 IOCTL） | `DispatchDeviceControl` |
| 18 | `IRP_MJ_CLEANUP` | `CloseHandle` 或 `CancelIo` 之类清理操作 | `DispatchCleanup` |
| 27 | `IRP_MJ_PNP` | 即插即用请求 | `DispatchPnP` |

`IRP_MJ_CREATE`与`IRP_MJ_CLOSE`分别对应了我们在用户态创建文件与关闭文件句柄的请求，需要注意的是对于`IRP_MJ_CLOSE`来说，只有当我们传入的文件句柄对应的那个内核文件的引用计数为0时，才会真正执行。

### DispatchCreate函数

由于`DispatchCreate`对应我们创建文件的函数，而文件句柄这是我们与驱动交互的入口，因此我将从这里开始分析，该函数的实现如图所示：

![](upload/attach/202508/951494_C4AJDZD7KE6ZT4E.webp)

可以看到，函数会根据`SystemBuffer + 0x8`的值来判断我们要创建的是`ProcessFile`还是`SocketFile`，根据微软文档的结构体定义，其中`SystemBuffer`实际上对应的是一片用户缓冲区。

在此之后，会将`FileObject`, `mode`, `SystemBuffer`作为参数传递给要执行的函数，而对于参数`FileObject`来说，当我们用户态获得一个句柄时，内核就会创建对应的`FileObject`，该结构体可以在Windbg中查看到，结构如下：

```c
5: kd> dt nt!\_FILE_OBJECT
+0x000 Type : Int2B
+0x002 Size : Int2B
+0x008 DeviceObject : Ptr64 \_DEVICE\_OBJECT
+0x010 Vpb : Ptr64 \_VPB
+0x018 FsContext : Ptr64 Void
+0x020 FsContext2 : Ptr64 Void
+0x028 SectionObjectPointer : Ptr64 \_SECTION\_OBJECT\_POINTERS
+0x030 PrivateCacheMap : Ptr64 Void
+0x038 FinalStatus : Int4B
+0x040 RelatedFileObject : Ptr64 \_FILE\_OBJECT
+0x048 LockOperation : UChar
+0x049 DeletePending : UChar
+0x04a ReadAccess : UChar
+0x04b WriteAccess : UChar
+0x04c DeleteAccess : UChar
+0x04d SharedRead : UChar
+0x04e SharedWrite : UChar
+0x04f SharedDelete : UChar
+0x050 Flags : Uint4B
+0x058 FileName : \_UNICODE\_STRING
+0x068 CurrentByteOffset : \_LARGE\_INTEGER
+0x070 Waiters : Uint4B
+0x074 Busy : Uint4B
+0x078 LastLock : Ptr64 Void
+0x080 Lock : \_KEVENT
+0x098 Event : \_KEVENT
+0x0b0 CompletionContext : Ptr64 \_IO\_COMPLETION\_CONTEXT
+0x0b8 IrpListLock : Uint8B
+0x0c0 IrpList : \_LIST\_ENTRY
+0x0d0 FileObjectExtension : Ptr64 Void
```

其中`DeviceObject`对应的就是驱动创建的那个设备，也就是`\Device\WS2IFSL`，而`FsContext`则对应了驱动程序自定义的数据，接下来首先分析`CreateProcessFile`函数中，主要部分如下：

![](upload/attach/202508/951494_D343CRQNQ34U356.webp)

函数首先根据我们传入的Handle的值，通过`ObReferenceObjectByHandle`获取到句柄对应的对象，该函数的原型为：

```c++
NTSTATUS ObReferenceObjectByHandle(
[in] HANDLE Handle,
[in] ACCESS\_MASK DesiredAccess,
[in, optional] POBJECT\_TYPE ObjectType,
[in] KPROCESSOR\_MODE AccessMode,
[out] PVOID \*Object,
[out, optional] POBJECT\_HANDLE\_INFORMATION HandleInformation
);
```

而在此处，由于传入的`ObjectType`参数为`PsThreadType`，表示我们期望获取的对象为一个线程对象，当获取成功后会进行判断线程对象是否属于当前进程，如果是的话，则通过`ExAllocatePoolWithQuotaTag`函数分配一个大小为0x108的内存池，而其中第一个参数为`POOL_TYPE`，这是一个枚举类型，根据查阅微软的文档，可以知道实际上函数中的512对应的是`NonPagedPoolNx`，也就是非分页池，其中的`Nx`代表了权限位为不可执行。

当内存申请成功后，会进行一系列的赋值操作，并且进入`InitializeRequestQueue`，由于赋的值是用户不可控制的，因此对于大部分值我们不用太过关心，只需要关注在`buffer + 0x0`处存储了字符串`corP`，`buffer + 0x8`存储了当前进程的PID，并且在`InitializeRequestQueue`的如图所示的位置：

![](upload/attach/202508/951494_D6ZNP63EPGVTBQ3.webp)

我们可以看到驱动程序将APC对象挂在了`buffer->ApcObject`的位置，该结构体实际上是我自己恢复的，对应的偏移实际上为`buffer+0x30`处，而第二个参数则是指明了要把APC挂在哪个线程上，这里的`pthread_Object`实际就是根据我们前面传入的句柄所获取到的，是用户可控的。

当赋值操作完成后，会将`buffer`挂在`fileObject + 0x18`的位置，根据前面给出的结构体定义，我们可以知道该偏移对应的是`FsContext`。

而对于`CreateSocketFile`函数，大致与`CreateProcessFile`的前置逻辑相同，函数如下：

![](upload/attach/202508/951494_VTDQBXYCDVJ765B.webp)

可以看到区别主要存在于赋值不同与`ObReferenceObjectByHandle`的不同，先看`ObReferenceObjectByHandle`，根据前面我们给出的函数原型可以知道，此时的`ObjectType`变为了`IoFileObjectType`，这代表文件对象类型，如用户空间打开的 **文件/设备句柄**，那应该传入哪个文件或设备的句柄呢？

我们继续看下面的一段逻辑，首先将`Object[0]`给了`v10`，并通过`IoGetRelatedDeviceObject`来获取与`Object[0]`相关的是设备，其中`DeviceObject`这个全局变量对应的实际上就是驱动程序所注册的设备：`\Device\WS2IFSL`，然后取值判断，这里取得值有一点绕，我们拆开来看。

- 首先`v10`对应的是`Object[0]`，这是一个指针数组，因此`*v10`指向了某个对象
- 对于`**(v10 + 3)`运算来说，这里看伪代码感觉有点不明白这个解引用的顺序，实际上看汇编很容易理解，这里`v10`对应的是`RSI`寄存器，而定位到对应的汇编可以看见，实际上进行的操作为`mov rax, [rsi+18h]`，而这个操作实际上就是取出`v10`指向的对象，取出偏移为`0x18`的位置的指针，再取出该指针偏移为`0x8`处的内容
- 对于`*(*(v10 + 3) + 8LL)`来说，实际上就是取出`v10`指向的对象，取出偏移为`0x18`的位置的指针，再取出该指针偏移为`0x8`处的内容

而根据条件判断的内容，驱动期望该指针偏移0处的内容为`corP`，期望该指针偏移8处的内容为PID，再根据前面`CreateProcessFile`函数对于赋值的处理，我们可以知道对于`ProcessFile`的`fileObject`，偏移`0x18`实际上对应了`FsContext`，而`*(FsContext)`与`*(FsContext + 0x8)`正好对应了字符串`corP`与进程的PID，因此我们可以知道，要传入的句柄实际上就是`ProcessFile`对应的句柄。

对于后面的赋值同样不用太过关注，只需要知道`buffer + 0x0`对应了字符串`kcoS`，而`buffer + 0x10`处存放了指向`ProcessFile`的`fileObject`的指针。

### DispatchReadWrite函数

根据`MajorFunction`数组的下标的宏定义可以知道，该函数对应了两种请求，分别为`IRP_MJ_READ`与`IRP_MJ_WRITE`，也就是对应了读写请求，该函数如下：

![](upload/attach/202508/951494_V68DXCPSJ3WNB79.webp)

从图中我们可以看出，实际上读写请求只对应了`SocketFile`，而进入函数`DoSocketReadWrite`之后，我们可以看到：

![](upload/attach/202508/951494_UC63W84FHA46B9B.webp)

函数首先将`fileObject + 0x18`处的值取出，这个偏移不论是对于`SocketFile`还是`ProcessFile`来说都是`FsContext`，而接下来，函数将`FsContext`作为参数传入了`GetSocketProcessReference`，该函数如下：

![](upload/attach/202508/951494_QTHZV3BTYHY2BWW.webp)

先是与锁相关的操作，而后将`FsContext->Processfile_ptr`引用计数加1，然后返回了`FsContext->Processfile_ptr`，而根据前面的分析，我们可以知道这里`FsContext->Processfile_ptr`指向的是`ProcessFile`的`fileObject`，而需要注意的是，在这里引用计数+1的操作也只针对了`ProcessFile`的`fileObject`，而并未针对`FsContext`指向的缓冲区。

接下来在返回了`fileObject`后，将其作为参数传入了`QueueRequest`函数：

![](upload/attach/202508/951494_64H56SY3SY6NTFM.webp)

这里由于我结构体恢复的某些字段可能不太对，导致看着很奇怪，实际上我分析了一下，这里似乎是在将某个节点挂入链表的尾部。

而后执行了`SignalRequest`：

![](upload/attach/202508/951494_79MJBY4CKZ8JJKW.webp)

该函数最重要的一点就是调用了`KeInsertQueueApc`函数，将APC挂入了我们传入的线程对应的内核线程的APC列表中，并等待执行。

这里要注意的是，一旦APC被挂入了列表，那么即使我们关闭了句柄，也可以通过`NtTestAlert`等函数来强制执行。

### DispatchClose函数

该函数对应的请求是`IRP_MJ_CLOSE`请求，也就是对应的我们在用户态关闭句柄的操作，但此操作真正的执行时机是当我们传入的句柄的引用计数为0时，该函数如下：

![](upload/attach/202508/951494_S9G7P856WHN4FBX.webp)

我们可以看见，该函数会根据`*(FsContext)`的值来选择关闭哪个句柄，我们关注值为`corP`的情况，假设我们在用户态分别打开了`ProcessFile`与`SocketFile`，并用`SocketFile`进行读写，当我们关闭`ProcessFile`时，并不会触发`DispatchClose`，因为根据我们之前的分析，在`SocketFile`的读写操作中会增加`ProcessFile`的引用计数（在创建时也会增加引用计数），那么读写完毕后，引用计数减1，由于创建时也增加了引用计数，因此此时引用计数为1，而后我们关闭`SocketFile`，此时引用计数为0，因此`ProcessFile`与`FsContext`被释放，而此时APC已经被挂在了线程的APC队列中，造成了UAF。

至此，漏洞的成因已经基本分析完毕，我们总结一下，个人认为该漏洞的根本原因在于两点：

- 缺乏细粒度的引用计数，因为只对`fileObject`进行了引用计数，但其中的`FsContext`并没有计算引用计数，只是跟着`fileObject`一起进行了释放
- APC队列中的函数是可以不依赖于句柄调用的，因此在缓冲区释放以后还可以使用

接下来就是分析该如何进行漏洞利用了。

## 漏洞利用

首先我们思考一下我们可以控制哪些东西，如何进行交互。

对于该漏洞而言，我们可以控制`ProcessFile`对应的线程对象，也就是说可以控制将APC绑定到哪个线程对象上，而`SocketFile`实际则是依赖于我们创建的`ProcessFile`对象。

因此一个基本的交互思路是，我们创建一个`ProcessFile`与`SocketFile`，并获取到对应的句柄，但这里需要注意一个关键点，在驱动程序注册设备时，仅仅使用了以下代码：

![](upload/attach/202508/951494_QB6RGCDKJQXDZR2.webp)

对于`IoCreateDevice`来说，`DeviceObject` 是内核态的设备对象（`DEVICE_OBJECT`），存在于内核空间，内核里这个对象表示该驱动，但它 **本身并没有用户态可访问的名字**。因此正常的`CreateFile`是无法获得到句柄的，而根据<a href="elink@879K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9j5h3u0K6i4K6u0W2j5X3I4#2k6h3k6J5L8%4y4@1M7$3g2U0N6i4u0A6N6s2W2Q4x3X3g2V1k6g2)9J5c8X3u0D9L8$3N6Q4x3V1j5J5x3o6t1H3i4K6u0r3x3o6q4Q4x3V1j5H3y4#2)9J5c8X3y4$3k6g2)9J5k6o6t1H3x3e0W2Q4x3X3b7I4x3U0p5#2i4K6u0V1j5h3&6S2L8s2W2K6K9i4y4Q4x3X3c8G2k6W2)9J5k6r3q4Q4x3X3c8#2M7$3g2Q4x3X3c8S2k6Y4c8W2M7W2)9J5k6r3k6J5k6h3g2Q4x3X3c8A6L8W2)9J5k6s2N6K6x3X3W2X3M7$3I4Q4x3V1j5`.">bluefrostlab</a>的文章，我们可以用`NtCreateFile`来获取句柄。

那么此时，我们需要思考如何组织我们要传入的数据（也就是线程的句柄）。

为了传入某个线程句柄，我们首先需要创建一个线程：
