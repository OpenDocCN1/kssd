
`VOID WINAPI testThreadA() {`  `while` `(TRUE) {`  `Sleep(``1000``);`  `printf(``"AAA\n"``);`  `}`  `}`  `HANDLE threadA` `=` `-``1``;`  `int` `main()`  `{`  `threadA` `=` `CreateThread(``0``,` `0``, (LPTHREAD_START_ROUTINE)testThreadA,` `0``,` `0``,` `0``);`  `SuspendThread(threadA);`  `__asm {`  `int` `3``;`  `}`  `ResumeThread(threadA);`  `system(``"pause"``);`  `return` `0``;`  `}`

## 基本流程

```
ResumeThread(ThreadHandle);
/↓
NtResumeThread(ThreadHandle);
/↓
PsResumeThread(Thread);
/↓
KeResumeThread(Thread);
/↓
KiWaitTest(Thread->SuspendSemaphore,increment(0));
/↓
KiUnwaitThread(increment(0),OUT &waitListEntry);
/↓
KiUnlinkThread(Thread,increment(0)); //断链WaitBlockList
/↓
KiReadyThread(); //挂入调度链表
```

## 调用关系

```
KiWaitTest -> KiUnwaitThread -> KiUnlinkThread
-> KiReadyThread
```

## 函数分析

以下操作流程用伪代码表示（非IDA）（if均成立）

### KeResumeThread

```
int __stdcall KeResumeThread(_ETHREAD *Thread)
{
    char SuspendCount = Thread->Tcb.SuspendCount;
    if ( SuspendCount ) //如果没调用过SuspendThread()挂起线程，该值为0
    {
        char newSuspendCount = SuspendCount - 1;
        Thread->Tcb.SuspendCount = newSuspendCount;
        if ( !newSuspendCount && !Thread->Tcb.FreezeCount )
        {
            ++Thread->Tcb.SuspendSemaphore.Header.SignalState;
            KiWaitTest(&Thread->Tcb.SuspendSemaphore, 0);
        }
    }
    KiUnlockDispatcherDatabase(KeRaiseIrqlToDpcLevel());
    return;
}
```

### KiWaitTest

```
__fastcall KiWaitTest(_KEVENT *Event, _KTHREAD *Increment)
{
    LIST_ENTRY waitListEntry;
    waitListEntry.Blink = &waitListEntry;
    waitListEntry.Flink = &waitListEntry;

    PLIST_ENTRY ListHead = &Event->Header.WaitListHead;
    PKWAIT_BLOCK = WaitEntry = Event->Header.WaitListHead.Flink;
    if(Event->Header.SignalState > 0){
        if ( WaitEntry != ListHead ){
            PKTHREAD Thread = WaitEntry->Thread;
            if(WaitEntry->WaitType == WaitAny){
                USHORT waitStatus = WaitEntry->WaitKey;
                UCHAR Type = Event->Header.Type;
            }
            //获得一个需要挂入调度链表的就绪线程链表
            KiUnwaitThread(Thread, waitStatus, increment, &waitListEntry);
            WaitEntry = ListHead->Flink;
            while(IsListEmpty(&waitListEntry)){
                PVOID nextThread = waitListEntry.Flink;
                WaitEntry = &waitListEntry;
                waitListEntry.Flink = waitListEntry.Flink->Flink;
                waitListEntry.Flink->Blink = &waitListEntry;
                KiReadyThread((KTHREAD *)(nextThread - 96));
            }
        }
    }
}
```

### KiUnwaitThread()

```
KiUnwaitThread(_KTHREAD *thread, LONG_PTR WaitStatus, int Increment, OUT _LIST_ENTRY *WaitListEntry){
    //将线程的等待块摘除
    KiUnlinkThread(thread, WaitStatus);
    CHAR Priority = thread->Priority;
    KPROCESS Process = thread->ApcState.Process;
    if ( Priority >= 16 ){
        if(WaitListEntry != NULL){
            PLIST_ENTRY prev = WaitListEntry->Blink;
            PLIST_ENTRY waitList = &thread->WaitListEntry;
            thread->WaitListEntry.Flink = WaitListEntry;
            thread->WaitListEntry.Blink = Blink;
            prev->Flink = &thread->WaitListEntry;
            WaitListEntry->Blink = &thread->WaitListEntry;
            return;
        }
    }
}
```

### KiUnlinkThread()

```
BOOL __fastcall KiUnlinkThread(_KTHREAD *Thread, LONG_PTR WaitStatus){
    Thread->WaitStatus |= WaitStatus;
    WaitBlockList = Thread->WaitBlockList;
    do{//将Thread从等待链中断链
        PLIST_ENTRY Flink = WaitBlockList->WaitListEntry.Flink;
        PLIST_ENTRY Blink = WaitBlockList->WaitListEntry.Blink;
        Blink->Flink = WaitBlockList->WaitListEntry.Flink;
        Flink->Blink = Blink;
        WaitBlockList = WaitBlockList->NextWaitBlock;
    }while ( WaitBlockList != Thread->WaitBlockList );
    return;
}
```

### KiReadyThread()

```
KiReadyThread(_KTHREAD *Thread){
    //设置字段属性
    Thread->State = Standby;
    Thread->WaitTime = KeTickCount
    Thread->NextProcessor = Thread->IdleProcessor
    Thread->State = Ready;

    //将线程挂入调度链表
    Thread->WaitListEntry.Flink = KiDispatcherReadyListHead
    Thread->WaitListEntry.Blink = KiDispatcherReadyListHead.Blink
    KiDispatcherReadyListHead.Blink.Flink = Thread->WaitListEntry
    KiDispatcherReadyListHead.Blink = Thread->WaitListEntry
    KiReadySummary |= 1 << Thread->priority;
}
```

### KiReadyThread() /汇编/
