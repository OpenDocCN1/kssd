
# 内核态进程管理器 Intercessor 和实现细节

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-262.htm`](https://www.pediy.com/kssd/pediy09/pediy09-262.htm)

> **BLOG 原文：** `http://www.csksoft.net/blog/post/Intercessor_taskmgr.html`
> **相关文件：**
> **已经修改下载地址**
> 下载地址：`http://www.csksoft.net/Intercessor_report_src_bin.rar`
> 由于文件太大，无法作为附件提供，抱歉

原理和核心驱动代码下载：`ftp://FTP_Visitor:visitor@ftp.csksoft.net/Public/Products/APP/Intercessor_report_src_bin.rar`

---

上学期 windows 高级操作系统作的大作业，主要对于枚举进程采用直接扫描 `EPROCESS` 的办法，也用到了 Hook SSDT 技术。所以有一定参考价值，同时工具本身也有其实用价值。

先介绍基本情况：

软件画面：

![`www.csksoft.net/BlogData/pic/intercessor_main.png`](img/7b4b82da388c2e991cde438aec8c4ac8.jpg)

下表列出了本软件中已实现的功能：

*   显示进程模块
*   `CPU` 使用率统计
*   内核进程描述
*   内核态下实现进程的监控
*   禁止系统运行某个可执行文件
*   进程创建或者退出自动刷新进程列表
*   强制进程结束，通过直接调用 `NtTerminateProcess` 实现
*   防止自身非法关闭或调试
*   搜索隐藏进程，可以探测多种隐藏技术
*   显示进程 `EPROCESS` 地址
*   显示当前进程内存与虚拟内存
*   显示与设置进程优先级别
*   显示当前进程的版本信息以及详细属性
*   蓝色突出显示隐藏进程

下面是简要的一些原理分析，对于具体的描述，参见原理和核心驱动代码下载给出的文档，里面有我当时写的报告，那个比较详细。

当然其中很多技术不是我创造的，所以先向各位前辈们致谢！

## 直接内存搜索枚举内存的原理和实现

在 `5.x` 的内核中，进程的 `EPROCESS` 往往存放于 `MmSystemRangeStart` 至 `System` 所属 `EPROCESS` 地址之间。

其中 `MmSystemRangeStart` 是一个内核导出的常量，在 `32` 位标准内存模式下，它的值为 `0x80000000`，而在 `PAE` 模式的系统中，这个值为 `0xC0000000`。对于 `System` 所属 `EPROCESS` 地址，可以通过驱动加载时 `PsGetCurrentProcess()` 获取，因为驱动的加载任务是在 `system` 进程中完成的。

在上述区间的具体取值确定下来后，接着就是要确定是否某一块内存片断是一个有效的 `EPROCESS` 结构。

在此首先需要确定 `EPROCESS` 在当前系统的表示。可以使用 `WinDbg` 的如下命令获取：

```
lkd> dt _eprocess
```

下面摘录 `winXP` 环境中 `EPROCESS` 的定义片断：

```
nt!_EPROCESS
  +0x000 Pcb           : _KPROCESS
  +0x06c ProcessLock   : _EX_PUSH_LOCK
  +0x070 CreateTime    : _LARGE_INTEGER
  +0x078 ExitTime      : _LARGE_INTEGER
  +0x080 RundownProtect: _EX_RUNDOWN_REF
  +0x084 UniqueProcessId: Ptr32 Void
  +0x088 ActiveProcessLinks: _LIST_ENTRY
  +0x090 QuotaUsage    : [3] Uint4B
  +0x09c QuotaPeak     : [3] Uint4B
//…
```
结构片断：`EPROCESS` 的定义

这个结构在 `WinXP` 中的大小是 `0x25C`，可以从上面的代码片断中看出，许多信息，比如创建等都可以直接在这个结构体中获取。

为了验证一个 `EPROCESS` 是否合法，需要考虑下面几个字段的数据：

```
+0x078 ExitTime      : _LARGE_INTEGER
+0x1b0 Peb           : Ptr32 _PEB
```

中，`ExitTime` 记录了该进程的退出时间。对于正在运行的进程，该项属性将永远为 `0`。`Peb` 指向进程环境块的指针，在所有 `EPROCESS` 中，该数据区的高 `16` 位应该相同。

