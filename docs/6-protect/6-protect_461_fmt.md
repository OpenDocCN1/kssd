
# 厌烦了 HeapMagic

> 原文：[www.pediy.com/kssd/pediy09/pediy09-561.htm](https://www.pediy.com/kssd/pediy09/pediy09-561.htm)

> 最近摆弄 `OD`, 试了一下 `Shub-Nigurrath` 的 `xADT 1.3`, 没想到又被 `ProcessHeap` 逮到了, 实在有点火大! 想起 `Themida` 玩这个也很起劲, 干脆让被调试程序使用标准堆, 彻底了断;-)

> 下面的代码从 `W2K` 源码里找的, 我自己也稀里糊涂, 乱解释的地方请指教.

> 看看 `heap.c`:

```c
RtlCreateHeap (
    IN ULONG Flags,
    IN PVOID HeapBase OPTIONAL,
    IN SIZE_T ReserveSize OPTIONAL,
    IN SIZE_T CommitSize OPTIONAL,
    IN PVOID Lock OPTIONAL,
    IN PRTL_HEAP_PARAMETERS Parameters OPTIONAL
    )

......

PVOID NextHeapHeaderAddress;
PHEAP_UNCOMMMTTED_RANGE UnCommittedRange, *pp;
RTL_HEAP_PARAMETERS TempParameters;
ULONG NtGlobalFlag = RtlGetNtGlobalFlags();

/* RtlCreateHeap 在 ntoskrnl 和 ntdll 各有 1 个, 从下面的
   条件编译指令看, 似乎是同一个文件编译出来的. 内核的
   RtlGetNtGlobalFlags 直接取的全局变量, ntdll 内
   的实现则是从 PEB 取的
*/

#ifndef NTOS_KERNEL_RUNTIME

PPEB Peb;

#else // NTOS_KERNEL_RUNTIME

extern SIZE_T MmHeapSegmentReserve;
extern SIZE_T MmHeapSegmentCommit;
extern SIZE_T MmHeapDeCommitTotalFreeThreshold;
extern SIZE_T MmHeapDeCommitFreeBlockThreshold;

#endif // NTOS_KERNEL_RUNTIME

......

//
//  If the caller does not want to skip heap validiation checks then we
//  need to validate the rest of the flags but simply masking out only
//  those flags that want on a create heap call
//

if (!(Flags & HEAP_SKIP_VALIDATION_CHECKS)) {

    if (Flags & ~HEAP_CREATE_VALID_MASK) {

        HeapDebugPrint(( "Invalid flags (%08x) specified to RtlCreateHeap\n", Flags ));
        HeapDebugBreak( NULL );

        Flags &= HEAP_CREATE_VALID_MASK;
    }
}

/*
这个标记 HEAP_SKIP_VALIDATION_CHECKS=0x10000000, 多次使用, 貌似可以不用
调试堆, 不过是从调用参数来的, 不大好改, 而且 Heap 相关函数不少 */

//
//  If nt global flags tells us to always do tail or free checking
//  or to disable coalescing then force those bits set in the user
//  specified flags
//

if (NtGlobalFlag & FLG_HEAP_ENABLE_TAIL_CHECK) {

    Flags |= HEAP_TAIL_CHECKING_ENABLED;
}

if (NtGlobalFlag & FLG_HEAP_ENABLE_FREE_CHECK) {

    Flags |= HEAP_FREE_CHECKING_ENABLED;
}

if (NtGlobalFlag & FLG_HEAP_DISABLE_COALESCING) {

    Flags |= HEAP_DISABLE_COALESCE_ON_FREE;
}

//
//  In the non kernel case we also check if we should
//  validate parameters, validate all, or do stack backtraces
//

Peb = NtCurrentPeb();

if (NtGlobalFlag & FLG_HEAP_VALIDATE_PARAMETERS) {

    Flags |= HEAP_VALIDATE_PARAMETERS_ENABLED;
}

if (NtGlobalFlag & FLG_HEAP_VALIDATE_ALL) {

    Flags |= HEAP_VALIDATE_ALL_ENABLED;
}

if (NtGlobalFlag & FLG_USER_STACK_TRACE_DB) {

    Flags |= HEAP_CAPTURE_STACK_BACKTRACES;
}

/* 测试 NtGlobalFlag */

........

#ifndef NTOS_KERNEL_RUNTIME

//
//  In the non kernel case check if we are creating a debug heap
//  the test checks that skip validation checks is false.
//

if (DEBUG_HEAP( Flags )) {

    return RtlDebugCreateHeap( Flags,
                              HeapBase,
                              ReserveSize,
                              CommitSize,
                              Lock,
                              Parameters );
}

#endif // NTOS_KERNEL_RUNTIME

/* 这里是我们感兴趣的了, DEBUG_HEAP(Flags)为 TRUE 则
   创建调试堆(注意内核模式永远不会用调试堆), 这个宏
   定义在 heappriv.h 内:

   #define HEAP_DEBUG_FLAGS   (HEAP_VALIDATE_PARAMETERS_ENABLED | \
                    HEAP_VALIDATE_ALL_ENABLED        | \
                    HEAP_CAPTURE_STACK_BACKTRACES    | \
                    HEAP_CREATE_ENABLE_TRACING       | \
                    HEAP_FLAG_PAGE_ALLOCS)
   #define DEBUG_HEAP(F)      ((F & HEAP_DEBUG_FLAGS) && !(F & HEAP_SKIP_VALIDATION_CHECKS))

   另外几个宏在 heap.h 内:

   #define HEAP_SIGNATURE             (ULONG)0xEEFFEEFF
   #define HEAP_LOCK_USER_ALLOCATED     (ULONG)0x80000000
   #define HEAP_VALIDATE_PARAMETERS_ENABLED    (ULONG)0x40000000
   #define HEAP_VALIDATE_ALL_ENABLED         (ULONG)0x20000000
   #define HEAP_SKIP_VALIDATION_CHECKS         (ULONG)0x10000000
   #define HEAP_CAPTURE_STACK_BACKTRACES       (ULONG)0x08000000

   #define CHECK_HEAP_TAIL_SIZE HEAP_GRANULARITY
   #define CHECK_HEAP_TAIL_FILL 0xAB
   #define FREE_HEAP_FILL 0xFEEEFEEE   <------- MAGIC ;-)
   #define ALLOC_HEAP_FILL 0xBAADF00D  <------- MAGIC
  */

看起来只要 PEB 内的 NtGlobalFlag 为 0, 就可以避开调试堆, 下面是 XP SP2 下的代码:

```asm
PAGE:004AC2B0 __stdcall MmCreatePeb(x, x, x) proc near
......
PAGE:004AC38A             mov     ecx, [ebp+var_1C]
PAGE:004AC38D             mov     [ecx+2], al
PAGE:004AC390             mov     eax, [ebp+var_1C]
PAGE:004AC393             mov     ecx, dword ptr _NtGlobalFlag <- 这里
PAGE:004AC399             mov     [eax+68h], ecx
PAGE:004AC39C             mov     eax, _MmCriticalSectionTimeout
PAGE:004AC3A1             mov     ecx, [ebp+var_1C]
PAGE:004AC3A4             mov     [ecx+70h], eax
```

创建 PEB 时用的是内核全局变量, 用 `WinDbg` 可以看到, 全局 `NtGlobalFlag` 为 0, 但被调试时 PEB 内的 `NtGlobalFlag` 却是 70h(如果不做手脚).

下面是 `ntdll` 内的代码:

```c
VOID
LdrpInitialize (
  IN PCONTEXT Context,
  IN PVOID SystemArgument1,
  IN PVOID SystemArgument2
)

