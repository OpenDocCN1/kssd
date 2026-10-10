

# anti trick Ⅲ －－erase module

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-657.htm`](https://www.pediy.com/kssd/pediy07/pediy7-657.htm)

;anti trick Ⅲ －－erase module  
;by 来自轻院的狼\[immlep\]  
;www.ptteam.com  
;http://immlep.blogone.net   
  
一个偶然的机会，在学习 PEB 的时候，发现擦除 EXE 文件的模块的话，ImpREC 会找不到正确的输入表，因为它找不到那个 DLL 文件，API 监视器无法正确监视 API 函数，因为它找不到那个 DLL 文件，当然这只不过是这些 API 监视器没有用到比较高深的东西，比如 native api，所以它们用的信息都是从 PEB 来的。所以。。  
  
PEB 的更详细结构可以查询 ntundoc.chm，或 baidu 查找。。。  
PEB 的结构：  
kd> !strct PEB  
!strct PEB  
struct \_PEB (sizeof=488)  
+000 byte InheritedAddressSpace  
+001 byte ReadImageFileExecOptions  
+002 byte BeingDebugged  
+003 byte SpareBool  
+004 void \*Mutant  
+008 void \*ImageBaseAddress  
+00c struct \_PEB\_LDR\_DATA \*Ldr  
+010 struct \_RTL\_USER\_PROCESS\_PARAMETERS \*ProcessParameters  
+014 void \*SubSystemData  
+018 void \*ProcessHeap  
+01c void \*FastPebLock  
+020 void \*FastPebLockRoutine  
+024 void \*FastPebUnlockRoutine  
+028 uint32 EnvironmentUpdateCount  
+02c void \*KernelCallbackTable  
+030 uint32 SystemReserved\[2\]  
+038 struct \_PEB\_FREE\_BLOCK \*FreeList  
+03c uint32 TlsExpansionCounter  
+040 void \*TlsBitmap  
+044 uint32 TlsBitmapBits\[2\]  
+04c void \*ReadOnlySharedMemoryBase  
+050 void \*ReadOnlySharedMemoryHeap  
+054 void \*\*ReadOnlyStaticServerData  
+058 void \*AnsiCodePageData  
+05c void \*OemCodePageData  
+060 void \*UnicodeCaseTableData  
+064 uint32 NumberOfProcessors  
+068 uint32 NtGlobalFlag  
+070 union \_LARGE\_INTEGER CriticalSectionTimeout  
+070 uint32 LowPart  
+074 int32 HighPart  
+070 struct \_\_unnamed3 u  
+070 uint32 LowPart  
+074 int32 HighPart  
+070 int64 QuadPart  
+078 uint32 HeapSegmentReserve  
+07c uint32 HeapSegmentCommit  
+080 uint32 HeapDeCommitTotalFreeThreshold  
+084 uint32 HeapDeCommitFreeBlockThreshold  
+088 uint32 NumberOfHeaps  
+08c uint32 MaximumNumberOfHeaps  
+090 void \*\*ProcessHeaps  
+094 void \*GdiSharedHandleTable  
+098 void \*ProcessStarterHelper  
+09c uint32 GdiDCAttributeList  
+0a0 void \*LoaderLock  
+0a4 uint32 OSMajorVersion  
+0a8 uint32 OSMinorVersion  
+0ac uint16 OSBuildNumber  
+0ae uint16 OSCSDVersion  
+0b0 uint32 OSPlatformId  
+0b4 uint32 ImageSubsystem  
+0b8 uint32 ImageSubsystemMajorVersion  
+0bc uint32 ImageSubsystemMinorVersion  
+0c0 uint32 ImageProcessAffinityMask  
+0c4 uint32 GdiHandleBuffer\[34\]  
+14c function \*PostProcessInitRoutine  
+150 void \*TlsExpansionBitmap  
+154 uint32 TlsExpansionBitmapBits\[32\]  
+1d4 uint32 SessionId  
+1d8 void \*AppCompatInfo  
+1dc struct \_UNICODE\_STRING CSDVersion  
+1dc uint16 Length  
+1de uint16 MaximumLength  
+1e0 uint16 \*Buffer  
  
\_PEB\_LDR\_DATA \*Ldr 的结构：  
kd> !strct PEB\_LDR\_DATA  
!strct PEB\_LDR\_DATA  
struct \_PEB\_LDR\_DATA (sizeof=36)  
+00 uint32 Length  
+04 byte Initialized  
+08 void \*SsHandle  
+0c struct \_LIST\_ENTRY InLoadOrderModuleList      ;存放着模块信息  
+0c struct \_LIST\_ENTRY \*Flink                     ;前一个链表成员地址  
+10 struct \_LIST\_ENTRY \*Blink                     ;后一个链表成员地址  
+14 struct \_LIST\_ENTRY InMemoryOrderModuleList  
+14 struct \_LIST\_ENTRY \*Flink  
+18 struct \_LIST\_ENTRY \*Blink  
+1c struct \_LIST\_ENTRY InInitializationOrderModuleList  
+1c struct \_LIST\_ENTRY \*Flink  
+20 struct \_LIST\_ENTRY \*Blink  
  
typedef struct \_LDR\_MODULE {  
  
  LIST\_ENTRY              InLoadOrderModuleList;  
  LIST\_ENTRY              InMemoryOrderModuleList;  
  LIST\_ENTRY              InInitializationOrderModuleList;  
  PVOID                   BaseAddress;  
  PVOID                   EntryPoint;  
  ULONG                   SizeOfImage;  
  UNICODE\_STRING          FullDllName;  
  UNICODE\_STRING          BaseDllName;  
  ULONG                   Flags;  
  SHORT                   LoadCount;  
  SHORT                   TlsIndex;  
  LIST\_ENTRY              HashTableEntry;  
  ULONG                   TimeDateStamp;  
  
} LDR\_MODULE, \*PLDR\_MODULE;  
  
  
代码如下，注意擦除模块有时可能会出现一些问题。。。。。请自己揣摩使用。。  
  
    mov eax, fs:\[30h\]                  ;Peb  
    mov eax, \[eax+0Ch\]                    ;Ldr  
    mov eax, \[eax+0Ch\]                    ;InLoadOrderModuleList  
    mov \[ebp+offset pfirstmod\], eax  
continue:       
    mov ecx, \[eax+30h\]                ; BaseDllName   模块名                         
    push eax              
    call @F  
    db 6Bh,00h,65h,00h,72h,00h,6Eh,00h,65h,00h,6Ch,00h,33h,00h,32h,00h,2Eh,00h,64h,00h,6Ch,00h,6Ch,00h,00h,00h   
    ;kernal32.dll  
        ;这里是 unicode 字符串，你也可以写 ascii 然后用 MultiByteToWideChar 转为字符串，或者用 Four-F 的$CTW0  
    @@:      
    push ecx                                          
    call \[ebp+\_lstrcmpiW\]                           ;比较是不是我们要擦除的模块，可以添加代码比较擦除多个模块。            
    ;invoke lstrcmpiW, ecx, addr modn                                       
    test eax, eax  
    pop eax  
    je wipemod                                      ;是就把它擦了。。  
    mov eax, \[eax\]                      
    cmp eax, \[ebp+offset pfirstmod\]  
    jne continue                                    ;继续取下一个模块。  
    jmp @F  
wipemod:  
        push eax                                ;这段代码把链表中一个成员擦除，也就时要擦除的那个模块的信息。  
    mov eax, \[eax+4\]  
    mov ebx, \[eax\]                ; Flink        要擦除  
    mov ebx, \[ebx\]  
    mov dword ptr \[eax\], ebx      
    mov ebx, \[eax+4\]            ; Blink  
    mov ebx, \[ebx+4\]  
    mov dword ptr \[eax+4\], ebx      
    pop eax  
    add eax, 8                                    
    mov eax, \[eax+4\]  
    mov ebx, \[eax\]                ; Flink  
    mov ebx, \[ebx\]  
    mov dword ptr \[eax\], ebx      
    mov ebx, \[eax+4\]            ; Blink  
    mov ebx, \[ebx+4\]  
    mov dword ptr \[eax+4\], ebx     
    @@:                              
        ;;;;;;查完所有的模块，跳到这里收工。。。  
附加是我把代码加到 yoda 里面去的了。。  
  
附件:antiod&im.rar  

> 引用:
> 
> * * *
> 
> *最初由 linestyle 发布*  
> **这个现象我在脱最新版的 Armadillo(最新的 Visual Assist)时候遇到过,XP 下 ImpREC 会找不到想要修复资源的 DLL 名字.后来我也没有具体验证到底是,Armadillo 是修改了楼上兄弟说的 PEB,还是过滤了 LIST 的显示名字,呵呵  
>   
> 不过我感觉的我这个问题有点奇怪,在 XP 他会隐藏那个要修复的 DLL.2000 下,就可以正常显示,所以我怀疑 Armadillo 就是修改了 PEB.  
>   
> 所以,楼上哥们你的方法在 2000 测试的还是 XP??我怀疑你的方法可能不同平台不通用. 猜测,别见怪:) **
> 
> * * *

  
  
谢谢楼上，有问题大家讨论![](img/710b5589904d0d2bf58a67748fca40e1.jpg) ![](img/710b5589904d0d2bf58a67748fca40e1.jpg) 没有看过最新版的 Armadillo，不知道是不是也用这个。。。不过这个方法在 2k 和 XP 下 test 过，没什么问题。。 ![](img/710b5589904d0d2bf58a67748fca40e1.jpg)

加点东西：  
typedef struct \_LDR\_MODULE  
{  
    LIST\_ENTRY        InLoadOrderModuleList;            // +0x00  
    LIST\_ENTRY        InMemoryOrderModuleList;          // +0x08  
    LIST\_ENTRY        InInitializationOrderModuleList;  // +0x10  
    PVOID             BaseAddress;                      // +0x18  
    PVOID             EntryPoint;                       // +0x1c  
    ULONG             SizeOfImage;                      // +0x20  
    UNICODE\_STRING    FullDllName;                      // +0x24  
    UNICODE\_STRING    BaseDllName;                      // +0x2c  
    ULONG             Flags;                            // +0x34  
    SHORT             LoadCount;                        // +0x38  
    SHORT             TlsIndex;                         // +0x3a  
    LIST\_ENTRY        HashTableEntry;                   // +0x3c  
    ULONG             TimeDateStamp;                    // +0x44  
                                                        // +0x48  
} LDR\_MODULE, \*PLDR\_MODULE; 

 4dd466d0e1e0cc123afbe504da2462ca