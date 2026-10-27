
### MapAddress\_PTE

```c
BOOLEAN
MapAddress_PTE(
ULONG64 SrcVa,
ULONG64 DstVa,
ULONG Size
)
{
ULONG64 SrcStartAddress = (ULONG64)PAGE_ALIGN(SrcVa);
ULONG64 DstStartAddress = (ULONG64)PAGE_ALIGN(DstVa);
ULONG64 EndOffset = (ULONG64)PAGE_ALIGN((PVOID)(SrcVa + Size));
ULONG64 PageCount = (EndOffset - SrcStartAddress) / PAGE_SIZE;
if (PageCount == 0)
{
PageCount = 1;
}
for (ULONG64 i = 0; i < PageCount; i++)
{
ULONG64 CurrentSrcVa = SrcStartAddress + (i * PAGE_SIZE);
ULONG64 CurrentDstVa = DstStartAddress + (i * PAGE_SIZE);
HARDWAREPTE* SrcPde = (HARDWAREPTE*)GetPde(CurrentSrcVa);
HARDWAREPTE* DstPde = (HARDWAREPTE*)GetPde(CurrentDstVa);
if (!MmIsAddressValid(SrcPde) || !SrcPde->valid)
{
return FALSE;
}
if (!MmIsAddressValid(DstPde) || !DstPde->valid)
{
return FALSE;
}
HARDWAREPTE* SrcPte = (HARDWAREPTE*)GetPte(CurrentSrcVa);
HARDWAREPTE* DstPte = (HARDWAREPTE*)GetPte(CurrentDstVa);
if (!MmIsAddressValid(SrcPte) || !SrcPte->valid)
{
return FALSE;
}
if (!MmIsAddressValid(DstPte) || !DstPte->valid)
{
return FALSE;
}
DstPte->page_frame_number = SrcPte->page_frame_number;
__invlpg((PVOID)CurrentDstVa);
}
return TRUE;
}
```

但是很遗憾，经过笔者实验后发现，这种修改PTE页帧号的手法，在处理一些普通的内存页面时，的确可以实现内存共享（稳定挺长时间不蓝屏）。但是涉及到dll模块内的内存时，修改后系统1分钟之内必定会出现蓝屏错误，错误码为`MEMORY_MANAGEMENT`

直接修改硬件PTE的方式似乎走不通了，笔者这里选择的是通过`MDL`内存映射实现了内存页面共享，实现思路如下：

1. 释放影子模块`.data`节区的内容，让该线性地址释放、空闲出来
2. 调用`MDL`内存映射，让系统将指定的物理页面，映射到笔者指定的线性地址处
3. 经过笔者测试后，该方法算是比较完美地解决全局变量内存共享的问题，关键代码如下所示：

```c
BOOLEAN
MapAddress_MDL(
ULONG64 SrcVa,
ULONG64 DstVa,
ULONG Size
)
{
ULONG64 SrcStartAddress = (ULONG64)PAGE_ALIGN(SrcVa);
ULONG64 DstStartAddress = (ULONG64)PAGE_ALIGN(DstVa);
ULONG64 EndOffset = (ULONG64)PAGE_ALIGN((PVOID)(SrcVa + Size));
ULONG64 PageCount = (EndOffset - SrcStartAddress) / PAGE_SIZE;
if (PageCount == 0)
{
PageCount = 1;
}
for (ULONG64 i = 0; i < PageCount; i++)
{
ULONG64 CurrentSrcVa = SrcStartAddress + (i * PAGE_SIZE);
ULONG64 CurrentDstVa = DstStartAddress + (i * PAGE_SIZE);
if (!MmIsAddressValid((PVOID)CurrentSrcVa))
{
DbgPrint(
"[MapAddress] Source address 0x%llx is invalid\n",
CurrentSrcVa
);
return FALSE;
}
PVOID BaseAddress = (PVOID)CurrentDstVa;
SIZE_T RegionSize = PAGE_SIZE;
NTSTATUS status = ZwFreeVirtualMemory(
NtCurrentProcess(),
&BaseAddress,
&RegionSize,
MEM_RELEASE
);
if (!NT_SUCCESS(status))
{
DbgPrint(
"[MapAddress] Failed to decommit target address 0x%llx, status=0x%x\n",
CurrentDstVa,
status
);
}
PMDL Mdl = IoAllocateMdl((PVOID)CurrentSrcVa, PAGE_SIZE, FALSE, FALSE, NULL);
if (!Mdl)
{
DbgPrint(
"[MapAddress] Failed to allocate MDL for 0x%llx\n",
CurrentSrcVa
);
return FALSE;
}
__try
{
MmProbeAndLockPages(Mdl, UserMode, IoModifyAccess);
}
__except (EXCEPTION_EXECUTE_HANDLER)
{
DbgPrint(
"[MapAddress] Failed to lock pages for 0x%llx\n",
CurrentSrcVa
);
IoFreeMdl(Mdl);
return FALSE;
}
PVOID MappedAddress = MmMapLockedPagesSpecifyCache(
Mdl,
UserMode,
MmCached,
(PVOID)CurrentDstVa,
FALSE,
NormalPagePriority
);
if (!MappedAddress)
{
DbgPrint(
"[MapAddress] Failed to map to address 0x%llx\n",
CurrentDstVa
);
MmUnlockPages(Mdl);
IoFreeMdl(Mdl);
return FALSE;
}
if (MappedAddress != (PVOID)CurrentDstVa)
{
DbgPrint(
"[MapAddress] Mapped to 0x%p instead of requested 0x%llx\n",
MappedAddress,
CurrentDstVa
);
MmUnmapLockedPages(MappedAddress, Mdl);
MmUnlockPages(Mdl);
IoFreeMdl(Mdl);
return FALSE;
}
if (!AddMdlMapEntry(CurrentDstVa, Mdl))
{
DbgPrint("[MapAddress] Failed to track MDL for 0x%llx\n", CurrentDstVa);
MmUnmapLockedPages(MappedAddress, Mdl);
MmUnlockPages(Mdl);
IoFreeMdl(Mdl);
return FALSE;
}
DbgPrint(
"[MapAddress] Successfully mapped 0x%llx -> 0x%llx (MDL tracked)\n",
CurrentSrcVa,
CurrentDstVa
);
}
return TRUE;
}
```

