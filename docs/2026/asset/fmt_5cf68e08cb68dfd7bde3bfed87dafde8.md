
# [原创]QQ电脑管家中的 Hook 过程分析

作者：Fypher

最近对QQ电脑管家中的`TsFltMgr.sys`做了些分析，发现不少有用的东西，这里跟大家分享一下 `TsFltMgr` 对 `KiFastCallEntry` 的 Hook 过程。

虽然整个过程中并没有新的技术，但毕竟是面向市场的产品，从兼容性、安全性出发，工作过程中需要把问题考虑全面一些、处理问题时尽量细致，这些都是值得学习的地方。

我们从这个函数开始：

```
BOOLEAN StartWork()
{
    ULONG ulOsVersion;

    if (InitSafeBootMode)
        return FALSE;

    ulOsVersion = GetOsVersion();
    if (ulOsVersion !=  OS_VERSION_ERROR)
    {
        ULONG ulKiFastCallEntry_Detour;

        if (!InitGlobalVars())
            return FALSE;

        if (!InitFakeSysCallTable())
            return FASLE;

        if ( ulOsVersion >= OS_VERSION_VISTA )
            ulKiFastCallEntry_Detour = (ULONG)KiFastCallEntry_Detour_AfterVista;
        else
            ulKiFastCallEntry_Detour = (ULONG)KiFastCallEntry_Detour_BeforeVista;

        return Hook(g_ulHookPoint, ulKiFastCallEntry_Detour);
    }

    return FALSE;
}
```

说明一下，在这篇文章中，我贴出的代码剔除了真实的 `TsFltMgr` 中跟 Hook 过程关系不紧密的部分，为了方便阅读，我还会重新组织了一些函数调用关系。但我会保持与 Hook 相关的流程同 `TsFltMgr` 一致。

在 `StartWork()` 中，先判断系统是否运行在安全模式中（为了抢占先机，`TsFltMgr` 以boot方式启动），是的话就不 Hook，再根据系统的版本号选择 Detour 函数（`GetOsVersion()` 通过 `BuildNumber` 来判断版本）。为什么要选择Detour函数？因为在 Vista 前和 Vista 后，`KiFastCallEntry` 的流程有点小区别（`ebx` 和 `edx` 的问题，自己去看看就明白了）。

`InitFakeSysCallTable()` 是初始化一张 `FakeSyscallTable` 表，想知道这个表是干啥的可以看看我的上一篇文章《QQ电脑管家中的`TsFltMgr` Hook框架分析》：<http://bbs.pediy.com/showthread.php?t=146156>

`InitGlobalVars()` 是初始化一些全局变量：

```
BOOLEAN InitGlobalVars()
{
    ……
    // InitRegKeys();

    pSysMods = (BYTE *)GetSystemModules();    // 这个函数貌似有点小 bug
    pModInfo = (PSYSTEM_MODULE_INFORMATION)(pSysMods + 4);

    g_KernelBase = pModInfo->Base;
    g_KernelSize = pModInfo->Size;

    ExFreePool(pSysMods);

    RtlInitUnicodeString(&usRoutineName, L"KeServiceDescriptorTable");
    g_KeServiceDescriptorTable = (ULONG)MmGetSystemRoutineAddress(&usRoutineName);
    g_KiServiceTable = *(PULONG)g_KeServiceDescriptorTable;
    g_ServiceNumber = *(PULONG)(g_KeServiceDescriptorTable + 8);

    RtlInitUnicodeString(&usRoutineName, L"MmUserProbeAddress");
    g_MmUserProbeAddress = (ULONG)MmGetSystemRoutineAddress(&usRoutineName);

    ……

    // 从 KeAddSystemServiceTable 函数到开始做特征码搜索
    GetSSDTShadow(&g_ShadowServiceTable, &g_ShadowServiceNumber);

    g_ulHookPoint = FindHookPoint();          // 找 Hook 点
    g_JmpBack = g_ulHookPoint + 8;

    // 为什么要这样？看看 Detour 就明白了
    g_MmUserProbeAddress = *(PULONG)g_MmUserProbeAddress;

    ……
}
```

