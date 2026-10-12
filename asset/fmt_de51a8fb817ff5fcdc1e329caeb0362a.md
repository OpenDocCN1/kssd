
以上汇编函数全部用于 KOL 纯 pascal 环境。

本贴将不断更新函数,并修复原有函数的 BUG,学习 DELPHI 内联汇编的朋友也可以更贴。

汇编函数自然是手工写的，但是调试的话也只能靠 OD 调试正确性了，

至于 API 的实现 比如你说的 `ExitProcess` 这 API 虽然位于 `kernel32.dll` 但是他依然调用了 `ntdll.dll` ，说明是在调用底层驱动, 比如 `ZwTerminateProcess` ，所以你如果要实现，你得从 R0 下手了。

`NTSTATUS`

`NtTerminateProcess(`
`__in_opt` `HANDLE` `ProcessHandle,`
`__in` `NTSTATUS` `ExitStatus`
`)`

```c
/*++

Routine Description:

    This function causes the specified process and all of
    its threads to terminate.

Arguments:

    ProcessHandle - Supplies a handle to the process to terminate.

    ExitStatus - Supplies the exit status associated with the process.

Return Value:

    NTSTATUS - Status of operation

--*/

{
    PETHREAD Thread, Self;
    PEPROCESS Process;
    PEPROCESS CurrentProcess;
    NTSTATUS st;
    BOOLEAN ProcessHandleSpecified;

    PAGED_CODE();

    Self = PsGetCurrentThread();

    CurrentProcess = PsGetCurrentProcessByThread (Self);

    if (ARGUMENT_PRESENT (ProcessHandle)) {
        ProcessHandleSpecified = TRUE;
    } else {
        ProcessHandleSpecified = FALSE;
        ProcessHandle = NtCurrentProcess();
    }

    st = ObReferenceObjectByHandle (ProcessHandle,
                                    PROCESS_TERMINATE,
                                    PsProcessType,
                                    KeGetPreviousModeByThread(&Self->Tcb),
                                    &Process,
                                    NULL);

    if (!NT_SUCCESS (st)) {
        return(st);
    }

    if (Process->Flags & PS_PROCESS_FLAGS_BREAK_ON_TERMINATION) {
        PspCatchCriticalBreak ("Terminating critical process 0x%p (%s)\n",
                                Process,
                                Process->ImageFileName);
    }

    //
    // Acquire rundown protection just so we can give the right errors
    //
    if (!ExAcquireRundownProtection (&Process->RundownProtect)) {
        ObDereferenceObject (Process);
        return STATUS_PROCESS_IS_TERMINATING;
    }

    //
    // Mark process as deleting except for the obscure delete self case.
    //
    if (ProcessHandleSpecified) {
        PS_SET_BITS (&Process->Flags, PS_PROCESS_FLAGS_PROCESS_DELETE);
    }

    st = STATUS_NOTHING_TO_TERMINATE;
    for (Thread = PsGetNextProcessThread (Process, NULL);
         Thread != NULL;
         Thread = PsGetNextProcessThread (Process, Thread)) {
        st = STATUS_SUCCESS;
        if (Thread != Self) {
            PspTerminateThreadByPointer (Thread, ExitStatus, FALSE);
        }
    }

    ExReleaseRundownProtection (&Process->RundownProtect);

    if (Process == CurrentProcess) {
        if (ProcessHandleSpecified) {
            ObDereferenceObject (Process);
            //
            // Never Returns
            //
            PspTerminateThreadByPointer (Self, ExitStatus, TRUE);
        }
    } else if (ExitStatus == DBG_TERMINATE_PROCESS) {
        DbgkClearProcessDebugObject (Process, NULL);
    }

    //
    // If there are no threads in this process then clear out its handle table.
    // Do the same for processes being debugged. This is so a process can never lock itself into the system
    // by debugging itself or have a handle open to itself.
    //
    if (st == STATUS_NOTHING_TO_TERMINATE || (Process->DebugPort != NULL && ProcessHandleSpecified)) {
        ObClearProcessHandleTable (Process);
        st = STATUS_SUCCESS;
    }

    ObDereferenceObject(Process);
    return st;
}
```