### 4.5 给影子模块挂Inline Hook

经过前文的各种调试、处理后，的确可以较为稳定地将代码执行流劫持到影子模块中

那么接下来完美就可以自由地做手脚了，笔者这里给影子模块中的`send`函数下`inline hook`，可以正常监控参数信息，如下所示：

```c
int
WINAPI
MySend(
_In_ SOCKET s,
_In_reads_bytes_(len) const char FAR* buf,
_In_ int len,
_In_ int flags
)
{
printf("--------------------------------------------------- \n");
printf("[+] Send Hook - Socket: %llu, Length: %d\n", (ULONG64)s, len);
printf("[+] Data:%s \n", buf);
printf("--------------------------------------------------- \n");
return g_OriginalSend(s, buf, len, flags);
}
```

使用调试器，`send`处的代码**没有任何修改**，但是实际上确实是被我们Hook了，并正常输出参数

![图片描述](upload/attach/202609/953262_WUU7CYJ3DTJKSP5.webp)

这部分代码已经整理好，Git仓库地址见文末

## 5. 提前接管用户态异常方法2

前文介绍了一种Hook `ntdll!Wow64PrepareForException`实现提前接管用户态异常的方法，但是毕竟是用户态的对抗，一些游戏安全厂商、安全开发者往往会直接Hook `ntdll!KUserExceptionDispatcher`实现总管用户态异常，那么我们提前接管用户态异常的行为，还是有可能会被监控、扫描出来

那么有没有一种方法？

在退回到用户态之前，直接在内核态提前接管用户态异常呢？

有的兄弟，有的，笔者接下来将会介绍一种**类似Hook nt!KiDispatchException的方法**，实现在内核态提前接管用户异常

前文长篇分析Win11内核异常体系、无痕Hook，都属于前期铺垫，现在笔者将带着各位朋友，把**对抗的层级拉到内核层**，本文精彩的部分才刚刚开始

### 5.1 nt!PsPicoDispatchException

我们重新回到`nt!KiDispatchException`中进行分析，在该函数的开头，会连续判断3个条件是否成立：

- 是否为首次异常派发
- 判断当前进程对象是否为空
- 判断当前进程`Process.PicoContext`是否为空

![图片描述](upload/attach/202609/953262_YX8Y7ZZEPZXHGHB.webp)

如果以上3点都满足，那么不调用`KiPreprocessFault`函数提前将`#DE`内部错误转换为文档化的错误码`C0000094`

![图片描述](upload/attach/202609/953262_89NDPMB6YHGHMFD.webp)

