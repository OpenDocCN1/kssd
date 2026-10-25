
# ring0 搜索内存枚举线程

> 原文：[`www.pediy.com/kssd/pediy10/97050.html`](https://www.pediy.com/kssd/pediy10/97050.html)

鸟文一篇,大大飘过...

一个多月前的文章,今天没事做,完善一下了.还有,今年吃的月饼难吃啊...

原理通过搜索枚举 `peb`,然后判定是否是合法的 `ethread` 来枚举线程.

参考文章 1:堕落天才(ring0 枚举进程,大部分的思路都是从这来的):`http://bbs.pediy.com/showthread.php?t=44243`

2:分页方式 combojiang(分页机制):`http://bbs.pediy.com/showthread.php?t=61327`

多多感激他们的文章了.

```c
#include <ntddk.h>

// 分页方式

#define NONPAE      0x00000001

#define PAE         0x00000002

#define NOTFOUND    0x00000000

#define OBJECT_HEADER_SIZE        0x018

#define OBJECT_HEADER_TYPE_OFFSET    0x008

#define ETHREAD_CID_OFFSET        0x1ec

#define ETHREAD_STARTADDRESS_OFFSET    0x224

#define ETHRAED_KTHREAD_OFFSET      0x000

#define KTHREAD_TEB_OFFSET        0x020

#define ETHREAD_EXITTIME_OFFSET      0x1c8

PETHREAD  pEThread;

PEPROCESS  pSystem;

ULONG    ThreadType;

ULONG    uThreadCount;

ULONG    ThreadMaxAddr;

// 确定分页模式

ULONG HowPaging()

{
  ULONG  i;
  ULONG  end;
  UNICODE_STRING  ustr;
  RtlInitUnicodeString(&ustr, L"MmIsAddressValid");    // 从 MmIsAddressValid 中查询分页方式
  i = (ULONG)MmGetSystemRoutineAddress(&ustr);

  for(end = i + 40; i < end; i++)  // 只搜索前 40 个字节
  {
    if(*(PCHAR)i == 0x2d)
    {
      if(*(PULONG)(i + 1) == 0x3fd00000)
      {
        // sub eax, 3fd00000h
        return NONPAE;
      }
      else if(*(PULONG)(i + 1) == 0x3fa00000)
      {
        // sub eax, 3fa00000h
        return PAE;
      }
    }
  }

  return NOTFOUND;

}

// va 需要调整多少

ULONG AddHowMuch(IN ULONG va, IN ULONG paging)

{
  ULONG  pte;
  ULONG  pde;
  if(paging == NONPAE)
  {
    pde = (((va >> 22) << 2 ) & 0xffc) + 0xc0300000;
    pte = (((va >> 12) << 2 ) & 0x3FFFFC) + 0xc0000000;
  }
  else
  {
    pde = (((va >> 21) << 3) & 0x3FF8) + 0xC0600000;
    pte = (((va >> 12) << 3) & 0x7FFFF8) + 0xC0000000;
  }

  if((* (PULONG)pde & 0x1) != 0)
  {
    if((* (PULONG)pde & 0x80) != 0)
    {
      return 0;
    }
    if((* (PULONG)pte & 0x01) != 0)
    {
      return 0;
    }
    else
    {
      return (0x1000 - 4);
    }
  }
  else
  {
    if(paging == NONPAE)
    {
      return (0x400000 - 4);
    }
    else
    {
      return (0x200000 - 4);
    }
  }

}

VOID Unload(IN PDRIVER_OBJECT  DriverObject)

{
  KdPrint(("Driver Unload!\n"));

}

ULONG GetThreadType()

{
  KdPrint(("GetThreadType!\n"));
  // 获得线程对象类型的值
  ThreadType = *(PULONG)((ULONG)pEThread - OBJECT_HEADER_SIZE + OBJECT_HEADER_TYPE_OFFSET);
  return ThreadType;

}

BOOLEAN IsThreadType(IN ULONG i)

{
  ULONG  objecttype;
  if(!MmIsAddressValid((PVOID)i) ||
    !MmIsAddressValid((PVOID)(i + 1)) ||
    !MmIsAddressValid((PVOID)(i + 2)) ||
    !MmIsAddressValid((PVOID)(i + 3)) ||
    )    // 多谢 Fypher 的提示.另外这里怕 MmIsAddressValid 被钩的话就自己实现吧
  {
    return FALSE;
  }

  objecttype = i - KTHREAD_TEB_OFFSET - ETHRAED_KTHREAD_OFFSET - OBJECT_HEADER_SIZE +OBJECT_HEADER_TYPE_OFFSET;
  if(MmIsAddressValid((PVOID)objecttype) ||
    MmIsAddressValid((PVOID)(objecttype + 1) ||
    MmIsAddressValid((PVOID)(objecttype + 2) ||
    MmIsAddressValid((PVOID)(objecttype + 3))
  {
    if(*(PULONG)objecttype == ThreadType)
    {
      return TRUE;
    }
  }
  return FALSE;

}

VOID ShowThread(IN ULONG i, IN ULONG pid)

{
  CLIENT_ID  cid;
  ULONG    teb;
  ULONG    address;
  // startaddress, cid, teb
  address = *(PULONG)(i + ETHREAD_STARTADDRESS_OFFSET);
  cid.UniqueProcess  = *(PHANDLE)((ULONG)i + ETHREAD_CID_OFFSET);
  cid.UniqueThread  = *(PHANDLE)((ULONG)i+ ETHREAD_CID_OFFSET + 4);
  teb = *(PULONG)(i + ETHRAED_KTHREAD_OFFSET + KTHREAD_TEB_OFFSET);
  if((ULONG)cid.UniqueProcess != pid)    // 判断是否是制定进程的线程
  {
    return ;
  }

  KdPrint(("address of ethread:0x%x\n", i));
  KdPrint(("thread addr:0x%x, tid:%5d, pid:0x%x, teb:0x%x ,_eprocess addr: 0x%x\n", address, (ULONG)cid.UniqueThread, (ULONG)cid.UniqueProcess, teb, i));
  uThreadCount++;

}

VOID EnumThread(IN ULONG pid, IN ULONG pagetype)

{
  ULONG  teb;
  ULONG  i, j;
  for(i = 0x80000000; i < (ULONG)ThreadMaxAddr; i += 4)
  {
    if(0 != (j = AddHowMuch(i, pagetype)))
    {
      // 地址是无效的,需要调整
      i += j;
      continue;
    }
    teb = *(PULONG)i;
    if((teb & 0xfff00fff) == 0x7ff00000 || teb == 0)    // teb 的值总是 0 或者前 12 位为 0x7ff
    {
      if(IsThreadType(i))
      {
        ShowThread(i - KTHREAD_TEB_OFFSET - ETHRAED_KTHREAD_OFFSET, pid);
      }
    }
  }

}

NTSTATUS DriverEntry(IN PDRIVER_OBJECT  DriverObject,
           IN PUNICODE_STRING  RegistryPath
           )

{
  ULONG  pagetype;
  pagetype = HowPaging();
  if(pagetype == NOTFOUND)
  {
    KdPrint(("HowPaging() failed!\n"));
    return STATUS_UNSUCCESSFUL;
  }

  pSystem = PsGetCurrentProcess();
  ThreadMaxAddr = (ULONG)pSystem + 0x1000;    // 这里的不太准确.
  EnumThread(4, pagetype);    // 列举 system 进程的 thread
  return STATUS_SUCCESS;

}
```

`DriverEntry` 中有个明显的缺点:

`ThreadMaxAddr = (ULONG)pSystem + 0x1000;`

我家里那台老古董(256 内存,赛扬 4 的 u),xp sp2(140 多个补丁没打,放弃了)从 `pspcidtable` 来看,`ethread` 的变化相当的大,有的竟然是 `0xff` 开头的.进程也可能是 `0xff` 开头的.

可能是系统版本问题.我笔记本 xp sp3 的(没带)却可以"比较"正常.

如果非要准确的列举,就只能全部搜 `2GB` 了,或者找出系统是怎么分配空间的.

代码编译通过,我没有试过,256 的老爷机装 `vmware` 不可能了.直接试的话,万一蓝屏有个闪失,光驱也没的,我不想悲剧.有个思想就可以了.
