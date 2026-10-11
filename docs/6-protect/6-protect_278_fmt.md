
# 围观一下 HS

> 原文：`www.pediy.com/kssd/pediy12/142823.html`

```c
//---------------------------------------发第二次了，再发不出去就不发了.

/* img/906f120cfe711ccbc8463fd6eb86eee9.jpg 尴尬 */

/* 只贴部分代码，大多都是小技巧没什么特别的地方。 */

//------------------xxxx.h

#define PAGEDCODE code_seg("PAGED")

#define LOCKEDCODE code_seg()

#define INITCODE code_seg("INIT")

#define PAGEDDATA data_seg("PAGED")

#define LOCKEDDATA data_seg()

#define INITDATA data_seg("INIT")

typedef struct _DEVICE_CONTEXTSION
{
  UNICODE_STRING driver_name;
  UNICODE_STRING symbol_name;
  PDEVICE_OBJECT device_object;
};

VOID DriverUnLoad(PDRIVER_OBJECT driver);

NTSTATUS DispatchIrp(PDEVICE_OBJECT driver,PIRP irp);

//----------------------------------------------

VOID LoadDriver_GetVersion(PDRIVER_OBJECT driver);

VOID Get_NtResumeThread();

VOID Hook_PsSuspendThread();

VOID My_PsSuspendThread();

VOID Un_PsSuspendThread();

VOID Enum_Thread(BOOL b_thread=FALSE);   //TRUE 表示暂停线程,FALSE 表示结束线程

BOOL Enum_Module(ULONG Thread_StartAddr);

//--守护 OD 和 CE--------------------------

VOID Hook_ObOpenObjectByPointer();

VOID Un_ObOpenObjectByPointer();

VOID SSDT_NtQuerySystemInformation();

VOID Un_NtQuerySystemInformation();

//------------------------------------

VOID Hook_NtOpenProcess();

VOID My_NtOpenProecss();

VOID Un_NtOpenProecss();

VOID Hook_NtReadVirtualMemory();

VOID My_NtReadVirtualMemory();

VOID Un_NtReadVirtualMemory();

VOID Hook_NtWriteVirtualMemory();

VOID My_NtWriteVirtualMemory();

VOID Un_NtWriteVirtualMemory();

VOID Hook_NtClose();

VOID My_NtClose();

VOID Un_NtClocse();

VOID Hook_NtDeviceIoControlFile();

VOID My_NtDeviceIoControlFile();

VOID Unnt_NtDeviceIoControlFile();

VOID Hook_KiAttachProcess();

VOID My_KiAttachProcess();

VOID Unnt_KiAttachProecss();

VOID Hook_IopXxxControlFile();

VOID My_IopXxxControlFile();

VOID Un_IopXxxControlFile();

VOID Hook_NtGetContextThread();

VOID My_NtGetContextThread();

VOID Un_NtGetContextThread();

VOID Hook_NtSetContextThread();

VOID My_NtSetContextThread();

VOID Un_NtSetContextThread();

VOID Hook_NtGetContextThread1();

VOID My_NtGetContextThread1();

VOID Un_NtGetContextThread1();

VOID Hook_NtQueryPerformanceCounter();

VOID My_NtQueryPerformanceCounter_1();

VOID My_NtQueryPerformanceCounter_2();

VOID Un_NtQueryPerformanceCounter();

VOID Hook_DebugPort();

VOID Un_DebugPort();

VOID Hook_NtSetInformationThread();

extern "C"

NTSTATUS __stdcall My_NtSetInformationThread  ( __in HANDLE  ThreadHandle,

                                               __in THREADINFOCLASS  ThreadInformationClass,

                                               __in_bcount(ThreadInformationLength) PVOID  ThreadInformation,

                                               __in ULONG  ThreadInformationLength

                                             ) ;

VOID Un_NtSetInnformationThread();

#endif
```

```cpp
//--------------------------------xxx.cpp

#pragma INITCODE

extern "C"

NTSTATUS DriverEntry(PDRIVER_OBJECT driver,PUNICODE_STRING driver_object_name)
{
    /*
    1.绕过 PsSuspendThread
    2.枚举 HS 线程暂停
    3.SSDT HOOK 守护进程
        1.NtOpenProcess
        2.NtReadVirtualMemory
    4.绕 SSDT 函数 HOOK
        1.NtOpenProcess
        2.NtReadVirtualMemory
        3.NtWriteVirtualMemory
        4.NtClose
        5.NtDeviceIoControlFile
        6.KiAttachProcess
        7.IopXxxControlFile
        8.NtGetContextThread
        9.NtSetContextThread
        10.NtGetContextThread 第二处
        11.NtQueryPerformanceCounter
    5.绕 Shadow SSDT 函数 HOOK
        1.HOOK KiFastCallEntry 绕 SSDT HOOK
        2.绕 NtGdiGetPixel
        3.Hook Shadow SSDT 守护窗口
    6.(_EPROECSS+bc)DebugPort 的处理
    7.HOOK NtSetInformationThread (HideFromDebugPort)  0x11 号功能
    8.SSDT HOOK 隐藏进程和模块
    */
  //------------------------------------------------------------
  LoadDriver_GetVersion(driver);
  Get_NtResumeThread();
  Hook_PsSuspendThread();

  Enum_Thread(TRUE);
  Hook_NtOpenProcess();
  Hook_NtReadVirtualMemory();
  Hook_NtWriteVirtualMemory();
  Hook_NtClose();
  Hook_NtDeviceIoControlFile();
  Hook_KiAttachProcess();
  Hook_IopXxxControlFile();
  Hook_NtGetContextThread();
  Hook_NtSetContextThread();
  Hook_NtGetContextThread1();
  Hook_NtQueryPerformanceCounter();
  Hook_DebugPort();
  Hook_NtSetInformationThread();
//  Hook_ObOpenObjectByPointer();
  SSDT_NtQuerySystemInformation();
  //-------------------------------------------------------------
  NTSTATUS status=STATUS_SUCCESS;
  driver->MajorFunction[IRP_MJ_CREATE]=DispatchIrp;
  driver->MajorFunction[IRP_MJ_READ]=DispatchIrp;
  driver->MajorFunction[IRP_MJ_WRITE]=DispatchIrp;
  driver->MajorFunction[IRP_MJ_CLOSE]=DispatchIrp;
  driver->MajorFunction[IRP_MJ_DEVICE_CONTROL]=DispatchIrp;
  driver->DriverUnload=DriverUnLoad;
  UNICODE_STRING driver_name;
  RtlInitUnicodeString(&driver_name,_Driver_name);
  PDEVICE_OBJECT device_object;
  status=IoCreateDevice(driver,NULL,&driver_name,FILE_DEVICE_UNKNOWN,NULL,FALSE,&device_object);
  if(!NT_SUCCESS(status))
  {
    KdPrint(("创建设备失败!\n"));
    return status;
  }
  KdPrint(("创建设备成功!\n"));
  device_object->Flags |= DO_BUFFERED_IO;
  UNICODE_STRING symbolic_name;
  RtlInitUnicodeString(&symbolic_name,_Symbolic_name);
  status=IoCreateSymbolicLink(&symbolic_name,&driver_name);
  if(!NT_SUCCESS(status))
  {
    IoDeleteDevice(device_object);
    KdPrint(("创建符号链接失败!\n"));
    return status;
  }
  KdPrint(("创建符号链接成功!\n"));
  return STATUS_SUCCESS;
}

#pragma PAGEDCODE

VOID DriverUnLoad(PDRIVER_OBJECT driver)
{
  UNICODE_STRING symbol_name;
  RtlInitUnicodeString(&symbol_name,_Symbolic_name);
  IoDeleteSymbolicLink(&symbol_name);
  IoDeleteDevice(driver->DeviceObject);
  KdPrint(("删除设备成功!\n"));
  //---------------------------------
  Un_PsSuspendThread();
  Un_NtOpenProecss();
  Un_NtReadVirtualMemory();
  Un_NtWriteVirtualMemory();
  Un_NtClocse();
  Unnt_NtDeviceIoControlFile();
  Unnt_KiAttachProecss();
  Un_IopXxxControlFile();
  Un_NtGetContextThread();
  Un_NtSetContextThread();
  Un_NtGetContextThread1();
  Un_NtQueryPerformanceCounter();
  Un_DebugPort();
  Un_NtSetInnformationThread();
//  Un_ObOpenObjectByPointer();
  Un_NtQuerySystemInformation();
  Enum_Thread(FALSE);
}

#pragma PAGEDCODE

NTSTATUS DispatchIrp(PDEVICE_OBJECT driver,PIRP irp)
{
  irp->IoStatus.Status=STATUS_SUCCESS;
  irp->IoStatus.Information=0;
  IoCompleteRequest(irp,IO_NO_INCREMENT);
  KdPrint(("进入 IRP 例程!\n"));
  return STATUS_SUCCESS;
}

//--------------------------------xxxx_struct.h

struct _SYSTEM_PROCESSOR_TIMES//处理器时间结构
{
  LARGE_INTEGER IdleTime;
  LARGE_INTEGER KernelTime;
  LARGE_INTEGER UserTime;;
  LARGE_INTEGER DpcTime;
  LARGE_INTEGER InterruptTime;
  ULONG InterruptCount;
};

typedef struct _SYSTEM_THREADS{
    LARGE_INTEGER KernelTime;
    LARGE_INTEGER UserTime;
    LARGE_INTEGER CreateTime;
    ULONG WaitTime;
    PVOID StartAddress;
    CLIENT_ID ClientId;
    KPRIORITY Priority;
    LONG BasePriority;
    ULONG ContextSwitches;
    ULONG ThreadState;
    ULONG WaitReason;
    ULONG Reversed;//add by achillis
} SYSTEM_THREAD_INFORMATION,*PSYSTEM_THREADS;

typedef struct _SYSTEM_PROCESSES{
  ULONG NextEntryDelta;
  ULONG ThreadCount;
  LARGE_INTEGER Reserved1[3];
  LARGE_INTEGER CreateTime;
  LARGE_INTEGER UserTime;
  LARGE_INTEGER KernelTime;
  UNICODE_STRING ProcessName;
  KPRIORITY BasePriority;
  ULONG ProcessId;
```



