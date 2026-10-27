
```asm
INITKDBG:000000014023930D                 mov     [r12+658h], rax
INITKDBG:0000000140239315                 mov     qword ptr [r12+660h], 101h
INITKDBG:0000000140239321                 mov     [r12+668h], rdx
INITKDBG:0000000140239329                 mov     [r12+648h], r13d
```

1804

```asm
//算法
loc_1402DC311:                          ; CODE XREF: INITKDBG:00000001402DC325j
INITKDBG:00000001402DC311                 xor     r8, [r9]
INITKDBG:00000001402DC314                 mov     ecx, r11d
INITKDBG:00000001402DC317                 rol     r8, cl
INITKDBG:00000001402DC31A                 add     r9, 8
INITKDBG:00000001402DC31E                 add     r10d, 0FFFFFFF8h
INITKDBG:00000001402DC322                 sub     rdx, rsi
INITKDBG:00000001402DC325                 jnz     short loc_1402DC311
INITKDBG:00000001402DC327
INITKDBG:00000001402DC327 loc_1402DC327:                          ; CODE XREF: INITKDBG:00000001402DC308j
INITKDBG:00000001402DC327                 test    r10d, r10d
INITKDBG:00000001402DC32A                 jz      short loc_1402DC342
INITKDBG:00000001402DC32C
INITKDBG:00000001402DC32C loc_1402DC32C:                          ; CODE XREF: INITKDBG:00000001402DC340j
INITKDBG:00000001402DC32C                 movzx   eax, byte ptr [r9]
INITKDBG:00000001402DC330                 mov     ecx, r11d
INITKDBG:00000001402DC333                 xor     r8, rax
INITKDBG:00000001402DC336                 add     r9, rsi
INITKDBG:00000001402DC339                 rol     r8, cl
INITKDBG:00000001402DC33C                 add     r10d, 0FFFFFFFFh
INITKDBG:00000001402DC340                 jnz     short loc_1402DC32C
INITKDBG:00000001402DC342
INITKDBG:00000001402DC342 loc_1402DC342:                          ; CODE XREF: INITKDBG:00000001402DC32Aj
INITKDBG:00000001402DC342                 mov     [r12+658h], r14
INITKDBG:00000001402DC34A                 lea     rcx, [rbp+0AB0h]
INITKDBG:00000001402DC351                 mov     edx, r13d
INITKDBG:00000001402DC354                 mov     r9, r15
INITKDBG:00000001402DC357                 mov     r11d, 0FFFFFFF8h
```

```asm
//
cmp     [r12+658h], r8  ; //658是否是pg的sign 这个位置 从pg 刚刚解密 还未执行时 就和现在是一样的
INITKDBG:00000001402DC38E                 jz      loc_1402DC414   ; //正常流程肯定会跳下去
INITKDBG:00000001402DC394                 mov     rax, [r12+588h] ; //TriageImagePageSize
INITKDBG:00000001402DC39C                 mov     ecx, [r12+684h] ; //这个00070989 不知道是什么
INITKDBG:00000001402DC3A4                 mov     [rax], r12
INITKDBG:00000001402DC3A7                 mov     [rax+10h], ecx
INITKDBG:00000001402DC3AA                 mov     eax, [r12+790h] ; //这里是0
INITKDBG:00000001402DC3B2                 mov     rcx, [r12+658h] ; //或许是sign
INITKDBG:00000001402DC3BA                 test    eax, eax
INITKDBG:00000001402DC3BC                 jnz     short loc_1402DC414
INITKDBG:00000001402DC3BE                 mov     rax, [r12+588h]
INITKDBG:00000001402DC3C6                 xor     rcx, r8
INITKDBG:00000001402DC3C9                 mov     [rax+18h], rcx  ; //不知所谓
INITKDBG:00000001402DC3CD                 mov     eax, [r12+790h]
INITKDBG:00000001402DC3D5                 test    eax, eax
INITKDBG:00000001402DC3D7                 jnz     short loc_1402DC414
INITKDBG:00000001402DC3D9                 mov     rax, 0A3A03F5891C8B4E8h
INITKDBG:00000001402DC3E3                 add     rax, r12
INITKDBG:00000001402DC3E6                 mov     [r12+798h], rax ; //蓝屏参数
INITKDBG:00000001402DC3EE                 xor     eax, eax
INITKDBG:00000001402DC3F0                 mov     [r12+7A0h], rax ; //清空低4byte 蓝屏代码
INITKDBG:00000001402DC3F8                 mov     qword ptr [r12+7A8h], 101h ; //蓝屏代码
INITKDBG:00000001402DC404                 mov     [r12+7B0h], r8  ; //效验和
INITKDBG:00000001402DC40C                 mov     [r12+790h], esi ; //esi为1
INITKDBG:00000001402DC414
```

第三处：

14393

懒得找了

1804