随后从全局变量`xmmword_140F05660`取出一个函数指针，通过`CFG`发起一次间接函数调用，如果该函数返回`TRUE`，那么`ntKiDispatchException`函数提前返回，所以说**该函数的异常接管优先级非常高**，早于`内核调试器`、`R3调试器`、`用户VEH`、`用户SEH`接管异常。这个函数调用其实是`nt!PsPicoDispatchException`，这其实和`WSL`子系统有关，笔者对此粗浅的理解如下：

> 由于WSL Linux子系统的存在，那么子系统中的进程就叫做WSL进程吧，这些进程内同样会出现各种异常，需要系统异常体系派发、处理，但是WSL进程毕竟不是原生的Win进程，微软希望的是由WSL子系统本身优先接管、处理异常
>
> 异常派发中，是如何区分原生Win进程、WSL子系统进程呢？
>
> 靠的就是EPROCESS.PicoContext字段是否为NULL，没错，判断条件非常单薄

![图片描述](upload/attach/202609/953262_7JA6XGXKNBE5BEW.webp)

### 5.2 分析PicoProvider注册流程

我们继续分析这个利用点，由于是从全局变量`xmmword_140F05660`取出一个函数指针，再通过`CFG`发起一次间接函数调用，那么简单粗暴的方法，直接特征码搜索到这处代码，然后将我们自己的异常回调写进去。

这种方法是可行的，但是笔者认为这并不是最优解，首先涉及到特征码，后续在兼容多版本时工程维护起来会麻烦得多。还有就是这种系统异常派发的关键代码路径上，涉及到的全局变量可能会有`PG`保护。

所以笔者这里的思路是，既然这是给`WSL`子系统准备的异常处理，那么我们自己能不能通过合法的方式，将自己的Pico处理回调注册进去？既免去后期特征码维护、还让系统自己把异常回调写到全局变量`xmmword_140F05660`中

思路确定了，现在开始分析，我们查看全局变量`xmmword_140F05660`的交叉引用，发现`PsRegisterPicoProvider`函数处有调用

![图片描述](upload/attach/202609/953262_M47PTP6KQJEETU8.webp)

我们直接定位到该函数处，更惊喜的发现，这个函数居然还是导出！这个函数本身不长，我们直接来看：

- 判断参数1的第一个成员是否等于88
- 判断参数2的第一个成员是否等于96
- 判断[参数1 + 72]处该成员，高11位是否为0
- 判断[参数1 + 76]处该成员，高11为是否为0
- 全局变量`PspPicoRegistrationDisabled = 0`，否则当前禁止注册`Pico`异常回调
- 不满足以上任意一项，`Pico`注册函数是都会返回失败

![图片描述](upload/attach/202609/953262_CPW6NCY2Z28FDN8.webp)

使用Windbg调试发现，当Win11 26H1正常启动进入桌面后，全局变量`PspPicoRegistrationDisabled`会被置1，看来微软并不想让其他驱动随便注册`Pico`回调

![图片描述](upload/attach/202609/953262_XVGK9QA7HMAP7PW.webp)

![图片描述](upload/attach/202609/953262_GV7KF7MEXADSNDW.webp)

我们继续分析`PsRegisterPicoProvider`函数，以下代码功能很清晰，将参数1中的各个字段成员保存到全局变量中，参数2是作为输出参数，接收结果

![图片描述](upload/attach/202609/953262_RJHKMJ264B8MK2S.webp)

根据以上汇编，笔者反推参数1、参数2的部分对象结构，如下所示：

```c
typedef struct _PS_PICO_EXCEPTION_ARG1
{
ULONG64 Size;
PVOID Pad1[3];
PVOID DispatchException; // 这个是关键，我们自定义的异常处理回调指针
PVOID Pad2[2];
ULONG Flag[2];
PVOID Pad3[3];
} PS_PICO_EXCEPTION_ARG1, *PPS_PICO_EXCEPTION_ARG1;
typedef struct _PS_PICO_EXCEPTION_ARG2
{
ULONG64 Size;
PVOID Pad1[11];
} PS_PICO_EXCEPTION_ARG2, *PPS_PICO_EXCEPTION_ARG2;
```

### 5.3 注册成为PicoProvider

`Pico`回调的注册流程已经分析清楚了，那么我们的注册思路、关键代码如下：

1. 从nt导出表找到`PsRegisterPicoProvider`函数指针
2. 构造注册参数1、参数2，补齐必要的字段信息，避免注册函数检查失败
3. 修改`PspPicoRegistrationDisabled`这个控制变量，手动置零
4. 调用`PsRegisterPicoProvider`注册`Pico`回调
5. 还原`PspPicoRegistrationDisabled`这个控制变量，由于该变量修改窗口期非常小，不会对系统运行有什么影响