除了这 `2` 项判断依据外，还可以根据包装 `EPROCESS` 的 `OBJECT_HEADER` 结构来作出判断。`OBJECT_HEADER` 结构的定义如下：

```
nt!_OBJECT_HEADER
  +0x000 PointerCount    : Int4B
  +0x004 HandleCount     : Int4B
  +0x004 NextToFree      : Ptr32 Void
  +0x008 Type            : Ptr32 _OBJECT_TYPE
  +0x00c NameInfoOffset  : UChar
  +0x00d HandleInfoOffset: UChar
  +0x00e QuotaInfoOffset : UChar
  +0x00f Flags           : UChar
  +0x010 ObjectCreateInfo: Ptr32 _OBJECT_CREATE_INFORMATION
  +0x010 QuotaBlockCharged: Ptr32 Void
  +0x014 SecurityDescriptor: Ptr32 Void
  +0x018 Body            : _QUAD
```
结构定义：`OBJECT_HEADER`

这个结构表示了一个内核对象，他出现于任何内核对象结构的头部，也包括了 `EPROCESS` 头部。其中我们感兴趣的数据域是：

```
+0x008 Type            : Ptr32 _OBJECT_TYPE
```

该指针指向了一个标示对象类别的结构体，所有 `EPROCESS` 结构外层的 `OBJECT_HEADER` 结构中，这项属性值应该相同。

通过上述分析，通过搜索内存枚举进程的流程如下：

1.  从 `MmSystemRangeStart` 至 `SYSTEM` 进程 `EPROCESS` 地址依次搜索
2.  将当前地址赋值给一个 `EPROCESS` 对象，将其中相关的数据域加以验证
3.  如果是合法 `EPROCESS`，则将其地址填入结果，同时控制搜索从当前位置+`sizeof(EPROCESS)` 开始
4.  继续下一次循环

以上方法还需要做的处理是：

*   获取一个标准的 `PEB` 指针，作为对照
*   获取一个标准的 `OBJECT_TYPE` 指针，作为对照
*   由于 `MmSystemRangeStart` 开始的内存并非连续的未分页内存，也就是说其地址空间并不是连续的，因而直接循环扫描将出现缺页错误，导致系统蓝屏崩溃。

由于目前 `system` 的 `eprocess` 地址是已知的，很自然的想法便是通过 `system eprocess` 中的数据作为对照数据。其中，`OBJECT_TYPE` 指针的确可以如此获取，然后对于 `PEB` 指针，在 `system eprocess` 中却为空值。

解决办法是通过 `eprocess` 中的 `ActiveProcessLinks` 数据域得到由 `system eprocess` 指向的第一个 `eprocess` 地址，然后获取其中的 `PEB` 数据。

对于第三个问题，解决办法是通过查找页表方式，尝试对当前虚拟地址查找其对应的 `PGDE` 和 `PTE` 表，如果找到，则表示该内存地址有效。如果无效，则跳过一定的长度搜索。

对于标准的内存模式，`PGDE` 和 `PTE` 如下图所示：

![](img/cd55c84329ea4f27d2aaf8e3f3c96392.jpg)

图：标准内存模式下页表分布

图中的 `Page Directory` 表存放于虚拟地址 `0xc0300000` 处，而每个 `Page Table` 紧跟于 `Page Directory` 表后存储。因而可以通过上述布局查找页表。

然而对于 `PAE` 模式，页表的表示方式以及存储方位均不同于标准模式，因而需要用其他方法获取，具体见下一节。

在解决上述问题后，进程枚举便可以工作了，但通过该算法仅能获得除了 `SYSTEM` 和 `IDLE` 外的进程，对于 `PID` 为 `0` 和 `4`（`win2000` 中为 `8`）的这 `2` 个核心进程，需要用其他方法获取。

对于 `SYSTEM` 进程，上文给出获取方法，而 `Idle Process` 进程需要用下面给出的办法：

获取 `Idle Process` 的 `EPROCESS` 地址

在 `5.x` 内核虚拟地址 `0xFFDFF000` 位置存放的称为 `Processor Control Region(KPCR)` 的结构，通过 `WinDbg` 获取他的结构描述为：

