
# [原创]凑个热闹：CVE-2014-1767 分析报告

## 写在前面

1. 本文首发于 <a href="elink@7caK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2V1k6h3u0#2k6%4N6S2M7W2)9J5k6h3y4G2L8g2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6z5e0p5g2Q4x3X3c8m8L8X3q4D9P5i4A6W2i4K6u0V1b7#2k6q4i4K6u0V1x3U0l9I4y4q4)9J5k6o6p5%4y4U0M7`.">Debugwar.com</a>，授权转载到看雪论坛，其他转载请注明出处。

2. 看雪的这个 Markdown 系统与我的写作系统有点冲突，排版可能会有问题，可以点上面的链接去看原文，获得更好的排版体验。

3. Linux 的输入法你懂的，可能会有错字别字，大家多包涵~

4. 其实本文今年年初的时候就写了大部分了，后来懒癌发作一直没有继续写完。最近看到论坛里不少人也发了这个漏洞的分析，索性就写完发到论坛里来凑个热闹。

## 简介

该漏洞为 Pwn2014 上 Siberas 团队公布的提权漏洞，此漏洞出现在 `AFD.sys` 文件中，POC 将以 `DoubleFree` 的形式触发漏洞，利用思路是通过将 `DoubleFree` 漏洞转换为 `UAF` 来构造 Exploit。

本文的调试环境为 Windows 7 SP1 x86，通过双虚拟机双机调试。这和参考 1 中漏洞发现者的环境不同，因此有些细节也不一样（例如漏洞发现者 PDF 中提到的内存为 `NonPagedPoolNx`，而我们的是 `NonPagePool`）。

本漏洞成因简单、利用思路也非常经典，因此极为适合新手作为漏洞学习的入门洞。

# 漏洞触发

## 漏洞触发现场

漏洞触发现场如下：

