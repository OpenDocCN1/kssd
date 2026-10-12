
# 内核层获取 SSDT 中函数原始地址

> 原文：[`www.pediy.com/kssd/pediy12/139524.html`](https://www.pediy.com/kssd/pediy12/139524.html)

昨天在看雪看到[《【下载】发个获得 SSDT 函数名和索引号的代码》](http://bbs.pediy.com/showthread.php?t=125699)，一时兴起整理了一份内核层根据 `ntdll.dll` 和 `nt` 模块文件来获取 SSDT 函数原始的地址的代码，可以用来检测和恢复 SSDT hook。

没什么技术含量，不过这却是我第一次规规矩矩地写代码，花费了一下午的时间调整代码格式，填写注释。希望能够尽可能地使代码易读易用易维护，当然，现在依旧感觉有很多不如意的地方，包括安全性与函数的封装，都有很大的修改空间，不过还是选择了先贴出来，如果感觉不爽的，可以自己修改一下。`![](img/f3773294bf3a914c9a8c0c242528a003.jpg "使眼色")`

因为感觉中文混在代码中看起来很别扭，所以注释都使用的英文，这对一个六级没有通过的人来说，真不是件容易的事情(`![](img/d23858dd221cce728a3a61b46badb044.jpg "恐怖!")`)，如果发现什么语法错误，请无视`~~`

最后，欢迎各位对代码的格式，注释的填写以及函数的封装等工程性的问题上提出建议`~~` `![](img/c1d8523a7842d7b67917c210ac6dcf18.jpg "微笑")`

代码:

```
/\*++

Module Name:

  Entry.c

Abstract:

  A sample with ListSSDT.c .

\--\*/

#include "ListSSDT.h"

//
//  function declaration
//

DRIVER_UNLOAD UnloadMe;
DRIVER_INITIALIZE DriverEntry;

#ifdef ALLOC_PRAGMA

#pragma alloc_text(INIT, DriverEntry)
#pragma alloc_text(PAGE, UnloadMe)

#endif

NTSTATUS 
DriverEntry(
  IN PDRIVER_OBJECT DriverObject,
  IN PUNICODE_STRING RegistryPath)
/\*++

Routine Description:

  Entry of driver.

Arguments:

  DriverObject - Pointer to driver object created by system.
  RegistryPath - Pointer to the Unicode name of the registry path
    for this driver.

Return Value:

  NT Status code

\--\*/
{
  NTSTATUS Status;
  ULONG NameLength;
  ULONG AddrLength;
  PVOID NameBuffer;
  PVOID AddrBuffer;
  PVOID AddrOrig;
  ULONG i;
  
  //
  //
  //Important Variables:
  //
  //  NameBuffer - Buffer to receive the function names of SSDT.
  //  AddrBuffer - Buffer to receive the current function address of SSDT.
  //  AddrOrig - Buffer to receive the original function address of SSDT.
  //
  //
  
  UNREFERENCED_PARAMETER( RegistryPath );

  //
  //  driver initialization
  //

  DriverObject->DriverUnload = UnloadMe;


  //
  //  get the size of all information.
  //

  Status = GetNeedLength( &NameLength, &AddrLength);

  if(!NT_SUCCESS(Status))
  {
    return Status;
  }

  //
  //  allocate memory
  //

  NameBuffer = ExAllocatePoolWithTag(NonPagedPool, NameLength, 'name');
  AddrBuffer = ExAllocatePoolWithTag(NonPagedPool, AddrLength, 'addr');
  AddrOrig = ExAllocatePoolWithTag(NonPagedPool, AddrLength, 'orig');

  if(NameBuffer == NULL)
  {
    return STATUS_UNSUCCESSFUL;
  }

  if(AddrBuffer == NULL)
  {
    ExFreePool(NameBuffer);
    return STATUS_UNSUCCESSFUL;
  }
  
  if(AddrOrig == NULL)
  {
    ExFreePool(NameBuffer);
    ExFreePool(AddrBuffer);
    return STATUS_UNSUCCESSFUL;
  }

  //
  //  get information
  //

  Status = GetSSDTInfo(NameBuffer, AddrBuffer, AddrOrig);


  //
  //  print result
  //

  if(NT_SUCCESS(Status))
  {
    for( i = 0; i < AddrLength/4; i++)
    {
      DbgPrint("%3d\t\t%s\t\t\t0x%08x\t0x%08x\n", 
         i, 
         (ULONG)NameBuffer + i * 0x32, 
         *(PULONG)((ULONG)AddrOrig + i * 0x04),
         *(PULONG)((ULONG)AddrBuffer + i * 0x04));
    }
  }


  //
  //  free memory
  //

  if(NameBuffer != NULL)
  {
    ExFreePool(NameBuffer);
    NameBuffer = NULL;
  }

  if(AddrBuffer != NULL)
  {
    ExFreePool(AddrBuffer);
    AddrBuffer = NULL;
  }
  if(AddrOrig != NULL)
  {
    ExFreePool(AddrOrig);
    AddrBuffer = NULL;
  }

  return Status;
}

VOID 
UnloadMe(
  IN PDRIVER_OBJECT DriverObject)
{
  UNREFERENCED_PARAMETER( DriverObject ); 

  //
  //  nothing to do
  //
}

```

添加一张截图:  

`![](img/5badc39fd00a4f91acd8a4077dc233c1.jpg)`  

呃，最后说一句，这个程序只适合 `WinXp`，`Win7` 下不成功，没有调试，可能是由于用于定位的特征字节不对造成的`~~~`

上传的附件

| `![](img/620251b33676a8e4b98ca0ece915ee8d.jpg)` || `ListSSDT.zip`  

 |