```
nt!_KPCR
  +0x000 NtTib          : _NT_TIB
  +0x01c SelfPcr        : Ptr32 _KPCR
  +0x020 Prcb           : Ptr32 _KPRCB
  +0x024 Irql           : UChar
  +0x028 IRR            : Uint4B
  +0x02c IrrActive      : Uint4B
  +0x030 IDR            : Uint4B
………
```
结构片段: `KPCR`

其中，`Prcb` 指向的 `KPRCB` 结构片段如下：

```
nt!_KPRCB
  +0x000 MinorVersion   : Uint2B
  +0x002 MajorVersion   : Uint2B
  +0x004 CurrentThread  : Ptr32 _KTHREAD
  +0x008 NextThread     : Ptr32 _KTHREAD
  +0x00c IdleThread     : Ptr32 _KTHREAD
  +0x010 Number         : Char
  +0x011 Reserved       : Char
………
```
结构片段: `KPRCB`

其中的 `CurrentThread` 指向了 `Idle Process` 线程的地址。分析 `KTHREAD` 结构，如下数据域是我们感兴趣的：

```
+0x034 ApcState       : _KAPC_STATE
```

他指向了 `KAPC_STATE` 结构，分析该结构，发现其中有如下数据域：

```
+0x010 Process        : Ptr32 _KPROCESS
```

这是一个指向 `KPROCESS` 的指针，回顾前面给出的 `EPROCESS` 定义，发现这个结构就出现在 `EPROCESS` 的头部，也就是说，`Process` 指向了 `Idle Process` 的 `EPROCESS` 指针。

因而，只需要更具上述过程找到 `KAPC_STATE.Process` 的数据便获取了 `Idle Process` 的 `EPROCESS` 指针。

## 对 PAE 环境的支持

在本软件开发初期调试阶段，发现直接内存扫描 `EPROCESS` 等方法无法在双核机器上工作，后来得知 `windows` 自动为双核机器开启了 `PAE(Physical Address Extension)` 模式。

对于开启时 `PAE` 模式的系统，内核模块将不再是以前的 `ntoskrnl.exe`，而是 `ntkrpamp.exe` 或者其他。因而许多内核符号地址以及数据结构存放地址均有变化。

对于开启了 `PAE` 模式的系统，其页表为如下结构：

![](img/94567b46f9b968d51598c10799a0b668.jpg)

图：`PAE` 模式的页表结构

同时页表的存放地址变为 `0xc0600000`，对于上述的虚拟内存有效性检测，需要作出修改才能适应 `PAE` 模式。

然而在本软件中，采用了内核提供的 `MmIsAddressValid` 函数来判断页表。这样可以直接避免 `PAE` 模式带来的差异，也降低了程序的复杂程度。

## 防护进程非法关闭的实现

在执行进程的销毁中，`NtTerminateProcess` 函数起到了决定性作用。因而最直观的想法就是对该函数进行 Hook。同时判断是否为需要保护的进程，如果是则强制函数返回。

但是，`NtTerminateProcess` 并没有在 `5.x` 的内核中导出。因而直接获取其入口地址进行 inline hook 的难度较大，且极容易受到诸如内核补丁等的干扰。这里采用的办法是使用修改 `SSDT` 表中 `NtTerminateProcess` 入口的地址实现对该函数的 Hook。

对于 `SSDT` 表，在内核中的导出符号为：`KeServiceDescriptorTable`
其结构定义如下：

```
struct ServiceDescriptorEntry {
  unsigned int *ServiceTableBase;
  unsigned int *ServiceCounterTableBase;
  unsigned int NumberOfServices;
  unsigned char *ParamTableBase;
}
```

其中 `ServiceTableBase` 为一个指向对应系统调用号处理函数的数组。因而，我们需要作的就是首先获取 `NtTerminateProcess` 的系统调用号，然后修改上述数组，将入口地址修改为我们的 Hook 替换函数。

对于 `NtTerminateProcess` 的系统调用号，可以通过它的 wrap 函数 `ZwTerminateProcess` 进行分析：

