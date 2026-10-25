
# 一段蛮古老的杀线程代码

> 原文：[`www.pediy.com/kssd/pediy10/59091.html`](https://www.pediy.com/kssd/pediy10/59091.html)

科普，呵呵。

引用: `/*`

`TerminateThread.c`

`By 炉子[0GiNr]`

`http://hi.baidu.com/breakinglove_`

`http://0ginr.com`

`*/`

```c
#include "ntddk.h"

#include "LDasm.h" //网上很多的，自己找一个好了。

typedef enum _KAPC_ENVIRONMENT {
    OriginalApcEnvironment,
    AttachedApcEnvironment,
    CurrentApcEnvironment,
    InsertApcEnvironment
} KAPC_ENVIRONMENT;

NTKERNELAPI
VOID
KeInitializeApc (
                 PKAPC Apc,
                 PETHREAD Thread,
                 KAPC_ENVIRONMENT Environment,
                 PKKERNEL_ROUTINE KernelRoutine,
                 PKRUNDOWN_ROUTINE RundownRoutine,
                 PKNORMAL_ROUTINE NormalRoutine,
                 KPROCESSOR_MODE ProcessorMode,
                 PVOID NormalContext
                 );

NTKERNELAPI
BOOLEAN
KeInsertQueueApc (
                  PKAPC Apc,
                  PVOID SystemArgument1,
                  PVOID SystemArgument2,
                  KPRIORITY Increment
                  );   

#define PS_CROSS_THREAD_FLAGS_SYSTEM 0x00000010UL

ULONG GetThreadFlagsOffset()
{
    UCHAR *cPtr, *pOpcode;
    ULONG Length;
    USHORT Offset;
    for (cPtr = (PUCHAR)PsTerminateSystemThread;  
        cPtr < (PUCHAR)PsTerminateSystemThread + 0x100;  
        cPtr += Length)  
    {
        Length = SizeOfCode(cPtr, &pOpcode);  
        if (!Length) break;     
        if (*(USHORT *)pOpcode == 0x80F6) //f6804802000010 test byte ptr [eax+248h],10h
        {
            Offset=*(USHORT *)((ULONG)pOpcode+2);  
            return Offset;
            //break;
        }
    }

    return 0;
}

VOID KernelTerminateThreadRoutine(
                                IN PKAPC Apc,
                                IN OUT PKNORMAL_ROUTINE *NormalRoutine,
                                IN OUT PVOID *NormalContext,
                                IN OUT PVOID *SystemArgument1,
                                IN OUT PVOID *SystemArgument2
                                )
{
    ULONG ThreadFlagsOffset=GetThreadFlagsOffset();
    PULONG ThreadFlags;
    DbgPrint("[TerminateThread] KernelTerminateThreadRoutine.\n");  
    ExFreePool(Apc);
    if (ThreadFlagsOffset)
    {
        ThreadFlags=(ULONG *)((ULONG)(PsGetCurrentThread())+ThreadFlagsOffset);  
        *ThreadFlags=(*ThreadFlags)|PS_CROSS_THREAD_FLAGS_SYSTEM;  
        PsTerminateSystemThread(STATUS_SUCCESS); //o(∩_∩)o
    }
    else
    {
        //failed :'(
    }

    return; //never be here
}

BOOLEAN TerminateThread(PETHREAD Thread)
{
    PKAPC Apc=NULL;
    BOOLEAN blnSucceed=FALSE;
    if (!MmIsAddressValid(Thread)) return FALSE; //error.
    Apc=ExAllocatePool(NonPagedPool,sizeof(KAPC));
    KeInitializeApc(Apc,
        Thread,
        OriginalApcEnvironment,
        KernelTerminateThreadRoutine,
        NULL,
        NULL,
        KernelMode,
        NULL); //special apc
    blnSucceed=KeInsertQueueApc(Apc,
        NULL,
        NULL,
        0);
    //add some code works like KeForceResumeThread here.
    return blnSucceed;
}

VOID DriverUnload(PDRIVER_OBJECT pDriverObj)
{     
    DbgPrint("[TerminateThread] Unloaded\n");
}

NTSTATUS DriverEntry(PDRIVER_OBJECT pDriverObj, PUNICODE_STRING pRegistryString)
{
    DbgPrint("[TerminateThread] DriverEntry.\n");
    TerminateThread((PETHREAD)0xff6f3c70); // for test
    pDriverObj->DriverUnload = DriverUnload;
    return STATUS_SUCCESS; //do NOT return an unsuccessful value here, or you need to wait for apc routine return.
}
```