```
  ULONG InheritedFromProcessId;  

  ULONG HandleCount;  

  ULONG SessionId;  

  ULONG_PTR PageDirectoryBase;  

  VM_COUNTERS VmCounters;  

  ULONG PrivatePageCount;// add by achillis  

  IO_COUNTERS IoCounters;  

  _SYSTEM_THREADS Threads[1];  

} SYSTEM_PROCESSES, *PSYSTEM_PROCESSES;  

typedef struct _SYSTEM_MODULE_INFORMATION_ENTRY  

{  

  ULONG Unknow1;  

  ULONG Unknow2;  

#ifdef _WIN64  

  ULONG Unknow3;  

  ULONG Unknow4:  

#endif  

  PVOID Base;  

  ULONG Size;  

  ULONG Flags;  

  USHORT Index;  

  USHORT NameLength;  

  USHORT LoadCount;  

  USHORT ModuleNameOffset;  

  char ImageName[256];  

}SYSTEM_MODULE_INFORMATION_ENTRY, *PSYSTEM_MODULE_INFORMATION_ENTRY;  

typedef struct _SYSTEM_MODULE_INFORMATION  

{  

  ULONG Count;//内核中以加载的模块的个数  

  SYSTEM_MODULE_INFORMATION_ENTRY Module[1];  

}SYSTEM_MODULE_INFORMATION, *PSYSTEM_MODULE_INFORMATION;  

typedef enum _SYSTEM_INFORMATION_CLASS {  

    SystemBasicInformation, // 0 Y N  

    SystemProcessorInformation, // 1 Y N  

    SystemPerformanceInformation, // 2 Y N  

    SystemTimeOfDayInformation, // 3 Y N  

    SystemNotImplemented1, // 4 Y N  

    SystemProcessesAndThreadsInformation, // 5 Y N  

    SystemCallCounts, // 6 Y N  

    SystemConfigurationInformation, // 7 Y N  

    SystemProcessorTimes, // 8 Y N  

    SystemGlobalFlag, // 9 Y Y  

    SystemNotImplemented2, // 10 Y N  

    SystemModuleInformation, // 11 Y N  

    SystemLockInformation, // 12 Y N  

    SystemNotImplemented3, // 13 Y N  

    SystemNotImplemented4, // 14 Y N  

    SystemNotImplemented5, // 15 Y N  

    SystemHandleInformation, // 16 Y N  

    SystemObjectInformation, // 17 Y N  

    SystemPagefileInformation, // 18 Y N  

    SystemInstructionEmulationCounts, // 19 Y N  

    SystemInvalidInfoClass1, // 20  

    SystemCacheInformation, // 21 Y Y  

    SystemPoolTagInformation, // 22 Y N  

    SystemProcessorStatistics, // 23 Y N  

    SystemDpcInformation, // 24 Y Y  

    SystemNotImplemented6, // 25 Y N  

    SystemLoadImage, // 26 N Y  

    SystemUnloadImage, // 27 N Y  

    SystemTimeAdjustment, // 28 Y Y  

    SystemNotImplemented7, // 29 Y N  

    SystemNotImplemented8, // 30 Y N  

    SystemNotImplemented9, // 31 Y N  

    SystemCrashDumpInformation, // 32 Y N  

    SystemExceptionInformation, // 33 Y N  

    SystemCrashDumpStateInformation, // 34 Y Y/N  

    SystemKernelDebuggerInformation, // 35 Y N  

    SystemContextSwitchInformation, // 36 Y N  

    SystemRegistryQuotaInformation, // 37 Y Y  

    SystemLoadAndCallImage, // 38 N Y  

    SystemPrioritySeparation, // 39 N Y  

    SystemNotImplemented10, // 40 Y N  

    SystemNotImplemented11, // 41 Y N  

    SystemInvalidInfoClass2, // 42  

    SystemInvalidInfoClass3, // 43  

    SystemTimeZoneInformation, // 44 Y N  

    SystemLookasideInformation, // 45 Y N  

    SystemSetTimeSlipEvent, // 46 N Y  

    SystemCreateSession, // 47 N Y  

    SystemDeleteSession, // 48 N Y  

    SystemInvalidInfoClass4, // 49  

    SystemRangeStartInformation, // 50 Y N  

    SystemVerifierInformation, // 51 Y Y  

    SystemAddVerifier, // 52 N Y  

    SystemSessionProcessesInformation // 53 Y N  

}SYSTEM_INFORMATION_CLASS, *PSYSTEM_INFORMATION_CLASS;  

extern "C"  

NTSYSAPI NTSTATUS NTAPI ZwQuerySystemInformation  (__in SYSTEM_INFORMATION_CLASS  SystemInformationClass,   

                              __out_bcount_opt(SystemInformationLength) PVOID  SystemInformation,   

                              __in ULONG  SystemInformationLength,   

                              __out_opt PULONG  ReturnLength    

                              );  

extern "C"  

NTKERNELAPI NTSTATUS PsLookupThreadByThreadId  (__in HANDLE  ThreadId,   

                            __deref_out PETHREAD*  Thread    

                            );  

extern "C"  

typedef  

NTSTATUS __stdcall PsSuspendThread ( IN PETHREAD  Thread,   

              OUT PULONG PreviousSuspendCount  OPTIONAL  

               ) ;  

extern "C"  

typedef  

NTSTATUS __stdcall PsResumeThread  ( IN PETHREAD  Thread,   

              OUT PULONG PreviousSuspendCount  OPTIONAL    

              ) ;  

extern "C"  

typedef  

NTSYSCALLAPI NTSTATUS (__stdcall* NtSetInformationThread)  ( __in HANDLE  ThreadHandle,   

                              __in THREADINFOCLASS  ThreadInformationClass,   

                              __in_bcount(ThreadInformationLength) PVOID  ThreadInformation,   

                              __in ULONG  ThreadInformationLength    

                              ) ;  

extern "C"  

typedef  

NTKERNELAPI NTSTATUS __stdcall ObOpenObjectByPointer  ( __in PVOID  Object,   

                      __in ULONG  HandleAttributes,   

                      __in_opt PACCESS_STATE  PassedAccessState,   

                      __in ACCESS_MASK  DesiredAccess,   

                      __in_opt POBJECT_TYPE  ObjectType,   

                      __in KPROCESSOR_MODE  AccessMode,   

                      __out PHANDLE  Handle    

                      ) ;  

extern "C"  

typedef  

NTSYSCALLAPI NTSTATUS NTAPI NtQuerySystemInformation  ( __in SYSTEM_INFORMATION_CLASS  SystemInformationClass,   

                              __out_bcount_opt(SystemInformationLength) PVOID  SystemInformation,   

                              __in ULONG  SystemInformationLength,   

                              __out_opt PULONG  ReturnLength    

                              );  

#endif  

//------------------------------------------xxxx_ssdt.h  

#define _windows_xp  51  

#define _Game_Name "OD.exe"  

#define _Game_Name1 "OllyDBG.EXE"  

#define _Game_Name2 "OllyICE.exe"  

#define _Game_Name3 "Cheat Engine.exe"  

#define _Game_Name4 "Cheat Engine.ex"  

#define _Game_Name5 "Cheat Engine"  

#define _UGame_Name  L"OD.exe"  

#define _UGame_Name1 L"OllyDBG.EXE"  

#define _UGame_Name2 L"OllyICE.exe"  

#define _UGame_Name3 L"Cheat Engine.exe"  

#define _UGame_Name4 L"Cheat Engine.ex"  

#define _UGame_Name5 L"Cheat Engine"  

PsSuspendThread* _PsSuspendThread;  

PsResumeThread* _PsResumeThread;  

NtSetInformationThread* _NtSetInformationThread;  

//-------------------------------------------------  

int in_NtResumeThread=0;      //保存 NtResumeThread 的序号  

int in_NtPsSuspendThread=0;   //保存 NtSuspendThread 的序号  

int in_NtOpenProcess=0;       //保存 NtOpenProcess 的序号  

int in_NtReadVirtualMemory=0; //保存 NtReadVirtualMemory 的序号  

int in_NtWriteVirtualMemory=0;//保存 NtWriteVirtualMemory 的序号  

int in_NtClose=0;             //保存 NtClose 的序号  

int in_NtDeviceIoControlFile=0;//保存 NtDeviceIoControlFile 的序号  

int in_NtFsControlFile=0;     //保存 NtFsControlFile 的序号  

int in_NtGetContextThread=0;  //保存 NtGetContextThread 的序号  

int in_NtTerminateThread=0;  //保存 NtTerminateThread 的序号  

int in_NtSetContextThread=0;  //保存 NtSetContextThread 的序号  

int in_NtQueryPerformanceCounter=0; //保存 NtQueryPerformanceCounter 的序号  

int in_NtSetInformationThread=0; //保存 NtSetInformationThread 的序号  

int in_NtQuerySystemInformation=0; //保存 NtQuerySystemInformation 的序号  

//-------------------------------------------------  

ULONG addr_ObWatchHandles;    //保存 ObWatchHandles 的地址  

ULONG addr_SEH_prolog;        //保存_SEH_prolog 的地址  

//-------------------------------------------------  

ULONG addr_NtOpenThread;          //保存 NtOpenThread 的地址  

ULONG u_PsSuspendThread;          //保存 PsSuspendTHread 的地址  

ULONG push_addr_PsSuspendThread;   

ULONG call_addr_PsSuspendThread;   

ULONG addr_NtOpenProcess;        //保存 NtOpenProcess 的地址  

ULONG push_addr_NtOpenProcess;  

ULONG call_addr_NtOpenProecss;  

ULONG addr_NtReadVirtualMemory;  //保存 NtReadVirtualMemory 的地址  

ULONG push_addr_NtReadVirtualMemory;  

ULONG call_addr_NtReadVirtualMemory;  
```



```c
ULONG addr_NtWriteVirtualMemory; //保存 NtWriteVirtualMemory 的地址
ULONG push_addr_NtWriteVirtualMemory;
ULONG call_addr_NtWriteVirtualMemory;

ULONG addr_NtClose;             //保存 NtClose 的地址
ULONG call_addr_NtClose;

ULONG addr_NtDeviceIoControlFile; //保存 NtDeviceIoControlFile 的地址
ULONG call_addr_NtDeviceIoControlFile;

ULONG addr_KiAttachProcess;     //保存 KiAttachProecss 的地址
ULONG call_addr_KiAttachProcess;

ULONG addr_IopXxxControlFile;   //保存 IopXxxControlFile 的地址
ULONG call_addr_IopXxxControlFile;

ULONG addr_NtGetContextThread;  //保存 NtGetContextThread 的地址
ULONG call_addr_NtGetContextThread;

ULONG addr_NtSetContextThread;  //保存 NtSetContextThread 的地址
ULONG call_addr_NtSetContextThread;

ULONG addr_NtGetContextThread1;  //保存 NtGetContextThread 的第二处 HOOK
ULONG call_addr_NtGetContextThread1;

ULONG Game_call_addr_1;         //获取 NtGetContextThread 的第一个 call 地址
ULONG Game_call_addr_2;         //获取 NtGetContextThread 的第二个 call 地址

ULONG addr_NtQueryPerformanceCounter; //保存 NtQueryPerformanceCounter 的第一个跳转地址
ULONG addr1_NtQueryPerformanceCounter;//保存 NtQueryPerformanceCounter 的第二个跳转地址
ULONG call_addr_NtQueryPerformanceCounter;

ULONG addr_NtSetInformationThread;    //保存 NtSetInformationThread 的地址
ULONG addr1_NtSetInformationThread;

//-------------------------------------------------
typedef struct _ServiceDescriptorTable {
  PVOID ServiceTableBase; //System Service Dispatch Table 的基地址
  PVOID ServiceCounterTable;
  //包含着 SSDT 中每个服务被调用次数的计数器。这个计数器一般由 sysenter 更新。
  unsigned int NumberOfServices;//由 ServiceTableBase 描述的服务的数目。
  PVOID ParamTableBase; //包含每个系统服务参数字节数表的基地址-系统服务参数表
}*PServiceDescriptorTable;

extern "C" PServiceDescriptorTable KeServiceDescriptorTable;

#pragma pack(1)
typedef struct _JMPCODE
{
  BYTE E9;
  ULONG Fun_addr;
  BYTE nop[2];
}jmp_code,*pjmp_code;
#pragma pack()

//-----------获取 ssdt 地址
#pragma PAGEDCODE
ULONG Getssdt_addr(int index)
{
  ULONG* xfun_addr,x_serviceDescriptor,x_addr;
  x_serviceDescriptor=(ULONG)KeServiceDescriptorTable->ServiceTableBase;
  KdPrint(("当前 ServiceDescriptorTable 的地址为:%x\n",x_serviceDescriptor));
  xfun_addr=(PULONG)(x_serviceDescriptor+index*4);
  x_addr=*xfun_addr;
  KdPrint(("当前 Index=%d 函数地址为:%x\n",index,x_addr));
  return x_addr;
}

//-----------------获取 ssdt 函数列表
#pragma PAGEDCODE
ULONG* Getsset_this(int index)
{
  ULONG* xfun_addr,x_servicedescriptor;
  x_servicedescriptor=(ULONG )KeServiceDescriptorTable->ServiceTableBase;
  KdPrint(("当前 ServiceDescriptorTable 的地址为:%x\n",x_servicedescriptor));
  xfun_addr=(PULONG)(x_servicedescriptor+index*4);
  return xfun_addr;
}

KIRQL kirql;

#pragma PAGEDCODE
VOID PAGED_Open()
{
  __asm
  {
    cli
    push eax
    mov eax,cr0
    and eax,not 10000h
    mov cr0,eax
    pop eax
  }
  kirql=KeRaiseIrqlToDpcLevel();
}

#pragma PAGEDCODE
VOID PAGED_Exit()
{
  KeLowerIrql(kirql);
  __asm
  {
    push eax
    mov eax,cr0
    or eax,10000h
    mov cr0,eax
    pop eax
    sti
  }
}

#pragma PAGEDCODE
VOID LoadDriver_GetVersion(PDRIVER_OBJECT driver)
{
  ULONG Majorversion,Minorversion,buildVersion;
  PsGetVersion(&Majorversion,&Minorversion,&buildVersion,NULL);
  DWORD dw_version=Majorversion*10+Minorversion;
  switch(dw_version)
  {
  case _windows_xp:
    KdPrint(("当前操作系统 windows xp sp3!\n"));
    in_NtPsSuspendThread=0xFE;             //函数序号
    in_NtResumeThread=0xCE;
    in_NtOpenProcess=0x7A;
    in_NtReadVirtualMemory=0xBA;
    in_NtWriteVirtualMemory=0x115;
    in_NtClose=0x19;
    in_NtDeviceIoControlFile=0x42;
    in_NtFsControlFile=0x54;
    in_NtGetContextThread=0x55;
    in_NtTerminateThread=0x102;
    in_NtSetContextThread=0xD5;
    in_NtQueryPerformanceCounter=0xA5;
    in_NtSetInformationThread=0xE5;
    in_NtQuerySystemInformation=0xAD;
    break;
  default:
    driver->DriverUnload=DriverUnLoad;
    break;
  }
}

#pragma PAGEDCODE
VOID Get_NtResumeThread()
{
  ULONG addr_NtResumeThread;
  BYTE* _bp;
  _bp=(BYTE*)Getssdt_addr(in_NtResumeThread);
  do
  {
    if(((*(_bp-4)==0x50)&&(*(_bp-3)==0xFF)&&(*(_bp))==0xE8)&&(*(_bp+5)==0x8B)&&(*(_bp+8)==0xE8))
    {
      break;
    }
    _bp++;
  } while (1);
  addr_NtResumeThread=(ULONG)_bp;
  __asm
  {
    push eax
    push ebx
    mov eax,addr_NtResumeThread
    add eax,1
    mov ebx,[eax]
    add eax,ebx
    add eax,5
    sub eax,1
    mov addr_NtResumeThread,eax
    pop ebx
    pop eax
  }
  _PsResumeThread=(PsResumeThread*)addr_NtResumeThread;
  KdPrint(("PsResumeThread 的地址为:%x\n",addr_NtResumeThread));
}

jmp_code _jmp_code_PsSuspendThread;
pjmp_code _pjmp_code_PsSuspendThread;

#pragma INITCODE
VOID Hook_PsSuspendThread()
{
  ULONG u_NtSuspendThread;
  BYTE* _bp;    //特征码搜索
  _bp=(BYTE*)Getssdt_addr(in_NtPsSuspendThread);
  do
  {
    if(((*(_bp-9)==0x7c)&&(*(_bp-4)==0x50)&&(*(_bp-3)==0xFF)&&(*(_bp))==0xE8)&&(*(_bp+5)==0x8B)&&(*(_bp+7)==0x8B))
    {
      break;
    }
    _bp++;
  }while(1);
  u_NtSuspendThread=(ULONG)_bp;
  __asm
  {
    push eax
    mov eax,u_NtSuspendThread
    add eax,1
    mov ebx,[eax]
    add eax,ebx
    add eax,5
    sub eax,1
    mov u_NtSuspendThread,eax
    pop eax
  }
  _PsSuspendThread=(PsSuspendThread*)u_NtSuspendThread;
  KdPrint(("PsSuspendThread 的地址为:%x\n",u_NtSuspendThread));
  u_PsSuspendThread=u_NtSuspendThread+2;
  KdPrint(("向 PsSuspendThread 写跳转的地址为:%x\n",u_PsSuspendThread));
  ULONG u_addr_PsSuspendThread;
  UNICODE_STRING push_addr;
  RtlInitUnicodeString(&push_addr,L"NtOpenProcess");
  push_addr_PsSuspendThread=(ULONG)MmGetSystemRoutineAddress(&push_addr);
  __asm
  {
    push eax
    mov eax,push_addr_PsSuspendThread
    add eax,6
    mov eax,[eax]
    add eax,0x28
    sub eax,0x284
    mov u_addr_PsSuspendThread,eax
    pop eax
  }
  addr_ObWatchHandles=u_addr_PsSuspendThread;
  KdPrint(("ObWatchHandles 的地址为:%x\n",u_addr_PsSuspendThread));
  push_addr_PsSuspendThread=u_addr_PsSuspendThread+0x784;
  KdPrint(("PsSuspendThread 的 push 地址为:%x\n",push_addr_PsSuspendThread));
  ULONG u_call_PsSuspendThread;
  UNICODE_STRING call_addr;
  RtlInitUnicodeString(&call_addr,L"NtOpenThread");
  call_addr_PsSuspendThread=(ULONG)MmGetSystemRoutineAddress(&call_addr);
  addr_NtOpenThread=call_addr_PsSuspendThread;
  __asm
  {
    push eax
    push ebx
    mov eax,call_addr_PsSuspendThread
    add eax,0xA
    add eax,1
    mov ebx,[eax]
    add eax,ebx
    add eax,5
    sub eax,1
    mov u_call_PsSuspendThread,eax
    pop ebx
    pop eax
  }
  addr_SEH_prolog=u_call_PsSuspendThread;
  call_addr_PsSuspendThread=u_call_PsSuspendThread;
  KdPrint(("PsSuspendThread 的 call 地址为:%x\n",call_addr_PsSuspendThread));
  _pjmp_code_PsSuspendThread=(pjmp_code)u_PsSuspendThread;
  _jmp_code_PsSuspendThread.E9=_pjmp_code_PsSuspendThread->E9;
  _jmp_code_PsSuspendThread.Fun_addr=_pjmp_code_PsSuspendThread->Fun_addr;
  PAGED_Open();
  _pjmp_code_PsSuspendThread->E9=0xE9;
  _pjmp_code_PsSuspendThread->Fun_addr=(ULONG)((ULONG)My_PsSuspendThread-u_PsSuspendThread-5);
  PAGED_Exit();
}

#pragma PAGEDCODE
VOID __declspec(naked) My_PsSuspendThread()
{
  __asm
  {
    push push_addr_PsSuspendThread
```



