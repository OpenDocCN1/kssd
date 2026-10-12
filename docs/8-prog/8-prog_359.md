
# BIOS Rootkit 实现分析与检测技术研究系列之－－IVT Hook 1

> 原文：[`www.pediy.com/kssd/pediy10/75604.html`](https://www.pediy.com/kssd/pediy10/75604.html)

**BIOS Rootkit 实现分析与检测技术研究系列之－－IVT Hook 1**

本节目录

1. 引言
2. IVT 和 IVT Hook 介绍
3. 使用 Bochs 调试 BIOS
4. 小结

## 引言

BIOS Rootkit 为了达到控制系统流程的目的，一般会采用 Hook IVT，即 Hook 中断向量表的方式来实现。IVT Hook 的实现和检测，涉及许多具体的问题。因此，我把 IVT Hook 分几部分来进行讨论。在本节和下面几节中，我们将讨论如何借助 Bochs 的帮助来查看原始 IVT、分析 IVT Hook 情况以及在此基础上写一个简单的 IVT 检测程序。闲话少说，这就开始我们的旅程吧。

## IVT 和 IVT Hook 介绍

### 1. 什么是 IVT

IVT（Interrupt Vector Table）即中断向量表。对于 IVT 的详细解释，这里主要参考了 Intel 公司的《Intel 64 and IA-32 Architectures Software Developer’s Manual Volume 3A: System Programming Guide》、王爽先生的《汇编语言（第二版）》和张银奎先生的著作《软件调试》（书很厚，但物超所值）。具体描述可以参见上述著作。

由于 BIOS Rootkit 存在于 BIOS 中，其初始运行环境为实模式。所以，我们讨论的 IVT 以及 IVT Hook，都是针对实模式下的 IVT 来进行的。

好，首先我们回忆一下：什么是中断？CPU 在执行指令时因为检测到预先定义的某个（或多个）条件而产生的同步事件（如除法指令的除零错误），大部分书籍称之为内中断（《软件调试》一书详细阐述了中断和异常的区别）。

CPU 收到中断信息后，需要对中断信息进行处理，而用来处理中断信息的程序称为中断处理程序。所谓中断向量就是中断处理程序的入口地址，而中断向量表（IVT）就是中断处理程序入口地址的列表。可以看到，IVT 就是一个入口地址表，如同 Windows 内核中的 SSDT 一样。对其进行 Hook 的方法与 Hook SSDT 类似。而 Hook 的目的，就是控制中断处理流程，进而控制系统的运行。

IA-32 架构规定，在实模式（Real Mode）下，IVT 在内存中保存，起始地址为 0。从内存 `0000:0000` 到 `0000:03FF` 的 1024 个内存单元中存放着中断向量表（IVT）。下图来自《Intel 64 and IA-32 Architectures Software Developer’s Manual Volume 3A: System Programming Guide》：

![](img/65606324877b94979a98cd3e34f1c417.jpg)

偷个懒，直接把《软件调试》上的相关描述进行引述，原文描述参见上面 Intel 的文献：

中断调用采用 `int n` 的形式，`n` 代表中断向量号（`n=1`，`2`，`3`……）。这样通过 `n*4` 这样的计算，可以迅速定位到各个中断号对应的内存地址。`n*4+2` 存放的是中断向量的段地址，`n*4` 存放的是中断向量的偏移地址（实模式下的汇编程序设计请参考文献）。这样 IVT 的 1024 个内存单元总共可以存放 256 个中断向量。

对于中断过程的执行步骤，描述如下：

1）将代码段寄存器 `CS` 和指令指针寄存器（`EIP`）的低 16 位（`IP`）压入堆栈。

2）将标志寄存器 `EFLAGS` 的低 16 位（`FLAGS`）压入堆栈。

3）清除标志寄存器的 `IF` 标志，以禁止其他中断。

4）清除标志寄存器的 `TF`（Trap Flag）、`RF`（Resume Flag）、`AC`（Alignment Check）标志。

5）使用向量号 `n` 作为索引，在 IVT 中找到对应的表项（`n*4+IVT` 表基地址）。

6）将表项中的段地址和偏移地址分别装入 `CS` 和 `IP` 寄存器中，并开始执行对应的代码。

7）中断例程总是以 `IRET` 指令结束。`IRET` 指令会从堆栈中弹出前面保存的 `CS`、`IP` 和标志寄存器值，然后返回执行被中断的程序。

### 2. IVT Hook

如果在第 6 个步骤，把我们自行设计的程序所在的段地址和偏移地址装入 `CS` 和 `IP` 寄存器中，CPU 将会运行我们的程序。这就是 IVT Hook 的理论基础。这里的 Hook 技术和传统 Hook 的概念没有任何区别。

## 使用 Bochs 调试 BIOS

在进行 IVT Hook 实例分析之前，先说一下我使用 Bochs 进行 BIOS 调试的一些体会，这也是进行后续讨论的实践前提。

这个系列的《简单 ISA 模块编写》一篇中，曾提醒大家进行 BIOS 刷写实验存在一定的风险。的确，一个程序编写上的小错误，或者刷新过程中电压的不稳定等等因素，都可能造成刷写的失败。而失败的结果往往都是不能正常引导计算机启动，即无法开机。在实验过程中，我把 BIOS 刷坏，开始都是抱去电脑维修店进行维修。对于刷写 BIOS 服务，我们那的当地价是 30 块钱刷一片，50 块刷两儿。刷的时候还要求你连主板一起也拆了去。费钱、费力、费时间。于是，我在淘宝上邮购了一套硬件刷写 BIOS 的编程器，问题多多，一二百的编程器还是不能和维修店里的一二千的相比。刷写无数次失败后，我决定找到一种价廉物美的实验方法。第一个感觉就是：虚拟机。

早就听说现在的虚拟机对 BIOS 能够进行仿真操作，先后从网上收集资料，然后自己实验比较。实验发现，对于主流的 VMWare、VirtualPC，对于 BIOS 实验较为困难。如 VMWare 的 BIOS 定制很麻烦，而且缺少通用性。因为和本节内容关系不大，所以具体细节以后有时间补上。众里寻他千百度，终于，第一次感受到了 Bochs 的强大。

Bochs 是很出名，这里不多介绍。大家平时多用 Bochs 进行操作系统的调试，进行 BIOS 调试的时候可能相对较少。这里，我说说对 BIOS 调试的体会。这里我使用的 Bochs 版本为 `v2.3.7`，在其帮助文档中，我们可以发现：`optromimage` 选项（It's amazing!）。这个选项允许我们对最多 4 个附加 BIOS 模块（如 ISA 模块）进行测试（当然包括调试），分别记为：`optromimage1`、`optromimage2`、`optromimage3`、`optromimage4`。下面是帮助原文：

-------------------------------------------------------------------------------------------

```
Example:    optromimage1: file=optionalrom.bin, address=0xd0000

This enables Bochs to load up to 4 optional ROM images.

Be sure to use a read-only area, typically between C8000 and EFFFF. These optional ROM images should not overwrite the rombios (located at F0000-FFFFF) and the videobios (located at C0000-C7FFF).

Those ROM images will be initialized by the BIOS if they contain the right signature (0x55AA).

It can also be a convenient way to upload some arbitrary code/data in the simulation, that can be retrieved by the boot loader
```

-------------------------------------------------------------------------------------------

这个选项应写在 `.bxrc` 配置文件中。帮助中建议我们把附加模块的加载地址设定在 `C8000` 和 `EFFFF` 之间，大家可以对照《简单 ISA 模块编写》一篇中的 BIOS 内存分布图来理解。其实，经过摸索发现，Bochs 指对设定在 `0xd0000~0xe0000` 范围之间的附加模块能够提供很好的支持，而其他范围基本都以失败告终。另外，大家还要注意一点就是：我们设计的附加模块（如 ISA 模块），除了本身大小应为 512 字节的倍数外，还要注意 `optromimage` 的支持范围，即 `0xd0000~0xe0000`。这个范围大小为 64K，因此如果你的附加模块总量，即最多 4 个附加模块的总大小不能超过 64K。否则，Bochs 会出现错误提示，并只会顺序调用在范围内的一个或几个模块。这个问题曾困扰我很久，后来反复研究 `optromimage` 的帮助终于明白。希望能给大家在做类似工作时有所帮助。

这样我们就可以把《简单 ISA 模块编写》一篇中提到的登录框程序，用 Bochs 来运行一下。配置文件相关选项就这样写：

--------------------------------------------------------------------

```
# filename of ROM images
romimage: file=../BIOS-bochs-latest
optromimage1: file=ISAOEM_NASM.bin, address=0xd0000
vgaromimage: file=../VGABIOS-lgpl-latest
```

--------------------------------------------------------------------

这里除了我们刚刚介绍过的 `optromimage`，还有 `romimage` 和 `vgaromimage`。`romimage` 允许你替换整个 BIOS 文件，这对特定 BIOS Rootkit 的编写有用，如 Kris 提到的直接修改 BIOS 入口的跳转指令：`jmp xxxx`，`xxxx` 换成我们的程序入口。这个问题可以单独进行讨论。`vgaromimage` 选项允许进行显卡 BIOS 的调试。下一步进行显卡 BIOS Rootkit 研究时，可能要借助它的帮忙。:)