以上代码中，`GetSystemModules()` 的实现如下：

```
PBYTE GetSystemModules() {
    PBYTE pSysMods = NULL;
    ULONG ulSize = 0;

    ZwQuerySystemInformation(SystemModuleInformation, &ulSize, 0, &ulSize);

    pSysMods = (PULONG)ExAllocatePoolWithTag(PagedPool, ulSize, 'tPyF');

    if (pModInfo)
    {
        NTSTATUS = ZwQuerySystemInformation(SystemModuleInformation, pSysMods, ulSize, NULL);
        if (!NT_SUCCESS( status ))
        {
            ExFreePool(pSysMods);
            pSysMods = NULL;
        }
    }
    return pSysMods;
}
```

这个函数可能有点小bug，因为在两次 `ZwQuerySystemInformation()` 调用之间 `ulSize` 可能会发生变化，不过这种 bug 的诱发概率很小。

回到正题， `FindHookPoint()` 查找 Hook 点时，依然通过特征码搜索：

```
ULONG FindHookPoint()  {
    ……
    ulKiSystemService = GetAddr_KiSystemService();
    if ( ulKiSystemService < g_KernelBase || ulKiSystemService > g_KernelBase + g_KernelSize )
        return 0;

    for (ulAddr = ulKiSystemService; ulAddr < ulKiSystemService + 1024; ++ulAddr) {
        if (!ulAddr || !MmIsAddressValid((PVOID)ulAddr))
            break;
        if ( RtlCompareMemory((PVOID)ulAddr, &g_Signature, sizeof(g_Signature)) == sizeof(g_Signature) )
            return ulAddr;
    }
    return 0;
}
```

搜索的起始地址是 `KiSystemService`，`GetAddr_KiSystemService()` 通过查询IDT中 `0x2e` 中断的处理函数取得。从兼容性上考虑，比 `rdmsr` 的方式要好。

现在到了关键的 `Hook(g_ulHookPoint, ulKiFastCallEntry_Detour)` 调用，接下来就结合注释和代码呈现一下这个过程：

```
BOOLEAN Hook (ULONG ulHookPoint, ULONG ulDetourAddr)
{
    PMDL pMdl;
    ULONG ulNewVirtualAddr;
    ULONG i;
    KAFFINITY CpuAffinity;
    ULONG ulNumberOfActiveCpu;
    KIRQL OldIrql;
    BOOLEAN bRet = FALSE;

    ULONG ulCurrentCpu;

    // MDL 法去掉写保护，比去掉 CR0 写保护位要好，因为后者更依赖硬件特性
    pMdl = MakeAddrWritable(ulHookPoint, 16, &ulNewVirtualAddr);
    if (!pMdl)
        return FALSE;

    // 对单核和多核的情况分别处理
    CpuAffinity = KeQueryActiveProcessors();
    ulNumberOfActiveCpu = 0;

    for (i = 0; i < 32; ++i) {
        if ( (CpuAffinity >> i) & 1 )
            ++ulNumberOfActiveCpu;
    }

    if ( ulNumberOfActiveCpu == 1 )
    {
        //
        // 单核，直接　Hook
        //

        // 通过提升 IRQL 来保证线程不被抢占，与cli相比，减少了对硬件特性的依赖
        OldIrql = KeRaiseIrqlToDpcLevel();

        HookInternal(ulNewVirtualAddr, 0xe9909090, ulDetourAddr - ulHookPoint - 8);

        KeLowerIrql(OldIrql);

        bRet = TRUE;
    }
    else
    {
        //
        // 多核处理，插DPC，把其它CPU全挂在一个自旋锁上，然后再 Hook
        //
        KeInitializeSpinLock(&g_SpinLock);
        for (i = 0; i < sizeof(g_Dpcs) / sizeof(KDPC); ++i) {
            KeInitializeDpc(&g_Dpcs[i], DpcRoutine, NULL);
        }

        g_ulNumberOfRaisedCpu = 0;
        KeAcquireSpinLock(&g_SpinLock, &OldIrql);

        ulCurrentCpu = KeGetCurrentProcessorNumber();

        // 重新获取一次 ulNumberOfActiveCpu
        ulNumberOfActiveCpu = 0;

        for (i = 0; i < 32; ++i) {
            if ((CpuAffinity >> i) & 1) {
                ++ulNumberOfActiveCpu;
                if (i != ulCurrentCpu) {
                    KeSetTargetProcessorDpc(&g_Dpcs[i], (CCHAR)i);
                    KeSetImportanceDpc(&g_Dpcs[i], HighImportance);
                    KeInsertQueueDpc(&g_Dpcs[i], NULL, NULL);
                }
            }
        }

        // 在有限的时间里无法完成 Hook 就放弃，可能是为了避免卡死系统
        for (i = 0; i < 16; i ++) {
            ULONG ulTmp = 1000000;
            while (ulTmp)
                ulTmp--;

            if ( g_ulNumberOfRaisedCpu == ulNumberOfActiveCpu - 1 ) {
                HookInternal(ulNewVirtualAddr, 0xe9909090, ulDetourAddr - ulHookPoint - 8);
                bRet = TRUE;
                break;
            }
        }

        KeReleaseSpinLock(&g_SpinLock, OldIrql);
    }

    MmUnlockPages(pMdl);
    IoFreeMdl(pMdl);
    return bRet;
}
```

