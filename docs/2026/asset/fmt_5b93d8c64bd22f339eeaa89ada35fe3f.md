
# [翻译]Windows调试内幕系列一之Windows用户模式调试内幕

作者： AlexIonescu

原文链接：0a9K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3!0H3k6h3&6J5j5$3g2Q4x3X3g2G2M7X3N6Q4x3V1k6S2M7Y4c8A6j5$3I4W2M7#2)9J5c8X3k6#2L8r3I4Q4y4h3k6$3K9h3g2%4i4K6u0r3x3U0b7`.

本人第一次翻译外文，可能会有各种缺点，甚至错误，敬请同行专家和广大读者给予斧正。谢谢！

## Windows User Mode Debugging Internals

### Introduction

The internal mechanisms of what allows user-mode debugging to work have rarely ever been fully explained. Even worse, these mechanisms have radically changed in Windows XP, when much of the support was re-written, as well as made more subsystem portable by including most of the routines in `ntdll`, as part of the Native API. This three part series will explain this functionality, starting from the Win32 (`kernel32`) viewpoint all the way down (or up) to the NT Kernel (`ntoskrnl`) component responsible for this support, called `Dbgk`, while taking a stop to the NT System Library (`ntdll`) and its `DbgUi` component.

The reader is expected to have some basic knowledge of C and general NT Kernel architecture and semantics. Also, this is not an introduction on what debugging is or how to write a debugger. It is meant as a reference for experienced debugger writers, or curious security experts.

### Windows用户模式调试内幕

#### 介绍：

关于用户模式调试的内部机制本来就很少有完整的解释。更糟糕的是，在Windows XP中这些机制根本上又被改变了，这时关于此的支持需要被重写，同时通过包含在大部分位于`ntdll`中的例程来生成更方便的子系统，作为本地（Native） API的一部分。接下来将分三个部分系列来解释这个功能，从Win32(`kerne32l`)的视点开始向下（或向上）到达NT内核 (`ntoskrnl`)，把负责这种支持的组件称为`Dbgk`，同时采取了停止到NT系统库（`ntdll`）和`DbgUi`组件。

这里要求读者具有C语言和一般的NT内核架构和语法的基本知识。这篇文章不是介绍什么是调试或者怎么编写一个调试器。它的目的是为有经验的调试器编写者，或者好奇的安全专家作参考。

## Win32 Debugging

The Win32 subsystem of NT has allowed the debugging of processes ever since the first release, with later releases adding more features and debugging help libraries, related to symbols and other PE information. However, relatively few things have changed to the outside API user, except for the welcome addition of the ability to stop debugging a process, without killing it, which was added in Windows XP. This release of NT also contained several overhauls to the underlying implementation, which will be discussed in detail. However, one important side-effect of these changes was that LPC (and `csrss.exe`) were not used anymore, which allowed debugging of this binary to happen (previously, debugging this binary was impossible, since it was the one responsible for handling the kernel-to-user notifications).

### Win32调试

NT的 Win32子系统从第一个版本发布开始就允许进程的调试，在之后的版本中又添加了更多的功能和调试帮助库，其中涉及到符号和其他PE信息。然而，对于外部的API用户，功能几乎没有什么改变，除了添加了受欢迎的停止对进程的调试，而不用结束进程，这种功能已经加入到Windows XP系统中。此版本的NT还包含了几处对底层实现的检查，这将要详细讨论。然而，这些变化的一个重要的副作用是，LPC（和`csrss.exe`）不再被使用，这使得调试此二进制得以发生（以前，调试此二进制是不可能的，因为它是负责处理内核层到应用层的通知）。

The basic Win32 APIs for dealing with debugging a process were simple: `DebugActiveProcess`, to attach, `WaitForDebugEvent`, to wait for debug events to come through, so that your debugging can handle them, and `ContinueDebugEvent`, to resume thread execution. The release of Windows XP added three more useful APIs: `DebugActiveProcessStop`, which allows you to stop debugging a process (detach), `DebugSetProcessKillOnExit`, which allows you to continue running a process even after its' been detached, and `DebugBreakProcess`, which allows you to perform a remote DebugBreak without having to manually create a remote thread. In Windows XP Service Pack 1, one more API was added, `CheckRemoteDebuggerPresent`. Much like its `IsDebuggerPresent` counterpart, this API allows you to check for a connected debugger in another process, without having to read the PEB remotely.

用于处理调试进程的基本Win32 API是很简单的：`DebugActiveProcess`，负责附加（attach），`WaitForDebugEvent`，等待调试事件的到来，这样你的调试就能处理它们，同时`ContinueDebugEvent`，负责恢复线程执行。Windows XP的发布版本增加了三个更有用的API：`DebugActiveProcessStop`，允许你停止调试一个进程（detach），`DebugSetProcessKillOnExit`，允许你继续运行一个进程，即使这个进程是在被停止调试之后，`DebugBreakProcess`，允许你完成一个远程DebugBreak，而不用手动创建一个远程线程。在Windows XP Service Pack 1中，又增加了一个API，`CheckRemoteDebuggerPresent`，它就像是`IsDebuggerPresent`的对应版本，这个API允许你检查一个进程是否连接着调试器，而不用远程地去读取进程的PED来进行判断。