Bochs 的调试命令和方法这里不多介绍，大家可以参阅 Bochs 的帮助文件。把我们的登录框程序和修改了的 `.bxrc` 配置文件放在同一目录下，运行后，我们会发现登录框程序运行。由于 Bochs 对于图形显示支持还不是很好，所以在 Bochs 中，你会发现提示输入密码的字符串没能很好显示，但这并不影响我们对功能的调试。效果图就不附上了。

## 小结

这一节我们对实模式下的 IVT 和 IVT Hook 进行了理论的铺垫，并对用 Bochs 进行 BIOS 调试的一些细节进行了探讨。在下一节中，我们来实际分析一下 IVT Hook 的应用。

待续... ![](img/c72cb8eba397ac67dda6e1ac58a9b627.jpg "微笑")

# BIOS Rootkit 实现分析与检测技术研究系列之－－IVT Hook 2

本节目录

1. 引言
2. 静态剖析 `leaving.bin`
3. 动态调试 `leaving.bin`
4. 小结
5. 附件

## 引言

上节我们对 IVT Hook 技术进行了理论铺垫，这节我们通过对实际的 BIOS Rootkit 进行分析，来加深对 IVT Hook 技术的理解。对于已发表的 BIOS Rootkit，目前只能找到的是：IceLord。IceLord 在 2007 年发表了一篇《BIOS RootKit:Welcome Home,My Lord!》的文章。文中描述了 BIOS Rootkit 实现的相关技术，并给出了实现的二进制演示文件 `IceLord.exe`（我拿到的是 `IcLord.exe`，这并不影响本文的讨论，下面论述我们姑且叫做 `IceLord.exe`）。而 `IceLord.exe` 用于嵌入 BIOS 的 ISA 模块文件为：`leaving.bin`。因此，在这个小节中我们主要对 `leaving.bin` 进行讨论。

