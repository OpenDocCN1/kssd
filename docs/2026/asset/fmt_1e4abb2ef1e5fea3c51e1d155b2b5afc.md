
```c
BOOLEAN
ExceptionStackTrace()
{
PVOID *StackBase = (PVOID *)__readgsqword(0x1A8);
PVOID *StackCurrent = (PVOID *)_AddressOfReturnAddress();
if ((ULONG64)StackBase - (ULONG64)StackCurrent > 0x6000 || StackCurrent > StackBase)
{
return FALSE;
}
BOOLEAN bFindKeThawExecution = FALSE;
BOOLEAN bFindKdpReport = FALSE;
BOOLEAN bFindKiDispatchException = FALSE;
for (; StackCurrent < StackBase; ++StackCurrent)
{
ULONG64 StackValue = *(ULONG64 *)StackCurrent;
if (StackValue <= (ULONG64)MM_HIGHEST_USER_ADDRESS)
{
continue;
}
if (!bFindKeThawExecution)
{
if (StackValue > g_TrackStackFunc.KeThawExecution &&
StackValue < g_TrackStackFunc.KeThawExecution + 0x100)
{
bFindKeThawExecution = TRUE;
}
continue;
}
if (!bFindKdpReport)
{
if (StackValue > g_TrackStackFunc.KdpReport &&
StackValue < g_TrackStackFunc.KdpReport + 0x150)
{
bFindKdpReport = TRUE;
}
continue;
}
if (!bFindKiDispatchException)
{
if (StackValue > g_TrackStackFunc.KiDispatchException &&
StackValue < g_TrackStackFunc.KiDispatchException + 0x100)
{
bFindKiDispatchException = TRUE;
return TRUE;
}
}
}
return FALSE;
}
```

### 6.2.4 从堆栈中恢复丢失的异常信息

顺利筛选出异常派发的调用来源后，那么新的问题来了，在多层的函数调用中，原始`ExceptionRecord`、`Context`等参数已经丢失了，并没有传递给我们的Hook异常回调

这个时候没招了嘛？接管异常的路子走不通了？

当然不会，此时我们可以分析上层函数的调用逻辑，向上回溯堆栈，尝试**从上层函数栈帧中恢复异常参数**，根据x64调用约定，调用子函数中，涉及到使用非易失寄存器时，由被调用方自己保存、恢复，所以这里笔者列出来整个调用链中，每一个函数的`序言部分代码`：

```c
// KdpTrap
PAGEKD:0000000140B662B8 48 8B C4 mov rax, rsp
PAGEKD:0000000140B662BB 48 89 58 08 mov [rax+8], rbx
PAGEKD:0000000140B662BF 48 89 68 10 mov [rax+10h], rbp
PAGEKD:0000000140B662C3 48 89 70 20 mov [rax+20h], rsi
PAGEKD:0000000140B662C7 57 push rdi
PAGEKD:0000000140B662C8 48 83 EC 40 sub rsp, 40h
// KdpReport
.text:00000001404CEF84 48 8B C4 mov rax, rsp
.text:00000001404CEF87 48 89 58 08 mov [rax+8], rbx // Context
.text:00000001404CEF8B 48 89 68 10 mov [rax+10h], rbp
.text:00000001404CEF8F 48 89 70 18 mov [rax+18h], rsi
.text:00000001404CEF93 48 89 78 20 mov [rax+20h], rdi
.text:00000001404CEF97 41 54 push r12
.text:00000001404CEF99 41 56 push r14
.text:00000001404CEF9B 41 57 push r15
.text:00000001404CEF9D 48 83 EC 20 sub rsp, 20h
// KdExitDebugger
PAGEKD:0000000140B66008 40 53 push rbx
PAGEKD:0000000140B6600A 48 83 EC 20 sub rsp, 20h
// KeThawExecution
.text:00000001404EE390 48 89 5C 24 08 mov [rsp+arg_0], rbx
.text:00000001404EE395 48 89 74 24 10 mov [rsp+arg_8], rsi
.text:00000001404EE39A 57 push rdi
.text:00000001404EE39B 48 83 EC 20 sub rsp, 20h
// KeQueryPerformanceCounter
.text:0000000140254AD0 40 53 push rbx
.text:0000000140254AD2 41 54 push r12
.text:0000000140254AD4 41 57 push r15
.text:0000000140254AD6 48 83 EC 30 sub rsp, 30h
```

我们从`KdpTrap`函数开始分析，该函数将`Context`保存到了`rbx`这个非易失寄存器中，由于调用了`KdpReport`函数的内部使用了`rbx`，会将原`rbx`保存到栈上，那么我们可以向上回溯堆栈，找到一个返回地址是处于`KdpTrap`函数范围内的，即可找到期栈帧位置，从中恢复`Context`异常上下文信息

