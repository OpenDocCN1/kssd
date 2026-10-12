
# 伪造句柄方法结束进程

> 原文：[`www.pediy.com/kssd/pediy11/115012.html`](https://www.pediy.com/kssd/pediy11/115012.html)

伪造句柄方法结束进程(无需驱动)

2010-6-13

系统为 WinXp

`XP_5.1.2600.2180`

`EULAID: XPSP2_RM.0_PRO_RTL_CN`

## TerminateProcess 函数分析

`TerminateProcess`

`NtTerminateProcess`

`PspTerminateThreadByPointer(ObReferenceObjectByHandle, PsGetNextProcessThread(loop))`

- `->Self: PspExitThread`
- `->Other: KeInsertQueueApc, KiInsertQueueApc`

```
/*
 * @implemented
 */

NTSTATUS
NTAPI
NtTerminateProcess(IN HANDLE ProcessHandle OPTIONAL,
                   IN NTSTATUS ExitStatus)
{
    NTSTATUS Status;
    PEPROCESS Process, CurrentProcess = PsGetCurrentProcess();
    PETHREAD Thread, CurrentThread = PsGetCurrentThread();
    BOOLEAN KillByHandle;

    PAGED_CODE();

    PSTRACE(PS_KILL_DEBUG,
            "ProcessHandle: %p ExitStatus: %p\n", ProcessHandle, ExitStatus);

    /* Were we passed a process handle? */
    if (ProcessHandle)
    {
        /* Yes we were, use it */
        KillByHandle = TRUE;
    }
    else
    {
        /* We weren't... we assume this is suicide */
        KillByHandle = FALSE;
        ProcessHandle = NtCurrentProcess();
    }

    /* Get the Process Object */
    Status = ObReferenceObjectByHandle(ProcessHandle,
                                       PROCESS_TERMINATE,
                                       PsProcessType,
                                       KeGetPreviousMode(),
                                       (PVOID*)&Process,
                                       NULL);

    if (!NT_SUCCESS(Status)) return(Status);

    /* Check if this is a Critical Process, and Bugcheck */
    if (Process->BreakOnTermination)
    {
        /* Break to debugger */
        PspCatchCriticalBreak("Terminating critical process 0x%p (%s)\n",
                             Process,
                             Process->ImageFileName);
    }

    /* Lock the Process */
    if (!ExAcquireRundownProtection(&Process->RundownProtect))
    {
        /* Failed to lock, fal */
        ObDereferenceObject (Process);
        return STATUS_PROCESS_IS_TERMINATING;
    }

    /* Set the delete flag, unless the process is comitting suicide */
    if (KillByHandle) PspSetProcessFlag(Process, PSF_PROCESS_DELETE_BIT);

    /* Get the first thread */
    Status = STATUS_NOTHING_TO_TERMINATE;
    Thread = PsGetNextProcessThread(Process, NULL);
    if (Thread)
    {
        /* We know we have at least a thread */
        Status = STATUS_SUCCESS;
        /* Loop and kill the others */
        do
        {
            /* Ensure it's not ours*/
            if (Thread != CurrentThread)
            {
                /* Kill it */
                PspTerminateThreadByPointer(Thread, ExitStatus, FALSE);
            }
            /* Move to the next thread */
            Thread = PsGetNextProcessThread(Process, Thread);
        } while (Thread);
    }

    /* Unlock the process */
    ExReleaseRundownProtection(&Process->RundownProtect);

    /* Check if we are killing ourselves */
    if (Process == CurrentProcess)
    {
        /* Also make sure the caller gave us our handle */
        if (KillByHandle)
        {
            /* Dereference the project */
            ObDereferenceObject(Process);
            /* Terminate ourselves */
            PspTerminateThreadByPointer(CurrentThread, ExitStatus, TRUE);
        }
    }
    else if (ExitStatus == DBG_TERMINATE_PROCESS)
    {
        /* Disable debugging on this process */
        DbgkClearProcessDebugObject(Process, NULL);
    }

    /* Check if there was nothing to terminate, or if we have a Debug Port */
    if ((Status == STATUS_NOTHING_TO_TERMINATE) ||
        ((Process->DebugPort) && (KillByHandle)))
    {
        /* Clear the handle table */
        ObClearProcessHandleTable(Process);
        /* Return status now */
        Status = STATUS_SUCCESS;
    }

    /* Decrease the reference count we added */
    ObDereferenceObject(Process);

    /* Return status */
    return Status;
}

/*  H:\ntoskrnl\ps\kill.c
 * See "Windows Internals" - Chapter 13, Page 49
 */

NTSTATUS
NTAPI
PspTerminateThreadByPointer(IN PETHREAD Thread,
                           IN NTSTATUS ExitStatus,
                           IN BOOLEAN bSelf)
{
    PKAPC Apc;
    NTSTATUS Status = STATUS_SUCCESS;
    ULONG Flags;

    PAGED_CODE();

    PSTRACE(PS_KILL_DEBUG, "Thread: %p ExitStatus: %p\n", Thread, ExitStatus);

    PSREFTRACE(Thread);

    /* Check if this is a Critical Thread, and Bugcheck */
    if (Thread->BreakOnTermination)
    {
        /* Break to debugger */
        PspCatchCriticalBreak("Terminating critical thread 0x%p (%s)\n",
                             Thread,
                             Thread->ThreadsProcess->ImageFileName);
    }

    /* Check if we are already inside the thread */
    if ((bSelf) || (PsGetCurrentThread() == Thread))
    {
        /* This should only happen at passive */
        ASSERT_IRQL_EQUAL(PASSIVE_LEVEL);
        /* Mark it as terminated */
        PspSetCrossThreadFlag(Thread, CT_TERMINATED_BIT);
        /* Directly terminate the thread */
        PspExitThread(ExitStatus);
    }

    /* This shouldn't be a system thread */
    if (Thread->SystemThread) return STATUS_ACCESS_DENIED;

    /* Allocate the APC */
    Apc = ExAllocatePoolWithTag(NonPagedPool, sizeof(KAPC), TAG_TERMINATE_APC);

    /* Set the Terminated Flag */
    Flags = Thread->CrossThreadFlags | CT_TERMINATED_BIT;

    /* Set it, and check if it was already set while we were running */
    if (!(InterlockedExchange((PLONG)&Thread->CrossThreadFlags, Flags) &
          CT_TERMINATED_BIT))
    {
        /* Initialize a Kernel Mode APC to Kill the Thread */
        KeInitializeApc(Apc,
                        &Thread->Tcb,
                        OriginalApcEnvironment,
                        PsExitSpecialApc,
                        PspExitApcRundown,
                        PspExitNormalApc,
                        KernelMode,
                        (PVOID)ExitStatus);

        /* Insert it into the APC Queue */
        if (!KeInsertQueueApc(Apc, Apc, NULL, 2))
        {
            /* The APC was already in the queue, fail */
            ExFreePool(Apc);
            Status = STATUS_UNSUCCESSFUL;
        }
        else
        {
            /* Forcefully resume the thread and return */
            KeForceResumeThread(&Thread->Tcb);
            return Status;
        }
    }

    /* We failed, free the APC */
    ExFreePool(Apc);

    /* Return Status */
    return Status;
}
```

## 伪造句柄结束进程

创建一个进程句柄，修改 `EPROCESS_XP` 的 `ThreadListHead` 指向目标的进程的线程，然后调用 `TerminateProcess`

```
CreateProcessA
pKrnProc = (NNtKrn::EPROCESS_XP *)ShellProc.dwObjAddr;
dwWriteAddr = (DWORD)(__int64)&pKrnProc->ThreadListHead;
dwSize = sizeof(NNtKrn::LIST_ENTRY);
Nntdll::WriteKernelMem(dwWriteAddr, dwSize, &DestThreadList);  //写入首线程
```

结果: 蓝屏

### 分析

经 `WinDbg` 调试发现死循环 循环代码

```
    Thread = PsGetNextProcessThread(Process, NULL);
    if (Thread)
    {
        /* We know we have at least a thread */
        Status = STATUS_SUCCESS;
        /* Loop and kill the others */
        do
        {
            /* Ensure it's not ours*/
            if (Thread != CurrentThread)
            {
                /* Kill it */
```