## 静态剖析 `leaving.bin`

`leaving.bin` 是 ISA 模块文件，也是整个 `IceLord.exe` 的核心。它作为资源类型附加于 `IceLord.exe` 文件中。直接用 `IDA Pro` 加载分析，如图：

![](img/0a00cec94900cf29453c269a7dd5c42c.jpg)

可以看到 `leaving.bin` 的标准 ISA 模块头。大小为 16 进制 `40h`，即 `64*512=32K`。

在 `loc_66` 标号处，如图：

![](img/973076c0e54923edfde7984ff2d9e87a.jpg)

我们看到橙色显示的 `1600h`。用鼠标指向 `1600h` 后，`IDA` 把我们引向该字段代表的区域，如图：

![](img/e3f23a2550f7b99ee7f893bd39320235.jpg)

我们可以清楚的看到“MZ”标志。我们知道“MZ”是 Windows 的 PE 可执行文件开头的特有标志，通过分析，这正是 `IceLord` 自带的 Payload 驱动文件 `protector.sys`。为了不影响 `IDA` 对 `leaving.bin` 的分析，我们用 `WinHex` 把这个程序提取后组成一个新的 `leaving.bin` 程序，暂时取名为 `leaving_new.bin`。然后再用 `IDA` 加载进行分析。

在《BIOS RootKit:Welcome Home,My Lord!》中，作者提到，`IceLord` 控制 CPU 流程的方法是通过 hook IVT 来实现的。即通过 hook 中断 `19h` 来实现。在《IVT Hook 1》小节我们对实模式下的中断向量表 IVT（Interrupt Vector Table）进行过介绍。我们注意到，在调用具体中断的过程中，系统在第 5、6 步的操作是（请参阅《IVT Hook 1》小节）：

-------------------------------------------------------------------------------------------------

5、使用向量号 `n` 作为索引，在 IVT 中找到对应的表项（`n*4+IVT` 表基地址）。

6、将表项中的段地址和偏移地址分别装入 `CS` 和 `IP` 寄存器中，并开始执行对应的代码。

-------------------------------------------------------------------------------------------------

