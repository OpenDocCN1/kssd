
# [原创]遍历系统中的回调

**目的：**遍历系统中的回调

**类型：**

- 与Xuetr遍历到的类型相同
- 如有雷同，还望见谅。。。有错误或者不恰当的地方请指正。
- 附件中代码大量冗余，可以将相同的部分写成一个函数，一开始没注意，懒得改了。。。

详细实现见代码

**环境：**WIN XP SP3 大量使用硬编码，本次仅在于实现我的虚拟机环境下的遍历，没有考虑寻找通用方法。

**参考资料：**

1. WRK 1.2
2. React OS
3. 标 题：替换进程创建监视函数
   作 者：鹿剑
   链 接：http://bbs.pediy.com/showthread.php?t=93759

**将包括：**

1. 创建进程、创建线程、加载模块回调
2. 注册表回调
3. 错误检测回调
4. Lego回调
5. 关机回调
6. 注销回调
7. 文件系统改变回调
8. 即插即用回调（还未完成，忘大家给些提示，我好补充完整；已经完成，见下面：（从2000里面抄抄捡捡，大牛莫笑。。。![](https://bbs.kanxue.com/view/img/face/50.gif)））

其实实现起来都差不多，一个链表头串起来，好几种只是结构不同而已

## 1. 创建进程、创建线程、加载模块回调

### 创建进程回调

注册创建进程回调使用函数`PsSetCreateProcessNotifyRoutine`，调用这个函数会将注册的信息保存到一个数组里面。

反汇编这个函数，会发现以下片段：

```
805d0c27 56              push    esi
805d0c28 57              push    edi
805d0c29 7464            je      nt!PsSetCreateProcessNotifyRoutine+0x73 (805d0c8f)
805d0c2b bf404a5680      mov     edi,offset nt!PspCreateProcessNotifyRoutine (80564a40)
805d0c30 57              push    edi
```

`PspCreateProcessNotifyRoutine`（`80564a40`）即这个数组的地址，这个数组是以下结构：

```
typedef struct _EX_FAST_REF
{
    union
    {
        PVOID Object;
        ULONG_PTR RefCnt:3;
        ULONG_PTR Value;
    };
} EX_FAST_REF, *PEX_FAST_REF;
```

低三位`RefCnt`是引用指针，`Value`指向一个结构，如下：

```
typedef struct _EX_CALLBACK_ROUTINE_BLOCK
{
    EX_RUNDOWN_REF RundownProtect;
    PEX_CALLBACK_FUNCTION Function;
    PVOID Context;
} EX_CALLBACK_ROUTINE_BLOCK, *PEX_CALLBACK_ROUTINE_BLOCK;
```

可以看到保存了回调函数的地址。

因而采用搜索数组的方式在进行遍历即可，因为在 WIN XP SP3 中数组元素为8个，所以遍历的时候就按八个来的，WIN 7 就不是了。

### 创建线程和加载模块回调

（2）创建线程和加载模块一样，略过。。。

## 2. 注册表回调

一开始真不知道哪个函数是注册注册表回调，尝试着在 MSDN 以 `cm`（Xuetr回调类型里就是`cmpCallBack`）开头看到了`CmRegisterCallback`，大致看了一下，跟上面的原理一样。。不过是不是八个元素没注意，按八个处理的。

## 3. 错误检测回调

函数是：

- `KeRegisterBugCheckCallback`
- `KeRegisterBugCheckReasonCallback`

反汇编`KeRegisterBugCheckCallback`:

```
804fa138 c7400458d75580  mov     dword ptr [eax+4],offset nt!KeBugCheckCallbackListHead (8055d758)
```

又一个链表头，查看WRK，这里没有考虑WRK与WIN XP 在该函数上可能的差异。。。得到以下信息：

```
/*
NTKERNELAPI

BOOLEAN

KeRegisterBugCheckCallback (
    __out PKBUGCHECK_CALLBACK_RECORD CallbackRecord,
    __in PKBUGCHECK_CALLBACK_ROUTINE CallbackRoutine,
    __in PVOID Buffer,
    __in ULONG Length,
    __in PUCHAR Component
)

typedef struct _KBUGCHECK_CALLBACK_RECORD {
    LIST_ENTRY Entry;
    PKBUGCHECK_CALLBACK_ROUTINE CallbackRoutine;
    PVOID Buffer;
    ULONG Length;
    PUCHAR Component;
    ULONG_PTR Checksum;
    UCHAR State;
} KBUGCHECK_CALLBACK_RECORD, *PKBUGCHECK_CALLBACK_RECORD;

LIST_ENTRY KeBugCheckCallbackListHead;
LIST_ENTRY KeBugCheckReasonCallbackListHead;
*/
```

结构类型就很清楚了，得到链表头，按照双向链表便利即可。

## 4. Lego回调

刚从XUETR里面看到这个还真不知道是什么，搜索一番后，得到了MJ的一些说明：

`ULONG PsSetLegoNotifyRoutine(PVOID notifyroutine)`

`notifyroutine`为需要设置的回调函数地址。

返回值是`_ETHREAD->Tcb->LegoData`的偏移量。

调用此函数，系统会将一个未导出的全局变量`_PspLegoNotifyRoutine`设置为你设定的回调函数地址。

当一个线程的`_ETHREAD->Tcb->LegoData`不为空，且`_PspLegoNotifyRoutine`不为空，那么当这个线程调用`PspExitThread`退出时，会调用`PspLegoNotifyRoutine`中的回调函数。

系统中只允许设置一个这样的回调函数。

有人跟着给出具体结构和定义：

```
typedef VOID (*PLEGO_NOTIFY_ROUTINE)( PKTHREAD Thread );

ULONG PsSetLegoNotifyRoutine( PLEGO_NOTIFY_ROUTINE LegoNotifyRoutine )
{
    PAGED_CODE();
    PspLegoNotifyRoutine = LegoNotifyRoutine;
    return FIELD_OFFSET(KTHREAD,LegoData);
}
```

硬编码搜索到这个全局变量，查看地址是否为0即可（不知是否有误，或者加上地址有效检测？）

```
nt!PsSetLegoNotifyRoutine:
805d299a 8bff            mov     edi,edi
805d299c 55              push    ebp
805d299d 8bec            mov     ebp,esp
805d299f 8b4508          mov     eax,dword ptr [ebp+8]
805d29a2 a3c0d26780      mov     dword ptr [nt!PspLegoNotifyRoutine (8067d2c0)],eax//这里
805d29a7 b8d0000000      mov     eax,0D0h
805d29ac 5d              pop     ebp
805d29ad c20400          ret     4
```

## 5. 关机回调

有两个函数：

- `IoRegisterShutdownNotification`
- `IoRegisterLastChanceShutdownNotification`

来看第一个：

相关结构和定义也是参考了WRK。。。。

```
LIST_ENTRY IopNotifyShutdownQueueHead;
LIST_ENTRY IopNotifyLastChanceShutdownQueueHead;

typedef struct _SHUTDOWN_PACKET {
    LIST_ENTRY ListEntry;
    PDEVICE_OBJECT DeviceObject;
} SHUTDOWN_PACKET, *PSHUTDOWN_PACKET;
```

要得到回调函数地址，需要得到设备对象，回调函数地址即`IRP_MJ_SHUTDOWN`的例程地址。

代码如下：

```
//#define IRP_MJ_SHUTDOWN                 0x10

ULONG GetNotifyAddr(PDEVICE_OBJECT Device_Object)
{
    ULONG Addr;
    PDRIVER_OBJECT Driver_Object;
    Driver_Object = Device_Object->DriverObject;
    Addr = *(ULONG*)((ULONG)Driver_Object + 0x38 + 0x40);
    return Addr;
}
```

## 6. 注销回调

搜了N久也不知道是哪个函数，于是看了看Xuetr，只有`mrxsmb.sys`注册了注销回调，查看其导入表看看有没有跟注册相关的字眼，于是找到了`SeRegisterLogonSessionTerminatedRoutine`，查了一下，果然是。

也是一个链表头的形式，相关结构和定义：

```
typedef NTSTATUS (*PSE_LOGON_SESSION_TERMINATED_ROUTINE) (
  IN PLUID LogonId
);

typedef struct _SEP_LOGON_SESSION_TERMINATED_NOTIFICATION {
    struct _SEP_LOGON_SESSION_TERMINATED_NOTIFICATION *Next;
    PSE_LOGON_SESSION_TERMINATED_ROUTINE CallbackRoutine;
} SEP_LOGON_SESSION_TERMINATED_NOTIFICATION, *PSEP_LOGON_SESSION_TERMINATED_NOTIFICATION;

NTSTATUS
SeRegisterLogonSessionTerminatedRoutine(
    IN PSE_LOGON_SESSION_TERMINATED_ROUTINE CallbackRoutine
);
```

## 7. 文件系统改变回调

相关的注册函数是：`IoRegisterFsRegistrationChange`

结构：

```
typedef struct _NOTIFICATION_PACKET
{
    LIST_ENTRY ListEntry;
    PDRIVER_OBJECT DriverObject;
    ULONG NotificationRoutine;
} NOTIFICATION_PACKET, *PNOTIFICATION_PACKET;
```

## 8. 即插即用回调

### 初次尝试

（1）注册的函数应该是`IoRegisterPlugPlayNotification`（由于搞不定，我都开始怀疑了。。。）

一开始想用上面的方法找到链表头，遍历链表即可，但是发现结果不对。。。

我找的是`IopDeferredRegistrationList`，是不是找错了？

WRK里面没有这个函数的实现，于是翻了翻React OS，找到参考的代码有这么几句：

```
InsertHeadList(&PnpNotifyListHead, &Entry->PnpNotifyList);
KeReleaseGuardedMutex(&PnpNotifyListLock);
DPRINT("IoRegisterPlugPlayNotification() returns NotificationEntry %p\n",
        Entry);
*NotificationEntry = Entry;
```

其中相关结构：

```
typedef struct _PNP_NOTIFY_ENTRY
{
    LIST_ENTRY PnpNotifyList;
    IO_NOTIFICATION_EVENT_CATEGORY EventCategory;
    PVOID Context;
    UNICODE_STRING Guid;
    PFILE_OBJECT FileObject;
    PDRIVER_NOTIFICATION_CALLBACK_ROUTINE PnpNotificationProc;
} PNP_NOTIFY_ENTRY, *PPNP_NOTIFY_ENTRY;
```

结果完全对不上。。。。。从`ntoskrnl.exe`中找到这个函数，无奈逆向太烂看不出来将注册信息保存到哪了。。。貌似是一个常量指向的地址。。。

（2）从函数实现来看，注册成功后，将相关结构的地址返回了，如果按照React OS的做法，这个结构是在整个链表里面的，结果自己注册了一个回调打算进行遍历，还是不行。。。。

还望大家指点。。。。

其余结果都能和Xuetr对得上

### 补充：Windows 2000源码实现

今天下了一份2000源代码回来，找到了`IoRegisterPlugPlayNotification`函数的实现部分，发现了一个链表`IopDeviceClassNotifyList`，用Windbg查看这个链表，得到如下信息：

```
|------------------------|
1.8067d080 -> e1dd7698 ->--^
  8067d080 <- e1dd7698 <----
  |________________________|
  |------------------------------------------------|
2.e1336d98 -> e1036468 -> e1307548 -> 8067d090 ->--^
  e1307548 <- 8067d090 <- e1336d98 <- e1036468 <----
  |________________________________________________|
  |------------------------------------------------|
4.e1000888 -> e1f58478 -> e1bc04a0 -> 8067d0a0 ->--^
  e1bc04a0 <- 8067d0a0 <- e1000888 <- e1f58478 <----
  |________________________________________________|
  |------------------------|
5.e1537858 -> 8067d0a8 ->--^
  e1537858 <- 8067d0a8 <----
  |________________________|
  |------------------------|
6.e156e7c8 -> 8067d0b0 ->--^
  e156e7c8 <- 8067d0b0 <----
  |________________________|
   |------------------------|
10.e1315d60 -> 8067d0d0 ->--^
   e1315d60 <- 8067d0d0 <----
   |________________________|
   |------------------------------------------------|
11.e1002ad8 -> e156fc48 -> e1301258 -> 8067d0d8 ->--^
   e1301258 <- 8067d0d8 <- e1002ad8 <- e156fc48 <----
   |________________________________________________|
   |------------------------------------------------------------|
12.e1023c40 -> e131ae08 -> e101ea20 -> e1f752c8 -> 8067d0e0 ->--^
   e1f752c8 <- 8067d0e0 <- e1023c40 <- e131ae08 <- e101ea20 <----
```
