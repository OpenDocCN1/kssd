
# [翻译] 难译 | `windbg` 乐趣之道（下）

## 前言

<a href="elink@4a1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1N6$3W2@1N6r3g2J5i4K6u0W2j5$3!0E0i4K6u0r3P5h3q4J5k6r3g2F1i4K6g2X3M7$3S2S2k6X3W2J5">Yarden Shafir</a> 分享了两篇非常通俗易懂的，关于 `windbg` 新引入的调试数据模型的文章。原文链接如下：

`part1`：<a href="elink@e3aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0k6h3c8A6N6h3#2Q4x3X3g2U0L8$3#2Q4x3V1k6Q4y4o6m8&6j5i4u0V1k6h3&6K6K9r3q4X3K9i4t1J5i4K6u0r3N6$3W2F1k6r3u0Y4i4K6u0V1N6r3S2W2i4K6u0V1k6Y4g2F1i4K6u0V1N6$3q4&6i4K6u0V1M7r3q4J5N6q4)9J5k6o6q4Q4x3X3b7J5k6e0b7&6y4K6R3%4z5e0q4X3z5h3t1`."><mark class="encrypted">b7eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0k6h3c8A6N6h3#2Q4x3X3g2U0L8$3#2Q4x3V1k6Q4y4o6m8&6j5i4u0V1k6h3&6K6K9r3q4X3K9i4t1J5i4K6u0r3N6$3W2F1k6r3u0Y4i4K6u0V1N6r3S2W2i4K6u0V1k6Y4g2F1i4K6u0V1N6$3q4&6i4K6u0V1M7r3q4J5N6q4)9J5k6o6q4Q4x3X3b7J5k6e0b7&6y4K6R3%4z5e0q4X3z5h3t1`.</mark></a>

`part2`：<a href="elink@896K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0k6h3c8A6N6h3#2Q4x3X3g2U0L8$3#2Q4x3V1k6Q4y4o6m8&6j5i4u0V1k6h3&6K6K9r3q4X3K9i4t1J5i4K6u0r3N6$3W2F1k6r3u0Y4i4K6u0V1N6r3S2W2i4K6u0V1k6Y4g2F1i4K6u0V1N6$3q4&6i4K6u0V1M7r3q4J5N6q4)9J5k6o6u0Q4x3X3b7%4j5e0V1H3y4r3y4T1j5e0f1@1x3K6f1`."><mark class="encrypted">c80K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0k6h3c8A6N6h3#2Q4x3X3g2U0L8$3#2Q4x3V1k6Q4y4o6m8&6j5i4u0V1k6h3&6K6K9r3q4X3K9i4t1J5i4K6u0r3N6$3W2F1k6r3u0Y4i4K6u0V1N6r3S2W2i4K6u0V1k6Y4g2F1i4K6u0V1N6$3q4&6i4K6u0V1M7r3q4J5N6q4)9J5k6o6u0Q4x3X3b7%4j5e0V1H3y4r3y4T1j5e0f1@1x3K6f1`.</mark></a>

本文是第二部分的译文。同样在有道词典、必应词典、谷歌翻译的大力帮助下完成，感谢以上翻译工具，我只是一个搬运工。**强烈建议**英文好的朋友阅读原文，因为在翻译的过程中不可避免的按我的理解做了调整。

> 第一部分译文在<a href="elink@c30K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1K9h3q4F1j5$3S2W2L8X3N6F1j5h3&6Q4x3X3g2Y4K9i4c8W2k6g2)9J5k6h3W2G2i4K6u0r3j5i4u0@1K9h3y4D9k6i4y4Q4x3V1k6@1M7X3q4F1M7$3I4S2N6r3W2G2L8W2)9J5k6s2N6A6L8X3c8T1k6#2)9J5k6s2c8Z5k6g2)9J5k6r3k6#2L8W2)9J5k6s2N6S2P5g2)9J5k6s2m8S2M7Y4b7I4i4K6u0r3">这里</a>。

以下是译文！

---

欢迎来到我试图让您享受在 `Windows` 上调试的第 `2` 部分（哇，我真是个书呆子）！

在第一部分中，我们已经了解了新调试数据模型的基本知识——使用新对象、使用自定义寄存器、搜索和过滤输出、声明匿名类型以及解析列表和数组。在这一部分中，我们将学习如何在 `dx` 中使用传统命令（**译注：** 原文是 `legacy commands`，我理解是在引入 `dx` 之前就存在的一些命令，比如 `dt`、`u` 等），了解令人惊叹的新反汇编器，创建合成方法和类型，查看断点的奇妙变化，并在调试器中使用文件系统。

