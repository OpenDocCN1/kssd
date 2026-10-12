
# 内存清零代码

> 原文：`www.pediy.com/kssd/pediy10/95030.html`

```
//====================================内存填零杀进程=====================================================
//4 个函数声明
void KeAttachProcess(PEPROCESS Process);

void KeDetachProcess();

NTSTATUS
ObOpenObjectByPointer(
            IN PVOID Object,
            IN ULONG HandleAttributes,
            IN PACCESS_STATE PassedAccessState OPTIONAL,
            IN ACCESS_MASK DesiredAccess,
            IN POBJECT_TYPE ObjectType OPTIONAL,
            IN KPROCESSOR_MODE AccessMode,
            OUT PHANDLE Handle
            );


NTSTATUS 
ZwTerminateProcess(
                   IN HANDLE ProcessHandle OPTIONAL,
                   IN NTSTATUS ExitStatus
                   );

//进程虚拟空间填 0
void DestoryProcessWithZero(ULONG eprocess)
{
  ULONG virtualAddr;
  PVOID handle;
  KeAttachProcess((PEPROCESS)eprocess);  //Attach 进程虚拟空间
  for(virtualAddr=0;virtualAddr<=0x7fffffff;virtualAddr+=0x1000)
  {   
    //蓝屏原因：用户内存是否可写要进行验证。用 ProbeForWrite 函数
    if(MmIsAddressValid((PVOID)virtualAddr))
    {
      _try
      {
        ProbeForWrite((PVOID)virtualAddr,0x1000,sizeof(ULONG));
        //RtlZeroMemory((PVOID)virtualAddr, 0x1000);
        memset((PVOID)virtualAddr,0xcc,0x1000);
      }_except(1)
      { 
        continue;   
      }
    }
    else
    {
      if(virtualAddr>0x1000000)  //填这么多足够破坏进程数据了
        break;
    }
  }
  KeDetachProcess();
  if(ObOpenObjectByPointer((PVOID)eprocess, 0, NULL, 0, NULL, KernelMode, &handle)!=STATUS_SUCCESS)
    return;
  ZwTerminateProcess((HANDLE)handle, STATUS_SUCCESS);
  ZwClose((HANDLE)handle );
}
```

这段代码会蓝屏的，但：

```
if(virtualAddr>0x1000000)  //填这么多足够破坏进程数据了
    break;
```

这个判断极大降低了蓝屏的概率，但填这么多其实是不够的。大部分程序默认加载地址是`0x00400000`，所以对大部分程序有效，但是有些别有用心的一小撮程序改掉加载地址后就无效了。

正确的清0代码如下：（出自`DebugMan`）

```
VOID ZeroIt(PEPROCESS pProcess){
        ULONG start,tmp;
        KAPC_STATE kapc;
        PHYSICAL_ADDRESS physicalAddr;
        KeStackAttachProcess(pProcess,&kapc);
        for(start=0x00010000;start< 0x60000000;start+=0x1000){
                physicalAddr = MmGetPhysicalAddress((PVOID)start);
                if( physicalAddr.HighPart > g_PhysicalPage.HighPart )
                        continue;
                if( physicalAddr.HighPart == g_PhysicalPage.HighPart &&
                        physicalAddr.LowPart >= g_PhysicalPage.LowPart   )
                        continue;
                if ( !(physicalAddr.HighPart | physicalAddr.LowPart) )
                        continue;
                if(start!=(ULONG)MmGetVirtualForPhysical(physicalAddr))
                        continue;
                __asm {
                        cli;
                        mov eax,cr0;
                        and eax,not 10000h;
                        mov cr0,eax;
                }
                __try{
                        RtlZeroMemory( (PVOID)start, 0x1000);
                }__except(1){
                }
                __asm {
                        mov  eax,cr0
                        or   eax,10000h
                        mov  cr0,eax
                        sti
                }
        }
        KeUnstackDetachProcess (&kapc);
}
```
