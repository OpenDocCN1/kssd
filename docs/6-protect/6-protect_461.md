

# 厌烦了 HeapMagic

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-561.htm`](https://www.pediy.com/kssd/pediy09/pediy09-561.htm)

> 最近摆弄 OD,试了一下 Shub-Nigurrath 的 xADT 1.3,没想到又被  
> ProcessHeap 逮到了,实在有点火大! 想起 Themida 玩这个也很起  
> 劲,干脆让被调试程序使用标准堆, 彻底了断;-)  
>   
>   
> 下面的代码从 W2K 源码里找的,我自己也稀里糊涂,乱解释的地方  
> 请指教.  
>   
>   
> 看看 heap.c:  
>   
> RtlCreateHeap (  
>     IN ULONG Flags,  
>     IN PVOID HeapBase OPTIONAL,  
>     IN SIZE\_T ReserveSize OPTIONAL,  
>     IN SIZE\_T CommitSize OPTIONAL,  
>     IN PVOID Lock OPTIONAL,  
>     IN PRTL\_HEAP\_PARAMETERS Parameters OPTIONAL  
>     )  
>   
>     ......  
>   
>     PVOID NextHeapHeaderAddress;  
>     PHEAP\_UNCOMMMTTED\_RANGE UnCommittedRange, \*pp;  
>     RTL\_HEAP\_PARAMETERS TempParameters;  
>     ULONG NtGlobalFlag = RtlGetNtGlobalFlags();  
>   
>     /\* RtlCreateHeap 在 ntoskrnl 和 ntdll 各有 1 个, 从下面的  
>        条件编译指令看,似乎是同一个文件编译出来的. 内核的  
>        RtlGetNtGlobalFlags 直接取的全局变量, ntdll 内  
>        的实现则是从 PEB 取的  
>     \*/  
>       
> #ifndef NTOS\_KERNEL\_RUNTIME  
>   
>     PPEB Peb;  
>   
> #else // NTOS\_KERNEL\_RUNTIME  
>   
>     extern SIZE\_T MmHeapSegmentReserve;  
>     extern SIZE\_T MmHeapSegmentCommit;  
>     extern SIZE\_T MmHeapDeCommitTotalFreeThreshold;  
>     extern SIZE\_T MmHeapDeCommitFreeBlockThreshold;  
>   
> #endif // NTOS\_KERNEL\_RUNTIME  
>   
>     ......  
>   
>     //  
>     //  If the caller does not want to skip heap validiation checks then we  
>     //  need to validate the rest of the flags but simply masking out only  
>     //  those flags that want on a create heap call  
>     //  
>   
>     if (!(Flags & HEAP\_SKIP\_VALIDATION\_CHECKS)) {  
>   
>         if (Flags & ~HEAP\_CREATE\_VALID\_MASK) {  
>   
>             HeapDebugPrint(( "Invalid flags (%08x) specified to RtlCreateHeap\\n", Flags ));  
>             HeapDebugBreak( NULL );  
>   
>             Flags &= HEAP\_CREATE\_VALID\_MASK;  
>         }  
>     }  
>   
>     /\*  
>     这个标记 HEAP\_SKIP\_VALIDATION\_CHECKS=0x10000000,多次使用,貌似可以不用  
>     调试堆,不过是从调用参数来的,不大好改,而且 Heap 相关函数不少 \*/  
>   
>     //  
>     //  If nt global flags tells us to always do tail or free checking  
>     //  or to disable coalescing then force those bits set in the user  
>     //  specified flags  
>     //  
>   
>     if (NtGlobalFlag & FLG\_HEAP\_ENABLE\_TAIL\_CHECK) {  
>   
>         Flags |= HEAP\_TAIL\_CHECKING\_ENABLED;  
>     }  
>   
>     if (NtGlobalFlag & FLG\_HEAP\_ENABLE\_FREE\_CHECK) {  
>   
>         Flags |= HEAP\_FREE\_CHECKING\_ENABLED;  
>     }  
>   
>     if (NtGlobalFlag & FLG\_HEAP\_DISABLE\_COALESCING) {  
>   
>         Flags |= HEAP\_DISABLE\_COALESCE\_ON\_FREE;  
>     }  
>   
>     //  
>     //  In the non kernel case we also check if we should  
>     //  validate parameters, validate all, or do stack backtraces  
>     //  
>   
>     Peb = NtCurrentPeb();  
>   
>     if (NtGlobalFlag & FLG\_HEAP\_VALIDATE\_PARAMETERS) {  
>   
>         Flags |= HEAP\_VALIDATE\_PARAMETERS\_ENABLED;  
>     }  
>   
>     if (NtGlobalFlag & FLG\_HEAP\_VALIDATE\_ALL) {  
>   
>         Flags |= HEAP\_VALIDATE\_ALL\_ENABLED;  
>     }  
>   
>     if (NtGlobalFlag & FLG\_USER\_STACK\_TRACE\_DB) {  
>   
>         Flags |= HEAP\_CAPTURE\_STACK\_BACKTRACES;  
>     }  
>   
>     /\* 测试 NtGlobalFlag \*/  
>   
>     ........  
>   
>     #ifndef NTOS\_KERNEL\_RUNTIME  
>   
>     //  
>     //  In the non kernel case check if we are creating a debug heap  
>     //  the test checks that skip validation checks is false.  
>     //  
>   
>     if (DEBUG\_HEAP( Flags )) {  
>   
>         return RtlDebugCreateHeap( Flags,  
>                                    HeapBase,  
>                                    ReserveSize,  
>                                    CommitSize,  
>                                    Lock,  
>                                    Parameters );  
>     }  
>   
>     #endif // NTOS\_KERNEL\_RUNTIME  
>   
>     /\* 这里是我们感兴趣的了,DEBUG\_HEAP(Flags)为 TRUE 则  
>        创建调试堆(注意内核模式永远不会用调试堆),这个宏  
>        定义在 heappriv.h 内:  
>   
>        #define HEAP\_DEBUG\_FLAGS   (HEAP\_VALIDATE\_PARAMETERS\_ENABLED | \\  
>                             HEAP\_VALIDATE\_ALL\_ENABLED        | \\  
>                             HEAP\_CAPTURE\_STACK\_BACKTRACES    | \\  
>                             HEAP\_CREATE\_ENABLE\_TRACING       | \\  
>                             HEAP\_FLAG\_PAGE\_ALLOCS)  
>        #define DEBUG\_HEAP(F)     ((F & HEAP\_DEBUG\_FLAGS) && !(F & HEAP\_SKIP\_VALIDATION\_CHECKS))  
>   
>          
>        另外几个宏在 heap.h 内:  
>   
>        #define HEAP\_SIGNATURE                      (ULONG)0xEEFFEEFF  
>        #define HEAP\_LOCK\_USER\_ALLOCATED            (ULONG)0x80000000  
>        #define HEAP\_VALIDATE\_PARAMETERS\_ENABLED    (ULONG)0x40000000  
>        #define HEAP\_VALIDATE\_ALL\_ENABLED           (ULONG)0x20000000  
>        #define HEAP\_SKIP\_VALIDATION\_CHECKS         (ULONG)0x10000000  
>        #define HEAP\_CAPTURE\_STACK\_BACKTRACES       (ULONG)0x08000000  
>   
>        #define CHECK\_HEAP\_TAIL\_SIZE HEAP\_GRANULARITY  
>        #define CHECK\_HEAP\_TAIL\_FILL 0xAB  
>        #define FREE\_HEAP\_FILL 0xFEEEFEEE   <------- MAGIC ;-)  
>        #define ALLOC\_HEAP\_FILL 0xBAADF00D  <------- MAGIC  
>    \*/  
>   
>   
>     看起来只要 PEB 内的 NtGlobalFlag 为 0,就可以避开调试堆,下面是 XP SP2 下的代码:  
>   
>     PAGE:004AC2B0 \_\_stdcall MmCreatePeb(x, x, x) proc near  
>     ......  
>     PAGE:004AC38A                 mov     ecx, \[ebp+var\_1C\]  
>     PAGE:004AC38D                 mov     \[ecx+2\], al  
>     PAGE:004AC390                 mov     eax, \[ebp+var\_1C\]  
>     PAGE:004AC393                 mov     ecx, dword ptr \_NtGlobalFlag <- 这里  
>     PAGE:004AC399                 mov     \[eax+68h\], ecx  
>     PAGE:004AC39C                 mov     eax, \_MmCriticalSectionTimeout  
>     PAGE:004AC3A1                 mov     ecx, \[ebp+var\_1C\]  
>     PAGE:004AC3A4                 mov     \[ecx+70h\], eax  
>   
>     创建 PEB 时用的是内核全局变量,用 WinDbg 可以看到,全局 NtGlobalFlag 为 0,  
>     但被调试时 PEB 内的 NtGlobalFlag 却是 70h(如果不做手脚).  
>   
>     下面是 ntdll 内的代码:  
>   
>     VOID  
>     LdrpInitialize (  
>        IN PCONTEXT Context,  
>        IN PVOID SystemArgument1,  
>        IN PVOID SystemArgument2  
>     )  
>   
>     Routine Description:  
>   
>        This function is called as a User-Mode APC routine as the first  
>        user-mode code executed by a new thread. It's function is to initialize  
>        loader context, perform module initialization callouts...  
>   
>     ......  
>     
>   
> //#if DBG  
>         if (TRUE)  
> //#else  
> //        if (Peb->BeingDebugged || Peb->ReadImageFileExecOptions)  
> //#endif  
>         {  
>             PWSTR pw;  
>   
>             pw = (PWSTR)Peb->ProcessParameters->ImagePathName.Buffer;  
>             if (!(Peb->ProcessParameters->Flags & RTL\_USER\_PROC\_PARAMS\_NORMALIZED)) {  
>                 pw = (PWSTR)((PCHAR)pw + (ULONG\_PTR)(Peb->ProcessParameters));  
>                 }  
>             UnicodeImageName.Buffer = pw;  
>             UnicodeImageName.Length = Peb->ProcessParameters->ImagePathName.Length;  
>             UnicodeImageName.MaximumLength = UnicodeImageName.Length;  
>   
>             //  
>             //  Hack for NT4 SP4.  So we don't overload another GlobalFlag  
>             //  bit that we have to be "compatible" with for NT5, look for  
>             //  another value named "DisableHeapLookaside".  
>             //  
>   
>             LdrQueryImageFileExecutionOptions( &UnicodeImageName,  
>                                                L"DisableHeapLookaside",  
>                                                REG\_DWORD,  
>                                                &RtlpDisableHeapLookaside,  
>                                                sizeof( RtlpDisableHeapLookaside ),  
>                                                NULL  
>                                              );  
>   
>             st = LdrQueryImageFileExecutionOptions( &UnicodeImageName,  
>                                                     L"GlobalFlag",  
>                                                     REG\_DWORD,  
>                                                     &Peb->NtGlobalFlag,  
>                                                     sizeof( Peb->NtGlobalFlag ),  
>                                                     NULL  
>                                                   );  
>             if (!NT\_SUCCESS( st )) {  
>   
>                 if (Peb->BeingDebugged) {  
>   
>         /\*\*\*  PEB.NtGlobalFlag 被改写了 \*\*\*/  
>   
>                     Peb->NtGlobalFlag |= FLG\_HEAP\_ENABLE\_FREE\_CHECK |  
>                                          FLG\_HEAP\_ENABLE\_TAIL\_CHECK |  
>                                          FLG\_HEAP\_VALIDATE\_PARAMETERS;  
>                     }  
>                 }  
>   
> #if defined(\_WIN64)  
>             NtHeader = RtlImageNtHeader(Peb->ImageBaseAddress);  
>             if (NtHeader && NtHeader->OptionalHeader.Magic == IMAGE\_NT\_OPTIONAL\_HDR32\_MAGIC) {  
>                 UseWOW64 = TRUE;  
>             }  
> #endif  
>         }  
>   
>         if ( Peb->NtGlobalFlag & FLG\_HEAP\_PAGE\_ALLOCS ) {  
>   
>             //  
>             // We will enable page heap (RtlpDebugPageHeap) only after  
>             // all other initializations for page heap are finished.  
>             //  
>   
>             //  
>             // If page heap is enabled we need to disable any flag that  
>             // might force creation of debug heaps for normal NT heaps.  
>             // This is due to a dependency between page heap and NT heap  
>             // where the page heap within PageHeapCreate tries to create  
>             // a normal NT heap to accomodate some of the allocations.  
>             // If we do not disable these flags we will get an infinite  
>             // recursion between RtlpDebugPageHeapCreate and RtlCreateHeap.  
>             //  
>   
>       /\*\*\* PEB.NtGlobalFlag 被改写了 \*\*\*/  
>   
>             Peb->NtGlobalFlag &= ~( FLG\_HEAP\_ENABLE\_TAGGING      |  
>                 FLG\_HEAP\_ENABLE\_TAG\_BY\_DLL   |  
>                 FLG\_HEAP\_ENABLE\_TAIL\_CHECK   |  
>                 FLG\_HEAP\_ENABLE\_FREE\_CHECK   |  
>                 FLG\_HEAP\_VALIDATE\_PARAMETERS |  
>                 FLG\_HEAP\_VALIDATE\_ALL        |  
>                 FLG\_USER\_STACK\_TRACE\_DB      );  
>   
>     
>     
> 大致就是这么个意思, 如果 PEB.BeginDebugged 为 TRUE,就要使用调试堆. 至于  
> BeingDebugged 在哪里被置 1(或者能否不让其被置 1,怀疑会影响调试)就没有管  
> 了,在 SoftICE 里试了一下,在这里 patch 就行.  
>   
>   
> WinXP SP1/SP2,都在 LdrpInitializeExecutionOptions 内,下面是 SP2 的:  
>   
>   
> .text:7C942D56 \_\_stdcall LdrpInitializeExecutionOptions(x, x) proc near  
> .text:7C942D56  
> .text:7C942D56                 mov     edi, edi  
> .text:7C942D58                 push    ebp  
> .text:7C942D59                 mov     ebp, esp  
> .text:7C942D5B                 sub     esp, 40h  
> .text:7C942D5E                 mov     eax, \_\_\_security\_cookie  
> .text:7C942D63                 push    ebx  
> .text:7C942D64                 push    esi  
> .text:7C942D65                 mov     esi, \[ebp+arg\_4\]  
> .text:7C942D68                 push    edi  
> .text:7C942D69                 mov     edi, \[ebp+arg\_0\]  
> .text:7C942D6C                 mov     \[ebp+var\_4\], eax  
> .text:7C942D6F                 lea     eax, \[ebp+var\_2C\]  
> .text:7C942D72                 push    eax  
> .text:7C942D73                 xor     ebx, ebx  
> .text:7C942D75                 push    edi  
> .text:7C942D76                 mov     \[ebp+var\_38\], edi  
> .text:7C942D79                 mov     \[ebp+var\_25\], bl  
> .text:7C942D7C                 call    LdrpOpenImageFileOptionsKey(x,x)  
> .text:7C942D81                 test    eax, eax  
> .text:7C942D83                 jge     sub\_7C95DE3C  
> .text:7C942D89                 test    dword ptr \[esi+68h\], 2000100h  
> .text:7C942D90                 jnz     sub\_7C95DF14  
> .text:7C942D96                 cmp     \[esi+2\], bl   <----- BeginDebugged  
> .text:7C942D99                 jnz     loc\_7C95DF23  <----- NOP 这 6 字节  
> .text:7C942D9F  
> .text:7C942D9F loc\_7C942D9F:                             
> .text:7C942D9F                 mov     ecx, \[ebp+var\_4\]  
> .text:7C942DA2                 mov     al, \[ebp+var\_25\]  
> .text:7C942DA5                 pop     edi  
> .text:7C942DA6                 pop     esi  
> .text:7C942DA7                 pop     ebx  
> .text:7C942DA8                 call    \_\_security\_check\_cookie(x)  
> .text:7C942DAD                 leave  
> .text:7C942DAE                 retn    8  
>   
> Patch 位置在 SP1 和 SP2 下不大相同,似乎没有合适的函数作搜索跳板,我是直接在  
> text 区段找的,只有 1 处满足类似下面的代码序列:  
>   
> .text:7C942D89                 test    dword ptr \[esi+68h\], 2000100h  
> .text:7C942D90                 jnz     sub\_7C95DF14  
> .text:7C942D96                 cmp     \[esi+2\], bl  
> .text:7C942D99                 jnz     loc\_7C95DF23  
>   
>       
> 我这里是 Hook 了 OD 自己的 WaitForDebugEvent 的, 这样 patch 就行:  
>   
> BOOL \_\_stdcall COllyDbg::NewWaitForDebugEvent(  
>   LPDEBUG\_EVENT lpDebugEvent,  
>   DWORD dwMilliseconds   
>   )  
> {  
>   DWORD   dwOldProt;  
>   DWORD   dwNewProt = PAGE\_EXECUTE\_READWRITE;  
>   BYTE    nops\[6\] = {0x90,0x90,0x90,0x90,0x90,0x90};  
>   ........  
>   
>   switch(lpDebugEvent->dwDebugEventCode)   
>   {  
>   case CREATE\_PROCESS\_DEBUG\_EVENT:  
>       
>     if(m\_PatchDebugHeap)  
>     {  
>          VirtualProtectEx(lpDebugEvent->u.CreateProcessInfo.hProcess,  
>                     (PVOID)m\_PatchDebugHeap,  
>               6,  
>               dwNewProt,  
>               &dwOldProt)   
>               );  
>         
>          WriteProcessMemory(lpDebugEvent->u.CreateProcessInfo.hProcess,  
>                   (PVOID)m\_PatchDebugHeap,  
>                nops,  
>                6,  
>                NULL   
>                );  
>   
>          VirtualProtectEx(lpDebugEvent->u.CreateProcessInfo.hProcess,  
>                  (PVOID)m\_PatchDebugHeap,  
>               6,  
>               dwOldProt,  
>               &dwNewProt  
>               );  
>       }  
>     break;  
>   
>                ......  
>   
>   
> m\_PatchDebugHeap 就是搜索出的 patch 地址.终于清静了;-) 

 4dd466d0e1e0cc123afbe504da2462ca