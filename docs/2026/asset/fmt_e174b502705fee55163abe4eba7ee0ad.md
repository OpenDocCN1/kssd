
`kd> dt _PEB`

```
ntdll!_PEB

   +0x000 InheritedAddressSpace : UChar

   +0x001 ReadImageFileExecOptions : UChar

   +0x002 BeingDebugged    : UChar

   +0x003 SpareBool        : UChar

   +0x004 Mutant           : Ptr32 Void

   +0x008 ImageBaseAddress : Ptr32 Void

   +0x00c Ldr              : Ptr32 _PEB_LDR_DATA

   +0x010 ProcessParameters : Ptr32 _RTL_USER_PROCESS_PARAMETERS

   +0x014 SubSystemData    : Ptr32 Void

   +0x018 ProcessHeap      : Ptr32 Void

   +0x01c FastPebLock      : Ptr32 _RTL_CRITICAL_SECTION

   +0x020 FastPebLockRoutine : Ptr32 Void

   +0x024 FastPebUnlockRoutine : Ptr32 Void

   +0x028 EnvironmentUpdateCount : Uint4B

   +0x02c KernelCallbackTable : Ptr32 Void

   +0x030 SystemReserved   : [1] Uint4B

   +0x034 AtlThunkSListPtr32 : Uint4B

   +0x038 FreeList         : Ptr32 _PEB_FREE_BLOCK

   +0x03c TlsExpansionCounter : Uint4B

   +0x040 TlsBitmap        : Ptr32 Void

   +0x044 TlsBitmapBits    : [2] Uint4B

   +0x04c ReadOnlySharedMemoryBase : Ptr32 Void

   +0x050 ReadOnlySharedMemoryHeap : Ptr32 Void

   +0x054 ReadOnlyStaticServerData : Ptr32 Ptr32 Void

   +0x058 AnsiCodePageData : Ptr32 Void

   +0x05c OemCodePageData  : Ptr32 Void

   +0x060 UnicodeCaseTableData : Ptr32 Void

   +0x064 NumberOfProcessors : Uint4B

   +0x068 NtGlobalFlag     : Uint4B

   +0x070 CriticalSectionTimeout : _LARGE_INTEGER

   +0x078 HeapSegmentReserve : Uint4B

   +0x07c HeapSegmentCommit : Uint4B

   +0x080 HeapDeCommitTotalFreeThreshold : Uint4B

   +0x084 HeapDeCommitFreeBlockThreshold : Uint4B

   +0x088 NumberOfHeaps    : Uint4B

   +0x08c MaximumNumberOfHeaps : Uint4B

   +0x090 ProcessHeaps     : Ptr32 Ptr32 Void

   +0x094 GdiSharedHandleTable : Ptr32 Void

   +0x098 ProcessStarterHelper : Ptr32 Void

   +0x09c GdiDCAttributeList : Uint4B

   +0x0a0 LoaderLock       : Ptr32 Void

   +0x0a4 OSMajorVersion   : Uint4B

   +0x0a8 OSMinorVersion   : Uint4B

   +0x0ac OSBuildNumber    : Uint2B

   +0x0ae OSCSDVersion     : Uint2B

   +0x0b0 OSPlatformId     : Uint4B

   +0x0b4 ImageSubsystem   : Uint4B

   +0x0b8 ImageSubsystemMajorVersion : Uint4B

   +0x0bc ImageSubsystemMinorVersion : Uint4B

   +0x0c0 ImageProcessAffinityMask : Uint4B

   +0x0c4 GdiHandleBuffer  : [34] Uint4B

   +0x14c PostProcessInitRoutine : Ptr32     void 

   +0x150 TlsExpansionBitmap : Ptr32 Void

   +0x154 TlsExpansionBitmapBits : [32] Uint4B

   +0x1d4 SessionId        : Uint4B

   +0x1d8 AppCompatFlags   : _ULARGE_INTEGER

   +0x1e0 AppCompatFlagsUser : _ULARGE_INTEGER

   +0x1e8 pShimData        : Ptr32 Void

   +0x1ec AppCompatInfo    : Ptr32 Void

   +0x1f0 CSDVersion       : _UNICODE_STRING

   +0x1f8 ActivationContextData : Ptr32 Void

   +0x1fc ProcessAssemblyStorageMap : Ptr32 Void

   +0x200 SystemDefaultActivationContextData : Ptr32 Void

   +0x204 SystemAssemblyStorageMap : Ptr32 Void

   +0x208 MinimumStackCommit : Uint4B
```

|  |  |  |
| --- | --- | --- |
| **偏移** | **名称** | **含义** |
| 0x002 | BeginDebugged | 是否被调试 |
| 0x008 | ImageBaseAddress | 执行映像的基地址 |
| 0x018 | ProcessHeap | 进程堆 |
| 0x064 | NumberOfProcessors | CPU个数 |
| 0x068 | NtGlobalFlag | 全局标志 |
| 0x078 | HeapSegmentReserve | 默认进程堆的总保留空间，1MB |
| 0x07C | HeapSegmentCommit | 默认进程堆的已提交空间 |
| 0x088 | NumberOfHeaps | 堆的个数 |
| 0x08C | MaximumNumberOfHeaps | 堆的最多个数 |
| 0x090 | ProcessHeaps | 保存堆句柄的数组地址 |
| 0x094 | GdiSharedHandleTable | GDI共享句柄表 |
| 0x0A4 | OSMajorVersion | 操作系统主版本号 |
| 0x0A8 | OSMinorVersion | 操作系统子版本号 |
| 0x0AC | OSBuildNumber | 操作系统构建号 |
| 0x0AE | OSCSDVersion | Service Pack版本号 |
| 0x0B0 | OSPlatformId | 系统类别，2代表NT，1代表9x，3代表Windows CE |
| 0x0B4 | ImageSubsystem | 环境子系统ID |
| 0x0B8 | ImageSubsystemMajorVersion | 环境子系统主版本号 |
| 0x0BC | ImageSubsystemMinorVersion | 环境子系统次版本号 |
| 0x1D4 | SessionId | 所属会话ID |

