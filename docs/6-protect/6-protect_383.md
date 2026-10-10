

# DebugPort 清零代码

> 原文：[`www.pediy.com/kssd/pediy12/132095.html`](https://www.pediy.com/kssd/pediy12/132095.html)

这几天翻看论坛里的精华里，谈到了 DebugPort 清零。但似乎都是说怎么 anti，没讲怎么实现。于是便动手写了一个![](img/906f120cfe711ccbc8463fd6eb86eee9.jpg "尴尬")不知道游戏保护中是不是也这么个思路。。在 XP SP3 下测试过。有不好的地方请大家指正。。

代码:

```
#include <ntddk.h>

PETHREAD pThreadObj = NULL;
BOOLEAN bTerminated = FALSE;
UCHAR szProcessName\[16\] = "TestCrackMe.exe";

VOID DriverUnload(PDRIVER\_OBJECT pDriverObject);
VOID AntiDbgThread(PVOID pContext);

NTSTATUS DriverEntry(PDRIVER\_OBJECT pDriverObject,PUNICODE\_STRING pRegistryPath)
{
  OBJECT\_ATTRIBUTES ObjAddr = {0};
  HANDLE ThreadHandle = 0;
  NTSTATUS NtStatus = STATUS\_SUCCESS;

  KdPrint(("Driver Entry"));

  pDriverObject->DriverUnload = DriverUnload;

  InitializeObjectAttributes(&ObjAddr,NULL,OBJ\_KERNEL\_HANDLE,0,NULL);

  NtStatus = PsCreateSystemThread(&ThreadHandle,THREAD\_ALL\_ACCESS,&ObjAddr,NULL,NULL,AntiDbgThread,NULL);

  if(NT\_SUCCESS(NtStatus))
  {
    KdPrint(("Thread Created"));

    NtStatus = ObReferenceObjectByHandle(ThreadHandle,THREAD\_ALL\_ACCESS,\*PsThreadType,KernelMode,&pThreadObj,NULL);

    ZwClose(ThreadHandle);

    if(!NT\_SUCCESS(NtStatus))
    {
      bTerminated = TRUE;
    }
  }

  return NtStatus;
}

VOID DriverUnload(PDRIVER\_OBJECT pDriverObject)
{
  bTerminated = TRUE;
  KeWaitForSingleObject(pThreadObj,Executive,KernelMode,FALSE,NULL);

  ObDereferenceObject(pThreadObj);
}

VOID AntiDbgThread(PVOID pContext)
{
  PEPROCESS pCurrentProcess = NULL;
  PEPROCESS pFirstProcess = NULL;
  LARGE\_INTEGER inteval;

  inteval.QuadPart = -20000000;

  KeSetPriorityThread(KeGetCurrentThread(),LOW\_REALTIME\_PRIORITY);

  while(1)
  {
    if(bTerminated)
    {
      break;
    }
    
    pCurrentProcess = IoGetCurrentProcess();
    pFirstProcess = pCurrentProcess;

    while(RtlCompareMemory(szProcessName,(PUCHAR)((ULONG)pCurrentProcess + 0x174),16) != 16)
    {
      pCurrentProcess = (PEPROCESS)(\*(PULONG)((ULONG)pCurrentProcess + 0x88) - 0x88);

      if(pCurrentProcess == pFirstProcess)
      {
        goto END;
      }
    }
    
    \*(PULONG)((ULONG)pCurrentProcess + 0xbc) = 0;
END:
    KeDelayExecutionThread(KernelMode,FALSE,&inteval);
  }
}

```

上传的附件
| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || TestCrackMe.rar  
 |
| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || antidbg.rar  
 |