```
; NTSTATUS __stdcall ZwTerminateProcess(HANDLE ProcessHandle,NTSTATUS ExitStatus)
public _ZwTerminateProcess@8
_ZwTerminateProcess@8 proc near

ProcessHandle= dword ptr  4
ExitStatus= dword ptr  8

mov     eax, 101h
lea     edx, [esp+ProcessHandle]
pushf
push    8
call    _KiSystemService
retn    8
```

其中第一行 `mov eax,101h` 是我们感兴趣的，`101h` 这个值就是 `NtTerminateProcess` 的系统服务号。而 `ZwTerminateProcess` 是一个系统导出的符号。

通过分析规律，其他 `Zw` 开头的 wrap 函数均以 `mov eax, i32` 的形式开始，因而可以通过如下宏获取任意 Wrap 函数对应目标函数的系统服务号：

```
#define SYSCALL_INDEX(_Function) *(PULONG)((PUCHAR)_Function+1)
```

今后的 Hook 操作便是对 `KeServiceDescriptorTable.ServiceTableBase[]` 进行查表和修改。



然而，即使在内核态，也无法直接对 `KeServiceDescriptorTable` 的数据进行修改，原因就是内核为了保证 `SSDT` 被非法修改，已将该段内存设为了只读模式。解决办法是通过将该段内存映射到一段可写入地址范围。下面代码通过创建 `MDL` 区段实现了该功能：

```
g_pmdlSystemCall = MmCreateMdl(NULL, KeServiceDescriptorTable.ServiceTableBase, KeServiceDescriptorTable.NumberOfServices*4);

if(!g_pmdlSystemCall)
    return;

MmBuildMdlForNonPagedPool(g_pmdlSystemCall);
```

今后，只需要对 `g_pmdlSystemCall` 指向的地址进行写入操作即可。

接下来需要制作用于替换 `NtTerminateProcess` 的 `Hook` 函数，已知 `NtTerminateProcess` 的原形如下：

```
NTSTATUS NtTerminateProcess(
              IN HANDLE ProcessHandle OPTIONAL,
              IN NTSTATUS ExitStatus
              );
```

其中 `ProcessHandle` 为需要关闭进程的对象句柄，可以通过 `ObReferenceObjectByHandle()` 将该句柄对应 `EPROCESS` 地址获取。因而，用以保护目标进程被恶意关闭的 `Hook` 替换函数如下：

```
NTSTATUS
NTAPI
NewNtTerminateProcess(
             IN HANDLE ProcessHandle OPTIONAL,
             IN NTSTATUS ExitStatus
             )
{
  PEPROCESS process_to_kill;
  if (ObReferenceObjectByHandle(ProcessHandle,GENERIC_READ,NULL,KernelMode,
    &process_to_kill,0) == STATUS_SUCCESS){

      if ( PEPROCESS2PROTECTED== process_to_kill &&
        PsGetCurrentProcess() != process_to_kill) return STATUS_ACCESS_DENIED;
  }
  return  Old_ZwTerminateProcess(ProcessHandle,ExitStatus);

}
```

其中，`PEPROCESS2PROTECTED` 为预先设置的需要保护的进程 `EPROCESS` 地址，它可以在主程序启动初期通过 `IoDeviceControl` 函数传送至驱动程序。

经过上述步骤以后，可以保护进程被非法关闭，效果图如下：

然而，目前仍旧可以通过附加调试器并强行终止进程调试的手段达到关闭进程的目的。原因在于附加调试进程使用了 `NtOpenProcess` 函数，因而为了进行较全面的保护，还需要 `Hook NtOpenProcess` 函数。拒绝一切对需要保护进程的开启操作。

但此举导致无法对本软件开启 `XP` 带来的主题界面，因而在没有特殊需要情况下，开启调试保护的驱动程序没有工作。

在进行了调试保护后，当尝试使用调试器附加进程时，将报错。下面是 `OllyIce` 尝试附加本软件时的错误信息：

**强制结束进程的实现**

由上述分析可知，强制结束进程的关键在于使用 `NtTerminateProcess` 函数。然后同样可以通过 `Hook` 该函数达到防止被关闭的目的。其中 `Hook` 手段有 `SSDT hook` 和 `inline hook`。

为了防止第三方进程采用同样的手段避免自身被非法关闭，可以采用下面 2 类办法：

