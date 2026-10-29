
# [原创]堆学习记录

[toc]

准备刷漏洞战争中的堆溢出部分，但是对于堆的了解较少，在这里记录一下关于堆的学习记录，有错误请各位大大拍砖

参考:

* 《软件调试》
* 《0day安全 软件漏洞分析技术》

# win32下的堆管理系统

## 1. win32堆管理器

win32堆管理器由`NTDLL.dll`实现，目的是为用户态的应用程序提供内存服务,从实现角度上来讲，内核态的池管理器和用户态的win32堆管理器默用的是同一套基础代码，它们以运行时的方式存在于`ntdll.dll`和`ntosknrl.exe`模块中。

## 2. CRT堆管理器

* CRT由C运行时库创建，CRT创建的堆有三种模式，分别是`SBH`，`ODLSBH`和`System Heap`模式，CRT运行时库选择一种模式创建相应的堆。
* 对于`SBH`和`OLDSBH`模式来说，CRT堆会从堆管理器中批发大块的内存，然后分割成小块的内存供程序使用，对于系统模式，CRT堆只是将堆分配请求转发给它基于的win32堆。因此处于系统模式的CRT堆只是对win32堆的简单封装。

# win32堆

## 1.体系结构

应用程序通过堆管理器提供的接口来分配或者是释放内存，整个堆管理器由堆管理api，前端堆，后端堆组成。在处理内存的分配和释放的时候，前端堆先做处理，前端堆有三种选择:`none`,预读列表(快表,`lookaside-list`),`LFH`(低碎片堆)。

## 2.堆的内部结构

和堆相关的数据结构主要有以下几个:

* `\_HEAP`
* `\_HEAP\_SEGMENT`
* `\_HEAP\_ENTRY`
* `\_HEAP\_FREE\_ENTRY`

堆管理器通过`\_HEAP`来记录和维护堆的管理信息，在`\_HEAP`结构中，可能存在一个或者若干个`\_HEAP\_SEGMENT`的信息。在我们创建堆时，堆管理器必然会像操作系统申请一块内存空间，那么申请的内存大小和这段内存空间的描述信息就由`\_HEAP\_SEGMENT`记录。在每一个`\_HEAP\_SEGMENT`结构中，可能存在若干个`\_HEAP\_ENTRY`和`\_HEAP\_FREE\_ENTRY`结构，其中`\_HEAP\_ENTRY`和`\_HEAP\_FREE\_ENTRY`都是对堆块的描述，只不过`\_HEAP\_FREE\_ENTRY`描述的是空闲的堆快结构，而由`\_HEAP\_ENTRY`描述的不一定都是空闲的堆块。

其中`\_HEAP\_ENTRY`和`\_HEAP\_FREE\_ENTRY`的结构体如下:

```
0:000> dt \_HEAP\_ENTRY
ntdll!\_HEAP\_ENTRY
+0x000 Size : Uint2B
+0x002 PreviousSize : Uint2B
+0x000 SubSegmentCode : Ptr32 Void
+0x004 SmallTagIndex : UChar
+0x005 Flags : UChar
+0x006 UnusedBytes : UChar
+0x007 SegmentIndex : UChar
0:000> dt ntdll!\_HEAP\_FREE\_ENTRY
+0x000 Size : Uint2B
+0x002 PreviousSize : Uint2B
+0x000 SubSegmentCode : Ptr32 Void
+0x004 SmallTagIndex : UChar
+0x005 Flags : UChar
+0x006 UnusedBytes : UChar
+0x007 SegmentIndex : UChar
+0x008 FreeList : \_LIST\_ENTRY
```

* **ps:上面两个结构体中比较坑的是，在xp sp3下，通过windbg查看`\_HEAP\_ENTRY`和`\_HEAP\_FREE\_ENTRY`结构体中的第四个字段为`SmallTagIndex`，但实际上这个字段应该是安全cookie,算法如下:`\_HEAP.\_HEAP\_ENTRY.cookie=\_HEAP\_ENTRY.cookie^((BYTE)&\_HEAP\_ENTRY/8)`**
* **在win7下，直接用`\_HEAP\_ENTRY`和`\_HEAP\_FREE\_ENTRY`查看数据，会存在问题，需要用`\_HEAP.ENCODING`字段进行解密。**

### `\_HEAP`结构

windows xp下的`\_HEAP`结构