对于 `int 19h` 中断，系统使用向量号 `19h`（注意这里是 16 进制的 `19`），然后在 IVT 中找到表项：`19h*4+0`（IVT 默认加载到内存 `0000:0000` 处），即地址 `0000:0064h` 处。然后把 `19h` 对应的中断处理函数的段地址和偏移地址放入 `CS`、`IP` 寄存器里执行。如果我们把 `0000:0064h` 该为 `0000:xxxx`，`xxxx` 代表我们自定义的函数地址，系统就会先执行我们定义的函数，以达到控制系统流程的目的，这种方法就是典型的 Hook 法。Hook 直译为钩子，通过钩子控制程序流程的方法是一种程序员普遍使用的方法，微软公司曾发行过 `detour` 函数库来帮助程序员简化这类开发。同时，Hook 法被广泛使用在各类 Rootkit 开发当中，《ROOTKITS－－Windows 内核的安全防护》一书对此有详细描述。通过钩子来控制流程的方法同样也适用于 BIOS Rootkit（关于如何 hook 中断的汇编实现，请参阅王爽的《汇编语言（第二版）》）。

我们来看一下 `leaving.bin` 是如何实现对 `int 19h` 中断的 hook 操作的，如图：

![](img/886ec383ad375dec0984826d3a378061.jpg)

`leaving.bin` 程序首先对原始的 `int 19h` 调用地址进行保存，以待后面进行恢复调用。然后，hook 掉原有地址，加入我们的自定义函数地址。图中 `New_int19h` 即为 `leaving.bin` 作者自定义的 hook 函数，当 `int 19h` 中断被系统调用时，会首先执行 `New_int19h`，如图：

![](img/be76e3bb547d5a264d0330283b294e19.jpg)

在新的 `int 19h` 中断处理函数中，我们发现 `leaving.bin` 的作者又 hook 了 `int 13h` 中断，中断 `int 13h` 是标准的处理磁盘的 BIOS 中断类型。这种思路也符合 `IceLord` 在文献中的描述，即 hook `int 19h`，再 hook `int 13h`，以达到操作磁盘的目的。如何检测这种 hook 操作，我们在这个系列的后续小节进行讨论。

静态剖析 `leaving.bin` 的讨论就在此告一段落。

## 动态调试 `leaving.bin`



### 中断处理与分析

在新的 `int 19h` 中断处理函数中，我们发现 `leaving.bin` 的作者又 hook 了 `int 13h` 中断，中断 `int 13h` 是标准的处理磁盘的 BIOS 中断类型。这种思路也符合 `IceLord` 在文献中的描述，即 hook `int 19h`，再 hook `int 13h`，以达到操作磁盘的目的。

这种 hook `int 19h` 中断的操作是否真正执行，我们需要验证。验证的执行，我们需要借助 `Bochs` 的帮助。我们在上节介绍过 `Bochs` 强大的调试功能，它甚至能够调试 BIOS 和操作系统。在开始调试之前，我们首先来回忆一下 `Bochs` 调试器的几条常用调试命令（更详细命令使用请参阅 `Bochs` 使用帮助）：

```
b addr    在某物理地址设置断点
c         继续执行，直到遇上断点
s         单步执行
xp        查看内存内容
u         对当前地址进行反汇编
```

### 配置与加载

首先要加载我们的目标文件 `leaving.bin`。结合系列的上一篇介绍，`Bochs` 对于文件的加载是通过改写配置文件来执行的。配置文件后缀名默认为 `.bxrc`，文件由许多选项组成：如 `romimage` 选项，代表需要加载的主 BIOS 文件的位置，默认为 `file=../BIOS-bochs-latest`。又如 `boot` 选项代表操作系统加载的位置，默认为 `c`，即 `c` 盘。在这里我们需要一个特殊的选项：`optromimage` 选项。

`optromimage` 选项允许 `Bochs` 加载用户自定义的 BIOS 附加模块，如 `ISA` 模块。这正符合我们的要求。我们的 `Bochs` 配置文件的关键选项如下：

```
# filename of ROM images
romimage: file=../BIOS-bochs-latest
optromimage1: file=leaving.bin, address=0xd0000
vgaromimage: file=../VGABIOS-lgpl-latest
```

在 `Bochs` 安装目录下新建一个 `bios` 文件夹，然后把 `Bochs` 自带的 `Linux Demo` 文件夹下的所有文件拷贝一份到 `bios` 文件夹中，并把 `leaving.bin` 模块文件也拷贝一份到这里。然后改写 `.bxrc` 配置文件相应处为以上选项。特别需要加入 `optromimage` 选项，保存配置文件。在该配置文件上点击鼠标右键选择“调试”，`Bochs` 将打开调试窗口开始进行调试，如图：

![](img/c6c76ed74afccf4aa6dfabab2208f2c5.jpg)

