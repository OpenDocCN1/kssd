
# [原创]调试陷阱`ThreadHideFromDebugger`的另一种对抗方法

测试环境： windows 7 x64

炒个冷饭，说说`ThreadHideFromDebugger`的另一种对抗方法，之所以说是另一种是因为第一种方法无论是r3还是r0都可以用hook解决。

接下来说说另一种方法，先来看一个函数定义：

```c
NTSTATUS
NTAPI
NtSetInformationThread(
    _In_ HANDLE ThreadHandle,
    _In_ THREADINFOCLASS ThreadInformationClass,
    _In_reads_bytes_(ThreadInformationLength) PVOID ThreadInformation,
    _In_ ULONG ThreadInformationLength
    );
```

它是这么用的：

```c
NtSetInformationThread(handle.get(), ThreadHideFromDebugger, nullptr, 0);
```

再来看一个结构定义：

```c
typedef enum _THREADINFOCLASS {
    ThreadBasicInformation          = 0,
    ThreadTimes                     = 1,
    ThreadPriority                  = 2,
    ThreadBasePriority              = 3,
    ThreadAffinityMask              = 4,
    ThreadImpersonationToken        = 5,
    ThreadDescriptorTableEntry      = 6,
    ThreadEnableAlignmentFaultFixup = 7,
    ThreadEventPair_Reusable        = 8,
    ThreadQuerySetWin32StartAddress = 9,
    ThreadZeroTlsCell               = 10,
    ThreadPerformanceCount          = 11,
    ThreadAmILastThread             = 12,
    ThreadIdealProcessor            = 13,
    ThreadPriorityBoost             = 14,
    ThreadSetTlsArrayAddress        = 15,   // Obsolete
    ThreadIsIoPending               = 16,
    ThreadHideFromDebugger          = 17,
    ThreadBreakOnTermination        = 18,
    ThreadSwitchLegacyState         = 19,
    ThreadIsTerminated              = 20,
    ThreadLastSystemCall            = 21,
    ThreadIoPriority                = 22,
    ThreadCycleTime                 = 23,
    ThreadPagePriority              = 24,
    ThreadActualBasePriority        = 25,
    ThreadTebInformation            = 26,
    ThreadCSwitchMon                = 27,   // Obsolete
    ThreadCSwitchPmu                = 28,
    ThreadWow64Context              = 29,
    ThreadGroupInformation          = 30,
    ThreadUmsInformation            = 31,   // UMS
    ThreadCounterProfiling          = 32,
    ThreadIdealProcessorEx          = 33,
    ThreadCpuAccountingInformation  = 34,
    ThreadSuspendCount              = 35,
    ThreadActualGroupAffinity       = 41,
    ThreadDynamicCodePolicyInfo     = 42,
    ThreadSubsystemInformation      = 45,
    MaxThreadInfoClass              = 51,
} THREADINFOCLASS;
```

今天我们只关心`ThreadHideFromDebugger`，由上面的结构可以看出它的值是：0x11。

好，那我们看下对于这个参数，`NtSetInformationThread`是怎么实现的：

```c
    case ThreadHideFromDebugger:
        if (ThreadInformationLength != 0) {
            return STATUS_INFO_LENGTH_MISMATCH;
        }

        st = ObReferenceObjectByHandle (ThreadHandle,
                                        THREAD_SET_INFORMATION,
                                        PsThreadType,
                                        PreviousMode,
                                        &Thread,
                                        NULL);

        if (!NT_SUCCESS (st)) {
            return st;
        }

        PS_SET_BITS (&Thread->CrossThreadFlags, PS_CROSS_THREAD_FLAGS_HIDEFROMDBG);

        ObDereferenceObject (Thread);

        return st;
        break;
```

接下来，来看看调试子系统对于有“`PS_CROSS_THREAD_FLAGS_HIDEFROMDBG`”这个标志的线程是怎么处理的：

```c
    Process = PsGetCurrentProcess();
    if (DebugException) {
        if (PsGetCurrentThread()->CrossThreadFlags&PS_CROSS_THREAD_FLAGS_HIDEFROMDBG) {
            Port = NULL;
        } else {
            Port = Process->DebugPort;
        }
        LpcPort = FALSE;
    } else {
        Port = Process->ExceptionPort;
        m.h.u2.ZeroInit = LPC_EXCEPTION;
        LpcPort = TRUE;
    }
```

emmm...破案了，原来清空`DebugPort`不是某P首创，人家巨硬的内核里本来就是这么做的，只不过一个是暴力清空而另一个是“假装没有”。

这也就非常符合微软对“`ThreadHideFromDebugger`”这个参数的描述：设置此参数将使这条线程对调试器“隐藏”。即调试器收不到调试信息，因而就会出现当调试器对被调试进程下断点，线程执行到被下断点代码的时候就会卡死这种现象。

OK，接下来我们考虑一下怎么干掉它。

看以下定义：

```c
#define RtlInterlockedSetBitsDiscardReturn(Flags, Flag) \
    (VOID) RtlInterlockedSetBits(Flags, Flag)

#define PS_SET_BITS(Flags, Flag) \
    RtlInterlockedSetBitsDiscardReturn (Flags, Flag)
```

还记得这句代码吗？

```c
PS_SET_BITS (&Thread->CrossThreadFlags, PS_CROSS_THREAD_FLAGS_HIDEFROMDBG);
```

从这句代码我们得到了关键点：`CrossThreadFlags`

看一下`ETHREAD`的结构：

`dt _ETHREAD`
