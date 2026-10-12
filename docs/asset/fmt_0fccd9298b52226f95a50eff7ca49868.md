
# `PspTerminateProcess` 杀进程

> 原文：[`www.pediy.com/kssd/pediy10/95033.html`](https://www.pediy.com/kssd/pediy10/95033.html)

代码:

```c
//===============================PspTerminateProcess 结束进程=============================================
/*PspTerminateProcess 是内核未导出函数，需要自己定位
方法有二：
1、暴力搜索内存
2、从函数调用中定位（跟定位 SSDT Shadow 一个原理）*/
//1、解析 Ntoskrnl.exe 文件定位地址

typedef struct _SYSTEM_MODULE_INFORMATION
{
  ULONG Reserved[2];
  PVOID Base;  //基址
  ULONG Size;  // 镜像大小
  ULONG Flags;
  USHORT Index;
  USHORT Unknown;
  USHORT LoadCount;
  USHORT ModuleNameOffset;
  CHAR ImageName[256];
  
}SYSTEM_MODULE_INFORMATION,*PSYSTEM_MODULE_INFORMATION;

typedef NTSTATUS (*PSPTERMINATETPROCESS)(PEPROCESS Process,NTSTATUS ExitStatus);

ULONG GetPspTerminateProcess()
{
  //利用 ZwQuerySystemInformation 依据 SystemModuleInformation 获取 Ntoskrnl.exe 基址和镜像大小，然后特征码搜索定位
  NTSTATUS status;
  PVOID buffer;
  ULONG size;   //动态分配大小
  ULONG code1=0x8b55ff8b, code2=0xa16456ec, code3=0x00000124, code4=0x3b08758b;   //SP2
  ULONG address;
  ULONG NtoskrnlBase;
  ULONG NtoskrnlEndAddress;
  ULONG i;
  PSYSTEM_MODULE_INFORMATION module;
  
  ZwQuerySystemInformation(SystemModuleInformation,&size,0,&size);  //返回需要 size 大小
  KdPrint(("[GetPspTerminateProcess] size:0x%x\n",size));
  buffer=ExAllocatePoolWithTag(NonPagedPool,size,MEM_TAG);
  if(buffer==NULL)
  {
    KdPrint(("[GetPspTerminateProcess] malloc memory failed\n"));
    return 0;
  }
  status=ZwQuerySystemInformation(SystemModuleInformation,buffer,size,0);
  if(!NT_SUCCESS(status))
  {
    KdPrint(("[GetPspTerminateProcess] query failed\n"));
    return status;
  }
  module=(PSYSTEM_MODULE_INFORMATION)((PULONG)buffer+1);
  NtoskrnlBase=(ULONG)module->Base;
  NtoskrnlEndAddress=(ULONG)module->Base+(ULONG)module->Size;
    KdPrint(("[GetPspTerminateProcess] Size :0x%x\n",(ULONG)module->Size));  //打印验证
  KdPrint(("[GetPspTerminateProcess] NtoskrnlBase :0x%x\n",NtoskrnlBase));  //打印验证
    KdPrint(("[GetPspTerminateProcess] NtoskrnlEndAddress :0x%x\n",NtoskrnlEndAddress));  //打印验证
  ExFreePool(buffer);
  //搜索特征码
  for(i=NtoskrnlBase;i<=NtoskrnlEndAddress;i++)
  {    
    if(MmIsAddressValid((PULONG)i))
    {  //蓝屏原因：搜索到之后就应该退出,少句代码 return address
      if((* (PULONG)i==code1)&&((* (PULONG)(i+4)==code2)&&((* (PULONG)(i+8)==code3)&&((* (PULONG)(i+12)==code4))
      {
        address=(ULONG)i;
        KdPrint(("[GetPspTerminateProcess] address :0x%x\n",address));  //打印地址
        return address;
      }
    }
  }
  return 0;
}

VOID KillProcessWithPsp(ULONG epro)
{
  PSPTERMINATETPROCESS MyPspTerminateProcess;
  ULONG address;
  address=GetPspTerminateProcess();
  if(MmIsAddressValid((PULONG)address))
  {
    MyPspTerminateProcess =(PSPTERMINATETPROCESS)address;  //定位 PspTerminateProcess
    MyPspTerminateProcess((PEPROCESS)epro,0);              //杀进程
  }
  else
    return;
  
}
```

`Anti PspTerminateProcess` 杀进程 and `Anti PspTerminateProcessByPointer`

遍历目标进程线程。找到 `ETHEAD`

再利用线程标记偏移找到并修改 `flags`

`|= PS_CROSS_THREAD_FLAGS_SYSTEM..`

关键代码段：

`//找到线程标记偏移`

`ULONG GetCrossThreadFlagsOffset()`

`{`

`static ULONG Offset = 0;`

`PUCHAR pProc;`

`if(Offset == 0)`

`pProc = (PUCHAR)PsIsSystemThread;`

`while( *pProc!=0x8B || *(pProc+1)!=0x80 )`

`pProc++;`

`DbgPrint("发现函数。address == 0x%0.8X",pProc);`

`Offset = *(PULONG)(pProc+2);`

`DbgPrint("偏移是 0x%0.8X",Offset);`

`return Offset;`

`}`

需要保护的时候

`Flags = (PULONG)( (PUCHAR)Thread + GetCrossThreadFlagsOffset() );`

`*Flags |= PS_CROSS_THREAD_FLAGS_SYSTEM;`