```
0:000> dt ntdll!\_HEAP 003b0000
+0x000 Entry : \_HEAP\_ENTRY
+0x008 Signature : 0xeeffeeff ;heap结构签名
+0x00c Flags : 0x50001060 ;堆属性
+0x010 ForceFlags : 0x40000060 ;
+0x014 VirtualMemoryThreshold : 0xfe00;最大堆块大小
+0x018 SegmentReserve : 0x100000
+0x01c SegmentCommit : 0x2000
+0x020 DeCommitFreeBlockThreshold : 0x200
+0x024 DeCommitTotalFreeThreshold : 0x2000
+0x028 TotalFreeSize : 0x130;空闲块总大小，以粒度为单位
+0x02c MaximumAllocationSize : 0x7ffdefff;堆可分配的最大内存
+0x030 ProcessHeapsListIndex : 6;堆在peb.processheaps数组中的索引+1
+0x032 HeaderValidateLength : 0x608
+0x034 HeaderValidateCopy : (null)
+0x038 NextAvailableTagIndex : 0
+0x03a MaximumTagIndex : 0
+0x03c TagEntries : (null)
+0x040 UCRSegments : (null)
+0x044 UnusedUnCommittedRanges : 0x003b0598 \_HEAP\_UNCOMMMTTED\_RANGE
+0x048 AlignRound : 0x17
+0x04c AlignMask : 0xfffffff8
+0x050 VirtualAllocdBlocks : \_LIST\_ENTRY [ 0x3b0050 - 0x3b0050 ]
+0x058 Segments : [64] 0x003b0640 \_HEAP\_SEGMENT
+0x158 u : \_\_unnamed
+0x168 u2 : \_\_unnamed
+0x16a AllocatorBackTraceIndex : 0
+0x16c NonDedicatedListLength : 1
+0x170 LargeBlocksIndex : (null)
+0x174 PseudoTagEntries : (null)
+0x178 FreeLists : [128] \_LIST\_ENTRY [ 0x3b0688 - 0x3b0688 ];空闲块
+0x578 LockVariable : 0x003b0608 \_HEAP\_LOCK
+0x57c CommitRoutine : (null)
+0x580 FrontEndHeap : (null)
+0x584 FrontHeapLockCount : 0
+0x586 FrontEndHeapType : 0 ''
+0x587 LastSegmentIndex : 0 ''
```

在这个结构体中,`FreeLists`是一个由128个`LIST\_ENTRY`结构体组成的结构体数组，在调用`HeapCreate`时，`Freelists[0]`指向尾块，尾块也就是堆刚创建出时最大的一个空闲块，数组中的其他元素指向自己。

## 3.创建进程时操作系统为进程创建的默认堆

`ntdll!KiUserApcDispather`->`ntdll!\_LdrpInitialize`->`ntdll!LdrplInitializeProcess`->`ntdll!RtlCreateHeap`,创建好的堆的句柄回保存在`PEB`结构的`ProcessHeap`字段中，`PEB`中关于堆的字段如下：

```
0:000> dt ntdll!\_PEB
+0x018 ProcessHeap : Ptr32 Void //堆句柄
+0x078 HeapSegmentReserve : Uint4B //默认堆的保留大小:1MB
+0x07c HeapSegmentCommit : Uint4B //默认堆的提交大小:8kb
+0x088 NumberOfHeaps : Uint4B //进程中堆的总数
+0x08c MaximumNumberOfHeaps : Uint4B //ProcessHeaps数组中数组元素的个数
+0x090 ProcessHeaps : Ptr32 Ptr32 Void//这个数组用来保存进程中创建的堆的句柄
```

## 4.私有堆

### 创建

可以通过`HeapCreate`这个api来创建属于进程的私有堆，这个api实际上回去调用`RtlCreateHeap`函数，创建完毕之后会将创建好的堆句柄保存到peb结构中。`HANDLE WINAPI HeapCreate(DWORD flOptions,SIZE\_T dwInitialSize, SIZE\_T dwMaximumSize)`这个api中，`flOptions`下面三个可选项:

* `HEAP\_CREATE\_ENABLE\_EXECUTE` 允许执行堆中的代码，也就是这时创建出的堆具有可执行属性。
* `HEAP\_GENERATE\_EXCEPTIONS` 使用异常来报告`HeapAlloc`和`HeapReAlloc`的错误而不是通过返回NULL。
* `HEAP\_NO\_SERIALIZE` 当堆函数访问这个堆时，不需要进行串行化控制。这么做可以提高操作堆的速度，但是在多线程同时操作堆时，可能出现问题。当创建堆时指定这个参数时，不能启用低碎片堆(`LFH`)。

在windbg中可以使用`!heap -h`指令来查看进程中的所有堆。`!heap -h`的查询结果也就是`peb.ProcessHeaps`中的值。

### 销毁

`HeapDestory`->`ntdll!RtlDestroyHeap`->`NtFreeVirtualMemory`。

`ntdll!RtlDestroyHeap`会将`PEB`堆列表中的堆要销毁堆的堆句柄移除掉，然后通过`NtFreeVirtualMemory`函数向内存管理器归还内存。

应用程序不应该也不许销毁默认堆,当进程退出和销毁进程对象时，系统都会调用`MnCleanProcessSpace`函数。

# 堆内存的申请和释放