> 通过上面的三个结构分别在用户层和内核层保留了创建的进程的所有的信息，操作系统通过这些信息完成对进程的管理工作。因此，在创建进程的过程中上述的三个结构体是一定要在内存中被申请出来并初始化的。

# 三.进程的创建

Windows API提供了很多用于创建进程的函数。其中最简单的是`CreateProcess`，该函数会尝试使用与创建者相同的访问令牌新建一个进程。如果需要不同的令牌，可以使用`CreateProcessAsUser`函数或`CreateProcessWithTokenW`函数来实现。如果系统快速用特定用户凭据登录并使用所获得的令牌创建进程，此时可以使用`CreateProcesssWithLoginW`来实现。

创建一个进程的主要步骤如下：

1. 验证参数，将Windows子系统标志和选项转换为原生形式，解析，验证并转换属性列表为原生形式
2. 打开要在进程中执行的映像文件（.exe)
3. 创建Windows执行体进程对象
4. 创建初始线程（栈，上下文，Windows执行体线程对象）
5. 执行创建之后需要的，与Windows子系统有关的进程初始化操作
6. 开始执行初始线程（除非指定了`CREATE_SUSPENDED`标志）
7. 在新进程和线程上下文中完成地址空间的初始化操作（如加载需要的DLL），并开始执行程序的入口点

所有文档化的进程创建函数最终都会调用`Kernel32.dll`中的`CreateProcessInternalW`。以常见的`CreateProcessW`函数为例，可以看到在这个函数中就是将参数压入栈中，然后调用`CreateProcessInternalW`，因此接下来会以此函数为主进行分析

```
.text:7C802336 ; BOOL __stdcall CreateProcessW(LPCWSTR lpApplicationName, LPWSTR lpCommandLine, LPSECURITY_ATTRIBUTES lpProcessAttributes, LPSECURITY_ATTRIBUTES lpThreadAttributes, BOOL bInheritHandles, DWORD dwCreationFlags, LPVOID lpEnvironment, LPCWSTR lpCurrentDirectory, LPSTARTUPINFOW lpStartupInfo, LPPROCESS_INFORMATION lpProcessInformation)

.text:7C802336                 public _CreateProcessW@40

.text:7C802336 _CreateProcessW@40 proc near            ; CODE XREF: UnhandledExceptionFilter(x)+835↓p

.text:7C802336                                         ; ConsoleIMERoutine(x)+D2↓p

.text:7C802336                                         ; DATA XREF: ...

.text:7C802336

.text:7C802336 lpApplicationName= dword ptr  8

.text:7C802336 lpCommandLine   = dword ptr  0Ch

.text:7C802336 lpProcessAttributes= dword ptr  10h

.text:7C802336 lpThreadAttributes= dword ptr  14h

.text:7C802336 bInheritHandles = dword ptr  18h

.text:7C802336 dwCreationFlags = dword ptr  1Ch

.text:7C802336 lpEnvironment   = dword ptr  20h

.text:7C802336 lpCurrentDirectory= dword ptr  24h

.text:7C802336 lpStartupInfo   = dword ptr  28h

.text:7C802336 lpProcessInformation= dword ptr  2Ch

.text:7C802336

.text:7C802336                 mov     edi, edi

.text:7C802338                 push    ebp

.text:7C802339                 mov     ebp, esp

.text:7C80233B                 push    0               ; hNewToken

.text:7C80233D                 push    [ebp+lpProcessInformation] ; lpProcessInformation

.text:7C802340                 push    [ebp+lpStartupInfo] ; lpStartupInfo

.text:7C802343                 push    [ebp+lpCurrentDirectory] ; lpCurrentDirectory

.text:7C802346                 push    [ebp+lpEnvironment] ; lpEnvironment

.text:7C802349                 push    [ebp+dwCreationFlags] ; dwCreationFlags

.text:7C80234C                 push    [ebp+bInheritHandles] ; bInheritHandles

.text:7C80234F                 push    [ebp+lpThreadAttributes] ; lpThreadAttributes

.text:7C802352                 push    [ebp+lpProcessAttributes] ; lpProcessAtributes

.text:7C802355                 push    [ebp+lpCommandLine] ; lpCommandLine

.text:7C802358                 push    [ebp+lpApplicationName] ; lpApplicationName

.text:7C80235B                 push    0               ; hToken

.text:7C80235D                 call    _CreateProcessInternalW@48 ; CreateProcessInternalW(x,x,x,x,x,x,x,x,x,x,x,x)

.text:7C802362                 pop     ebp

.text:7C802363                 retn    28h

.text:7C802363 _CreateProcessW@40 endp
```

## 1.转换并验证参数和标志

在函数最开始，首先是将参数赋值到相应的局部变量中，同时也会一部分局部变量初始化为0。