```assembly
  mov eax,u_PsSuspendThread  
  add eax,0xA  
  push eax  
  mov eax,call_addr_PsSuspendThread  
  jmp eax  
}
}
```

```cpp
#pragma PAGEDCODE  
VOID Un_PsSuspendThread()  
{  
  PAGED_Open();  
  _pjmp_code_PsSuspendThread->E9=_jmp_code_PsSuspendThread.E9;  
  _pjmp_code_PsSuspendThread->Fun_addr=_jmp_code_PsSuspendThread.Fun_addr;  
  PAGED_Exit();  
}  

#pragma PAGEDCODE  

BOOL Enum_Module(ULONG Thread_StartAddr)  
{  
  NTSTATUS status;  
  PCHAR ModuleNames;  
  ULONG len;  
  PVOID pbuffer=NULL;  
  PSYSTEM_MODULE_INFORMATION Module_Enum;  
  status=ZwQuerySystemInformation((SYSTEM_INFORMATION_CLASS)11,NULL,0,&len);  
  pbuffer=ExAllocatePool(NonPagedPool,len);  
  if(pbuffer==NULL)  
  {  
    KdPrint(("分配内存失败!\r\n"));  
    return FALSE;  
  }  
  status=ZwQuerySystemInformation((SYSTEM_INFORMATION_CLASS)11,pbuffer,len,NULL);  
  Module_Enum=(PSYSTEM_MODULE_INFORMATION)pbuffer;  
  for(unsigned int i=0;i<Module_Enum->Count;i++)  
  {  
    if(ModuleNames=strstr(Module_Enum->Module[i].ImageName,"EagleXNt.sys"))  
    {  
      ULONG Module_Min=(ULONG)Module_Enum->Module[i].Base;  
      ULONG Module_Max=Module_Min+Module_Enum->Module[i].Size;  
      if((Module_Min<=Thread_StartAddr)&&(Thread_StartAddr<=Module_Max))  
      {  
        return TRUE;  
      }  
    }  
  }  
  ExFreePool(pbuffer);  
  return FALSE;  
}  

#pragma PAGEDCODE  

VOID Enum_Thread(BOOL b_thread)  
{  
  NTSTATUS status;  
  PVOID pbuffer=NULL;  
  ULONG len;  
  PSYSTEM_PROCESSES Process_Enum;  
  PSYSTEM_THREADS Thread_Enum;  
  status=ZwQuerySystemInformation((SYSTEM_INFORMATION_CLASS)5,NULL,0,&len);  
  KdPrint(("枚举进程时分配的内存大小:%d\n",len));  
  pbuffer=ExAllocatePool(NonPagedPool,len);  
  if(pbuffer==NULL)  
  {  
    KdPrint(("分配内存失败!\r\n"));  
    return ;  
  }  
  status=ZwQuerySystemInformation((SYSTEM_INFORMATION_CLASS)5,pbuffer,len,NULL);  
  Process_Enum=(PSYSTEM_PROCESSES)pbuffer;  
  do  
  {  
    if(Process_Enum->ThreadCount&&Process_Enum->ProcessId)  
    {  
      Thread_Enum=(PSYSTEM_THREADS)Process_Enum->Threads;  
      for(unsigned int i=0;i<Process_Enum->ThreadCount;i++)  
      {  
        BOOL bl_modlule=FALSE;  
        bl_modlule=Enum_Module((ULONG)Thread_Enum->StartAddress);  
        if(bl_modlule==TRUE)  
        {  
          PETHREAD thread_temp;  
          PsLookupThreadByThreadId(Thread_Enum->ClientId.UniqueThread,&thread_temp);  
          if(b_thread==TRUE)  
          {  
            _PsSuspendThread(thread_temp,NULL);  
            KdPrint(("暂停<->线程信息:Thread[%d]  StartAddr=%08x   TID:%d \r\n",i+1,Thread_Enum->StartAddress,Thread_Enum->ClientId.UniqueThread));  
          }  
          if(b_thread==FALSE)  
          {  
            _PsResumeThread(thread_temp,NULL);  
            KdPrint(("删除<->线程信息:Thread[%d]  StartAddr=%08x   TID:%d \r\n",i+1,Thread_Enum->StartAddress,Thread_Enum->ClientId.UniqueThread));  
          }  
        }  
        Thread_Enum++;  
      }  
    }  
    if(Process_Enum->NextEntryDelta==NULL)  
    {  
      break;  
    }  
    Process_Enum=(PSYSTEM_PROCESSES)(((PUCHAR)Process_Enum)+Process_Enum->NextEntryDelta);  
  }while(1);  
  ExFreePool(pbuffer);  
}  

jmp_code _jmp_code_NtOpenProecss;  
pjmp_code _pjmp_code_NtOpenProecss;  

#pragma INITCODE  
VOID Hook_NtOpenProcess()  
{  
  ULONG my_function_addr;  
  addr_NtOpenProcess=Getssdt_addr(in_NtOpenProcess);  
  addr_NtOpenProcess+=5;  
  KdPrint(("向 NtOpenProcess 写跳转地址为:%x\n",addr_NtOpenProcess));  
  __asm  
  {  
    push eax  
    mov eax,My_NtOpenProecss  
    mov my_function_addr,eax  
    pop eax  
  }  
  KdPrint(("自定义 NtOpenProcess 的地址为:%x\n",my_function_addr));  
  push_addr_NtOpenProcess=addr_ObWatchHandles+0x25c;  
  KdPrint(("NtOpenProcess 的 push 地址为:%x\n",push_addr_NtOpenProcess));  
  call_addr_NtOpenProecss=addr_SEH_prolog;  
  KdPrint(("NtOpenProcess 的 call 地址为:%x\n",call_addr_NtOpenProecss));  
  _pjmp_code_NtOpenProecss=(pjmp_code)addr_NtOpenProcess;  
  _jmp_code_NtOpenProecss.E9=_pjmp_code_NtOpenProecss->E9;  
  _jmp_code_NtOpenProecss.Fun_addr=_pjmp_code_NtOpenProecss->Fun_addr;  
  PAGED_Open();  
  _pjmp_code_NtOpenProecss->E9=0xE9;  
  _pjmp_code_NtOpenProecss->Fun_addr=(ULONG)(my_function_addr-addr_NtOpenProcess-5);  
  PAGED_Exit();  
}  

#pragma PAGEDCODE  
VOID __declspec(naked) My_NtOpenProecss()  
{  
  __asm  
  {  
    push push_addr_NtOpenProcess  
    mov eax,addr_NtOpenProcess  
    add eax,0xA  
    push eax  
    mov eax,call_addr_NtOpenProecss  
    jmp eax  
  }  
}  

#pragma PAGEDCODE  
VOID Un_NtOpenProecss()  
{  
  PAGED_Open();  
  _pjmp_code_NtOpenProecss->E9=_jmp_code_NtOpenProecss.E9;  
  _pjmp_code_NtOpenProecss->Fun_addr=_jmp_code_NtOpenProecss.Fun_addr;  
  PAGED_Exit();  
}  

jmp_code _jmp_code_NtReadVirtualMemory;  
pjmp_code _pjmp_code_NtReadVirtualMemory;  

#pragma INITCODE  
VOID Hook_NtReadVirtualMemory()  
{  
  ULONG My_Function_Addr;  
  addr_NtReadVirtualMemory=(ULONG)Getssdt_addr(in_NtReadVirtualMemory);  
  addr_NtReadVirtualMemory+=2;  
  KdPrint(("向 NtReadVirtualMemory 写跳转地址为:%x\n",addr_NtReadVirtualMemory));  
  __asm  
  {  
    push eax  
    mov eax,My_NtReadVirtualMemory  
    mov My_Function_Addr,eax  
    pop eax  
  }  
  KdPrint(("自定义 NtReadVirtualMemory 的地址为:%x\n",My_Function_Addr));  
  __asm  
  {  
    push eax  
    mov eax,addr_NtReadVirtualMemory  
    add eax,1  
    mov eax,[eax]  
    mov push_addr_NtReadVirtualMemory,eax  
    pop eax  
  }  
  KdPrint(("NtReadVirtualMemory 的 push 地址为:%x\n",push_addr_NtReadVirtualMemory));  
  call_addr_NtReadVirtualMemory=addr_SEH_prolog;  
  KdPrint(("NtReadVirtualMemory 的 call 地址为:%x\n",call_addr_NtReadVirtualMemory));  
  _pjmp_code_NtReadVirtualMemory=(pjmp_code)addr_NtReadVirtualMemory;  
  _jmp_code_NtReadVirtualMemory.E9=_pjmp_code_NtReadVirtualMemory->E9;  
  _jmp_code_NtReadVirtualMemory.Fun_addr=_pjmp_code_NtReadVirtualMemory->Fun_addr;  
  PAGED_Open();  
  _pjmp_code_NtReadVirtualMemory->E9=0xE9;  
  _pjmp_code_NtReadVirtualMemory->Fun_addr=(ULONG)(My_Function_Addr-addr_NtReadVirtualMemory-5);  
  PAGED_Exit();  
}  

#pragma PAGEDCODE  
VOID __declspec(naked) My_NtReadVirtualMemory()  
{  
  __asm  
  {  
    push push_addr_NtReadVirtualMemory  
    mov eax,addr_NtReadVirtualMemory  
    add eax,0xA  
    push eax  
    mov eax,call_addr_NtReadVirtualMemory  
    jmp eax  
  }  
}  

#pragma PAGEDCODE  
VOID Un_NtReadVirtualMemory()  
{  
  PAGED_Open();  
  _pjmp_code_NtReadVirtualMemory->E9=_jmp_code_NtReadVirtualMemory.E9;  
  _pjmp_code_NtReadVirtualMemory->Fun_addr=_jmp_code_NtReadVirtualMemory->Fun_addr;  
  PAGED_Exit();  
}  

jmp_code _jmp_code_NtWriteVirtrualMemory;  
pjmp_code _pjmp_code_NtWriteVirtualMemory;  

#pragma INITCODE  
VOID Hook_NtWriteVirtualMemory()  
{  
  ULONG My_Function_Addr;  
  addr_NtWriteVirtualMemory=(ULONG)Getssdt_addr(in_NtWriteVirtualMemory);  
  addr_NtWriteVirtualMemory+=2;  
  KdPrint(("向 NtWriteVirtualMemory 写跳转地址为:%x\n",addr_NtWriteVirtualMemory));  
  __asm  
  {  
    push eax  
    mov eax,My_NtWriteVirtualMemory  
    mov My_Function_Addr,eax  
    pop eax  
  }  
  KdPrint(("自定义 NtWriteVirtualMemory 的地址为:%x\n",My_Function_Addr));  
  __asm  
  {  
    push eax  
    mov eax,addr_NtWriteVirtualMemory  
    add eax,1  
    mov eax,[eax]  
    mov push_addr_NtWriteVirtualMemory,eax  
    pop eax  
  }  
  KdPrint(("NtWriteVirtualMemory 的 push 地址为:%x\n",push_addr_NtWriteVirtualMemory));
```



