
# SSDT HOOK bypass 某游戏 and Themida ThreadHideFromDebugger

> 原文：[`www.pediy.com/kssd/pediy10/78552.html`](https://www.pediy.com/kssd/pediy10/78552.html)

![](img/3e9367e5826886294bccbb08463f3e34.jpg "酷")

大牛绕路！XX 同学勿怒！

应朋友的求助，调试了一下 `XXGAME`，发现载入后，断点下不了，`OD` 挂上就是个摆设？

立马想到了 `ntdll!ZwSetInformationThread` | `ThreadHideFromDebugger`

载入，下 `ZwSetInformationThread` 断点，果然。

`XXGAME` 的保护是用 `ntdll!ZwSetInformationThread` 传入 `ThreadHideFromDebugger` 来进行反调试的，貌似和以前看到牛人分析的 `bypass themida anti` 差不多哦，有料可参。

那么这个 XX 保护也可以这样搞？

试手一下，发现果然。

代码：

```c
///////////////////////////////////////////////////////////////////////////////
///
/// Copyright (c) 2008 - <dts>
///
/// Original filename: ZwHook.c
/// Project          : ZwHook
/// Date of creation : 2008-11-19
/// Author(s)        : 梧桐
///
/// Purpose          : <description>
///
/// Revisions:
///  0000 [2008-11-19] Initial revision.
///
///////////////////////////////////////////////////////////////////////////////

#include "ntddk.h"

#pragma pack(1)
typedef struct ServiceDescriptorEntry
{
        unsigned int *ServiceTableBase;
        unsigned int *ServiceCounterTableBase; //Used only in checked build
        unsigned int NumberOfServices;
        unsigned char *ParamTableBase;
} ServiceDescriptorTableEntry_t, *PServiceDescriptorTableEntry_t;

#pragma pack()

__declspec(dllimport)  ServiceDescriptorTableEntry_t KeServiceDescriptorTable;

#define SYSTEMSERVICE(_function)  KeServiceDescriptorTable.ServiceTableBase[ *(PULONG)((PUCHAR)_function+1)]

PMDL  g_pmdlSystemCall;
PVOID *MappedSystemCallTable;

#define SYSCALL_INDEX(_Function) *(PULONG)((PUCHAR)_Function+1)

#define HOOK_SYSCALL(_Function, _Hook)  \
       (PVOID) InterlockedExchange( (PLONG) &MappedSystemCallTable[SYSCALL_INDEX(_Function)], (LONG) _Hook)

#define UNHOOK_SYSCALL(_Function, _Hook)  \
       InterlockedExchange( (PLONG) &MappedSystemCallTable[SYSCALL_INDEX(_Function)], (LONG) _Hook)

NTSYSAPI
NTSTATUS
NTAPI  ZwSetInformationThread(
                IN HANDLE  ThreadHandle,
                IN THREADINFOCLASS  ThreadInformationClass,
                IN PVOID  ThreadInformation,
                IN ULONG  ThreadInformationLength
                );

typedef NTSTATUS (*ZWSETINFORMATIONTHREAD)(
  IN HANDLE  ThreadHandle,
  IN THREADINFOCLASS  ThreadInformationClass,
  IN PVOID  ThreadInformation,
  IN ULONG  ThreadInformationLength
  );

ZWSETINFORMATIONTHREAD        OldZwSetInformationThread;

NTSTATUS NewZwSetInformationThread(
                   IN HANDLE  ThreadHandle,
                   IN THREADINFOCLASS  ThreadInformationClass,
                   IN PVOID  ThreadInformation,
                   IN ULONG  ThreadInformationLength)
{ 
  NTSTATUS ntStatus;

  if(ThreadInformationClass == 17) //ANTI-DEBUG
    ntStatus =  STATUS_SUCCESS;
  else
    ntStatus = ((ZWSETINFORMATIONTHREAD)(OldZwSetInformationThread))(
    ThreadHandle,
    ThreadInformationClass,
    ThreadInformation,
    ThreadInformationLength);
  return ntStatus;
}

VOID OnUnload(IN PDRIVER_OBJECT DriverObject)
{
   DbgPrint("ROOTKIT: OnUnload called\n");

   // unhook system calls
   UNHOOK_SYSCALL( ZwSetInformationThread, OldZwSetInformationThread);

   // Unlock and Free MDL
   if(g_pmdlSystemCall)
   {
     MmUnmapLockedPages(MappedSystemCallTable, g_pmdlSystemCall);
     IoFreeMdl(g_pmdlSystemCall);
   }
}


NTSTATUS DriverEntry(IN PDRIVER_OBJECT theDriverObject, 
           IN PUNICODE_STRING theRegistryPath)
{
   // Register a dispatch function for Unload
   theDriverObject->DriverUnload  = OnUnload; 

   // save old system call locations
   OldZwSetInformationThread =(ZWSETINFORMATIONTHREAD)(SYSTEMSERVICE(ZwSetInformationThread));

   // Map the memory into our domain so we can change the permissions on the MDL
   g_pmdlSystemCall = MmCreateMdl(NULL, KeServiceDescriptorTable.ServiceTableBase, KeServiceDescriptorTable.NumberOfServices*4);
   if(!g_pmdlSystemCall)
      return STATUS_UNSUCCESSFUL;

   MmBuildMdlForNonPagedPool(g_pmdlSystemCall);

   // Change the flags of the MDL
   g_pmdlSystemCall->MdlFlags = g_pmdlSystemCall->MdlFlags | MDL_MAPPED_TO_SYSTEM_VA;

   MappedSystemCallTable = MmMapLockedPages(g_pmdlSystemCall, KernelMode);

   // hook system calls
   HOOK_SYSCALL( ZwSetInformationThread, NewZwSetInformationThread);
                                 
   return STATUS_SUCCESS;
}
```
