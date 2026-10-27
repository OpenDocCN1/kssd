
```
ULONG64 MmGetNextSession;
ULONG64 MmQuitNextSession;
ULONG64 MmAttachSession;
ULONG64 MmDetachSession;
ULONG64 MmGetSessionIdEx;
ULONG64 MmIsSessionAddress;
ULONG64 MmIsAddressValid;
ULONG64 MmSessionGetWin32Callouts;
ULONG64 KeInsertQueueApc;
ULONG64 KeWaitForSingleObject;
ULONG64 Unknow5;
ULONG64 ExReferenceCallBackBlock;
ULONG64 ExGetCallBackBlockRoutine;
ULONG64 ExDereferenceCallBackBlock;
ULONG64 KiMarkBugCheckRegions;
ULONG64 PspEnumerateCallback;
ULONG64 CmpEnumerateCallback;
ULONG64 DbgEnumerateCallback;
ULONG64 ExpEnumerateCallback;
ULONG64 ExpGetNextCallback;
ULONG64 EmpCheckErrataList;
ULONG64 KiSchedulerApcTerminate;
ULONG64 KiSchedulerApc;
ULONG64 EmpCheckErrataList;
ULONG64 KiSwInterruptDispatch;
ULONG64 MmAllocatePagesForMdlEx;
ULONG64 MmAllocateMappingAddress;
ULONG64 MmMapLockedPagesWithReservedMapping;
ULONG64 MmUnmapReservedMapping;
ULONG64 KiSwInterruptDispatch+0x12a0;
ULONG64 KiSwInterruptDispatch+0x1310;
ULONG64 MmAcquireLoadLock;
ULONG64 MmReleaseLoadLock;
ULONG64 KeEnumerateQueueApc;
ULONG64 KeIsApcRunningThread;
ULONG64 KiSwInterruptDispatch+0xe70;
ULONG64 PsAcquireProcessExitSynchronization;
ULONG64 ObDereferenceProcessHandleTable
ULONG64 PsGetNextProcess;
ULONG64 PsQuitNextProcess;
ULONG64 MmIsSessionLeaderProcess;
ULONG64 PsInvokeWin32Callout;
ULONG64 MmEnumerateAddressSpaceAndReferenceImages;
ULONG64 PsGetProcessProtection;
ULONG64 PsGetProcessSignatureLevel;
ULONG64 PsGetProcessSectionBaseAddress;
ULONG64 SeCompareSigningLevels;
ULONG64 KeComputeSha256;
ULONG64 KeComputeParallelSha256;
ULONG64 KeSetEvent;
ULONG64 RtlpConvertFunctionEntry;
ULONG64 RtlpLookupPrimaryFunctionEntry;
ULONG64 RtlIsMultiSessionSku;
ULONG64 KiEnumerateCallback;
ULONG64 KeStackAttachProcess;
ULONG64 KeUnstackDetachProcess;
ULONG64 VslVerifyPage;
ULONG64 KiGetInterruptObjectAddress;
ULONG64 Unknow6;
ULONG64 Unknow7;
ULONG64 Unknow8;
ULONG64 Unknow9;
ULONG64 Unknow10;
ULONG64 Unknow11;
ULONG64 KiEntropyTimingRoutine;
ULONG64 KiProcessListHead;
ULONG64 KiProcessListLock;
ULONG64 Unknow12;//(pG  Encrypt?)
ULONG64 Unknow13;//(pG  Encrypt?)
ULONG64 PsActiveProcessHead;
ULONG64 PsInvertedFunctionTable;
ULONG64 PsLoadedModuleList;
ULONG64 PsLoadedModuleResource;
ULONG64 PsLoadedModuleSpinLock;
ULONG64 PspActiveProcessLock;
ULONG64 PspCidTable;
ULONG64 ExpUuidLock;
ULONG64 AlpcpPortListLock;
ULONG64 KeServiceDescriptorTable;
ULONG64 KeServiceDescriptorTableShadow;
ULONG64 KeServiceDescriptorTableFilter;
ULONG64 VfThunksExtended;
ULONG64 PsWin32CallBack;
ULONG64 TriageImagePageSize;
ULONG64 KiTableInformation;
ULONG64 HandleTableListHead;
ULONG64 SeNullSid;
ULONG64 Unknow14;
ULONG64 Unknow15;//(fffff780`00000000)
ULONG64 Unknow16;//(a9ad6402`bae5207b)
ULONG64 SeProtectedMapping;
ULONG64 $$0;//????
ULONG64 KiStackProtectNotifyEvent;
ULONG64 Unknow17;//??fffff900`00000000
ULONG64 RtlpInvertedFunctionTable;
ULONG64 KiIsrThunk;
ULONG64 Unknow17;
ULONG64 Unknow18;
ULONG64 Unknow19;
ULONG64 Unknow20;
ULONG64 Unknow21;
ULONG64 Unknow22;
ULONG64 PatchGuard\_PgContext;//+0x8B8  KeDebugCheck
ULONG64 PatchGuard\_CmpAppendDllSecitonSign\_Self;//+0x7B0 KeDebugCheck
}
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2018-9-8 14:00 被落笔飞花编辑，原因：