```asm
mov     [r12+0C4h], esi
INITKDBG:00000001402DC970                 cmp     [r12+8B8h], r8
INITKDBG:00000001402DC978                 jz      ExEcOtherSafeCall ; //正常直接 的话 必然相等然后走
INITKDBG:00000001402DC97E                 mov     rax, [r12+588h]
INITKDBG:00000001402DC986                 mov     ecx, [r12+684h]
INITKDBG:00000001402DC98E                 mov     [rax], r12
INITKDBG:00000001402DC991                 mov     [rax+10h], ecx
INITKDBG:00000001402DC994                 mov     eax, [r12+790h]
INITKDBG:00000001402DC99C                 mov     rdx, [r12+8B8h]
INITKDBG:00000001402DC9A4                 test    eax, eax
INITKDBG:00000001402DC9A6                 jnz     short ExEcOtherSafeCall ; //这里的eax 如果不是0啊 出去就蓝屏了 各种奇怪的错误
INITKDBG:00000001402DC9A8                 mov     rax, [r12+588h]
INITKDBG:00000001402DC9B0                 mov     rcx, r8
INITKDBG:00000001402DC9B3                 xor     rcx, rdx
INITKDBG:00000001402DC9B6                 mov     [rax+18h], rcx
INITKDBG:00000001402DC9BA                 mov     eax, [r12+790h]
INITKDBG:00000001402DC9C2                 test    eax, eax
INITKDBG:00000001402DC9C4                 jnz     short ExEcOtherSafeCall ; //这里的eax 如果不是0啊 出去就蓝屏了 各种奇怪的错误
INITKDBG:00000001402DC9C6                 mov     rax, 0A3A03F5891C8B4E8h
INITKDBG:00000001402DC9D0                 add     rax, r12
INITKDBG:00000001402DC9D3                 mov     [r12+798h], rax
INITKDBG:00000001402DC9DB                 xor     eax, eax
INITKDBG:00000001402DC9DD                 mov     [r12+7A0h], rax
INITKDBG:00000001402DC9E5                 mov     qword ptr [r12+7A8h], 109h
INITKDBG:00000001402DC9F1                 mov     [r12+7B0h], r8
INITKDBG:00000001402DC9F9                 mov     [r12+790h], r14d
INITKDBG:00000001402DCA01                 jmp     short loc_1402DCA05 ; //判断 这一处 如果要修改蓝屏代码 必须也给Nop点 下面的eax必须为0

ExEcOtherSafeCall:                      ; CODE XREF: INITKDBG:00000001402DC978j
INITKDBG:00000001402DCA03                                         ; INITKDBG:00000001402DC9A6j ...
INITKDBG:00000001402DCA03                 xor     eax, eax        ; //这里的eax 如果不是0啊 出去就蓝屏了 各种奇怪的错误
INITKDBG:00000001402DCA05
```

待更新研究。。

Blog：272K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6J5k6h3&6B7K9h3&6Y4i4K6u0W2N6$3q4F1k6H3`.`.

```c
struct _PatchGuard_Context_15063{
UCHAR CmpAppendDllSection[0xE8];//other Pg version dynamic
ULONG64 ExAcquireResourceSharedLite:
ULONG64 ExAcquireResourceExclusiveLite;
ULONG64 ExAllocatePoolWithTag;
ULONG64 ExFreePool;
ULONG64 ExMapHandleToPointer;
ULONG64 ExQueueWorkItem;
ULONG64 ExReleaseResourceLite;
ULONG64 ExUnlockHandleTableEntry;
ULONG64 ExAcquirePushLockExclusiveEx;
ULONG64 ExReleasePushLockExclusiveEx;
ULONG64 ExAcquirePushLockSharedEx;
ULONG64 ExReleasePushLockSharedEx;
ULONG64 KeAcquireInStackQueuedSpinLockAtDpcLevel;
ULONG64 ExAcquireSpinLockSharedAtDpcLevel;
ULONG64 KeBugCheckEx;
ULONG64 KeDelayExecutionThread;
ULONG64 KeEnterCriticalRegionThread;
ULONG64 KeLeaveCriticalRegion;
ULONG64 KeEnterGuardedRegion;
ULONG64 KeLeaveGuardedRegion;
ULONG64 KeReleaseInStackQueuedSpinLockFromDpcLevel;
ULONG64 ExReleaseSpinLockSharedFromDpcLevel;
ULONG64 KeRevertToUserGroupAffinityThread;
ULONG64 KeProcessorGroupAffinity;
ULONG64 KeInitializeEnumerationContext;
ULONG64 KeEnumerateNextProcessor;
ULONG64 KeCountSetBitsAffinityEx;
ULONG64 KeQueryAffinityProcess;
ULONG64 KeQueryAffinityThread;
ULONG64 KeSetSystemGroupAffinityThread;
ULONG64 KeSetCoalescableTimer;
ULONG64 ObfDereferenceObject;
ULONG64 ObReferenceObjectByName;
ULONG64 RtlImageDirectoryEntryToData;
ULONG64 RtlImageNtHeader;
ULONG64 RtlLookupFunctionTable;
ULONG64 RtlPcToFileHeader;
ULONG64 RtlSectionTableFromVirtualAddress;
ULONG64 DbgPrint;
ULONG64 MmAllocateIndependentPages;
ULONG64 MmFreeIndependentPages;
ULONG64 MmSetPageProtection;
ULONG64 Unknow1;
ULONG64 Unknow2;
ULONG64 Unknow3;
ULONG64 Unknow4;
ULONG64 RtlLookupFunctionEntry;
ULONG64 KeAcquireSpinLockRaiseToDpc;
ULONG64 KeReleaseSpinLock;
ULONG64 MmGetSessionById;
```
