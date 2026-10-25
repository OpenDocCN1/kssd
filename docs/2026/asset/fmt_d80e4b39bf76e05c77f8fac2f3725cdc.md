
# [原创]详解七句汇编获取`Kernel32`模块地址

在科锐学习壳的过程中，为了获取了`Kernel32`的基址的代码，在加密壳的时候直接使用。使用Onlydbg逆向截取到相应代码，当时的我无法理解这些代码，非常苦恼。在科锐老师的辛勤教导和自己的不断学习下，现在的我有所理解，写个解析记录分享一下，代码块如下：

```
GetKernel32Base proc
assume fs:nothing ;开始使用fs寄存器
mov eax, dword ptr fs : [30h] ;指向PEB结构
mov eax, dword ptr[eax + 0Ch] ;指向LDR Ptr32 \_PEB\_LDR\_DATA
mov eax, dword ptr[eax + 0Ch] ;指向InLoadOrderModuleList \_LIST\_ENTRY
mov eax, dword ptr[eax] ;移动\_LIST\_ENTRY
mov eax, dword ptr[eax] ;指向Kernel32
mov eax, dword ptr[eax + 18h] ;指向DllBase基址
ret
GetKernelBase endp
```

## 1、第一行代码：`assume fs:nothing`

`FS`寄存器指向当前活动线程的`TEB`结构（线程结构），缺省情况下`fs:error`表示`fs`未使用，若要引用`fs`寄存器，需`fs:nothing`后，才可使用。

## 2、第二行代码：`mov eax,dword ptr fs:[30h]`

我们使用Windbg来观察结构体，辅助我们的理解。输入`dt _teb`，`0x30`处是：`ProcessEnvironmentBlock`，类型是：`Ptr32 _PEB`，指向`PEB`。

![图片描述](upload/attach/202111/848111\_39VMJ2QJQBWRKBE.png)

附带`Teb`结构：

