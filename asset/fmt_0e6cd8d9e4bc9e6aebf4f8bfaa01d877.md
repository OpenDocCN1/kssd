
+0x003 `SpareBool` : `0` `''`
+0x004 `Mutant` : `0xffffffff`
+0x008 `ImageBaseAddress` : `0x01000000`
+0x00c `Ldr` : `0x00191ea0`
+0x010 `ProcessParameters` : `0x00020000`
+0x014 `SubSystemData` : `(null)`
+0x018 `ProcessHeap` : `0x00090000`
+0x01c `FastPebLock` : `0x77fc49e0`
+0x020 `FastPebLockRoutine` : `0x77f5b2a0`
+0x024 `FastPebUnlockRoutine` : `0x77f5b380`
+0x028 `EnvironmentUpdateCount` : `1`
+0x02c `KernelCallbackTable` : `0x77d42a38`
+0x030 `SystemReserved` : `[1] 0`
+0x034 `ExecuteOptions` : `0y00`
+0x034 `SpareBits` : `0y000000000000000000000000000000` `(0)`
+0x038 `FreeList` : `(null)`
+0x03c `TlsExpansionCounter` : `0`
+0x040 `TlsBitmap` : `0x77fc4680`
+0x044 `TlsBitmapBits` : `[2] 0x7ff`
+0x04c `ReadOnlySharedMemoryBase` : `0x7f6f0000`
+0x050 `ReadOnlySharedMemoryHeap` : `0x7f6f0000`
+0x054 `ReadOnlyStaticServerData` : `0x7f6f0688` -> `(null)`
+0x058 `AnsiCodePageData` : `0x7ffb0000`
+0x05c `OemCodePageData` : `0x7ffc1000`
+0x060 `UnicodeCaseTableData` : `0x7ffd2000`
+0x064 `NumberOfProcessors` : `1`
+0x068 `NtGlobalFlag` : `0x70`
+0x070 `CriticalSectionTimeout` : `_LARGE_INTEGER` `0xffffe86d``079b8000`
+0x078 `HeapSegmentReserve` : `0x100000`
+0x07c `HeapSegmentCommit` : `0x2000`
+0x080 `HeapDeCommitTotalFreeThreshold` : `0x10000`
+0x084 `HeapDeCommitFreeBlockThreshold` : `0x1000`
+0x088 `NumberOfHeaps` : `5`
+0x08c `MaximumNumberOfHeaps` : `0x10`
+0x090 `ProcessHeaps` : `0x77fc5a80` -> `0x00090000`
+0x094 `GdiSharedHandleTable` : `0x00360000`
+0x098 `ProcessStarterHelper` : `(null)`
+0x09c `GdiDCAttributeList` : `0x14`
+0x0a0 `LoaderLock` : `0x77fc1774`
+0x0a4 `OSMajorVersion` : `5`
+0x0a8 `OSMinorVersion` : `1`
+0x0ac `OSBuildNumber` : `0xa28`
+0x0ae `OSCSDVersion` : `0x100`
+0x0b0 `OSPlatformId` : `2`
+0x0b4 `ImageSubsystem` : `2`
+0x0b8 `ImageSubsystemMajorVersion` : `4`
+0x0bc `ImageSubsystemMinorVersion` : `0`
+0x0c0 `ImageProcessAffinityMask` : `0`
+0x0c4 `GdiHandleBuffer` : `[34] 0`
+0x14c `PostProcessInitRoutine` : `(null)`
+0x150 `TlsExpansionBitmap` : `0x77fc4660`
+0x154 `TlsExpansionBitmapBits` : `[32] 0`
+0x1d4 `SessionId` : `0`
+0x1d8 `AppCompatFlags` : `_ULARGE_INTEGER` `0x0`
+0x1e0 `AppCompatFlagsUser` : `_ULARGE_INTEGER` `0x0`
+0x1e8 `pShimData` : `(null)`
+0x1ec `AppCompatInfo` : `(null)`
+0x1f0 `CSDVersion` : `_UNICODE_STRING` `"Service Pack 1"`
+0x1f8 `ActivationContextData` : `0x00080000`
+0x1fc `ProcessAssemblyStorageMap` : `0x000929a8`
+0x200 `SystemDefaultActivationContextData` : `0x00070000`
+0x204 `SystemAssemblyStorageMap` : `(null)`
+0x208 `MinimumStackCommit` : `0`

`0:000>`

如果你还记得的话，`!teb` 命令可以显示"线程环境块"的信息。其中一个值就是 `PEB` 的地址。所以你看到，它不仅能打印出数据结构的信息，还能根据数据的类型打印出信息。那么我为什么要讲述这些呢？因为我们需要写调试扩展来打印出数据结构中的东西，如果你自己写代码完成上面的命令，那每次数据结构的改变，你的代码也都要跟着改变，所以使用`"dt"`命令会很方便。

### 开始编写调试扩展

我这里建议写的调试扩展是`!dumpstrings`，这个命令可以遍历内存，然后打印出其中的信息。然后我们还可以写`!dumpstrings esp` 这样的命令来 Dump 出栈中的信息。