Routine Description:

  This function is called as a User-Mode APC routine as the first
  user-mode code executed by a new thread. It's function is to initialize
  loader context, perform module initialization callouts...

......

//#if DBG
    if (TRUE)
//#else
//        if (Peb->BeingDebugged || Peb->ReadImageFileExecOptions)
//#endif
    {
        PWSTR pw;

        pw = (PWSTR)Peb->ProcessParameters->ImagePathName.Buffer;
        if (!(Peb->ProcessParameters->Flags & RTL_USER_PROC_PARAMS_NORMALIZED)) {
            pw = (PWSTR)((PCHAR)pw + (ULONG_PTR)(Peb->ProcessParameters));
        }
        UnicodeImageName.Buffer = pw;
        UnicodeImageName.Length = Peb->ProcessParameters->ImagePathName.Length;
        UnicodeImageName.MaximumLength = UnicodeImageName.Length;

        //
        //  Hack for NT4 SP4.  So we don't overload another GlobalFlag
        //  bit that we have to be "compatible" with for NT5, look for
        //  another value named "DisableHeapLookaside".
        //

        LdrQueryImageFileExecutionOptions( &UnicodeImageName,
                                           L"DisableHeapLookaside",
                                           REG_DWORD,
```



# 文档结构与代码逻辑解析

以下文档详细阐述了 Windows 系统中 `PEB`（Process Environment Block）的相关标志位操作、汇编代码段分析，以及基于 `OD` 的调试堆补丁（Patch）实现方案。

## 核心逻辑与代码片段

首先，获取并处理 `RtlpDisableHeapLookaside` 相关数据：

```
& RtlpDisableHeapLookaside,
sizeof( RtlpDisableHeapLookaside ),
NULL
);

st = LdrQueryImageFileExecutionOptions( &UnicodeImageName,
            L"GlobalFlag",
            REG_DWORD,
            &Peb->NtGlobalFlag,
            sizeof( Peb->NtGlobalFlag ),
            NULL
);

if (!NT_SUCCESS( st )) {
    if (Peb->BeingDebugged) {
        /**** PEB.NtGlobalFlag 被改写了 ****/
        Peb->NtGlobalFlag |= FLG_HEAP_ENABLE_FREE_CHECK |
                FLG_HEAP_ENABLE_TAIL_CHECK |
                FLG_HEAP_VALIDATE_PARAMETERS;
        }
    }

#if defined(_WIN64)
    NtHeader = RtlImageNtHeader(Peb->ImageBaseAddress);
    if (NtHeader && NtHeader->OptionalHeader.Magic == IMAGE_NT_OPTIONAL_HDR32_MAGIC) {
        UseWOW64 = TRUE;
    }
#endif
}