a) 恢复 `SSDT` 函数入口或恢复 `inline hook` 修改过的函数入口代码

b) 直接调用未导出的 `PspTerminateProcess` 函数

其中，第一种做法较为常见，而第二类做法操作难度较大，主要是因为 `PspTerminateProcess` 为非导出的符号，而且目前关于该函数使用的相关资料几乎为零。通过 `IDA pro5` 对内核进行反工程可以看到该函数的汇编代码，摘录部分片段如下：

```
PAGE:00555772 ; __stdcall PspTerminateProcess(x, x)
PAGE:00555772 arg_0           = dword ptr  8
PAGE:00555772 arg_4           = dword ptr  0Ch
PAGE:00555772
PAGE:00555772                 mov     edi, edi
PAGE:00555774                 push    ebp
PAGE:00555775                 mov     ebp, esp
PAGE:00555777                 push    esi
PAGE:00555778                 mov     eax, large fs:124h
PAGE:0055577E                 mov     esi, [ebp+arg_0]
PAGE:00555781                 cmp     esi, [eax+44h]
PAGE:00555784                 jnz     short loc_55578D
PAGE:00555786                 mov     eax, 0C000000Dh
PAGE:0055578B                 jmp     short loc_5557E7
PAGE:0055578D loc_55578D:                       ; CODE XREF: PspTerminateProcess(x,x)+12j
PAGE:0055578D                 push    edi
PAGE:0055578E                 lea     edi, [esi+248h]
PAGE:00555794                 test    byte ptr [edi+1], 20h
```
代码：`PspTerminateProcess` 函数片断

可以猜测其参数与 `NtTerminateProcess` 相似，然而直接调用具有下面几点风险：

a) `NtTerminateProcess` 未被导出，难以有效定位

b) `NtTerminateProcess` 是内部函数，直接用其销毁进程会导致系统不稳定

因而，为了保证本软件的稳定性，在本次实现中没有考虑第二种做法。然后对于大部分情况，本软件强制结束进程能力十分有效。

**用户态读写物理内存的实现**

为了精简内核程序的设计，本软件将大部分进程操作功能在用户态程序加以实现。这里便需要用户态程序具有物理内存的读写能力。

对于读写物理内存，可以通过设置内核目录树中 `\Device\PhysicalMemory` 对象 `ACL` 权限，将写入权限附加去该对象，并使用 `NtOpenSection` 打开。这部分内容已比较成熟，可以找到许多相关文章。

接下来的问题是如何将虚拟内存转化为物理内存。由于大部分的内存读取是在内核地址空间中的，因而有一种较为简单的映射模式：

```
  if (vAddress < 0x80000000L || vAddress >= 0xA0000000L) {
    add.QuadPart = (ULONGLONG) vAddress & 0xFFFF000;
  } else {
    add.QuadPart = (ULONGLONG) vAddress & 0x1FFFF000;
  }
  add.QuadPart = add.QuadPart + (ULONGLONG) (vAddress & 0xFFF);
```
代码：将内核地址转化为物理地址的经验代码

上述代码可以很有效的转化在未分页区段的内核内存。然而在本次试验中，时常出现 `EPROCESS` 存在于分页内存的情况，因而上述方法未必有效。

解决办法有下面 2 种

a) 模拟内核查找页表的模式完成函书转化

b) 通过 `Callgate` 技术从 `ring3` 动态提权至 `ring0`，调用内核函数 `MmGetPhysicalAddress` 完成内存转化。

在本软件代码中，均实现了 2 种方式的功能，然而考虑到 `Callgate` 的不稳定因素，最终只采用第一种方案。下面将分别介绍。

对于模拟内核查找页表，上文已经介绍过利用该项技术实现了有效地址的判断。对于用户态实现，大致算法相同，唯一需要解决的问题是页表存放地址 `0xC0300000` 这是一内核态虚拟地址，同样需要作物理内存转化。这样一来似乎陷入了循环。

不过实际情况是 `5.x` 内核对于页表的存放地点一般是固定的，如下是未开启 `PAE` 模式中各内核的页表存放的物理地址：

内核版本 页表物理地址

`win2k` `0x30000`

`XP` `0x39000`

`2003` `0x39000`