\```c
call\_addr\_NtWriteVirtualMemory=addr\_SEH\_prolog;

KdPrint(("NtWriteVirtualMemory 的 call 地址为:%x\\n",call\_addr\_NtWriteVirtualMemory));

\_pjmp\_code\_NtWriteVirtualMemory=(pjmp\_code)addr\_NtWriteVirtualMemory;

\_jmp\_code\_NtWriteVirtrualMemory.E9=\_pjmp\_code\_NtWriteVirtualMemory->E9;

\_jmp\_code\_NtWriteVirtrualMemory.Fun\_addr=\_pjmp\_code\_NtWriteVirtualMemory->Fun\_addr;

PAGED\_Open();

\_pjmp\_code\_NtWriteVirtualMemory->E9=0xE9;

\_pjmp\_code\_NtWriteVirtualMemory->Fun\_addr=(ULONG)(My\_Function\_Addr-addr\_NtWriteVirtualMemory-5);

PAGED\_Exit();

\}

#pragma PAGEDCODE

VOID \_\_declspec(naked) My\_NtWriteVirtualMemory()

\{

\_\_asm

\{
push push\_addr\_NtWriteVirtualMemory
mov eax,addr\_NtWriteVirtualMemory
add eax,0xA
push eax
mov eax,call\_addr\_NtWriteVirtualMemory
jmp eax
\}

\}

#pragma PAGEDCODE

VOID Un\_NtWriteVirtualMemory()

\{

PAGED\_Open();

\_pjmp\_code\_NtWriteVirtualMemory->E9=\_jmp\_code\_NtWriteVirtrualMemory.E9;

\_pjmp\_code\_NtWriteVirtualMemory->Fun\_addr=\_jmp\_code\_NtWriteVirtrualMemory.Fun\_addr;

PAGED\_Exit();

\}
\```

\```c
jmp\_code \_jmp\_code\_NtClose;

pjmp\_code \_pjmp\_code\_NtClose;

#pragma INITCODE

VOID Hook\_NtClose()

\{

ULONG My\_Function\_Addr;

addr\_NtClose=(ULONG)Getssdt\_addr(in\_NtClose);

call\_addr\_NtClose=addr\_NtClose-0x1A2;

KdPrint(("NtClose 的 call 地址为:%x\\n",call\_addr\_NtClose));

addr\_NtClose+=0x12;

KdPrint(("向 NtClose 写跳转的地址为:%x\\n",addr\_NtClose));

\_\_asm

\{
push eax
mov eax,My\_NtClose
mov My\_Function\_Addr,eax
pop eax
\}

KdPrint(("自定义 NtClose 的地址为:%x\\n",My\_Function\_Addr));

\_pjmp\_code\_NtClose=(pjmp\_code)addr\_NtClose;

\_jmp\_code\_NtClose.E9=\_pjmp\_code\_NtClose->E9;

\_jmp\_code\_NtClose.Fun\_addr=\_pjmp\_code\_NtClose->Fun\_addr;

\_jmp\_code\_NtClose.nop\[0\]=\_pjmp\_code\_NtClose->nop\[0\];

PAGED\_Open();

\_pjmp\_code\_NtClose->E9=0xE9;

\_pjmp\_code\_NtClose->Fun\_addr=(ULONG)(My\_Function\_Addr-addr\_NtClose-5);

\_pjmp\_code\_NtClose->nop\[0\]=0x90;

PAGED\_Exit();

\}

#pragma PAGEDCODE

VOID \_\_declspec(naked) My\_NtClose()

\{

\_\_asm

\{
push 0
push eax
push dword ptr \[ebp+8\]
mov eax,addr\_NtClose
add eax,0xB
push eax
mov eax,call\_addr\_NtClose
jmp eax
\}

\}

#pragma PAGEDCODE

VOID Un\_NtClocse()

\{

PAGED\_Open();

\_pjmp\_code\_NtClose->E9=\_jmp\_code\_NtClose.E9;

\_pjmp\_code\_NtClose->Fun\_addr=\_jmp\_code\_NtClose.Fun\_addr;

\_pjmp\_code\_NtClose->nop\[0\]=\_jmp\_code\_NtClose.nop\[0\];

PAGED\_Exit();

\}
\```

\```c
jmp\_code \_jmp\_code\_NtDeviceIoControlFile;

pjmp\_code \_pjmp\_code\_NtDeviceIoControlFile;

#pragma INITCODE

VOID Hook\_NtDeviceIoControlFile()

\{

ULONG My\_Function\_Addr;

ULONG u\_NtFsControlFile;

addr\_NtDeviceIoControlFile=(ULONG)Getssdt\_addr(in\_NtDeviceIoControlFile);

addr\_NtDeviceIoControlFile+=0x1F;

KdPrint(("向 NtDeviceIoControlFile 写跳转的地址为:%x\\n",addr\_NtDeviceIoControlFile));

u\_NtFsControlFile=(ULONG)Getssdt\_addr(in\_NtFsControlFile);

call\_addr\_IopXxxControlFile=u\_NtFsControlFile;

u\_NtFsControlFile=u\_NtFsControlFile+0x25;

\_\_asm

\{
push eax
push ebx
mov eax,u\_NtFsControlFile
add eax,1
mov ebx,\[eax\]
add eax,ebx
add eax,5
sub eax,1
mov call\_addr\_NtDeviceIoControlFile,eax
pop ebx
pop eax
\}

addr\_IopXxxControlFile=call\_addr\_NtDeviceIoControlFile;

KdPrint(("NtDeviceIoControlFile 的 call 地址为:%x\\n",call\_addr\_NtDeviceIoControlFile));

\_\_asm

\{
push eax
mov eax,My\_NtDeviceIoControlFile
mov My\_Function\_Addr,eax
pop eax
\}

KdPrint(("自定义 NtDeviceIoControlFile 的地址为:%x\\n",My\_Function\_Addr));

\_pjmp\_code\_NtDeviceIoControlFile=(pjmp\_code)addr\_NtDeviceIoControlFile;

\_jmp\_code\_NtDeviceIoControlFile.E9=\_pjmp\_code\_NtDeviceIoControlFile->E9;

\_jmp\_code\_NtDeviceIoControlFile.Fun\_addr=\_pjmp\_code\_NtDeviceIoControlFile->Fun\_addr;

PAGED\_Open();

\_pjmp\_code\_NtDeviceIoControlFile->E9=0xE9;

\_pjmp\_code\_NtDeviceIoControlFile->Fun\_addr=(ULONG)(My\_Function\_Addr-addr\_NtDeviceIoControlFile-5);

PAGED\_Exit();

\}

#pragma PAGEDCODE

VOID \_\_declspec(naked) My\_NtDeviceIoControlFile()

\{

\_\_asm

\{
push dword ptr \[ebp+0Ch\]
push dword ptr \[ebp+8\]
mov eax,addr\_NtDeviceIoControlFile
add eax,0xB
push eax
mov eax,call\_addr\_NtDeviceIoControlFile
jmp eax
\}

\}

#pragma PAGEDCODE

VOID Unnt\_NtDeviceIoControlFile()

\{

PAGED\_Open();

\_pjmp\_code\_NtDeviceIoControlFile->E9=\_jmp\_code\_NtDeviceIoControlFile.E9;

\_pjmp\_code\_NtDeviceIoControlFile->Fun\_addr=\_jmp\_code\_NtDeviceIoControlFile.Fun\_addr;

PAGED\_Exit();

\}
\```

\```c
jmp\_code \_jmp\_code\_KiAttachProcess;

pjmp\_code \_pjmp\_code\_KiAttachProcess;

#pragma INITCODE

VOID Hook\_KiAttachProcess()

\{

ULONG My\_Function\_Addr;

ULONG u\_KeAttachProcess;

UNICODE\_STRING KeAttachProcess\_name;

RtlInitUnicodeString(&KeAttachProcess\_name,L"KeAttachProcess");

u\_KeAttachProcess=(ULONG)MmGetSystemRoutineAddress(&KeAttachProcess\_name);

KdPrint(("套出 KeAttachProcess 的地址为:%x\\n",u\_KeAttachProcess));

BYTE\* \_bp;

\_bp=(BYTE\*)u\_KeAttachProcess;

do

\{
if((\*(\_bp-2)==0x57)&&(\*(\_bp-1)==0x56)&&(\*(\_bp)==0xE8))

\{
break;
\}

\_bp++;

\}while(1);

addr\_KiAttachProcess=(ULONG)\_bp;

\_\_asm

\{
push eax
push ebx
mov eax,addr\_KiAttachProcess
add eax,1
mov ebx,\[eax\]
add eax,ebx
add eax,5
sub eax,1
mov addr\_KiAttachProcess,eax
pop ebx
pop eax
\}

KdPrint(("KiAttachProcess 的地址为%x\\n",addr\_KiAttachProcess));

call\_addr\_KiAttachProcess=addr\_KiAttachProcess-0x488;

KdPrint(("KiAttachProcess 的 call 地址为:%x\\n",call\_addr\_KiAttachProcess));

addr\_KiAttachProcess+=0x11;

KdPrint(("向 KiAttachProcess 写跳转地址:%x\\n",addr\_KiAttachProcess));

\_\_asm

\{
push eax
mov eax,My\_KiAttachProcess
mov My\_Function\_Addr,eax
pop eax
\}

KdPrint(("自定义 KiAttachProcess 的地址为:%x\\n",My\_Function\_Addr));

\_pjmp\_code\_KiAttachProcess=(pjmp\_code)addr\_KiAttachProcess;

\_jmp\_code\_KiAttachProcess.E9=\_pjmp\_code\_KiAttachProcess->E9;

\_jmp\_code\_KiAttachProcess.Fun\_addr=\_pjmp\_code\_KiAttachProcess->Fun\_addr;

\_jmp\_code\_KiAttachProcess.nop\[0\]=\_pjmp\_code\_KiAttachProcess->nop\[0\];

\_jmp\_code\_KiAttachProcess.nop\[1\]=\_pjmp\_code\_KiAttachProcess->nop\[1\];

PAGED\_Open();

\_pjmp\_code\_KiAttachProcess->E9=0xE9;

\_pjmp\_code\_KiAttachProcess->Fun\_addr=(ULONG)(My\_Function\_Addr-addr\_KiAttachProcess-5);

\_pjmp\_code\_KiAttachProcess->nop\[0\]=0x90;

\_pjmp\_code\_KiAttachProcess->nop\[1\]=0x90;

PAGED\_Exit();

\}

#pragma PAGEDCODE

VOID \_\_declspec(naked) My\_KiAttachProcess()

\{

\_\_asm

\{
inc word ptr \[edi+60h\]
lea ebx,\[esi+34h\]
push ebx
mov eax,addr\_KiAttachProcess
add eax,0xD
push eax
mov eax,call\_addr\_KiAttachProcess
jmp eax
\}

\}

#pragma PAGEDCODE

VOID Unnt\_KiAttachProecss()

\{

PAGED\_Open();

\_pjmp\_code\_KiAttachProcess->E9=\_jmp\_code\_KiAttachProcess.E9;

\_pjmp\_code\_KiAttachProcess->Fun\_addr=\_jmp\_code\_KiAttachProcess.Fun\_addr;

\_pjmp\_code\_KiAttachProcess->nop\[0\]=\_jmp\_code\_KiAttachProcess.nop\[0\];

\_pjmp\_code\_KiAttachProcess->nop\[1\]=\_jmp\_code\_KiAttachProcess.nop\[1\];
\```



## 代码片段 1：Hook IopXxxControlFile

```c
PAGED_Exit();

}

jmp_code _jmp_code_IopXxxControlFile;

pjmp_code _pjmp_code_IopXxxControlFile;

#pragma INITCODE

VOID Hook_IopXxxControlFile()

{
  ULONG My_Function_Addr;

  BYTE* _bp;

  _bp=(BYTE*)addr_IopXxxControlFile;

  do

  {
    if((*( _bp-11)==0x6A)&&(*( _bp-6)==0xFF)&&(*( _bp-3)==0xFF)&&(*( _bp)==0xE8)&&(*( _bp+5)==0x83)&&(*( _bp+9)==0x8B))
    {
      break;
    }
    _bp++;
  }while(1);

  addr_IopXxxControlFile=(ULONG)_bp-6;

  KdPrint(("向 IopXxxControlFile 写跳转地址为:%x\\n",addr_IopXxxControlFile));

  _bp=(BYTE*)call_addr_IopXxxControlFile;

  do

  {
    if((*( _bp-11)==0x6A)&&(*( _bp-6)==0xFF)&&(*( _bp-3)==0xFF)&&(*( _bp)==0xE8)&&(*( _bp+5)==0x83)&&(*( _bp+9)==0x8B))
    {
      break;
    }
    _bp++;
  }while(1);

  call_addr_IopXxxControlFile=(ULONG)_bp;

  __asm
  {
    push eax
    push ebx
    mov eax,call_addr_IopXxxControlFile
    add eax,1
    mov ebx,[eax]
    add eax,ebx
    add eax,5
    sub eax,1
    mov call_addr_IopXxxControlFile,eax
    pop ebx
    pop eax
  }

  KdPrint(("IopXxxControlFile 的 call 地址为:%x\\n",call_addr_IopXxxControlFile));

  __asm
  {
    push eax
    mov eax,My_IopXxxControlFile
    mov My_Function_Addr,eax
    pop eax
  }

  KdPrint(("自定义 IopXxxControlFile 的地址为:%x\\n",My_Function_Addr));

  _pjmp_code_IopXxxControlFile=(pjmp_code)addr_IopXxxControlFile;

  _jmp_code_IopXxxControlFile.E9=_pjmp_code_IopXxxControlFile->E9;

  _jmp_code_IopXxxControlFile.Fun_addr=_pjmp_code_IopXxxControlFile->Fun_addr;

  _jmp_code_IopXxxControlFile.nop[0]=_pjmp_code_IopXxxControlFile->nop[0];

  PAGED_Open();

  _pjmp_code_IopXxxControlFile->E9=0xE9;

  _pjmp_code_IopXxxControlFile->Fun_addr=(ULONG)(My_Function_Addr-addr_IopXxxControlFile-5);

  _pjmp_code_IopXxxControlFile->nop[0]=0x90;

  PAGED_Exit();

}

#pragma PAGEDCODE

VOID __declspec(naked) My_IopXxxControlFile()

{
  __asm
  {
    push dword ptr [ebp-40h]
    push dword ptr [ebp-24h]
    mov eax,addr_IopXxxControlFile
    add eax,0xB
    push eax
    mov eax,call_addr_IopXxxControlFile
    jmp eax
  }

}
```

## 代码片段 2：Unhook IopXxxControlFile 与 Hook NtGetContextThread 结构定义

```c
#pragma PAGEDCODE

VOID Un_IopXxxControlFile()

{
  PAGED_Open();

  _pjmp_code_IopXxxControlFile->E9=_jmp_code_IopXxxControlFile.E9;

  _pjmp_code_IopXxxControlFile->Fun_addr=_jmp_code_IopXxxControlFile.Fun_addr;

  _pjmp_code_IopXxxControlFile->nop[0]=_jmp_code_IopXxxControlFile.nop[0];

  PAGED_Exit();

}

jmp_code _jmp_code_NtGetContextThread;

pjmp_code _pjmp_code_NtGetContextThread;

#pragma INITCODE

VOID Hook_NtGetContextThread()

