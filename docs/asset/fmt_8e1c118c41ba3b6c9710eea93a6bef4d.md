
# inlineHook 的入学者的拙见

> 原文：[`www.pediy.com/kssd/pediy10/101362.html`](https://www.pediy.com/kssd/pediy10/101362.html)

最近工作闲，看了看和工作不太沾边的东西，其中看了`inLineHook`,下边是我的一点理解，不知正确与否。

一般步骤就是：内存中找到函数地址保存原函数前几字节（具体几字节需要用内核调试器调试）在原函数的前几字节写入跳转到我们函数的指令（计算好地址）在自己的函数里完成处理恢复原函数的前几字节，跳转到原函数处

我觉得在`inlineHook`的时候堆栈的变化特别重要，有两种情况，如图 1 和图 2

图 1 示例的是被`HOOK`的函数是用堆栈来传递参数，所以我们函数里不能改变了`EBP`，`ESP`的值

图 2 示例的是被`HOOK`的函数是用寄存器来传递参数（`FASTCALL`），所以我们的函数里是可以自由使用的，只要进入被`HOOK`函数时相应的寄存器值不变就行

现在有一个疑问就是，第 1 种情况：不能使用堆栈，那我们的函数岂不是什么功能都不能做就只能简单的跳转到原函数去，示例代码 1 后面给出（网上的示例）

第 2 种情况只能对于`FASTCALL`的函数适用，这么想的话那`inlineHook`会受限制的吗？示例代码 2 后面给出

![](img/6ae6463721d23e5900c46900c722b1f5.jpg)

图 1

![](img/749b79b674fc97c61a4026d5df51cf31.jpg)

图 2

**代码 1：**

```c
#include <ntddk.h>

#include <ntifs.h>

#include <windef.h>

ULONG g_KiInsertQueueApc;

ULONG g_uCr0;

BYTE g_HookCode[5] = { 0xe9, 0, 0, 0, 0 };

BYTE g_OrigCode[5] = { 0 }; // 原函数的前字节内容

BYTE jmp_orig_code[7] = { 0xEA, 0, 0, 0, 0, 0x08, 0x00 };

BOOL g_bHooked = FALSE;

VOID

fake_KiInsertQueueApc (

    PKAPC Apc,

    KPRIORITY Increment

    );

VOID

Proxy_KiInsertQueueApc (

    PKAPC Apc,

    KPRIORITY Increment

    );

void WPOFF()

{

    ULONG uAttr;

    _asm

    {

        push eax;

        mov eax, cr0;

        mov uAttr, eax;

        and eax, 0FFFEFFFFh; // CR0 16 BIT = 0

        mov cr0, eax;

        pop eax;

        cli

    };

    g_uCr0 = uAttr; //保存原有的 CRO 性

}

VOID WPON()

{

    _asm

    {

        sti

        push eax;

        mov eax, g_uCr0; //恢原有 CR0 性

        mov cr0, eax;

        pop eax;

    };

}

//

// 停止 inline hook

//

VOID UnHookKiInsertQueueApc ()

{

    KIRQL  oldIrql;

    WPOFF();

    oldIrql = KeRaiseIrqlToDpcLevel();

    RtlCopyMemory ( (BYTE*)g_KiInsertQueueApc, g_OrigCode, 5 );

    KeLowerIrql(oldIrql);

    WPON();

    g_bHooked = FALSE;

}

//

// 开始 inline hook --  KiInsertQueueApc

//

VOID HookKiInsertQueueApc ()

{ 

    KIRQL  oldIrql;

    if (g_KiInsertQueueApc == 0) {

        DbgPrint("KiInsertQueueApc == NULL\n");

        return;

    }

    //DbgPrint("开始 inline hook --  KiInsertQueueApc\n");

    DbgPrint( "KiInsertQueueApc 的地址 t0x%08x\n", (ULONG)g_KiInsertQueueApc );

    // 保存原函数的前字节内容

    RtlCopyMemory (g_OrigCode, (BYTE*)g_KiInsertQueueApc, 5);

    *( (ULONG*)(g_HookCode + 1) ) = (ULONG)fake_KiInsertQueueApc - (ULONG)g_KiInsertQueueApc - 5;

    // 禁止系统写保护，提升 IRQL 到 DPC

    WPOFF();

    oldIrql = KeRaiseIrqlToDpcLevel();

    RtlCopyMemory ( (BYTE*)g_KiInsertQueueApc, g_HookCode, 5 );

    *( (ULONG*)(jmp_orig_code + 1) ) = (ULONG) ( (BYTE*)g_KiInsertQueueApc + 5 );

    RtlCopyMemory ( (BYTE*)Proxy_KiInsertQueueApc, g_OrigCode, 5);

    RtlCopyMemory ( (BYTE*)Proxy_KiInsertQueueApc + 5, jmp_orig_code, 7);

    // 恢复写保护，降低 IRQL

    KeLowerIrql(oldIrql);

    WPON();

    g_bHooked = TRUE;

}

//

// 跳转到我们的函数里面进行预处理

//

__declspec (naked) 

VOID

fake_KiInsertQueueApc (

    PKAPC Apc,

    KPRIORITY Increment

    )

{

    // 去掉 DbgPrint,不然这个 hook 会产生递归

    //DbgPrint("inline hook --  KiInsertQueueApc 成功\n");

    __asm 

    {

                 jmp Proxy_KiInsertQueueApc

    }

}

//

// 代理函数，负责跳转到原函数中继续执行

//

__declspec (naked) 

VOID

Proxy_KiInsertQueueApc (

    PKAPC Apc,

    KPRIORITY Increment

    )

{

    __asm {  // 共字节

            _emit 0x90

            _emit 0x90

            _emit 0x90

            _emit 0x90

            _emit 0x90  // 前字节实现原函数的头字节功能

            _emit 0x90  // 这个填充 jmp

            _emit 0x90

            _emit 0x90

            _emit 0x90

            _emit 0x90  // 这字节保存原函数+5 处的地址

            _emit 0x90   

            _emit 0x90  // 因为是长转移,所以必须是 0x0080

    }

}

ULONG GetFunctionAddr( IN PCWSTR FunctionName)

{

    UNICODE_STRING UniCodeFunctionName;

    RtlInitUnicodeString( &UniCodeFunctionName, FunctionName );

    return (ULONG)MmGetSystemRoutineAddress( &UniCodeFunctionName );   

}

//根据特征值，从 KeInsertQueueApc 搜索中搜索 KiInsertQueueApc

ULONG FindKiInsertQueueApcAddress()

{

  char * Addr_KeInsertQueueApc = 0;

  int i = 0;

  char Findcode[] = { 0xE8, 0xcc, 0x29, 0x00, 0x00 };

  ULONG Addr_KiInsertQueueApc = 0;

    Addr_KeInsertQueueApc = (char *) GetFunctionAddr(L"KeInsertQueueApc");

  for(i = 0; i < 100; i ++)

  {

        if( Addr_KeInsertQueueApc[i] == Findcode[0] && 

      Addr_KeInsertQueueApc[i + 1] == Findcode[1] && 

      Addr_KeInsertQueueApc[i + 2] == Findcode[2] && 

      Addr_KeInsertQueueApc[i + 3] == Findcode[3] && 

      Addr_KeInsertQueueApc[i + 4] == Findcode[4] 

      )

    {

      Addr_KiInsertQueueApc = (ULONG)&Addr_KeInsertQueueApc[i] + 0x29cc + 5;

      break;

    }

  }

  return Addr_KiInsertQueueApc;

}

VOID OnUnload( IN PDRIVER_OBJECT DriverObject )

{

  DbgPrint("My Driver Unloaded!");

  UnHookKiInsertQueueApc();

}

NTSTATUS DriverEntry( IN PDRIVER_OBJECT theDriverObject, IN PUNICODE_STRING theRegistryPath )

{

  DbgPrint("My Driver Loaded!");

  theDriverObject->DriverUnload = OnUnload;

  g_KiInsertQueueApc = FindKiInsertQueueApcAddress();

  HookKiInsertQueueApc();

  return STATUS_SUCCESS;

} 
```

**代码 2：**

```c
//===================inline hook KiInsertQueueApc====================

//KiInsertQueueApc 为内核未导出函数，可以从导出函数 KeInsertQueueApc 定位

//修改 KiInsertQueueApc 开头 5 字节

//处理函数思路：apc-->kthread---apc_state--eprocess--进程名字

//HookKiInsertQueueApc---DetourMyKiInsertQueueApc---UnHookKiInsertQueueApc

#include "ntddk.h"

ULONG CR0VALUE;

ULONG g_KiInsertQueueApc;

unsigned char JmpAddress[5]={0xE9,0,0,0,0};       //跳转到 HOOK 函数的地址

unsigned char  OriginalBytes[5]={0};               //保存原始函数前五个字

VOID FASTCALL DetourMyKiInsertQueueApc(IN PKAPC Apc,IN KPRIORITY Increment);

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

//1、获取 KiInsertQueueApc 地址

ULONG GetFunctionAddr( IN PCWSTR FunctionName)     //PCWSTR 常量指针，指向 16 位 UNICODE

{

        UNICODE_STRING UniCodeFunctionName;

        RtlInitUnicodeString( &UniCodeFunctionName, FunctionName );

        return (ULONG)MmGetSystemRoutineAddress( &UniCodeFunctionName );   

}

ULONG GetKiInsertQueueApcAddr()

{

        ULONG sp_code1=0x28,sp_code2=0xe8,sp_code3=0xd88a;  //特征码,sp_code3 windbg 显示错误，应该为 d88a

        ULONG address=0;

        PUCHAR addr;

        PUCHAR p;

        addr=(PUCHAR)GetFunctionAddr(L"KeInsertQueueApc");

        for(p=addr;p<p+PAGE_SIZE;p++)

        {

                if(*(p-1)==sp_code1)&&(*p==sp_code2)&&(*(PUSHORT)(p+5)==sp_code3))

                {

                        address=*(PULONG)(p+1)+(ULONG)(p+5);

                        break;

                }

        }

        KdPrint(("[KeInsertQueueApc] addr %x\n",(ULONG)addr));

    KdPrint(("[KiInsertQueueApc] address %x\n",address));

    return address;

}

VOID HookKiInsertQueueApc()

{    

        KIRQL Irql;

        g_KiInsertQueueApc=GetKiInsertQueueApcAddr();
```