Because of NT's architecture, these APIs, on recent versions of Windows (2003 will be used as an example, but the information applies to XP as well) do not much do much work themselves. Instead, they do the typical job of calling out the native functions required, and then process the output so that the Win32 caller can have it in a format that is compatible with Win9x and the original Win32 API definition. Let's look at these very simple implementations:

由于Windows NT的架构，在最新的Windows版本中（2003将作为一个例子使用，但是这些信息同样应用于XP上），这些API本身做很少的工作。相反，它们以一种典型的方法实现，即调用所需要的本地函数（native functions），然后处理输出，这样Win32调用者能够形成一种格式，就是能够兼容Win9x和原始的Win32 API定义。让我们来看看这些API很简单的实现：

```
BOOL
WINAPI
DebugActiveProcess(IN DWORD dwProcessId)
{
    NTSTATUS Status;
    HANDLE Handle;
    /* Connect to the debugger */
    Status = DbgUiConnectToDbg();
    if (!NT_SUCCESS(Status))
    {
        SetLastErrorByStatus(Status);
        return FALSE;
    }
    /* Get the process handle */
    Handle = ProcessIdToHandle(dwProcessId);
    if (!Handle) return FALSE;
    /* Now debug the process */
    Status = DbgUiDebugActiveProcess(Handle);
    NtClose(Handle);
    /* Check if debugging worked */
    if (!NT_SUCCESS(Status))
    {
        /* Fail */
        SetLastErrorByStatus(Status);
        return FALSE;
    }
    /* Success */
    return TRUE;
}
```

As you can see, the only work that's being done here is to create the initial connection to the user-mode debugging component, which is done through the `DbgUi` Native API Set, located in `ntdll`, which we'll see later. Because `DbgUi` uses handles instead of PIDs, the PID must first be converted with a simple helper function:

正如你所看到的，`DebugActiveProcess`函数唯一做的工作就是创建一个初始连接，用于连接到用户模式的调试组件，并且它是通过位于`ntdll`中的`DbgUi` Native API集合函数来实现的，这些DbgUi函数我们会在之后介绍。由于`DbgUi`函数使用进程句柄（handle）代替进程PID，所以PID首先需要被转换为句柄，通过下面一个简单的帮助函数：

```
HANDLE
WINAPI
ProcessIdToHandle(IN DWORD dwProcessId)
{
    NTSTATUS Status;
    OBJECT_ATTRIBUTES ObjectAttributes;
    HANDLE Handle;
    CLIENT_ID ClientId;
    /* If we don't have a PID, look it up */
    if (dwProcessId == -1) dwProcessId = (DWORD)CsrGetProcessId();
    /* Open a handle to the process */
    ClientId.UniqueProcess = (HANDLE)dwProcessId;
    InitializeObjectAttributes(&ObjectAttributes, NULL, 0, NULL, NULL);
    Status = NtOpenProcess(&Handle,
                           PROCESS_ALL_ACCESS,
                           &ObjectAttributes,
                           &ClientId);
    if (!NT_SUCCESS(Status))
    {
        /* Fail */
        SetLastErrorByStatus(Status);
        return 0;
    }
    /* Return the handle */
    return Handle;
}
```

If you are not familiar with Native API, it is sufficient to say that this code is the simple equivalent of an OpenProcess on the PID, so that a handle can be obtained. Going back to `DebugActiveProcess`, the final call which does the work is `DbgUiDebugActiveProcess`, which is again located in the Native API. After the connection is made, we can close the handle that we had obtained from the PID previously. Other APIs function much in the same way. Let's take a look at two of the newer XP ones:

如果你不熟悉Native API，你可以这样理解，这个代码可以简单地等价于用`OpenProcess`函数打开PID，这样获取进程句柄。回到`DebugActiveProcess`函数，最后调用`DbgUiDebugActiveProcess`函数来完成工作，它也是位于Native API中。在完成连接之后，我们可以关闭之前通过PID获取的句柄了。其它的API函数也是通过类似的这种方式来工作。接下来我们来看看两个在XP中新加入的函数：

```
BOOL
WINAPI
DebugBreakProcess(IN HANDLE Process)
{
    NTSTATUS Status;
    /* Send the breakin request */
    Status = DbgUiIssueRemoteBreakin(Process);
    if(!NT_SUCCESS(Status))
    {
        /* Failure */
        SetLastErrorByStatus(Status);
        return FALSE;
    }
    /* Success */
    return TRUE;
}
```

```
BOOL
WINAPI
DebugSetProcessKillOnExit(IN BOOL KillOnExit)
{
    HANDLE Handle;
    NTSTATUS Status;
    ULONG State;
    /* Get the debug object */
    Handle = DbgUiGetThreadDebugObject();
    if (!Handle)
    {
        /* Fail */
        SetLastErrorByStatus(STATUS_INVALID_HANDLE);
```
