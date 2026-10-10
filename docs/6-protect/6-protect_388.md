

# 修改访问 DebugPort 的函数偏移。

> 原文：[`www.pediy.com/kssd/pediy12/139302.html`](https://www.pediy.com/kssd/pediy12/139302.html)

//重在思路。  
  

代码:

```
#include <ntddk.h>
#include <windef.h>

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
} SYSTEM\_INFORMATION\_CLASS;
typedef struct \_SYSTEM\_MODULE\_INFORMATION { 
ULONG Reserved\[2\];
PVOID Base;
ULONG Size;
ULONG Flags;
USHORT Index;
USHORT Unknown;
USHORT LoadCount;
USHORT ModuleNameOffset;
CHAR ImageName\[256\];
} SYSTEM\_MODULE\_INFORMATION, \*PSYSTEM\_MODULE\_INFORMATION;

#define DELAY\_ONE\_MICROSECOND (-10)
#define DELAY\_ONE\_MILLISECOND (DELAY\_ONE\_MICROSECOND\*1000)

//全局量
UCHAR OrigCode\[2\]={0xBC,0x00};
UCHAR EditCode\[2\]={0x5C,0x02};

ULONG DbgkpSetProcessDebugObject;
ULONG PspProcessDelete;
ULONG DbgkForwardException;
ULONG DbgkpQueueMessage;
ULONG DbgkpMarkProcessPeb;
ULONG DbgkExitProcess;
ULONG DbgkExitThread;
ULONG DbgkCreateThread;
ULONG PspExitThread;
ULONG NtTerminateProcess;
ULONG PsIsProcessBeingDebugged;
ULONG PsGetProcessDebugPort;
ULONG NtQueryInformationPro;     //这个函数居然有导出
ULONG KiDispatchException;

//以下函数测试仅仅是附加没使用
ULONG PspCreateProcess;        //1 处
ULONG PspTerminateProcess;      //1 处
ULONG ObpCloseHandleTableEntry;      //1 处
ULONG ObpCloseHandle;        //1 处
ULONG MmCreatePeb;        //1 处
ULONG DbgkMapViewOfSection;      //1 处
ULONG DbgkUnMapViewOfSection;      //1 处
ULONG DbgkpCloseObject;        //3 处
ULONG DbgkCopyProcessDebugPort;      //4 处  这里和下面的函数相差无几 注意
ULONG DbgkOpenProcessDebugPort;      //2 处      
ULONG DbgkClearProcessDebugObject;


//Offset
//DbgkpSetProcessDebugObject
ULONG DbgkpSetProcessDebugObject\_Offset1;
ULONG DbgkpSetProcessDebugObject\_Offset2;
ULONG DbgkpSetProcessDebugObject\_Offset3;
ULONG DbgkpSetProcessDebugObject\_Offset4;
//PspProcessDelete
ULONG PspProcessDelete\_Offset1;
ULONG PspProcessDelete\_Offset2;
ULONG PspProcessDelete\_Offset3;

//DbgkForwardException
ULONG DbgkForwardException\_Offset1;
//DbgkpQueueMessage
ULONG DbgkpQueueMessage\_Offset1;
//DbgkpMarkProcessPeb
ULONG DbgkpMarkProcessPeb\_Offset1;
ULONG DbgkExitProcess\_Offset1;
ULONG DbgkExitThread\_Offset1;
ULONG DbgkCreateThread\_Offset1;
ULONG PspExitThread\_Offset1;
ULONG NtTerminateProcess\_Offset1;
ULONG PsIsProcessBeingDebugged\_Offset1;
ULONG PsGetProcessDebugPort\_Offset1;
ULONG NtQueryInformationPro\_Offset1;
ULONG KiDispatchException\_Offset1;
//以下函数偏移同上

ULONG PspCreateProcess\_Offset1;       //1 处
ULONG PspTerminateProcess\_Offset1;      //1 处
ULONG ObpCloseHandleTableEntry\_Offset1;      //1 处
ULONG ObpCloseHandle\_Offset1;        //1 处
ULONG MmCreatePeb\_Offset1;        //1 处
ULONG DbgkMapViewOfSection\_Offset1;      //1 处
ULONG DbgkUnMapViewOfSection\_Offset1;      //1 处

ULONG DbgkpCloseObject\_Offset1;        //3 处
ULONG DbgkpCloseObject\_Offset2;
ULONG DbgkpCloseObject\_Offset3;

ULONG DbgkCopyProcessDebugPort\_Offset1;      //4 处
ULONG DbgkCopyProcessDebugPort\_Offset2;
ULONG DbgkCopyProcessDebugPort\_Offset3;
ULONG DbgkCopyProcessDebugPort\_Offset4;

ULONG DbgkOpenProcessDebugPort\_Offset1;      //2 处  
ULONG DbgkOpenProcessDebugPort\_Offset2;
ULONG DbgkClearProcessDebugObject\_Offset1; //2
ULONG DbgkClearProcessDebugObject\_Offset2;







//API 申明
VOID StartInLineHook();
VOID Unload (IN PDRIVER\_OBJECT DriverObject);
VOID WPOFF(VOID);
VOID WPON(VOID);
PVOID GetUndocumentFunctionAdress();
NTSYSAPI
NTSTATUS
NTAPI
ZwQuerySystemInformation(
IN SYSTEM\_INFORMATION\_CLASS SystemInformationClass,
IN OUT PVOID SystemInformation,
IN ULONG SystemInformationLength,
OUT PULONG ReturnLength OPTIONAL
);

//获取未知函数地址
PVOID
GetUndocumentFunctionAdress()
{
  ULONG size,index;
  PULONG buf;
    ULONG i;
  PSYSTEM\_MODULE\_INFORMATION module;
  ULONG Modcnt=0;
     ULONG ntosknlBase;
  ULONG ntosknlEndAddr;
  ULONG curAddr;
  NTSTATUS status;

  //DbgkpSetProcessDebugObject
  ULONG  code1\_3=0x5318ec83,code2\_3=0xa1645756,code3\_3=0x00000124,code4\_3=0x8df84589;
  //DbgkForwardException
  ULONG  c1\_Fption=0x8378ec83,c2\_Fption=0x5300a065,c3\_Fption=0x5c8845c7,c4\_Fption=0xc7007800;
  //DbgkpQueueMessage
  ULONG  c1\_QuMess=0x00b8ec81,c2\_QuMess=0x56530000,c3\_QuMess=0x8914758b,c4\_QuMess=0x6583fc75;
  //PspProcessDelete
  ULONG  c1\_PsPrDe=0x5318ec83,c2\_PsPrDe=0x08758b56,c3\_PsPrDe=0x9e39db33,c4\_PsPrDe=0x00000088;
  //DbgkpMarkProcessPeb
  ULONG  c1\_DbMarP=0x6974c084,c2\_DbMarP=0xbe39ff33,c3\_DbMarP=0x000001b0,c4\_DbMarP=0x458d5474;
  //DbgkExitProcess------------DbgkExitThread
  ULONG  c1\_DbExPr=0x6478ec83,c2\_DbExPr=0x000124a1,c3\_DbExPr=0x44488b00,c4\_DbExPr=0x0124a164,c5\_DbExPr=0x80f60000;
  //DbgkCreateThread
  ULONG  c1\_DbCrTh=0x0124a164,c2\_DbCrTh=0x45890000,c3\_DbCrTh=0x44708bc4,c4\_DbCrTh=0x33d07589;
  //PspExitThread
  ULONG  c1\_PsExTh=0x0124a164,c2\_PsExTh=0xf08b0000,c3\_PsExTh=0x8bb87589,c4\_PsExTh=0x000220be;
  //NtTerminateProcess
  ULONG  c1\_NtTePr=0x5310ec83,c2\_NtTePr=0xa1645756,c3\_NtTePr=0x00000124,c4\_NtTePr=0x00087d83;
  //PsIsProcessBeingDebugged
  ULONG  c1\_PsPrBd=0x8308458b,c2\_PsPrBd=0x0000bcb8,c3\_PsPrBd=0x950f0000,c4\_PsPrBd=0x04c25dc0;
  //PsGetProcessDebugPort
  ULONG  c1\_PsGPDP=0x8b08458b,c2\_PsGPDP=0x0000bc80,c3\_PsGPDP=0x04c25d00,c4\_PsGPDP=0xcccccc00;
  //NtQueryInformationProcess
  ULONG  c1\_NtQuIP=0x0124a164,c2\_NtQuIP=0x808a0000,c3\_NtQuIP=0x00000140,c4\_NtQuIP=0x84e44588;
  //KiDispatchException
  ULONG  c1\_KiDiEx=0x8be44589,c2\_KiDiEx=0xb5890875,c3\_KiDiEx=0xfffffd14,c4\_KiDiEx=0x890c4d8b;
  //PspCreateProcess
  ULONG  c1\_PsCrPr=0x56535151,c2\_PsCrPr=0x183d8b57,c3\_PsCrPr=0xbb804d91,c4\_PsCrPr=0x8055b540;
  
  
  
  
  
  
  
  
  
  //功能号为 11，先获取所需的缓冲区大小
  ZwQuerySystemInformation(11,&size, 0, &size);

  //申请内存失败就返回 成功就存在 BUF 里
  if(NULL==(buf = (PULONG)ExAllocatePool(PagedPool, size)))
  {
  DbgPrint("failed alloc memory failed  \\n");
  return 0;
  }
    
  //再次执行,将枚举结果放到指定的内存区域 (buf)
  status=ZwQuerySystemInformation(11,buf, size , 0);
  if(!NT\_SUCCESS( status ))
  {
    DbgPrint("failed  query\\n");
  return 0;
  }
  //module 就是 buf 地址+4 一个 ULONG 的长度 1 为 4.2 为 8
  module = (PSYSTEM\_MODULE\_INFORMATION)(( PULONG )buf + 1);

  ntosknlEndAddr=(ULONG)module->Base+(ULONG)module->Size;
  ntosknlBase=(ULONG)module->Base;
  curAddr=ntosknlBase;
  //释放申请的内存
  ExFreePool(buf);
  
  //在指定模块内存范围内搜索.
  for (i=curAddr;i<=ntosknlEndAddr;i++)  
  {
  if ((\*((ULONG \*)i)==c1\_Fption)&&(\*((ULONG \*)(i+4))==c2\_Fption)&&(\*((ULONG \*)(i+8))==c3\_Fption)&&(\*((ULONG \*)(i+12))==c4\_Fption)) 
  {      
  DbgkForwardException = i-0x5;
  //DbgPrint("\[DbgkForwardException\] %x\\n",DbgkForwardException);
  //break;
  }

  if ((\*((ULONG \*)i)==code1\_3)&&(\*((ULONG \*)(i+4))==code2\_3)&&(\*((ULONG \*)(i+8))==code3\_3)&&(\*((ULONG \*)(i+12))==code4\_3)) 
  {      
  DbgkpSetProcessDebugObject = i-0x5;
  //DbgPrint("\[DbgkpSetProcessDebugObject\] %x\\n",DbgkpSetProcessDebugObject);  
  //break;
  }
  if ((\*((ULONG \*)i)==c1\_QuMess)&&(\*((ULONG \*)(i+4))==c2\_QuMess)&&(\*((ULONG \*)(i+8))==c3\_QuMess)&&(\*((ULONG \*)(i+12))==c4\_QuMess)) 
  {      
  DbgkpQueueMessage = i-0x5;
  //DbgPrint("\[DbgkpQueueMessage\] %x\\n",DbgkpQueueMessage);  
  //break;
  }
  if ((\*((ULONG \*)i)==c1\_PsPrDe)&&(\*((ULONG \*)(i+4))==c2\_PsPrDe)&&(\*((ULONG \*)(i+8))==c3\_PsPrDe)&&(\*((ULONG \*)(i+12))==c4\_PsPrDe)) 
  {      
  PspProcessDelete = i-0x5;
  //DbgPrint("\[PspProcessDelete\] %x\\n",PspProcessDelete);  
  //break;
  }
  if ((\*((ULONG \*)i)==c1\_DbMarP)&&(\*((ULONG \*)(i+4))==c2\_DbMarP)&&(\*((ULONG \*)(i+8))==c3\_DbMarP)&&(\*((ULONG \*)(i+12))==c4\_DbMarP)) 
  {      
  DbgkpMarkProcessPeb = i-0x1a;
  //DbgPrint("\[DbgkpMarkProcessPeb\] %x\\n",DbgkpMarkProcessPeb);  
  //break;
  }
  if ((\*((ULONG \*)i)==c1\_DbExPr)&&(\*((ULONG \*)(i+4))==c2\_DbExPr)&&(\*((ULONG \*)(i+8))==c3\_DbExPr)&&(\*((ULONG \*)(i+12))==c4\_DbExPr)&&(\*((ULONG \*)(i+16))==c5\_DbExPr)) 
  {//DbgkExitThread,DbgkExitProcess
  DbgkExitProcess=0;
  DbgkExitProcess = i-0x5;
  DbgkExitProcess\_Offset1=DbgkExitProcess+0x22;
  
  WPOFF();
  RtlCopyMemory((PVOID)DbgkExitProcess\_Offset1,(PVOID)EditCode,2);
  WPON();

  //DbgPrint("\[DbgkExitThread\_DbgkExitProcess\] %x\\n",DbgkExitProcess);
  }
  if ((\*((ULONG \*)i)==c1\_DbCrTh)&&(\*((ULONG \*)(i+4))==c2\_DbCrTh)&&(\*((ULONG \*)(i+8))==c3\_DbCrTh)&&(\*((ULONG \*)(i+12))==c4\_DbCrTh)) 
  {      
  DbgkCreateThread = i-0xF;
  //DbgPrint("\[DbgkCreateThread\] %x\\n",DbgkCreateThread);
  }
  if ((\*((ULONG \*)i)==c1\_PsExTh)&&(\*((ULONG \*)(i+4))==c2\_PsExTh)&&(\*((ULONG \*)(i+8))==c3\_PsExTh)&&(\*((ULONG \*)(i+12))==c4\_PsExTh)) 
  {      
  PspExitThread = i-0xC;
  //DbgPrint("\[PspExitThread\] %x\\n",PspExitThread);
  }
  if ((\*((ULONG \*)i)==c1\_NtTePr)&&(\*((ULONG \*)(i+4))==c2\_NtTePr)&&(\*((ULONG \*)(i+8))==c3\_NtTePr)&&(\*((ULONG \*)(i+12))==c4\_NtTePr)) 
  {      
  NtTerminateProcess = i-0x5;
  //DbgPrint("\[NtTerminateProcess\] %x\\n",NtTerminateProcess);
  }
  if ((\*((ULONG \*)i)==c1\_PsPrBd)&&(\*((ULONG \*)(i+4))==c2\_PsPrBd)&&(\*((ULONG \*)(i+8))==c3\_PsPrBd)&&(\*((ULONG \*)(i+12))==c4\_PsPrBd)) 
  {      
  PsIsProcessBeingDebugged = i-0x5;
  //DbgPrint("\[PsIsProcessBeingDebugged\] %x\\n",PsIsProcessBeingDebugged);
  }
  if ((\*((ULONG \*)i)==c1\_PsGPDP)&&(\*((ULONG \*)(i+4))==c2\_PsGPDP)&&(\*((ULONG \*)(i+8))==c3\_PsGPDP)&&(\*((ULONG \*)(i+12))==c4\_PsGPDP)) 
  {      
  PsGetProcessDebugPort = i-0x5;
  //DbgPrint("\[PsGetProcessDebugPort\] %x\\n",PsGetProcessDebugPort);
  }
  if ((\*((ULONG \*)i)==c1\_NtQuIP)&&(\*((ULONG \*)(i+4))==c2\_NtQuIP)&&(\*((ULONG \*)(i+8))==c3\_NtQuIP)&&(\*((ULONG \*)(i+12))==c4\_NtQuIP)) 
  {      
  NtQueryInformationPro = i-0xF;
  //DbgPrint("\[NtQueryInformationProcess\] %x\\n",NtQueryInformationPro);
  }
  if ((\*((ULONG \*)i)==c1\_KiDiEx)&&(\*((ULONG \*)(i+4))==c2\_KiDiEx)&&(\*((ULONG \*)(i+8))==c3\_KiDiEx)&&(\*((ULONG \*)(i+12))==c4\_KiDiEx)) 
  {      
  KiDispatchException = i-0x14;
  //DbgPrint("\[KiDispatchException\] %x\\n",KiDispatchException);
  }
  if ((\*((ULONG \*)i)==c1\_PsCrPr)&&(\*((ULONG \*)(i+4))==c2\_PsCrPr)&&(\*((ULONG \*)(i+8))==c3\_PsCrPr)&&(\*((ULONG \*)(i+12))==c4\_PsCrPr)) 
  {      
  PspCreateProcess = i-0x5;
  DbgPrint("\[PspCreateProcess\] %x\\n",PspCreateProcess);
  }
  
  
  
  
  
  
  
  }


  //DbgkpSetProcessDebugObject
  DbgkpSetProcessDebugObject\_Offset1=DbgkpSetProcessDebugObject+0x58;
  DbgkpSetProcessDebugObject\_Offset2=DbgkpSetProcessDebugObject+0x66;
  DbgkpSetProcessDebugObject\_Offset3=DbgkpSetProcessDebugObject+0x80;
  DbgkpSetProcessDebugObject\_Offset4=DbgkpSetProcessDebugObject+0x10C;
  
  //DbgkForwardException
  DbgkForwardException\_Offset1=DbgkForwardException+0x40;
  
  //DbgkpQueueMessage
  DbgkpQueueMessage\_Offset1=DbgkpQueueMessage+0x7D;
  
  //PspProcessDelete
  PspProcessDelete\_Offset1=PspProcessDelete+0x9E;
  //PspProcessDelete\_Offset2=PspProcessDelete+0x7D;   //----------------------------------------------------
  //PspProcessDelete\_Offset3=PspProcessDelete+0x7D;
  
  //DbgkpMarkProcessPeb
  DbgkpMarkProcessPeb\_Offset1=DbgkpMarkProcessPeb+0x44;
  
  //DbgkExitProcess
  DbgkExitProcess\_Offset1=DbgkExitProcess+0x22;
  
  //DbgkCreateThread
  DbgkCreateThread\_Offset1=DbgkCreateThread+0x127;

  //PspExitThread
  PspExitThread\_Offset1=PspExitThread+0x268;
  
  //NtTerminateProcess
  NtTerminateProcess\_Offset1=NtTerminateProcess+0x123;
  
  //PsIsProcessBeingDebugged
  PsIsProcessBeingDebugged\_Offset1=PsIsProcessBeingDebugged+0xA;
  
  //PsGetProcessDebugPort
  PsGetProcessDebugPort\_Offset1=PsGetProcessDebugPort+0xA;
  
  //NtQueryInformationProcess
  NtQueryInformationPro\_Offset1=NtQueryInformationPro+0x108;
  
  //KiDispatchException
  KiDispatchException\_Offset1=KiDispatchException+0x189;
  
  
  
  
  
  return 0;
}


//入口函数
NTSTATUS DriverEntry(IN PDRIVER\_OBJECT DriverObject,PUNICODE\_STRING RegistryPath)
{
  DriverObject->DriverUnload = Unload;
  
  DbgPrint(".");
  DbgPrint("加载完成\\n");
  
  GetUndocumentFunctionAdress();
  StartInLineHook(); 

  return STATUS\_SUCCESS;
}

//卸载函数
VOID Unload (IN PDRIVER\_OBJECT DriverObject)
{
  WPOFF(); 
  //DbgkpSetProcessDebugObject
  RtlCopyMemory((PVOID)DbgkpSetProcessDebugObject\_Offset1,OrigCode,2);  
  RtlCopyMemory((PVOID)DbgkpSetProcessDebugObject\_Offset2,OrigCode,2);  
  RtlCopyMemory((PVOID)DbgkpSetProcessDebugObject\_Offset3,OrigCode,2);  
  RtlCopyMemory((PVOID)DbgkpSetProcessDebugObject\_Offset4,OrigCode,2);  
  
  
  //DbgkExitProcess
  RtlCopyMemory((PVOID)DbgkExitProcess\_Offset1,OrigCode,2); 
  
  //DbgkForwardException
  RtlCopyMemory((PVOID)DbgkForwardException\_Offset1,OrigCode,2); 
  
  //DbgkpQueueMessage
  RtlCopyMemory((PVOID)DbgkpQueueMessage\_Offset1,OrigCode,2); 
  
  //PspProcessDelete
  RtlCopyMemory((PVOID)PspProcessDelete\_Offset1,OrigCode,2); 
  
  //DbgkpMarkProcessPeb
  RtlCopyMemory((PVOID)DbgkpMarkProcessPeb\_Offset1,OrigCode,2); 
  
  //DbgkCreateThread
  RtlCopyMemory((PVOID)DbgkCreateThread\_Offset1,OrigCode,2); 
  
  //PspExitThread
  RtlCopyMemory((PVOID)PspExitThread\_Offset1,OrigCode,2); 
  
  //NtTerminateProcess
  RtlCopyMemory((PVOID)NtTerminateProcess\_Offset1,OrigCode,2); 
  
  //PsIsProcessBeingDebugged
  RtlCopyMemory((PVOID)PsIsProcessBeingDebugged\_Offset1,OrigCode,2); 
  
  //PsGetProcessDebugPort
  RtlCopyMemory((PVOID)PsGetProcessDebugPort\_Offset1,OrigCode,2);
  
  //NtQueryInformationPro
  RtlCopyMemory((PVOID)NtQueryInformationPro\_Offset1,OrigCode,2);
  
  //KiDispatchException
  RtlCopyMemory((PVOID)KiDispatchException\_Offset1,OrigCode,2);

  WPON(); 
  DbgPrint("卸载完成!\\n");
}




//开始 HOOK
VOID StartInLineHook()
{
  //测试函数修改位置是否正确
  DbgPrint("\[TestCode\] %x\\n",KiDispatchException\_Offset1);

//开始改写
WPOFF(); 
  //DbgkpSetProcessDebugObject
  RtlCopyMemory((PVOID)DbgkpSetProcessDebugObject\_Offset1,(PVOID)EditCode,2); 
  RtlCopyMemory((PVOID)DbgkpSetProcessDebugObject\_Offset2,(PVOID)EditCode,2);
  RtlCopyMemory((PVOID)DbgkpSetProcessDebugObject\_Offset3,(PVOID)EditCode,2);
  RtlCopyMemory((PVOID)DbgkpSetProcessDebugObject\_Offset4,(PVOID)EditCode,2);

  //DbgkForwardException
  RtlCopyMemory((PVOID)DbgkForwardException\_Offset1,(PVOID)EditCode,2);

  //DbgkpQueueMessage
  RtlCopyMemory((PVOID)DbgkpQueueMessage\_Offset1,(PVOID)EditCode,2);

  //PspProcessDelete    
  RtlCopyMemory((PVOID)PspProcessDelete\_Offset1,(PVOID)EditCode,2);

  //DbgkpMarkProcessPeb
  RtlCopyMemory((PVOID)DbgkpMarkProcessPeb\_Offset1,(PVOID)EditCode,2);

  //DbgkCreateThread
  RtlCopyMemory((PVOID)DbgkCreateThread\_Offset1,(PVOID)EditCode,2);
  
  //PspExitThread
  RtlCopyMemory((PVOID)PspExitThread\_Offset1,(PVOID)EditCode,2);

  //NtTerminateProcess
  RtlCopyMemory((PVOID)NtTerminateProcess\_Offset1,(PVOID)EditCode,2);
  
  //PsIsProcessBeingDebugged
  RtlCopyMemory((PVOID)PsIsProcessBeingDebugged\_Offset1,(PVOID)EditCode,2);
  
  //PsGetProcessDebugPort
  RtlCopyMemory((PVOID)PsGetProcessDebugPort\_Offset1,(PVOID)EditCode,2);
  
  //NtQueryInformationProcess
  RtlCopyMemory((PVOID)NtQueryInformationPro\_Offset1,(PVOID)EditCode,2);
  
  //KiDispatchException
  RtlCopyMemory((PVOID)KiDispatchException\_Offset1,(PVOID)EditCode,2);
  
WPON();
}

//读写保护开关
VOID WPOFF(VOID)
{
\_\_asm
{
cli
mov eax,cr0
and eax,not 10000h
mov cr0,eax
}
}
VOID WPON(VOID)
{
\_\_asm
{
mov eax,cr0
or eax,10000h
mov cr0,eax
sti
}
}

```