
# 几个常用的 WinDbg 命令

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-26.htm`](https://www.pediy.com/kssd/pediy09/pediy09-26.htm)

## 1. 查询符号

```
kd> x nt!KeServiceDescriptorTable*
8046e100 nt!KeServiceDescriptorTableShadow = <no type information>
8046e0c0 nt!KeServiceDescriptorTable = <no type information>

kd> ln 8046e100
(8046e100)   nt!KeServiceDescriptorTableShadow   | (8046e140)   nt!MmSectionExtendResource
Exact matches:
nt!KeServiceDescriptorTableShadow = <no type information>
```

## 2. 下载系统文件的符号

```
symchk c:\winnt\system32\ntoskrnl.exe /s srv*c:\symbols*http://msdl.microsoft.com/download/symbols

SYMCHK: FAILED files = 0
SYMCHK: PASSED + IGNORED files = 1
```

## 3. 查看 event 对象的信号状态

```
!object \BaseNamedObjects
dt -b nt!_KEVENT xxxxxxxx
```

## 4. 查看 LastError 值

```
!gle
```

## 5. 指定进制形式

`0x`/`0n`/`0t`/`0y` 分别表示 16/10/8/2 进制。

```
? 0x12345678+0n10
Evaluate expression: 305419906 = 12345682
```

## 6. 过滤命令窗口输出信息

```
.prompt_allow -reg +dis -ea -src -sym
```

## 7. `.formats` 命令

以多种格式显示表达式的值。

```
0:000> .formats @eax
Evaluate expression:
 Hex:     00181eb4
 Decimal: 1580724
 Octal:   00006017264
 Binary: 00000000 00011000 00011110 10110100
 Chars:   ....
 Time:    Mon Jan 19 15:05:24 1970
 Float:   low 2.21507e-039 high 0
 Double: 7.80981e-318