获取了页表物理地址后，通过 `NtMapViewOfSection` 函数即可将该部分物理内存映射至本地程序地址空间进行操作。

对于使用 `Callgate` 调用 `MmGetPhysicalAddress` 的方法，首先需要遍历 `GDT` 表找到空白项用于存放 `CALLGATE`。

`x86` 指令体系中可以用如下指令获取 `GDT` 表地址，同时它支持 `ring3` 级别程序的调用:

```
_asm sgdt gGdt_addr;
```

`CALLGATE` 是一个特殊的 `GDT` 表项，他用于实现 `ring3` 程序直接调用高级别程序代码运行。这样为 `Ring3` 程序的动态提权提供了可能。

本软件中函数 `InstallCallgate` 实现了 `CallGate` 代码的设置，该函数从 `GDT` 表末尾开始搜索没有使用的 `GDT` 表项，然后设置需要调用的 `CALLGATE` 信息：

```
     CgDesc->offset_0_15  = (WORD) (Function & 0xFFFF);
     CgDesc->selector     = 0x8;
     CgDesc->param_count  = 0;
     CgDesc->some_bits    = 0;
     CgDesc->type         = 0xC;     // 32-bits callgate junior
     CgDesc->app_system   = 0;      // A system segment
     CgDesc->dpl          = 3;      // Ring 3 code can call
     CgDesc->present      = 1;
     CgDesc->offset_16_31 = (WORD) (Function >> 16);
     pGATMap->Desc = CgDesc;
```
代码：`InstallCallgate` 设置 callgate 片段

其中 `type` 字段的设置表明了此项 `GDT` 表为 `Callgate`，而 `Callgate` 函数的高低 16 位分别被设置到 `offset_0_15` 与 `offset_16_31` 数据域中。

设置完 `Callgate` 后，需要定义 `Callgate` 函数，由于该函数不支持传统的函数调用规范，也不希望编译器加入其它无用代码，需要在函数申明前加上 `__declspec(naked)` 编译指令。同时由于希望 `Callgate` 函数使用同程序相同的地址空间，也就是不希望在执行 `CallGate` 时发生进程调度，可以在 `CallGate` 中调用 `CLI` 指令暂时关闭中断。

为了方便，我们编写了下面宏负责 `CallGate` 函数的申明：

```
//CALL GATE STUB
#define DefineRing0Stub(func)  \
   __declspec(naked) void func() { \
     __asm {           \
     __asm  pushad       \
     __asm  pushf        \
     __asm  cli          \
        }


#define EndDefineRing0Stub()  \
      _asm {        \
     __asm  popf    \
     __asm  popad    \
     __asm  retf    \
      }          \
   }
```

接下来的任务就是在 `Callgate` 中调用内核 `API MmGetPhysicalAddress`。其代码如下：

```
DefineRing0Stub(cgNewGetPhysicalAddress)
    CurMap.pAddress = MmGetPhysicalAddress(CurMap.pAddress);
EndDefineRing0Stub()
```

这里还需要解决 2 个问题：

第一是 `MmGetPhysicalAddress` 函数入口地址的获取，由于该函数由 `ntoskrnl.exe` 模块导出，无法通过静态导入表绑定入口地址。解决办法是强行用 `LoadLibrary` 加载 `ntoskrnl.exe` 模块，然后使用 `GetProcAddress` 获取该函数在用户进程中的映射地址，接着通过 `ZwQuerySystemInformation` 查询 `ntoskrnl.exe` 的基址，从而将函数 `MmGetPhysicalAddress` 入后地址换算的真正的内核地址。

具体代码如下：

```
static inline DWORD GetKFuncAddr(LPCSTR funcName)
{
  DWORD ans = NULL;
  if (g_hKMODULE) {
    ans = ((DWORD) g_kernel_base - (DWORD)g_hKMODULE +
      (DWORD) GetProcAddress(g_hKMODULE, funcName));
  }
  return ans;
}
```

上述函数实现了获取任意导出的内核函数地址的功能，其中 `g_hKMODULE` 为通过 `LoadLibrary` 加载的 `ntoskrnl.exe` 的用户态基址。而 `ntoskrnl.exe` 内核基址由 `g_kernel_base` 给出，该变量可以通过程序中的函数：`DWORD GetModuleBase(LPCSTR name)` 获取。