![图片描述](upload/attach/202609/953262_U2PSUNYNSXKD3TR.webp)

### 6.2.5 构造中断返回帧，恢复异常派发流程

由于我们接管了异常后，不能从正常栈中取出返回地址一层层返回，需要从`KeQueryPerformanceCounter`函数直接折返到`KdpTrap`的返回地址处，即直接跳转回到`KiDispatchException`函数内，流程如下所示：

![图片描述](upload/attach/202609/953262_EVXXBTCJ72Q9V7Z.webp)

要做到这一点，我们要保证进入、退出`KdpTrap`函数时非易失寄存器现场要保持一致，那么就需要往上回溯栈帧，恢复所有被破坏的非易失寄存器环境，笔者这里给出部分实验代码：

```c
typedef struct _EXECUTION_CONTEXT
{
ULONGLONG RFlags; // +0x00
ULONGLONG Rax; // +0x08
ULONGLONG Rcx; // +0x10
ULONGLONG Rdx; // +0x18
ULONGLONG Rbx; // +0x20
ULONGLONG Rsp; // +0x28
ULONGLONG Rbp; // +0x30
ULONGLONG Rsi; // +0x38
ULONGLONG Rdi; // +0x40
ULONGLONG R8; // +0x48
ULONGLONG R9; // +0x50
ULONGLONG R12; // +0x58
ULONGLONG R13; // +0x60
ULONGLONG R14; // +0x68
ULONGLONG R15; // +0x70
ULONGLONG Rip; // +0x78
} EXECUTION_CONTEXT, *PEXECUTION_CONTEXT;
VOID
ContinueExecution()
{
PVOID *StackBase = (PVOID *)__readgsqword(0x1A8);
PVOID *CurStackPtr = (PVOID *)_AddressOfReturnAddress();
if ((ULONG64)StackBase - (ULONG64)CurStackPtr > 0x6000 || CurStackPtr > StackBase)
{
return VOID();
}
BOOLEAN bFindKdpReport = FALSE;
BOOLEAN bFindKdpTrap = FALSE;
BOOLEAN bFindKdTrap = FALSE;
EXECUTION_CONTEXT ExecutionContext = { 0 };
ExecutionContext.R13 = Asm_GetR13();
ExecutionContext.RFlags = Asm_GetRFlags() | 0x200;
for (; CurStackPtr < StackBase; ++CurStackPtr)
{
ULONG64 StackValue = *(ULONG64 *)CurStackPtr;
if (StackValue <= (ULONG64)MM_HIGHEST_USER_ADDRESS)
{
continue;
}
if (!bFindKdpReport)
{
if (StackValue > g_TrackStackFunc.KdpTrap &&
StackValue < g_TrackStackFunc.KdpTrap + 0x100)
{
ExecutionContext.Rbx = *(ULONG64 *)((ULONG64)CurStackPtr + 0x08);
ExecutionContext.Rbp = *(ULONG64 *)((ULONG64)CurStackPtr + 0x10);
ExecutionContext.Rsi = *(ULONG64 *)((ULONG64)CurStackPtr + 0x18);
ExecutionContext.Rdi = *(ULONG64 *)((ULONG64)CurStackPtr + 0x20);
ExecutionContext.R12 = *(ULONG64 *)((ULONG64)CurStackPtr - 0x8);
ExecutionContext.R14 = *(ULONG64 *)((ULONG64)CurStackPtr - 0x10);
ExecutionContext.R15 = *(ULONG64 *)((ULONG64)CurStackPtr - 0x18);
bFindKdpReport = TRUE;
}
continue;
}
if (!bFindKdTrap)
{
if (StackValue > g_TrackStackFunc.KiDispatchException &&
StackValue < g_TrackStackFunc.KiDispatchException + 0x100)
{
ExecutionContext.Rip = StackValue;
ExecutionContext.Rsp = (ULONG64)CurStackPtr + 8;
ExecutionContext.Rbx = *(ULONG64 *)((ULONG64)CurStackPtr + 0x08);
ExecutionContext.Rbp = *(ULONG64 *)((ULONG64)CurStackPtr + 0x10);
ExecutionContext.Rsi = *(ULONG64 *)((ULONG64)CurStackPtr + 0x18);
ExecutionContext.Rdi = *(ULONG64 *)((ULONG64)CurStackPtr - 0x8);
bFindKdTrap = TRUE;
break;
}
}
}
if (bFindKdpReport &&
bFindKdpReport &&
bFindKdTrap)
{
// DbgBreakPoint();
ContinueWithContext(&ExecutionContext);
KeBugCheck(0);
}
else
{
DbgBreakPoint();
KeBugCheck(0);
}
return VOID();
}
```

