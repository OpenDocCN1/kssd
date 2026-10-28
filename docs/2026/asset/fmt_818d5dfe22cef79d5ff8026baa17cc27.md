
For those familiar with object-oriented programming, this will seem similar to the concept of accessor and mutator methods. Even though Win32 has perfect access to this handle and could simply read it on its own, the NT developers decided to make `DbgUi` much like a class, and make sure access to the handle goes through these public methods. This design allows the debug handle to be stored anywhere else if necessary, and only these two APIs will require changes, instead of multiple DLLs in Win32.

对于那些熟悉面向对象编程的，这似乎类似于访问器和突变方法的概念（get和set方法）。即使Win32针对这个句柄具有完善的访问，并且它自己可以简单地读取句柄，NT开发者还是决定把`DbgUi`设计得像一个类，并确保通过这些公有方法来访问这个句柄。这种设计允许调试句柄根据需要可以存储在任何地方，且只有这两个API将需要改变，而不是Win32里多个DLL。

Now for a visit of the wait/continue functions, which under Win32 were simply wrappers:

现在来看看等待和继续（wait/continue）函数，在上层Win32它们是简单包装者：

```
NTSTATUS
NTAPI
DbgUiContinue(IN PCLIENT_ID ClientId,
              IN NTSTATUS ContinueStatus)
{
    /* Tell the kernel object to continue */
    return ZwDebugContinue(NtCurrentTeb()->DbgSsReserved[1],
                           ClientId,
                           ContinueStatus);
}

NTSTATUS
NTAPI
DbgUiWaitStateChange(OUT PDBGUI_WAIT_STATE_CHANGE DbgUiWaitStateCange,
                     IN PLARGE_INTEGER TimeOut OPTIONAL)
{
    /* Tell the kernel to wait */
    return NtWaitForDebugEvent(NtCurrentTeb()->DbgSsReserved[1],
                               TRUE,
                               TimeOut,
                               DbgUiWaitStateCange);
}
```

Not surprisingly, these functions are also wrappers in `DbgUi`. However, this is where things start to get interesting, since if you'll recall, `DbgUi` uses a completely different structure for debug events, called `DBGUI_WAIT_STATE_CHANGE`. There is one API that we have left to look at, which does the conversion, so first, let's look at the documentation for this structure:

毫不奇怪，在`DbgUi`层这些函数同样也是包装者。然而，这就是事情开始变得有趣了，如果你回忆一下，`DbgUi`为调试事件使用一个完全不同的结构，称为`DBGUI_WAIT_STATE_CHANGE`。这里有个API我们留到之后再看，它完成转换功能，首先，我们来看看这个结构的文档说明：

```
//
// User-Mode Debug State Change Structure
//

typedef struct _DBGUI_WAIT_STATE_CHANGE
{
    DBG_STATE NewState;
    CLIENT_ID AppClientId;
    union
    {
        struct
        {
            HANDLE HandleToThread;
            DBGKM_CREATE_THREAD NewThread;
        } CreateThread;
        struct
        {
            HANDLE HandleToProcess;
            HANDLE HandleToThread;
            DBGKM_CREATE_PROCESS NewProcess;
        } CreateProcessInfo;
        DBGKM_EXIT_THREAD ExitThread;
        DBGKM_EXIT_PROCESS ExitProcess;
        DBGKM_EXCEPTION Exception;
        DBGKM_LOAD_DLL LoadDll;
        DBGKM_UNLOAD_DLL UnloadDll;
    } StateInfo;
} DBGUI_WAIT_STATE_CHANGE, *PDBGUI_WAIT_STATE_CHANGE;
```

The fields should be pretty self-explanatory, so let's look at the `DBG_STATE` enumeration:

这个结构各个域应该是很好的自我解释，所以我们来看看`DBG_STATE`枚举：

```
//
// Debug States
//

typedef enum _DBG_STATE
{
    DbgIdle,
    DbgReplyPending,
    DbgCreateThreadStateChange,
    DbgCreateProcessStateChange,
    DbgExitThreadStateChange,
    DbgExitProcessStateChange,
    DbgExceptionStateChange,
    DbgBreakpointStateChange,
    DbgSingleStepStateChange,
    DbgLoadDllStateChange,
    DbgUnloadDllStateChange
} DBG_STATE, *PDBG_STATE;
```

If you take a look at the Win32 `DEBUG_EVENT` structure and associated debug event types, you'll notice some differences which might be useful to you. For starters, Exceptions, Breakpoints and Single Step exceptions are handled differently. In the Win32 world, only two distinctions are made: `RIP_EVENT` for exceptions, and `EXCEPTION_DEBUG_EVENT` for a debug event. Although code can later figure out if this was a breakpoint or single step, this information comes directly in the native structure. You will also notice that `OUTPUT_DEBUG_STRING` event is missing. Here, it's `DbgUi` that's at a disadvantage, since the information is sent as an Exception, and post-processing is required (which we'll take a look at soon). There are also two more states that Win32 does not support, which is the Idle state and the Reply Pending state. These don't offer much information from the point of view of a debugger, so they are ignored.

