
# Make a debugger

> 原文：[`www.pediy.com/kssd/pediy10/59496.html`](https://www.pediy.com/kssd/pediy10/59496.html)

Isaiah 大牛提及了 1 下,能不能自己构造一套调试的框架呢？不依赖于操作系统的提供的机制。比如调试事件。

于是偶发 1 个讨论帖子,看看有无大牛对这些东西感兴趣,指点 1 下![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸")

我们的目的.一些加壳软件使用自调试,比如 arm 的 `copymem2`,我们不能方便的调试子进程,因为调试端口已经被占用,`OllyDbg` 不能 attach,我们希望实现 1 个自己的调试器注入进去,实现 `windbg` 类似的入侵调试功能,我的想法的制作 1 个开源 `debuger` 计划,欢迎大家帮助开发完善这个小工具,这个小工具并不打算替代 `OllyDbg`,仅仅是在某些 `OllyDbg` 一时无法使用的时候作为临时 `debuger` 使用![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸")

`SoftICE` 是完全自己实现了调试过程

`OllyDbg` 则是利用 Platform SDK 和最新的 `DbgHelp.dll` 提供的 API 作为引擎写的 debugger，也即是的 debugger 的 Host 端

其实 CPU 为 Debug 提供了强力的支持，比如设单步跟踪的标志,还有 `Drx` 机制

我们来看看

`OllyDbg` 使用了下面的一系列 API

```
================

  GetThreadContext

   SetThreadContext

  DebugActiveProcess

  ReadProcessMemory

  WriteProcessMemory

  CreateProcess with DEBUG_PROCESS flag

  WaitForDebugEvent

================
```

       其中一些 API 不是调试特有的,我们可以继续使用

我们需要修正的是 `DebugActiveProcess` 和 `WaitForDebugEvent`

简单起见,我们只模拟 attach 过程

       一个调试器,当 `DebugActiveProcess` 上一个进程以后,被调试进程主线程被挂起,这种状况将持续到我们的程序调用 `WaitForDebugEvent` 为止

引用:     背景知识: user-mode debuger 工作流程

    <1>debuger 创建一个新进程，或 attach 一个正在运行的进程。我们称这个进程为 B。

    <2>debuger 等待进程 B 产生 debug 事件

    <3>进程 B 产生 debug 事件，发送消息给 debuger，进程挂起，等待 debuger 指令。

    <3>debuger 处理 debug 事件，发送消息给进程 B。

    <4>进程 B 接受 debuger 发送的消息，进程复苏。

    <5>循环 2-4

   消息传递是通过 lpc port 来进行的，流程如下所示：

    debuger <--> kernel <--> process B

    上面所说的消息结构如下：

```
    typedef struct _DEBUG_MESSAGE
    {
    PORT_MESSAGE        PORT_MSG;
    DEBUG_EVENT        DebugEvent;
    }DEBUG_MESSAGE, *PDEBUG_MESSAGE;
    typedef struct _PORT_MESSAGE
    {
    USHORT                  DataSize;//数据长度
    USHORT                  MessageSize;//总长度
    USHORT                  MessageType;
    USHORT                  DataInfoOffset;
    CLIENT_ID               ClientId;
    ULONG                   MessageId;
    ULONG                   SectionSize;
    //UCHAR            Data[];
    }PORT_MESSAGE, *PPORT_MESSAGE;
```

    在`\Microsoft SDK\samples\winbase\Debug` 目录下有几个简单的 user-mode debuger 的源代码,

大家可以参考一下。

   我们不做进一步深入,只说说我们需要的东西,那就是 `DEBUG_EVENT`

   `OllyDbg` 关键部分是对于 `DEBUG_EVENT` 的处理

引用:  背景知识:

`EXCEPTION_DEBUG_EVENT`：产生调试例外

`CRATE_THREAD_DEBUG_EVENT`：新的线程产生

`CREATE_PROCESS_DEBUG_EVENT`：新的进程产生。注：在 `DEBUG_ONLY_THIS_PROCESS` 时只有一次，

在 `DEBUG_PROCESS` 时如果该程序启动了子进程就可能有多次。

`EXIT_THREAD_DEBUG_EVENT`：一个线程运行中止

`EXIT_PROCESS_DEBUG_EVENT`：一个进程中止。注：在 `DEBUG_ONLY_THIS_PROCESS` 时只有一次，

在 `DEBUG_PROCESS` 可能有多次。

`LOAD_DLL_DEBUG_EVENT`：一个 DLL 模块被载入。

`UNLOAD_DLL_DEBUG_EVENT`：一个 DLL 模块被卸载。 

本讨论帖子不求完美,尽可能多保存 1 点相关 undocu 资料

还是简单起先,我们只处理 `EXCEPTION_DEBUG_EVENT` 事件

引用: 

```
typedef struct _EXCEPTION_DEBUG_INFO { 
  EXCEPTION_RECORD ExceptionRecord; 
  DWORD dwFirstChance; 
} EXCEPTION_DEBUG_INFO, *LPEXCEPTION_DEBUG_INFO;

typedef struct _EXCEPTION_RECORD { 
  DWORD ExceptionCode; 
  DWORD ExceptionFlags; 
  struct _EXCEPTION_RECORD *ExceptionRecord; 
  PVOID ExceptionAddress; 
  DWORD NumberParameters; 
  ULONG_PTR ExceptionInformation[EXCEPTION_MAXIMUM_PARAMETERS];
} EXCEPTION_RECORD, *PEXCEPTION_RECORD;
```

产生中断的地址 `ExceptionAddress` 和产生中断的信息代码 `ExceptionCode`

这 2 个是我们必须模拟的,继续简单化处理,我们只管 2 个

引用: 

`EXCEPTION_BREAKPOINT`：断点中断信息代码

`EXCEPTION_SINGLE_STEP`：单步中断信息代码 

我们现在来看看中断,中断,我们先说 `int3`,最简单,向指定地址写入 `CC`

我们需要接管异常处理流程,那么我们的入侵调试器需要有 1 个 dll

用于注入被调试进程,用来接管线程异常,从而我们能够处理异常

引用: 操作系统为每个线程分配一个 TEB 结构的数据块，并用 FS 指向它：

```
NT_TIB STRUCT   
    +0 ExceptionList   dd      ?
     StackBase   dd     ?
     StackLimit   dd      ?
     SubSystemTib   dd      ?
     union
         FiberData  dd      ?
         Version  dd       ?
     ends
     ArbitraryUserPointer dd ?
     Self    dd  ?
NT_TIB ENDS
EXCEPTION_REGISTRATION_RECORD struct
  +0 prev  dd ?
  +4 handler  dd ?
EXCEPTION_REGISTRATION_RECORD ends
```

FS 指向 `NT_TIB` 结构，FS:[0] 就指向了 `EXCEPTION_REGISTRATION_RECORD` 结构

`EXCEPTION_RECORD` 结构包含了发生异常的详细信息,这些信息独立于 CPU

```
EXCEPTION_RECORD STRUCT
  +0 ExceptionCode         DWORD       ? ；异常发生的原因代码
  +4 ExceptionFlags        DWORD       ? ；异常处理操作标志
  +8 pExceptionRecord      DWORD       ?
  +0Ch ExceptionAddress      DWORD       ? ；异常发生的地址
  +10h NumberParameters      DWORD       ?
  +14h ExceptionInformation  DWORD EXCEPTION_MAXIMUM_PARAMETERS dup(?)
EXCEPTION_RECORD ENDS
```

异常发生时，操作系统向引起异常的线程的堆栈压入 3 个结构：

`EXCEPTION_RECORD`, `CONTEXT`, `EXCEPTION_POINTERS`

```
EXCEPTION_POINTERS STRUCT
  +0 pExceptionRecord  DWORD       ?
  +4 ContextRecord     DWORD       ?
EXCEPTION_POINTERS ENDS
```

即是说：`EXCEPTION_POINTERS` 包含指向 `EXCEPTION_RECORD`, `CONTEXT` 这两个结构的指针

`CONTEXT` 结构包含了特定处理器的寄存器数据

```
CONTEXT STRUCT
   ContextFlags  DWORD       ?
 //调试寄存器
  +4 iDr0          DWORD       ?
  +8 iDr1          DWORD       ?
  +0C iDr2          DWORD       ?
  +10 iDr3          DWORD       ?
  +14 iDr6          DWORD       ?
  +18 iDr7          DWORD       ?
 //浮点寄存器
   FloatSave     FLOATING_SAVE_AREA <>
 //段寄存器
  +8C regGs         DWORD       ?
  +90 regFs         DWORD       ?
  +94 regEs         DWORD       ?
  +98 regDs         DWORD       ?
 //通用寄存器
  +9C regEdi        DWORD       ?
  +A0 regEsi        DWORD       ?
  +A4 regEbx        DWORD       ?
  +A8 regEdx        DWORD       ?
  +AC regEcx        DWORD       ?
  +B0 regEax        DWORD       ?
 //控制寄存器
  +B4 regEbp        DWORD       ?
  +B8 regEip        DWORD       ?
  +BC regCs         DWORD       ?
  +C0 regFlag       DWORD       ?
  +C4 regEsp        DWORD       ?
  +C8 regSs         DWORD       ?
   ExtendedRegisters db MAXIMUM_SUPPORTED_EXTENSION dup(?)
CONTEXT ENDS
```

  我们最好直接更改线程的异常处理链,把我们的处理过程放在第一个位置

当然我们可以使用 `SetUnhandledExceptionFilter`,但是那样我们的异常处理过程

被安排到了最后,但是作为做实验,使用 `SetUnhandledExceptionFilter` 可以

方便的安装处理过程

整理 1 下我们一共需要的工作--最简单的

1. 1 个 dll,用来注入被调试进程,安装 `ExceptionFilter`

2. 主 debuger,和 dll 通讯,接受异常分析并且处理

3. 主 exe,完成以下 api 工作,他们不是调试特有 api,可以工作

   `GetThreadContext`

    `SetThreadContext`

   `ReadProcessMemory`

   `WriteProcessMemory`

4. 关键功能,异常处理部分,偶打算完成 `int3`, `drx`, 内存断点

这里需要感谢大牛们对 `OllyDbg` 各种功能的完整分析

内存断点其实就是改变内存页属性,使之发生异常

从本质上来说,断点就是发生异常![](img/022c916b9660e467a8663bd57a4ae6d1.jpg "狂笑")

5. 界面设计,我的想法还是参照 `OllyDbg`,一个反汇编窗口+1 个寄存器窗口+1 个内存查看窗口

其实堆栈查看和内存查看窗口差不多,不过是自动定位 `esp` 的地址显示的那片内存

不过偶基本不会做界面,汗..

代码部分偶还没有来得及写,其实主要是不太会写![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")

希望大牛参与帮助 哈哈![](img/022c916b9660e467a8663bd57a4ae6d1.jpg "狂笑")![](img/022c916b9660e467a8663bd57a4ae6d1.jpg "狂笑")

欢迎大家参与讨论 哈哈![](img/c72cb8eba397ac67dda6e1ac58a9b627.jpg "微笑")![](img/c72cb8eba397ac67dda6e1ac58a9b627.jpg "微笑")

贴个测试代码

1. 异常程序

```cpp
// dbgtest.cpp : Defines the entry point for the console application.
//

#include "stdafx.h"
#include "windows.h"

int main(int argc, char* argv[])
{
  MessageBox(0,"int3","dbgtest",MB_OK);
  _asm int 3
  MessageBox(0,"ok","dbgtest",MB_OK);
  return 0;
}
```

这个程序直接运行是会异常的,

我们需要注入 1 个 `DbgDll`,执行如下过程

```cpp
DWORD WINAPI InjectDbgDll(IN LPINJECT_DATA lpData)
{
        SetUnhandledExceptionFilter(UnhandledExceptionFilter);
    return 0;
}

LONG WINAPI UnhandledExceptionFilter(IN struct _EXCEPTION_POINTERS* ExceptionInfo)
{
    return -1;
}
```

什么也不处理,直接返回,这样这个程序就可以处理异常.运行下去了

不过这个模式不适合 `int3` 断点,`int3` 断点需要还原代码,`eip-1`

但是内存异常和 `drx` 这样处理应该可以
