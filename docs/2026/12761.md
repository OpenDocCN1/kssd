

# [更新完毕]调试器实现（1、11、19、29、34、45、50楼更新）（50楼已上传源码）

## 目录

- 1楼 第一章 调试器框架流程、用户输入处理及三种断点介绍
- 11楼 第二章 INT3断点
- 19楼 第三章 硬件断点
- 29楼 第四章 多内存断点
- 34楼 第五章 单步异常的处理
- 45楼 第六章 功能扩展
- 50楼 调试器源码

## 前言

作为一个软件逆向工程的研究者，必然会用到调试工具，如果能够掌握调试器工作的原理必然对软件调试会有更深入的了解。

本文根据笔者自己实现一个完整的调试器的过程，结合OD在调试功能上的实现方法，详细讲述一个调试器其主要功能的实现步骤，及实现过程中需要注意的细节，力求在原理、方法上能让阅读者有所收获。由于本人能力有限，文中必然有错漏之处，恳请读者不吝赐教。

## 第一章 调试器框架流程、用户输入处理及三种断点介绍

Windows三环下的调试器是利用异常处理来实现的。其主要流程可简要概述如下：创建或附加一个调试进程，然后进入等待调试事件的循环，整个调试工作大部分都在处理异常调试事件中完成。除了进程创建时需要在入口点设断点（以便程序断在入口点）、进程结束时进行一些必要的释放资源的操作。

创建调试进程和普通的创建进程一样都是使用`CreateProcess` API。只是创建调试进程时需要设置`CreateProcess`的第6个参数`dwCreationFlags`为`DEBUG_PROCESS`，这些细节都可以在MSDN中查到。

等待调试事件的循环可以直接在MSDN中找到VC的源码，在MSDN搜索API：`WaitForDebugEvent`，然后看Debugging Overview中的Using Debugging Support。即可找到VC代码如下：

```cpp
for(;;)
{
    WaitForDebugEvent(&DebugEv, INFINITE);
    switch (DebugEv.dwDebugEventCode)
    {
        case EXCEPTION_DEBUG_EVENT:
            switch (DebugEv.u.Exception.ExceptionRecord.ExceptionCode)
            {
                case EXCEPTION_ACCESS_VIOLATION:
                case EXCEPTION_BREAKPOINT:
                case EXCEPTION_DATATYPE_MISALIGNMENT:
                case EXCEPTION_SINGLE_STEP:
                case DBG_CONTROL_C:
            }
        case CREATE_THREAD_DEBUG_EVENT:
        case CREATE_PROCESS_DEBUG_EVENT:
        case EXIT_THREAD_DEBUG_EVENT:
        case EXIT_PROCESS_DEBUG_EVENT:
        case LOAD_DLL_DEBUG_EVENT:
        case UNLOAD_DLL_DEBUG_EVENT:
        case OUTPUT_DEBUG_STRING_EVENT:
    }
    ContinueDebugEvent(DebugEv.dwProcessId,
                       DebugEv.dwThreadId, dwContinueStatus);
}
```

为节省篇幅，我将代码中的注释已经删除，详细的注释说明请参阅MSDN。

通过这个循环我们可以看到`WaitForDebugEvent`这个API可以获得的调试事件类型包括了“异常事件”，“创建线程事件”，“创建进程事件”，“退出线程事件”，“退出进程事件”，“加载DLL事件”，“卸载DLL事件”，“调试输出字符串事件”。

在创建调试进程成功之后，会捕获“创建进程事件”，在创建进程事件中我们可以获得程序的入口地址（OEP），我们可以在OEP处设置一个一次性的软件断点（INT3），使得我们的被调试程序有机会断在OEP处等待用户输入。

用户的输入包括查看反汇编代码，查看数据，查看寄存器内容，下各种断点，单步步入，单步步过，运行（到某地址）等各种命令。断点一般包括软件断点（INT3断点），硬件断点和内存断点。

用户输入的命令可分为两种类型：

1. 控制程序流程的命令。如单步步入（T）、步过（P），运行（G）等；
2. 断点相关操作，如设置（bp）、查看（bl）、取消（bc）各种断点；查看反汇编代码（U）、数据（D）、寄存器（R）等。

可以设计一个等待用户输入的函数，当用户输入的是T、P、G时，函数返回值为TRUE，表示程序需要往下运行（单步或运行）。为其他输入时，函数返回值为FALSE，表示其他的操作或错误操作。

三种断点分别对应的异常类型及处理：

1. 软件断点（INT3）：对应的异常是`EXCEPTION_BREAKPOINT`，处理方式为恢复代码为原来字节，并将EIP减一，并设置单步以便进入单步后重新设置这个一般断点。
2. 硬件断点：对应的异常是`EXCEPTION_SINGLE_STEP`，处理方式要看是否为硬件执行断点，如果是硬件执行断点，则先取消断点，设置单步，进入单步后重新设置这个硬件执行断点。
3. 内存断点：将要设置内存断点的分页属性设置为`PAGE_NOACCESS`，则会产生`EXCEPTION_ACCESS_VIOLATION`异常。进入异常后先取消断点，设置单步，进入单步后再重新设置这个内存断点。

第一章先介绍到这里，下一章将介绍软件断点（INT3断点）。查看反汇编代码，查看数据和查看寄存器值比较简单我就不多说了，如果想要了解也可以从我以后发的源码中阅读到。

## 本系列文章参考书目、资料如下

1. 《加密与解密3》 编著：段钢
2. 《调试寄存器（DRx）理论与实践》 作者：Hume/冷雨飘心
3. 《数据结构》 作者：严蔚敏

附件为《调试器使用手册》和调试器可执行文件。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

## 上传的附件

- [调试器使用手册.rar](javascript:void(0);)（17.03kb，1257次下载）
- [SunDbg.rar](javascript:void(0);)（85.76kb，1584次下载）

