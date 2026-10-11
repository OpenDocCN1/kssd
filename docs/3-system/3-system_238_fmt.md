
# 城里城外看 SSDT

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-645.htm`](https://www.pediy.com/kssd/pediy09/pediy09-645.htm)
>
> **实在是好文,忍不住要转过来分享**
>
> 转自 `http://www.titilima.cn/readarticle.php?id=78`

**城里城外看 SSDT**

2007 年 7 月 5 日作于京师海淀

2006 年，中国互联网上的斗争硝烟弥漫。这时的战场上，先前颇为流行的窗口挂钩、`API` 挂钩、进程注入等技术已然成为昨日黄花，大有逐渐淡出之势；取而代之的，则是更狠毒、更为赤裸裸的词汇：驱动、隐藏进程、`Rootkit`……

在这篇文章里，李马首度从 `ring3`（应用层）的围城跨出，一跃而投身于 `ring0`（内核层）这一更广阔的天地，为你展现 `SSDT`（系统服务描述符表）的奥秘。

点这里下载本文的配套代码

### 引子

2006 年，中国互联网上的斗争硝烟弥漫。这时的战场上，先前颇为流行的窗口挂钩、`API` 挂钩、进程注入等技术已然成为昨日黄花，大有逐渐淡出之势；取而代之的，则是更狠毒、更为赤裸裸的词汇：驱动、隐藏进程、`Rootkit`……

前不久，我不经意翻出自己 2005 年 9 月写下的一篇文章《`DLL` 的远程注入技术》，在下面看到了一位名叫 `L4bm0s` 的网友说这种技术已经过时了。虽然我也曾想过拟出若干辩解之词聊作应对，不过最终还是作罢了毕竟，拿出些新的、有技术含量的东西才是王道。于是这一次，李马首度从 `ring3`（应用层）的围城跨出，一跃而投身于 `ring0`（内核层）这一更广阔的天地，便有了这篇《城里城外看 `SSDT`》。顾名思义，城里和城外的这一墙之隔，就是 `ring3` 与 `ring0` 的分界。

在这篇文章里，我会用到太多杂七杂八的东西，比如汇编，比如内核调试器，比如 `DDK`。这诚然是一件令我瞻前顾后畏首畏尾的事情一方面在 `ring0` 我不得不依靠这些东西，另一方面我实在担心它们会导致我这篇文章的阅读门槛过高。所以，我决定尽可能少地涉及驱动、内核与 `DDK`，也不会对诸如如何使用内核调试器等问题作任何讲解你只需要知道我大概在做些什么，这就足够了。

### 什么是 SSDT？

什么是 `SSDT`？自然，这个是我必须回答的问题。不过在此之前，请你打开命令行（`cmd.exe`）窗口，并输入“`dir`”并回车好了，列出了当前目录下的所有文件和子目录。

那么，以程序员的视角来看，整个过程应该是这样的：

*   由用户输入 `dir` 命令。
*   `cmd.exe` 获取用户输入的 `dir` 命令，在内部调用对应的 `Win32 API` 函数 `FindFirstFile`、`FindNextFile` 和 `FindClose`，获取当前目录下的文件和子目录。
*   `cmd.exe` 将文件名和子目录输出至控制台窗口，也就是返回给用户。

到此为止我们可以看到，`cmd.exe` 扮演了一个非常至关重要的角色，也就是用户与 `Win32 API` 的交互。你大概已经可以猜到，我下面要说到的 `SSDT` 亦必将扮演这个角色，这实在是一点新意都没有。

没错，你猜对了。`SSDT` 的全称是 `System Services Descriptor Table`，系统服务描述符表。这个表就是一个把 `ring3` 的 `Win32 API` 和 `ring0` 的内核 `API` 联系起来的角色，下面我将以 `API` 函数 `OpenProcess` 为例说明这个联系的过程。

你可以用任何反汇编工具来打开你的 `kernel32.dll`，然后你会发现在 `OpenProcess` 中有类似这样的汇编代码：

```
call ds:NtOpenProcess
```

这就是说，`OpenProcess` 调用了 `ntdll.dll` 的 `NtOpenProcess` 函数。那么继续反汇编之，你会发现 `ntdll.dll` 中的这个函数很短：

```
mov eax, 7Ah
mov edx, 7FFE0300h
call dword ptr [edx]
retn 10h
```

另外，`call` 的一句实质是调用了 `KiFastSystemCall`：

```
mov edx, esp
sysenter
```

上面是我的 `XP Professional sp2` 中 `ntdll.dll` 的反汇编结果，如果你用的是 `2000` 系统，那么可能是这个样子：

```
mov eax, 6Ah
lea edx, [esp+4]
int 2Eh
retn 10h
```

虽然它们存在着些许不同，但都可以这么来概括：

*   把一个数放入 `eax`（`XP` 是 `0x7A`，`2000` 是 `0x6A`），这个数值称作系统的服务号。
*   把参数堆栈指针（`esp+4`）放入 `edx`。
*   `sysenter` 或 `int 2Eh`。

好了，你在 `ring3` 能看到的东西就到此为止了。事实上，在 `ntdll.dll` 中的这些函数可以称作真正的 `NT` 系统服务的存根（`Stub`）函数。分隔 `ring3` 与 `ring0` 城里城外的这一道叹息之墙，也正是由它们打通的。接下来 `SSDT` 就要出场了，come some music。

### 站在城墙看城外

插一句先，貌似到现在为止我仍然没有讲出来 `SSDT` 是个什么东西，真正可以算是“犹抱琵琶半遮面”了。书接上文，在你调用 `sysenter` 或 `int 2Eh` 之后，`Windows` 系统将会捕获你的这个调用，然后进入 `ring0` 层，并调用内核服务函数 `NtOpenProcess`，这个过程如下图所示。

![www.titilima.cn/pics/ssdtdump1.gif](img/98ac1903564f1a9a8c1fee807e188241.jpg)

`SSDT` 在这个过程中所扮演的角色是至关重要的。让我们先看一看它的结构，如下图。

![www.titilima.cn/pics/ssdtdump2.gif](img/f9ac9252805f77e0683ea616102a0ebe.jpg)

当程序的处理流程进入 `ring0` 之后，系统会根据服务号（`eax`）在 `SSDT` 这个系统服务描述符表中查找对应的表项，这个找到的表项就是系统服务函数 `NtOpenProcess` 的真正地址。之后，系统会根据这个地址调用相应的系统服务函数，并把结果返回给 `ntdll.dll` 中的 `NtOpenProcess`。图中的“`SSDT`”所示即为系统服务描述符表的各个表项；右侧的“`ntoskrnl.exe`”则为 `Windows` 系统内核服务进程（`ntoskrnl` 即为 `NT OS KerneL` 的缩写），它提供了相对应的各个系统服务函数。`ntoskrnl.exe` 这个文件位于 `Windows` 的 `system32` 目录下，有兴趣的朋友可以反汇编一下。

附带说两点。根据你处理器的不同，系统内核服务进程可能也是不一样的。真正运行于系统上的内核服务进程可能还有 `ntkrnlmp.exe`、`ntkrnlpa.exe` 这样的情况不过为了统一起见，下文仍统称这个进程为 `ntoskrnl.exe`。另外，`SSDT` 中的各个表项也未必会全部指向 `ntoskrnl.exe` 中的服务函数，因为你机器上的杀毒监控或其它驱动程序可能会改写 `SSDT` 中的某些表项这也就是所谓的“挂钩 `SSDT`”以达到它们的“主动防御”式杀毒方式或其它的特定目的。

### KeServiceDescriptorTable

事实上，`SSDT` 并不仅仅只包含一个庞大的地址索引表，它还包含着一些其它有用的信息，诸如地址索引的基地址、服务函数个数等等。`ntoskrnl.exe` 中的一个导出项 `KeServiceDescriptorTable` 即是 `SSDT` 的真身，亦即它在内核中的数据实体。`SSDT` 的数据结构定义如下：

```
typedef struct _tagSSDT {
    PVOID pvSSDTBase;
    PVOID pvServiceCounterTable;
    ULONG ulNumberOfServices;
    PVOID pvParamTableBase;
} SSDT, *PSSDT;
```

其中，`pvSSDTBase` 就是上面所说的“系统服务描述符表”的基地址。`pvServiceCounterTable` 则指向另一个索引表，该表包含了每个服务表项被调用的次数；不过这个值只在 `Checkd Build` 的内核中有效，在 `Free Build` 的内核中，这个值总为 `NULL`（注：`Check/Free` 是 `DDK` 的 `Build` 模式，如果你只使用 `SDK`，可以简单地把它们理解为 `Debug/Release`）。`ulNumberOfServices` 表示当前系统所支持的服务个数。`pvParamTableBase` 指向 `SSPT`（`System Service Parameter Table`，即系统服务参数表），该表格包含了每个服务所需的参数字节数。

下面，让我们开看看这个结构里边到底有什么。打开内核调试器（以 `kd` 为例），输入命令显示 `KeServiceDescriptorTable`，如下。

```
lkd> dd KeServiceDescriptorTable l4
8055ab80 804e3d20 00000000 0000011c 804d9f48
```

接下来，亦可根据基地址与服务总数来查看整个服务表的各项：

```
lkd> dd 804e3d20 l11c
804e3d20 80587691 f84317aa f84317b4 f84317be
804e3d30 f84317c8 f84317d2 f84317dc f84317e6
804e3d40 8057741c f84317fa f8431804 f843180e
804e3d50 f8431818 f8431822 f843182c f8431836
...
```

你获得的结果可能和我会有不同我指的是那堆以十六进制 `f` 开头的地址项，因为我的 `SSDT` 被 `System Safety Monitor` 接管了，没留下几个原生的 `ntoskrnl.exe` 表项。

现在是写些代码的时候了。`KeServiceDescriptorTable` 及 `SSDT` 各个表项的读取只能在 `ring0` 层完成，于是这里我使用了内核驱动并借助 `DeviceIoControl` 来完成。其中 `DeviceIoControl` 的分发代码实现如下面的代码所示，没有什么技术含量，所以不再解释。

```
switch ( IoControlCode )
{
case IOCTL_GETSSDT:
    {
        __try
        {
            ProbeForWrite( OutputBuffer, sizeof( SSDT ), sizeof( ULONG ) );
            RtlCopyMemory( OutputBuffer, KeServiceDescriptorTable, sizeof( SSDT ) );
        }
        __except ( EXCEPTION_EXECUTE_HANDLER )
        {
            IoStatus->Status = GetExceptionCode();
        }
    }
    break;
case IOCTL_GETPROC:
    {
        ULONG uIndex = 0;
        PULONG pBase = NULL;

        __try
        {
            ProbeForRead( InputBuffer, sizeof( ULONG ), sizeof( ULONG ) );
            ProbeForWrite( OutputBuffer, sizeof( ULONG ), sizeof( ULONG ) );
        }
        __except( EXCEPTION_EXECUTE_HANDLER )
        {
            IoStatus->Status = GetExceptionCode();
            break;
        }

        uIndex = *(PULONG)InputBuffer;
        if ( KeServiceDescriptorTable->ulNumberOfServices <= uIndex )
        {
            IoStatus->Status = STATUS_INVALID_PARAMETER;
            break;
        }
        pBase = KeServiceDescriptorTable->pvSSDTBase;
        *((PULONG)OutputBuffer) = *( pBase + uIndex );
    }
    break;
// ...
}
```

补充一下，再。`DDK` 的头文件中有一件很遗憾的事情，那就是其中并未声明 `KeServiceDescriptorTable`，不过我们可以自己手动添加之：

```
extern PSSDT KeServiceDescriptorTable;
```

当然，如果你对 `DDK` 开发实在不感兴趣的话，亦可以直接使用配套代码压缩包中的 `SSDTDump.sys`，并使用 `DeviceIoControl` 发送 `IOCTL_GETSSDT` 和 `IOCTL_GETPROC` 控制码即可；或者，直接调用我为你准备好的两个函数：

```
BOOL GetSSDT( IN HANDLE hDriver, OUT PSSDT buf );
BOOL GetProc( IN HANDLE hDriver, IN ULONG ulIndex, OUT PULONG buf );
```

### 获取详细模块信息

虽然我们现在可以获取任意一个服务号所对应的函数地址了已经，但是你可能仍然不满意，认为只有获得了这个服务函数所在的模块才是王道。换句话说，对于一个干净的 `SSDT` 表来说，它里边的表项应该都是指向 `ntoskrnl.exe` 的；如果 `SSDT` 之中有若干个表项被改写（挂钩），那么我们应该知道是哪一个或哪一些模块替换了这些服务。

首先我们需要获得当前在 `ring0` 层加载了那些模块。如我在本文开头所说，为了尽可能地少涉及 `ring0` 层的东西，于是在这里我使用了 `ntdll.dll` 的 `NtQuerySystemInformation` 函数。关键代码如下：

```
typedef struct _SYSTEM_MODULE_INFORMATION
{
    ULONG Reserved[2];
    PVOID Base;
    ULONG Size;
    ULONG Flags;
    USHORT Index;
    USHORT Unknown;
    USHORT LoadCount;
    USHORT ModuleNameOffset;
    CHAR ImageName[256];
} SYSTEM_MODULE_INFORMATION, *PSYSTEM_MODULE_INFORMATION;

typedef struct _tagSysModuleList {
    ULONG ulCount;
    SYSTEM_MODULE_INFORMATION smi[1];
} SYSMODULELIST, *PSYSMODULELIST;

s = NtQuerySystemInformation( SystemModuleInformation, pRet,
    sizeof( SYSMODULELIST ), &nRetSize );
if ( STATUS_INFO_LENGTH_MISMATCH == s )
{
    // 缓冲区太小，重新分配
    delete pRet;
    pRet = (PSYSMODULELIST)new BYTE[nRetSize];
```



```
s = NtQuerySystemInformation( SystemModuleInformation, pRet,
    nRetSize, &nRetSize );
}
```

需要说明的是，这个函数是利用内核的 `PsLoadedModuleList` 链表来枚举系统模块的，因此如果你遇到了能够隐藏驱动的 Rootkit，那么这种方法是无法找到被隐藏的模块的。在这种情况下，枚举系统的 `\\Driver` 目录对象可能可以更好解决这个问题，在此不再赘述了。

接下来，是根据 `SSDT` 中的地址表项查找模块。有了 `SYSTEM_MODULE_INFORMATION` 结构中的模块基地址与模块大小，这个工作完成起来也很容易：

```
BOOL FindModuleByAddr( IN ULONG ulAddr, IN PSYSMODULELIST pList,
                      OUT LPSTR buf, IN DWORD dwSize )
{
    for ( ULONG i = 0; i < pList->ulCount; ++i )
    {
        ULONG ulBase = (ULONG)pList->smi[i].Base;
        ULONG ulMax  = ulBase + pList->smi[i].Size;
        if ( ulBase <= ulAddr && ulAddr < ulMax )
        {
            // 对于路径信息，截取之
            PCSTR pszModule = strrchr( pList->smi[i].ImageName, '\\' );
            if ( NULL != pszModule )
            {
                lstrcpynA( buf, pszModule + 1, dwSize );
            }
            else
            {
                lstrcpynA( buf, pList->smi[i].ImageName, dwSize );
            }
            return TRUE;
        }
    }
    return FALSE;
}
```

### 详细枚举系统服务项

到现在为止，还遗留有一个问题，就是获得服务号对应的服务函数名。比如 `XP` 下 `0x7A` 对应着 `NtOpenProcess`，但是到 `2000` 下，`NtOpenProcess` 就改为 `0x6A` 了。

有一个好消息一个坏消息，你先听哪个？

什么坏消息？

`Windows` 并没有给我们开放这样现成的函数，所有的工作都需要我们自己来做。

那好消息呢？

牛粪有的是。

坏了，串词儿了。好消息是我们可以通过枚举 `ntdll.dll` 的导出函数来间接枚举 `SSDT` 所有表项所对应的函数，因为所有的内核服务函数对应于 `ntdll.dll` 的同名函数都是这样开头的：

```
mov eax, <ServiceIndex>
```

对应的机器码为：

```
B8 <ServiceIndex>
```

再说一遍：非常幸运，仅就我手头上的 `2000 sp4`、`XP`、`XP sp1`、`XP sp2`、`2003` 的 `ntdll.dll` 而言，无一例外。不过 `Mark Russinovich` 的《深入解析 Windows 操作系统》一书中指出，`IA64` 的调用方式与此不同由于手头上没有相应的文件，所以在这里不进行讨论了。

接着说。我们可以把 `mov` 的一句用如下的一个结构来表示：

```
#pragma pack( push, 1 )
typedef struct _tagSSDTEntry {
    BYTE  byMov;   // 0xb8
    DWORD dwIndex;
} SSDTENTRY;
#pragma pack( pop )
```

那么，我们可以对 `ntdll.dll` 的所有导出函数进行枚举，并筛选出“`Nt`”开头者，以 `SSDTENTRY` 的结构取出其开头 5 个字节进行比对这就是整个的枚举过程。相关的 `PE` 文件格式解析我不再解释，可参考注释。整个代码如下：

```
#define MOV        0xb8

void EnumSSDT( IN HANDLE hDriver, IN HMODULE hNtDll )
{
    DWORD dwOffset                = (DWORD)hNtDll;
    PIMAGE_EXPORT_DIRECTORY pExpDir = NULL;
    int nNameCnt                  = 0;
    LPDWORD pNameArray           = NULL;
    int i                         = 0;

    // 到 PE 头部
    dwOffset += ((PIMAGE_DOS_HEADER)hNtDll)->e_lfanew + sizeof( DWORD );
    // 到第一个数据目录
    dwOffset += sizeof( IMAGE_FILE_HEADER ) + sizeof( IMAGE_OPTIONAL_HEADER )
        - IMAGE_NUMBEROF_DIRECTORY_ENTRIES * sizeof( IMAGE_DATA_DIRECTORY );
    // 到导出表位置
    dwOffset = (DWORD)hNtDll
        + ((PIMAGE_DATA_DIRECTORY)dwOffset)->VirtualAddress;
    pExpDir = (PIMAGE_EXPORT_DIRECTORY)dwOffset;

    nNameCnt = pExpDir->NumberOfNames;
    // 到函数名 RVA 数组
    pNameArray = (LPDWORD)( (DWORD)hNtDll + pExpDir->AddressOfNames );

    // 初始化系统模块链表
    PSYSMODULELIST pList = CreateModuleList( hNtDll );

    // 循环查找函数名
    for ( i = 0; i < nNameCnt; ++i )
    {
        PCSTR pszName = (PCSTR)( pNameArray[i] + (DWORD)hNtDll );
        if ( 'N' == pszName[0] && 't' == pszName[1] )
        {
            // 找到了函数，则定位至查找表
            LPWORD pOrdNameArray = (LPWORD)( (DWORD)hNtDll + pExpDir->AddressOfNameOrdinals );
            // 定位至总表
            LPDWORD pFuncArray   = (LPDWORD)( (DWORD)hNtDll + pExpDir->AddressOfFunctions );
            LPCVOID pFunc        = (LPCVOID)( (DWORD)hNtDll + pFuncArray[pOrdNameArray[i]] );
            
            // 解析函数，获取服务名
            SSDTENTRY entry;
            CopyMemory( &entry, pFunc, sizeof( SSDTENTRY ) );
            if ( MOV == entry.byMov )
            {
                ULONG ulAddr = 0;
                GetProc( hDriver, entry.dwIndex, &ulAddr );
                
                CHAR strModule[MAX_PATH] = "[Unknown Module]";
                FindModuleByAddr( ulAddr, pList, strModule, MAX_PATH );
                printf( "0x%04X\t%s\t0x%08X\t%s\r\n", entry.dwIndex,
                    strModule, ulAddr, pszName );
            }
        }
    }

    DestroyModuleList( pList );
}
```

下图是示例程序 `SSDTDump` 在 `XP sp2` 上的部分运行截图，显示了 `SSDT` 的基地址、服务个数，以及各个表项所对应的服务号、所在模块、地址和服务名。

![](img/42ad707aa0e2d29accbdad25787d9426.jpg)

### 结语

`ring3` 与 `ring0`，城里与城外之间为一道叹息之墙所间隔，`SSDT` 则是越过此墙的一道必经之门。因此，很多杀毒软件也势必会围绕着它大做文章。无论是 `System Safety Monitor` 的系统监控，还是卡巴斯基的主动防御，都是挂钩了 `SSDT`。这样，病毒尚在 `ring3` 内发作之时，便被扼杀于摇篮之内。

内核最高权限，本就是兵家必争之地，魔高一尺道高一丈的争夺于此亦已变成颇为稀松平常之事。可以说和这些争夺比起来，`SSDT` 的相关技术简直不值一提。但最初发作的病毒体总是从 `ring3` 开始的换句话说，任你未来会成长为何等的武林高手，我都可以在你学走路的时候杀掉你知晓了 `SSDT` 的这点优势，所有的病毒咂吧咂吧也就都没味儿了。所以说么，杀毒莫如防毒。

就此打住罢，貌似扯远大发了。
