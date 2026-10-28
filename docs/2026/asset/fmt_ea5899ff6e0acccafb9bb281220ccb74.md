
# [翻译]Windows调试内幕系列二之Windows本地（Native）调试内幕

*Windows Native Debugging Internals*

作者：AlexIonescu<br>
原文链接：[0d7K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3!0H3k6h3&6J5j5$3g2Q4x3X3g2G2M7X3N6Q4x3V1k6S2M7Y4c8A6j5$3I4W2M7#2)9J5c8X3k6#2L8r3I4Q4y4h3k6$3K9h3g2%4i4K6u0r3x3U0f1](<https://bbs.kanxue.com/elink@4bfK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3!0H3k6h3&6J5j5$3g2Q4x3X3g2G2M7X3N6Q4x3V1k6S2M7Y4c8A6j5$3I4W2M7#2)9J5c8X3k6#2L8r3I4Q4y4h3k6$3K9h3g2%4i4K6u0r3x3U0f1>).<br>
系列的第一部分链接：<http://bbs.pediy.com/showthread.php?t=188300>

## Introduction / 介绍

In part two of this three part article series, the native interface to Windows debugging is dissected in detail. The reader is expected to have some basic knowledge of C and general NT Kernel architecture and semantics. Also, this is not an introduction on what debugging is or how to write a debugger. It is meant as a reference for experienced debugger writers, or curious security experts.

这篇文章是这个系列的第二部分，将要详细分析Windows调试的本地接口。读者应该具有C语言和一般的NT内核架构和语法的基本知识。同时，这篇文章不是介绍什么是调试，或者怎样编写一个调试器。它只是为有经验的调试器编写者，或者好奇的安全专家作一个参考。

## Native Debugging / 本地调试

Now it's time to look at the native side of things, and how the wrapper layer inside `ntdll.dll` communicates with the kernel. The advantage of having the `DbgUi` layer is that it allows better separation between Win32 and the NT Kernel, which has always been a part of NT design. `NTDLL` and `NTOSKRNL` are built together, so it's normal for them to have intricate knowledge of each others. They share the same structures, they need to have the same system call IDs, etc. In a perfect world, the NT Kernel should have to know nothing about Win32.

现在来看看Native内容的一面，以及位于`ntdd.dll`里的包装层是怎样和Windows内核通信的。具有`DbgUi`层的优点是，它允许Win32和NT内核之间更好地分离，这一直都是NT设计的一部分。`NTDLL`和`NTOSKRNL`是构建在一起的，所以它们彼此之间有复杂的联系，这是很正常的。它们共享相同的结构，它们需要有相同的系统调用身份，等等。最好的设计应该是，NT内核不应该知道关于Win32的任何细节。

Additionally, it helps anyone that wants to write debugging capabilities inside a native application, or to write a fully-featured native-mode debugger. Without `DbgUi`, one would have to call the `Nt*DebugObject` APIs manually, and do some extensive pre/post processing in some cases. `DbgUi` simplifies all this work to a simple call, and provides a clean interface to do it. If the kernel changes internally, `DbgUi` will probably stay the same, only its internal code would be modified.

此外，它可以帮助那些想要在本地应用程序中编写调试功能，或者编写一个全功能的本地模式调试器。如果没有`DbgUi`，他们将不得不手动调用`Nt*DebugObject` API，并在某些情况下，做一些扩展的预/后（pre/post）处理。`DbgUi`简化了所有这一切工作，通过一个简单的调用，并提供了一个干净的接口来完成。如果内核内部发生改变，`DbgUi`可能会保持不变，只有内部的代码将被修改。

We start our exploration with the function responsible for creating and associating a Debug Object with the current Process. Unlike in the Win32 world, there is a clear distinction between creating a Debug Object, and actually attaching to a process.

我们从一个函数开始探索，这个函数负责对当前进程创建并关联一个调试对象。不像在Win32的世界里，这里在创建一个调试对象和实际附加到进程之间有一个明显的区别。

```c
NTSTATUS
NTAPI
DbgUiConnectToDbg(VOID)
{
    OBJECT_ATTRIBUTES ObjectAttributes;

    /* Don't connect twice */
    if (NtCurrentTeb()->DbgSsReserved[1]) return STATUS_SUCCESS;

    /* Setup the Attributes */
    InitializeObjectAttributes(&ObjectAttributes, NULL, 0, NULL, 0);

    /* Create the object */
    return ZwCreateDebugObject(&NtCurrentTeb()->DbgSsReserved[1],
                               DEBUG_OBJECT_ALL_ACCESS,
                               &ObjectAttributes,
                               TRUE);
}
```

As you can see, this is a trivial implementation, but it shows us two things. Firstly, a thread can only have one debug object associated to it, and secondly, the handle to this object is stored in the `TEB`'s `DbgSsReserved` array field. Recall that in Win32, the first index, `[0]`, is where the Thread Data was stored. We've now learnt that `[1]` is where the handle is stored.

正如你所看到的，这是一个简单的实现，但它告诉我们两件事。首先，一个线程只能有一个调试对象与之关联，其次，该对象的句柄存储在当前线程环境块（`TEB`）的`DbgSsReserved`数组域中。记得在Win32中，该数组的第0号索引，即`DbgSsReserved[0]`存储了线程数据结构（`DBGSS_THREAD_DATA`），我们现在已经知道该数组的第1号索引，即`DbgSsReserved[1]`存储了该对象的句柄。

Now let's see how attaching and detaching are done:

现在让我们来看看，附加和分离是怎样完成的：

```c
NTSTATUS
NTAPI
DbgUiDebugActiveProcess(IN HANDLE Process)
{
    NTSTATUS Status;

    /* Tell the kernel to start debugging */
    Status = NtDebugActiveProcess(Process, NtCurrentTeb()->DbgSsReserved[1]);

    if (NT_SUCCESS(Status))
    {
        /* Now break-in the process */
        Status = DbgUiIssueRemoteBreakin(Process);

        if (!NT_SUCCESS(Status))
        {
            /* We couldn't break-in, cancel debugging */
            DbgUiStopDebugging(Process);
        }
    }

    /* Return status */
    return Status;
}

NTSTATUS
NTAPI
DbgUiStopDebugging(IN HANDLE Process)
{
    /* Call the kernel to remove the debug object */
    return NtRemoveProcessDebug(Process, NtCurrentTeb()->DbgSsReserved[1]);
}
```

Again, these are very simple implementations. We can learn, however, that the kernel is not responsible for actually breaking inside the remote process, but that this is done by the native layer. This `DbgUiIssueRemoteBreakin` API is also used by Win32 when calling `DebugBreakProcess`, so let's look at it:

再次，这是非常简单的实现。我们能够知道，然而，这个内核不负责实际中断远程进程，而是由本地层（native layer）来完成的。在Win32中，当调用`DebugBreakProcess`函数时，`DbgUiIssueRemoteBreakin` API也被使用，让我们来看看它：

```c
NTSTATUS
NTAPI
DbgUiIssueRemoteBreakin(IN HANDLE Process)
{
    HANDLE hThread;
    CLIENT_ID ClientId;
    NTSTATUS Status;

    /* Create the thread that will do the breakin */
    Status = RtlCreateUserThread(Process,
                                 NULL,
                                 FALSE,
                                 0,
                                 0,
                                 PAGE_SIZE,
                                 (PVOID)DbgUiRemoteBreakin,
                                 NULL,
                                 &hThread,
                                 &ClientId);

    /* Close the handle on success */
    if(NT_SUCCESS(Status)) NtClose(hThread);

    /* Return status */
    return Status;
}
```

All it does is create a remote thread inside the process, and then return to the caller. Does that remote thread do anything magic? Let's see:

它的作用就是在目标进程中创建一个远程线程，然后返回至它的调用者。那么，远程线程做了什么魔法？让我们来看看：

```c
VOID
NTAPI
DbgUiRemoteBreakin(VOID)
{
    /* Make sure a debugger is enabled; if so, breakpoint */
    if (NtCurrentPeb()->BeingDebugged) DbgBreakPoint();

    /* Exit the thread */
    RtlExitUserThread(STATUS_SUCCESS);
}
```

Nothing special at all; the thread makes sure that the process is really being debugged, and then issues a breakpoint. And, because this API is exported, you can call it locally from your own process to issue a debug break (but note that you will kill your own thread). In our look at the Win32 Debugging implementation, we've noticed that the actual debug handle is never used, and that calls always go through `DbgUi`. Then the `NtSetInformationDebugObject` system call was called, a special `DbgUi` API was called before, to actually get the debug object associated with the thread. This API also has a counterpart, so let's see both in action:

根本没有什么特别之处，这个线程确保目标进程正在被调试，然后产生一个断点。而且，因为`DbgBreakPoint`这个API已经导出，你可以在你自己的进程中本地调用它，用来产生一个调试断点（注意，这样将结束你自己的线程）。我们从Win32调试实现中看到，注意到，实际的调试句柄从来没有被使用过，并且这种调用总是通过`DbgUi`函数。之后`NtSetInformationDebugObject`系统调用被调用，在一个特殊的`DbgUi` API被调用之前，用来实际获取线程相关联的调试对象。这也有一个对应的API，让我们来看看这两个API的实现：

```c
HANDLE
NTAPI
DbgUiGetThreadDebugObject(VOID)
{
    /* Just return the handle from the TEB */
    return NtCurrentTeb()->DbgSsReserved[1];
}

VOID
NTAPI
DbgUiSetThreadDebugObject(HANDLE DebugObject)
{
    /* Just set the handle in the TEB */
    NtCurrentTeb()->DbgSsReserved[1] = DebugObject;
}
```
