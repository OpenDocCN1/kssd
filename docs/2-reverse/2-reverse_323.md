

# Debug Tutorial(完全翻译)

> 原文：[`www.pediy.com/kssd/pediy12/104828-4.html`](https://www.pediy.com/kssd/pediy12/104828-4.html)

 

编写 WinDbg 扩展  
介绍  
  欢迎来到这个调试系列的第 4 篇。在这篇中，我们会稍微偏离一点实际的调试，而去研究一下对调试有帮助的东西。我绝对不会为了写调试扩展而去写，写这些东西必须要有一些原因，否则会变得很乏味。一旦你觉得你很愿意去做这件事，那将会是自然而然的事情。  
  在我进行调试的时候，我总是去检查栈和其他地方来寻找某些字符串。为什么我要这样做呢?我们不是电脑，我们懂语言，而不是数字。而许多程序甚至驱动程序都是以字符串为基础写的，这并不是说一切都是字符串，但总有一些字符串。如果你想的话，你也完全可以不用字符串，你可以只用数字。但是如果你要展现一个 UI 界面的话，也应该有一些字符串，不可能全部都用数字。程序员也是人，也要用某种语言去交流，而不是用二进制去交流。因此，程序内部会使用到很多字符串来代表一些东西，甚至驱动程序也是。  
  
有字符串又怎样呢？  
  它们让程序的可读性更强，这是最重要的一条。可能你在栈中找到了一个字符串，你会知道程序到底在干什么。这些字符串可能是环境字符串，文件名，设备名(COM1,\\Device\\xxx,etc)，对象名，用户名，GUIDs，等等。这些信息能很好的帮助你跟踪程序。  
  另外一个有趣的是，我们知道最常见的缓冲区溢出都要归咎于不当的字符串操作。可能是忘记了最后的 NULL，也可能是错误的判断了 API 的返回信息。这样的话，如果程序确实出现了这种情况，追踪将会变得更简单。  
  
我们该如何开始  
  我们从栈开始。当我们出错中断下来，首先就是使用”kb”和”DDS ESP”，然后就是使用”DC ESP”。让我们来看看例子。  
0:000> dc esp  
0006febc  77d43a09 77d43c7d 0006fefc 00000000  .:.w}<.w........  
0006fecc  00000000 00000000 00000000 0006ff1c  ................  
0006fedc  010028e4 0006fefc 00000000 00000000  .(..............  
0006feec  00000000 00000000 77e7ad86 00091ee7  ...........w....  
0006fefc  001a03e4 00000118 0000ffff bf8a75ed  .............u..  
0006ff0c  0768a2ca 00000229 00000251 00000000  ..h.)...Q.......  
0006ff1c  0006ffc0 01006c54 01000000 00000000  ....Tl..........  
0006ff2c  00091ee7 0000000a 00000000 00000000  ................  
0:000> dc  
0006ff3c  7ffdf000 80543940 f544fc5c 00000044  ....@9T.\\.D.D...  
0006ff4c  00092b28 00092b48 00092b70 00000000  (+..H+..p+......  
0006ff5c  00000000 00000000 00000000 00000000  ................  
0006ff6c  00000000 00000000 00000000 00000000  ................  
0006ff7c  00000000 ffffffff ffffffff ffffffff  ................  
0006ff8c  00091ee7 00000000 00000001 00272620  ............ &'.  
0006ff9c  00272d00 00000000 00000000 0006ff34  .-'.........4...  
0006ffac  e24296d0 0006ffe0 01006d14 01001888  ..B......m......  
0:000>  
0006ffbc  00000000 0006fff0 77e814c7 00000000  ...........w....  
0006ffcc  00000000 7ffdf000 f544fcf0 0006ffc8  ..........D.....  
0006ffdc  80534504 ffffffff 77e94809 77e91210  .ES......H.w...w  
0006ffec  00000000 00000000 00000000 01006ae0  .............j..  
0006fffc  00000000 78746341 00000020 00000001  ....Actx .......  
0007000c  00000654 0000007c 00000000 00000020  T...|....... ...  
0007001c  00000000 00000014 00000001 00000003  ................  
0007002c  00000034 000000ac 00000001 00000000  4...............  
0:000>  
0007003c  00000000 00000000 00000000 00000000  ................  
0007004c  00000002 00000000 00000000 00000000  ................  
0007005c  00000168 00000190 00000000 2d59495b  h...........\[IY-  
0007006c  000002f8 00000032 0000032c 000002b8  ....2...,.......  
0007007c  00000010 00000002 0000008c 00000002  ................  
0007008c  00000001 000000ac 00000538 00000001  ........8.......  
0007009c  00000002 000005e4 00000070 00000001  ........p.......  
000700ac  64487353 0000002c 00000001 00000001  SsHd,...........  
0:000>  
000700bc  00000003 00000002 0000008c 00000001  ................  
000700cc  00000000 0000002c 0000005e 0000005e  ....,...^...^...  
000700dc  00000000 00000000 00000000 00000000  ................  
000700ec  00000000 00000000 00000000 00000000  ................  
000700fc  00000000 00000002 00000028 00000034  ........(...4...  
0007010c  003a0043 0057005c 004e0049 004f0044  C.:.\\.W.I.N.D.O.  
0007011c  00530057 0030002e 0057005c 006e0069  W.S...0.\\.W.i.n.  
0007012c  00780053 005c0073 00000000 00000000  S.x.s.\\.........  
我打开了 notepad.exe，然后中断下来，Dump 出主线程的栈。栈中的字符串都是局部数据，例如 char x\[10\];。另外其他地方也会存储一些字符串，然后用指针来引用他们，或者直接通过参数传递，比如 CreateFile 的第一个参数就是接收一个字符串。  
  那么，我们通常都该做些什么呢？我会搜索栈，然后找一些可能的字符串，使用 DC 和 DA 或 DU 命令显示其中的字符串。但是这是件很乏味，并且很慢的事情。并且没有任何调试器命令可以帮助做这些事情，因此我只能自己写。  
  
如何自己写？  
  WinDbg 可以使用一些符合微软规范了 DLL，也就是说你可以写插件。你可以写类似于这样的插件!<mydatastructure> <address>来 Dump 出数据结构中的成员。然后，WinDbg 提供了”dt”命令，如果你有 PDB 文件的话，它能帮助你完成这样的功能。  
  使用”dt <yourdll>!<your data structure>”可以把结构中的名称都打印出来。我们来看一个例子。  
0:000> dt ntdll!\_PEB  
   +0x000 InheritedAddressSpace : UChar  
   +0x001 ReadImageFileExecOptions : UChar  
   +0x002 BeingDebugged    : UChar  
   +0x003 SpareBool        : UChar  
   +0x004 Mutant           : Ptr32 Void  
   +0x008 ImageBaseAddress : Ptr32 Void  
   +0x00c Ldr              : Ptr32 \_PEB\_LDR\_DATA  
   +0x010 ProcessParameters : Ptr32 \_RTL\_USER\_PROCESS\_PARAMETERS  
   +0x014 SubSystemData    : Ptr32 Void  
   +0x018 ProcessHeap      : Ptr32 Void  
   +0x01c FastPebLock      : Ptr32 \_RTL\_CRITICAL\_SECTION  
   +0x020 FastPebLockRoutine : Ptr32 Void  
   +0x024 FastPebUnlockRoutine : Ptr32 Void  
   +0x028 EnvironmentUpdateCount : Uint4B  
   +0x02c KernelCallbackTable : Ptr32 Void  
   +0x030 SystemReserved   : \[1\] Uint4B  
   +0x034 ExecuteOptions   : Pos 0, 2 Bits  
   +0x034 SpareBits        : Pos 2, 30 Bits  
   +0x038 FreeList         : Ptr32 \_PEB\_FREE\_BLOCK  
   +0x03c TlsExpansionCounter : Uint4B  
   +0x040 TlsBitmap        : Ptr32 Void  
   +0x044 TlsBitmapBits    : \[2\] Uint4B  
   +0x04c ReadOnlySharedMemoryBase : Ptr32 Void  
   +0x050 ReadOnlySharedMemoryHeap : Ptr32 Void  
   +0x054 ReadOnlyStaticServerData : Ptr32 Ptr32 Void  
   +0x058 AnsiCodePageData : Ptr32 Void  
   +0x05c OemCodePageData  : Ptr32 Void  
   +0x060 UnicodeCaseTableData : Ptr32 Void  
   +0x064 NumberOfProcessors : Uint4B  
   +0x068 NtGlobalFlag     : Uint4B  
   +0x070 CriticalSectionTimeout : \_LARGE\_INTEGER  
   +0x078 HeapSegmentReserve : Uint4B  
   +0x07c HeapSegmentCommit : Uint4B  
   +0x080 HeapDeCommitTotalFreeThreshold : Uint4B  
   +0x084 HeapDeCommitFreeBlockThreshold : Uint4B  
   +0x088 NumberOfHeaps    : Uint4B  
   +0x08c MaximumNumberOfHeaps : Uint4B  
   +0x090 ProcessHeaps     : Ptr32 Ptr32 Void  
   +0x094 GdiSharedHandleTable : Ptr32 Void  
   +0x098 ProcessStarterHelper : Ptr32 Void  
   +0x09c GdiDCAttributeList : Uint4B  
   +0x0a0 LoaderLock       : Ptr32 Void  
   +0x0a4 OSMajorVersion   : Uint4B  
   +0x0a8 OSMinorVersion   : Uint4B  
   +0x0ac OSBuildNumber    : Uint2B  
   +0x0ae OSCSDVersion     : Uint2B  
   +0x0b0 OSPlatformId     : Uint4B  
   +0x0b4 ImageSubsystem   : Uint4B  
   +0x0b8 ImageSubsystemMajorVersion : Uint4B  
   +0x0bc ImageSubsystemMinorVersion : Uint4B  
   +0x0c0 ImageProcessAffinityMask : Uint4B  
   +0x0c4 GdiHandleBuffer  : \[34\] Uint4B  
   +0x14c PostProcessInitRoutine : Ptr32  
   +0x150 TlsExpansionBitmap : Ptr32 Void  
   +0x154 TlsExpansionBitmapBits : \[32\] Uint4B  
   +0x1d4 SessionId        : Uint4B  
   +0x1d8 AppCompatFlags   : \_ULARGE\_INTEGER  
   +0x1e0 AppCompatFlagsUser : \_ULARGE\_INTEGER  
   +0x1e8 pShimData        : Ptr32 Void  
   +0x1ec AppCompatInfo    : Ptr32 Void  
   +0x1f0 CSDVersion       : \_UNICODE\_STRING  
   +0x1f8 ActivationContextData : Ptr32 Void  
   +0x1fc ProcessAssemblyStorageMap : Ptr32 Void  
   +0x200 SystemDefaultActivationContextData : Ptr32 Void  
   +0x204 SystemAssemblyStorageMap : Ptr32 Void  
   +0x208 MinimumStackCommit : Uint4B  
0:000>  
上面我们 Dump 出了\_PEB 结构中的信息。下面我们来找到自己的 PEB 然后 Dump。  
0:000> !teb  
TEB at 7ffde000  
    ExceptionList:        0006ffb0  
    StackBase:            00070000  
    StackLimit:           0005f000  
    SubSystemTib:         00000000  
    FiberData:            00001e00  
    ArbitraryUserPointer: 00000000  
    Self:                 7ffde000  
    EnvironmentPointer:   00000000  
    ClientId:             00000b80 . 00000f40  
    RpcHandle:            00000000  
    Tls Storage:          00000000  
    PEB Address:          7ffdf000  
    LastErrorValue:       0  
    LastStatusValue:      c0000034  
    Count Owned Locks:    0  
    HardErrorMode:        0  
0:000> dt ntdll!\_PEB 7ffdf000  
   +0x000 InheritedAddressSpace : 0 ''  
   +0x001 ReadImageFileExecOptions : 0 ''  
   +0x002 BeingDebugged    : 0x1 ''  
   +0x003 SpareBool        : 0 ''  
   +0x004 Mutant           : 0xffffffff  
   +0x008 ImageBaseAddress : 0x01000000  
   +0x00c Ldr              : 0x00191ea0  
   +0x010 ProcessParameters : 0x00020000  
   +0x014 SubSystemData    : (null)  
   +0x018 ProcessHeap      : 0x00090000  
   +0x01c FastPebLock      : 0x77fc49e0  
   +0x020 FastPebLockRoutine : 0x77f5b2a0  
   +0x024 FastPebUnlockRoutine : 0x77f5b380  
   +0x028 EnvironmentUpdateCount : 1  
   +0x02c KernelCallbackTable : 0x77d42a38  
   +0x030 SystemReserved   : \[1\] 0  
   +0x034 ExecuteOptions   : 0y00  
   +0x034 SpareBits        : 0y000000000000000000000000000000 (0)  
   +0x038 FreeList         : (null)  
   +0x03c TlsExpansionCounter : 0  
   +0x040 TlsBitmap        : 0x77fc4680  
   +0x044 TlsBitmapBits    : \[2\] 0x7ff  
   +0x04c ReadOnlySharedMemoryBase : 0x7f6f0000  
   +0x050 ReadOnlySharedMemoryHeap : 0x7f6f0000  
   +0x054 ReadOnlyStaticServerData : 0x7f6f0688  -> (null)  
   +0x058 AnsiCodePageData : 0x7ffb0000  
   +0x05c OemCodePageData  : 0x7ffc1000  
   +0x060 UnicodeCaseTableData : 0x7ffd2000  
   +0x064 NumberOfProcessors : 1  
   +0x068 NtGlobalFlag     : 0x70  
   +0x070 CriticalSectionTimeout : \_LARGE\_INTEGER 0xffffe86d\`079b8000  
   +0x078 HeapSegmentReserve : 0x100000  
   +0x07c HeapSegmentCommit : 0x2000  
   +0x080 HeapDeCommitTotalFreeThreshold : 0x10000  
   +0x084 HeapDeCommitFreeBlockThreshold : 0x1000  
   +0x088 NumberOfHeaps    : 5  
   +0x08c MaximumNumberOfHeaps : 0x10  
   +0x090 ProcessHeaps     : 0x77fc5a80  -> 0x00090000  
   +0x094 GdiSharedHandleTable : 0x00360000  
   +0x098 ProcessStarterHelper : (null)  
   +0x09c GdiDCAttributeList : 0x14  
   +0x0a0 LoaderLock       : 0x77fc1774  
   +0x0a4 OSMajorVersion   : 5  
   +0x0a8 OSMinorVersion   : 1  
   +0x0ac OSBuildNumber    : 0xa28  
   +0x0ae OSCSDVersion     : 0x100  
   +0x0b0 OSPlatformId     : 2  
   +0x0b4 ImageSubsystem   : 2  
   +0x0b8 ImageSubsystemMajorVersion : 4  
   +0x0bc ImageSubsystemMinorVersion : 0  
   +0x0c0 ImageProcessAffinityMask : 0  
   +0x0c4 GdiHandleBuffer  : \[34\] 0  
   +0x14c PostProcessInitRoutine : (null)  
   +0x150 TlsExpansionBitmap : 0x77fc4660  
   +0x154 TlsExpansionBitmapBits : \[32\] 0  
   +0x1d4 SessionId        : 0  
   +0x1d8 AppCompatFlags   : \_ULARGE\_INTEGER 0x0  
   +0x1e0 AppCompatFlagsUser : \_ULARGE\_INTEGER 0x0  
   +0x1e8 pShimData        : (null)  
   +0x1ec AppCompatInfo    : (null)  
   +0x1f0 CSDVersion       : \_UNICODE\_STRING "Service Pack 1"  
   +0x1f8 ActivationContextData : 0x00080000  
   +0x1fc ProcessAssemblyStorageMap : 0x000929a8  
   +0x200 SystemDefaultActivationContextData : 0x00070000  
   +0x204 SystemAssemblyStorageMap : (null)  
   +0x208 MinimumStackCommit : 0  
0:000>  
如果你还记得的话，!teb 命令可以显示”线程环境块”的信息。其中一个值就是 PEB 的地址。所以你看到，它不仅能打印出数据结构的信息，还能根据数据的类型打印出信息。那么我为什么要讲述这些呢？因为我们需要写调试扩展来打印出数据结构中的东西，如果你自己写代码完成上面的命令，那每次数据结构的改变，你的代码也都要跟着改变，所以使用”dt”命令会很方便。  
  
开始编写调试扩展  
  我这里建议写的调试扩展是!dumpstrings，这个命令可以遍历内存，然后打印出其中的信息。然后我们还可以写!dumpstrings esp 这样的命令来 Dump 出栈中的信息。  
  首先，我们来看看怎样编写扩展。WinDbg 需要你至少导出两个函数。如下：  
/\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
 \* ExtensionApiVersion  
 \*  
 \* Purpose: WINDBG will call this function to get the version  
 \*          of the API  
 \*  
 \*  Parameters:  
 \*     Void  
 \*  
 \*  Return Values:  
 \*     Pointer to a EXT\_API\_VERSION structure.  
 \*  
 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*/                
LPEXT\_API\_VERSION WDBGAPI ExtensionApiVersion (void)  
{  
    return &g\_ExtApiVersion;  
}  
  
  
/\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
 \* WinDbgExtensionDllInit  
 \*  
 \* Purpose: WINDBG will call this function to initialize  
 \*          the API  
 \*  
 \*  Parameters:  
 \*     Pointer to the API functions, Major Version, Minor Version  
 \*  
 \*  Return Values:  
 \*     Nothing  
 \*  
 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*/                
VOID WDBGAPI WinDbgExtensionDllInit (PWINDBG\_EXTENSION\_APIS   
           lpExtensionApis, USHORT usMajorVersion,   
           USHORT usMinorVersion)  
{  
     ExtensionApis = \*lpExtensionApis;  
}  
第一个函数 ExtensionApiVersion，就是返回一个版本号，我们只要提供一个版本号就行了。下面是 g\_ExtApiVersion 的声明：  
/\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
 \* Global Variable Needed For Versioning  
 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*/                
EXT\_API\_VERSION g\_ExtApiVersion = {  
         5 ,  
         5 ,  
         EXT\_API\_VERSION\_NUMBER ,  
         0  
     } ;  
EXT\_API\_VERSION\_NUMBER 在 wdbexts.h 有声明。要注意的是，还有其他很多调试扩展 DLL，比如 ntsdexts.h。这里我们只对现在版本的 CDB 和 WinDbg 调试器作讲解。我这里使用的也是 windbgexts.h，而不是 ntsdexts.h。如果你看头文件的话，你会注意到这两个你都有。  
  下面是 EXT\_API\_VERSION\_NUMBER 在我系统上的显示：  
#define EXT\_API\_VERSION\_NUMBER   5  
  
typedef struct EXT\_API\_VERSION {  
    USHORT  MajorVersion;  
    USHORT  MinorVersion;  
    USHORT  Revision;  
    USHORT  Reserved;  
} EXT\_API\_VERSION, \*LPEXT\_API\_VERSION;  
那么 5，5 是怎么得到的呢？这是我从 WinDbg 其他的调试扩展中找到的。我还发现老版本的 WinDbg 使用的是 3，5。这些都只要根据实际情况填写就可以了，然后我们需要编写一个框架，然后编写我们的命令。  
  WinDbgExtensionDllInit 这个 API 只是给你一个虚拟函数表，这个表命名为一个固定的名字，当然不是必须的，但会让你编写起来更简单。原因是 windbgexts.h 包含了一些宏，可以调用这个结构中的函数，如果不使用相同的名字，这些宏你就无法使用了。下面是我全局变量：  
/\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
 \* Global Variable Needed For Functions  
 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*/                
WINDBG\_EXTENSION\_APIS ExtensionApis = {0};  
这个结构并不大，使用宏让你的工作变得更见简单。下面是 WINDBGEXTS.H 宏的显示：  
#define dprintf          (ExtensionApis.lpOutputRoutine)  
#define GetExpression    (ExtensionApis.lpGetExpressionRoutine)  
#define GetSymbol        (ExtensionApis.lpGetSymbolRoutine)  
#define Disassm          (ExtensionApis.lpDisasmRoutine)  
#define CheckControlC    (ExtensionApis.lpCheckControlCRoutine)  
#define ReadMemory       (ExtensionApis.lpReadProcessMemoryRoutine)  
#define WriteMemory      (ExtensionApis.lpWriteProcessMemoryRoutine)  
#define GetContext       (ExtensionApis.lpGetThreadContextRoutine)  
#define SetContext       (ExtensionApis.lpSetThreadContextRoutine)  
#define Ioctl            (ExtensionApis.lpIoctlRoutine)  
#define StackTrace       (ExtensionApis.lpStackTraceRoutine)  
下面我们该干什么呢？除了这两个 API，你还可以写一个 CheckVersion()函数，让你的命令工作在特定的 WINDBG 版本下。但是我发现这完全没用，所以我并没有写。所以我们开始写自己的函数吧。  
  这第一个函数很简单。我们将完成”!help”来打印帮助信息。  
/\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
 \* !help  
 \*  
 \* Purpose: WINDBG will call this API when the user types !help  
 \*            
 \*  
 \*  Parameters:  
 \*     N/A  
 \*  
 \*  Return Values:  
 \*     N/A  
 \*  
 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*/  
DECLARE\_API (help)  
{  
    dprintf("Toby's Debug Extensions\\n\\n");  
    dprintf("!dumpstrings <ADDRESS register> - Dumps 20 strings in"\\  
       "ANSI/UNICODE using this address as a pointer to strings (useful for" \\  
       "dumping strings on the stack) \\n");  
       /\* String Split so it is readable in this article. \*/  
}  
Dprintf();就是调试输出，和 printf()一样，它将信息打印到 debugger 中。DECLARE\_API(<command>)可以帮助你很简单的声明一个 API 名字。记住，这个函数名字和你在调试器中使用命令的名字一样。在这个例子中，我们就使用!help 或者!<dllname>.help。这是一个很简单的函数，仅仅打印一些消息给用户。  
  下面我们要做的就是完成这个字符串函数。这个函数需要内存地址作为参数，我会将这个函数写的更像现有的命令，比如第一次你使用 dc <address>，显示了一些信息，然后再使用 dc，会接着上面的最后一个地址显示信息。下面我们来看看。  
/\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
 \* !dumpstrings  
 \*  
 \* Purpose: WINDBG will call this API when the user types !dumpstrings  
 \*            
 \*  
 \*  Parameters:  
 \*     !dumpstrings or !dumpstrings <ADDRESS register>  
 \*  
 \*  Return Values:  
 \*     N/A  
 \*  
 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*/  
DECLARE\_API (dumpstrings)  
{  
    static ULONG Address = 0;  
    ULONG  GetAddress, StringAddress, Index = 0, Bytes;  
    WCHAR MyString\[51\] = {0};  
      
      
    GetAddress = GetExpression(args);  
  
    if(GetAddress != 0)  
    {  
        Address = GetAddress;  
    }  
          
    dprintf("STACK   ADDR   STRING \\n");  
  
    for(Index = 0; Index < 4\*20; Index+=4)  
    {  
        memset(MyString, 0, sizeof(MyString));  
          
        Bytes = 0;  
  
        ReadMemory(Address + Index, &StringAddress,   
                           sizeof(StringAddress), &Bytes);  
  
        if(Bytes)  
        {  
           Bytes = 0;  
  
           ReadMemory(StringAddress, MyString,   
                 sizeof(MyString) - 2, &Bytes);  
  
           if(Bytes)  
           {  
              dprintf("%08x : %08x = (UNICODE) \\"%ws\\"\\n",   
                       Address + Index, StringAddress, MyString);  
              dprintf("%08x : %08x = (ANSI)    \\"%s\\"\\n",   
                       Address + Index, StringAddress, MyString);  
           }  
           else  
           {  
              dprintf("%08x : %08x =  Address Not Valid\\n",   
                             Address + Index, StringAddress);  
           }  
        }  
        else  
        {  
           dprintf("%08x : Address Not Valid\\n", Address + Index);  
        }  
    }  
  
    Address += Index;  
}  
我使用的第一个函数是 GetExpression()。在新版本的 WINDBG 中，它的用法像这样。ADDRESS GetExpression(SYMBOLIC  STRING)，你可以传递一个符号字符串，比如这个命令的参数，在这里是地址。参数就存储在 args 中，因此我们只要传递 args 就可以了。这里它会将符号，地址，或寄存器解析成数字，比如我们传递 ESP 参数，就会取 ESP 中的值。  
  我定义了一个静态变量，如果 GetExpression()返回 0，可能就是没有参数，这种情况下，就可以使用静态变量中存储的值继续后面的工作，并且会在这个值之后继续显示，这就完成了上面说的功能，函数的最后我都会存储这个 Address。  
  下面使用的函数是 dprintf()，上面我已经解释过了，和 printf 一样。在每次循环中，我都会将地址+4，然后从 0 到 4\*20 打印 20 个地址中的值。  
  但是你不能简单的使用这个地址，因为你并不在程序的进程空间中，因此，WINDBG 提供 ReadMemory 这样一个函数，这个函数有 4 个参数(如果你在 MSDN 中找不到某个 API 的话，你可以到 windbgexts.h 中去找原型)。  
ReadMemory(Address In Process To Read,  
           Local Variable to store the memory read,   
           size of the local variable,   
           pointer to a DWORD that returns the number   
           of bytes read from the memory location);  
因此，我们把程序中内存的指针传递过去，还有一个存放数据的指针用来返回，最后是返回的字节数。如果没有字节返回，我们只要打印出不正确的地址信息即可；如果有字节返回，我们就可以用这个地址读取 49 个字节(我们使用了 51 是因为最后要放两个 NULL，支持 Unicode)。如果成功读取，就可以使用 dprintf()以 ANSI 和 UNICODE 方式打印出字符串，如果返回 0 字节，就打印错误信息。  
  下面我们需要建立一个.DEF 文件来导出函数。  
LIBRARY "TDBG.DLL"  
  
EXPORTS  
    WinDbgExtensionDllInit  
    ExtensionApiVersion  
    dumpstrings  
help  
  
下面我们生成这个工程。我喜欢使用 make files，并且我喜欢使用 Visual Slickedit 作为编辑器。我从来不使用 VC++IDE。因此，在这个工程中，我创建了一个 makefile。首先，运行 VC++的 BIN 目录中的 VCVARS32.BAT，我喜欢把它放到 C:\\方便使用，然后在源代码目录下使用 nmake 命令就可以了。  
C:\\Programming\\Programs\\debugext\\src\\debug>\\vcvars32  
Setting environment for using Microsoft Visual C++ tools.  
C:\\Programming\\Programs\\debugext\\src\\debug>nmake  
  
Microsoft (R) Program Maintenance Utility   Version 6.00.8168.0  
Copyright (C) Microsoft Corp 1988-1998. All rights reserved.  
  
        cl /nologo /MD /W3 /Oxs /Zi  /I ".\\inc"  /D "WIN32" /DLL /D "\_WINDOWS"  
/Fr.\\obj\\i386\\\\ /Fo.\\obj\\i386\\\\ /Fd.\\obj\\i386\\\\ /c .\\tdbg.c  
tdbg.c  
C:\\PROGRA~1\\MICROS~2\\VC98\\INCLUDE\\wdbgexts.h(526) : warning C4101: 'li' : unrefe  
renced local variable  
        link.exe /DLL /nologo /def:tdbg.def /out:..\\..\\bin\\tdbg.dll  /pdb:tdbg.p  
db /debug /debugtype:both USER32.LIB  KERNEL32.LIB .\\obj\\i386\\tdbg.obj  
   Creating library ..\\..\\bin\\tdbg.lib and object ..\\..\\bin\\tdbg.exp  
        rebase.exe -b 0x00100000 -x ..\\..\\bin -a ..\\..\\bin\\tdbg.dll  
  
REBASE: Total Size of mapping 0x00010000  
REBASE: Range 0x00100000 -0x00110000  
  
C:\\Programming\\Programs\\debugext\\src\\debug>nmake clean  
  
Microsoft (R) Program Maintenance Utility   Version 6.00.8168.0  
Copyright (C) Microsoft Corp 1988-1998. All rights reserved.  
  
Deleted file - C:\\Programming\\Programs\\debugext\\src\\debug\\obj\\i386\\tdbg.obj  
Deleted file - C:\\Programming\\Programs\\debugext\\src\\debug\\obj\\i386\\tdbg.sbr  
Deleted file - C:\\Programming\\Programs\\debugext\\src\\debug\\obj\\i386\\vc60.pdb  
  
C:\\Programming\\Programs\\debugext\\src\\debug>  
如果你想再生成一遍，你需要改变一下你的代码以让日期更新，或使用”nmake  clean”重新生成。现在你就可以得到这个文件了。  
Volume in drive C has no label.  
 Volume Serial Number is 2CF8-F7B5  
  
 Directory of C:\\Programming\\Programs\\debugext\\bin  
  
03/25/2004  08:56 PM    <DIR>          .  
03/25/2004  08:56 PM    <DIR>          ..  
03/25/2004  09:53 PM             2,412 tdbg.dbg  
03/25/2004  09:53 PM            20,752 tdbg.dll  
03/25/2004  09:53 PM             1,044 tdbg.exp  
03/25/2004  09:53 PM             2,538 tdbg.lib  
03/25/2004  09:53 PM            82,944 tdbg.pdb  
               5 File(s)        109,690 bytes  
               2 Dir(s)  12,229,009,408 bytes free  
  
下面只要把这个文件复制到 WinDbg 能找到的地方就可以了。然后你就可以用!load 和!unload 来加载和卸载这个扩展。最后，你就可以使用!<dll name>.<function name>或!<function name>了，加上 DLL 的名字会强制 WINDBG 找这个 DLL 中的相应函数，不同 DLL 可能使用相同的名字，所以这样就可以区分开了。  
  
我们来试试成果  
我们已经创建了这个调试扩展，将它放到 WINDBG 可以加载的目录中后，就可以使用!tdbg.dumpstrings  esp 来 Dump 栈中所有的字符串了。下面我要 Dump 栈中所有字符串的指针了，我们来看看会发生什么。  
0:000> !tdbg.dumpstrings esp  
STACK   ADDR   STRING  
0006febc : 77d43a09 = (UNICODE) ""  
0006febc : 77d43a09 = (ANSI)    "┬►"  
0006fec0 : 77d43c7d = (UNICODE) ""  
0006fec0 : 77d43c7d = (ANSI)    "N♦∙☻☺"  
0006fec4 : 0006fefc = (UNICODE) ""  
0006fec4 : 0006fefc = (ANSI)    "▐♥↔"  
0006fec8 : 00000000 =  Address Not Valid  
0006fecc : 00000000 =  Address Not Valid  
0006fed0 : 00000000 =  Address Not Valid  
0006fed4 : 00000000 =  Address Not Valid  
0006fed8 : 0006ff1c = (UNICODE) ""  
0006fed8 : 0006ff1c = (ANSI)    "└ ♠"  
0006fedc : 010028e4 = (UNICODE) ""  
0006fedc : 010028e4 = (ANSI)    "└uΦ├∩   5"  
0006fee0 : 0006fefc = (UNICODE) ""  
0006fee0 : 0006fefc = (ANSI)    "▐♥↔"  
0006fee4 : 00000000 =  Address Not Valid  
0006fee8 : 00000000 =  Address Not Valid  
0006feec : 00000000 =  Address Not Valid  
0006fef0 : 00000000 =  Address Not Valid  
0006fef4 : 77e7ad86 = (UNICODE) ""  
0006fef4 : 77e7ad86 = (ANSI)    "|$♦"  
0006fef8 : 00091ee8 = (UNICODE) ""  
0006fef8 : 00091ee8 = (ANSI)    ""  
0006fefc : 001d03de = (UNICODE) ""  
0006fefc : 001d03de = (ANSI)    ""  
0006ff00 : 00000118 =  Address Not Valid  
0006ff04 : 0000ffff =  Address Not Valid  
0006ff08 : bf8a75ed =  Address Not Valid  
0:000> !tdbg.dumpstrings  
STACK   ADDR   STRING  
0006ff0c : 077d5cc8 =  Address Not Valid  
0006ff10 : 000001f3 =  Address Not Valid  
0006ff14 : 000001df =  Address Not Valid  
0006ff18 : 00000000 =  Address Not Valid  
0006ff1c : 0006ffc0 = (UNICODE) ""  
0006ff1c : 0006ffc0 = (ANSI)    "≡ ♠"  
0006ff20 : 01006c54 = (UNICODE) ""  
0006ff20 : 01006c54 = (ANSI)    "≡u9\]ΣuV ▄↕"  
0006ff24 : 01000000 = (UNICODE) ""  
0006ff24 : 01000000 = (ANSI)    "MZ"  
0006ff28 : 00000000 =  Address Not Valid  
0006ff2c : 00091ee8 = (UNICODE) ""  
0006ff2c : 00091ee8 = (ANSI)    ""  
0006ff30 : 0000000a =  Address Not Valid  
0006ff34 : 00000000 =  Address Not Valid  
0006ff38 : 00000000 =  Address Not Valid  
0006ff3c : 7ffdf000 = (UNICODE) ""  
0006ff3c : 7ffdf000 = (ANSI)    ""  
0006ff40 : 80543940 =  Address Not Valid  
0006ff44 : f4910c5c =  Address Not Valid  
0006ff48 : 00000044 =  Address Not Valid  
0006ff4c : 00092b30 = (UNICODE) ""  
0006ff4c : 00092b30 = (ANSI)    ""  
0006ff50 : 00092b50 = (UNICODE) ""  
0006ff50 : 00092b50 = (ANSI)    "WinSta0\\Default"  
0006ff54 : 00092b78 = (UNICODE) ""  
0006ff54 : 00092b78 = (ANSI)    "C:\\WINDOWS.0\\System32\\notepad.exe"  
0006ff58 : 00000000 =  Address Not Valid  
0:000> !tdbg.dumpstrings  
STACK   ADDR   STRING  
0006ff5c : 00000000 =  Address Not Valid  
0006ff60 : 00000000 =  Address Not Valid  
0006ff64 : 00000000 =  Address Not Valid  
0006ff68 : 00000000 =  Address Not Valid  
0006ff6c : 00000000 =  Address Not Valid  
0006ff70 : 00000000 =  Address Not Valid  
0006ff74 : 00000000 =  Address Not Valid  
0006ff78 : 00000000 =  Address Not Valid  
0006ff7c : 00000000 =  Address Not Valid  
0006ff80 : ffffffff =  Address Not Valid  
0006ff84 : ffffffff =  Address Not Valid  
0006ff88 : ffffffff =  Address Not Valid  
0006ff8c : 00091ee8 = (UNICODE) ""  
0006ff8c : 00091ee8 = (ANSI)    ""  
0006ff90 : 00000000 =  Address Not Valid  
0006ff94 : 00000001 =  Address Not Valid  
0006ff98 : 00272620 = (UNICODE) ""  
0006ff98 : 00272620 = (ANSI)    "(&'"  
0006ff9c : 00272d00 = (UNICODE) ""  
0006ff9c : 00272d00 = (ANSI)    "░-'"  
0006ffa0 : 00000000 =  Address Not Valid  
0006ffa4 : 00000000 =  Address Not Valid  
0006ffa8 : 0006ff34 = (UNICODE) ""  
0006ffa8 : 0006ff34 = (ANSI)    ""  
我们在栈中找到了两个字符串，看起来不错。你会得到很多垃圾和没用的信息，还有你想得到的字符串信息。  
  
总结  
  我希望你很高兴自己学习编写调试扩展，并且这个例子会给你带来帮助。另外还有其他一些 API 你可以使用，后面的教程可能也会解释一些，基本的操作这里都已经讲述了。将来，我们会探索更多的高级调试命令。