### 6.2.6 大量踩坑

这一部分内容，单从劫持异常的原理来说，其实不算是非常难，但是在上手写代码、调试时，笔者遇到遇到了大量踩坑的地方，其中感受最强烈的2点如下：

1. 这种劫持异常的点，正好处在系统派发异常到内核调试器路径上，导致调试比较困难。笔者甚至遇到了无限异常递归的尴尬情况，即虚拟机无影响、但是`WinDbg`一直接收不到异常

2. 挂内核调试器、不挂内核调试器这2种情况，似乎会改变系统内核异常派发的逻辑，导致不挂调试器时加载驱动卡死，挂上`WinDbg`后再加载驱动，问题却消失了

由于以上2点问题的存在，导致这个实验本身进度非常缓慢，达不到笔者心中可公开PoC的标准，所以笔者决定暂时不放出这部分实验的工程实现，对这部分内容感兴趣的朋友可以关注一下，日后工程完善了，笔者会放出实验工程文件

## 7. Git仓库开源地址

劫持`Wow64PrepareForException`接管异常，实现无痕Hook，仓库地址：<a href="elink@383K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6e0K9r3W2F1L8W2)9J5k6p5S2G2L8h3g2Q4x3V1k6u0L8Y4k6A6M7$3W2T1L8r3g2t1L8$3!0C8i4K6u0r3N6s2u0W2k6g2)9J5c8X3#2S2K9h3&6Q4x3V1k6u0L8Y4k6A6M7$3W2T1L8r3g2t1L8$3!0C8i4K6g2X3f1$3S2S2k6r3!0%4f1r3q4Y4k6b7`.`.">InvisibleHook_ShadowPage</a>

注册`PicoDispatchException`接管异常，实现无痕Hook，仓库地址：<a href="elink@f13K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6e0K9r3W2F1L8W2)9J5k6p5S2G2L8h3g2Q4x3V1k6u0L8Y4k6A6M7$3W2T1L8r3g2t1L8$3!0C8i4K6u0r3N6s2u0W2k6g2)9J5c8X3#2S2K9h3&6Q4x3V1k6u0L8Y4k6A6M7$3W2T1L8r3g2t1L8$3!0C8i4K6g2X3f1$3W2U0L8@1g2^5j5$3g2H3N6r3W2G2L8R3`.`.">InvisibleHook_PicoException</a>

如果各位朋友看完本文觉得有所帮助、启发，欢迎点赞、收藏本文。另外，欢迎各位朋友给仓库Star！

各位看雪论坛的朋友大家好，我是`__Shinn`，凌晨码字，精神有点迷糊，帖子中如果存在表述不严谨、甚至出错的地方，欢迎在评论区指正

本文定位为`抛砖引玉`，分享一些`实验性质`的思路、代码实现方法，本文在结构上大体分为了以下2部分

笔者最初原稿是4种提前接管异常的手法，后来发现，利用`InstrumentationCallback`提前接管异常，论坛内已经有师傅发文公开了，所以发帖前删除了这部分内容，避免重复赘述

本文涉及到的大部分工程文件已经整理好并开源到Github，欢迎给仓库Star

本次分析的目标系统版本为`Win11 26H1 28000`，如图所示：

![图片描述](https://bbs.kanxue.com/upload/attach/202609/953262_S7F3HD99AASJSC3.webp)

首先我们可以主动制造一个`除零`异常，如下所示：

当CPU检测到除零异常后会抛出`#DE`。由于`#DE`的异常向量号为0，所以CPU会找到IDT表中第0项中断门描述符，从中解析出异常处理回调的函数入口，将执行流转移到该入口。使用`WinDbg`调试如下所示：

![图片描述](https://bbs.kanxue.com/upload/attach/202609/953262_FKGH7GN28M6AA9G.webp)

由于在虚拟机中调试，并没启用`内核页表隔离`，所以这里的异常处理例程为`KiDivideErrorFault`，正常用户的PC开启该机制后，CPU抛出`#DE`异常实际执行的是`KiDivideErrorFaultShadow`

内核页表隔离并不是Win下特有的，在Linux下该技术叫做`KPTI`

在早期的Windows、Linux中，用户态和内核态会共用同一份页表基址（`Cr3`）。当前`Cr3`除了映射用户地址空间，还会完整映射内核地址空间。由于现代CPU存在预测执行机制，以 2018 年公开的 **Meltdown（CVE-2017-5754）** 为例，攻击者可以构造一段用户态代码，使CPU在权限检查完成前短暂读取内核地址中的数据，虽然这次读取最终会被判定为非法并回滚，但是CPU回滚状态时并不会清空缓存内容，通过测信道攻击后，R3可以读取到R0的数据