{
  ULONG My_Function_Addr;

  BYTE* _bp;

  _bp=(BYTE*)Getssdt_addr(in_NtGetContextThread);

  do

  {
    if((*( _bp-15)==0x50)&&(*( _bp-5)==0x6A)&&(*( _bp-3)==0xFF)&&(*( _bp)==0xE8)&&(*( _bp+5)==0x8B)&&(*( _bp+7)==0x85))
    {
      break;
    }
    _bp++;
  } while (1);

  addr_NtGetContextThread=(ULONG)_bp-5;

  KdPrint(("向 NtGetContextThread 写跳转的地址为:%x\\n",addr_NtGetContextThread));

  _bp=(BYTE*)Getssdt_addr(in_NtTerminateThread);

  do

  {
    if((*( _bp-15)==0x50)&&(*( _bp-5)==0x6A)&&(*( _bp-3)==0xFF)&&(*( _bp)==0xE8)&&(*( _bp+5)==0x8B)&&(*( _bp+7)==0x85))
    {
      break;
    }
    _bp++;
  } while (1);

  call_addr_NtGetContextThread=(ULONG)_bp;

  KdPrint(("NtGetContextThread 的 call 来自于:%x\\n",call_addr_NtGetContextThread));

  __asm
  {
    push eax
    push ebx
    mov eax,call_addr_NtGetContextThread
    add eax,1
    mov ebx,[eax]
    add eax,ebx
    add eax,5
    sub eax,1
    mov call_addr_NtGetContextThread,eax
    pop ebx
    pop eax
  }

  call_addr_NtSetContextThread=call_addr_NtGetContextThread;

  KdPrint(("从 NtTerminateThread 获取的 NtGetContextThread 的 call ObReferenceObjectByHandle 的地址为:%x\\n",call_addr_NtGetContextThread));

  __asm
  {
    push eax
    mov eax,My_NtGetContextThread
    mov My_Function_Addr,eax
    pop eax
  }

  KdPrint(("自定义 NtGetContextThread 的地址为:%x\\n",My_Function_Addr));

  Game_call_addr_1=addr_NtGetContextThread+5;

  __asm
  {
    push eax
    push ebx
    mov eax,Game_call_addr_1
    add eax,1
    mov ebx,[eax]
    add eax,ebx
    add eax,5
    sub eax,1
    mov Game_call_addr_1,eax
    pop ebx
    pop eax
  }

  KdPrint(("获取 NtGetContextThread 的 call 获取的第一个 call 地址为:%x\\n",Game_call_addr_1));

  _pjmp_code_NtGetContextThread=(pjmp_code)addr_NtGetContextThread;

  _jmp_code_NtGetContextThread.E9=_pjmp_code_NtGetContextThread->E9;

  _jmp_code_NtGetContextThread.Fun_addr=_pjmp_code_NtGetContextThread->Fun_addr;

  PAGED_Open();

  _pjmp_code_NtGetContextThread->E9=0xE9;

  _pjmp_code_NtGetContextThread->Fun_addr=(ULONG)(My_Function_Addr-addr_NtGetContextThread-5);

  PAGED_Exit();

}
```

## 代码片段 3：My_NtGetContextThread 实现

```c
PEPROCESS eprocess;

ANSI_STRING GetProcess_1;

ANSI_STRING OD_Name_1;

ANSI_STRING OD_Name_2;

ANSI_STRING OD_Name_3;

#pragma PAGEDCODE

VOID __declspec(naked) My_NtGetContextThread()

{
  __asm
  {
    pushad
    pushf
  }

  eprocess=PsGetCurrentProcess();

  RtlInitAnsiString(&GetProcess_1,(PCSZ)((ULONG)eprocess+0x174));

  RtlInitAnsiString(&OD_Name_1,_Game_Name);

  RtlInitAnsiString(&OD_Name_2,_Game_Name1);

  RtlInitAnsiString(&OD_Name_3,_Game_Name2);

  if(RtlCompareString(&GetProcess_1,&OD_Name_1,TRUE)==0)
  {
    __asm
    {
      popf
      popad
      push 8
      push dword ptr [ebp+8]
      mov eax,addr_NtGetContextThread
      add eax,0xA
      push eax
      jmp call_addr_NtGetContextThread
    }
  }

  else
  {
    if(RtlCompareString(&GetProcess_1,&OD_Name_2,TRUE)==0)
    {
      __asm
      {
        popf
        popad
        push 8
        push dword ptr [ebp+8]
        mov eax,addr_NtGetContextThread
        add eax,0xA
        push eax
        jmp call_addr_NtGetContextThread
      }
    }
    else
    {
      if(RtlCompareString(&GetProcess_1,&OD_Name_3,TRUE)!=0)
      {
        __asm
        {
          popf
          popad
          push 8
          push dword ptr [ebp+8]
          mov eax,addr_NtGetContextThread
          add eax,0xA
          push eax
          jmp call_addr_NtGetContextThread
        }
      }
      else
      {
        __asm
        {
          popf
          popad
          push 8
          push dword ptr [ebp+8]
          mov eax,addr_NtGetContextThread
          add eax,0xA
          push eax
          jmp Game_call_addr_1
        }
      }
    }
  }

}
```

## 代码片段 4：Unhook NtGetContextThread 与 NtSetContextThread 变量声明

```c
#pragma PAGEDCODE

VOID Un_NtGetContextThread()

{
  PAGED_Open();

  _pjmp_code_NtGetContextThread->E9=_jmp_code_NtGetContextThread.E9;

  _pjmp_code_NtGetContextThread->Fun_addr=_jmp_code_NtGetContextThread.Fun_addr;

  PAGED_Exit();

}

jmp_code _jmp_code_NtSetContextThread;

pjmp_code _pjmp_code_NtSetContextThread;

#pragma INITCODE
```

## 代码片段 5：Hook NtSetContextThread 部分实现

```c
VOID Hook_NtSetContextThread()

{
  ULONG My_Function_Addr;

  BYTE* _bp;

  _bp=(BYTE*)Getssdt_addr(in_NtSetContextThread);

  do

  {
    if((*( _bp-15)==0x50)&&(*( _bp-5)==0x6A)&&(*( _bp-3)==0xFF)&&(*( _bp)==0xE8)&&(*( _bp+5)==0x8B)&&(*( _bp+7)==0x85))
    {
      break;
    }
    _bp++;
  } while (1);

  addr_NtSetContextThread=(ULONG)_bp-5;
```



`KdPrint(("向 NtSetContextThread 写跳转地址的:%x\\n",addr_NtSetContextThread));`

`__asm`
```
{
  push eax
  mov eax,My_NtSetContextThread
  mov My_Function_Addr,eax
  pop eax
}
```
`KdPrint(("自定义 NtSetContextThread 的地址为:%x\\n",My_Function_Addr));`

`_pjmp_code_NtSetContextThread=(pjmp_code)addr_NtSetContextThread;`
`_jmp_code_NtSetContextThread.E9=_pjmp_code_NtSetContextThread->E9;`
`_jmp_code_NtSetContextThread.Fun_addr=_pjmp_code_NtSetContextThread->Fun_addr;`

`PAGED_Open();`
`_pjmp_code_NtSetContextThread->E9=0xE9;`
`_pjmp_code_NtSetContextThread->Fun_addr=(ULONG)(My_Function_Addr-addr_NtSetContextThread-5);`
`PAGED_Exit();`
```
}

#pragma PAGEDCODE

VOID __declspec(naked) My_NtSetContextThread()
{
  __asm
  {
    push 1
    push dword ptr [ebp+8]
    mov eax,addr_NtSetContextThread
    add eax,0xA
    push eax
    mov eax,call_addr_NtSetContextThread
    jmp eax
  }
}

#pragma PAGEDCODE

VOID Un_NtSetContextThread()
{
  PAGED_Open();
  _pjmp_code_NtSetContextThread->E9=_jmp_code_NtSetContextThread.E9;
  _pjmp_code_NtSetContextThread->Fun_addr=_jmp_code_NtSetContextThread.Fun_addr;
  PAGED_Exit();
}
```

`jmp_code _jmp_code_NtGetContextThread1;`
`pjmp_code _pjmp_code_NtGetContextThread1;`

`#pragma INITCODE`

`VOID Hook_NtGetContextThread1()`
```
{
  ULONG My_Function_Addr;
  BYTE* _bp;
  _bp=(BYTE*)Getssdt_addr(in_NtGetContextThread);
  do
  {
    if(( *(_bp-7)==0xBE)&&(*(_bp-2)==0x8B)&&(*(_bp-1)==0xCF)&&(*(_bp))==0xE8)&&(*(_bp+5)==0x5F)&&(*(_bp+6)==0x8B))
    {
      break;
    }
    _bp++;
  } while (1);
  addr_NtGetContextThread1=(ULONG)_bp-9;
  KdPrint(("向 NtGetContextThread 写跳转地址为:%x\\n",addr_NtGetContextThread1));
  _bp=(BYTE*)Getssdt_addr(in_NtTerminateThread);
  do
  {
    if(( *(_bp-9)==0xE8)&&(*(_bp-4)==0x8B)&&(*(_bp-2)==0x8B)&&(*(_bp))==0xE8)&&(*(_bp+5)==0x8B)&&(*(_bp+6)==0xC7))
    {
      break;
    }
    _bp++;
  } while (1);
  call_addr_NtGetContextThread1=(ULONG)_bp;
  __asm
  {
    push eax
    push ebx
    mov eax,call_addr_NtGetContextThread1
    add eax,1
    mov ebx,[eax]
    add eax,ebx
    add eax,5
    sub eax,1
    mov call_addr_NtGetContextThread1,eax
    pop ebx
    pop eax
  }
  KdPrint(("从 NtTerminateThread 获取的 NtGetContextThread 的第二个 call 地址为:%x\\n",call_addr_NtGetContextThread1));
  __asm
  {
    push eax
    mov eax,My_NtGetContextThread1
    mov My_Function_Addr,eax
    pop eax
  }
  KdPrint(("NtGetContextThread 的第二个自写函数地址为:%x\\n",My_Function_Addr));
  Game_call_addr_2=addr_NtGetContextThread1+9;
  __asm
  {
    push eax
    push ebx
    mov eax,Game_call_addr_2
    add eax,1
    mov ebx,[eax]
    add eax,ebx
    add eax,5
    sub eax,1
    mov Game_call_addr_2,eax
    pop ebx
    pop eax
  }
  KdPrint(("获取 NtGetContextThread 的 call 获取的第二个 call 地址为:%x\\n",Game_call_addr_2));
  _pjmp_code_NtGetContextThread1=(pjmp_code)addr_NtGetContextThread1;
  _jmp_code_NtGetContextThread1.E9=_pjmp_code_NtGetContextThread1->E9;
  _jmp_code_NtGetContextThread1.Fun_addr=_pjmp_code_NtGetContextThread1->Fun_addr;
  _jmp_code_NtGetContextThread1.nop[0]=_pjmp_code_NtGetContextThread1->nop[0];
  _jmp_code_NtGetContextThread1.nop[1]=_pjmp_code_NtGetContextThread1->nop[1];
  PAGED_Open();
  _pjmp_code_NtGetContextThread1->E9=0xE9;
  _pjmp_code_NtGetContextThread1->Fun_addr=(ULONG)(My_Function_Addr-addr_NtGetContextThread1-5);
  _pjmp_code_NtGetContextThread1->nop[0]=0x90;
  _pjmp_code_NtGetContextThread1->nop[1]=0x90;
  PAGED_Exit();
}
```

`PEPROCESS eprocess_temp;`
`ANSI_STRING GetProcess_temp1;`
`ANSI_STRING OD_Name_temp1;`
`ANSI_STRING OD_Name_temp2;`
`ANSI_STRING OD_Name_temp3;`

`#pragma PAGEDCODE`

`VOID __declspec(naked) My_NtGetContextThread1()`
```
{
  __asm
  {
    pushad
    pushf
  }
  eprocess_temp=PsGetCurrentProcess();
  RtlInitAnsiString(&GetProcess_temp1,(PCSZ)((ULONG)eprocess_temp+0x174));
  RtlInitAnsiString(&OD_Name_temp1,_Game_Name);
  RtlInitAnsiString(&OD_Name_temp2,_Game_Name1);
  RtlInitAnsiString(&OD_Name_temp3,_Game_Name2);
  if(RtlCompareString(&GetProcess_temp1,&OD_Name_temp1,TRUE)==0)
  {
    __asm
    {
      popf
      popad
      jmp __tiao2
      mov esi,0C0000008h
__tiao2:
      mov ecx,edi
      mov eax,addr_NtGetContextThread1
      add eax,0xE
      push eax
      jmp call_addr_NtGetContextThread1
    }
  }
  else
  {
    if(RtlCompareString(&GetProcess_temp1,&OD_Name_temp2,TRUE)==0)
    {
      __asm
      {
        popf
        popad
        jmp __tiaotemp1
        mov esi,0C0000008h
__tiaotemp1:
        mov ecx,edi
        mov eax,addr_NtGetContextThread1
        add eax,0xE
        push eax
        jmp call_addr_NtGetContextThread1
      }
    }
    else
    {
      if(RtlCompareString(&GetProcess_temp1,&OD_Name_temp3,TRUE)==0)
      {
        __asm
        {
          popf
          popad
          jmp __tiaotemp2
          mov esi,0C0000008h
__tiaotemp2:
          mov ecx,edi
          mov eax,addr_NtGetContextThread1
          add eax,0xE
          push eax
          jmp call_addr_NtGetContextThread1
        }
      }
      else
      {
        __asm
        {
          popf
          popad
          jmp __tiaotemp3
          mov esi,0C0000008h
__tiaotemp3:
          mov ecx,edi
          mov eax,addr_NtGetContextThread1
          add eax,0xE
          push eax
          jmp Game_call_addr_2
        }
      }
    }
  }
}
```

`#pragma PAGEDCODE`

`VOID Un_NtGetContextThread1()`
```
{
  PAGED_Open();
  _pjmp_code_NtGetContextThread1->E9=_jmp_code_NtGetContextThread1.E9;
  _pjmp_code_NtGetContextThread1->Fun_addr=_jmp_code_NtGetContextThread1.Fun_addr;
  _pjmp_code_NtGetContextThread1->nop[0]=_jmp_code_NtGetContextThread1.nop[0];
  _pjmp_code_NtGetContextThread1->nop[1]=_pjmp_code_NtGetContextThread1->nop[1];
  PAGED_Exit();
}
```

`jmp_code _jmp_code_NtQueryPerformanceCounter;`
`pjmp_code _pjmp_code_NtQueryPerformanceCounter;`
`jmp_code _jmp_code_NtQueryPerformanceCounter_1;`
`pjmp_code _pjmp_code_NtQueryPerformanceCounter_1;`

`#pragma INITCODE`

