

# fs TIB TEB PEB

> 原文：[`www.pediy.com/kssd/pediy10/100190.html`](https://www.pediy.com/kssd/pediy10/100190.html)

在本学院拜读多位大牛的著作,自己整理的一些结构(有点不够完整)....  
都是用户模式的结构  
首先感谢看雪提供这么好的学习环境  
声明:以下内容完全取自于看雪,我只是灌水  
附件:方便自己阅读的一个小程式..怡笑大方了...  
  
fs:7FFDF000  
nt!\_TEB  
TEB at      fs:7FFDF000  
  
   +0x000   NtTib                       // \_NT\_TIB  
   +0x01c   EnvironmentPointer          // Ptr32 Void  
   +0x020   ClientId                    // \_CLIENT\_ID  
   +0x028   ActiveRpcHandle             // Ptr32 Void  
   +0x02c   ThreadLocalStoragePointer   // Ptr32 Void  
   +0x030   ProcessEnvironmentBlock     // Ptr32 \_PEB  这里指向 PEB 表，即进程环境块  
            LastErrorValue  
            LastStatusValue  
            Count Owned Locks  
            HardErrorsMode  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
typedef struct \_NT\_TIB          //sizeof  1ch  
{  
 00h   struct \_EXCEPTION\_REGISTRATION  \*ExceptionList;          //SEH 链入口  
 04h   PVOID                            StackBase;              //堆栈基址  
 08h   PVOID                            StackLimit;             //堆栈大小  
 0ch   PVOID                            SubSystemTib;  
       union {  
           PVOID                FiberData;  
 10h       DWORD                Version;  
       };  
 14h   PVOID                            ArbitraryUserPointer;  
 18h   struct \_NT\_TIB                   \*Self;                  //本 NT\_TIB 结构自身的线性地址  
}NT\_TIB;  
  
typedef     NT\_TIB      \*PNT\_TIB;  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
SEH 链入口  
  
fs\[0\]->\*ExceptionList  
  
typedef struc \_EXCEPTION\_REGISTRATION  
{  
    struc EXCEPTION\_REGISTRATION    \*Prev;      //前一个\_EXCEPTION\_REGISTRATION 结构  
    DWORD                           Handler;    //异常处理过程地址  
    struct scopetable\_entry         \*scopetable;  
    int                             trylevel;  
    int                             \_ebp;  
    PEXCEPTION\_POINTERS             xpointers;  
}  
    EXCEPTION\_REGISTRATION,  
    \*PEXCEPTION\_REGISTRATION;  
////////////////////////////////////////////////  
typedef struct \_EXCEPTION\_POINTERS  
{  
    PEXCEPTION\_RECORD   ExceptionRecord;        //指向一个 EXCEPTION\_RECORD 结构  
    PCONTEXT            ContextRecord;          //指向向一个 CONTEXT 结构  
}  
    EXCEPTION\_POINTERS,  
    \*PEXCEPTION\_POINTERS;  
/////////////////////////////////////////////////  
typedef struct \_EXCEPTION\_RECORD  
{  
  00h  DWORD                     ExceptionCode;      //异常事件码  
  04h  DWORD                     ExceptionFlags;     //标志  
  08h  struct \_EXCEPTION\_RECORD  \*ExceptionRecord;   //下一个 EXCEPTION\_RECORD 结构地址  
  0ch  PVOID                     ExceptionAddress;   //异常发生的地址  
  10h  DWORD                     NumberParameters;   //ExceptionInformation 的 dword 数目  
  14h  ULONG\_PTR ExceptionInformation\[EXCEPTION\_MAXIMUM\_PARAMETERS\];  
} 1ch  
    EXCEPTION\_RECORD;  
  
    typedef     EXCEPTION\_RECORD        \*PEXCEPTION\_RECORD;  
    #define     EXCEPTION\_MAXIMUM\_PARAMETERS    15  
/////////////////////////////////////////////////////////////////  
typedef struct \_CONTEXT {  
    DWORD           ContextFlags    // -|               +00h  
    DWORD           Dr0             //  |               +04h  
    DWORD           Dr1             //  |               +08h  
    DWORD           Dr2             //  >调试寄存器     +0Ch  
    DWORD           Dr3             //  |               +10h  
    DWORD           Dr6             //  |               +14h  
    DWORD           Dr7             // -|               +18h  
  
    FLOATING\_SAVE\_AREA FloatSave;   //浮点寄存器区      +1Ch~~~88h  
  
    DWORD           SegGs           //-|                +8Ch  
    DWORD           SegFs           // |\\段寄存器       +90h  
    DWORD           SegEs           // |/               +94h  
    DWORD           SegDs           //-|                +98h  
  
    DWORD           Edi             //\_\_\_\_\_\_\_\_          +9Ch  
    DWORD           Esi             // |  通用          +A0h  
    DWORD           Ebx             // |   寄           +A4h  
    DWORD           Edx             // |   存           +A8h  
    DWORD           Ecx             // |   器           +ACh  
    DWORD           Eax             //\_|\_\_\_ 组\_          +B0h  
  
    DWORD           Ebp             //++++++            +B4h  
    DWORD           Eip             // |控制            +B8h  
    DWORD           SegCs           // |寄存            +BCh  
    DWORD           EFlag           // |器组            +C0h  
    DWORD           Esp             // |                +C4h  
    DWORD           SegSs           //++++++            +C8h  
  
    BYTE    ExtendedRegisters\[MAXIMUM\_SUPPORTED\_EXTENSION\];  
} CONTEXT;  
    typedef     CONTEXT     \*PCONTEXT;  
    #define     MAXIMUM\_SUPPORTED\_EXTENSION     512  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
fs:\[30\]->PEB  
typedef struct \_PEB {               // Size: 0x1D8  
    000h    UCHAR           InheritedAddressSpace;  
    001h    UCHAR           ReadImageFileExecOptions;  
    002h    UCHAR           BeingDebugged;              //Debug 运行标志  
    003h    UCHAR           SpareBool;  
    004h    HANDLE          Mutant;  
    008h    HINSTANCE       ImageBaseAddress;           //程序加载的基地址  
    00Ch    struct \_PEB\_LDR\_DATA    \*Ldr                //Ptr32 \_PEB\_LDR\_DATA  
    010h    struct \_RTL\_USER\_PROCESS\_PARAMETERS  \*ProcessParameters;  
    014h    ULONG           SubSystemData;  
    018h    HANDLE          DefaultHeap;  
    01Ch    KSPIN\_LOCK      FastPebLock;  
    020h    ULONG           FastPebLockRoutine;  
    024h    ULONG           FastPebUnlockRoutine;  
    028h    ULONG           EnvironmentUpdateCount;  
    02Ch    ULONG           KernelCallbackTable;  
    030h    LARGE\_INTEGER   SystemReserved;  
    038h    struct \_PEB\_FREE\_BLOCK  \*FreeList  
    03Ch    ULONG           TlsExpansionCounter;  
    040h    ULONG           TlsBitmap;  
    044h    LARGE\_INTEGER   TlsBitmapBits;  
    04Ch    ULONG           ReadOnlySharedMemoryBase;  
    050h    ULONG           ReadOnlySharedMemoryHeap;  
    054h    ULONG           ReadOnlyStaticServerData;  
    058h    ULONG           AnsiCodePageData;  
    05Ch    ULONG           OemCodePageData;  
    060h    ULONG           UnicodeCaseTableData;  
    064h    ULONG           NumberOfProcessors;  
    068h    LARGE\_INTEGER   NtGlobalFlag;               // Address of a local copy  
    070h    LARGE\_INTEGER   CriticalSectionTimeout;  
    078h    ULONG           HeapSegmentReserve;  
    07Ch    ULONG           HeapSegmentCommit;  
    080h    ULONG           HeapDeCommitTotalFreeThreshold;  
    084h    ULONG           HeapDeCommitFreeBlockThreshold;  
    088h    ULONG           NumberOfHeaps;  
    08Ch    ULONG           MaximumNumberOfHeaps;  
    090h    ULONG           ProcessHeaps;  
    094h    ULONG           GdiSharedHandleTable;  
    098h    ULONG           ProcessStarterHelper;  
    09Ch    ULONG           GdiDCAttributeList;  
    0A0h    KSPIN\_LOCK      LoaderLock;  
    0A4h    ULONG           OSMajorVersion;  
    0A8h    ULONG           OSMinorVersion;  
    0ACh    USHORT          OSBuildNumber;  
    0AEh    USHORT          OSCSDVersion;  
    0B0h    ULONG           OSPlatformId;  
    0B4h    ULONG           ImageSubsystem;  
    0B8h    ULONG           ImageSubsystemMajorVersion;  
    0BCh    ULONG           ImageSubsystemMinorVersion;  
    0C0h    ULONG           ImageProcessAffinityMask;  
    0C4h    ULONG           GdiHandleBuffer\[0x22\];  
    14Ch    ULONG           PostProcessInitRoutine;  
    150h    ULONG           TlsExpansionBitmap;  
    154h    UCHAR           TlsExpansionBitmapBits\[0x80\];  
    1D4h    ULONG           SessionId;  
} PEB, \*PPEB;  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
PEB\[0C\]->PEB\_LDR\_DATA  
  
typedef struct \_PEB\_LDR\_DATA  
{  
　ULONG         Length;                             // 00h  
　BOOLEAN       Initialized;                        // 04h  
　PVOID         SsHandle;                           // 08h  
　LIST\_ENTRY    InLoadOrderModuleList;              // 0ch  
　LIST\_ENTRY    InMemoryOrderModuleList;            // 14h  
　LIST\_ENTRY    InInitializationOrderModuleList;    // 1ch  
}  
    PEB\_LDR\_DATA,  
    \*PPEB\_LDR\_DATA;                                 // 24h  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
PEB\_LDR\_DATA\[\]->LIST\_ENTRY  
  
nt!\_LIST\_ENTRY  
   +0x000   Flink       : Ptr32 \_LIST\_ENTRY     //表示从前往后  
   +0x004   Blink       : Ptr32 \_LIST\_ENTRY     //表示从后往前  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
LIST\_ENTRY\[00\].LDR\_MODULE  
  
typedef struct \_LDR\_MODULE  
{  
    LIST\_ENTRY        InLoadOrderModuleList;            // 00h  
    LIST\_ENTRY        InMemoryOrderModuleList;          // 08h  
    LIST\_ENTRY        InInitializationOrderModuleList;  // 10h      ntdll  
    PVOID             BaseAddress;                      // 18h      kernel32.dll  
    PVOID             EntryPoint;                       // 1ch  
    ULONG             SizeOfImage;                      // 20h  
    UNICODE\_STRING    FullDllName;                      // 24h  
    UNICODE\_STRING    BaseDllName;                      // 2ch  
    ULONG             Flags;                            // 34h  
    SHORT             LoadCount;                        // 38h  
    SHORT             TlsIndex;                         // 3ah  
    HANDLE            SectionHandle;                    // 3ch  
    ULONG             CheckSum;                         // 40h  
    ULONG             TimeDateStamp;                    // 44h  
                                                        // 48h  
} LDR\_MODULE, \*PLDR\_MODULE;  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || TEB1.rar  
 |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || TEB2.rar  
 |