if ( Peb->NtGlobalFlag & FLG_HEAP_PAGE_ALLOCS ) {

    //
    // We will enable page heap (RtlpDebugPageHeap) only after
    // all other initializations for page heap are finished.
    //

    //
    // If page heap is enabled we need to disable any flag that
    // might force creation of debug heaps for normal NT heaps.
    // This is due to a dependency between page heap and NT heap
    // where the page heap within PageHeapCreate tries to create
    // a normal NT heap to accomodate some of the allocations.
    // If we do not disable these flags we will get an infinite
    // recursion between RtlpDebugPageHeapCreate and RtlCreateHeap.
    //

    /**** PEB.NtGlobalFlag 被改写了 ****/

    Peb->NtGlobalFlag &= ~( FLG_HEAP_ENABLE_TAGGING     |
        FLG_HEAP_ENABLE_TAG_BY_DLL   |
        FLG_HEAP_ENABLE_TAIL_CHECK   |
        FLG_HEAP_ENABLE_FREE_CHECK   |
        FLG_HEAP_VALIDATE_PARAMETERS |
        FLG_HEAP_VALIDATE_ALL        |
        FLG_USER_STACK_TRACE_DB      );
}
```

### 逻辑说明
大致逻辑如下：如果 `Peb->BeingDebugged` 为 `TRUE`，则启用调试堆。关于 `BeingDebugged` 在哪里被置 `1`（或者能否不让其被置 `1`，怀疑会影响调试）未做进一步处理，但在 `SoftICE` 中测试表明，在此处进行 Patch 即可生效。

## 汇编代码参考（WinXP SP2）

在 `WinXP SP1`/`SP2` 中，相关逻辑位于 `LdrpInitializeExecutionOptions` 内部。以下是 `SP2` 版本的汇编代码片段：

```asm
.text:7C942D56 __stdcall LdrpInitializeExecutionOptions(x, x) proc near
.text:7C942D56
.text:7C942D56                 mov     edi, edi
.text:7C942D58                 push    ebp
.text:7C942D59                 mov     ebp, esp
.text:7C942D5B                 sub     esp, 40h
.text:7C942D5E                 mov     eax, ___security_cookie
.text:7C942D63                 push    ebx
.text:7C942D64                 push    esi
.text:7C942D65                 mov     esi, [ebp+arg_4]
.text:7C942D68                 push    edi
.text:7C942D69                 mov     edi, [ebp+arg_0]
.text:7C942D6C                 mov     [ebp+var_4], eax
.text:7C942D6F                 lea     eax, [ebp+var_2C]
.text:7C942D72                 push    eax
.text:7C942D73                 xor     ebx, ebx
.text:7C942D75                 push    edi
.text:7C942D76                 mov     [ebp+var_38], edi
.text:7C942D79                 mov     [ebp+var_25], bl
.text:7C942D7C                 call    LdrpOpenImageFileOptionsKey(x,x)
.text:7C942D81                 test    eax, eax
.text:7C942D83                 jge     sub_7C95DE3C
.text:7C942D89                 test    dword ptr [esi+68h], 2000100h
.text:7C942D90                 jnz     sub_7C95DF14
.text:7C942D96                 cmp     [esi+2], bl   <----- BeginDebugged
.text:7C942D99                 jnz     loc_7C95DF23  <----- NOP 这 6 字节
.text:7C942D9F
.text:7C942D9F loc_7C942D9F:
.text:7C942D9F                 mov     ecx, [ebp+var_4]
.text:7C942DA2                 mov     al, [ebp+var_25]
.text:7C942DA5                 pop     edi
.text:7C942DA6                 pop     esi
.text:7C942DA7                 pop     ebx
.text:7C942DA8                 call    __security_check_cookie(x)
.text:7C942DAD                 leave
.text:7C942DAE                 retn    8
```

### Patch 位置说明
`Patch` 位置在 `SP1` 和 `SP2` 下略有不同。似乎没有合适的函数作为搜索跳板，通常直接在 `text` 区段寻找。只有 `1` 处满足类似以下代码序列：

```asm
.text:7C942D89                 test    dword ptr [esi+68h], 2000100h
.text:7C942D90                 jnz     sub_7C95DF14
.text:7C942D96                 cmp     [esi+2], bl
.text:7C942D99                 jnz     loc_7C95DF23
```

## C++ 实现示例（Hook OD）

这里展示了 Hook `OD` 自身的 `WaitForDebugEvent` 函数以实现上述 Patch 的 C++ 代码示例：

```cpp
BOOL __stdcall COllyDbg::NewWaitForDebugEvent(
  LPDEBUG_EVENT lpDebugEvent,
  DWORD dwMilliseconds
  )
{
  DWORD   dwOldProt;
  DWORD   dwNewProt = PAGE_EXECUTE_READWRITE;
  BYTE    nops[6] = {0x90,0x90,0x90,0x90,0x90,0x90};
  // ...
  
  switch(lpDebugEvent->dwDebugEventCode)
  {
  case CREATE_PROCESS_DEBUG_EVENT:
    
    if(m_PatchDebugHeap)
    {
        VirtualProtectEx(lpDebugEvent->u.CreateProcessInfo.hProcess,
                    (PVOID)m_PatchDebugHeap,
                6,
                dwNewProt,
                &dwOldProt)
                ;
        
        WriteProcessMemory(lpDebugEvent->u.CreateProcessInfo.hProcess,
                    (PVOID)m_PatchDebugHeap,
                  nops,
                  6,
                  NULL 
                  );
        
        VirtualProtectEx(lpDebugEvent->u.CreateProcessInfo.hProcess,
                  (PVOID)m_PatchDebugHeap,
                6,
                dwOldProt,
                &dwNewProt
                );
    }
    break;
    
    // ...
  }
}
```

`m_PatchDebugHeap` 即为搜索出的 Patch 地址。

`4dd466d0e1e0cc123afbe504da2462ca`