`VOID Hook_NtQueryPerformanceCounter()`
```
{
  ULONG My_temp;
  BYTE* _bp;
  _bp=(BYTE*)addr_NtOpenThread;
  do
  {
    if(( *(_bp-18)==0xC3)&&(*(_bp-14)==0x8B)&&(*(_bp-7)==0xEB)&&(*(_bp))==0xE8)&&(*(_bp+5)==0xC2)&&(*(_bp+6)==0x10))
    {
      break;
    }
    _bp++;
  } while (1);
  call_addr_NtQueryPerformanceCounter=(ULONG)_bp;
  __asm
  {
    push eax
    push ebx
    mov eax,call_addr_NtQueryPerformanceCounter
    add eax,1
    mov ebx,[eax]
    add eax,ebx
    add eax,5
    sub eax,1
    mov call_addr_NtQueryPerformanceCounter,eax
    pop ebx
    pop eax
  }
  call_addr_NtQueryPerformanceCounter=call_addr_NtQueryPerformanceCounter-addr_NtQueryPerformanceCounter;
  KdPrint(("NtQueryPerformanceCounter 的 call 地址为:%x\\n",call_addr_NtQueryPerformanceCounter));
  _bp=(BYTE*)Getssdt_addr(in_NtQueryPerformanceCounter);
  do
  {
    if(( *(_bp-10)==0x8B)&&(*(_bp-7)==0x89)&&(*(_bp-4)==0x83)&&(*(_bp))==0xEB)&&(*(_bp+2)==0x8B)&&(*(_bp+5)==0x8B))
    {
      break;
    }
    _bp++;
```



```
  } while (1);  

  addr_NtQueryPerformanceCounter=(ULONG)_bp;  

  KdPrint(("第一个 NtQueryPerformanceCounter 的 JMP 地址为:%x\n",addr_NtQueryPerformanceCounter));  

  _bp=(BYTE*)addr_NtQueryPerformanceCounter;  

  do  

  {  

    if((*( _bp-11)==0xC3)&&(*(_bp-10)==0x8B)&&(*(_bp-7)==0x83)&&(*(_bp-3)==0x8B)&&(*(_bp)==0xEB)&&(*(_bp+5)==0x50)&&(*(_bp+6)==0xFF))  

    {  

      break;  

    }  

    _bp++;  

  } while (1);  

  addr1_NtQueryPerformanceCounter=(ULONG)_bp;  

  KdPrint(("第二个 NtQueryPerformanceCounter 的 JMP 地址为:%x\n",addr1_NtQueryPerformanceCounter));  

  __asm  

  {  

    push eax  
      mov eax,My_NtQueryPerformanceCounter_1  
      mov My_temp,eax  
      pop eax  

  }  

  KdPrint(("NtQueryPerformanceCounter 第一个自定义函数地址为:%x\n",My_temp));  

  _pjmp_code_NtQueryPerformanceCounter=(pjmp_code)addr_NtQueryPerformanceCounter;  

  _jmp_code_NtQueryPerformanceCounter.E9=_pjmp_code_NtQueryPerformanceCounter->E9;  

  _jmp_code_NtQueryPerformanceCounter.Fun_addr=_pjmp_code_NtQueryPerformanceCounter->Fun_addr;  

  PAGED_Open();  

  _pjmp_code_NtQueryPerformanceCounter->E9=0xE9;  

  _pjmp_code_NtQueryPerformanceCounter->Fun_addr=(ULONG)(My_temp-addr_NtQueryPerformanceCounter-5);  

  PAGED_Exit();  

  __asm  

  {  

    push eax  
    mov eax,My_NtQueryPerformanceCounter_2  
    mov My_temp,eax  
    pop eax  

  }  

  KdPrint(("NtQueryPerformanceCounter 第二个自定义函数地址为:%x\n",My_temp));  

  _pjmp_code_NtQueryPerformanceCounter_1=(pjmp_code)addr1_NtQueryPerformanceCounter;  

  _jmp_code_NtQueryPerformanceCounter_1.E9=_pjmp_code_NtQueryPerformanceCounter_1->E9;  

  _jmp_code_NtQueryPerformanceCounter_1.Fun_addr=_pjmp_code_NtQueryPerformanceCounter_1->Fun_addr;  

  PAGED_Open();  

  _pjmp_code_NtQueryPerformanceCounter_1->E9=0xE9;  

  _pjmp_code_NtQueryPerformanceCounter_1->Fun_addr=(ULONG)(My_temp-addr1_NtQueryPerformanceCounter-5);  

  PAGED_Exit();  

}  

#pragma PAGEDCODE  

VOID __declspec(naked)My_NtQueryPerformanceCounter_1()  

{  

  __asm  

  {  

    mov eax,addr_NtQueryPerformanceCounter  
    add eax,0x47  
    push eax  
    xor eax,eax  
    jmp call_addr_NtQueryPerformanceCounter  

  }  

}  

#pragma PAGEDCODE  

VOID __declspec(naked)My_NtQueryPerformanceCounter_2()  

{  

  __asm  

  {  

    pushad  
    mov eax,addr1_NtQueryPerformanceCounter  
    add eax,0x2D  
    push eax  
    popad  
    jmp call_addr_NtQueryPerformanceCounter  

  }  

}  

#pragma PAGEDCODE  

VOID Un_NtQueryPerformanceCounter()  

{  

  PAGED_Open();  

  _pjmp_code_NtQueryPerformanceCounter->E9=_jmp_code_NtQueryPerformanceCounter.E9;  

  _pjmp_code_NtQueryPerformanceCounter->Fun_addr=_jmp_code_NtQueryPerformanceCounter.Fun_addr;  

  _pjmp_code_NtQueryPerformanceCounter_1->E9=_jmp_code_NtQueryPerformanceCounter_1.E9;  

  _pjmp_code_NtQueryPerformanceCounter_1->Fun_addr=_jmp_code_NtQueryPerformanceCounter_1.Fun_addr;  

  PAGED_Exit();  

}  

struct _str_index  

{  

  ULONG KiDispatchException;        //1 处  

  ULONG NtQueryInformationProcess;      //1 处  

  ULONG PspCreateProcess;        //1 处  

  ULONG PsGetProcessDebugPort;      //1 处  

  ULONG PsIsProcessBeingDebugged;      //1 处  

  ULONG NtTerminateProcess;        //1 处  

  ULONG PspTerminateProcess;      //1 处  

  ULONG PspExitThread;        //1 处  

  ULONG PspProcessDelete[2];       //3 处  

  ULONG ObpCloseHandleTableEntry;      //1 处  

  ULONG ObpCloseHandle;        //1 处  

  ULONG MmCreatePeb;        //1 处  

  ULONG DbgkCreateThread;        //1 处  

  ULONG DbgkExitThread;        //1 处  

  ULONG DbgkExitProcess;        //1 处  

  ULONG DbgkMapViewOfSection;      //1 处  

  ULONG DbgkUnMapViewOfSection;      //1 处  

  ULONG DbgkpMarkProcessPeb;      //1 处  

  ULONG DbgkpCloseObject[2];        //3 处  

  ULONG DbgkCopyProcessDebugPort[3];      //4 处  

  ULONG DbgkOpenProcessDebugPort[1];      //2 处  

  ULONG DbgkpQueueMessage;        //1 处  

  ULONG DbgkClearProcessDebugObject[1];      //2 处  

  ULONG DbgkpSetProcessDebugObject[4];      //5 处  

  ULONG DbgkForwardException;      //1 处  

}str_index,*pstr_index;  

BYTE debugport_in[2]={0xBC,0x00};   //还原修改的函数  

BYTE debugPort_out[2]={0x5C,0x02};  //将 DebugPort+bc 移至 DebugPort+25C  

#pragma PAGEDCODE  

VOID Hook_DebugPort()  

{  

  /*  

  1.枚举模块获取首地址  

  2.定位函数地址  

  3.修改函数  

  */  

  NTSTATUS status;  

  ULONG len;  

  PVOID Buffer=NULL;  

  PCHAR GamePath;  

  ULONG addr_start;  

  ULONG addr_size;  

  PSYSTEM_MODULE_INFORMATION enum_module;  

  ZwQuerySystemInformation((SYSTEM_INFORMATION_CLASS)11,NULL,0,&len);  

  Buffer=ExAllocatePool(NonPagedPool,len);  

  if(Buffer==NULL)  

  {  

    KdPrint(("分配内存失败!\r\n"));  

    return ;  

  }  

  status=ZwQuerySystemInformation((SYSTEM_INFORMATION_CLASS)11,Buffer,len,NULL);  

  enum_module=(PSYSTEM_MODULE_INFORMATION)Buffer;  

  addr_start=(ULONG)enum_module->Module[0].Base;  

  addr_size=addr_start+(ULONG)enum_module->Module[0].Size;  

  ExFreePool(Buffer);  

  KdPrint(("模块名:%s   入口地址为:%x   模块结束地址:%x\r\n",enum_module->Module[0].ImageName,addr_start,addr_size));  

  BYTE* _bp;  

  _bp=(BYTE*)addr_start;  

  do  

  {  

    if((*( _bp-9)==0x64)&&(*(_bp-3)==0x8B)&&(*(_bp)==0x39)&&(*(_bp+6)==0x74)&&(*(_bp+8)==0x6A)&&(*(_bp+10)==0x8D))  

    {  

      str_index.KiDispatchException=(ULONG)_bp+2;  

      KdPrint(("KiDispatchException 访问 DebugPort 的地址为:%x\n",str_index.KiDispatchException));  

      break;  

    }  

    _bp++;  

  } while (1);  

  _bp=(BYTE*)addr_start;  

  do  

  {  

    if((*( _bp-19)==0xE8)&&(*(_bp-14)==0x85)&&(*(_bp-12)==0x0F)&&(*(_bp-6)==0x8B)&&(*(_bp)==0x8B)&&(*(_bp+6)==0xF7))  

    {  

      str_index.NtQueryInformationProcess=(ULONG)_bp+2;  

      KdPrint(("NtQueryInformationProecss 访问 DebugPort 的地址为:%x\n",str_index.NtQueryInformationProcess));  

      break;  

    }  

    _bp++;  

  } while (1);  

  _bp=(BYTE*)addr_start;  

  do  

  {  

    if((*( _bp-13)==0x8B)&&(*(_bp-11)==0x3B)&&(*(_bp-9)==0x0F)&&(*(_bp-3)==0x8B)&&(*(_bp)==0x89)&&(*(_bp+6)==0xF6)&&(*(_bp+10)==0x74))  

    {  

      str_index.PspCreateProcess=(ULONG)_bp+2;  

      KdPrint(("PspCreateProess 访问 DebugPort 的地址为:%x\n",str_index.PspCreateProcess));  

      break;  

    }  

    _bp++;  

  } while (1);  

  _bp=(BYTE*)addr_start;  

  do  

  {  

    if((*( _bp-5)==0x8B)&&(*(_bp-3)==0x8B)&&(*(_bp)==0x8B)&&(*(_bp+1)==0x80)&&(*(_bp+2)==0xBC)&&(*(_bp+6)==0x5D)&&(*(_bp+7)==0xC2)&&(*(_bp+8)==0x04))  

    {  

      str_index.PsGetProcessDebugPort=(ULONG)_bp+2;  

      KdPrint(("PsGetProcessDeubugPort 访问 DebugPort 的地址为:%x\n",str_index.PsGetProcessDebugPort));  

      break;  

    }  

    _bp++;  

  }while(1);  

  _bp=(BYTE*)addr_start;  

  do  

  {  

    if((*( _bp-5)==0x8B)&&(*(_bp-3)==0x8B)&&(*(_bp)==0x83)&&(*(_bp+1)==0xB8)&&(*(_bp+7)==0x0F)&&(*(_bp+10)==0x5D)&&(*(_bp+11)==0xC2))  

    {  

      str_index.PsIsProcessBeingDebugged=(ULONG)_bp+2;  

      KdPrint(("PsIsProcessBeingDebugged 访问 DebugPort 的地址为:%x\n",str_index.PsIsProcessBeingDebugged));  

      break;  

    }  

    _bp++;  

  } while (1);  

  _bp=(BYTE*)addr_start;  

  do  

  {  

    if((*( _bp-9)==0x81)&&(*(_bp-2)==0x74)&&(*(_bp)==0x83)&&(*(_bp+1)==0xBB)&&(*(_bp+2)==0xBC)&&(*(_bp+7)==0x74)&&(*(_bp+9)==0x80))  

    {  

      str_index.NtTerminateProcess=(ULONG)_bp+2;  

      KdPrint(("NtTerminateProecss 访问 DebugPort 的地址为:%x\n",str_index.NtTerminateProcess));  

      break;  

    }  

    _bp++;  

  } while (1);  

  _bp=(BYTE*)addr_start;  

  do  

  {  
```