通过`HeapAlloc`这个api可以从堆中分配空间出来,`LPVOID WINAPI HeapAlloc`(HANDLE hHeap,DWORD dwFlags,SIZE\_T dwBytes)`, `dwFlags`有下面几个选项:

* `HEAP\_GENERATE\_EXCEPTIONS` 系统通过返回异常而不是返回NULL来报告错误
* `HEAP\_NO\_SERIALIZE` 不对堆进行串行话控制。
* `HEAP\_ZERO\_MEMORY` 分配的内存将被初始化为0。

ps:指定这些值中的任何值将覆盖使用`HeapCreate`创建堆时指定的对应值 。

也可以使用`malloc`，`calloc`或者是`new`运算符来从堆中分配内存，上面提到CRT堆有三种模式，大多数情况下，运行时库都会选择系统模式，此时CRT堆建立在win32堆之上。

通过`HeapFree`这个api可以从释放`HeapAlloc`申请处的内存空间，`free`或者是`delete`最终都会跳到这个api上来释放内存。

## winxp 堆内存释放和合并-空闲双向链表

实验代码:

```
#include <stdio.h>
#include <windows.h>
main()
{
HLOCAL h1,h2,h3,h4,h5,h6;
HANDLE hp;
hp = HeapCreate(0,0x1000,0x10000);
getchar();
\_\_asm int 3
h1 = HeapAlloc(hp,HEAP\_ZERO\_MEMORY,3);
h2 = HeapAlloc(hp,HEAP\_ZERO\_MEMORY,5);
h3 = HeapAlloc(hp,HEAP\_ZERO\_MEMORY,6);
h4 = HeapAlloc(hp,HEAP\_ZERO\_MEMORY,8);
h5 = HeapAlloc(hp,HEAP\_ZERO\_MEMORY,19);
h6 = HeapAlloc(hp,HEAP\_ZERO\_MEMORY,24);
HeapFree(hp,0,h1); //free to freelist[2]
HeapFree(hp,0,h3); //free to freelist[2]
HeapFree(hp,0,h5); //free to freelist[4]
HeapFree(hp,0,h4); //coalese h3,h4,h5,link the large block to freelist[8]
return 0;
}
```

执行程序之后用windbg附加，然后f5，断到int 3，在windbg中查看空闲表

```
0:000>dd 0x003a0000+0x178
003a0178 003a0688 003a0688 003a0180 003a0180
003a0188 003a0188 003a0188 003a0190 003a0190
003a0198 003a0198 003a0198 003a01a0 003a01a0
003a01a8 003a01a8 003a01a8 003a01b0 003a01b0
```

`Heap.FreeLists[0]`的flink和blink分别都指向`0x003a0688`.通过`!heap -a 003a0000`查到的结果，我们来看分配到的堆块结构,在windbg中输入

`!heap -x 003a0640`或者是`dt ntdll!\_HEAP\_ENTRY`来查看堆结构:

```
0:000> !heap -x 003a0640
Entry User Heap Segment Size PrevSize Unused Flags
-----------------------------------------------------------------------------
003a0640 003a0648 003a0000 003a0640 40 640 0 busy
0:000> dt ntdll!\_HEAP\_ENTRY 003a0640
+0x000 Size : 8
+0x002 PreviousSize : 0xc8
+0x000 SubSegmentCode : 0x00c80008
+0x004 SmallTagIndex : 0 ''
+0x005 Flags : 0x1 ''
+0x006 UnusedBytes : 0 ''
+0x007 SegmentIndex : 0 '
```

其中`\_HEAP\_ENTRY.Size`为堆块的大小/堆块分配粒度，堆块分配粒度一般都为8，可以用`!heap -v`堆基址来查看

然后单步执行完 `h1 = HeapAlloc(hp,HEAP\_ZERO\_MEMORY,3)`，观察空闲块的变化

```
0:000> dd 0x003a0000+0x178
003a0178 003a0698 003a0698 003a0180 003a0180
003a0188 003a0188 003a0188 003a0190 003a0190
003a0198 003a0198 003a0198 003a01a0 003a01a0
003a01a8 003a01a8 003a01a8 003a01b0 003a01b0
```

此时`Heap.FreeLists[0]`的flink和blink分别都指向`0x003a0698`,在windbg中输入`!heap -x 0x003a0698`或者是`dt ntdll!\_HEAP\_FREE\_ENTRY 0x003a0698-8`来查看堆结构:

```
0:000> !heap -x 003a0698
Entry User Heap Segment Size PrevSize Unused Flags
-----------------------------------------------------------------------------
003a0690 003a0698 003a0000 003a0640 970 10 0 free last
0:000> dt ntdll!\_HEAP\_FREE\_ENTRY 003a0698-8
+0x000 Size : 0x12e
+0x002 PreviousSize : 2
+0x000 SubSegmentCode : 0x0002012e
+0x004 SmallTagIndex : 0 ''
+0x005 Flags : 0x10 ''
+0x006 UnusedBytes : 0 ''
+0x007 SegmentIndex : 0 ''
+0x008 FreeList : \_LIST\_ENTRY [ 0x3a0178 - 0x3a0178 ]
```

在`\_HEAP\_FREE\_ENTRY`中多了一个`LIST\_ENTRY`成员，而这个成员的flink和blink都只想freelist[0]。单步执行完`HeapFree(hp,0,h1);`在windbg中观察下内存的变化:
