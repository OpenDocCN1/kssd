
因为 x64 位中 SSDT 表是加密的，SSDT 中的每一项占 4 个字节，但并不是对应系统服务的地址。因为 x64 中地址为 64 位，而 SSDT 每一项只有 4 个字节（32 位），所以无法直接存放服务地址。

其实际存储的 4 个字节中，前 28 位表示对应系统服务相对于 SSDT 表基地址的偏移，后 4 位则表示参数个数：如果对应服务的参数个数小于 4，则其值为 0；如果不小于 4，则其值为参数个数减去 4。

所以我们在进行 SSDT Hook 时，向 SSDT 表项中填入的函数必须位于 `ntoskrnl.exe` 模块中，原因是函数到 SSDT 表基地址的偏移大小必须小于 4 个字节。

因此，我们选取 `ntoskrnl.exe` 中很少使用的函数 `KeBugCheckEx` 作为中转函数，将需要 Hook 的 SSDT 表项修改为 `KeBugCheckEx` 函数，然后对 `KeBugCheck` 进行 Inline Hook，跳转到我们的函数中进行过滤。

## 2.2 MSR 寄存器

MSR（Model Specific Register）是 x86 架构中的概念，指的是在 x86 架构处理器中，用于控制 CPU 运行、功能开关、调试、跟踪程序执行、监测 CPU 性能等方面的一系列寄存器。

MSR 寄存器的雏形开始于 Intel 80386 和 80486 处理器。到 Intel Pentium 处理器时，Intel 正式引入 `RDMSR` 和 `WRMSR` 两个指令，用于读写 MSR 寄存器。此时 MSR 才算被正式引入。

在引入 `RDMSR` 和 `WRMSR` 指令的同时，也引入了 `CPUID` 指令。该指令用于指明具体 CPU 芯片中哪些功能可用，或者这些功能对应的 MSR 寄存器是否存在。软件可以通过 `CPUID` 指令查询某些功能是否受当前 CPU 支持。

去 AMD 的官网，下载 AMD 芯片手册：

```text
193K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2L8h3c8Q4x3X3g2U0L8$3#2Q4x3V1k6J5k6i4y4G2N6i4u0U0k6i4y4Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3c8Y4N6h3W2V1k6i4y4Q4x3X3c8E0j5h3&6#2j5h3I4K6i4K6u0r3
```

之后，查找 `MSRC000_0082`，如下图：

![](https://bbs.kanxue.com/upload/attach/202202/945611_GBZHN3CQ8CESFS8.png)

可见，`C000_0082` 是 `SYSCALL_Target_Address`。在 WinDbg 中，我们看看这个地址是什么。

```text
5: kd> rdmsr c0000082
msr[c0000082] = fffff800`0408eec0
5: kd> uf fffff800`0408eec0
Flow analysis was incomplete, some code may be missing
nt!KiSystemCall64:
fffff800`0408eec0 0f01f8          swapgs
fffff800`0408eec3 654889242510000000 mov qword ptr gs:[10h],rsp
fffff800`0408eecc 65488b2425a8010000 mov rsp,qword ptr gs:[1A8h]
。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。。
fffff800`0408eff2 4c8d1547a92300 lea r10,[nt!KeServiceDescriptorTable (fffff800`042c9940)]
fffff800`0408eff9 4c8d1d80a92300 lea r11,[nt!KeServiceDescriptorTableShadow (fffff800`042c9980)]
fffff800`0408f000 f7830001000080000000 test dword ptr [rbx+100h],80h
```

`KeServiceDescriptorTableShadow` 就是 SSDT 表的地址。记住这里，后面代码中要用到。

## 2.3 跳转指令

x64 下面常用的跳转指令如下。

### 1. `mov rax, addr; jmp rax`

```text
48 B8 qword_addr -> mov rax, qword_addr
FF E0 -> jmp rax
```

### 2. `push / ret`

```text
68 dword_addr -> push dword_addr
C3 -> ret
```

注意：`push` 的立即数是 `dword`，但由于对齐关系，实际占用 64 位。如果跳转地址超过 2GB，要使用其他指令。`push` 的立即数不能是 64 位。

### 3. `jmp`、`call`

```text
FF15 dword_offset qword_addr -> call [qword_addr]
FF25 dword_offset qword_addr -> jmp [qword_addr]
```

简单说明如下：

在下条指令偏移 `dword_offset` 处，是要跳转的地址 `qword_addr`。当 `dword_offset=0x00000000` 时，`qword_addr` 就在指令的第 6 个字节。

但是在 x86 下：

```text
FF15 dword_addr -> call [dword_addr]
FF25 dword_addr -> jmp [dword_addr]
```

没有偏移，后面直接跟着绝对地址。

## 2.4 Shellcode 执行出现 `movaps` 错误

我在写 Shellcode 的时候出现了 `movaps` 错误，一直以为是代码有问题。后来经过排查和搜索，发现问题是：在 Shellcode 中调用函数之前，要保证 `rsp` 是 16 字节对齐的。

我在这里耽误了一些时间，你在使用时尤其要注意。

## 2.5 为什么要自己实现 `GetModuleHandle` 和 `GetProcAddress`

首先要明白，在 `win7_sp1_x64` 下，`CreateProcess` 的创建流程如下：

| 步骤 | 调用函数 |
| --- | --- |
| 1 | `Kernel32!CreateProcess` |
| 2 | `Kernel32!CreateProcessW` |
| 3 | `Kernel32!CreateProcessInternalW` |
| 4 | `ntdll!NtCreateProcessEx` |
| 5 | `ntdll!NtCreateThread` |
| 6 | `ntdll!NtResumeThread` |

此时，主进程的大部分创建工作已经完成，尤其是 `ntdll.dll` 已经加载，这一点非常重要。主线程刚刚被创建，此时还没有加载除 `ntdll.dll` 之外的其他 DLL。

而 `GetModuleHandle` 和 `GetProcAddress` 函数位于 `kernel32.dll` 中，所以需要自己实现这两个函数，以获取 DLL 的句柄和函数地址。`kernel32.dll` 会在进程创建完成、主线程初始化输入表时才载入。

## 2.6 实现 `GetModuleHandle` 和 `GetProcAddress` 的概略思路

### 2.6.1 实现 `GetModuleHandle`

`GetModuleHandle` 用于获取 DLL 基址。现在要获取 `ntdll.dll` 基址，可以通过暴力搜索内存实现。

主要用到的两个函数是：

- `ZwQueryVirtualMemory`
- `ZwQuerySystemInformation`

其中，`ZwQuerySystemInformation` 用到的类型是 `SystemEmulationBasicInformation`。这个参数很重要，我找了很多资料才找到。

`ZwQueryVirtualMemory` 用到的类型是 `MemoryBasicInformation`，以及另外一个未文档化的参数 `MEMORY_SECTION_NAME`。

此时要注意，x64 和 x86 获取系统信息的方式稍有不同：

- x86：`ZwQuerySystemInformation` 的第一个参数是 `SystemBasicInformation`。

### 2.6.2 实现 `GetProcAddress`

导出表有一个结构 `IMAGE_EXPORT_DIRECTORY`，主要注意其中的 4 个字段：

1. `AddressOfNames`
2. `AddressOfNameOrdinals`
3. `AddressOfFunctions`
4. `NumberOfNames`

![](https://bbs.kanxue.com/upload/attach/202202/945611_2KQRNEVE95Q6FS9.png)

代码实现的基本思路是：

循环 `NumberOfNames` 次，在 `AddressOfName` 地址中查找 `targetFunc`。找到函数时，记住循环次数 `i`，通过 `AddressOfNameOrdinals` 找到函数索引号，例如为 `b`，最后函数地址为 `AddressOfFunctions[b]`。

代码基本流程如下：

```c
ImageExportDirectory = NtHeader.OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_EXPORT].VirtualAddress;
```

```c
AddressOfFunctions = MappingBaseAddress + ImageExportDirectory->AddressOfFunctions;
AddressOfNames = MappingBaseAddress + ImageExportDirectory->AddressOfNames;
AddressOfNameOrdinals = MappingBaseAddress + ImageExportDirectory->AddressOfNameOrdinals;
```

通过循环 `ImageExportDirectory->NumberOfNames` 次，比较名字是否和 `NtResumeThread` 相等。不相等则继续循环；相等时，通过 `AddressOfNameOrdinals[i]` 得到函数索引 `FunctionOrdinal`，然后函数地址就等于：

```c
FunctionAddress = (PVOID)((UINT8*)MappingBaseAddress + AddressOfFunctions[FunctionOrdinal]);
```

这里的实现是为了得到 `ntdll!ldrLoadDLL` 函数地址。

## 2.7 需要干掉 PageGuard

# 2. 代码编写核心思路

Hook 函数 `nt!NtResumeThread` 的 SSDT 表项，将其修改为 `KeBugCheckEx` 函数地址，然后 Inline Hook 到 `FakeResumeThead` 函数去执行，最后执行 Shellcode。

# 3. 代码编写详细思路

## 3.1 获取 SSDT 表地址

```text
kd> rdmsr c0000082
msr[c0000082] = ffff800`04038bc0
kd> uf ffff800`04038bc0
nt!KiSystemCall64:
......
nt!KiSystemServiceRepeat:
fffff800`03f07a72 4c8d1587be1f00 lea r10,[nt!KeServiceDescriptorTable (fffff800`040be940)]
fffff800`03f07a79 4c8d1d40bf1f00 lea r11,[nt!KeServiceDescriptorTableShadow (fffff800`040be980)]
```

通过 `PUCHAR StartSearchAddress = (PUCHAR)__readmsr(0xC0000082);` 作为起始地址，然后在 `PAGE_SIZE` 范围内搜索特征码（`4c\8d\15`），再将地址加上偏移，得到 SSDT 表地址。

## 3.2 获取 `NtResumeThread` 函数在 SSDT 表中的索引

在 `win7_sp1_x64` 系统中，反汇编 `ntdll` 可知：

![](https://bbs.kanxue.com/upload/attach/202202/945611_6H8QWTYCR7ZNWHF.png)

从函数地址偏移 4 个或 1 个字节，可以得到此函数在 SSDT 表中的索引。

## 3.3 计算原始函数地址

原始函数地址 = SSDT 基址 + 函数索引号得到的偏移地址 `>>4`。

```c
__OldNtResumeThreadAddress =
    SSDTAddress->ServiceTableBase +
    (SSDTAddress->ServiceTableBase[_FunctionIndexInSSDT] >> 4);
```

## 3.4 Hook SSDT 表

A、通过 `__readcr0` 寄存器关闭写保护。

B、Inline Hook：保存 `KeBugCheck` 前面的 14 个字节，然后将其修改为跳转到 Shellcode 执行。跳转汇编如下：

```asm
"\xFF\x25\x00\x00\x00\x00\xFF\xFF\xFF\xFF\xFF\xFF\xFF\xFF";
```

C、计算 `KeBugCheck` 在 SSDT 表中基于 `ServerTableBase` 的偏移值，然后填入之前 `NtResumeThread` 的索引位置。

这样，执行 `NtResumeThread` 函数时，实际会先执行 `KeBugCheck` 函数。又由于 `KeBugCheck` 已经被 Hook，因此会在 `KeBugCheck` 函数中执行 Shellcode。

## 3.5 关于 `FakeNtResumeThread`

在这个函数里，我们主要做的事情如下：

1. 通过 `PsGetProcessImageFileName` 函数，获取要注入进程的名字。
2. 暴力搜索内存，获得 `ntdll.dll` 的基址，通过导出表获取 `ldrLoadDLL` 地址。
3. 通过 `KeStackAttachProcess` 函数附加到进程。
4. 通过 `PsGetContextThread` 函数获取进程的 `rip`。
5. 通过 `PsSetContextThread` 函数修改进程的 `rip`。

## 3.6 关于 Shellcode

Shellcode 结构体如下：

```c
typedef struct _INJECT_DATA {
    CHAR ShellCode[0xA0];
    /* offset = 0xA0 */ PWCHAR PathToFile;        // LdrLoadDll 的第一个参数
    /* offset = 0xA8 */ ULONG64 DllCharacteristics;
    /* offset = 0xB0 */ PUNICODE_STRING pDllPath; // PUNICODE_STRING DllPath
    /* offset = 0xB8 */ PHANDLE ModuleHandle;    // DLL 句柄
    /* offset = 0xC0 */ ULONG64 AddrOfLdrLoadDll; // LdrLoadDll 地址
    /* offset = 0xC8 */ ULONG64 OriginalEIP;      // 原线程的 EIP
    /* offset = 0xD0 */ UNICODE_STRING usDllPath;
    /* offset = 0xE0 */ WCHAR wDllPath[256];      // DLL 路径，也就是 usDllPath 中的 Buffer
} INJECT_DATA;
```

汇编代码如下：
