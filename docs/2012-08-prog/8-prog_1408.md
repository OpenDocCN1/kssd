
# 暴力搜索内存枚举进程的学习笔记

> 原文：[`www.pediy.com/kssd/pediy10/75681.html`](https://www.pediy.com/kssd/pediy10/75681.html)

在前面的一篇文章中我写了点关于用 `PspCidTable` 枚举进程的学习笔记。不过一种方法并不能满足，所以又在网上找了一点关于暴力搜索内存枚举进程的资料，然后写了一些代码。最后写这篇文章作为自己的学习笔记。

既然是搜索，就要明白以下几个问题：

1. 关于进程，我们应该搜索它的什么结构？
2. 从哪里搜索到哪里结束？
3. 如果空间很大，如何加速搜索？
4. 如何防止访问无效内存空间造成 `BSOD`？

## 1. 关于搜索进程的什么结构

我在网上找了一下，大概搜索的是进程的 `PEB`（`PEB` (`Process Environment Block`) 进程环境块）。主要原因是 `PEB` 地址的高 4 位是相同的。

```
PROCESS 85982da0  SessionId: none  Cid: 03f4    Peb: 7ffde000  ParentCid: 0004
    DirBase: 0b0d0020  ObjectTable: e1010d50  HandleCount:  20.
    Image: smss.exe

PROCESS 85953880  SessionId: none  Cid: 0404    Peb: 7ffdc000  ParentCid: 03f4
    DirBase: 0b0d0040  ObjectTable: 00000000  HandleCount:   0.
    Image: autochk.exe

PROCESS 8597eda0  SessionId: 0  Cid: 043c    Peb: 7ffd9000  ParentCid: 03f4
    DirBase: 0b0d0080  ObjectTable: e17eac78  HandleCount: 608.
    Image: csrss.exe

PROCESS 858cb618  SessionId: 0  Cid: 0458    Peb: 7ffd7000  ParentCid: 03f4
    DirBase: 0b0d00a0  ObjectTable: e1a5f9e8  HandleCount: 491.
    Image: winlogon.exe
```

没错，都是 `7ffdxxxx`。

我们知道，在 2000 系统中 `PEB` 的地址是固定的 `7FFDF000h`。可是在 `XP SP2` 的系统后这个值就发生了变化，但是其地址的高 4 位却没有变化。具体可以看看这些代码。

```c
PVOID HighestVadAddress;
LARGE_INTEGER CurrentTime;
HighestVadAddress = (PVOID) ((PCHAR)MM_HIGHEST_VAD_ADDRESS + 1);
KeQueryTickCount (&CurrentTime);
CurrentTime.LowPart &= ((X64K >> PAGE_SHIFT) - 1);
if (CurrentTime.LowPart <= 1) {
  CurrentTime.LowPart = 2;
  }
HighestVadAddress = (PVOID) ((PCHAR)HighestVadAddress - (CurrentTime.LowPart<<PAGE_SHIFT));
```

可以看出，`PEB` 地址的设置跟时间是有关系的。在 `WRK` 中可以发现这样的定义：

```c
#define X64K (ULONG)65536
#define MM_HIGHEST_VAD_ADDRESS ((PVOID)((ULONG_PTR)MM_HIGHEST_USER_ADDRESS - (64 * 1024)))
```

用 `WinDbg` 可以观察到 `MM_HIGHEST_USER_ADDRESS`：

```
lkd> dd MmHighestUserAddress
80558e3c  7ffeffff 00000000 00000001 00000000
```

那么 `MM_HIGHEST_VAD_ADDRESS` 就是 `7ffdffff` 了，那么 `HighestVadAddress` 就是 `7ffe0000`。而我们知道 `PAGE_SHIFT` 其实就是 `12`，那么 `((X64K >> PAGE_SHIFT) - 1)` 就是 `0xF`。`PEB` 变化的情况就只剩下 16 种了，接下来的一个条件判断让 0 到 `F` 十六种情况一下子剩下 14 种了。最后用 `CurrentTime.LowPart<<PAGE_SHIFT` 的范围就是在 `0x2000` 到 `0xf000` 中，这样 `HighestVadAddress` 的地址就在 `7ffd1000` 到 `7ffde000` 之间的 14 种情况了。

## 2. 从哪里开始搜索，到哪里结束呢？

用 `WinDbg` 的 `!process` 命令观察发现所有进程的 `EProcess` 结构地址，可以发现都在 `0x80000000` 到 `"system"` 进程的 `EPROCESS` 之间。那么我们的搜索范围就确定了下来。至于 `Window` 怎么分配这些空间的我还没有研究，等有时间再说吧。

## 3. 如果空间很大，如何加速搜索？

## 4. 如何防止访问无效内存空间造成 BSOD？

这个问题网上的前辈们也提供了很好的方法。那就是验证 `PTE` 和 `PDE` 的有效性。这里就要说到地址的转换问题了。同时这样还可以有效避免 `BSOD`。

虚拟地址->虚拟地址对应的 `PDE` 地址 `PDE_Address=(VirtualAddress>>22)*4+0xC0300000`
虚拟地址->虚拟地址对应的 `PTE` 地址 `PTE_Address=(VirtualAddress>>12)*4+0xC0000000`

```c
typedef struct _HARDWARE_PTE_X86
{
     ULONG Valid:1;
     ULONG Write:1;
     ULONG Owner:1;
     ULONG WriteThrough:1;
     ULONG CacheDisable:1;
     ULONG Accessed:1;
     ULONG Dirty:1;
     ULONG LargePage:1;
     ULONG Global:1;
     ULONG CopyOnWrite:1;
     ULONG Prototype: 1;
     ULONG reserved: 1;
     ULONG PageFrameNumber:20;
} HARDWARE_PTE_X86, *PHARDWARE_PTE_X86;
```

看看 `PTE` 的结构可知我们主要要判断 `Valid`，`LargePage`。
然后判断其有效性，达到后来加速的效果。

根据前辈的代码，我自己重写了一下判断函数。

```c
NTSTATUS NTAPI MmIsAddressInPageValid( IN PULONG Address ,OUT PULONG pType )
{
#ifndef PAGE_VALID
#define PDE_VALID 0x1
#define PTE_VALID 0x2
#define PDE_INVALID 0x3
#define PTE_INVALID 0x4
#define ADDR_INVALID 0x0
#endif
  ULONG ulAddr;
  ULONG ulPTE;
  ULONG ulPDE;
  ulAddr = (ULONG)Address;
  ulPDE = 0xc0300000 + ( ulAddr >> 22 ) * 4;
  //KdPrint(( "ulPDE%x" ,ulPDE ));
  if ( !MmIsAddressValid( (PULONG)ulPDE ) )
  {
    if ( pType != NULL )
    {
      *pType = ADDR_INVALID;
    }
    return STATUS_UNSUCCESSFUL;
  }
  if ( ( (*(PULONG)ulPDE) & 0x1 ) != 0 )
  {
    if ( ( (*(PULONG)ulPDE) & 0x80 ) != 0 )
    {
      if ( pType != NULL )
      {
        *pType = PDE_VALID;
      }
      return STATUS_SUCCESS;
    }
    ulPTE = 0xc0000000 + ( ulAddr >> 12 ) * 4;
    //KdPrint(( "ulPTE:%x" ,ulPTE ));
    if ( !MmIsAddressValid( (PULONG)ulPTE ) )
    {
      if ( pType != NULL )
      {
        *pType = ADDR_INVALID;
      }
      return STATUS_UNSUCCESSFUL;
    }
    if ( ( (*(PULONG)ulPTE) & 0x1 ) != 0 )
    {
      if ( pType != NULL )
      {
        *pType = PTE_VALID;
      }
      return STATUS_SUCCESS;
    }
    else
    {
      if ( pType != NULL )
      {
        *pType = PTE_INVALID;
      }
      return STATUS_UNSUCCESSFUL;
    }
  }
  if ( pType != NULL )
  {
    *pType = PDE_INVALID;
  }
  return STATUS_UNSUCCESSFUL;
}
```

最后要做的事情就是写几个函数，得到自己要的变量。网上这样的代码很多，我只是重写了一下，呵呵。

得到 `Process Type` 的函数。

```c
ULONG NTAPI GetProcessType()
{
  ULONG ulHeader;
  ULONG Type;
  ulHeader = (ULONG)PsGetCurrentProcess();
  ulHeader = OBJECT_TO_OBJECT_HEADER( ulHeader );
  Type = *(PULONG)(ulHeader + OBJECT_TYPE_OFFSET);
  return Type;
}
```

判断是否是一个有效的 `EPROCESS` 的地址。

```c
NTSTATUS NTAPI MmIsProcessAddr( IN PULONG Address )
{
  ULONG ulProcessType;
  ULONG ulAddr;
  ULONG ObjectTypeAddr;
  ULONG ObjectType;
  ulAddr = (ULONG)Address;
  if ( !NT_SUCCESS( MmIsAddressInPageValid( Address ,NULL ) ) )
  {
    KdPrint(( "Address is invalid" ));
    return STATUS_UNSUCCESSFUL;
  }
  ObjectTypeAddr = OBJECT_TO_OBJECT_HEADER( ulAddr ) + OBJECT_TYPE_OFFSET;
  if ( !NT_SUCCESS( MmIsAddressInPageValid( (PULONG)ObjectTypeAddr ,NULL ) ) )
  {
    KdPrint(( "ObjectTypeAddr is invalid" ));
    return STATUS_UNSUCCESSFUL;
  }
  ObjectType = *(PULONG)ObjectTypeAddr;
  if ( ObjectType == GetProcessType() )
  {
    return STATUS_SUCCESS;
  }
  return STATUS_UNSUCCESSFUL;
}
```

显示进程的函数。

```c
void NTAPI ShowProcessInformation( IN PULONG Address )
{
  PLARGE_INTEGER pExitTime;
  ULONG PID;
  PUCHAR pFileName;
  pExitTime = (PLARGE_INTEGER)( (ULONG)Address + EXIT_TIME_OFFSET );
  if ( pExitTime->QuadPart != 0 )
  {
    return;
  }
  PID = *(PULONG)( (ULONG)Address + PROCESS_ID_OFFSET );
  pFileName = (PUCHAR)( (ULONG)Address + FILE_NAME_OFFSET );
  KdPrint(( "ID:%d\t\tName:%s\n" ,PID ,pFileName ));
}
```

得到 `PEB` 地址的高 4 位，然后这里不能用 `System` 的 `PEB`。因为 `System` 的 `PEB` 总是 `0`。

```
PROCESS 867bc2c0  SessionId: none  Cid: 0004    Peb: 00000000  ParentCid: 0000
    DirBase: 01038000  ObjectTable: e1000b80  HandleCount: 506.
    Image: System
```

```c
ULONG NTAPI GetPebAddressType()
{
  ULONG PebAddr;
  ULONG ulProcessAddr;
  PULONG pSystem;
  pSystem = (PULONG)PsGetCurrentProcess();
  ulProcessAddr = (ULONG)(((PLIST_ENTRY)( (ULONG)pSystem + PROCESS_LINK_OFFSET ))->Flink)
          - PROCESS_LINK_OFFSET;
  PebAddr = *(PULONG)( ulProcessAddr + PEB_OFFSET );
  return ( PebAddr & 0xffff0000 );
}
```

最后的这个就是搜索函数。

```c
void NTAPI DetectProcessBySearchMemory()
{
  PULONG pSystem;
  ULONG ulPoint;
  ULONG ulPebAddr;
  ULONG ulPageSign;
  pSystem = (PULONG)PsGetCurrentProcess();
  for ( ulPoint = 0x80000000 ;ulPoint < (ULONG)pSystem ;ulPoint += 4 )
  {
    if ( NT_SUCCESS( MmIsAddressInPageValid( (PLONG)ulPoint ,&ulPageSign ) ) )
    {
      ulPebAddr = *(PULONG)ulPoint;
      if ( ( ulPebAddr & 0xffff0000 ) == GetPebAddressType() )
      {
        if ( NT_SUCCESS( MmIsProcessAddr( (PULONG)(ulPoint - PEB_OFFSET) ) ) )
        {
          ShowProcessInformation( (PULONG)( ulPoint - PEB_OFFSET ) );
        }
      }
    }
    else if ( ulPageSign == PTE_INVALID )
    {
      if ( ulPoint != 0x80000000 )
      {
        ulPoint -= 4;
      }
      ulPoint += 0x1000;
    }
    else if ( ulPageSign == PDE_INVALID )
    {



```
if ( ulPoint != 0x80000000 )  

  {  

    ulPoint -= 4;  

  }  

  ulPoint += 0x400000;  

}  

else  

{  

  KdPrint(( "Error!" ));  

}  

}
```

![](img/c9a6638d1ca6160d1085807441a0b595.jpg)  

参考文档：  

加密与解密（第三版）  

JIURL 玩玩 Win2k 内存篇 分页机制  

ring0 检测隐藏进程  

WRK1.2

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || EnumProcessByDetectMemory.rar  

 |