如果你对比一下Win32 `DEBUG_EVENT`结构和相关的调试事件类型，你将会注意到一些差异，这些差异可能对你有用（我把`DEBUG_EVENT`结构定义加到下面）。首先，异常，断点和单步异常的处理方式不同。在Win32世界里，只有两个区别：`RIP_EVENT`异常和`EXCEPTION_DEBUG_EVENT`调试事件。虽然，代码在后面能够搞清楚这是否是一个断点或者单步，这个信息直接来自本地结构。你同时也会注意到`OUTPUT_DEBUG_STRING`事件消失了。在这里，这是`DbgUi`处于劣势，因为这信息是作为一个异常发送的，同时后处理是必需的（我们很快就会看到）。还有两个状态是Win32不支持的，那是空闲（Idle）状态和回复未决（Reply Pending）状态。从一个调试器的角度来看，这些没有提供太多的信息，所以它们被忽略了。

```
typedef struct _DEBUG_EVENT {
        DWORD dwDebugEventCode;
        DWORD dwProcessId;  DWORD dwThreadId;
        union {
                EXCEPTION_DEBUG_INFO Exception;
                CREATE_THREAD_DEBUG_INFO CreateThread;
                CREATE_PROCESS_DEBUG_INFO CreateProcessInfo;
                EXIT_THREAD_DEBUG_INFO ExitThread;
                EXIT_PROCESS_DEBUG_INFO ExitProcess;
                LOAD_DLL_DEBUG_INFO LoadDll;
                UNLOAD_DLL_DEBUG_INFO UnloadDll;
                OUTPUT_DEBUG_STRING_INFO DebugString;
                RIP_INFO RipInfo;
        }u;
} DEBUG_EVENT,  *LPDEBUG_EVENT;
```

Now let's take a look at the actual structures seen in the unions:

现在来看看在共同体中的那些实际结构体定义：

```
//
// Debug Message Structures
//

typedef struct _DBGKM_EXCEPTION
{
    EXCEPTION_RECORD ExceptionRecord;
    ULONG FirstChance;
} DBGKM_EXCEPTION, *PDBGKM_EXCEPTION;

typedef struct _DBGKM_CREATE_THREAD
{
    ULONG SubSystemKey;
    PVOID StartAddress;
} DBGKM_CREATE_THREAD, *PDBGKM_CREATE_THREAD;

typedef struct _DBGKM_CREATE_PROCESS
{
    ULONG SubSystemKey;
    HANDLE FileHandle;
    PVOID BaseOfImage;
    ULONG DebugInfoFileOffset;
    ULONG DebugInfoSize;
    DBGKM_CREATE_THREAD InitialThread;
} DBGKM_CREATE_PROCESS, *PDBGKM_CREATE_PROCESS;

typedef struct _DBGKM_EXIT_THREAD
{
    NTSTATUS ExitStatus;
} DBGKM_EXIT_THREAD, *PDBGKM_EXIT_THREAD;

typedef struct _DBGKM_EXIT_PROCESS
{
    NTSTATUS ExitStatus;
} DBGKM_EXIT_PROCESS, *PDBGKM_EXIT_PROCESS;

typedef struct _DBGKM_LOAD_DLL
{
    HANDLE FileHandle;
    PVOID BaseOfDll;
    ULONG DebugInfoFileOffset;
    ULONG DebugInfoSize;
    PVOID NamePointer;
} DBGKM_LOAD_DLL, *PDBGKM_LOAD_DLL;

typedef struct _DBGKM_UNLOAD_DLL
{
    PVOID BaseAddress;
} DBGKM_UNLOAD_DLL, *PDBGKM_UNLOAD_DLL;
```

If you're familiar with the `DEBUG_EVENT` structure, you should notice some subtle differences. First of all, no indication of the process name, which explains why MSDN documents this field being optional and not used by Win32. You will also notice the lack of a pointer to the `TEB` in the thread structure. Finally, unlike new processes, Win32 does display the name of any new DLL loaded, but this also seems to be missing in the Load DLL structure; we'll see how this and other changes are dealt with soon. As far as extra information goes however, we have the `SubsystemKey` field. Because NT was designed to support multiple subsystems, this field is critical to identifying from which subsystem the new thread or process was created from. Windows 2003 SP1 adds support for debugging POSIX applications, and while I haven't looked at the POSIX debug APIs, I'm convinced they're built around the `DbgUi` implementation, and that this field is used differently by the POSIX library (much like Win32 ignores it).

如果你熟悉`DEBUG_EVENT`结构，你应该注意到一些细微的差异。首先，没有迹象表明该进程的名称，这解释了为什么MSDN文档说明这个字段是可选的，同时不被Win32所使用。你同时也应注意到在线程结构中缺少指向`TEB`结构的指针。最后，不像新进程，Win32显示任何新加载的DLL的名称，但这似乎也不存在于Load DLL结构中；我们将很快看到这个和其它一些变化是怎样被处理的。至于额外的信息，我们有`SubsystemKey`这个字段。因为NT被设计成支持多个子系统，这个字段是关键，用于标明新线程或者新进程是由那个子系统所创建的。Windows 2003 SP1添加了对POSIX应用程序的调试支持，而我还没看到POSIX调试API，我确信它们就是建立在`DbgUi`的实现上，这个字段被POSIX库不同地使用（很像Win32忽略它）。
