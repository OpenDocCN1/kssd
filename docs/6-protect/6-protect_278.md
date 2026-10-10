

# 围观一下 HS

> 原文：[`www.pediy.com/kssd/pediy12/142823.html`](https://www.pediy.com/kssd/pediy12/142823.html)

//---------------------------------------发第二次了，再发不出去就不发了.  
![](img/906f120cfe711ccbc8463fd6eb86eee9.jpg "尴尬") 只贴部分代码，大多都是小技巧没什么特别的地方。  
//------------------xxxx.h  
#define PAGEDCODE code\_seg("PAGED")  
#define LOCKEDCODE code\_seg()  
#define INITCODE code\_seg("INIT")  
  
#define PAGEDDATA data\_seg("PAGED")  
#define LOCKEDDATA data\_seg()  
#define INITDATA data\_seg("INIT")  
  
typedef struct \_DEVICE\_CONTEXTSION  
{  
  UNICODE\_STRING driver\_name;  
  UNICODE\_STRING symbol\_name;  
  PDEVICE\_OBJECT device\_object;  
};  
  
  
VOID DriverUnLoad(PDRIVER\_OBJECT driver);  
NTSTATUS DispatchIrp(PDEVICE\_OBJECT driver,PIRP irp);  
//----------------------------------------------  
VOID LoadDriver\_GetVersion(PDRIVER\_OBJECT driver);  
VOID Get\_NtResumeThread();  
VOID Hook\_PsSuspendThread();  
VOID My\_PsSuspendThread();  
VOID Un\_PsSuspendThread();  
VOID Enum\_Thread(BOOL b\_thread=FALSE);   //TRUE 表示暂停线程,FALSE 表示结束线程  
BOOL Enum\_Module(ULONG Thread\_StartAddr);  
//--守护 OD 和 CE--------------------------  
VOID Hook\_ObOpenObjectByPointer();  
VOID Un\_ObOpenObjectByPointer();  
VOID SSDT\_NtQuerySystemInformation();  
VOID Un\_NtQuerySystemInformation();  
//------------------------------------  
VOID Hook\_NtOpenProcess();  
VOID My\_NtOpenProecss();  
VOID Un\_NtOpenProecss();  
VOID Hook\_NtReadVirtualMemory();  
VOID My\_NtReadVirtualMemory();  
VOID Un\_NtReadVirtualMemory();  
VOID Hook\_NtWriteVirtualMemory();  
VOID My\_NtWriteVirtualMemory();  
VOID Un\_NtWriteVirtualMemory();  
VOID Hook\_NtClose();  
VOID My\_NtClose();  
VOID Un\_NtClocse();  
VOID Hook\_NtDeviceIoControlFile();  
VOID My\_NtDeviceIoControlFile();  
VOID Unnt\_NtDeviceIoControlFile();  
VOID Hook\_KiAttachProcess();  
VOID My\_KiAttachProcess();  
VOID Unnt\_KiAttachProecss();  
VOID Hook\_IopXxxControlFile();  
VOID My\_IopXxxControlFile();  
VOID Un\_IopXxxControlFile();  
VOID Hook\_NtGetContextThread();  
VOID My\_NtGetContextThread();  
VOID Un\_NtGetContextThread();  
VOID Hook\_NtSetContextThread();  
VOID My\_NtSetContextThread();  
VOID Un\_NtSetContextThread();  
VOID Hook\_NtGetContextThread1();  
VOID My\_NtGetContextThread1();  
VOID Un\_NtGetContextThread1();  
VOID Hook\_NtQueryPerformanceCounter();  
VOID My\_NtQueryPerformanceCounter\_1();  
VOID My\_NtQueryPerformanceCounter\_2();  
VOID Un\_NtQueryPerformanceCounter();  
VOID Hook\_DebugPort();  
VOID Un\_DebugPort();  
VOID Hook\_NtSetInformationThread();  
extern "C"  
NTSTATUS \_\_stdcall My\_NtSetInformationThread  ( \_\_in HANDLE  ThreadHandle,    
                         \_\_in THREADINFOCLASS  ThreadInformationClass,    
                         \_\_in\_bcount(ThreadInformationLength) PVOID  ThreadInformation,    
                         \_\_in ULONG  ThreadInformationLength     
                           ) ;  
VOID Un\_NtSetInnformationThread();  
#endif  
//--------------------------------xxx.cpp  
#pragma INITCODE  
extern "C"  
NTSTATUS DriverEntry(PDRIVER\_OBJECT driver,PUNICODE\_STRING driver\_object\_name)  
{  
      /\*  
1.绕过 PsSuspendThread  
2.枚举 HS 线程暂停  
3.SSDT HOOK 守护进程  
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
5.绕 Shadow SSDT 函数 HOOK  
     1.HOOK KiFastCallEntry 绕 SSDT HOOK  
   2.绕 NtGdiGetPixel  
   3.Hook Shadow SSDT 守护窗口  
6.(\_EPROECSS+bc)DebugPort 的处理  
7.HOOK NtSetInformationThread (HideFromDebugPort)  0x11 号功能  
8.SSDT HOOK 隐藏进程和模块  
  \*/  
  //------------------------------------------------------------  
  LoadDriver\_GetVersion(driver);  
  Get\_NtResumeThread();  
  Hook\_PsSuspendThread();    
  Enum\_Thread(TRUE);  
  Hook\_NtOpenProcess();  
  Hook\_NtReadVirtualMemory();  
  Hook\_NtWriteVirtualMemory();  
  Hook\_NtClose();  
  Hook\_NtDeviceIoControlFile();  
  Hook\_KiAttachProcess();  
  Hook\_IopXxxControlFile();  
  Hook\_NtGetContextThread();  
  Hook\_NtSetContextThread();  
  Hook\_NtGetContextThread1();  
  Hook\_NtQueryPerformanceCounter();  
  Hook\_DebugPort();  
  Hook\_NtSetInformationThread();  
//  Hook\_ObOpenObjectByPointer();  
  SSDT\_NtQuerySystemInformation();  
  //-------------------------------------------------------------  
  NTSTATUS status=STATUS\_SUCCESS;  
  driver->MajorFunction\[IRP\_MJ\_CREATE\]=DispatchIrp;  
  driver->MajorFunction\[IRP\_MJ\_READ\]=DispatchIrp;  
  driver->MajorFunction\[IRP\_MJ\_WRITE\]=DispatchIrp;  
  driver->MajorFunction\[IRP\_MJ\_CLOSE\]=DispatchIrp;  
  driver->MajorFunction\[IRP\_MJ\_DEVICE\_CONTROL\]=DispatchIrp;  
  driver->DriverUnload=DriverUnLoad;  
    
  UNICODE\_STRING driver\_name;  
  RtlInitUnicodeString(&driver\_name,\_Driver\_name);  
    
  PDEVICE\_OBJECT device\_object;  
  
  status=IoCreateDevice(driver,NULL,&driver\_name,FILE\_DEVICE\_UNKNOWN,NULL,FALSE,&device\_object);  
  if(!NT\_SUCCESS(status))  
  {  
    KdPrint(("创建设备失败!\\n"));  
    return status;  
  }  
  KdPrint(("创建设备成功!\\n"));  
  
  device\_object->Flags |= DO\_BUFFERED\_IO;  
    
  UNICODE\_STRING symbolic\_name;  
  RtlInitUnicodeString(&symbolic\_name,\_Symbolic\_name);  
  status=IoCreateSymbolicLink(&symbolic\_name,&driver\_name);  
  if(!NT\_SUCCESS(status))  
  {  
    IoDeleteDevice(device\_object);  
    KdPrint(("创建符号链接失败!\\n"));  
    return status;  
  }  
  KdPrint(("创建符号链接成功!\\n"));  
  return STATUS\_SUCCESS;  
}  
  
#pragma PAGEDCODE  
VOID DriverUnLoad(PDRIVER\_OBJECT driver)  
{  
  UNICODE\_STRING symbol\_name;  
  RtlInitUnicodeString(&symbol\_name,\_Symbolic\_name);  
  IoDeleteSymbolicLink(&symbol\_name);  
  IoDeleteDevice(driver->DeviceObject);  
  KdPrint(("删除设备成功!\\n"));  
  //---------------------------------  
  Un\_PsSuspendThread();  
  Un\_NtOpenProecss();  
  Un\_NtReadVirtualMemory();  
  Un\_NtWriteVirtualMemory();  
  Un\_NtClocse();  
  Unnt\_NtDeviceIoControlFile();  
  Unnt\_KiAttachProecss();  
  Un\_IopXxxControlFile();  
  Un\_NtGetContextThread();  
  Un\_NtSetContextThread();  
  Un\_NtGetContextThread1();  
  Un\_NtQueryPerformanceCounter();  
  Un\_DebugPort();  
  Un\_NtSetInnformationThread();  
//  Un\_ObOpenObjectByPointer();  
  Un\_NtQuerySystemInformation();  
  Enum\_Thread(FALSE);  
}  
  
#pragma PAGEDCODE  
NTSTATUS DispatchIrp(PDEVICE\_OBJECT driver,PIRP irp)  
{  
  irp->IoStatus.Status=STATUS\_SUCCESS;  
  irp->IoStatus.Information=0;  
  IoCompleteRequest(irp,IO\_NO\_INCREMENT);  
  KdPrint(("进入 IRP 例程!\\n"));  
  return STATUS\_SUCCESS;  
}  
//--------------------------------xxxx\_struct.h  
struct \_SYSTEM\_PROCESSOR\_TIMES//处理器时间结构  
{  
  LARGE\_INTEGER IdleTime;  
  LARGE\_INTEGER KernelTime;  
  LARGE\_INTEGER UserTime;;  
  LARGE\_INTEGER DpcTime;  
  LARGE\_INTEGER InterruptTime;  
  ULONG InterruptCount;  
};  
  
typedef struct \_SYSTEM\_THREADS{  
    LARGE\_INTEGER KernelTime;  
    LARGE\_INTEGER UserTime;  
    LARGE\_INTEGER CreateTime;  
    ULONG WaitTime;  
    PVOID StartAddress;  
    CLIENT\_ID ClientId;  
    KPRIORITY Priority;  
    LONG BasePriority;  
    ULONG ContextSwitches;  
    ULONG ThreadState;  
    ULONG WaitReason;  
    ULONG Reversed;//add by achillis  
} SYSTEM\_THREAD\_INFORMATION,\*PSYSTEM\_THREADS;  
  
typedef struct \_SYSTEM\_PROCESSES{  
  ULONG NextEntryDelta;  
  ULONG ThreadCount;  
  LARGE\_INTEGER Reserved1\[3\];  
  LARGE\_INTEGER CreateTime;  
  LARGE\_INTEGER UserTime;  
  LARGE\_INTEGER KernelTime;  
  UNICODE\_STRING ProcessName;  
  KPRIORITY BasePriority;  
  ULONG ProcessId;  
  ULONG InheritedFromProcessId;  
  ULONG HandleCount;  
  ULONG SessionId;  
  ULONG\_PTR PageDirectoryBase;  
  VM\_COUNTERS VmCounters;  
  ULONG PrivatePageCount;// add by achillis  
  IO\_COUNTERS IoCounters;  
  \_SYSTEM\_THREADS Threads\[1\];  
} SYSTEM\_PROCESSES, \*PSYSTEM\_PROCESSES;  
  
typedef struct \_SYSTEM\_MODULE\_INFORMATION\_ENTRY  
{  
  ULONG Unknow1;  
  ULONG Unknow2;  
#ifdef \_WIN64  
  ULONG Unknow3;  
  ULONG Unknow4:  
#endif  
  PVOID Base;  
  ULONG Size;  
  ULONG Flags;  
  USHORT Index;  
  USHORT NameLength;  
  USHORT LoadCount;  
  USHORT ModuleNameOffset;  
  char ImageName\[256\];  
}SYSTEM\_MODULE\_INFORMATION\_ENTRY, \*PSYSTEM\_MODULE\_INFORMATION\_ENTRY;  
  
typedef struct \_SYSTEM\_MODULE\_INFORMATION  
{  
  ULONG Count;//内核中以加载的模块的个数   
  SYSTEM\_MODULE\_INFORMATION\_ENTRY Module\[1\];  
}SYSTEM\_MODULE\_INFORMATION, \*PSYSTEM\_MODULE\_INFORMATION;  
  
typedef enum \_SYSTEM\_INFORMATION\_CLASS {  
    SystemBasicInformation, // 0 Y N  
    SystemProcessorInformation, // 1 Y N  
    SystemPerformanceInformation, // 2 Y N  
    SystemTimeOfDayInformation, // 3 Y N  
    SystemNotImplemented1, // 4 Y N  
    SystemProcessesAndThreadsInformation, // 5 Y N  
    SystemCallCounts, // 6 Y N  
    SystemConfigurationInformation, // 7 Y N  
    SystemProcessorTimes, // 8 Y N  
    SystemGlobalFlag, // 9 Y Y  
    SystemNotImplemented2, // 10 Y N  
    SystemModuleInformation, // 11 Y N  
    SystemLockInformation, // 12 Y N  
    SystemNotImplemented3, // 13 Y N  
    SystemNotImplemented4, // 14 Y N  
    SystemNotImplemented5, // 15 Y N  
    SystemHandleInformation, // 16 Y N  
    SystemObjectInformation, // 17 Y N  
    SystemPagefileInformation, // 18 Y N  
    SystemInstructionEmulationCounts, // 19 Y N  
    SystemInvalidInfoClass1, // 20  
    SystemCacheInformation, // 21 Y Y  
    SystemPoolTagInformation, // 22 Y N  
    SystemProcessorStatistics, // 23 Y N  
    SystemDpcInformation, // 24 Y Y  
    SystemNotImplemented6, // 25 Y N  
    SystemLoadImage, // 26 N Y  
    SystemUnloadImage, // 27 N Y  
    SystemTimeAdjustment, // 28 Y Y  
    SystemNotImplemented7, // 29 Y N  
    SystemNotImplemented8, // 30 Y N  
    SystemNotImplemented9, // 31 Y N  
    SystemCrashDumpInformation, // 32 Y N  
    SystemExceptionInformation, // 33 Y N  
    SystemCrashDumpStateInformation, // 34 Y Y/N  
    SystemKernelDebuggerInformation, // 35 Y N  
    SystemContextSwitchInformation, // 36 Y N  
    SystemRegistryQuotaInformation, // 37 Y Y  
    SystemLoadAndCallImage, // 38 N Y  
    SystemPrioritySeparation, // 39 N Y  
    SystemNotImplemented10, // 40 Y N  
    SystemNotImplemented11, // 41 Y N  
    SystemInvalidInfoClass2, // 42  
    SystemInvalidInfoClass3, // 43  
    SystemTimeZoneInformation, // 44 Y N  
    SystemLookasideInformation, // 45 Y N  
    SystemSetTimeSlipEvent, // 46 N Y  
    SystemCreateSession, // 47 N Y  
    SystemDeleteSession, // 48 N Y  
    SystemInvalidInfoClass4, // 49  
    SystemRangeStartInformation, // 50 Y N  
    SystemVerifierInformation, // 51 Y Y  
    SystemAddVerifier, // 52 N Y  
    SystemSessionProcessesInformation // 53 Y N  
}SYSTEM\_INFORMATION\_CLASS, \*PSYSTEM\_INFORMATION\_CLASS;  
  
extern "C"  
NTSYSAPI NTSTATUS NTAPI ZwQuerySystemInformation  (\_\_in SYSTEM\_INFORMATION\_CLASS  SystemInformationClass,    
                           \_\_out\_bcount\_opt(SystemInformationLength) PVOID  SystemInformation,    
                           \_\_in ULONG  SystemInformationLength,    
                           \_\_out\_opt PULONG  ReturnLength     
                           );  
extern "C"  
NTKERNELAPI NTSTATUS PsLookupThreadByThreadId  (\_\_in HANDLE  ThreadId,    
                        \_\_deref\_out PETHREAD \*  Thread     
                        );  
extern "C"  
typedef  
NTSTATUS \_\_stdcall PsSuspendThread ( IN PETHREAD  Thread,    
              OUT PULONG PreviousSuspendCount  OPTIONAL  
               ) ;  
extern "C"  
typedef  
NTSTATUS \_\_stdcall PsResumeThread  ( IN PETHREAD  Thread,    
              OUT PULONG PreviousSuspendCount  OPTIONAL     
              ) ;  
extern "C"  
typedef  
NTSYSCALLAPI NTSTATUS (\_\_stdcall\* NtSetInformationThread)  ( \_\_in HANDLE  ThreadHandle,    
                           \_\_in THREADINFOCLASS  ThreadInformationClass,    
                           \_\_in\_bcount(ThreadInformationLength) PVOID  ThreadInformation,    
                           \_\_in ULONG  ThreadInformationLength     
                           ) ;  
extern "C"  
typedef  
NTKERNELAPI NTSTATUS \_\_stdcall ObOpenObjectByPointer  ( \_\_in PVOID  Object,    
                       \_\_in ULONG  HandleAttributes,    
                       \_\_in\_opt PACCESS\_STATE  PassedAccessState,    
                       \_\_in ACCESS\_MASK  DesiredAccess,    
                       \_\_in\_opt POBJECT\_TYPE  ObjectType,    
                       \_\_in KPROCESSOR\_MODE  AccessMode,    
                       \_\_out PHANDLE  Handle     
                       ) ;  
extern "C"  
typedef  
NTSYSCALLAPI NTSTATUS NTAPI NtQuerySystemInformation  ( \_\_in SYSTEM\_INFORMATION\_CLASS  SystemInformationClass,    
                             \_\_out\_bcount\_opt(SystemInformationLength) PVOID  SystemInformation,    
                             \_\_in ULONG  SystemInformationLength,    
                             \_\_out\_opt PULONG  ReturnLength     
                             );  
  
#endif  
  
//------------------------------------------xxxx\_ssdt.h  
#define \_windows\_xp  51  
  
#define \_Game\_Name "OD.exe"  
#define \_Game\_Name1 "OllyDBG.EXE"  
#define \_Game\_Name2 "OllyICE.exe"  
#define \_Game\_Name3 "Cheat Engine.exe"  
#define \_Game\_Name4 "Cheat Engine.ex"  
#define \_Game\_Name5 "Cheat Engine"  
#define \_UGame\_Name  L"OD.exe"  
#define \_UGame\_Name1 L"OllyDBG.EXE"  
#define \_UGame\_Name2 L"OllyICE.exe"  
#define \_UGame\_Name3 L"Cheat Engine.exe"  
#define \_UGame\_Name4 L"Cheat Engine.ex"  
#define \_UGame\_Name5 L"Cheat Engine"  
  
PsSuspendThread\* \_PsSuspendThread;  
PsResumeThread\* \_PsResumeThread;  
NtSetInformationThread\* \_NtSetInformationThread;  
//-------------------------------------------------  
int in\_NtResumeThread=0;      //保存 NtResumeThread 的序号  
int in\_NtPsSuspendThread=0;   //保存 NtSuspendThread 的序号  
int in\_NtOpenProcess=0;       //保存 NtOpenProcess 的序号  
int in\_NtReadVirtualMemory=0; //保存 NtReadVirtualMemory 的序号  
int in\_NtWriteVirtualMemory=0;//保存 NtWriteVirtualMemory 的序号  
int in\_NtClose=0;             //保存 NtClose 的序号  
int in\_NtDeviceIoControlFile=0;//保存 NtDeviceIoControlFile 的序号  
int in\_NtFsControlFile=0;     //保存 NtFsControlFile 的序号  
int in\_NtGetContextThread=0;  //保存 NtGetContextThread 的序号  
int in\_NtTerminateThread=0;  //保存 NtTerminateThread 的序号  
int in\_NtSetContextThread=0;  //保存 NtSetContextThread 的序号  
int in\_NtQueryPerformanceCounter=0; //保存 NtQueryPerformanceCounter 的序号  
int in\_NtSetInformationThread=0; //保存 NtSetInformationThread 的序号  
int in\_NtQuerySystemInformation=0; //保存 NtQuerySystemInformation 的序号  
//-------------------------------------------------  
ULONG addr\_ObWatchHandles;    //保存 ObWatchHandles 的地址  
ULONG addr\_SEH\_prolog;        //保存\_SEH\_prolog 的地址   
//-------------------------------------------------  
ULONG addr\_NtOpenThread;         //保存 NtOpenThread 的地址  
ULONG u\_PsSuspendThread;         //保存 PsSuspendTHread 的地址  
ULONG push\_addr\_PsSuspendThread;    
ULONG call\_addr\_PsSuspendThread;    
ULONG addr\_NtOpenProcess;        //保存 NtOpenProcess 的地址  
ULONG push\_addr\_NtOpenProcess;  
ULONG call\_addr\_NtOpenProecss;  
ULONG addr\_NtReadVirtualMemory;  //保存 NtReadVirtualMemory 的地址  
ULONG push\_addr\_NtReadVirtualMemory;  
ULONG call\_addr\_NtReadVirtualMemory;  
ULONG addr\_NtWriteVirtualMemory; //保存 NtWriteVirtualMemory 的地址  
ULONG push\_addr\_NtWriteVirtualMemory;  
ULONG call\_addr\_NtWriteVirtualMemory;  
ULONG addr\_NtClose;              //保存 NtClose 的地址  
ULONG call\_addr\_NtClose;  
ULONG addr\_NtDeviceIoControlFile;//保存 NtDeviceIoControlFile 的地址  
ULONG call\_addr\_NtDeviceIoControlFile;  
ULONG addr\_KiAttachProcess;      //保存 KiAttachProecss 的地址  
ULONG call\_addr\_KiAttachProcess;  
ULONG addr\_IopXxxControlFile;    //保存 IopXxxControlFile 的地址  
ULONG call\_addr\_IopXxxControlFile;  
ULONG addr\_NtGetContextThread;   //保存 NtGetContextThread 的地址  
ULONG call\_addr\_NtGetContextThread;    
ULONG addr\_NtSetContextThread;   //保存 NtSetContextThread 的地址  
ULONG call\_addr\_NtSetContextThread;  
ULONG addr\_NtGetContextThread1;  //保存 NtGetContextThread 的第二处 HOOK  
ULONG call\_addr\_NtGetContextThread1;  
ULONG Game\_call\_addr\_1;          //获取 NtGetContextThread 的第一个 call 地址  
ULONG Game\_call\_addr\_2;          //获取 NtGetContextThread 的第二个 call 地址  
ULONG addr\_NtQueryPerformanceCounter; //保存 NtQueryPerformanceCounter 的第一个跳转地址  
ULONG addr1\_NtQueryPerformanceCounter;//保存 NtQueryPerformanceCounter 的第二个跳转地址  
ULONG call\_addr\_NtQueryPerformanceCounter;  
ULONG addr\_NtSetInformationThread;    //保存 NtSetInformationThread 的地址  
ULONG addr1\_NtSetInformationThread;  
//-------------------------------------------------  
typedef struct \_ServiceDescriptorTable {  
  PVOID ServiceTableBase; //System Service Dispatch Table 的基地址    
  PVOID ServiceCounterTable;  
  //包含着 SSDT 中每个服务被调用次数的计数器。这个计数器一般由 sysenter 更新。   
  unsigned int NumberOfServices;//由 ServiceTableBase 描述的服务的数目。    
  PVOID ParamTableBase; //包含每个系统服务参数字节数表的基地址-系统服务参数表   
}\*PServiceDescriptorTable;   
extern "C" PServiceDescriptorTable KeServiceDescriptorTable;  
  
#pragma pack(1)  
typedef struct \_JMPCODE  
{  
  BYTE E9;  
  ULONG Fun\_addr;  
  BYTE nop\[2\];  
}jmp\_code,\*pjmp\_code;  
#pragma pack()  
  
//-----------获取 ssdt 地址  
#pragma PAGEDCODE  
ULONG Getssdt\_addr(int index)  
{  
  ULONG\* xfun\_addr,x\_serviceDescriptor,x\_addr;  
  x\_serviceDescriptor=(ULONG)KeServiceDescriptorTable->ServiceTableBase;  
  KdPrint(("当前 ServiceDescriptorTable 的地址为:%x\\n",x\_serviceDescriptor));  
  xfun\_addr=(PULONG)(x\_serviceDescriptor+index\*4);  
  x\_addr=\*xfun\_addr;  
  KdPrint(("当前 Index=%d 函数地址为:%x\\n",index,x\_addr));  
  return x\_addr;  
}  
//-----------------获取 ssdt 函数列表  
#pragma PAGEDCODE   
ULONG\* Getsset\_this(int index)  
{  
  ULONG\* xfun\_addr,x\_servicedescriptor;  
  x\_servicedescriptor=(ULONG )KeServiceDescriptorTable->ServiceTableBase;  
  KdPrint(("当前 ServiceDescriptorTable 的地址为:%x\\n",x\_servicedescriptor));  
  xfun\_addr=(PULONG)(x\_servicedescriptor+index\*4);  
  return xfun\_addr;  
}  
  
KIRQL kirql;  
#pragma PAGEDCODE  
VOID PAGED\_Open()  
{  
  \_\_asm  
  {  
    cli  
    push eax  
    mov eax,cr0  
    and eax,not 10000h  
    mov cr0,eax  
    pop eax  
  }  
  kirql=KeRaiseIrqlToDpcLevel();  
}  
  
#pragma PAGEDCODE  
VOID PAGED\_Exit()  
{  
  KeLowerIrql(kirql);  
  \_\_asm  
  {  
    push eax  
    mov eax,cr0  
    or eax,10000h  
    mov cr0,eax  
    pop eax  
    sti  
  }  
}  
  
#pragma PAGEDCODE  
VOID LoadDriver\_GetVersion(PDRIVER\_OBJECT driver)  
{  
  ULONG Majorversion,Minorversion,buildVersion;  
  PsGetVersion(&Majorversion,&Minorversion,&buildVersion,NULL);  
  DWORD dw\_version=Majorversion\*10+Minorversion;  
  switch(dw\_version)  
  {  
  case \_windows\_xp:  
    KdPrint(("当前操作系统 windows xp sp3!\\n"));  
    in\_NtPsSuspendThread=0xFE;              //函数序号  
    in\_NtResumeThread=0xCE;  
    in\_NtOpenProcess=0x7A;  
    in\_NtReadVirtualMemory=0xBA;  
    in\_NtWriteVirtualMemory=0x115;  
    in\_NtClose=0x19;  
    in\_NtDeviceIoControlFile=0x42;  
    in\_NtFsControlFile=0x54;  
    in\_NtGetContextThread=0x55;  
    in\_NtTerminateThread=0x102;  
    in\_NtSetContextThread=0xD5;  
    in\_NtQueryPerformanceCounter=0xA5;  
    in\_NtSetInformationThread=0xE5;  
    in\_NtQuerySystemInformation=0xAD;  
    break;  
  default:  
    driver->DriverUnload=DriverUnLoad;  
    break;  
  }  
}  
  
#pragma PAGEDCODE  
VOID Get\_NtResumeThread()  
{  
  ULONG addr\_NtResumeThread;  
  BYTE\* \_bp;  
  \_bp=(BYTE\*)Getssdt\_addr(in\_NtResumeThread);  
  do   
  {  
    if((\*(\_bp-4)==0x50)&&(\*(\_bp-3)==0xFF)&&(\*(\_bp)==0xE8)&&(\*(\_bp+5)==0x8B)&&(\*(\_bp+8)==0xE8))  
    {  
      break;  
    }  
    \_bp++;  
  } while (1);  
  addr\_NtResumeThread=(ULONG)\_bp;  
  \_\_asm  
  {  
    push eax  
    push ebx  
    mov eax,addr\_NtResumeThread  
    add eax,1  
    mov ebx,\[eax\]  
    add eax,ebx  
    add eax,5  
    sub eax,1  
    mov addr\_NtResumeThread,eax  
    pop ebx  
    pop eax  
  }  
  \_PsResumeThread=(PsResumeThread\*)addr\_NtResumeThread;  
  KdPrint(("PsResumeThread 的地址为:%x\\n",addr\_NtResumeThread));   
}  
  
jmp\_code \_jmp\_code\_PsSuspendThread;  
pjmp\_code \_pjmp\_code\_PsSuspendThread;  
#pragma INITCODE  
VOID Hook\_PsSuspendThread()  
{  
  ULONG u\_NtSuspendThread;  
  BYTE\* \_bp;    //特征码搜索  
  \_bp=(BYTE\*)Getssdt\_addr(in\_NtPsSuspendThread);  
  do  
  {  
    if((\*(\_bp-9)==0x7c)&&(\*(\_bp-4)==0x50)&&(\*(\_bp-3)==0xFF)&&(\*(\_bp)==0xE8)&&(\*(\_bp+5)==0x8B)&&(\*(\_bp+7)==0x8B))  
    {  
      break;  
    }  
    \_bp++;  
  }while(1);  
  u\_NtSuspendThread=(ULONG)\_bp;  
  \_\_asm  
  {  
    push eax  
    mov eax,u\_NtSuspendThread  
    add eax,1  
    mov ebx,\[eax\]  
    add eax,ebx  
    add eax,5  
    sub eax,1  
    mov u\_NtSuspendThread,eax  
    pop eax  
  }  
  \_PsSuspendThread=(PsSuspendThread\*)u\_NtSuspendThread;  
  KdPrint(("PsSuspendThread 的地址为:%x\\n",u\_NtSuspendThread));  
  u\_PsSuspendThread=u\_NtSuspendThread+2;  
  KdPrint(("向 PsSuspendThread 写跳转的地址为:%x\\n",u\_PsSuspendThread));  
  ULONG u\_addr\_PsSuspendThread;  
  UNICODE\_STRING push\_addr;  
  RtlInitUnicodeString(&push\_addr,L"NtOpenProcess");  
  push\_addr\_PsSuspendThread=(ULONG)MmGetSystemRoutineAddress(&push\_addr);  
  \_\_asm  
  {  
    push eax  
    mov eax,push\_addr\_PsSuspendThread  
    add eax,6  
    mov eax,\[eax\]  
    add eax,0x28  
    sub eax,0x284  
    mov u\_addr\_PsSuspendThread,eax  
    pop eax  
  }  
  addr\_ObWatchHandles=u\_addr\_PsSuspendThread;  
  KdPrint(("ObWatchHandles 的地址为:%x\\n",u\_addr\_PsSuspendThread));  
  push\_addr\_PsSuspendThread=u\_addr\_PsSuspendThread+0x784;  
  KdPrint(("PsSuspendThread 的 push 地址为:%x\\n",push\_addr\_PsSuspendThread));  
  ULONG u\_call\_PsSuspendThread;  
  UNICODE\_STRING call\_addr;  
  RtlInitUnicodeString(&call\_addr,L"NtOpenThread");  
  call\_addr\_PsSuspendThread=(ULONG)MmGetSystemRoutineAddress(&call\_addr);  
  addr\_NtOpenThread=call\_addr\_PsSuspendThread;  
  \_\_asm  
  {  
    push eax  
    push ebx  
    mov eax,call\_addr\_PsSuspendThread  
    add eax,0xA  
    add eax,1  
    mov ebx,\[eax\]  
    add eax,ebx  
    add eax,5  
    sub eax,1  
    mov u\_call\_PsSuspendThread,eax  
    pop ebx  
    pop eax  
  }  
  addr\_SEH\_prolog=u\_call\_PsSuspendThread;  
  call\_addr\_PsSuspendThread=u\_call\_PsSuspendThread;  
  KdPrint(("PsSuspendThread 的 call 地址为:%x\\n",call\_addr\_PsSuspendThread));  
  \_pjmp\_code\_PsSuspendThread=(pjmp\_code)u\_PsSuspendThread;  
  \_jmp\_code\_PsSuspendThread.E9=\_pjmp\_code\_PsSuspendThread->E9;  
  \_jmp\_code\_PsSuspendThread.Fun\_addr=\_pjmp\_code\_PsSuspendThread->Fun\_addr;  
  PAGED\_Open();  
  \_pjmp\_code\_PsSuspendThread->E9=0xE9;  
  \_pjmp\_code\_PsSuspendThread->Fun\_addr=(ULONG)((ULONG)My\_PsSuspendThread-u\_PsSuspendThread-5);  
  PAGED\_Exit();  
}  
  
#pragma PAGEDCODE  
VOID \_\_declspec(naked) My\_PsSuspendThread()  
{  
  \_\_asm  
  {  
    push push\_addr\_PsSuspendThread  
    mov eax,u\_PsSuspendThread  
    add eax,0xA  
    push eax  
    mov eax,call\_addr\_PsSuspendThread  
    jmp eax  
  }  
}  
  
#pragma PAGEDCODE  
VOID Un\_PsSuspendThread()  
{  
  PAGED\_Open();  
  \_pjmp\_code\_PsSuspendThread->E9=\_jmp\_code\_PsSuspendThread.E9;  
  \_pjmp\_code\_PsSuspendThread->Fun\_addr=\_jmp\_code\_PsSuspendThread.Fun\_addr;  
  PAGED\_Exit();  
}  
  
#pragma PAGEDCODE  
BOOL Enum\_Module(ULONG Thread\_StartAddr)  
{  
  NTSTATUS status;  
  PCHAR ModuleNames;  
  ULONG len;  
  PVOID pbuffer=NULL;  
  PSYSTEM\_MODULE\_INFORMATION Module\_Enum;  
  status=ZwQuerySystemInformation((SYSTEM\_INFORMATION\_CLASS)11,NULL,0,&len);  
  pbuffer=ExAllocatePool(NonPagedPool,len);  
  if(pbuffer==NULL)  
  {  
    KdPrint(("分配内存失败!\\r\\n"));  
    return FALSE;  
  }  
  status=ZwQuerySystemInformation((SYSTEM\_INFORMATION\_CLASS)11,pbuffer,len,NULL);  
  Module\_Enum=(PSYSTEM\_MODULE\_INFORMATION)pbuffer;  
  for(unsigned int i=0;i<Module\_Enum->Count;i++)  
  {  
    if(ModuleNames=strstr(Module\_Enum->Module\[i\].ImageName,"EagleXNt.sys"))  
    {  
      ULONG Module\_Min=(ULONG)Module\_Enum->Module\[i\].Base;  
      ULONG Module\_Max=Module\_Min+Module\_Enum->Module\[i\].Size;  
      if((Module\_Min<=Thread\_StartAddr)&&(Thread\_StartAddr<=Module\_Max))  
      {  
        return TRUE;  
      }  
    }  
  }  
  ExFreePool(pbuffer);  
  return FALSE;  
}  
  
#pragma PAGEDCODE  
VOID Enum\_Thread(BOOL b\_thread)  
{  
  NTSTATUS status;  
  PVOID pbuffer=NULL;  
  ULONG len;  
  PSYSTEM\_PROCESSES Process\_Enum;  
  PSYSTEM\_THREADS Thread\_Enum;  
  status=ZwQuerySystemInformation((SYSTEM\_INFORMATION\_CLASS)5,NULL,0,&len);  
  KdPrint(("枚举进程时分配的内存大小:%d\\n",len));  
  pbuffer=ExAllocatePool(NonPagedPool,len);  
  if(pbuffer==NULL)  
  {  
    KdPrint(("分配内存失败!\\r\\n"));  
    return ;  
  }  
  status=ZwQuerySystemInformation((SYSTEM\_INFORMATION\_CLASS)5,pbuffer,len,NULL);  
  Process\_Enum=(PSYSTEM\_PROCESSES)pbuffer;  
  do  
  {  
    if(Process\_Enum->ThreadCount&&Process\_Enum->ProcessId)  
    {  
      Thread\_Enum=(PSYSTEM\_THREADS)Process\_Enum->Threads;  
      for(unsigned int i=0;i<Process\_Enum->ThreadCount;i++)  
      {  
        BOOL bl\_modlule=FALSE;  
        bl\_modlule=Enum\_Module((ULONG)Thread\_Enum->StartAddress);  
        if(bl\_modlule==TRUE)  
        {  
          PETHREAD thread\_temp;  
          PsLookupThreadByThreadId(Thread\_Enum->ClientId.UniqueThread,&thread\_temp);  
          if(b\_thread==TRUE)  
          {  
            \_PsSuspendThread(thread\_temp,NULL);  
            KdPrint(("暂停<->线程信息:Thread\[%d\]  StartAddr=%08x   TID:%d \\r\\n",i+1,Thread\_Enum->StartAddress,Thread\_Enum->ClientId.UniqueThread));  
          }  
          if(b\_thread==FALSE)  
          {  
            \_PsResumeThread(thread\_temp,NULL);  
            KdPrint(("删除<->线程信息:Thread\[%d\]  StartAddr=%08x   TID:%d \\r\\n",i+1,Thread\_Enum->StartAddress,Thread\_Enum->ClientId.UniqueThread));  
          }  
        }  
        Thread\_Enum++;  
      }  
    }  
    if(Process\_Enum->NextEntryDelta==NULL)  
    {  
      break;  
    }  
    Process\_Enum=(PSYSTEM\_PROCESSES)(((PUCHAR)Process\_Enum)+Process\_Enum->NextEntryDelta);  
  }while(1);  
  ExFreePool(pbuffer);  
}  
  
jmp\_code \_jmp\_code\_NtOpenProecss;  
pjmp\_code \_pjmp\_code\_NtOpenProecss;  
#pragma INITCODE  
VOID Hook\_NtOpenProcess()  
{  
  ULONG my\_function\_addr;  
  addr\_NtOpenProcess=Getssdt\_addr(in\_NtOpenProcess);  
  addr\_NtOpenProcess+=5;  
  KdPrint(("向 NtOpenProcess 写跳转地址为:%x\\n",addr\_NtOpenProcess));  
  \_\_asm  
  {  
    push eax  
    mov eax,My\_NtOpenProecss  
    mov my\_function\_addr,eax  
    pop eax  
  }  
  KdPrint(("自定义 NtOpenProcess 的地址为:%x\\n",my\_function\_addr));  
  push\_addr\_NtOpenProcess=addr\_ObWatchHandles+0x25c;  
  KdPrint(("NtOpenProcess 的 push 地址为:%x\\n",push\_addr\_NtOpenProcess));  
  call\_addr\_NtOpenProecss=addr\_SEH\_prolog;  
  KdPrint(("NtOpenProcess 的 call 地址为:%x\\n",call\_addr\_NtOpenProecss));  
  \_pjmp\_code\_NtOpenProecss=(pjmp\_code)addr\_NtOpenProcess;  
  \_jmp\_code\_NtOpenProecss.E9=\_pjmp\_code\_NtOpenProecss->E9;  
  \_jmp\_code\_NtOpenProecss.Fun\_addr=\_pjmp\_code\_NtOpenProecss->Fun\_addr;  
  PAGED\_Open();  
  \_pjmp\_code\_NtOpenProecss->E9=0xE9;  
  \_pjmp\_code\_NtOpenProecss->Fun\_addr=(ULONG)(my\_function\_addr-addr\_NtOpenProcess-5);  
  PAGED\_Exit();  
}  
  
#pragma PAGEDCODE  
VOID \_\_declspec(naked) My\_NtOpenProecss()  
{  
  \_\_asm  
  {  
    push push\_addr\_NtOpenProcess  
    mov eax,addr\_NtOpenProcess  
    add eax,0xA  
    push eax  
    mov eax,call\_addr\_NtOpenProecss  
    jmp eax  
  }  
}  
  
#pragma PAGEDCODE  
VOID Un\_NtOpenProecss()  
{  
  PAGED\_Open();  
  \_pjmp\_code\_NtOpenProecss->E9=\_jmp\_code\_NtOpenProecss.E9;  
  \_pjmp\_code\_NtOpenProecss->Fun\_addr=\_jmp\_code\_NtOpenProecss.Fun\_addr;  
  PAGED\_Exit();  
}  
  
jmp\_code \_jmp\_code\_NtReadVirtualMemory;  
pjmp\_code \_pjmp\_code\_NtReadVirtualMemory;  
#pragma INITCODE  
VOID Hook\_NtReadVirtualMemory()  
{  
  ULONG My\_Function\_Addr;  
  addr\_NtReadVirtualMemory=(ULONG)Getssdt\_addr(in\_NtReadVirtualMemory);  
  addr\_NtReadVirtualMemory+=2;  
  KdPrint(("向 NtReadVirtualMemory 写跳转地址为:%x\\n",addr\_NtReadVirtualMemory));  
  \_\_asm  
  {  
    push eax  
    mov eax,My\_NtReadVirtualMemory  
    mov My\_Function\_Addr,eax  
    pop eax  
  }  
  KdPrint(("自定义 NtReadVirtualMemory 的地址为:%x\\n",My\_Function\_Addr));  
  \_\_asm  
  {  
    push eax  
    mov eax,addr\_NtReadVirtualMemory  
    add eax,1  
    mov eax,\[eax\]  
    mov push\_addr\_NtReadVirtualMemory,eax  
    pop eax  
  }  
  KdPrint(("NtReadVirtualMemory 的 push 地址为:%x\\n",push\_addr\_NtReadVirtualMemory));  
  call\_addr\_NtReadVirtualMemory=addr\_SEH\_prolog;  
  KdPrint(("NtReadVirtualMemory 的 call 地址为:%x\\n",call\_addr\_NtReadVirtualMemory));  
  \_pjmp\_code\_NtReadVirtualMemory=(pjmp\_code)addr\_NtReadVirtualMemory;  
  \_jmp\_code\_NtReadVirtualMemory.E9=\_pjmp\_code\_NtReadVirtualMemory->E9;  
  \_jmp\_code\_NtReadVirtualMemory.Fun\_addr=\_pjmp\_code\_NtReadVirtualMemory->Fun\_addr;  
  PAGED\_Open();  
  \_pjmp\_code\_NtReadVirtualMemory->E9=0xE9;  
  \_pjmp\_code\_NtReadVirtualMemory->Fun\_addr=(ULONG)(My\_Function\_Addr-addr\_NtReadVirtualMemory-5);  
  PAGED\_Exit();  
}  
  
#pragma PAGEDCODE  
VOID \_\_declspec(naked) My\_NtReadVirtualMemory()  
{  
  \_\_asm  
  {  
    push push\_addr\_NtReadVirtualMemory  
    mov eax,addr\_NtReadVirtualMemory  
    add eax,0xA  
    push eax  
    mov eax,call\_addr\_NtReadVirtualMemory  
    jmp eax  
  }  
}  
  
#pragma PAGEDCODE  
VOID Un\_NtReadVirtualMemory()  
{  
  PAGED\_Open();  
  \_pjmp\_code\_NtReadVirtualMemory->E9=\_jmp\_code\_NtReadVirtualMemory.E9;  
  \_pjmp\_code\_NtReadVirtualMemory->Fun\_addr=\_jmp\_code\_NtReadVirtualMemory.Fun\_addr;  
  PAGED\_Exit();  
}  
  
jmp\_code \_jmp\_code\_NtWriteVirtrualMemory;  
pjmp\_code \_pjmp\_code\_NtWriteVirtualMemory;  
#pragma INITCODE  
VOID Hook\_NtWriteVirtualMemory()  
{  
  ULONG My\_Function\_Addr;  
  addr\_NtWriteVirtualMemory=(ULONG)Getssdt\_addr(in\_NtWriteVirtualMemory);  
  addr\_NtWriteVirtualMemory+=2;  
  KdPrint(("向 NtWriteVirtualMemory 写跳转地址为:%x\\n",addr\_NtWriteVirtualMemory));  
  \_\_asm  
  {  
    push eax  
    mov eax,My\_NtWriteVirtualMemory  
    mov My\_Function\_Addr,eax  
    pop eax  
  }  
  KdPrint(("自定义 NtWriteVirtualMemory 的地址为:%x\\n",My\_Function\_Addr));  
  \_\_asm  
  {  
    push eax  
    mov eax,addr\_NtWriteVirtualMemory  
    add eax,1  
    mov eax,\[eax\]  
    mov push\_addr\_NtWriteVirtualMemory,eax  
    pop eax  
  }  
  KdPrint(("NtWriteVirtualMemory 的 push 地址为:%x\\n",push\_addr\_NtWriteVirtualMemory));  
  call\_addr\_NtWriteVirtualMemory=addr\_SEH\_prolog;  
  KdPrint(("NtWriteVirtualMemory 的 call 地址为:%x\\n",call\_addr\_NtWriteVirtualMemory));  
  \_pjmp\_code\_NtWriteVirtualMemory=(pjmp\_code)addr\_NtWriteVirtualMemory;  
  \_jmp\_code\_NtWriteVirtrualMemory.E9=\_pjmp\_code\_NtWriteVirtualMemory->E9;  
  \_jmp\_code\_NtWriteVirtrualMemory.Fun\_addr=\_pjmp\_code\_NtWriteVirtualMemory->Fun\_addr;  
  PAGED\_Open();  
  \_pjmp\_code\_NtWriteVirtualMemory->E9=0xE9;  
  \_pjmp\_code\_NtWriteVirtualMemory->Fun\_addr=(ULONG)(My\_Function\_Addr-addr\_NtWriteVirtualMemory-5);  
  PAGED\_Exit();  
}  
  
#pragma PAGEDCODE  
VOID \_\_declspec(naked) My\_NtWriteVirtualMemory()  
{  
  \_\_asm  
  {  
    push push\_addr\_NtWriteVirtualMemory  
    mov eax,addr\_NtWriteVirtualMemory  
    add eax,0xA  
    push eax  
    mov eax,call\_addr\_NtWriteVirtualMemory  
    jmp eax  
  }  
}  
  
#pragma PAGEDCODE  
VOID Un\_NtWriteVirtualMemory()  
{  
  PAGED\_Open();  
  \_pjmp\_code\_NtWriteVirtualMemory->E9=\_jmp\_code\_NtWriteVirtrualMemory.E9;  
  \_pjmp\_code\_NtWriteVirtualMemory->Fun\_addr=\_jmp\_code\_NtWriteVirtrualMemory.Fun\_addr;  
  PAGED\_Exit();  
}  
  
jmp\_code \_jmp\_code\_NtClose;  
pjmp\_code \_pjmp\_code\_NtClose;  
#pragma INITCODE  
VOID Hook\_NtClose()  
{  
  ULONG My\_Function\_Addr;  
  addr\_NtClose=(ULONG)Getssdt\_addr(in\_NtClose);  
  call\_addr\_NtClose=addr\_NtClose-0x1A2;  
  KdPrint(("NtClose 的 call 地址为:%x\\n",call\_addr\_NtClose));  
  addr\_NtClose+=0x12;  
  KdPrint(("向 NtClose 写跳转的地址为:%x\\n",addr\_NtClose));  
  \_\_asm  
  {  
    push eax  
    mov eax,My\_NtClose  
    mov My\_Function\_Addr,eax  
    pop eax  
  }  
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
}  
   
#pragma PAGEDCODE  
VOID \_\_declspec(naked) My\_NtClose()  
{  
  \_\_asm  
  {  
    push 0  
    push eax  
    push dword ptr \[ebp+8\]  
    mov eax,addr\_NtClose  
    add eax,0xB  
    push eax  
    mov eax,call\_addr\_NtClose  
    jmp eax  
  }  
}  
  
#pragma PAGEDCODE  
VOID Un\_NtClocse()  
{  
  PAGED\_Open();  
  \_pjmp\_code\_NtClose->E9=\_jmp\_code\_NtClose.E9;  
  \_pjmp\_code\_NtClose->Fun\_addr=\_jmp\_code\_NtClose.Fun\_addr;  
  \_pjmp\_code\_NtClose->nop\[0\]=\_jmp\_code\_NtClose.nop\[0\];  
  PAGED\_Exit();  
}  
  
jmp\_code \_jmp\_code\_NtDeviceIoControlFile;  
pjmp\_code \_pjmp\_code\_NtDeviceIoControlFile;  
#pragma INITCODE   
VOID Hook\_NtDeviceIoControlFile()  
{  
  ULONG My\_Function\_Addr;  
  ULONG u\_NtFsControlFile;  
  addr\_NtDeviceIoControlFile=(ULONG)Getssdt\_addr(in\_NtDeviceIoControlFile);  
  addr\_NtDeviceIoControlFile+=0x1F;  
  KdPrint(("向 NtDeviceIoControlFile 写跳转的地址为:%x\\n",addr\_NtDeviceIoControlFile));  
  u\_NtFsControlFile=(ULONG)Getssdt\_addr(in\_NtFsControlFile);  
  call\_addr\_IopXxxControlFile=u\_NtFsControlFile;  
  u\_NtFsControlFile=u\_NtFsControlFile+0x25;  
  \_\_asm  
  {  
    push eax  
    push ebx  
    mov eax,u\_NtFsControlFile  
    add eax,1  
    mov ebx,\[eax\]  
    add eax,ebx  
    add eax,5  
    sub eax,1  
    mov call\_addr\_NtDeviceIoControlFile,eax  
    pop ebx  
    pop eax  
  }  
  addr\_IopXxxControlFile=call\_addr\_NtDeviceIoControlFile;  
  KdPrint(("NtDeviceIoControlFile 的 call 地址为:%x\\n",call\_addr\_NtDeviceIoControlFile));  
  \_\_asm  
  {  
    push eax  
    mov eax,My\_NtDeviceIoControlFile  
    mov My\_Function\_Addr,eax  
    pop eax  
  }  
  KdPrint(("自定义 NtDeviceIoControlFile 的地址为:%x\\n",My\_Function\_Addr));  
  \_pjmp\_code\_NtDeviceIoControlFile=(pjmp\_code)addr\_NtDeviceIoControlFile;  
  \_jmp\_code\_NtDeviceIoControlFile.E9=\_pjmp\_code\_NtDeviceIoControlFile->E9;  
  \_jmp\_code\_NtDeviceIoControlFile.Fun\_addr=\_pjmp\_code\_NtDeviceIoControlFile->Fun\_addr;  
  PAGED\_Open();  
  \_pjmp\_code\_NtDeviceIoControlFile->E9=0xE9;  
  \_pjmp\_code\_NtDeviceIoControlFile->Fun\_addr=(ULONG)(My\_Function\_Addr-addr\_NtDeviceIoControlFile-5);  
  PAGED\_Exit();  
}  
  
#pragma PAGEDCODE  
VOID \_\_declspec(naked) My\_NtDeviceIoControlFile()  
{  
  \_\_asm  
  {  
    push dword ptr \[ebp+0Ch\]  
    push dword ptr \[ebp+8\]  
    mov eax,addr\_NtDeviceIoControlFile  
    add eax,0xB  
    push eax  
    mov eax,call\_addr\_NtDeviceIoControlFile  
    jmp eax  
  }  
}  
  
#pragma PAGEDCODE  
VOID Unnt\_NtDeviceIoControlFile()  
{  
  PAGED\_Open();  
  \_pjmp\_code\_NtDeviceIoControlFile->E9=\_jmp\_code\_NtDeviceIoControlFile.E9;  
  \_pjmp\_code\_NtDeviceIoControlFile->Fun\_addr=\_jmp\_code\_NtDeviceIoControlFile.Fun\_addr;  
  PAGED\_Exit();  
}  
  
jmp\_code \_jmp\_code\_KiAttachProcess;  
pjmp\_code \_pjmp\_code\_KiAttachProcess;  
#pragma INITCODE  
VOID Hook\_KiAttachProcess()  
{  
  ULONG My\_Function\_Addr;  
  ULONG u\_KeAttachProcess;  
  UNICODE\_STRING KeAttachProcess\_name;  
  RtlInitUnicodeString(&KeAttachProcess\_name,L"KeAttachProcess");  
  u\_KeAttachProcess=(ULONG)MmGetSystemRoutineAddress(&KeAttachProcess\_name);  
  KdPrint(("套出 KeAttachProcess 的地址为:%x\\n",u\_KeAttachProcess));  
  BYTE\* \_bp;  
  \_bp=(BYTE\*)u\_KeAttachProcess;  
  do   
  {  
    if((\*(\_bp-2)==0x57)&&(\*(\_bp-1)==0x56)&&(\*(\_bp)==0xE8))  
    {  
      break;  
    }  
    \_bp++;  
  }while(1);  
  addr\_KiAttachProcess=(ULONG)\_bp;  
  \_\_asm  
  {  
    push eax  
    push ebx  
    mov eax,addr\_KiAttachProcess  
    add eax,1  
    mov ebx,\[eax\]  
    add eax,ebx  
    add eax,5  
    sub eax,1  
    mov addr\_KiAttachProcess,eax  
    pop ebx  
    pop eax  
  }  
  KdPrint(("KiAttachProcess 的地址为%x\\n",addr\_KiAttachProcess));  
  call\_addr\_KiAttachProcess=addr\_KiAttachProcess-0x488;  
  KdPrint(("KiAttachProcess 的 call 地址为:%x\\n",call\_addr\_KiAttachProcess));  
  addr\_KiAttachProcess+=0x11;  
  KdPrint(("向 KiAttachProcess 写跳转地址:%x\\n",addr\_KiAttachProcess));  
  \_\_asm  
  {  
    push eax  
    mov eax,My\_KiAttachProcess  
    mov My\_Function\_Addr,eax  
    pop eax  
  }  
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
}  
  
#pragma PAGEDCODE  
VOID \_\_declspec(naked) My\_KiAttachProcess()  
{  
  \_\_asm  
  {  
    inc word ptr \[edi+60h\]  
    lea ebx,\[esi+34h\]  
    push ebx  
    mov eax,addr\_KiAttachProcess  
    add eax,0xD  
    push eax  
    mov eax,call\_addr\_KiAttachProcess  
    jmp eax  
  }  
}  
  
#pragma PAGEDCODE  
VOID Unnt\_KiAttachProecss()  
{  
  PAGED\_Open();  
  \_pjmp\_code\_KiAttachProcess->E9=\_jmp\_code\_KiAttachProcess.E9;  
  \_pjmp\_code\_KiAttachProcess->Fun\_addr=\_jmp\_code\_KiAttachProcess.Fun\_addr;  
  \_pjmp\_code\_KiAttachProcess->nop\[0\]=\_jmp\_code\_KiAttachProcess.nop\[0\];  
  \_pjmp\_code\_KiAttachProcess->nop\[1\]=\_jmp\_code\_KiAttachProcess.nop\[1\];  
  PAGED\_Exit();  
}  
  
jmp\_code \_jmp\_code\_IopXxxControlFile;  
pjmp\_code \_pjmp\_code\_IopXxxControlFile;  
#pragma INITCODE  
VOID Hook\_IopXxxControlFile()  
{  
  ULONG My\_Function\_Addr;  
  BYTE\* \_bp;  
  \_bp=(BYTE\*)addr\_IopXxxControlFile;  
  do   
  {  
    if((\*(\_bp-11)==0x6A)&&(\*(\_bp-6)==0xFF)&&(\*(\_bp-3)==0xFF)&&(\*(\_bp)==0xE8)&&(\*(\_bp+5)==0x83)&&(\*(\_bp+9)==0x8B))  
    {  
      break;  
    }  
    \_bp++;  
  }while(1);  
  addr\_IopXxxControlFile=(ULONG)\_bp-6;  
  KdPrint(("向 IopXxxControlFile 写跳转地址为:%x\\n",addr\_IopXxxControlFile));  
  \_bp=(BYTE\*)call\_addr\_IopXxxControlFile;  
  do   
  {  
    if((\*(\_bp-11)==0x6A)&&(\*(\_bp-6)==0xFF)&&(\*(\_bp-3)==0xFF)&&(\*(\_bp)==0xE8)&&(\*(\_bp+5)==0x83)&&(\*(\_bp+9)==0x8B))  
    {  
      break;  
    }  
    \_bp++;  
  }while(1);  
  call\_addr\_IopXxxControlFile=(ULONG)\_bp;  
  \_\_asm   
  {  
    push eax  
    push ebx  
    mov eax,call\_addr\_IopXxxControlFile  
    add eax,1  
    mov ebx,\[eax\]  
    add eax,ebx  
    add eax,5  
    sub eax,1  
    mov call\_addr\_IopXxxControlFile,eax  
    pop ebx  
    pop eax  
  }  
  KdPrint(("IopXxxControlFile 的 call 地址为:%x\\n",call\_addr\_IopXxxControlFile));  
  \_\_asm  
  {  
    push eax  
    mov eax,My\_IopXxxControlFile  
    mov My\_Function\_Addr,eax  
    pop eax  
  }  
  KdPrint(("自定义 IopXxxControlFile 的地址为:%x\\n",My\_Function\_Addr));  
  \_pjmp\_code\_IopXxxControlFile=(pjmp\_code)addr\_IopXxxControlFile;  
  \_jmp\_code\_IopXxxControlFile.E9=\_pjmp\_code\_IopXxxControlFile->E9;  
  \_jmp\_code\_IopXxxControlFile.Fun\_addr=\_pjmp\_code\_IopXxxControlFile->Fun\_addr;  
  \_jmp\_code\_IopXxxControlFile.nop\[0\]=\_pjmp\_code\_IopXxxControlFile->nop\[0\];  
  PAGED\_Open();  
  \_pjmp\_code\_IopXxxControlFile->E9=0xE9;  
  \_pjmp\_code\_IopXxxControlFile->Fun\_addr=(ULONG)(My\_Function\_Addr-addr\_IopXxxControlFile-5);  
  \_pjmp\_code\_IopXxxControlFile->nop\[0\]=0x90;  
  PAGED\_Exit();  
}  
  
#pragma PAGEDCODE  
VOID \_\_declspec(naked) My\_IopXxxControlFile()  
{  
  \_\_asm  
  {  
    push dword ptr \[ebp-40h\]  
    push dword ptr \[ebp-24h\]  
    mov eax,addr\_IopXxxControlFile  
    add eax,0xB  
    push eax  
    mov eax,call\_addr\_IopXxxControlFile  
    jmp eax  
  }  
}  
  
#pragma PAGEDCODE  
VOID Un\_IopXxxControlFile()  
{  
  PAGED\_Open();  
  \_pjmp\_code\_IopXxxControlFile->E9=\_jmp\_code\_IopXxxControlFile.E9;  
  \_pjmp\_code\_IopXxxControlFile->Fun\_addr=\_jmp\_code\_IopXxxControlFile.Fun\_addr;  
  \_pjmp\_code\_IopXxxControlFile->nop\[0\]=\_jmp\_code\_IopXxxControlFile.nop\[0\];  
  PAGED\_Exit();  
}  
  
jmp\_code \_jmp\_code\_NtGetContextThread;  
pjmp\_code \_pjmp\_code\_NtGetContextThread;  
#pragma INITCODE  
VOID Hook\_NtGetContextThread()  
{  
  ULONG My\_Function\_Addr;  
  BYTE\* \_bp;  
  \_bp=(BYTE\*)Getssdt\_addr(in\_NtGetContextThread);  
  do   
  {  
    if((\*(\_bp-15)==0x50)&&(\*(\_bp-5)==0x6A)&&(\*(\_bp-3)==0xFF)&&(\*(\_bp)==0xE8)&&(\*(\_bp+5)==0x8B)&&(\*(\_bp+7)==0x85))  
    {  
      break;  
    }  
    \_bp++;  
  } while (1);  
  addr\_NtGetContextThread=(ULONG)\_bp-5;  
  KdPrint(("向 NtGetContextThread 写跳转的地址为:%x\\n",addr\_NtGetContextThread));  
  \_bp=(BYTE\*)Getssdt\_addr(in\_NtTerminateThread);  
  do   
  {  
    if((\*(\_bp-15)==0x50)&&(\*(\_bp-5)==0x6A)&&(\*(\_bp-3)==0xFF)&&(\*(\_bp)==0xE8)&&(\*(\_bp+5)==0x8B)&&(\*(\_bp+7)==0x85))  
    {  
      break;  
    }  
    \_bp++;  
  } while (1);  
  call\_addr\_NtGetContextThread=(ULONG)\_bp;  
  KdPrint(("NtGetContextThread 的 call 来自于:%x\\n",call\_addr\_NtGetContextThread));  
  \_\_asm  
  {  
    push eax  
    push ebx  
    mov eax,call\_addr\_NtGetContextThread  
    add eax,1  
    mov ebx,\[eax\]  
    add eax,ebx  
    add eax,5  
    sub eax,1  
    mov call\_addr\_NtGetContextThread,eax  
    pop ebx  
    pop eax  
  }  
  call\_addr\_NtSetContextThread=call\_addr\_NtGetContextThread;  
  KdPrint(("从 NtTerminateThread 获取的 NtGetContextThread 的 call ObReferenceObjectByHandle 的地址为:%x\\n",call\_addr\_NtGetContextThread));  
  \_\_asm  
  {  
    push eax  
    mov eax,My\_NtGetContextThread  
    mov My\_Function\_Addr,eax  
    pop eax  
  }  
  KdPrint(("自定义 NtGetContextThread 的地址为:%x\\n",My\_Function\_Addr));  
  Game\_call\_addr\_1=addr\_NtGetContextThread+5;  
  \_\_asm  
  {  
    push eax  
    push ebx  
    mov eax,Game\_call\_addr\_1  
    add eax,1  
    mov ebx,\[eax\]  
    add eax,ebx  
    add eax,5  
    sub eax,1  
    mov Game\_call\_addr\_1,eax  
    pop ebx  
    pop eax  
  }  
  KdPrint(("获取 NtGetContextThread 的 call 获取的第一个 call 地址为:%x\\n",Game\_call\_addr\_1));  
  \_pjmp\_code\_NtGetContextThread=(pjmp\_code)addr\_NtGetContextThread;  
  \_jmp\_code\_NtGetContextThread.E9=\_pjmp\_code\_NtGetContextThread->E9;  
  \_jmp\_code\_NtGetContextThread.Fun\_addr=\_pjmp\_code\_NtGetContextThread->Fun\_addr;  
  PAGED\_Open();  
  \_pjmp\_code\_NtGetContextThread->E9=0xE9;  
  \_pjmp\_code\_NtGetContextThread->Fun\_addr=(ULONG)(My\_Function\_Addr-addr\_NtGetContextThread-5);  
  PAGED\_Exit();  
}  
  
PEPROCESS eprocess;  
ANSI\_STRING GetProcess\_1;  
ANSI\_STRING OD\_Name\_1;  
ANSI\_STRING OD\_Name\_2;  
ANSI\_STRING OD\_Name\_3;  
#pragma PAGEDCODE  
VOID \_\_declspec(naked) My\_NtGetContextThread()  
{  
  \_\_asm  
  {  
    pushad  
    pushf  
  }  
  eprocess=PsGetCurrentProcess();  
  RtlInitAnsiString(&GetProcess\_1,(PCSZ)((ULONG)eprocess+0x174));  
  RtlInitAnsiString(&OD\_Name\_1,\_Game\_Name);  
  RtlInitAnsiString(&OD\_Name\_2,\_Game\_Name1);  
  RtlInitAnsiString(&OD\_Name\_3,\_Game\_Name2);  
  if(RtlCompareString(&GetProcess\_1,&OD\_Name\_1,TRUE)==0)  
  {  
    \_\_asm  
    {  
      popf  
      popad  
      push 8  
      push dword ptr \[ebp+8\]  
      mov eax,addr\_NtGetContextThread  
      add eax,0xA  
      push eax  
      jmp call\_addr\_NtGetContextThread  
    }  
  }  
  else  
  {  
    if(RtlCompareString(&GetProcess\_1,&OD\_Name\_2,TRUE)==0)  
    {  
      \_\_asm  
      {  
        popf  
        popad  
        push 8  
        push dword ptr \[ebp+8\]  
        mov eax,addr\_NtGetContextThread  
        add eax,0xA  
        push eax  
        jmp call\_addr\_NtGetContextThread  
      }  
    }  
    else  
    {  
      if(RtlCompareString(&GetProcess\_1,&OD\_Name\_3,TRUE)!=0)  
      {  
        \_\_asm  
        {  
          popf  
          popad  
          push 8  
          push dword ptr \[ebp+8\]  
          mov eax,addr\_NtGetContextThread  
          add eax,0xA  
          push eax  
          jmp call\_addr\_NtGetContextThread  
        }  
      }  
      else  
      {  
        \_\_asm  
        {  
          popf  
          popad  
          push 8  
          push dword ptr \[ebp+8\]  
          mov eax,addr\_NtGetContextThread  
          add eax,0xA  
          push eax  
          jmp Game\_call\_addr\_1  
        }  
      }  
    }  
      
  }  
}  
  
#pragma PAGEDCODE  
VOID Un\_NtGetContextThread()  
{  
  PAGED\_Open();  
  \_pjmp\_code\_NtGetContextThread->E9=\_jmp\_code\_NtGetContextThread.E9;  
  \_pjmp\_code\_NtGetContextThread->Fun\_addr=\_jmp\_code\_NtGetContextThread.Fun\_addr;  
  PAGED\_Exit();  
}  
  
jmp\_code \_jmp\_code\_NtSetContextThread;  
pjmp\_code \_pjmp\_code\_NtSetContextThread;  
#pragma INITCODE   
VOID Hook\_NtSetContextThread()  
{  
  ULONG My\_Function\_Addr;  
  BYTE\* \_bp;  
  \_bp=(BYTE\*)Getssdt\_addr(in\_NtSetContextThread);  
  do   
  {  
    if((\*(\_bp-15)==0x50)&&(\*(\_bp-5)==0x6A)&&(\*(\_bp-3)==0xFF)&&(\*(\_bp)==0xE8)&&(\*(\_bp+5)==0x8B)&&(\*(\_bp+7)==0x85))  
    {  
      break;  
    }  
    \_bp++;  
  } while (1);  
  addr\_NtSetContextThread=(ULONG)\_bp-5;  
  KdPrint(("向 NtSetContextThread 写跳转地址的:%x\\n",addr\_NtSetContextThread));  
  \_\_asm  
  {  
    push eax  
    mov eax,My\_NtSetContextThread  
    mov My\_Function\_Addr,eax  
    pop eax  
  }  
  KdPrint(("自定义 NtSetContextThread 的地址为:%x\\n",My\_Function\_Addr));  
  \_pjmp\_code\_NtSetContextThread=(pjmp\_code)addr\_NtSetContextThread;  
  \_jmp\_code\_NtSetContextThread.E9=\_pjmp\_code\_NtSetContextThread->E9;  
  \_jmp\_code\_NtSetContextThread.Fun\_addr=\_pjmp\_code\_NtSetContextThread->Fun\_addr;  
  PAGED\_Open();  
  \_pjmp\_code\_NtSetContextThread->E9=0xE9;  
  \_pjmp\_code\_NtSetContextThread->Fun\_addr=(ULONG)(My\_Function\_Addr-addr\_NtSetContextThread-5);  
  PAGED\_Exit();  
}  
  
#pragma PAGEDCODE  
VOID \_\_declspec(naked) My\_NtSetContextThread()  
{  
  \_\_asm  
  {  
    push 1  
    push dword ptr \[ebp+8\]  
    mov eax,addr\_NtSetContextThread  
    add eax,0xA  
    push eax  
    mov eax,call\_addr\_NtSetContextThread  
    jmp eax  
  }  
}  
  
#pragma PAGEDCODE  
VOID Un\_NtSetContextThread()  
{  
  PAGED\_Open();  
  \_pjmp\_code\_NtSetContextThread->E9=\_jmp\_code\_NtSetContextThread.E9;  
  \_pjmp\_code\_NtSetContextThread->Fun\_addr=\_jmp\_code\_NtSetContextThread.Fun\_addr;  
  PAGED\_Exit();  
}  
  
jmp\_code \_jmp\_code\_NtGetContextThread1;  
pjmp\_code \_pjmp\_code\_NtGetContextThread1;  
#pragma INITCODE  
VOID Hook\_NtGetContextThread1()  
{  
  ULONG My\_Function\_Addr;  
  BYTE\* \_bp;  
  \_bp=(BYTE\*)Getssdt\_addr(in\_NtGetContextThread);  
  do   
  {  
    if((\*(\_bp-7)==0xBE)&&(\*(\_bp-2)==0x8B)&&(\*(\_bp-1)==0xCF)&&(\*(\_bp)==0xE8)&&(\*(\_bp+5)==0x5F)&&(\*(\_bp+6)==0x8B))  
    {  
      break;  
    }  
    \_bp++;  
  } while (1);  
  addr\_NtGetContextThread1=(ULONG)\_bp-9;  
  KdPrint(("向 NtGetContextThread 写跳转地址为:%x\\n",addr\_NtGetContextThread1));  
  \_bp=(BYTE\*)Getssdt\_addr(in\_NtTerminateThread);  
  do   
  {  
    if((\*(\_bp-9)==0xE8)&&(\*(\_bp-4)==0x8B)&&(\*(\_bp-2)==0x8B)&&(\*(\_bp)==0xE8)&&(\*(\_bp+5)==0x8B)&&(\*(\_bp+6)==0xC7))  
    {  
      break;  
    }  
    \_bp++;  
  } while (1);  
  call\_addr\_NtGetContextThread1=(ULONG)\_bp;  
  \_\_asm  
  {  
    push eax  
    push ebx  
    mov eax,call\_addr\_NtGetContextThread1  
    add eax,1  
    mov ebx,\[eax\]  
    add eax,ebx  
    add eax,5  
    sub eax,1  
    mov call\_addr\_NtGetContextThread1,eax  
    pop ebx  
    pop eax  
  }  
  KdPrint(("从 NtTerminateThread 获取的 NtGetContextThread 的第二个 call 地址为:%x\\n",call\_addr\_NtGetContextThread1));  
  \_\_asm  
  {  
    push eax  
    mov eax,My\_NtGetContextThread1  
    mov My\_Function\_Addr,eax  
    pop eax  
  }  
  KdPrint(("NtGetContextThread 的第二个自写函数地址为:%x\\n",My\_Function\_Addr));  
  Game\_call\_addr\_2=addr\_NtGetContextThread1+9;  
  \_\_asm  
  {  
    push eax  
    push ebx  
    mov eax,Game\_call\_addr\_2  
    add eax,1  
    mov ebx,\[eax\]  
    add eax,ebx  
    add eax,5  
    sub eax,1  
    mov Game\_call\_addr\_2,eax  
    pop ebx  
    pop eax  
  }  
  KdPrint(("获取 NtGetContextThread 的 call 获取的第二个 call 地址为:%x\\n",Game\_call\_addr\_2));  
  \_pjmp\_code\_NtGetContextThread1=(pjmp\_code)addr\_NtGetContextThread1;  
  \_jmp\_code\_NtGetContextThread1.E9=\_pjmp\_code\_NtGetContextThread1->E9;  
  \_jmp\_code\_NtGetContextThread1.Fun\_addr=\_pjmp\_code\_NtGetContextThread1->Fun\_addr;  
  \_jmp\_code\_NtGetContextThread1.nop\[0\]=\_pjmp\_code\_NtGetContextThread1->nop\[0\];  
  \_jmp\_code\_NtGetContextThread1.nop\[1\]=\_pjmp\_code\_NtGetContextThread1->nop\[1\];  
  PAGED\_Open();  
  \_pjmp\_code\_NtGetContextThread1->E9=0xE9;  
  \_pjmp\_code\_NtGetContextThread1->Fun\_addr=(ULONG)(My\_Function\_Addr-addr\_NtGetContextThread1-5);  
  \_pjmp\_code\_NtGetContextThread1->nop\[0\]=0x90;  
  \_pjmp\_code\_NtGetContextThread1->nop\[1\]=0x90;  
  PAGED\_Exit();  
}  
  
PEPROCESS eprocess\_temp;  
ANSI\_STRING GetProcess\_temp1;  
ANSI\_STRING OD\_Name\_temp1;  
ANSI\_STRING OD\_Name\_temp2;  
ANSI\_STRING OD\_Name\_temp3;  
#pragma PAGEDCODE  
VOID \_\_declspec(naked) My\_NtGetContextThread1()  
{  
  \_\_asm  
  {  
    pushad  
    pushf  
  }  
  eprocess\_temp=PsGetCurrentProcess();  
  RtlInitAnsiString(&GetProcess\_temp1,(PCSZ)((ULONG)eprocess\_temp+0x174));  
  RtlInitAnsiString(&OD\_Name\_temp1,\_Game\_Name);  
  RtlInitAnsiString(&OD\_Name\_temp2,\_Game\_Name1);  
  RtlInitAnsiString(&OD\_Name\_temp3,\_Game\_Name2);  
  if(RtlCompareString(&GetProcess\_temp1,&OD\_Name\_temp1,TRUE)==0)  
  {  
    \_\_asm  
    {  
      popf  
      popad  
      jmp \_\_tiao2  
      mov esi,0C0000008h  
\_\_tiao2:  
      mov ecx,edi  
      mov eax,addr\_NtGetContextThread1  
      add eax,0xE  
      push eax  
      jmp call\_addr\_NtGetContextThread1  
    }  
  }  
  else  
  {  
    if(RtlCompareString(&GetProcess\_temp1,&OD\_Name\_temp2,TRUE)==0)  
    {  
      \_\_asm  
      {  
        popf  
        popad  
        jmp \_\_tiaotemp1  
        mov esi,0C0000008h  
\_\_tiaotemp1:  
        mov ecx,edi  
        mov eax,addr\_NtGetContextThread1  
        add eax,0xE  
        push eax  
        jmp call\_addr\_NtGetContextThread1  
      }  
    }  
    else  
    {  
      if(RtlCompareString(&GetProcess\_temp1,&OD\_Name\_temp3,TRUE)==0)  
      {  
        \_\_asm  
        {  
          popf  
          popad  
          jmp \_\_tiaotemp2  
          mov esi,0C0000008h  
\_\_tiaotemp2:  
          mov ecx,edi  
          mov eax,addr\_NtGetContextThread1  
          add eax,0xE  
          push eax  
          jmp call\_addr\_NtGetContextThread1  
        }  
      }  
      else  
      {  
        \_\_asm  
        {  
          popf  
          popad  
          jmp \_\_tiaotemp3  
          mov esi,0C0000008h  
\_\_tiaotemp3:  
          mov ecx,edi  
          mov eax,addr\_NtGetContextThread1  
          add eax,0xE  
          push eax  
          jmp Game\_call\_addr\_2  
        }  
      }  
    }  
  }  
}  
  
#pragma PAGEDCODE  
VOID Un\_NtGetContextThread1()  
{  
  PAGED\_Open();  
  \_pjmp\_code\_NtGetContextThread1->E9=\_jmp\_code\_NtGetContextThread1.E9;  
  \_pjmp\_code\_NtGetContextThread1->Fun\_addr=\_jmp\_code\_NtGetContextThread1.Fun\_addr;  
  \_pjmp\_code\_NtGetContextThread1->nop\[0\]=\_jmp\_code\_NtGetContextThread1.nop\[0\];  
  \_pjmp\_code\_NtGetContextThread1->nop\[1\]=\_pjmp\_code\_NtGetContextThread1->nop\[1\];  
  PAGED\_Exit();  
}  
  
jmp\_code \_jmp\_code\_NtQueryPerformanceCounter;  
pjmp\_code \_pjmp\_code\_NtQueryPerformanceCounter;  
jmp\_code \_jmp\_code\_NtQueryPerformanceCounter\_1;  
pjmp\_code \_pjmp\_code\_NtQueryPerformanceCounter\_1;  
#pragma INITCODE  
VOID Hook\_NtQueryPerformanceCounter()  
{  
  ULONG My\_temp;  
  BYTE\* \_bp;  
  \_bp=(BYTE\*)addr\_NtOpenThread;  
  do   
  {  
    if((\*(\_bp-18)==0xC3)&&(\*(\_bp-14)==0x8B)&&(\*(\_bp-7)==0xEB)&&(\*(\_bp)==0xE8)&&(\*(\_bp+5)==0xC2)&&(\*(\_bp+6)==0x10))  
    {  
      break;  
    }  
    \_bp++;  
  } while (1);  
  call\_addr\_NtQueryPerformanceCounter=(ULONG)\_bp;  
  \_\_asm  
  {  
    push eax  
    push ebx  
    mov eax,call\_addr\_NtQueryPerformanceCounter  
    add eax,1  
    mov ebx,\[eax\]  
    add eax,ebx  
    add eax,5  
    sub eax,1  
    mov call\_addr\_NtQueryPerformanceCounter,eax  
    pop ebx  
    pop eax  
  }  
  call\_addr\_NtQueryPerformanceCounter=call\_addr\_NtQueryPerformanceCounter-addr\_NtQueryPerformanceCounter;  
  KdPrint(("NtQueryPerformanceCounter 的 call 地址为:%x\\n",call\_addr\_NtQueryPerformanceCounter));  
  \_bp=(BYTE\*)Getssdt\_addr(in\_NtQueryPerformanceCounter);  
  do   
  {  
    if((\*(\_bp-10)==0x8B)&&(\*(\_bp-7)==0x89)&&(\*(\_bp-4)==0x83)&&(\*(\_bp)==0xEB)&&(\*(\_bp+2)==0x8B)&&(\*(\_bp+5)==0x8B))  
    {  
      break;  
    }  
    \_bp++;  
  } while (1);  
  addr\_NtQueryPerformanceCounter=(ULONG)\_bp;  
  KdPrint(("第一个 NtQueryPerformanceCounter 的 JMP 地址为:%x\\n",addr\_NtQueryPerformanceCounter));  
  \_bp=(BYTE\*)addr\_NtQueryPerformanceCounter;  
  do   
  {  
    if((\*(\_bp-11)==0xC3)&&(\*(\_bp-10)==0x8B)&&(\*(\_bp-7)==0x83)&&(\*(\_bp-3)==0x8B)&&(\*(\_bp)==0xEB)&&(\*(\_bp+5)==0x50)&&(\*(\_bp+6)==0xFF))  
    {  
      break;  
    }  
    \_bp++;  
  } while (1);  
  addr1\_NtQueryPerformanceCounter=(ULONG)\_bp;  
  KdPrint(("第二个 NtQueryPerformanceCounter 的 JMP 地址为:%x\\n",addr1\_NtQueryPerformanceCounter));  
  \_\_asm  
  {  
    push eax  
      mov eax,My\_NtQueryPerformanceCounter\_1  
      mov My\_temp,eax  
      pop eax  
  }  
  KdPrint(("NtQueryPerformanceCounter 第一个自定义函数地址为:%x\\n",My\_temp));  
  \_pjmp\_code\_NtQueryPerformanceCounter=(pjmp\_code)addr\_NtQueryPerformanceCounter;  
  \_jmp\_code\_NtQueryPerformanceCounter.E9=\_pjmp\_code\_NtQueryPerformanceCounter->E9;  
  \_jmp\_code\_NtQueryPerformanceCounter.Fun\_addr=\_pjmp\_code\_NtQueryPerformanceCounter->Fun\_addr;  
  PAGED\_Open();  
  \_pjmp\_code\_NtQueryPerformanceCounter->E9=0xE9;  
  \_pjmp\_code\_NtQueryPerformanceCounter->Fun\_addr=(ULONG)(My\_temp-addr\_NtQueryPerformanceCounter-5);  
  PAGED\_Exit();  
  \_\_asm  
  {  
    push eax  
    mov eax,My\_NtQueryPerformanceCounter\_2  
    mov My\_temp,eax  
    pop eax  
  }  
  KdPrint(("NtQueryPerformanceCounter 第二个自定义函数地址为:%x\\n",My\_temp));  
  \_pjmp\_code\_NtQueryPerformanceCounter\_1=(pjmp\_code)addr1\_NtQueryPerformanceCounter;  
  \_jmp\_code\_NtQueryPerformanceCounter\_1.E9=\_pjmp\_code\_NtQueryPerformanceCounter\_1->E9;  
  \_jmp\_code\_NtQueryPerformanceCounter\_1.Fun\_addr=\_pjmp\_code\_NtQueryPerformanceCounter\_1->Fun\_addr;  
  PAGED\_Open();  
  \_pjmp\_code\_NtQueryPerformanceCounter\_1->E9=0xE9;  
  \_pjmp\_code\_NtQueryPerformanceCounter\_1->Fun\_addr=(ULONG)(My\_temp-addr1\_NtQueryPerformanceCounter-5);  
  PAGED\_Exit();  
}  
  
#pragma PAGEDCODE  
VOID \_\_declspec(naked)My\_NtQueryPerformanceCounter\_1()  
{  
  \_\_asm  
  {  
    mov eax,addr\_NtQueryPerformanceCounter  
    add eax,0x47  
    push eax  
    xor eax,eax  
    jmp call\_addr\_NtQueryPerformanceCounter  
  }  
}  
  
#pragma PAGEDCODE  
VOID \_\_declspec(naked)My\_NtQueryPerformanceCounter\_2()  
{  
  \_\_asm  
  {  
    pushad  
    mov eax,addr1\_NtQueryPerformanceCounter  
    add eax,0x2D  
    push eax  
    popad  
    jmp call\_addr\_NtQueryPerformanceCounter  
  }  
}  
  
#pragma PAGEDCODE  
VOID Un\_NtQueryPerformanceCounter()  
{  
  PAGED\_Open();  
  \_pjmp\_code\_NtQueryPerformanceCounter->E9=\_jmp\_code\_NtQueryPerformanceCounter.E9;  
  \_pjmp\_code\_NtQueryPerformanceCounter->Fun\_addr=\_jmp\_code\_NtQueryPerformanceCounter.Fun\_addr;  
  \_pjmp\_code\_NtQueryPerformanceCounter\_1->E9=\_jmp\_code\_NtQueryPerformanceCounter\_1.E9;  
  \_pjmp\_code\_NtQueryPerformanceCounter\_1->Fun\_addr=\_jmp\_code\_NtQueryPerformanceCounter\_1.Fun\_addr;  
  PAGED\_Exit();  
}  
  
struct \_str\_index  
{  
  ULONG KiDispatchException;        //1 处  
  ULONG NtQueryInformationProcess;      //1 处  
  ULONG PspCreateProcess;        //1 处  
  ULONG PsGetProcessDebugPort;      //1 处  
  ULONG PsIsProcessBeingDebugged;      //1 处  
  ULONG NtTerminateProcess;        //1 处  
  ULONG PspTerminateProcess;      //1 处  
  ULONG PspExitThread;        //1 处  
  ULONG PspProcessDelete\[2\];       //3 处  
  ULONG ObpCloseHandleTableEntry;      //1 处  
  ULONG ObpCloseHandle;        //1 处  
  ULONG MmCreatePeb;        //1 处  
  ULONG DbgkCreateThread;        //1 处  
  ULONG DbgkExitThread;        //1 处  
  ULONG DbgkExitProcess;        //1 处  
  ULONG DbgkMapViewOfSection;      //1 处  
  ULONG DbgkUnMapViewOfSection;      //1 处  
  ULONG DbgkpMarkProcessPeb;      //1 处  
  ULONG DbgkpCloseObject\[2\];        //3 处  
  ULONG DbgkCopyProcessDebugPort\[3\];      //4 处  
  ULONG DbgkOpenProcessDebugPort\[1\];      //2 处  
  ULONG DbgkpQueueMessage;        //1 处  
  ULONG DbgkClearProcessDebugObject\[1\];      //2 处  
  ULONG DbgkpSetProcessDebugObject\[4\];      //5 处  
  ULONG DbgkForwardException;      //1 处   
}str\_index,\*pstr\_index;  
BYTE debugport\_in\[2\]={0xBC,0x00};   //还原修改的函数  
BYTE debugPort\_out\[2\]={0x5C,0x02};  //将 DebugPort+bc 移至 DebugPort+25C  
#pragma PAGEDCODE  
VOID Hook\_DebugPort()  
{  
  /\*  
  1.枚举模块获取首地址  
  2.定位函数地址  
  3.修改函数  
  \*/  
  NTSTATUS status;  
  ULONG len;  
  PVOID Buffer=NULL;  
  PCHAR GamePath;  
  ULONG addr\_start;  
  ULONG addr\_size;  
  PSYSTEM\_MODULE\_INFORMATION enum\_module;  
  ZwQuerySystemInformation((SYSTEM\_INFORMATION\_CLASS)11,NULL,0,&len);  
  Buffer=ExAllocatePool(NonPagedPool,len);  
  if(Buffer==NULL)  
  {  
    KdPrint(("分配内存失败!\\r\\n"));  
    return ;  
  }  
  status=ZwQuerySystemInformation((SYSTEM\_INFORMATION\_CLASS)11,Buffer,len,NULL);  
  enum\_module=(PSYSTEM\_MODULE\_INFORMATION)Buffer;  
  addr\_start=(ULONG)enum\_module->Module\[0\].Base;  
  addr\_size=addr\_start+(ULONG)enum\_module->Module\[0\].Size;  
  ExFreePool(Buffer);  
  KdPrint(("模块名:%s   入口地址为:%x   模块结束地址:%x\\r\\n",enum\_module->Module\[0\].ImageName,addr\_start,addr\_size));  
  BYTE\* \_bp;  
  \_bp=(BYTE\*)addr\_start;  
  do   
  {  
    if((\*(\_bp-9)==0x64)&&(\*(\_bp-3)==0x8B)&&(\*(\_bp)==0x39)&&(\*(\_bp+6)==0x74)&&(\*(\_bp+8)==0x6A)&&(\*(\_bp+10)==0x8D))  
    {  
      str\_index.KiDispatchException=(ULONG)\_bp+2;  
      KdPrint(("KiDispatchException 访问 DebugPort 的地址为:%x\\n",str\_index.KiDispatchException));  
      break;  
    }  
    \_bp++;  
  } while (1);  
  \_bp=(BYTE\*)addr\_start;  
  do   
  {  
    if((\*(\_bp-19)==0xE8)&&(\*(\_bp-14)==0x85)&&(\*(\_bp-12)==0x0F)&&(\*(\_bp-6)==0x8B)&&(\*(\_bp)==0x8B)&&(\*(\_bp+6)==0xF7))  
    {  
      str\_index.NtQueryInformationProcess=(ULONG)\_bp+2;  
      KdPrint(("NtQueryInformationProecss 访问 DebugPort 的地址为:%x\\n",str\_index.NtQueryInformationProcess));  
      break;  
    }  
    \_bp++;  
  } while (1);  
  \_bp=(BYTE\*)addr\_start;  
  do   
  {  
    if((\*(\_bp-13)==0x8B)&&(\*(\_bp-11)==0x3B)&&(\*(\_bp-9)==0x0F)&&(\*(\_bp-3)==0x8B)&&(\*(\_bp)==0x89)&&(\*(\_bp+6)==0xF6)&&(\*(\_bp+10)==0x74))  
    {  
      str\_index.PspCreateProcess=(ULONG)\_bp+2;  
      KdPrint(("PspCreateProess 访问 DebugPort 的地址为:%x\\n",str\_index.PspCreateProcess));  
      break;  
    }  
    \_bp++;  
  } while (1);  
  \_bp=(BYTE\*)addr\_start;  
  do  
  {  
    if((\*(\_bp-5)==0x8B)&&(\*(\_bp-3)==0x8B)&&(\*(\_bp)==0x8B)&&(\*(\_bp+1)==0x80)&&(\*(\_bp+2)==0xBC)&&(\*(\_bp+6)==0x5D)&&(\*(\_bp+7)==0xC2)&&(\*(\_bp+8)==0x04))  
    {  
      str\_index.PsGetProcessDebugPort=(ULONG)\_bp+2;  
      KdPrint(("PsGetProcessDeubugPort 访问 DebugPort 的地址为:%x\\n",str\_index.PsGetProcessDebugPort));  
      break;  
    }  
    \_bp++;  
  }while(1);  
  \_bp=(BYTE\*)addr\_start;  
  do   
  {  
    if((\*(\_bp-5)==0x8B)&&(\*(\_bp-3)==0x8B)&&(\*(\_bp)==0x83)&&(\*(\_bp+1)==0xB8)&&(\*(\_bp+7)==0x0F)&&(\*(\_bp+10)==0x5D)&&(\*(\_bp+11)==0xC2))  
    {  
      str\_index.PsIsProcessBeingDebugged=(ULONG)\_bp+2;  
      KdPrint(("PsIsProcessBeingDebugged 访问 DebugPort 的地址为:%x\\n",str\_index.PsIsProcessBeingDebugged));  
      break;  
    }  
    \_bp++;  
  } while (1);  
  \_bp=(BYTE\*)addr\_start;  
  do   
  {  
    if((\*(\_bp-9)==0x81)&&(\*(\_bp-2)==0x74)&&(\*(\_bp)==0x83)&&(\*(\_bp+1)==0xBB)&&(\*(\_bp+2)==0xBC)&&(\*(\_bp+7)==0x74)&&(\*(\_bp+9)==0x80))  
    {  
      str\_index.NtTerminateProcess=(ULONG)\_bp+2;  
      KdPrint(("NtTerminateProecss 访问 DebugPort 的地址为:%x\\n",str\_index.NtTerminateProcess));  
      break;  
    }  
    \_bp++;  
  } while (1);  
  \_bp=(BYTE\*)addr\_start;  
  do   
  {  
    if((\*(\_bp-6)==0x8B)&&(\*(\_bp-4)==0x85)&&(\*(\_bp-2)==0x75)&&(\*(\_bp)==0x39)&&(\*(\_bp+1)==0x86)&&(\*(\_bp+2)==0xBC)&&(\*(\_bp+6)==0x74)&&(\*(\_bp+8)==0x56))  
    {  
      str\_index.PspTerminateProcess=(ULONG)\_bp+2;  
      KdPrint(("PspTerminateProecss 访问 DebugPort 的地址为:%x\\n",str\_index.PspTerminateProcess));  
      break;  
    }  
    \_bp++;  
  } while (1);  
  \_bp=(BYTE\*)addr\_start;  
  do   
  {  
    if((\*(\_bp-5)==0xE8)&&(\*(\_bp)==0x83)&&(\*(\_bp+1)==0xBF)&&(\*(\_bp+2)==0xBC)&&(\*(\_bp+7)==0x74)&&(\*(\_bp+9)==0xF6))  
    {  
      str\_index.PspExitThread=(ULONG)\_bp+2;  
      KdPrint(("PspExitThread 访问 DebugPort 的地址为:%x\\n",str\_index.PspExitThread));  
      break;  
    }  
    \_bp++;  
  } while (1);  
  \_bp=(BYTE\*)addr\_start;  
  do   
  {  
    if((\*(\_bp-8)==0x8B)&&(\*(\_bp-5)==0xE8)&&(\*(\_bp)==0x83)&&(\*(\_bp+1)==0xBF)&&(\*(\_bp+2)==0xBC)&&(\*(\_bp+7)==0x74)&&(\*(\_bp+9)==0xF6))  
    {  
      str\_index.PspProcessDelete\[0\]=(ULONG)\_bp+2;  
      KdPrint(("第一处 PspProcessDelete 访问 DebugPotr 的地址为:%x\\n",str\_index.PspProcessDelete\[0\]));  
      break;  
    }  
    \_bp++;  
  } while (1);  
  \_bp=(BYTE\*)addr\_start;  
  do   
  {  
    if((\*(\_bp-11)==0xE8)&&(\*(\_bp-6)==0x89)&&(\*(\_bp-4)==0x66)&&(\*(\_bp)==0x8D)&&(\*(\_bp+1)==0xBE)&&(\*(\_bp+2)==0xBC)&&(\*(\_bp+6)==0x8B))  
    {  
      str\_index.PspProcessDelete\[1\]=(ULONG)\_bp+2;  
      KdPrint(("第二处 PspProcessDelete 访问 DebugPotr 的地址为:%x\\n",str\_index.PspProcessDelete\[1\]));  
      break;  
    }  
    \_bp++;  
  } while (1);  
  \_bp=(BYTE\*)addr\_start;  
  do   
  {  
    if((\*(\_bp-9)==0x81)&&(\*(\_bp-2)==0x74)&&(\*(\_bp)==0x83)&&(\*(\_bp+1)==0xBB)&&(\*(\_bp+2)==0xBC)&&(\*(\_bp+7)==0x74))  
    {  
      str\_index.PspProcessDelete\[2\]=(ULONG)\_bp+2;  
      KdPrint(("第三处 PspProcessDelete 访问 DebugPotr 的地址为:%x\\n",str\_index.PspProcessDelete\[2\]));  
      break;  
    }  
    \_bp++;  
  } while (1);  
  \_bp=(BYTE\*)addr\_start;  
  do   
  {  
    if((\*(\_bp-9)==0x64)&&(\*(\_bp-3)==0x8B)&&(\*(\_bp)==0x83)&&(\*(\_bp+1)==0xB8)&&(\*(\_bp+2)==0xBC)&&(\*(\_bp+7)==0x74))  
    {  
      str\_index.ObpCloseHandleTableEntry=(ULONG)\_bp+2;  
      KdPrint(("ObpCloseHandleTableEntry 访问 DebugPort 的地址为:%x\\n",str\_index.ObpCloseHandleTableEntry));  
      break;  
    }  
    \_bp++;  
  } while (1);  
  \_bp=(BYTE\*)addr\_start;  
  do   
  {  
    if((\*(\_bp-9)==0xF6)&&(\*(\_bp-2)==0x75)&&(\*(\_bp)==0x39)&&(\*(\_bp+1)==0x8B)&&(\*(\_bp+2)==0xBC)&&(\*(\_bp+6)==0x74))  
    {  
      str\_index.ObpCloseHandle=(ULONG)\_bp+2;  
      KdPrint(("ObpCloseHandle 访问 DebugPort 的地址为:%x\\n",str\_index.ObpCloseHandle));  
      break;  
    }  
    \_bp++;  
  } while (1);  
  \_bp=(BYTE\*)addr\_start;  
  do   
  {  
    if((\*(\_bp-8)==0x8B)&&(\*(\_bp-5)==0x89)&&(\*(\_bp-2)==0x33)&&(\*(\_bp)==0x39)&&(\*(\_bp+1)==0x9E)&&(\*(\_bp+6)==0x0F))  
    {  
      str\_index.MmCreatePeb=(ULONG)\_bp+2;  
      KdPrint(("MmCreatePeb 访问 DebugPort 的地址为:%x\\n",str\_index.MmCreatePeb));  
      break;  
    }  
    \_bp++;  
  } while (1);  
  \_bp=(BYTE\*)addr\_start;  
  do   
  {  
    if((\*(\_bp-6)==0x50)&&(\*(\_bp-5)==0xE8)&&(\*(\_bp)==0x39)&&(\*(\_bp+1)==0x9E)&&(\*(\_bp+2)==0xBC)&&(\*(\_bp+6)==0x0F)&&(\*(\_bp+12)==0x39))  
    {  
      str\_index.DbgkCreateThread=(ULONG)\_bp+2;  
      KdPrint(("DbgkCreateThread 访问 DebugPort 的地址为:%x\\n",str\_index.DbgkCreateThread));  
      break;  
    }  
    \_bp++;  
  } while (1);  
  \_bp=(BYTE\*)addr\_start;  
  do   
  {  
    if((\*(\_bp-9)==0xF6)&&(\*(\_bp-2)==0x75)&&(\*(\_bp)==0x8B)&&(\*(\_bp+1)==0x89)&&(\*(\_bp+2)==0xBC)&&(\*(\_bp+6)==0x85))  
    {  
      str\_index.DbgkExitThread=(ULONG)\_bp+2;  
      KdPrint(("DbgkExitThread 访问 DebugPort 的地址为:%x\\n",str\_index.DbgkExitThread));  
      break;  
    }  
    \_bp++;  
  } while (1);  
  \_bp=(BYTE\*)str\_index.DbgkExitThread+10;  
  do   
  {  
    if((\*(\_bp-9)==0xF6)&&(\*(\_bp-2)==0x75)&&(\*(\_bp)==0x8B)&&(\*(\_bp+1)==0x89)&&(\*(\_bp+2)==0xBC)&&(\*(\_bp+6)==0x85))  
    {  
      str\_index.DbgkExitProcess=(ULONG)\_bp+2;  
      KdPrint(("DbgkExitProcess 访问 DebugProt 的地址为:%x\\n",str\_index.DbgkExitProcess));  
      break;  
    }  
    \_bp++;  
  } while (1);  
  \_bp=(BYTE\*)addr\_start;  
  do   
  {  
    if((\*(\_bp-6)==0x74)&&(\*(\_bp-4)==0x33)&&(\*(\_bp-2)==0xEB)&&(\*(\_bp)==0x8B)&&(\*(\_bp+1)==0x89)&&(\*(\_bp+6)==0x3B))  
    {  
      str\_index.DbgkMapViewOfSection=(ULONG)\_bp+2;  
      KdPrint(("DbgkMapViewOfSection 访问 DebugPort 的地址为:%x\\n",str\_index.DbgkMapViewOfSection));  
      break;  
    }  
    \_bp++;  
  } while (1);  
  \_bp=(BYTE\*)addr\_start;  
  do   
  {  
    if((\*(\_bp-9)==0xF6)&&(\*(\_bp-2)==0x75)&&(\*(\_bp)==0x83)&&(\*(\_bp+1)==0xB9)&&(\*(\_bp+2)==0xBC)&&(\*(\_bp+7)==0x74))  
    {  
      str\_index.DbgkUnMapViewOfSection=(ULONG)\_bp+2;  
      KdPrint(("DbgkUnMapViewOfSection 访问 DebugPort 的地址为:%x\\n",str\_index.DbgkUnMapViewOfSection));  
      break;  
    }  
    \_bp++;  
  } while (1);  
  \_bp=(BYTE\*)addr\_start;  
  do   
  {  
    if((\*(\_bp-11)==0xFF)&&(\*(\_bp-5)==0x89)&&(\*(\_bp-2)==0x33)&&(\*(\_bp)==0x39)&&(\*(\_bp+1)==0xBE)&&(\*(\_bp+6)==0x0F)&&(\*(\_bp+9)==0x8B))  
    {  
      str\_index.DbgkpMarkProcessPeb=(ULONG)\_bp+2;  
      KdPrint(("DbgkpMarkProcessPeb 访问 DebugPort 的地址为:%x\\n",str\_index.DbgkpMarkProcessPeb));  
      break;  
    }  
    \_bp++;  
  } while (1);  
  \_bp=(BYTE\*)addr\_start;  
  do   
  {  
    if((\*(\_bp-5)==0xEB)&&(\*(\_bp-3)==0x8B)&&(\*(\_bp)==0x81)&&(\*(\_bp+1)==0xC3)&&(\*(\_bp+2)==0xBC)&&(\*(\_bp+6)==0x39)&&(\*(\_bp+8)==0x75))  
    {  
      str\_index.DbgkpCloseObject\[0\]=(ULONG)\_bp+2;  
      KdPrint(("第一个 DbgkpCloseObject 访问 DeBugPort 的地址为:%x\\n",str\_index.DbgkpCloseObject\[0\]));  
      break;  
    }  
    \_bp++;  
  } while (1);  
  \_bp=(BYTE\*)addr\_start;  
  do   
  {  
    if((\*(\_bp-5)==0xFF)&&(\*(\_bp-3)==0x8B)&&(\*(\_bp)==0x05)&&(\*(\_bp+1)==0xBC)&&(\*(\_bp+5)==0x8B)&&(\*(\_bp+7)==0x33))  
    {  
      str\_index.DbgkpCloseObject\[1\]=(ULONG)\_bp+1;  
      KdPrint(("第二个 DbgkpCloseObject 访问 DebugPort 的地址为:%x\\n",str\_index.DbgkpCloseObject\[1\]));  
      break;  
    }  
    \_bp++;  
  } while (1);  
  \_bp=(BYTE\*)addr\_start;  
  do  
  {  
    if((\*(\_bp-6)==0xC6)&&(\*(\_bp-2)==0xFF)&&(\*(\_bp)==0x39)&&(\*(\_bp+1)==0x9F)&&(\*(\_bp+6)==0x75)&&(\*(\_bp+8)==0x8B))  
    {  
      str\_index.DbgkpCloseObject\[2\]=(ULONG)\_bp+2;  
      KdPrint(("第三个 DbgkpCloseObject 访问 DebugPort 的地址为:%x\\n",str\_index.DbgkpCloseObject\[2\]));  
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
      KdPrint(("第一个 DbgkCopyProcessDebugPort 访问 DebugPort 的地址为:%x\\n",str\_index.DbgkCopyProcessDebugPort\[0\]));  
      break;  
    }  
    \_bp++;  
  } while (1);  
  \_bp=(BYTE\*)addr\_start;  
  do   
  {  
    if((\*(\_bp-4)==0x56)&&(\*(\_bp-3)==0x8B)&&(\*(\_bp)==0x81)&&(\*(\_bp+1)==0xC6)&&(\*(\_bp+2)=0xBC)&&(\*(\_bp+6)==0x83)&&(\*(\_bp+9)==0x74))  
    {  
      str\_index.DbgkCopyProcessDebugPort\[1\]=(ULONG)\_bp+2;  
      KdPrint(("第二个 DbgkCopyProcessDebugPort 访问 DebugPort 的地址为:%x\\n",str\_index.DbgkCopyProcessDebugPort\[1\]));  
      break;  
    }  
    \_bp++;  
  } while (1);  
  \_bp=(BYTE\*)addr\_start;  
  do   
  {  
    if((\*(\_bp-8)==0xE8)&&(\*(\_bp-3)==0x8B)&&(\*(\_bp)==0x89)&&(\*(\_bp+1)==0xB0)&&(\*(\_bp+2)==0xBC)&&(\*(\_bp+6)==0x8B))  
    {  
      str\_index.DbgkCopyProcessDebugPort\[2\]=(ULONG)\_bp+2;  
      KdPrint(("第三个 DbgkCopyProcessDebugPort 访问 DebugPort 的地址为:%x\\n",str\_index.DbgkCopyProcessDebugPort\[2\]));  
      break;  
    }  
    \_bp++;  
  } while (1);  
  \_bp=(BYTE\*)addr\_start;  
  do   
  {  
    if((\*(\_bp-5)==0x51)&&(\*(\_bp-4)==0x56)&&(\*(\_bp-3)==0x8B)&&(\*(\_bp)==0x81)&&(\*(\_bp+1)==0xC6)&&(\*(\_bp+6)==0x83)&&(\*(\_bp+9)==0xC7))  
    {  
      str\_index.DbgkCopyProcessDebugPort\[3\]=(ULONG)\_bp+2;  
      KdPrint(("第四个 DbgkCopyProcessDebugPort 访问 DebugPort 的地址为:%x\\n",str\_index.DbgkCopyProcessDebugPort\[3\]));  
      break;  
    }  
    \_bp++;  
  } while (1);  
  \_bp=(BYTE\*)addr\_start;  
  do   
  {  
    if((\*(\_bp-5)==0x51)&&(\*(\_bp-4)==0x56)&&(\*(\_bp-3)==0x8B)&&(\*(\_bp)==0x81)&&(\*(\_bp+1)==0xC6)&&(\*(\_bp+6)==0x83)&&(\*(\_bp+9)==0xC7))  
    {  
      str\_index.DbgkOpenProcessDebugPort\[0\]=(ULONG)\_bp+2;  
      KdPrint(("第一个 DbgkOpenProcessDebugPort 的访问地址为:%x\\n",str\_index.DbgkOpenProcessDebugPort\[0\]));  
      break;  
    }  
    \_bp++;  
  } while (1);  
  \_bp=(BYTE\*)addr\_start;  
  do   
  {  
    if((\*(\_bp-9)==0xFF)&&(\*(\_bp-3)==0x8B)&&(\*(\_bp)==0x8B)&&(\*(\_bp+1)==0x80)&&(\*(\_bp+6)==0x89)&&(\*(\_bp+9)==0x8B))  
    {  
      str\_index.DbgkOpenProcessDebugPort\[1\]=(ULONG)\_bp+2;  
      KdPrint(("第二个 DbgkOpenProcessDebugPort 访问 DebugPort 的地址为:%x\\n",str\_index.DbgkOpenProcessDebugPort\[1\]));  
      break;  
    }  
    \_bp++;  
  } while (1);  
  \_bp=(BYTE\*)addr\_start;  
  do   
  {  
    if((\*(\_bp-9)==0xFF)&&(\*(\_bp-3)==0x8B)&&(\*(\_bp)==0x8B)&&(\*(\_bp+1)==0x80)&&(\*(\_bp+6)==0x89)&&(\*(\_bp+9)==0x8B))  
    {  
      str\_index.DbgkpQueueMessage=(ULONG)\_bp+2;  
      KdPrint(("DbgkpQueueMessage 访问 DebugPort 的地址为:%x\\n",str\_index.DbgkpQueueMessage));  
      break;  
    }  
    \_bp++;  
  } while (1);  
  \_bp=(BYTE\*)addr\_start;  
  do   
  {  
    if((\*(\_bp-5)==0xFF)&&(\*(\_bp-3)==0x8B)&&(\*(\_bp)==0x05)&&(\*(\_bp+1)==0xBC)&&(\*(\_bp+5)==0x8B)&&(\*(\_bp+7)==0x33))  
    {  
      str\_index.DbgkClearProcessDebugObject\[0\]=(ULONG)\_bp+1;  
      KdPrint(("DbgkClearProcessDebugObject 访问 DebugPort 的地址为:%x\\n",str\_index.DbgkClearProcessDebugObject\[0\]));  
      break;  
    }  
    \_bp++;  
  } while (1);  
  \_bp=(BYTE\*)addr\_start;  
  do   
  {  
    if((\*(\_bp-6)==0xC6)&&(\*(\_bp-2)==0xFF)&&(\*(\_bp)==0x39)&&(\*(\_bp+1)==0x9F)&&(\*(\_bp+6)==0x75)&&(\*(\_bp+8)==0x8B))  
    {  
      str\_index.DbgkClearProcessDebugObject\[1\]=(ULONG)\_bp+2;  
      KdPrint(("DbgkClearProcessDebugObject 访问 DebugPort 的地址为:%x\\n",str\_index.DbgkClearProcessDebugObject\[1\]));  
      break;  
    }  
    \_bp++;  
  } while (1);  
  \_bp=(BYTE\*)addr\_start;  
  do   
  {  
    if((\*(\_bp-6)==0xC6)&&(\*(\_bp-2)==0xFF)&&(\*(\_bp)==0x39)&&(\*(\_bp+1)==0x9F)&&(\*(\_bp+6)==0x75)&&(\*(\_bp+8)==0x8B))  
    {  
      str\_index.DbgkpSetProcessDebugObject\[0\]=(ULONG)\_bp+2;  
      KdPrint(("第一个 DbgkpSetProcessDebugObject 访问 DebugPort 的地址为:%x\\n",str\_index.DbgkpSetProcessDebugObject\[0\]));  
      break;  
    }  
    \_bp++;  
  } while (1);  
  \_bp=(BYTE\*)addr\_start;  
  do   
  {  
    if((\*(\_bp-6)==0x8B)&&(\*(\_bp-3)==0x8B)&&(\*(\_bp)==0x89)&&(\*(\_bp+1)==0x87)&&(\*(\_bp+6)==0xE8)&&(\*(\_bp+11)==0xFF))  
    {  
      str\_index.DbgkpSetProcessDebugObject\[1\]=(ULONG)\_bp+2;  
      KdPrint(("第二个 DbgkpSetProcessDebugObject 访问 DebugPort 的地址为:%x\\n",str\_index.DbgkpSetProcessDebugObject\[1\]));  
      break;  
    }  
    \_bp++;  
  } while (1);  
  \_bp=(BYTE\*)addr\_start;  
  do   
  {  
    if((\*(\_bp-6)==0x8B)&&(\*(\_bp-4)==0x85)&&(\*(\_bp-2)==0x74)&&(\*(\_bp)==0x83)&&(\*(\_bp+1)==0xA7)&&(\*(\_bp+7)==0xB9))  
    {  
      str\_index.DbgkpSetProcessDebugObject\[2\]=(ULONG)\_bp+2;  
      KdPrint(("第三个 DbgkpSetProcessDebugObject 访问 DebugPort 的地址为:%x\\n",str\_index.DbgkpSetProcessDebugObject\[2\]));  
      break;  
    }  
    \_bp++;  
  } while (1);  
  \_bp=(BYTE\*)addr\_start;  
  do   
  {  
    if((\*(\_bp-6)==0xC6)&&(\*(\_bp-2)==0xFF)&&(\*(\_bp)==0x83)&&(\*(\_bp+1)==0xBF)&&(\*(\_bp+7)==0x74)&&(\*(\_bp+9)==0xC7))  
    {  
      str\_index.DbgkpSetProcessDebugObject\[3\]=(ULONG)\_bp+2;  
      KdPrint(("第四个 DbgkpSetProcessDebugObject 访问 DebugPort 的地址为:%x\\n",str\_index.DbgkpSetProcessDebugObject\[3\]));  
      break;  
    }  
    \_bp++;  
  } while (1);  
  \_bp=(BYTE\*)addr\_start;  
  do   
  {  
    if((\*(\_bp-7)==0xE8)&&(\*(\_bp-2)==0xEB)&&(\*(\_bp)==0x83)&&(\*(\_bp+1)==0xA7)&&(\*(\_bp+7)==0xC7))  
    {  
      str\_index.DbgkpSetProcessDebugObject\[4\]=(ULONG)\_bp+2;  
      KdPrint(("第五个 DbgkpSetProcessDebugObject 访问 DebugPort 的地址为:%x\\n",str\_index.DbgkpSetProcessDebugObject\[4\]));  
      break;  
    }  
    \_bp++;  
  } while (1);  
  \_bp=(BYTE\*)addr\_start;  
  do   
  {  
    if((\*(\_bp-6)==0x74)&&(\*(\_bp-4)==0x33)&&(\*(\_bp-2)==0xEB)&&(\*(\_bp)==0x8B)&&(\*(\_bp+1)==0x81)&&(\*(\_bp+6)==0x32)&&(\*(\_bp+8)==0xEB))  
    {  
      str\_index.DbgkForwardException=(ULONG)\_bp+2;  
      KdPrint(("DbgkForwardException 访问 DeubugPort 的地址为:%x\\n",str\_index.DbgkForwardException));  
      break;  
    }  
    \_bp++;  
  } while (1);  
  PAGED\_Open();  
  RtlCopyMemory((PVOID)str\_index.KiDispatchException,debugPort\_out,2);  
  RtlCopyMemory((PVOID)str\_index.NtQueryInformationProcess,debugPort\_out,2);  
  RtlCopyMemory((PVOID)str\_index.PspCreateProcess,debugPort\_out,2);  
  RtlCopyMemory((PVOID)str\_index.PsGetProcessDebugPort,debugPort\_out,2);  
  RtlCopyMemory((PVOID)str\_index.PsIsProcessBeingDebugged,debugPort\_out,2);  
  RtlCopyMemory((PVOID)str\_index.NtTerminateProcess,debugPort\_out,2);  
  RtlCopyMemory((PVOID)str\_index.PspTerminateProcess,debugPort\_out,2);  
  RtlCopyMemory((PVOID)str\_index.PspExitThread,debugPort\_out,2);  
  RtlCopyMemory((PVOID)str\_index.PspProcessDelete\[0\],debugPort\_out,2);  
  RtlCopyMemory((PVOID)str\_index.PspProcessDelete\[1\],debugPort\_out,2);  
  RtlCopyMemory((PVOID)str\_index.PspProcessDelete\[2\],debugPort\_out,2);  
  RtlCopyMemory((PVOID)str\_index.ObpCloseHandleTableEntry,debugPort\_out,2);  
  RtlCopyMemory((PVOID)str\_index.ObpCloseHandle,debugPort\_out,2);  
  RtlCopyMemory((PVOID)str\_index.MmCreatePeb,debugPort\_out,2);  
  RtlCopyMemory((PVOID)str\_index.DbgkCreateThread,debugPort\_out,2);  
  RtlCopyMemory((PVOID)str\_index.DbgkExitThread,debugPort\_out,2);  
  RtlCopyMemory((PVOID)str\_index.DbgkExitProcess,debugPort\_out,2);  
  RtlCopyMemory((PVOID)str\_index.DbgkMapViewOfSection,debugPort\_out,2);  
  RtlCopyMemory((PVOID)str\_index.DbgkUnMapViewOfSection,debugPort\_out,2);  
  RtlCopyMemory((PVOID)str\_index.DbgkpMarkProcessPeb,debugPort\_out,2);  
  RtlCopyMemory((PVOID)str\_index.DbgkpCloseObject\[0\],debugPort\_out,2);  
  RtlCopyMemory((PVOID)str\_index.DbgkpCloseObject\[1\],debugPort\_out,2);  
  RtlCopyMemory((PVOID)str\_index.DbgkpCloseObject\[2\],debugPort\_out,2);  
  RtlCopyMemory((PVOID)str\_index.DbgkCopyProcessDebugPort\[0\],debugPort\_out,2);  
  RtlCopyMemory((PVOID)str\_index.DbgkCopyProcessDebugPort\[1\],debugPort\_out,2);  
  RtlCopyMemory((PVOID)str\_index.DbgkCopyProcessDebugPort\[2\],debugPort\_out,2);  
  RtlCopyMemory((PVOID)str\_index.DbgkCopyProcessDebugPort\[3\],debugPort\_out,2);  
  RtlCopyMemory((PVOID)str\_index.DbgkOpenProcessDebugPort\[0\],debugPort\_out,2);  
  RtlCopyMemory((PVOID)str\_index.DbgkOpenProcessDebugPort\[1\],debugPort\_out,2);  
  RtlCopyMemory((PVOID)str\_index.DbgkpQueueMessage,debugPort\_out,2);  
  RtlCopyMemory((PVOID)str\_index.DbgkClearProcessDebugObject\[0\],debugPort\_out,2);  
  RtlCopyMemory((PVOID)str\_index.DbgkClearProcessDebugObject\[1\],debugPort\_out,2);  
  RtlCopyMemory((PVOID)str\_index.DbgkpSetProcessDebugObject\[0\],debugPort\_out,2);  
  RtlCopyMemory((PVOID)str\_index.DbgkpSetProcessDebugObject\[1\],debugPort\_out,2);  
  RtlCopyMemory((PVOID)str\_index.DbgkpSetProcessDebugObject\[2\],debugPort\_out,2);  
  RtlCopyMemory((PVOID)str\_index.DbgkpSetProcessDebugObject\[3\],debugPort\_out,2);  
  RtlCopyMemory((PVOID)str\_index.DbgkpSetProcessDebugObject\[4\],debugPort\_out,2);  
  RtlCopyMemory((PVOID)str\_index.DbgkForwardException,debugPort\_out,2);  
  PAGED\_Exit();  
}  
  
#pragma PAGEDCODE  
VOID Un\_DebugPort()  
{  
  PAGED\_Open();  
  RtlCopyMemory((PVOID)str\_index.KiDispatchException,debugport\_in,2);  
  RtlCopyMemory((PVOID)str\_index.NtQueryInformationProcess,debugport\_in,2);  
  RtlCopyMemory((PVOID)str\_index.PspCreateProcess,debugport\_in,2);  
  RtlCopyMemory((PVOID)str\_index.PsGetProcessDebugPort,debugport\_in,2);  
  RtlCopyMemory((PVOID)str\_index.PsIsProcessBeingDebugged,debugport\_in,2);  
  RtlCopyMemory((PVOID)str\_index.NtTerminateProcess,debugport\_in,2);  
  RtlCopyMemory((PVOID)str\_index.PspTerminateProcess,debugport\_in,2);  
  RtlCopyMemory((PVOID)str\_index.PspExitThread,debugport\_in,2);  
  RtlCopyMemory((PVOID)str\_index.PspProcessDelete\[0\],debugport\_in,2);  
  RtlCopyMemory((PVOID)str\_index.PspProcessDelete\[1\],debugport\_in,2);  
  RtlCopyMemory((PVOID)str\_index.PspProcessDelete\[2\],debugport\_in,2);  
  RtlCopyMemory((PVOID)str\_index.ObpCloseHandleTableEntry,debugport\_in,2);  
  RtlCopyMemory((PVOID)str\_index.ObpCloseHandle,debugport\_in,2);  
  RtlCopyMemory((PVOID)str\_index.MmCreatePeb,debugport\_in,2);  
  RtlCopyMemory((PVOID)str\_index.DbgkCreateThread,debugport\_in,2);  
  RtlCopyMemory((PVOID)str\_index.DbgkExitThread,debugport\_in,2);  
  RtlCopyMemory((PVOID)str\_index.DbgkExitProcess,debugport\_in,2);  
  RtlCopyMemory((PVOID)str\_index.DbgkMapViewOfSection,debugport\_in,2);  
  RtlCopyMemory((PVOID)str\_index.DbgkUnMapViewOfSection,debugport\_in,2);  
  RtlCopyMemory((PVOID)str\_index.DbgkpMarkProcessPeb,debugport\_in,2);  
  RtlCopyMemory((PVOID)str\_index.DbgkpCloseObject\[0\],debugport\_in,2);  
  RtlCopyMemory((PVOID)str\_index.DbgkpCloseObject\[1\],debugport\_in,2);  
  RtlCopyMemory((PVOID)str\_index.DbgkpCloseObject\[2\],debugport\_in,2);  
  RtlCopyMemory((PVOID)str\_index.DbgkCopyProcessDebugPort\[0\],debugport\_in,2);  
  RtlCopyMemory((PVOID)str\_index.DbgkCopyProcessDebugPort\[1\],debugport\_in,2);  
  RtlCopyMemory((PVOID)str\_index.DbgkCopyProcessDebugPort\[2\],debugport\_in,2);  
  RtlCopyMemory((PVOID)str\_index.DbgkCopyProcessDebugPort\[3\],debugport\_in,2);  
  RtlCopyMemory((PVOID)str\_index.DbgkOpenProcessDebugPort\[0\],debugport\_in,2);  
  RtlCopyMemory((PVOID)str\_index.DbgkOpenProcessDebugPort\[1\],debugport\_in,2);  
  RtlCopyMemory((PVOID)str\_index.DbgkpQueueMessage,debugport\_in,2);  
  RtlCopyMemory((PVOID)str\_index.DbgkClearProcessDebugObject\[0\],debugport\_in,2);  
  RtlCopyMemory((PVOID)str\_index.DbgkClearProcessDebugObject\[1\],debugport\_in,2);  
  RtlCopyMemory((PVOID)str\_index.DbgkpSetProcessDebugObject\[0\],debugport\_in,2);  
  RtlCopyMemory((PVOID)str\_index.DbgkpSetProcessDebugObject\[1\],debugport\_in,2);  
  RtlCopyMemory((PVOID)str\_index.DbgkpSetProcessDebugObject\[2\],debugport\_in,2);  
  RtlCopyMemory((PVOID)str\_index.DbgkpSetProcessDebugObject\[3\],debugport\_in,2);  
  RtlCopyMemory((PVOID)str\_index.DbgkpSetProcessDebugObject\[4\],debugport\_in,2);  
  RtlCopyMemory((PVOID)str\_index.DbgkForwardException,debugport\_in,2);  
  PAGED\_Exit();  
}  
  
#pragma INITCODE  
VOID Hook\_NtSetInformationThread()  
{  
  addr\_NtSetInformationThread=(ULONG)Getsset\_this(in\_NtSetInformationThread);  
  addr1\_NtSetInformationThread=(ULONG)Getssdt\_addr(in\_NtSetInformationThread);  
  \_NtSetInformationThread=(NtSetInformationThread\*)addr1\_NtSetInformationThread;  
  PAGED\_Open();  
  \*((ULONG\*)addr\_NtSetInformationThread)=(ULONG)My\_NtSetInformationThread;  
  PAGED\_Exit();  
}  
  
#pragma PAGEDCODE  
extern "C"  
NTSTATUS \_\_stdcall My\_NtSetInformationThread  ( \_\_in HANDLE  ThreadHandle,    
                        \_\_in THREADINFOCLASS  ThreadInformationClass,    
                        \_\_in\_bcount(ThreadInformationLength) PVOID  ThreadInformation,    
                        \_\_in ULONG  ThreadInformationLength     
                           )   
{  
  if(ThreadInformationClass==0x11)  
  {  
    KdPrint(("发现 ANTI 行为!\\r\\n"));  
    KdPrint(("NtSetInformationThread 的参数:%8x %8x %8x %8x",ThreadHandle,ThreadInformationClass,ThreadInformation,ThreadInformationLength));  
    ThreadInformationClass=(THREADINFOCLASS)0xFF;  
    KdPrint(("NtSetInformationThread 的修改后的参数:%8x %8x %8x %8x",ThreadHandle,ThreadInformationClass,ThreadInformation,ThreadInformationLength));  
    return STATUS\_SUCCESS;  
  }  
  return ((NtSetInformationThread)\_NtSetInformationThread)(ThreadHandle,ThreadInformationClass,ThreadInformation,ThreadInformationLength);  
}  
  
#pragma PAGEDCODE  
VOID Un\_NtSetInnformationThread()  
{  
  PAGED\_Open();  
  \*((ULONG\*)addr\_NtSetInformationThread)=addr1\_NtSetInformationThread;  
  PAGED\_Exit();  
}  
  
#endif