DPC历程只是简单地卡在自旋锁上：

```
VOID DpcRoutine(PKDPC pDpc, PVOID DeferredContext, PVOID SystemArgument1, PVOID SystemArgument2)
{
    KIRQL OldIrql;

    OldIrql = KeRaiseIrqlToDpcLevel();
    InterlockedIncrement(&g_ulNumberOfRaisedCpu);

    KeAcquireSpinLockAtDpcLevel(&g_SpinLock);
    KeReleaseSpinLockFromDpcLevel(&g_SpinLock);
    KeLowerIrql(OldIrql);
}
```

插DPC解决多核的同步问题我最初是在 《RootKits》一书上看到，不过相比书里的方法（DPC历程死循环）我觉得这里处理得更有技巧。

`MakeAddrWritable()` 也贴一下吧：

```
PMDL MakeAddrWritable (ULONG ulOldAddress, ULONG ulSize, ULONG * pulNewAddress) {
    PMDL pMdl = IoAllocateMdl((PVOID)ulOldAddress, ulSize, FALSE, TRUE, NULL);
    if ( pMdl )
    {
        PVOID pNewAddr;
        MmProbeAndLockPages(pMdl, KernelMode, IoWriteAccess);

        if ( pMdl->MdlFlags & (MDL_MAPPED_TO_SYSTEM_VA | MDL_SOURCE_IS_NONPAGED_POOL ))
            pNewAddr = pMdl->MappedSystemVa;
        else
            pNewAddr = MmMapLockedPagesSpecifyCache(pMdl, KernelMode, MmCached, NULL, FALSE, NormalPagePriority);

        if ( !pNewAddr ) {
            MmUnlockPages(pMdl);
            IoFreeMdl(pMdl);
            pMdl = 0;
        }

        if ( pulNewAddress )
            *pulNewAddress = (ULONG)pNewAddr;
    }
    return pMdl;
}
```

`MakeAddrWritable()` 中的 `MmMapLockedPagesSpecifyCache()` 比暴力改标志位要好一些。

最后还剩下一个 `HookInternal()` 做实际性的 Hook 工作：

```
VOID HookInternal(ULONG ulHookPoint, ULONG ulE9909090, ULONG ulJmpOffSet) {
    __asm {
        mov edi, ulHookPoint;

        mov eax, [edi];                 // orig ins
        mov edx, [edi + 4];             // orig ins

        mov ebx, ulE9909090;
        mov ecx, ulJmpOffSet;

        // Compare EDX:EAX with m64. If equal, set ZF and load ECX:EBX into m64.
        // Else, clear ZF and load m64 into EDX:EAX.
        lock cmpxchg8b qword ptr [edi];
    }
}
```