在实现了虚拟地址到物理地址的转化后，便很容易的实现了用户态程序直接对内存的读写。

**动态监视进程创建、销毁的实现**

对于监视进程的创建销毁，本软件使用了内核函数 `PsSetCreateProcessNotifyRoutine` 函数，他将进程创建、销毁的信息回调给我们设置的函数。

然而关键问题在于如何实现内核态程序在收到消息后通知用户程序。下面是几种可行的方法：

a) 在用户态设立单独进程，以阻塞方式请求 `IoDeviceControl`。而内核态函数以异步方式处理该请求，当有进程创建消息时，完成一个 `IRP` 请求。从而唤醒了用户态线程。实现了消息通知。

b) 在用户台采用异步的文件读写请求，内核态使用发送 `KEvent` 消息实现消息通知

c) 又用户态程序定期轮询。

前 2 种方案的优点在于可以在进程创建或销毁后立即通知用户程序，从而得以立即刷新进程列表。但是实现异步 `IRP` 请求设计复杂，同时要避免线程锁死等问题。

因而本软件使用了第三种方法。考虑到软件本身需要一固定频率刷新进程信息，因而完全可以使用户态程序占有主导权。



具体实现方式是在内核程序中设立全局队列，每当有新的进程创建/销毁消息时，便往队列中追加新的项目。当用户程序读取该队列时，则将队列清空。表示之前的进程监控情况已转达给用户程序。

同时为了保护内核安全，全局队列以 20 笔记录为限，如果记录了超过 20 笔记录仍没有用户程序读取该队列，则设置一个 `Dirty` 标志，表示记录溢出。并且停止消息的添加。在今后用户程序读取队列时，发现 `Dirty` 标志则强制刷新进程列表，防止有“影子”进程的存在。

经过实践证明，该方法不但实践简单，同时也具有很高的效率。每秒一次的轮询足够给用户以瞬时刷新进程列表的感觉。

## 参考文献

*   `Windows Internals,Fourth Edition`
*   `Undocumented Windows NT`
*   `Undocumented Windows.2000.SECRETS.A PROGRAMMERS.COOKBOOK`
*   `Playing with Windows /dev/(k)mem, By crazylord,`  
    `http://www.phrack.org/phrack/59/p59-0x10.txt`
*   `浅谈 NT 下 Ring3 无驱进入 Ring0 的方法, By hopy,`  
    `http://bbs.pediy.com/showthread.php?threadid=37946`
*   `必备绝技 hook 大法( 中 ), By Lvg,`  
    `http://bbs.pediy.com/showthread.php?threadid=42422`
*   `Do All In One EXE File Under Win32, By icelord,`  
    `www.xfocus.net/articles/200610/887.html`
*   `How PAE X86 Works.mht, Microsoft Tech Net,`  
    `http://technet2.microsoft.com/windowsserver/en/library/00284c8d-7a42-40f2-8a01-8de61dccd8c91033.mspx`
*   `内存与进程管理器, By greatdong,`  
    `https://www.xfocus.net/bbs/index.php?act=ST&f=2&t=58182`
*   `Ring0 下搜索内存枚举隐藏进程, By uty,`  
    `http://www.cnxhacker.net/Article/show/3412.html`
*   `Process Termination: Known Methods, By EP_X0FF,`  
    `http://forum.sysinternals.com/forum_posts.asp?TID=5865`
*   `Administrator 用户直接获取 SYSTEM 权限, By scz,`  
    `www.nsfocus.net/index.php?act=magazine&do=view&mid=1900`
*   `获得进程的 EPROCESS, By MustBE`  
    `www.xfocus.net/articles/200406/706.html`
*   `获取 Windows 系统的内核变量, By 于`  
    `www.nsfocus.net/index.php?act=magazine&do=view&mid=2248`
*   `一些常用内核变量的定位, By flyingkisser`  
    `http://forum.byr.cn/pc/index.php?id=flyingkisser&pno=7`
*   `Windows Template Library (WTL),`  
    `http://www.codeproject.com/wtl/`

`4dd466d0e1e0cc123afbe504da2462ca`