```

## 8. 异常处理相关

有 `sx`, `sxd`, `sxe`, `sxi`, `sxn`, `sxr` 几条命令可用来设置异常和事件的处理方式。比如：

```
0:000> sxe ld
```

可以在加载 `dll` 时中断下来。

## 9. 内核调试时切换进程

```
lkd> !process 0 0
lkd> .process xxxxxxxx
```

## 10. 在桌面上建立 WinDbg.exe 快捷方式

可在桌面上建立一个 `WinDbg.exe` 的快捷方式，然后在该快捷方式的属性里设置如下命令行：

```
C:\WinDBG\windbg.exe -c ".prompt_allow +dis -reg -ea -src -sym; .enable_unicode 1; .enable_long_status 1; .logopen /t c:\dbglog\dbglog.txt"
```

## 11. 本机内核调试

通过 `File/Kernel Debug…` 菜单可以打开内核调试选择窗口，选择最后一个 `Local` 选项页，确定后可以以内核方式调试本地机器。这时所有会挂起系统的命令都用不了了，但可以读写系统内存。另外，有一个方便的用途是用来查看系统结构，比如：`dt nt!_EPROCESS`。

---

# WinDbg 10 大调试命令

## `u`: 反编译机器码

在检查 `crash dump` 是否正确时，你已经用过了此命令，`u` 命令有三种格式：

1. `u <from>` 从地址 `<from>` 开始反编译 8 个机器码。
2. `u <from> <to>` 反编译 `<from>` 到 `<to>` 之间的所有机器码。
3. `u` 不提供任何参数时，从上次 `u` 命令停止的位置开始反编译。

当然，反编译大段代码是十分厌烦的，但如果你只想知道在特定地址发生的事情，那这是最便捷的方法。或许 `u` 命令最令人感兴趣的特性是它可以解析代码引用到的符号----即使是目标模块没有导出的符号。

## `db`, `dw` 和 `dd`: Dump Memory BYTEs、WORDs 和 DWORDs

如果你当前感兴趣的内存数据是二进制的，那么调试器的 16 进制转储命令将能完成此任务。根据你对源地址（`source address`）数据类型的判断，来选择 `dd`（针对 `BYTES`）、`dw`（针对 `WORDS`）、`dd`（针对 `DWORDS`）。

1. `db` 将指定内存范围里的数据显示为两个部分：左边是 16 进制表示（每 2 个 8 `bit` 一组），右边是对应的 `ASCI` 码。
2. `dw` 仅按照 16 进制显示（16 `bit` 一组）
3. `dd` 仅按照 16 进制显示（32 `bit` 一组）

此组命令可以使用与 `u` 命令相同的参数。注意，`<to>` 所指示的地址内容，也会被显示出来。如果没有任何参数，将显示接下来的 128 个字节。

## `x`: 检查符号

`x` 命令非常重要。它可以根据已安装的符号文件创建一个列表。典型的使用方式如下：

1. `x *!*` 显示所有可用符号的模块。在启动后，默认只有 `ntoskrnl.exe` 的符号是可用的。其他模块的符号可以使用 `.reload` 命令来加载。
2. `x <module>!<filter>` 显示模块 `<module>` 的符号文件中的符号名称，`<filter>` 可以包括通配符 `?` 和 `*`。`<module>` 必须属于 `x *!*` 列出的模块名。例如，`x nt!*` 将列出在内核符号文件 `ntoskrnl.dbg` 中找到的所有符号，`x win32k!*` 将列出 `win32k.dbg` 提供的符号。如果调试器报告说“Couldn’t resolv‘X….’”，尝试用 `.reload` 再次加载所有的符号文件。
3. `x <filter>` 显示所有可用符号的一个子集，该子集不匹配 `<filter>` 表达式。本质上，这是 `x <module>!<filter>` 的一个变形，在这里 `<module>!` 被省略了。随符号名一起显示的，还有与其相关的虚拟地址。对于函数名，与其对应的就是函数的入口地址。对于变量，就是改变量的基地址。该命令值得注意的地方是，它可以输出很多内部符号（`internal symbols`），这些在可执行文件的导出表中都是找不到的。

## `ln`: 列出最近的符号

`ln` 是我最喜欢的一个命令。因为它可以快速且简单的访问已安装的符号文件。算是 `x` 命令的理想补充。不过后者适用于列出所有系统符号的地址。`ln` 命令则用于按照地址或名称查找符号。

1. `ln <address>` 显示 `<address>` 指示的地址以及和其前后相邻的地址的符号信息。
2. `ln <symbol>` 将符号名解析为与其对应的虚拟地址。其过程与 `ln <address>` 类似。像 `x` 命令一样，调试器知道所有导出的以及一些内部的符号。因此，对于想弄清楚出现在反编译列表或 16 进制转储中的不明指针的确切含义的人有着非常大的帮助。注意，`u`、`db`、`dw`、`dd` 也会使用符号文件。

## `!processfield`: 列出 EPROCESS 的成员

该命令前的 `!` 号，意味着它来自于调试器的扩展模块—`kdextx86.dll`。该命令可显示内核用来代表一个进程的 `EPROCESS` 结构（该结构并没有正式的说明文档）的成员及其偏移量。

尽管该命令仅列出了成员的偏移量，但你也能很容易的猜出其正确的类型。例如，`LockEvent` 位于 `0x70` 处，其下一个成员的偏移量为 `0x80`。则该成员占用了 16 个字节，这与 `KEVENT` 结构非常类似。

## `!threadfields`: 列出 ETHREAD 成员

这是 `kdextx86.dll` 提供的另一个强大的选项。和 `!processfields` 类似，它列出未文档化的 `ETHREAD` 结构的成员及其偏移量。内核使用它表示一个线程。

## `!drivers`: 列出已加载的 Drivers

`kdextx86.dll` 真是太棒了。`!drivers` 列出了当前运行的内核和文件系统模块的详细信息。如果检查 `crash dump`，该命令会列出系统崩溃那一刻的系统状态。示例 1-3 是我机器上输出的摘要。注意，在输出的最后一行，导致 `Windows 2000` 崩溃的 `Driver` 的地址为 `0xBECC2000`，这显然是 `w2k_kill.sys` 引发蓝屏后显示的地址。

> **注**：在 `i386kd.exe`（`ver: 6.3.0017.0`）中，`!driver` 命令已不被支持。取而代之的是 `lm` 命令。该命令的一般用法是：`lm t n`

## `!sel`: 检查 Selector 的值

`!sel` 实现于 `kdextx86.dll`。它用来显示 16 个连续的 `memory selector`（按地址升序排列）。你可以反复的使用此命令直到出现“`Selector is invalid`”。在第四章将讨论 `Memory Selector`，到时我会提供一个示列代码来演示如何在你的程序中 `crack selectors`。

> **注**：在新的调试器中，该命令已不被支持，取而代之的是：`dg` 命令。其一般性用法为：`dg.` 注意末尾的 `.` 符号。`dg` 命令最多可列出 256 个 `Selector`。调试器的 `Online Help` 中有详细说明。