```text
****************************************************************
*                                                              *
* Bugcheck Analysis                                            *
*                                                              *
****************************************************************
BAD_POOL_CALLER (c2)
The current thread is making a bad pool request. Typically this is at a bad IRQL level or double freeing the same allocation, etc.
Arguments:
Arg1: 00000007, Attempt to free pool which was already freed
Arg2: 00001097, Pool tag value from the pool header
Arg3: 08b50005, Contents of the first 4 bytes of the pool header
Arg4: 8757da60, Address of the block of pool being deallocated
Debugging Details:
------------------
KEY_VALUES_STRING: 1
Key : Analysis.CPU.mSec
Value: 5140
Key : Analysis.DebugAnalysisManager
Value: Create
Key : Analysis.Elapsed.mSec
Value: 17640
Key : Analysis.Init.CPU.mSec
Value: 3765
Key : Analysis.Init.Elapsed.mSec
Value: 107564
Key : Analysis.Memory.CommitPeak.Mb
Value: 69
Key : WER.OS.Branch
Value: win7sp1_rtm
Key : WER.OS.Timestamp
Value: 2010-11-19T18:50:00Z
Key : WER.OS.Version
Value: 7.1.7601.17514
BUGCHECK_CODE: c2
BUGCHECK_P1: 7
BUGCHECK_P2: 1097
BUGCHECK_P3: 8b50005
BUGCHECK_P4: ffffffff8757da60
POOL_ADDRESS: 8757da60 Nonpaged pool
FREED_POOL_TAG: Mdl_
PROCESS_NAME: CVE-2014-1767.exe
STACK_TEXT:
80f6454c 83ce6589 00000003 610826c5 00000065 nt!RtlpBreakWithStatusInstruction
80f6459c 83ce7085 00000003 8757da58 000001ff nt!KiBugCheckDebugBreak+0x1c
80f64960 83d2cc4e 000000c2 00000007 00001097 nt!KeBugCheck2+0x68b
80f649d8 83c8276a 8757da60 00000000 89136350 nt!ExFreePoolWithTag+0x1b2
80f649ec 8e3a6eb0 8757da60 00000000 8e38989f nt!IoFreeMdl+0x70
80f64a08 8e3898ac 00000000 00000001 0ec1bfa8 afd!AfdReturnTpInfo+0xad
80f64a44 8e38abba 0ec1bf00 000120c3 8e38aa8c afd!AfdTliGetTpInfo+0x89
80f64aec 8e38f2bc 89067880 87610030 80f64b14 afd!AfdTransmitPackets+0x12e
80f64afc 83c43047 87610030 890a8ee0 890a8ee0 afd!AfdDispatchDeviceControl+0x3b
80f64b14 83e199d5 89067880 890a8ee0 890a8fbc nt!IofCallDriver+0x63
80f64b34 83e1bdc8 87610030 89067880 00000000 nt!IopSynchronousServiceTail+0x1f8
80f64bd0 83e22d9d 87610030 890a8ee0 00000000 nt!IopXxxControlFile+0x6aa
80f64c04 83c4987a 00000050 00000000 00000000 nt!NtDeviceIoControlFile+0x2a
<Intermediate frames may have been skipped due to lack of complete unwind>
80f64c04 778770b4 (T) 00000050 00000000 00000000 nt!KiFastCallEntry+0x12a
<Intermediate frames may have been skipped due to lack of complete unwind>
002cf478 77875864 (T) 75a0989d 00000050 00000000 ntdll!KiFastSystemCallRet
002cf47c 75a0989d 00000050 00000000 00000000 ntdll!ZwDeviceIoControlFile+0xc
002cf4dc 75d2a671 00000050 000120c3 01326200 KERNELBASE!DeviceIoControl+0xf6
002cf508 013217be 00000050 000120c3 01326200 kernel32!DeviceIoControlImplementation+0x80
WARNING: Stack unwind information not available. Following frames may be wrong.
002cf6f8 01321396 00000012 000c0ffc 000c0fec CVE_2014_1767+0x17be
002cf798 75d33c45 00000000 75d33c45 7ffd9000 CVE_2014_1767+0x1396
002cf7ac 778937f5 7ffd9000 77be9209 00000000 kernel32!BaseThreadInitThunk+0xe
002cf7ec 778937c8 013214c0 7ffd9000 00000000 ntdll!__RtlUserThreadStart+0x70
002cf804 00000000 013214c0 7ffd9000 00000000 ntdll!_RtlUserThreadStart+0x1b
SYMBOL_NAME: afd!AfdReturnTpInfo+ad
MODULE_NAME: afd
IMAGE_NAME: afd.sys
STACK_COMMAND: .thread ; .cxr ; kb
FAILURE_BUCKET_ID: 0xc2_7_Mdl__afd!AfdReturnTpInfo+ad
OS_VERSION: 7.1.7601.17514
BUILDLAB_STR: win7sp1_rtm
OSPLATFORM_TYPE: x86
OSNAME: Windows 7
FAILURE_ID_HASH: {7fe1e721-1d80-7be3-9354-8d3b5b5ab1ef}
Followup: MachineOwner
---------
```

由上述第 69 行可知，为 `IoFreeMdl` 在释放 `8757da60` 时出现 `Double Free` 错误导致。到这里，很自然会产生如下几个疑问：

- 为什么会 `DoubleFree`？<--- 终极问题
- `8757da60` 这个 MDL 是在哪里分配的？
- 第一次是在哪里释放的？
- 第二次是在哪里释放的？

其中，第四个问题可以立刻解答，因为当前现场即是第二次释放现场。

为了解答上面的第一个终极问题，我们要先搞清楚问题 2、3。

## 寻找内存分配时机

> 注意：由于该漏洞属于内核级漏洞，因此调试过程中需要不停重启，因此下文中截图涉及内存地址的地方会有差异。

既然是寻找 MDL 的内存分配，首先看一眼 `IoAllocateMdl` 的调用交叉引用：

![图片描述](upload/attach/202206/156241_8M525MMBDZAU9RY.png)

Emmm……打消了一个一个找的念头。

