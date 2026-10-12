
# code:插 APC 杀进程

> 原文：[`www.pediy.com/kssd/pediy10/95032.html`](https://www.pediy.com/kssd/pediy10/95032.html)

代码:

```c
//===========================================插 APC 杀进程=================================================
//进程结束内幕：
//
//
//
//原理：遍历进程所有线程---初始化 APC---插入 APC
//结束线程的内核 Apc 例程

#define PS_CROSS_THREAD_FLAGS_SYSTEM 0x00000010UL

typedef enum _KAPC_ENVIRONMENT {
    OriginalApcEnvironment,
    AttachedApcEnvironment,
    CurrentApcEnvironment,
    InsertApcEnvironment
} KAPC_ENVIRONMENT;

VOID KeInitializeApc (
            PKAPC Apc,
            PETHREAD Thread,
            KAPC_ENVIRONMENT Environment,
            PKKERNEL_ROUTINE KernelRoutine,
            PKRUNDOWN_ROUTINE RundownRoutine,
            PKNORMAL_ROUTINE NormalRoutine,
            KPROCESSOR_MODE ProcessorMode,
            PVOID NormalContext
            );

BOOLEAN KeInsertQueueApc(PKAPC Apc,PVOID SystemArg1,PVOID SystemArg2,KPRIORITY Increment);

VOID  KernelKillThreadRoutine(IN PKAPC Apc,
                              IN OUT PKNORMAL_ROUTINE *NormalRoutine,
                              IN OUT PVOID *NormalContext,
                              IN OUT PVOID *SystemArgument1,
                              IN OUT PVOID *SystemArgument2)
{
  //调用 PsTerminateSystemThread 结束线程
  //修改当前线程的 ThreadFlags 为系统线程
  PULONG ThreadFlags;
  ExFreePool(Apc);  //释放 APC
  ThreadFlags=(ULONG *)((ULONG)PsGetCurrentThread()+0x248);  //ETHREAD 中 CrossThreadFlags 的偏移量为 0x248
  if(MmIsAddressValid(ThreadFlags))   //地址进行下验证
  {
    *ThreadFlags=(*ThreadFlags) | PS_CROSS_THREAD_FLAGS_SYSTEM; //修改为系统权限
    PsTerminateSystemThread(STATUS_SUCCESS); //结束系统线程，需要修改权限
    //PspExitThread(STATUS_SUCCESS);根据 PspTerminateThreadByPointer 定位 PspExitThread 地址
  }
}


VOID  KillProcessWithApc(ULONG ep_ro)
{
  //遍历线程有 2 种做法：1、PsGetNextProcessThread(未导出函数，自己定位地址)  2、从 EPROCESS 的 list 链中 ActiveThreads 记录线程数量
  //3、遍历 pspcidtable
  BOOLEAN status;
  PKAPC ExitApc=NULL;
  PEPROCESS eprocess;
  PETHREAD  ethread;
  ULONG i;
  ULONG num;   //线程数量
    ULONG Head;  //链表头
  ULONG address;//地址
  num=*(ULONG *)(ep_ro+0x1a0);   //EPROCESS 中 ActiveThreads 的数量  0x1a0 是 EPROCESS 中 ActiveThread 的偏移量
    KdPrint(("[RecordThreadAddress] num: 0x%x\n",num));    //打印线程数量
  Head=ep_ro+0x190;                //List_entry 第一个节点地址
  for(i=0;i<num;i++)
  {    
    //记录线程地址
    Head=(ULONG)((PLIST_ENTRY)Head)->Flink;
        address=Head-0x22c;
        KdPrint(("[RecordThreadAddress] address: 0x%x\n",address));      //打印线程地址
    ethread=(PETHREAD)address;                           //转换成线程指针 
    ExitApc=(PKAPC)ExAllocatePoolWithTag(NonPagedPool,sizeof(KAPC),MEM_TAG);
    if(ExitApc==NULL)
    {
      KdPrint(("[KillProcessWithApc] malloc memory failed \n"));
      return;
    }
    KeInitializeApc(ExitApc,
      ethread,                   //线程
      OriginalApcEnvironment,
      KernelKillThreadRoutine,
            NULL,
            NULL,
            KernelMode,
            NULL);//为线程初始化 APC
    status=KeInsertQueueApc(ExitApc,ExitApc,NULL,2);   //插入 Apc 到线程队列
    if(status==STATUS_SUCCESS)
      KdPrint(("KeInsertQueueApc  success\n"));  
    else
            KdPrint(("KeInsertQueueApc  failed\n"));
      }
}
```

`Anti KillProcessWithApc`  

Hook `KiInsertQueueApc`  

然后对 `Increment` 判断。。发现是插 APC。直接返回。  

或者坏一点。把调用者的线程穿进去。。嘿嘿。。  

不废话了。上代码。

代码:

```c
BOOLEAN  HookKiInsertQueueApc()
{
  BYTE* FunctionAddress;
  BYTE* CurrentAddress;
  ULONG tempAddr,HookAddress,NewOffset;
  PVOID KeInsertQueueApcAddr;
  UNICODE_STRING Uni_ObCreateObject;

  RtlInitUnicodeString(&Uni_ObCreateObject,L"KeInsertQueueApc");
  KeInsertQueueApcAddr =  MmGetSystemRoutineAddress(&Uni_ObCreateObject);

  if(KeInsertQueueApcAddr == NULL)
  {
    return FALSE;
  }
  FunctionAddress=(BYTE*)KeInsertQueueApcAddr;
  for(CurrentAddress=FunctionAddress;CurrentAddress<FunctionAddress+0x200; CurrentAddress++)
  {
    if(MmIsAddressValid((BYTE*)CurrentAddress))
    {
      if(*(BYTE*)CurrentAddress==0x28&&*(BYTE*)(CurrentAddress+1)==0xe8)
      {
        tempAddr = *(ULONG*)(CurrentAddress+2);
        if(MmIsAddressValid((ULONG*)((BYTE*)(CurrentAddress+1)+1)))
        {

          if(tempAddr&0x10000000)
          {
            NewOffset = (ULONG)Fake_KiInsertQueueApc+0xFFFFFFFB-(ULONG)(CurrentAddress+1);
            g_OldObpAllocateObjectOffset = *(ULONG*)((BYTE*)(CurrentAddress+2));
            HookAddress=*(ULONG*)((BYTE*)(CurrentAddress+2))+(ULONG)(CurrentAddress+1)-0xFFFFFFFB;
            g_TargetMmExchangeValue = (ULONG*)((BYTE*)(CurrentAddress+2));
            //DbgPrint("地址 %x",HookAddress);
            MmExchangeValue((ULONG*)((BYTE*)(CurrentAddress+2)),NewOffset);
            bIsHook =TRUE;
          }
          else
          {
            NewOffset = (ULONG)Fake_KiInsertQueueApc-(ULONG)(CurrentAddress+1)-5;
            g_OldObpAllocateObjectOffset = *(ULONG*)((BYTE*)(CurrentAddress+2));
            HookAddress=*(ULONG*)((BYTE*)(CurrentAddress+2))+(ULONG)(CurrentAddress+1)+5;
            g_TargetMmExchangeValue = (ULONG*)((BYTE*)(CurrentAddress+2));
            MmExchangeValue((ULONG*)((BYTE*)(CurrentAddress+2)),NewOffset);
            bIsHook =TRUE;
          }
          g_OldKiInsertQueueApc = (KIINSERTQUEUEAPC)HookAddress;
          //DbgPrint("状态 %x", g_OldObpAllocateObject);
          break;
        }
      }
    }
  }
  return TRUE;
}
```

//注释：其中的 `MmExchangeValu` 为自定义函数。就是先备份 CR0。然后修改 CR0 以过内存写保护。

//最后把头字节复制过去。最后再恢复保护..

HOOK 上了之后就简单了。。处理函数仁者见仁智者见智。。我说的只是最基本的。。

不过这个基本的办法。已经可以防御。。
