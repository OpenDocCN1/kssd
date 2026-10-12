
# inline hook SSDT 躲避 Themida 的 ThreadHideFromDebugger （学习笔记 2）

> 原文：[`www.pediy.com/kssd/pediy10/78423.html`](https://www.pediy.com/kssd/pediy10/78423.html)

Themida 保护的程序会采用 `ThreadHideFromDebugger` 来反调试，这个一旦设置，就一直起效，所以对于需要附加的程序就不太好办了，或者有的程序直接调用 `int 2e` 来反调试，虽然有插件比如 `invisible` 可以躲过，但是需要附加进行调试的程序就不起作用了。

比如我以前写的一个反调试函数：

```c
void anti()
{
    /////////////////////////////////////////////////////////////////////////////////////////////
    ///////////////////////////////////////////////////////////////////////////////////////
    //NtSetInformationThread 功能设置调试端口为 0

    ::GetCurrentThread();
    __asm                                                 // xp 系统
    {
        push 0                 //InformationLength
        push 0                 //ThreadInformation
        push 0x11              //11 就是 ThreadHideFromDebugger
        push eax               //当前线程句柄
        mov eax, 0xe5          //EAX  NtSetInformationThread 调用号
        mov edx, esp           //EDX  当前堆栈放
        int 0x2e
        add esp, 0x10
    }
    ::GetCurrentThread();
    __asm                                                 // 2000 系统
    {
        push 0
        push 0
        push 0x11
        push eax
        mov eax, 0xc6
        mov edx, esp
        int 0x2e
        add esp, 0x10
    }
}
```

为了躲避上面的 `anti`，`HOOK SSDT` 是个不错的方法。

下面的原理就是 `HOOK SSDT`，原理是我学习笔记 1 中的原理，不过 `HOOK` 单元改了下，ring3 的拿到 RING0 要改下 = =。

`HOOK SSDT` 分 5 步：

1.  申请一个全局变量用来存放被破坏的指令
2.  声明一个类型用来强制转换调用自己的 `POldNtWriteProcessMemroy`
3.  自己的函数
4.  安装 `HOOK`
5.  卸载 `HOOK`

其中起关键作用的是下面的代理函数：

```c
NTSTATUS __stdcall //第三步 自己的函数
MyNtSetInformationThread(
    IN HANDLE ThreadHandle,
    IN THREADINFOCLASS ThreadInformationClass,
    IN PVOID ThreadInformation,
    IN ULONG ThreadInformationLength
)
{
    if(ThreadInformationClass == 0x11)
    {
        DbgPrint("发现 ANTI DEBUG 行为");
        DbgPrint("NtSetInformationThread 参数 %8x %8x %8x %8x ", ThreadHandle, ThreadInformationClass, ThreadInformationClass, ThreadInformationLength); // 打印出参数
        ThreadInformationClass = (THREADINFOCLASS)0xff;
        DbgPrint("参数被我改过后是   %8x %8x %8x %8x ", ThreadHandle, ThreadInformationClass, ThreadInformationClass, ThreadInformationLength);
        return STATUS_SUCCESS;     //直接返回成功  这样 ANTI 就失效了
    }

    //强制转换成函数的形式
    return  ((SYSNTSETINFORMATIIONTHREAD)OldNtSetInformationThread)(  //  放行
        ThreadHandle,
        ThreadInformationClass,
        ThreadInformation,
        ThreadInformationLength
    );
}
```

下面是全部的代码，附件中也包含完成的代码。

SYS 加载有，你只要调用下 `anti` 这个函数，代理函数就会输出参数并且改变参数：

```c
void * OldNtSetInformationThread;                   //第一步 申请一个全局变量 用来存放被破坏的指令
//NtSetInformationThread
typedef NTSTATUS (__stdcall *SYSNTSETINFORMATIIONTHREAD)  //第二步 声明一个类型 用来强制转换 调用自己的 POldNtWriteProcessMemroy
(
IN HANDLE ThreadHandle,
IN THREADINFOCLASS ThreadInformationClass,
IN PVOID ThreadInformation,
IN ULONG ThreadInformationLength
);
NTSTATUS __stdcall                   //第三步 自己的函数
MyNtSetInformationThread(
    IN HANDLE ThreadHandle,
    IN THREADINFOCLASS ThreadInformationClass,
    IN PVOID ThreadInformation,
    IN ULONG ThreadInformationLength
)
{
    if(ThreadInformationClass == 0x11)
    {
        DbgPrint("发现 ANTI DEBUG 行为");
        DbgPrint("NtSetInformationThread 参数 %8x %8x %8x %8x ", ThreadHandle, ThreadInformationClass, ThreadInformationClass, ThreadInformationLength); //证明自己存在
        ThreadInformationClass = (THREADINFOCLASS)0xff;
        DbgPrint("参数被我改过后是   %8x %8x %8x %8x ", ThreadHandle, ThreadInformationClass, ThreadInformationClass, ThreadInformationLength); //证明自己存在
        return STATUS_SUCCESS;     //直接返回成功
    }

    //强制转换成函数的形式
    return  ((SYSNTSETINFORMATIIONTHREAD)OldNtSetInformationThread)(  //  放行
        ThreadHandle,
        ThreadInformationClass,
        ThreadInformation,
        ThreadInformationLength
    );
}

ULONG AddrNtSetInformationThread;
VOID OnUnload(IN PDRIVER_OBJECT DriverObject);
// 驱动程序加载时调用 DriverEntry 例程
NTSTATUS DriverEntry(PDRIVER_OBJECT pDriverObj, PUNICODE_STRING pRegistryString)
{
    DbgPrint("Hook NtSetInformationThread DriverEntry");
    pDriverObj->DriverUnload = OnUnload;
    AddrNtSetInformationThread = *PULONG((ULONG)KeServiceDescriptorTable->ServiceTable + 0xe5 * 4);
    AfxHookCode((void*)AddrNtSetInformationThread, (void*)MyNtSetInformationThread, (void**)&OldNtSetInformationThread, 10);  //第四步 AFXHOOK
// if(驱动加载失败)
//    一定要 AfxUnHookCode  不然驱动加载失败 自己分配的内存代码也就失效了 系统经过这个 SSDT 也就蓝了
    return STATUS_SUCCESS;
}

//////////////////////////////////////////////////////
VOID OnUnload(IN PDRIVER_OBJECT DriverObject)
{
    AfxUnHookCode((void*)AddrNtSetInformationThread, OldNtSetInformationThread, 10);  //第五部 卸载  驱动卸载一定要还原 HOOK 因为驱动卸载了，自己分配的内存也就失效了  所以自己的 POldNtWriteProcessMemroy 指向的代码也就没了
    DbgPrint("Unload Hook NtSetInformationThread");
}
```

下面是 RING 0 `HOOK` 模块：

```c
#ifndef __AFXHOOKCODE_H__
#define __AFXHOOKCODE_H__

bool AfxHookCode(void* TargetProc, void* NewProc, void ** l_OldProc, int bytescopy = 5)
{
    *l_OldProc =  ExAllocatePool(NonPagedPool, (bytescopy+5));        // 执行被覆盖的指令 再跳到原来的代码上运行
    memcpy(*l_OldProc, TargetProc, bytescopy);            // 事先保存被破坏的指令
    __asm{
        cli
        mov  eax,cr0
        and  eax,not 10000h
        mov  cr0,eax
    }

        // 我的内存的代码执行完 跳到原来的 代码+破坏的代码的长度 上去
    *((unsigned char*)(*l_OldProc) + bytescopy) = 0xe9;
            // 我内存代码跳到原来代码上的偏移
    *(unsigned int *)((unsigned char*)(*l_OldProc) +bytescopy + 1)=(unsigned int)(TargetProc) + bytescopy - ( (unsigned int)((*l_OldProc)) + 5 + bytescopy ) ;

           //被 HOOK 的函数头改为 jmp
    *(unsigned char*)TargetProc =(unsigned char)0xe9;
           //被 HOOK 的地方跳到我的新过程 接受过滤
    *(unsigned int*)((unsigned int)TargetProc +1) = (unsigned int)NewProc - ( (unsigned int)TargetProc + 5);
    __asm{
        mov  eax,cr0
        or  eax,10000h
        mov  cr0,eax
        sti
    }
    return true;
}

bool AfxUnHookCode(void* TargetAddress, void * l_SavedCode, unsigned int len)
{
    __asm{
        cli
        mov  eax,cr0
        and  eax,not 10000h
        mov  cr0,eax
    }
    // 恢复 HOOK
    memcpy(TargetAddress, l_SavedCode, len);
    __asm{
        mov  eax,cr0
        or  eax,10000h
        mov  cr0,eax
        sti
    }
    return true;
}
#endif
```

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | Inline Hook SSDT.rar |
| :--- | :--- |