首先，我们来看看怎样编写扩展。`WinDbg` 需要你至少导出两个函数。如下：

```c
/****************************************************************
 * ExtensionApiVersion
 *
 * Purpose: WINDBG will call this function to get the version
 *          of the API
 *
 *  Parameters:
 *     Void
 *
 *  Return Values:
 *     Pointer to a EXT_API_VERSION structure.
 *
 ****************************************************************/

LPEXT_API_VERSION WDBGAPI ExtensionApiVersion (void)
{
    return &g_ExtApiVersion;
}

/****************************************************************
 * WinDbgExtensionDllInit
 *
 * Purpose: WINDBG will call this function to initialize
 *          the API
 *
 *  Parameters:
 *     Pointer to the API functions, Major Version, Minor Version
 *
 *  Return Values:
 *     Nothing
 *
 ****************************************************************/

VOID WDBGAPI WinDbgExtensionDllInit (PWINDBG_EXTENSION_APIS 
         lpExtensionApis, USHORT usMajorVersion, 
         USHORT usMinorVersion)
{
    ExtensionApis = *lpExtensionApis;
}
```

第一个函数 `ExtensionApiVersion`，就是返回一个版本号，我们只要提供一个版本号就行了。下面是 `g_ExtApiVersion` 的声明：

```c
/****************************************************************
 * Global Variable Needed For Versioning
 ****************************************************************/

EXT_API_VERSION g_ExtApiVersion = {
         5 ,
         5 ,
         EXT_API_VERSION_NUMBER ,
         0
     } ;
```

`EXT_API_VERSION_NUMBER` 在 `wdbexts.h` 有声明。要注意的是，还有其他很多调试扩展 DLL，比如 `ntsdexts.h`。这里我们只对现在版本的 `CDB` 和 `WinDbg` 调试器作讲解。我这里使用的也是 `windbgexts.h`，而不是 `ntsdexts.h`。如果你看头文件的话，你会注意到这两个你都有。

下面是 `EXT_API_VERSION_NUMBER` 在我系统上的显示：

```c
#define EXT_API_VERSION_NUMBER   5

typedef struct EXT_API_VERSION {
    USHORT  MajorVersion;
    USHORT  MinorVersion;
    USHORT  Revision;
    USHORT  Reserved;
} EXT_API_VERSION, *LPEXT_API_VERSION;
```

那么 `5`，`5` 是怎么得到的呢？这是我从 `WinDbg` 其他的调试扩展中找到的。我还发现老版本的 `WinDbg` 使用的是 `3`，`5`。这些都只要根据实际情况填写就可以了，然后我们需要编写一个框架，然后编写我们的命令。

`WinDbgExtensionDllInit` 这个 API 只是给你一个虚拟函数表，这个表命名为一个固定的名字，当然不是必须的，但会让你编写起来更简单。原因是 `windbgexts.h` 包含了一些宏，可以调用这个结构中的函数，如果不使用相同的名字，这些宏你就无法使用了。下面是我全局变量：

```c
/****************************************************************
 * Global Variable Needed For Functions
 ****************************************************************/

WINDBG_EXTENSION_APIS ExtensionApis = {0};
```

这个结构并不大，使用宏让你的工作变得更见简单。下面是 `WINDBGEXTS.H` 宏的显示：

```c
#define dprintf          (ExtensionApis.lpOutputRoutine)
#define GetExpression    (ExtensionApis.lpGetExpressionRoutine)
#define GetSymbol        (ExtensionApis.lpGetSymbolRoutine)
#define Disassm          (ExtensionApis.lpDisasmRoutine)
#define CheckControlC    (ExtensionApis.lpCheckControlCRoutine)
#define ReadMemory       (ExtensionApis.lpReadProcessMemoryRoutine)
#define WriteMemory      (ExtensionApis.lpWriteProcessMemoryRoutine)
#define GetContext       (ExtensionApis.lpGetThreadContextRoutine)
#define SetContext       (ExtensionApis.lpSetThreadContextRoutine)
#define Ioctl            (ExtensionApis.lpIoctlRoutine)
#define StackTrace       (ExtensionApis.lpStackTraceRoutine)
```

下面我们该干什么呢？除了这两个 API，你还可以写一个 `CheckVersion()`函数，让你的命令工作在特定的 `WINDBG` 版本下。但是我发现这完全没用，所以我并没有写。所以我们开始写自己的函数吧。

这第一个函数很简单。我们将完成`"!help"`来打印帮助信息。

```c
/****************************************************************
 * !help
 *
 * Purpose: WINDBG will call this API when the user types !help
 *          
 *
 *  Parameters:
 *     N/A
 *
 *  Return Values:
 *     N/A
 *
 ****************************************************************/

DECLARE_API (help)
{
    dprintf("Toby's Debug Extensions\n\n");
    dprintf("!dumpstrings <ADDRESS register> - Dumps 20 strings in"\
       "ANSI/UNICODE using this address as a pointer to strings (useful for" \
       "dumping strings on the stack) \n");
       /* String Split so it is readable in this article. */
}
```