然后想利用条件断点在 `IoAllocateMdl` 返回 `8757da60` 这个地址时下断，发现一旦下了断点，这个地址会发生改变——似乎进入死胡同了（后来发现其实多断几次总是能断到的）。

这里网上大部分的分析文章都是从 POC 的两次 `DeviceIOControl` 着手分析。本文本着自虐的原则，笔者会假设我们并不知道 POC 的内容是什么，这次 `BugCheck` 只是漏洞挖掘的 Fuzz 环境中发现的一次内核级异常，我们将尽量尝试贴近真实的漏洞挖掘环境。

首先上 IDA 看一下出现异常的现场上下文是怎样的（注意，下图中已经加载了符号分析得到的数据结构）：

![图片描述](upload/attach/202206/156241_N3UAMNBMYSWTRY7.png)

有的朋友可能好奇这些数据结构是如何分析得到的，在这里笔者只能告诉大家，通过对：`AfdReturnTpInfo`、`AfdTransmitFile`、`AfdTliGetTpInfo` 可以得到如下关键结构：

```c
struct struc_UnkObj
{
    NTSTATUS Status;
    int Length;
    PVOID VirtualAddress;
    PMDL MdlAddress;
    PFILE_OBJECT Object;
    int field_14;
};

struct struc_TPInfo
{
    PIRP field_0;
    int field_4;
    int field_8;
    int field_C;
    int field_10;
    int field_14;
    int field_18;
    PIRP Irp3;
    struc_UnkObj *UnkObjArray;
    int field_24;
    int UnkCounter;
    int field_2C;
    int IrpCounter;
    int field_34;
    int AfdTransmitIoLength;
    int field_3C;
    int field_40;
    int field_44;
    int field_48;
    int field_4C;
    int field_50;
    int field_54;
    int field_58;
    int field_5C;
    int field_60;
    int field_64;
    int field_68;
    int field_6C;
    PIRP Irp;
    PIRP Irp2;
    int field_78;
    int field_7C;
    int field_80;
    int field_84;
    int field_88;
    int field_8C;
    int field_90;
    int field_94;
};
```

然后看一眼出问题的 `struc_UnkObj->MdlAddress` 的交叉引用：

![图片描述](upload/attach/202206/156241_SPBAC8RWG687MNS.png)

发现对 `MdlAddress` 成员的写操作，均在 `AfdTransmitFile` 函数中，然后在调试器中对几次 `MdlAddress` 赋值的地方下断点，中断后情况如下：

![图片描述](upload/attach/202206/156241_26DN6RTMTNDCCZK.png)

此时看一下 `eax` 的值（`8757da60`），发现和本文一开始异常发生时，`IoFreeMdl` 尝试释放的值一致。说明我们找对分配内存的地方了：

![图片描述](upload/attach/202206/156241_XNKTYQ3FW8SJYPV.png)

其实简单一点来说就是，`struc_TPInfo`（`v6` 和 `v7`）由下图 92 行的 `AlfTliGetIpInfo` 构造并初始化、`struc_TPInfo->struc_UnkObj`（`v50` 和 `v11`）的 `MdlAddress` 成员由下图 122 行分配。

![图片描述](upload/attach/202206/156241_YSPX9PXDJM2QE9S.png)

特别强调，由于内存通过上图 92 行的 `AfdTliGetTpInfo` 分配 `struc_TPInfo` 结构后，继续由 122 行的 `IoAllocateMdl` 分配了一个初始化的 MDL 内存，因此第一次 `IoFreeMdl` 释放这块内存时是不会蓝屏的（详见下一节分析）。

## 寻找第一次释放时机

OK，既然目前内存何时分配已经清楚了，那我们来看一下这块内存第一次是何时释放的。

在 WinDbg 中下如下断点：

```text
bp nt!IoFreeMdl ".if(poi(@esp + 0x4) = 0xffffffff`8757da60){}.else{gc;}"
```

注意，这里有一个坑，由于笔者是用 64 位 WinDBG 调试 32 位系统，上面断点的值是 64 位的，实测使用 32 位的值无法中断。

中断在如下地址：