选择默认选项 `6`，程序跳入 BIOS 开始执行，我们看到前面分析过的 `BOOTBLOCK` 中的跳转语句，这里是 `jmp far f000:e05b`，如图所示：

![](img/1449cf75515d069d956dd0a669f82cd9.jpg)

### 动态调试追踪

我们调试的目的是需要查看 `leaving.bin` 模块是否真的 hook 了 `int 19h` 中断的调用。最好的方法就是跟踪 `int 19h` 中断调用，看其是否跳入 `leaving.bin` 自定义的函数地址来。跟踪需要断点的帮忙。在对 BIOS 设置断点的实际验证中，很多在物理地址设置的断点都未能被触发。由于 `Bochs` 没有对设置中断断点的命令，所以我们在这里使用一个笨办法，就是通过 `u` 反汇编命令直接在调试窗口查看调用 `int 19h` 中断的语句，然后直接在其所在地址下断。

首先在调试窗口中输入 `s` 单步执行命令，使得程序进入 BIOS 主程序。然后我们输入命令：`u /500`，代表从当前位置开始反汇编 `500` 条语句，拖动调试窗口下拉条到顶部，我们发现了 `int 19h` 中断调用的语句地址为 `0x000fe2a7`，如图：

![](img/47f926a71285dea8765547f8d3aa4c0a.jpg)

在地址 `0x000fe2a7` 处下断：`b 0x000fe2a7`。然后输入 `c` 执行程序命令，我们成功来到了断点处，如图：

![](img/0519e9e99f566f48a52fef7399b1bd9f.jpg)

输入 `s`，并进行反汇编：`u /20`，我们看到了真实的被 `leaving.bin` hook 过的中断处理函数入口：`0x00097ca2`，如图：

![](img/bd155fd49ee7ffbd7145db280270c9cc.jpg)

对照我们在 `IDA` 静态反汇编的结果（对照上面的 `New_int19h` 反汇编结果），用 `Bochs` 动态调试得到的与其完全吻合。这也证实了 `leaving.bin` 确实如文献所说，对 `int 19h` 进行了 hook 操作，进而控制了 CPU 的执行流程。

### 特征码搜索与启发

继续剖析 `leaving.bin` 文件，文献中提到其对于操作系统改写的实现，主要是基于对 `Ntldr` 这个 Windows 启动文件特征码的搜索上，我们在 `IDA` 的反汇编结果中也得到了证实，如图：

![](img/8c012f70fa5cd2af4e7b507f17881d3f.jpg)

可以看出程序对特征码 `0ABF3C033h` 的识别工作。联系 `leaving.bin` 对于 `hook int13h` 的一些程序片段，如图：

![](img/0d972ff287930cc9a65b3c5bdb088abd.jpg)

我们不难发现，`leaving.bin` 对 `NTLDR` 文件进行内存特征码搜索的思想，受到了著名 Boot Rootkit 项目 `eEye BootRoot` 的启发。`eEye BootRoot` 项目由 `eEye` 公司安全研发人员 `Derek Soeder` 和 `Ryan Permeh` 创建，是一个旨在研究如何攻击 `MBR`（Master Boot Record）从而实现 Rootkit 的研究项目。其目的是提供一种先于操作系统执行 Rootkit 的可能性，并研究对其的检测和防御技术。`eEye BootRoot` 项目在 2005 年 `Black Hat USA` 大会上首次公布。可以访问 `eEye` 的网站进行进一步了解。

在完成特征码的搜索操作后，`leaving.bin` 会把保存过的原始 `int 13h` 中断向量值，重新写回原处，如图：

![](img/6850039a29920f466d017cec0cd09b53.jpg)

这样的写回操作，会干扰我们将在系列后续小节中编写的，IVT Hook 检测程序的检测结果，敬请期待。

动态调试 `leaving.bin` 的讨论就在此告一段落。

### 小结

在这篇文章中，我们详细讨论了 BIOS Rootkit 的 ISA 模块 `leaving.bin` 的执行情况。特别从静态和动态两个方面对 IVT Hook 技术进行了探讨。通过对 BIOS Rootkit 的实例分析，让我们对 IVT Hook 技术有了更深的理解。

### 附件

附件里附上删除了 `protector.sys` 的 `leaving_new.bin` 和 `leaving_new.idb`，供大家分析。

`附件.rar`

**PS**：为了给大家直观的印象，在这个小节特意多抓了些图片，影响网页速度了。

待续... ![](img/c72cb8eba397ac67dda6e1ac58a9b627.jpg "微笑")