```
if((\*(\_bp-6)==0x8B)&&(\*(\_bp-4)==0x85)&&(\*(\_bp-2)==0x75)&&(\*(\_bp)==0x39)&&(\*(\_bp+1)==0x86)&&(\*(\_bp+2)==0xBC)&&(\*(\_bp+6)==0x74)&&(\*(\_bp+8)==0x56))
{
  str\_index.PspTerminateProcess=(ULONG)\_bp+2;
  KdPrint(("PspTerminateProecss 访问 DebugPort 的地址为:%x\n",str\_index.PspTerminateProcess));
  break;
}
\_bp++;
} while (1);
\_bp=(BYTE\*)addr\_start;
do
{
  if((\*(\_bp-5)==0xE8)&&(\*(\_bp)==0x83)&&(\*(\_bp+1)==0xBF)&&(\*(\_bp+2)==0xBC)&&(\*(\_bp+7)==0x74)&&(\*(\_bp+9)==0xF6))
  {
    str\_index.PspExitThread=(ULONG)\_bp+2;
    KdPrint(("PspExitThread 访问 DebugPort 的地址为:%x\n",str\_index.PspExitThread));
    break;
  }
  \_bp++;
} while (1);
\_bp=(BYTE\*)addr\_start;
do
{
  if((\*(\_bp-8)==0x8B)&&(\*(\_bp-5)==0xE8)&&(\*(\_bp)==0x83)&&(\*(\_bp+1)==0xBF)&&(\*(\_bp+2)==0xBC)&&(\*(\_bp+7)==0x74)&&(\*(\_bp+9)==0xF6))
  {
    str\_index.PspProcessDelete\[0\]=(ULONG)\_bp+2;
    KdPrint(("第一处 PspProcessDelete 访问 DebugPotr 的地址为:%x\n",str\_index.PspProcessDelete\[0\]));
    break;
  }
  \_bp++;
} while (1);
\_bp=(BYTE\*)addr\_start;
do
{
  if((\*(\_bp-11)==0xE8)&&(\*(\_bp-6)==0x89)&&(\*(\_bp-4)==0x66)&&(\*(\_bp)==0x8D)&&(\*(\_bp+1)==0xBE)&&(\*(\_bp+2)==0xBC)&&(\*(\_bp+6)==0x8B))
  {
    str\_index.PspProcessDelete\[1\]=(ULONG)\_bp+2;
    KdPrint(("第二处 PspProcessDelete 访问 DebugPotr 的地址为:%x\n",str\_index.PspProcessDelete\[1\]));
    break;
  }
  \_bp++;
} while (1);
\_bp=(BYTE\*)addr\_start;
do
{
  if((\*(\_bp-9)==0x81)&&(\*(\_bp-2)==0x74)&&(\*(\_bp)==0x83)&&(\*(\_bp+1)==0xBB)&&(\*(\_bp+2)==0xBC)&&(\*(\_bp+7)==0x74))
  {
    str\_index.PspProcessDelete\[2\]=(ULONG)\_bp+2;
    KdPrint(("第三处 PspProcessDelete 访问 DebugPotr 的地址为:%x\n",str\_index.PspProcessDelete\[2\]));
    break;
  }
  \_bp++;
} while (1);
\_bp=(BYTE\*)addr\_start;
do
{
  if((\*(\_bp-9)==0x64)&&(\*(\_bp-3)==0x8B)&&(\*(\_bp)==0x83)&&(\*(\_bp+1)==0xB8)&&(\*(\_bp+2)==0xBC)&&(\*(\_bp+7)==0x74))
  {
    str\_index.ObpCloseHandleTableEntry=(ULONG)\_bp+2;
    KdPrint(("ObpCloseHandleTableEntry 访问 DebugPort 的地址为:%x\n",str\_index.ObpCloseHandleTableEntry));
    break;
  }
  \_bp++;
} while (1);
\_bp=(BYTE\*)addr\_start;
do
{
  if((\*(\_bp-9)==0xF6)&&(\*(\_bp-2)==0x75)&&(\*(\_bp)==0x39)&&(\*(\_bp+1)==0x8B)&&(\*(\_bp+2)==0xBC)&&(\*(\_bp+6)==0x74))
  {
    str\_index.ObpCloseHandle=(ULONG)\_bp+2;
    KdPrint(("ObpCloseHandle 访问 DebugPort 的地址为:%x\n",str\_index.ObpCloseHandle));
    break;
  }
  \_bp++;
} while (1);
\_bp=(BYTE\*)addr\_start;
do
{
  if((\*(\_bp-8)==0x8B)&&(\*(\_bp-5)==0x89)&&(\*(\_bp-2)==0x33)&&(\*(\_bp)==0x39)&&(\*(\_bp+1)==0x9E)&&(\*(\_bp+6)==0x0F))
  {
    str\_index.MmCreatePeb=(ULONG)\_bp+2;
    KdPrint(("MmCreatePeb 访问 DebugPort 的地址为:%x\n",str\_index.MmCreatePeb));
    break;
  }
  \_bp++;
} while (1);
\_bp=(BYTE\*)addr\_start;
do
{
  if((\*(\_bp-6)==0x50)&&(\*(\_bp-5)==0xE8)&&(\*(\_bp)==0x39)&&(\*(\_bp+1)==0x9E)&&(\*(\_bp+2)==0xBC)&&(\*(\_bp+6)==0x0F)&&(\*(\_bp+12)==0x39))
  {
    str\_index.DbgkCreateThread=(ULONG)\_bp+2;
    KdPrint(("DbgkCreateThread 访问 DebugPort 的地址为:%x\n",str\_index.DbgkCreateThread));
    break;
  }
  \_bp++;
} while (1);
\_bp=(BYTE\*)addr\_start;
do
{
  if((\*(\_bp-9)==0xF6)&&(\*(\_bp-2)==0x75)&&(\*(\_bp)==0x8B)&&(\*(\_bp+1)==0x89)&&(\*(\_bp+2)==0xBC)&&(\*(\_bp+6)==0x85))
  {
    str\_index.DbgkExitThread=(ULONG)\_bp+2;
    KdPrint(("DbgkExitThread 访问 DebugPort 的地址为:%x\n",str\_index.DbgkExitThread));
    break;
  }
  \_bp++;
} while (1);
\_bp=(BYTE\*)str\_index.DbgkExitThread+10;
do
{
  if((\*(\_bp-9)==0xF6)&&(\*(\_bp-2)==0x75)&&(\*(\_bp)==0x8B)&&(\*(\_bp+1)==0x89)&&(\*(\_bp+2)==0xBC)&&(\*(\_bp+6)==0x85))
  {
    str\_index.DbgkExitProcess=(ULONG)\_bp+2;
    KdPrint(("DbgkExitProcess 访问 DebugProt 的地址为:%x\n",str\_index.DbgkExitProcess));
    break;
  }
  \_bp++;
} while (1);
\_bp=(BYTE\*)addr\_start;
do
{
  if((\*(\_bp-6)==0x74)&&(\*(\_bp-4)==0x33)&&(\*(\_bp-2)==0xEB)&&(\*(\_bp)==0x8B)&&(\*(\_bp+1)==0x89)&&(\*(\_bp+6)==0x3B))
  {
    str\_index.DbgkMapViewOfSection=(ULONG)\_bp+2;
    KdPrint(("DbgkMapViewOfSection 访问 DebugPort 的地址为:%x\n",str\_index.DbgkMapViewOfSection));
    break;
  }
  \_bp++;
} while (1);
\_bp=(BYTE\*)addr\_start;
do
{
  if((\*(\_bp-9)==0xF6)&&(\*(\_bp-2)==0x75)&&(\*(\_bp)==0x83)&&(\*(\_bp+1)==0xB9)&&(\*(\_bp+2)==0xBC)&&(\*(\_bp+7)==0x74))
  {
    str\_index.DbgkUnMapViewOfSection=(ULONG)\_bp+2;
    KdPrint(("DbgkUnMapViewOfSection 访问 DebugPort 的地址为:%x\n",str\_index.DbgkUnMapViewOfSection));
    break;
  }
  \_bp++;
} while (1);
\_bp=(BYTE\*)addr\_start;
do
{
  if((\*(\_bp-11)==0xFF)&&(\*(\_bp-5)==0x89)&&(\*(\_bp-2)==0x33)&&(\*(\_bp)==0x39)&&(\*(\_bp+1)==0xBE)&&(\*(\_bp+6)==0x0F)&&(\*(\_bp+9)==0x8B))
  {
    str\_index.DbgkpMarkProcessPeb=(ULONG)\_bp+2;
    KdPrint(("DbgkpMarkProcessPeb 访问 DebugPort 的地址为:%x\n",str\_index.DbgkpMarkProcessPeb));
    break;
  }
  \_bp++;
} while (1);
\_bp=(BYTE\*)addr\_start;
do
{
  if((\*(\_bp-5)==0xEB)&&(\*(\_bp-3)==0x8B)&&(\*(\_bp)==0x81)&&(\*(\_bp+1)==0xC3)&&(\*(\_bp+2)==0xBC)&&(\*(\_bp+6)==0x39)&&(\*(\_bp+8)==0x75))
  {
    str\_index.DbgkpCloseObject\[0\]=(ULONG)\_bp+2;
    KdPrint(("第一个 DbgkpCloseObject 访问 DeBugPort 的地址为:%x\n",str\_index.DbgkpCloseObject\[0\]));
    break;
  }
  \_bp++;
} while (1);
\_bp=(BYTE\*)addr\_start;
do
{
  if((\*(\_bp-5)==0xFF)&&(\*(\_bp-3)==0x8B)&&(\*(\_bp)==0x05)&&(\*(\_bp+1)==0xBC)&&(\*(\_bp+5)==0x8B)&&(\*(\_bp+7)==0x33))
  {
    str\_index.DbgkpCloseObject\[1\]=(ULONG)\_bp+1;
    KdPrint(("第二个 DbgkpCloseObject 访问 DebugPort 的地址为:%x\n",str\_index.DbgkpCloseObject\[1\]));
    break;
  }
  \_bp++;
} while (1);
\_bp=(BYTE\*)addr\_start;
do
{
  if((\*(\_bp-6)==0xC6)&&(\*(\_bp-2)==0xFF)&&(\*(\_bp)==0x39)&&(\*(\_bp+1)==0x9F)&&(\*(\_bp+6)==0x75)&&(\*(\_bp+8)==0x8B))
  {
    str\_index.DbgkpCloseObject\[2\]=(ULONG)\_bp+2;
    KdPrint(("第三个 DbgkpCloseObject 访问 DebugPort 的地址为:%x\n",str\_index.DbgkpCloseObject\[2\]));
    break;
  }
  \_bp++;
}while(1);
\_bp=(BYTE\*)addr\_start;
do
{
  if((\*(\_bp-6)==0x55)&&(\*(\_bp-5)==0x8B)&&(\*(\_bp-3)==0x8B)&&(\*(\_bp)==0x83)&&(\*(\_bp+1)==0xA0)&&(\*(\_bp+2)==0xBC)&&(\*(\_bp+7)==0x56))
  {
    str\_index.DbgkCopyProcessDebugPort\[0\]=(ULONG)\_bp+2;
    KdPrint(("第一个 DbgkCopyProcessDebugPort 访问 DebugPort 的地址为:%x\n",str\_index.DbgkCopyProcessDebugPort\[0\]));
    break;
  }
  \_bp++;
} while (1);
\_bp=(BYTE\*)addr\_start;
do
{
  if((\*(\_bp-4)==0x56)&&(\*(\_bp-3)==0x8B)&&(\*(\_bp)==0x81)&&(\*(\_bp+1)==0xC6)&&(\*(\_bp+2)=0xBC)&&(\*(\_bp+6)==0x83)&&(\*(\_bp+9)==0x74))
  {
    str\_index.DbgkCopyProcessDebugPort\[1\]=(ULONG)\_bp+2;
    KdPrint(("第二个 DbgkCopyProcessDebugPort 访问 DebugPort 的地址为:%x\n",str\_index.DbgkCopyProcessDebugPort\[1\]));
    break;
  }
  \_bp++;
} while (1);
\_bp=(BYTE\*)addr\_start;
do
{
  if((\*(\_bp-8)==0xE8)&&(\*(\_bp-3)==0x8B)&&(\*(\_bp)==0x89)&&(\*(\_bp+1)==0xB0)&&(\*(\_bp+2)==0xBC)&&(\*(\_bp+6)==0x8B))
  {
    str\_index.DbgkCopyProcessDebugPort\[2\]=(ULONG)\_bp+2;
```