听起来有很多内容。因为确实有很多内容。我们开始吧！

## 传统命令（Legacy Commands）

新数据模型完全改变了调试体验。但有时确实需要使用我们已经习惯的旧命令或扩展（这些功能在 `dx` 中没有对应的实现）。

我们仍然可以通过 `Debugger.Utility.Control.ExecuteCommand` 在 `dx` 中使用这些旧命令，它让我们可以将传统命令作为 `dx` 查询的一部分运行。例如，我们可以使用传统的 `u` 命令查看第二个栈帧中 `RIP` 所指向的地址对应的反汇编。

由于 `dx` 输出默认是十进制的，而传统命令只接受十六进制输入，我们首先需要使用 `ToDisplayString("x")` 将其转换为十六进制：

```
dx Debugger.Utility.Control.ExecuteCommand("u " + @$curstack.Frames[1].Attributes.InstructionOffset.ToDisplayString("x"))
Debugger.Utility.Control.ExecuteCommand("u " + @$curstack.Frames[1].Attributes.InstructionOffset.ToDisplayString("x"))
[0x0] : kdnic!TXTransmitQueuedSends+0x125:
[0x1] : fffff807`52ad2b61 4883c430 add rsp,30h
[0x2] : fffff807`52ad2b65 5b pop rbx
[0x3] : fffff807`52ad2b66 c3 ret
[0x4] : fffff807`52ad2b67 4c8d4370 lea r8,[rbx+70h]
[0x5] : fffff807`52ad2b6b 488bd7 mov rdx,rdi
[0x6] : fffff807`52ad2b6e 488d4b60 lea rcx,[rbx+60h]
[0x7] : fffff807`52ad2b72 4c8b15d7350000 mov r10,qword ptr [kdnic!_imp_ExInterlockedInsertTailList (fffff807`52ad6150)]
[0x8] : fffff807`52ad2b79 e8123af8fb call nt!ExInterlockedInsertTailList (fffff807`4ea56590)
```

另一个有用的传统命令是 `!irp`。这个命令为我们提供了大量关于 `IRP` 的信息，所以没必要大费周章的使用 `dx` 重新实现它。

我们将尝试对 `lsass.exe` 进程中所有的 `IRP` 执行 `!irp` 命令。让我们看看整个过程：

首先，我们需要找到 `lsass.exe` 进程的容器。我们已经知道如何使用 `Where()` 来做到这一点。我们选择返回的第一个进程。通常应该只有一个 `lsass`，除非机器上有服务隔离。

> **译注：** 服务隔离对应的原文是 `server silos`，在网上找到一个[解释]( <mark class="encrypted">543K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2@1k6h3y4Z5L8%4m8W2k6r3W2S2i4K6u0W2j5$3!0E0i4K6u0r3k6r3g2X3K9h3&6A6N6r3W2G2L8W2)9J5c8U0t1#2z5e0x3&6i4K6u0r3M7$3W2D9L8#2)9J5z5b7`.`.</mark> ：A silo in IT is an isolated point in a system where data is kept and segregated from other parts of the architecture.

```
dx @$lsass = @$cursession.Processes.Where(p => p.Name == "lsass.exe").First()
```

然后，我们需要遍历进程中每个线程的 `IrpList` 并获得 `IRP` 本身。我们可以很容易地通过前面提到的 `FromListEntry()` 做到这一点。我们只选取包含 `IRP` 的线程：

```
dx -r4 @$irpThreads = @$lsass.Threads.Select(t => new {irp = Debugger.Utility.Collections.FromListEntry(t.KernelObject.IrpList, "nt!_IRP", "ThreadListEntry")}).Where(t => t.irp.Count() != 0)
@$irpThreads = @$lsass.Threads.Select(t => new {irp =
Debugger.Utility.Collections.FromListEntry(t.KernelObject.IrpList, "nt!_IRP", "ThreadListEntry")}).Where(t => t.irp.Count() != 0)
[0x384]
irp
[0x0] [Type: _IRP]
[<Raw View>] [Type: _IRP]
IoStack : Size = 12, Current IRP_MJ_DIRECTORY_CONTROL / 0x2 for Device for "\FileSystem\Ntfs"
CurrentThread : 0xffffb90a59477080 [Type: _ETHREAD *]
[0x1] [Type: _IRP]
[<Raw View>] [Type: _IRP]
IoStack : Size = 12, Current IRP_MJ_DIRECTORY_CONTROL / 0x2 for Device for "\FileSystem\Ntfs"
CurrentThread : 0xffffb90a59477080 [Type: _ETHREAD *]
```

我们可以在这里停一下，点击其中一个 `IRP` 的 `IoStack`（或者运行 `-r5` 来查看全部内容）可以得到调用栈信息：

```
dx @$irpThreads.First().irp[0].IoStack
@$irpThreads.First().irp[0].IoStack : Size = 12, Current IRP_MJ_DIRECTORY_CONTROL / 0x2 for Device for "\FileSystem\Ntfs"
[0] : IRP_MJ_CREATE / 0x0 for {...} [Type: _IO_STACK_LOCATION]
[1] : IRP_MJ_CREATE / 0x0 for {...} [Type: _IO_STACK_LOCATION]
[2] : IRP_MJ_CREATE / 0x0 for {...} [Type: _IO_STACK_LOCATION]
[3] : IRP_MJ_CREATE / 0x0 for {...} [Type: _IO_STACK_LOCATION]
[4] : IRP_MJ_CREATE / 0x0 for {...} [Type: _IO_STACK_LOCATION]
[5] : IRP_MJ_CREATE / 0x0 for {...} [Type: _IO_STACK_LOCATION]
[6] : IRP_MJ_CREATE / 0x0 for {...} [Type: _IO_STACK_LOCATION]
[7] : IRP_MJ_CREATE / 0x0 for {...} [Type: _IO_STACK_LOCATION]
[8] : IRP_MJ_CREATE / 0x0 for {...} [Type: _IO_STACK_LOCATION]
[9] : IRP_MJ_CREATE / 0x0 for {...} [Type: _IO_STACK_LOCATION]
[10] : IRP_MJ_DIRECTORY_CONTROL / 0x2 for Device for "\FileSystem\Ntfs" [Type: _IO_STACK_LOCATION]
[11] : IRP_MJ_DIRECTORY_CONTROL / 0x2 for Device for "\FileSystem\FltMgr" [Type: _IO_STACK_LOCATION]
```

最后，我们将遍历每个线程，并且遍历线程中的每个 `IRP`，并对其执行 `Executeccommand !IRP <irp address>`。这里我们也需要类型转换和 `ToDisplayString("x")` 来匹配传统命令所期望的格式。（`!irp` 的输出很长，所以我们将其截断，只关注感兴趣的数据）：

```
dx -r3 @$irpThreads.Select(t => t.irp.Select(i => Debugger.Utility.Control.ExecuteCommand("!irp " + ((__int64)&i).ToDisplayString("x"))))
@$irpThreads.Select(t => t.irp.Select(i => Debugger.Utility.Control.ExecuteCommand("!irp " + ((__int64)&i).ToDisplayString("x"))))
[0x384]
[0x0]
[0x0] : Irp is active with 12 stacks 11 is current (= 0xffffb90a5b8f4d40)
[0x1] : No Mdl: No System Buffer: Thread ffffb90a59477080: Irp stack trace.
[0x2] : cmd flg cl Device File Completion-Context
[0x3] : [N/A(0), N/A(0)]
...
[0x34] : Irp Extension present at 0xffffb90a5b8f4dd0:
[0x1]
[0x0] : Irp is active with 12 stacks 11 is current (= 0xffffb90a5bd24840)
[0x1] : No Mdl: No System Buffer: Thread ffffb90a59477080: Irp stack trace.
[0x2] : cmd flg cl Device File Completion-Context
[0x3] : [N/A(0), N/A(0)]
...
[0x34] : Irp Extension present at 0xffffb90a5bd248d0:
```

`!irp` 提供给我们的大部分信息都可以通过使用 `dx` 解析 `IRP` 并转储其 `IoStack` 获得。但是有一些信息如果不使用传统命令则很难获得。比如，`IrpExtension` 是否存在及其地址，以及可能与 `IRP` 关联的 `Mdl` 的信息。

## 反汇编器（Disassembler）

我们以 `u` 命令为例，下面的例子展示的是：在 `dx` 中通过 `Debugger.Utility.Code.CreateDisassember` 和 `DisassembleBlock` 创建可以遍历和搜索的反汇编代码：
