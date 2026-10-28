
```
NTSTATUS

NTAPI

KeDelayExecutionThread(IN KPROCESSOR_MODE WaitMode,

                       IN BOOLEAN Alertable,

                       IN PLARGE_INTEGER Interval OPTIONAL)

{

    PKTIMER Timer;

    PKWAIT_BLOCK TimerBlock;

    PKTHREAD Thread = KeGetCurrentThread();

    NTSTATUS WaitStatus;

    BOOLEAN Swappable;

    PLARGE_INTEGER OriginalDueTime;

    LARGE_INTEGER DueTime, NewDueTime, InterruptTime;

    ULONG Hand = 0;



    if (Thread->WaitNext)

        ASSERT(KeGetCurrentIrql() == DISPATCH_LEVEL);

    else

        ASSERT(KeGetCurrentIrql() <= APC_LEVEL);



    /* If this is a user-mode wait of 0 seconds, yield execution */

    if (!(Interval->QuadPart) && (WaitMode != KernelMode))

    {

        /* Make sure the wait isn't alertable or interrupting an APC */

        if (!(Alertable) && !(Thread->ApcState.UserApcPending))

        {

            /* Yield execution */

            return NtYieldExecution();

        }

    }



    /* Setup the original time and timer/wait blocks */

    OriginalDueTime = Interval;

    Timer = &Thread->Timer;

    TimerBlock = &Thread->WaitBlock[TIMER_WAIT_BLOCK];



    /* Check if the lock is already held */

    if (!Thread->WaitNext) goto WaitStart;



    /*  Otherwise, we already have the lock, so initialize the wait */

    Thread->WaitNext = FALSE;

    KxDelayThreadWait();



    /* Start wait loop */

    for (;;)

    {

        /* Disable pre-emption */

        Thread->Preempted = FALSE;



        /* Check if a kernel APC is pending and we're below APC_LEVEL */

        if ((Thread->ApcState.KernelApcPending) && !(Thread->SpecialApcDisable) &&

            (Thread->WaitIrql < APC_LEVEL))

        {

            /* Unlock the dispatcher */

            KiReleaseDispatcherLock(Thread->WaitIrql);

        }

        else

        {

            /* Check if we have to bail out due to an alerted state */

            WaitStatus = KiCheckAlertability(Thread, Alertable, WaitMode);  // Alertable为TRUE以及UserMode时置位

            if (WaitStatus != STATUS_WAIT_0) break;



            /* Check if the timer expired */

            InterruptTime.QuadPart = KeQueryInterruptTime();

            if ((ULONGLONG)InterruptTime.QuadPart >= Timer->DueTime.QuadPart)

            {

                /* It did, so we don't need to wait */

                goto NoWait;

            }



            /* It didn't, so activate it */

            Timer->Header.Inserted = TRUE;



            /* Handle Kernel Queues */

            if (Thread->Queue) KiActivateWaiterQueue(Thread->Queue);



            /* Setup the wait information */

            Thread->State = Waiting;



            /* Add the thread to the wait list */

            KiAddThreadToWaitList(Thread, Swappable);



            /* Insert the timer and swap the thread */

            ASSERT(Thread->WaitIrql <= DISPATCH_LEVEL);

            KiSetThreadSwapBusy(Thread);

            KxInsertTimer(Timer, Hand);

            WaitStatus = (NTSTATUS)KiSwapThread(Thread, KeGetCurrentPrcb());



            /* Check if were swapped ok */

            if (WaitStatus != STATUS_KERNEL_APC)

            {

                /* This is a good thing */

                if (WaitStatus == STATUS_TIMEOUT) WaitStatus = STATUS_SUCCESS;



                /* Return Status */

                return WaitStatus;

            }



            /* Recalculate due times */

            Interval = KiRecalculateDueTime(OriginalDueTime,

                                            &DueTime,

                                            &NewDueTime);

        }



WaitStart:

        /* Setup a new wait */

        Thread->WaitIrql = KeRaiseIrqlToSynchLevel();

        KxDelayThreadWait();

        KiAcquireDispatcherLockAtDpcLevel();

    }



    /* We're done! */

    KiReleaseDispatcherLock(Thread->WaitIrql);

    return WaitStatus;



NoWait:

    /* There was nothing to wait for. Did we have a wait interval? */

    if (!Interval->QuadPart)

    {

        /* Unlock the dispatcher and do a yield */

        KiReleaseDispatcherLock(Thread->WaitIrql);

        return NtYieldExecution();

    }



    /* Unlock the dispatcher and adjust the quantum for a no-wait */

    KiReleaseDispatcherLockFromDpcLevel();

    KiAdjustQuantumThread(Thread);

    return STATUS_SUCCESS;

}



FORCEINLINE

NTSTATUS

KiCheckAlertability(IN PKTHREAD Thread,

                    IN BOOLEAN Alertable,

                    IN KPROCESSOR_MODE WaitMode)

{

    /* Check if the wait is alertable */

    if (Alertable)

    {

        /* It is, first check if the thread is alerted in this mode */

        if (Thread->Alerted[WaitMode])

        {

            /* It is, so bail out of the wait */

            Thread->Alerted[WaitMode] = FALSE;

            return STATUS_ALERTED;

        }

        else if ((WaitMode != KernelMode) &&

                (!IsListEmpty(&Thread->ApcState.ApcListHead[UserMode])))

        {

            /* It's isn't, but this is a user wait with queued user APCs */

            Thread->ApcState.UserApcPending = TRUE; // 前提是队列里一定有未决APC

            return STATUS_USER_APC;

        }

        else if (Thread->Alerted[KernelMode])

        {

            /* It isn't that either, but we're alered in kernel mode */

            Thread->Alerted[KernelMode] = FALSE;

            return STATUS_ALERTED;

        }

    }

    else if ((WaitMode != KernelMode) && (Thread->ApcState.UserApcPending))

    {

        /* Not alertable, but this is a user wait with pending user APCs */

        return STATUS_USER_APC;

    }



    /* Otherwise, we're fine */

    return STATUS_WAIT_0;

}
```

但是调用这些API，尤其是在进程在PE导入表初始化阶段，跟其他产商都使用APC的时候非常容易出现兼容问题，因为这些API一调用，PE初始化过程立即被抢占转而执行APC，而ShellCode调用的API 可能有`LdrGetProcedureAddress`这些默认在PE初始化完成时才可以调用的API。

`LdrGetProcedureAddress`提前执行那么即将引起系统DLL的`DllMain`的提前初始化......

而且有些情况，不小心代码执行到了`FreeLibrary`，他会检查当前链表中是否有引用计数为0的Dll，如果有那么就会将这些Dll全部卸载。在PE初始化未完成时，系统Dll一般都会有引用计数，但是一些软件自己的Dll以导入表的方式被其他模块引用的情况下很可能在`FreeLibrary`检查时被卸载。

其实像这类API都应该被禁用，因为太不稳定了，而且APC只要是在进程的PE初始化阶段被插入就可以执行起来，因为PE初始化完成后会自行调用`ZwTestAlert`。线程刚刚处于可觉醒状态那么就标志着他初始化工作已经完毕。

### 三. 吐槽

细看反汇编代码，真的是漏洞百出，实在不忍心吐槽了，反正需要改的地方我能帮他改的都改了。。

不知道你们有没有升级到Win10 RS4正式版，这个Device Guard跟vmware冲突，搞死我了，还好看了vmware官网的解决办法关了DeviceGuard。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2018-6-16 20:24 被FaEry编辑，原因：

上传的附件：

- [HWInject.zip](javascript:void(0);) （129.60kb，123次下载）