```
KdPrint(("第三个 DbgkCopyProcessDebugPort 访问 DebugPort 的地址为:%x\\n",str_index.DbgkCopyProcessDebugPort[2]));  

break;  

}  

_bp++;  

} while (1);  

_bp=(BYTE*)addr_start;  

do  

{  

if((\*(_bp-5)==0x51)&&(\*(_bp-4)==0x56)&&(\*(_bp-3)==0x8B)&&(\*(_bp)==0x81)&&(\*(_bp+1)==0xC6)&&(\*(_bp+6)==0x83)&&(\*(_bp+9)==0xC7))  

{  

  str_index.DbgkCopyProcessDebugPort[3]=(ULONG)_bp+2;  

  KdPrint(("第四个 DbgkCopyProcessDebugPort 访问 DebugPort 的地址为:%x\\n",str_index.DbgkCopyProcessDebugPort[3]));  

  break;  

}  

_bp++;  

} while (1);  

_bp=(BYTE*)addr_start;  

do  

{  

if((\*(_bp-5)==0x51)&&(\*(_bp-4)==0x56)&&(\*(_bp-3)==0x8B)&&(\*(_bp)==0x81)&&(\*(_bp+1)==0xC6)&&(\*(_bp+6)==0x83)&&(\*(_bp+9)==0xC7))  

{  

  str_index.DbgkOpenProcessDebugPort[0]=(ULONG)_bp+2;  

  KdPrint(("第一个 DbgkOpenProcessDebugPort 的访问地址为:%x\\n",str_index.DbgkOpenProcessDebugPort[0]));  

  break;  

}  

_bp++;  

} while (1);  

_bp=(BYTE*)addr_start;  

do  

{  

if((\*(_bp-9)==0xFF)&&(\*(_bp-3)==0x8B)&&(\*(_bp)==0x8B)&&(\*(_bp+1)==0x80)&&(\*(_bp+6)==0x89)&&(\*(_bp+9)==0x8B))  

{  

  str_index.DbgkOpenProcessDebugPort[1]=(ULONG)_bp+2;  

  KdPrint(("第二个 DbgkOpenProcessDebugPort 访问 DebugPort 的地址为:%x\\n",str_index.DbgkOpenProcessDebugPort[1]));  

  break;  

}  

_bp++;  

} while (1);  

_bp=(BYTE*)addr_start;  

do  

{  

if((\*(_bp-9)==0xFF)&&(\*(_bp-3)==0x8B)&&(\*(_bp)==0x8B)&&(\*(_bp+1)==0x80)&&(\*(_bp+6)==0x89)&&(\*(_bp+9)==0x8B))  

{  

  str_index.DbgkpQueueMessage=(ULONG)_bp+2;  

  KdPrint(("DbgkpQueueMessage 访问 DebugPort 的地址为:%x\\n",str_index.DbgkpQueueMessage));  

  break;  

}  

_bp++;  

} while (1);  

_bp=(BYTE*)addr_start;  

do  

{  

if((\*(_bp-5)==0xFF)&&(\*(_bp-3)==0x8B)&&(\*(_bp)==0x05)&&(\*(_bp+1)==0xBC)&&(\*(_bp+5)==0x8B)&&(\*(_bp+7)==0x33))  

{  

  str_index.DbgkClearProcessDebugObject[0]=(ULONG)_bp+1;  

  KdPrint(("DbgkClearProcessDebugObject 访问 DebugPort 的地址为:%x\\n",str_index.DbgkClearProcessDebugObject[0]));  

  break;  

}  

_bp++;  

} while (1);  

_bp=(BYTE*)addr_start;  

do  

{  

if((\*(_bp-6)==0xC6)&&(\*(_bp-2)==0xFF)&&(\*(_bp)==0x39)&&(\*(_bp+1)==0x9F)&&(\*(_bp+6)==0x75)&&(\*(_bp+8)==0x8B))  

{  

  str_index.DbgkClearProcessDebugObject[1]=(ULONG)_bp+2;  

  KdPrint(("DbgkClearProcessDebugObject 访问 DebugPort 的地址为:%x\\n",str_index.DbgkClearProcessDebugObject[1]));  

  break;  

}  

_bp++;  

} while (1);  

_bp=(BYTE*)addr_start;  

do  

{  

if((\*(_bp-6)==0xC6)&&(\*(_bp-2)==0xFF)&&(\*(_bp)==0x39)&&(\*(_bp+1)==0x9F)&&(\*(_bp+6)==0x75)&&(\*(_bp+8)==0x8B))  

{  

  str_index.DbgkpSetProcessDebugObject[0]=(ULONG)_bp+2;  

  KdPrint(("第一个 DbgkpSetProcessDebugObject 访问 DebugPort 的地址为:%x\\n",str_index.DbgkpSetProcessDebugObject[0]));  

  break;  

}  

_bp++;  

} while (1);  

_bp=(BYTE*)addr_start;  

do  

{  

if((\*(_bp-6)==0x8B)&&(\*(_bp-3)==0x8B)&&(\*(_bp)==0x89)&&(\*(_bp+1)==0x87)&&(\*(_bp+6)==0xE8)&&(\*(_bp+11)==0xFF))  

{  

  str_index.DbgkpSetProcessDebugObject[1]=(ULONG)_bp+2;  

  KdPrint(("第二个 DbgkpSetProcessDebugObject 访问 DebugPort 的地址为:%x\\n",str_index.DbgkpSetProcessDebugObject[1]));  

  break;  

}  

_bp++;  

} while (1);  

_bp=(BYTE*)addr_start;  

do  

{  

if((\*(_bp-6)==0x8B)&&(\*(_bp-4)==0x85)&&(\*(_bp-2)==0x74)&&(\*(_bp)==0x83)&&(\*(_bp+1)==0xA7)&&(\*(_bp+7)==0xB9))  

{  

  str_index.DbgkpSetProcessDebugObject[2]=(ULONG)_bp+2;  

  KdPrint(("第三个 DbgkpSetProcessDebugObject 访问 DebugPort 的地址为:%x\\n",str_index.DbgkpSetProcessDebugObject[2]));  

  break;  

}  

_bp++;  

} while (1);  

_bp=(BYTE*)addr_start;  

do  

{  

if((\*(_bp-6)==0xC6)&&(\*(_bp-2)==0xFF)&&(\*(_bp)==0x83)&&(\*(_bp+1)==0xBF)&&(\*(_bp+7)==0x74)&&(\*(_bp+9)==0xC7))  

{  

  str_index.DbgkpSetProcessDebugObject[3]=(ULONG)_bp+2;  

  KdPrint(("第四个 DbgkpSetProcessDebugObject 访问 DebugPort 的地址为:%x\\n",str_index.DbgkpSetProcessDebugObject[3]));  

  break;  

}  

_bp++;  

} while (1);  

_bp=(BYTE*)addr_start;  

do  

{  

if((\*(_bp-7)==0xE8)&&(\*(_bp-2)==0xEB)&&(\*(_bp)==0x83)&&(\*(_bp+1)==0xA7)&&(\*(_bp+7)==0xC7))  

{  

  str_index.DbgkpSetProcessDebugObject[4]=(ULONG)_bp+2;  

  KdPrint(("第五个 DbgkpSetProcessDebugObject 访问 DebugPort 的地址为:%x\\n",str_index.DbgkpSetProcessDebugObject[4]));  

  break;  

}  

_bp++;  

} while (1);  

_bp=(BYTE*)addr_start;  

do  

{  

if((\*(_bp-6)==0x74)&&(\*(_bp-4)==0x33)&&(\*(_bp-2)==0xEB)&&(\*(_bp)==0x8B)&&(\*(_bp+1)==0x81)&&(\*(_bp+6)==0x32)&&(\*(_bp+8)==0xEB))  

{  

  str_index.DbgkForwardException=(ULONG)_bp+2;  

  KdPrint(("DbgkForwardException 访问 DeubugPort 的地址为:%x\\n",str_index.DbgkForwardException));  

  break;  

}  

_bp++;  

} while (1);  

PAGED_Open();  

RtlCopyMemory((PVOID)str_index.KiDispatchException,debugPort_out,2);  

RtlCopyMemory((PVOID)str_index.NtQueryInformationProcess,debugPort_out,2);  

RtlCopyMemory((PVOID)str_index.PspCreateProcess,debugPort_out,2);  

RtlCopyMemory((PVOID)str_index.PsGetProcessDebugPort,debugPort_out,2);  

RtlCopyMemory((PVOID)str_index.PsIsProcessBeingDebugged,debugPort_out,2);  

RtlCopyMemory((PVOID)str_index.NtTerminateProcess,debugPort_out,2);  

RtlCopyMemory((PVOID)str_index.PspTerminateProcess,debugPort_out,2);  

RtlCopyMemory((PVOID)str_index.PspExitThread,debugPort_out,2);  

RtlCopyMemory((PVOID)str_index.PspProcessDelete[0],debugPort_out,2);  

RtlCopyMemory((PVOID)str_index.PspProcessDelete[1],debugPort_out,2);  

RtlCopyMemory((PVOID)str_index.PspProcessDelete[2],debugPort_out,2);  

RtlCopyMemory((PVOID)str_index.ObpCloseHandleTableEntry,debugPort_out,2);  

RtlCopyMemory((PVOID)str_index.ObpCloseHandle,debugPort_out,2);  

RtlCopyMemory((PVOID)str_index.MmCreatePeb,debugPort_out,2);  

RtlCopyMemory((PVOID)str_index.DbgkCreateThread,debugPort_out,2);  

RtlCopyMemory((PVOID)str_index.DbgkExitThread,debugPort_out,2);  

RtlCopyMemory((PVOID)str_index.DbgkExitProcess,debugPort_out,2);  

RtlCopyMemory((PVOID)str_index.DbgkMapViewOfSection,debugPort_out,2);  

RtlCopyMemory((PVOID)str_index.DbgkUnMapViewOfSection,debugPort_out,2);  

RtlCopyMemory((PVOID)str_index.DbgkpMarkProcessPeb,debugPort_out,2);  

RtlCopyMemory((PVOID)str_index.DbgkpCloseObject[0],debugPort_out,2);  

RtlCopyMemory((PVOID)str_index.DbgkpCloseObject[1],debugPort_out,2);  

RtlCopyMemory((PVOID)str_index.DbgkpCloseObject[2],debugPort_out,2);  

RtlCopyMemory((PVOID)str_index.DbgkCopyProcessDebugPort[0],debugPort_out,2);  

RtlCopyMemory((PVOID)str_index.DbgkCopyProcessDebugPort[1],debugPort_out,2);  

RtlCopyMemory((PVOID)str_index.DbgkCopyProcessDebugPort[2],debugPort_out,2);  

RtlCopyMemory((PVOID)str_index.DbgkCopyProcessDebugPort[3],debugPort_out,2);  

RtlCopyMemory((PVOID)str_index.DbgkOpenProcessDebugPort[0],debugPort_out,2);  

RtlCopyMemory((PVOID)str_index.DbgkOpenProcessDebugPort[1],debugPort_out,2);  

RtlCopyMemory((PVOID)str_index.DbgkpQueueMessage,debugPort_out,2);  

RtlCopyMemory((PVOID)str_index.DbgkClearProcessDebugObject[0],debugPort_out,2);  

RtlCopyMemory((PVOID)str_index.DbgkClearProcessDebugObject[1],debugPort_out,2);  

RtlCopyMemory((PVOID)str_index.DbgkpSetProcessDebugObject[0],debugPort_out,2);  

RtlCopyMemory((PVOID)str_index.DbgkpSetProcessDebugObject[1],debugPort_out,2);
```



```c
  RtlCopyMemory((PVOID)str_index.DbgkpSetProcessDebugObject[2],debugPort_out,2);  
  RtlCopyMemory((PVOID)str_index.DbgkpSetProcessDebugObject[3],debugPort_out,2);  
  RtlCopyMemory((PVOID)str_index.DbgkpSetProcessDebugObject[4],debugPort_out,2);  
  RtlCopyMemory((PVOID)str_index.DbgkForwardException,debugPort_out,2);  
  PAGED_Exit();  
}  
#pragma PAGEDCODE  
VOID Un_DebugPort()  
{  
  PAGED_Open();  
  RtlCopyMemory((PVOID)str_index.KiDispatchException,debugport_in,2);  
  RtlCopyMemory((PVOID)str_index.NtQueryInformationProcess,debugport_in,2);  
  RtlCopyMemory((PVOID)str_index.PspCreateProcess,debugport_in,2);  
  RtlCopyMemory((PVOID)str_index.PsGetProcessDebugPort,debugport_in,2);  
  RtlCopyMemory((PVOID)str_index.PsIsProcessBeingDebugged,debugport_in,2);  
  RtlCopyMemory((PVOID)str_index.NtTerminateProcess,debugport_in,2);  
  RtlCopyMemory((PVOID)str_index.PspTerminateProcess,debugport_in,2);  
  RtlCopyMemory((PVOID)str_index.PspExitThread,debugport_in,2);  
  RtlCopyMemory((PVOID)str_index.PspProcessDelete[0],debugport_in,2);  
  RtlCopyMemory((PVOID)str_index.PspProcessDelete[1],debugport_in,2);  
  RtlCopyMemory((PVOID)str_index.PspProcessDelete[2],debugport_in,2);  
  RtlCopyMemory((PVOID)str_index.ObpCloseHandleTableEntry,debugport_in,2);  
  RtlCopyMemory((PVOID)str_index.ObpCloseHandle,debugport_in,2);  
  RtlCopyMemory((PVOID)str_index.MmCreatePeb,debugport_in,2);  
  RtlCopyMemory((PVOID)str_index.DbgkCreateThread,debugport_in,2);  
  RtlCopyMemory((PVOID)str_index.DbgkExitThread,debugport_in,2);  
  RtlCopyMemory((PVOID)str_index.DbgkExitProcess,debugport_in,2);  
  RtlCopyMemory((PVOID)str_index.DbgkMapViewOfSection,debugport_in,2);  
  RtlCopyMemory((PVOID)str_index.DbgkUnMapViewOfSection,debugport_in,2);  
  RtlCopyMemory((PVOID)str_index.DbgkpMarkProcessPeb,debugport_in,2);  
  RtlCopyMemory((PVOID)str_index.DbgkpCloseObject[0],debugport_in,2);  
  RtlCopyMemory((PVOID)str_index.DbgkpCloseObject[1],debugport_in,2);  
  RtlCopyMemory((PVOID)str_index.DbgkpCloseObject[2],debugport_in,2);  
  RtlCopyMemory((PVOID)str_index.DbgkCopyProcessDebugPort[0],debugport_in,2);  
  RtlCopyMemory((PVOID)str_index.DbgkCopyProcessDebugPort[1],debugport_in,2);  
  RtlCopyMemory((PVOID)str_index.DbgkCopyProcessDebugPort[2],debugport_in,2);  
  RtlCopyMemory((PVOID)str_index.DbgkCopyProcessDebugPort[3],debugport_in,2);  
  RtlCopyMemory((PVOID)str_index.DbgkOpenProcessDebugPort[0],debugport_in,2);  
  RtlCopyMemory((PVOID)str_index.DbgkOpenProcessDebugPort[1],debugport_in,2);  
  RtlCopyMemory((PVOID)str_index.DbgkpQueueMessage,debugport_in,2);  
  RtlCopyMemory((PVOID)str_index.DbgkClearProcessDebugObject[0],debugport_in,2);  
  RtlCopyMemory((PVOID)str_index.DbgkClearProcessDebugObject[1],debugport_in,2);  
  RtlCopyMemory((PVOID)str_index.DbgkpSetProcessDebugObject[0],debugport_in,2);  
  RtlCopyMemory((PVOID)str_index.DbgkpSetProcessDebugObject[1],debugport_in,2);  
  RtlCopyMemory((PVOID)str_index.DbgkpSetProcessDebugObject[2],debugport_in,2);  
  RtlCopyMemory((PVOID)str_index.DbgkpSetProcessDebugObject[3],debugport_in,2);  
  RtlCopyMemory((PVOID)str_index.DbgkpSetProcessDebugObject[4],debugport_in,2);  
  RtlCopyMemory((PVOID)str_index.DbgkForwardException,debugport_in,2);  
  PAGED_Exit();  
}  
#pragma INITCODE  
VOID Hook_NtSetInformationThread()  
{  
  addr_NtSetInformationThread=(ULONG)Getsset_this(in_NtSetInformationThread);  
  addr1_NtSetInformationThread=(ULONG)Getssdt_addr(in_NtSetInformationThread);  
  _NtSetInformationThread=(NtSetInformationThread*)addr1_NtSetInformationThread;  
  PAGED_Open();  
  *((ULONG*)addr_NtSetInformationThread)=(ULONG)My_NtSetInformationThread;  
  PAGED_Exit();  
}  
#pragma PAGEDCODE  
extern "C"  
NTSTATUS __stdcall My_NtSetInformationThread  (__in HANDLE  ThreadHandle,  
                  __in THREADINFOCLASS  ThreadInformationClass,  
                  __in_bcount(ThreadInformationLength) PVOID  ThreadInformation,  
                  __in ULONG  ThreadInformationLength    
                     )  
{  
  if(ThreadInformationClass==0x11)  
  {  
    KdPrint(("发现 ANTI 行为!\r\n"));  
    KdPrint(("NtSetInformationThread 的参数:%8x %8x %8x %8x",ThreadHandle,ThreadInformationClass,ThreadInformation,ThreadInformationLength));  
    ThreadInformationClass=(THREADINFOCLASS)0xFF;  
    KdPrint(("NtSetInformationThread 的修改后的参数:%8x %8x %8x %8x",ThreadHandle,ThreadInformationClass,ThreadInformation,ThreadInformationLength));  
    return STATUS_SUCCESS;  
  }  
  return ((NtSetInformationThread)_NtSetInformationThread)(ThreadHandle,ThreadInformationClass,ThreadInformation,ThreadInformationLength);  
}  
#pragma PAGEDCODE  
VOID Un_NtSetInnformationThread()  
{  
  PAGED_Open();  
  *((ULONG*)addr_NtSetInformationThread)=addr1_NtSetInformationThread;  
  PAGED_Exit();  
}  
#endif
```