```
0:000> dt \_teb
ntdll!\_TEB
+0x000 NtTib : \_NT\_TIB
+0x01c EnvironmentPointer : Ptr32 Void
+0x020 ClientId : \_CLIENT\_ID
+0x028 ActiveRpcHandle : Ptr32 Void
+0x02c ThreadLocalStoragePointer : Ptr32 Void
+0x030 ProcessEnvironmentBlock : Ptr32 \_PEB
+0x034 LastErrorValue : Uint4B
+0x038 CountOfOwnedCriticalSections : Uint4B
+0x03c CsrClientThread : Ptr32 Void
+0x040 Win32ThreadInfo : Ptr32 Void
+0x044 User32Reserved : [26] Uint4B
+0x0ac UserReserved : [5] Uint4B
+0x0c0 WOW32Reserved : Ptr32 Void
+0x0c4 CurrentLocale : Uint4B
+0x0c8 FpSoftwareStatusRegister : Uint4B
+0x0cc ReservedForDebuggerInstrumentation : [16] Ptr32 Void
+0x10c SystemReserved1 : [26] Ptr32 Void
+0x174 PlaceholderCompatibilityMode : Char
+0x175 PlaceholderHydrationAlwaysExplicit : UChar
+0x176 PlaceholderReserved : [10] Char
+0x180 ProxiedProcessId : Uint4B
+0x184 \_ActivationStack : \_ACTIVATION\_CONTEXT\_STACK
+0x19c WorkingOnBehalfTicket : [8] UChar
+0x1a4 ExceptionCode : Int4B
+0x1a8 ActivationContextStackPointer : Ptr32 \_ACTIVATION\_CONTEXT\_STACK
+0x1ac InstrumentationCallbackSp : Uint4B
+0x1b0 InstrumentationCallbackPreviousPc : Uint4B
+0x1b4 InstrumentationCallbackPreviousSp : Uint4B
+0x1b8 InstrumentationCallbackDisabled : UChar
+0x1b9 SpareBytes : [23] UChar
+0x1d0 TxFsContext : Uint4B
+0x1d4 GdiTebBatch : \_GDI\_TEB\_BATCH
+0x6b4 RealClientId : \_CLIENT\_ID
+0x6bc GdiCachedProcessHandle : Ptr32 Void
+0x6c0 GdiClientPID : Uint4B
+0x6c4 GdiClientTID : Uint4B
+0x6c8 GdiThreadLocalInfo : Ptr32 Void
+0x6cc Win32ClientInfo : [62] Uint4B
+0x7c4 glDispatchTable : [233] Ptr32 Void
+0xb68 glReserved1 : [29] Uint4B
+0xbdc glReserved2 : Ptr32 Void
+0xbe0 glSectionInfo : Ptr32 Void
+0xbe4 glSection : Ptr32 Void
+0xbe8 glTable : Ptr32 Void
+0xbec glCurrentRC : Ptr32 Void
+0xbf0 glContext : Ptr32 Void
+0xbf4 LastStatusValue : Uint4B
+0xbf8 StaticUnicodeString : \_UNICODE\_STRING
+0xc00 StaticUnicodeBuffer : [261] Wchar
+0xe0c DeallocationStack : Ptr32 Void
+0xe10 TlsSlots : [64] Ptr32 Void
+0xf10 TlsLinks : \_LIST\_ENTRY
+0xf18 Vdm : Ptr32 Void
+0xf1c ReservedForNtRpc : Ptr32 Void
+0xf20 DbgSsReserved : [2] Ptr32 Void
+0xf28 HardErrorMode : Uint4B
+0xf2c Instrumentation : [9] Ptr32 Void
+0xf50 ActivityId : \_GUID
+0xf60 SubProcessTag : Ptr32 Void
+0xf64 PerflibData : Ptr32 Void
+0xf68 EtwTraceData : Ptr32 Void
+0xf6c WinSockData : Ptr32 Void
+0xf70 GdiBatchCount : Uint4B
+0xf74 CurrentIdealProcessor : \_PROCESSOR\_NUMBER
+0xf74 IdealProcessorValue : Uint4B
+0xf74 ReservedPad0 : UChar
+0xf75 ReservedPad1 : UChar
+0xf76 ReservedPad2 : UChar
+0xf77 IdealProcessor : UChar
+0xf78 GuaranteedStackBytes : Uint4B
+0xf7c ReservedForPerf : Ptr32 Void
+0xf80 ReservedForOle : Ptr32 Void
+0xf84 WaitingOnLoaderLock : Uint4B
+0xf88 SavedPriorityState : Ptr32 Void
+0xf8c ReservedForCodeCoverage : Uint4B
+0xf90 ThreadPoolData : Ptr32 Void
+0xf94 TlsExpansionSlots : Ptr32 Ptr32 Void
+0xf98 MuiGeneration : Uint4B
+0xf9c IsImpersonating : Uint4B
+0xfa0 NlsCache : Ptr32 Void
+0xfa4 pShimData : Ptr32 Void
+0xfa8 HeapData : Uint4B
+0xfac CurrentTransactionHandle : Ptr32 Void
+0xfb0 ActiveFrame : Ptr32 \_TEB\_ACTIVE\_FRAME
+0xfb4 FlsData : Ptr32 Void
+0xfb8 PreferredLanguages : Ptr32 Void
+0xfbc UserPrefLanguages : Ptr32 Void
+0xfc0 MergedPrefLanguages : Ptr32 Void
+0xfc4 MuiImpersonation : Uint4B
+0xfc8 CrossTebFlags : Uint2B
+0xfc8 SpareCrossTebBits : Pos 0, 16 Bits
+0xfca SameTebFlags : Uint2B
+0xfca SafeThunkCall : Pos 0, 1 Bit
+0xfca InDebugPrint : Pos 1, 1 Bit
+0xfca HasFiberData : Pos 2, 1 Bit
+0xfca SkipThreadAttach : Pos 3, 1 Bit
+0xfca WerInShipAssertCode : Pos 4, 1 Bit
+0xfca RanProcessInit : Pos 5, 1 Bit
+0xfca ClonedThread : Pos 6, 1 Bit
+0xfca SuppressDebugMsg : Pos 7, 1 Bit
+0xfca DisableUserStackWalk : Pos 8, 1 Bit
+0xfca RtlExceptionAttached : Pos 9, 1 Bit
+0xfca InitialThread : Pos 10, 1 Bit
+0xfca SessionAware : Pos 11, 1 Bit
+0xfca LoadOwner : Pos 12, 1 Bit
+0xfca LoaderWorker : Pos 13, 1 Bit
+0xfca SkipLoaderInit : Pos 14, 1 Bit
+0xfca SpareSameTebBits : Pos 15, 1 Bit
+0xfcc TxnScopeEnterCallback : Ptr32 Void
+0xfd0 TxnScopeExitCallback : Ptr32 Void
+0xfd4 TxnScopeContext : Ptr32 Void
+0xfd8 LockCount : Uint4B
+0xfdc WowTebOffset : Int4B
+0xfe0 ResourceRetValue : Ptr32 Void
+0xfe4 ReservedForWdf : Ptr32 Void
+0xfe8 ReservedForCrt : Uint8B
+0xff0 EffectiveContainerId : \_GUID
```

## 3、第三行代码：`mov eax,dword ptr [eax+0Ch]`

输入`dt _peb`，`0x0C`处是：`Ldr`，类型是：`Ptr32 _PEB_LDR_DATA`。

![图片描述](upload/attach/202111/848111\_TSEY3KWQ8RZM6XX.png)

附带`Peb`结构：
