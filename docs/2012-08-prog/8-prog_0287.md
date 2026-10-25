
# Hook `ProbeForWrite` 探测隐藏进程

> 原文：[`www.pediy.com/kssd/pediy12/135777.html`](https://www.pediy.com/kssd/pediy12/135777.html)

今天闲着没事，随便写写，大牛们别笑话哈。

`ProbeForWrite` 这个函数系统调用的频率很高，所以我 hook 这个函数来查看一下系统中调用这个函数的进程，或许可以发现隐藏的进程。

```c
#include "Driver.h"

ULONG oldAddress;

int i=0;

ULONG b[10000];

ULONG GetFunctionAddress(PCWSTR FunctionName)//得到函数的地址

{

  UNICODE_STRING  HookFunctionName;

  RtlInitUnicodeString(&HookFunctionName,FunctionName);   

  return (ULONG)MmGetSystemRoutineAddress(&HookFunctionName);

}

void unhook()//恢复钩子

{

  KIRQL oldIRQL;

  ULONG Address;

  UCHAR JmpProbeForWrite[5] = {0x8b,0xff,0x55,0x8b,0xec};   

  Address=GetFunctionAddress(L"ProbeForWrite");

  oldIRQL=KeRaiseIrqlToDpcLevel();

  __asm

  {

    cli

    mov eax,cr0

    and eax,not 10000h

    mov cr0,eax

  }

  RtlCopyMemory((PUCHAR)Address,JmpProbeForWrite,5);

  __asm

  {

    mov eax,cr0

    or eax,10000h

    mov cr0,eax

    sti

  }

  KeLowerIrql(oldIRQL);

  return;

}

#pragma LOCKEDCODE

void myProbeForWrite()//hook 后跳转到的我的函数

{

  int j=i,k;

  PUCHAR bPEP;

  PUCHAR f_name;

  if(i==10000)

  {

    unhook();

  }

  b[i]=(ULONG)PsGetCurrentProcessId();

  for(k=0;k<j;k++)

  {

    if(b[k]==b[j])

    {

      goto lopo;

    }

  }

  bPEP=(PUCHAR)PsGetCurrentProcess();

      DbgPrint("The ProcessId is %d,EPROCESS=%x\n",(ULONG)PsGetCurrentProcessId(),(ULONG)PsGetCurrentProcess());

      f_name=(bPEP+0x174);

      DbgPrint("Process name is %s\n",f_name);

lopo:

  i++;

  __asm

  {

      mov eax,oldAddress

      pop  edi

      pop  ebp

      mov  edi,edi

            push ebp

            mov  ebp,esp

      jmp  eax

  }

}

void hook_ProbW()//hook  ProbeForWrite 这个函数

{

  KIRQL oldIRQL;

  ULONG Address;

  UCHAR JmpProbeForWrite[5] = {0xE9,0,0,0,0};   

  Address=GetFunctionAddress(L"ProbeForWrite");

  __asm

  {

    mov eax,Address

    lea esi,[eax+5]

    mov eax,esi

    mov oldAddress,eax

  }

  *(ULONG *)(JmpProbeForWrite+1)=(ULONG)myProbeForWrite-((ULONG)Address+5);

  oldIRQL=KeRaiseIrqlToDpcLevel();

  __asm

  {

    cli

    mov eax,cr0

    and eax,not 10000h

    mov cr0,eax

  }

  RtlCopyMemory((PUCHAR)Address,JmpProbeForWrite,5);

  __asm

  {

    mov eax,cr0

    or eax,10000h

    mov cr0,eax

    sti

  }

  KeLowerIrql(oldIRQL);

  return;

}

//主函数

#pragma INITCODE

extern "C" NTSTATUS DriverEntry (

      IN PDRIVER_OBJECT pDriverObject, 

      IN PUNICODE_STRING pRegistryPath  ) 

{

  DbgPrint("Enter DriverEntry\n");

  hook_ProbW();

  DbgPrint("leave DriverEntry\n");

  return STATUS_SUCCESS;

}

//卸载函数

#pragma PAGEDCODE

VOID HelloDDKUnload (IN PDRIVER_OBJECT pDriverObject) 

{

  DbgPrint("leave unload!!!\n");

}
```

编译运行后果然发现了三个隐藏的进程用冰刃和 `XueTr` 都看不到这几个进程。待会得去研究研究这几个进程。
