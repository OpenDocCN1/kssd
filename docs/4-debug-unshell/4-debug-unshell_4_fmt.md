
# 脱壳入门初级教学

> 原文：[`www.pediy.com/kssd/pediy10/20366.html`](https://www.pediy.com/kssd/pediy10/20366.html)

标题: 脱壳基础知识入门及 FAQ  
作者: kanxue  
时间: 2006-01-14 17:20  
链接: `http://bbs.pediy.com/showthread.php?t=20366`

**声明：本篇文章个人用户可以免费的收藏、参考，但是谢绝任何商业网站及传统媒体（如出版物）摘编转载！**

现在加解密发展己形成 2 个分支了，一个就是传统的算法，另一个就是加密壳。越来越多的软件采用了密码学相关算法，现在要做出一个软件注册机己不象前几年那么容易，这就要求解密者必须要有一定的数学功底和密码学知识，而这些在短时间内是不容易掌握的。除了密码学的应用，越来越多的软件加壳了，因此要求解密者必须掌握一些脱壳技术，这就使得壳成了解密必须迈过的一个门槛。壳发展到今天，强度越来越高了，将许多人挡在门外，使得大家望壳兴叹。另外，论坛现在两极分化比较严重，高手讨论的脱壳技术新手看不懂，很多人想学脱壳，但看到壳这么难，只好放弃了，造成新手与高手间一个断档，为了鼓励更多新人加入脱壳的行列，很有必要将壳有关知识总结一下。www.pediy.com 主页提供的教学确实有点过时了，己到非更新不可了。为此，整理这篇脱壳入门指导的文章，方便脱壳新手们学习。相对于密码学算法，脱壳并不难，只要肯花时间，短期内还是比较容易取得成绩的。

但是，不建议那些加解密刚入门，调试一个普通软件都费劲的朋友来脱壳。至少要等你有一定的调试技能再来学脱壳。也就是说必须掌握这篇文章所讲的东西：[新手入门必读（FAQ)](http://bbs.pediy.com/showthread.php?s=&threadid=769)

**在本篇文章没完成前，不欢迎转载!** 我会利用业余时间不断更新中，为保持文章连续性，因此锁帖，如学习过程中有问题请开新帖大家一起交流。**这篇入门教学将长期置顶，以代替原来脱壳版块的 FAQ，收集一些常见的脱壳问题。希望更多的朋友一起参与进来，如果想参与进来请告诉我撰写的主题，我会开通你跟此帖的权限。**

注：本人水平有限，因此这篇文章只是负责将新手领进门，更深入技术的请参考论坛中的精华文章。

代码:

```
第一课 PE 格式.................2 楼 

第二课 SEH 技术................3 楼 

第三课 认识壳.................4 楼 

第四课 常见压缩壳与加密壳......5 楼 

第五课 文件类型分析............6 楼

第六课 寻找 OEP................7 楼

第七课 Dump 内存映像............9 楼

第八课 重建输入表.............10 楼

第九课 手动确定 IAT 的地址与大小.11 楼

第十课 DLL 文件脱壳 .........12 楼

第十一课 优化与自校验去除........12 楼

```

## 第一课 PE 格式

要想学脱壳，第一步就得掌握 PE 格式，PE 是 Portable Executable File Format（可移植的执行体）简写，它是目前 Windows 平台上的主流可执行文件格式。

Microsoft Visual C++提供的 [WINNT.H](http://bbs.pediy.com/upload/bbs/unpackfaq/WINNTh.rar) 里有 PE 数据结构的完整定义。

推荐文档：

* [ah007 翻译的“PE 文件格式”1.9 版](http://bbs.pediy.com/showthread.php?threadid=21932)
* [qduwg 翻译的 PE 文件格式](http://bbs.pediy.com/showthread.php?threadid=19618)
* [Iczelion's 的 PE 文件格式](http://www.pediy.com/tutorial/chap8/Chap8-1.htm)
* [微软官方提供的 PE 文档（英文）(Revision 8.1- March 27, 2008)](http://www.microsoft.com/whdc/system/platform/firmware/PECOFF.mspx)
* [微软官方提供的 PE 文档(Revision 8.1)（zhzhtst 翻译成中文）](http://bbs.pediy.com/showthread.php?t=61497)
* [探索 PE 文件内幕 Win32 可移植可执行文件格式之旅（zhzhtst 翻译）](http://bbs.pediy.com/showthread.php?t=61498)
* [深入剖析 Win32 可移植可执行文件格式（zhzhtst 翻译）](http://bbs.pediy.com/showthread.php?t=61499)
* [PE 结构各字段偏移参考](http://bbs.pediy.com/upload/bbs/unpackfaq/ARTeam%20PE_appendix1_offsets.htm)

学习 PE 格式的方法是自己先准备一个十六进制工具，如 [HexWorkshop](http://www.pediy.com/tools/Editors.htm)，[WinHex](http://www.pediy.com/tools/Editors.htm)，用这些工具打开一个 EXE 文件对照着学。强烈推荐你用 [Stud\_PE v.2.2.0.5](http://www.pediy.com/tools/PEtools.htm) 这款工具辅助学习 PE 格式。PE 格式学习的重点是在输入表（Import Table）这块。

Stud\_PE 工具界面：

![bbs.pediy.com/upload/bbs/unpackfaq/Stud\_PE.gif](http://bbs.pediy.com/upload/bbs/unpackfaq/Stud_PE.gif)

PE 结构图：

![bbs.pediy.com/upload/bbs/unpackfaq/pe.gif](http://bbs.pediy.com/upload/bbs/unpackfaq/pe.gif)

## 第二课 SEH 技术

结构化异常处理（Structured Exception Handling，SEH）是 Windows 操作系统处理程序错误或异常的技术。SEH 是 Windows 操作系统的一种系统机制，与特定的程序设计语言无关。

外壳程序里大量地使用了 SEH，如果不了解 SEH，将会使你跟踪十分困难。

* [SEH in ASM 研究(一)by hume](http://www.pediy.com/bbshtml/bbs4/kanxue310.htm)
* [SEH in ASM 研究(二)by hume](http://www.pediy.com/bbshtml/BBS4/kanxue370.htm)
* [Structured Exception Handling](http://bbs.pediy.com/showthread.php?threadid=14042 )
* [加密与解密二版菜鸟学习笔记(2) - SEH 结构化异常处理 by ytcswb](http://bbs.pediy.com/showthread.php?threadid=10651 )

由于 [Ollydbg](http://www.pediy.com/tools/Debuggers.htm) 对 SEH 处理异常灵活，因此脱壳用 Ollydbg 会大大提高效率。

附 CONTEXT 结构环境：

代码:

```
typedef struct _CONTEXT {


/*000*/ DWORD       ContextFlags;


/*004*/ DWORD       Dr0;
/*008*/ DWORD       Dr1;
/*00C*/ DWORD       Dr2;
/*010*/ DWORD       Dr3;
/*014*/ DWORD       Dr6;
/*018*/ DWORD       Dr7;


/*01C*/ FLOATING_SAVE_AREA FloatSave;


/*08C*/ DWORD       SegGs;
/*090*/ DWORD       SegFs;
/*094*/ DWORD       SegEs;
/*098*/ DWORD       SegDs;


/*09C*/ DWORD       Edi;
/*0A0*/ DWORD       Esi;
/*0A4*/ DWORD       Ebx;
/*0A8*/ DWORD       Edx;
/*0AC*/ DWORD       Ecx;
/*0B0*/ DWORD       Eax;


/*0B4*/ DWORD       Ebp;
/*0B8*/ DWORD       Eip;
/*0BC*/ DWORD       SegCs;
/*0C0*/ DWORD       EFlags;
/*0C4*/ DWORD       Esp;
/*0C8*/ DWORD       SegSs;


/*0CC*/     BYTE    ExtendedRegisters[MAXIMUM_SUPPORTED_EXTENSION];


/*2CC*/ } CONTEXT;

```

## 第三课 认识壳

### 1. 什么是壳？

在一些计算机软件里也有一段专门负责保护软件不被非法修改或反编译的程序。它们一般都是先于程序运行，拿到控制权，然后完成它们保护软件的任务。由于这段程序和自然界的壳在功能上有很多相同的地方，基于命名的规则，就把这样的程序称为“壳”了。

推荐文档：

* [一切从“壳”开始](http://www.pediy.com/tutorial/chap8/Chap8-2.htm)

描述壳的示意图：

![bbs.pediy.com/upload/bbs/unpackfaq/intrpack.gif](http://bbs.pediy.com/upload/bbs/unpackfaq/intrpack.gif)

### 2. 壳的加载过程

这里谈的加壳工具不是 WinZIP、WinRAR 等数据压缩工具，而是谈压缩可执行文件 EXE 或 DLL 的工具。加壳过的 EXE 文件是可执行文件，它可以同正常的 EXE 文件一样执行。用户执行的实际上是外壳程序，这个外壳程序负责把用户原来的程序在内存中解压缩，并把控制权交还给解开后的真正程序，这一切工作都是在内存中运行的，整个过程对用户是透明的。

壳和病毒在某些方面比较类似，都需要比原程序代码更早的获得控制权。壳修改了原程序的执行文件的组织结构，从而能够比原程序的代码提前获得控制权，并且不会影响原程序的正常运行。

这里简单说说一般壳的装载过程。（参考了 Ljtt 以前写过的一篇文章）

1. 获取壳自己所需要使用的 API 地址

如果用 PE 编辑工具查看加壳后的文件，会发现未加壳的文件和加壳后的文件的输入表不一样，加壳后的输入表一般所引入的 DLL 和 API 函数很少，甚至只有 `Kernel32.dll` 以及 `GetProcAddress` 这个 API 函数。

壳实际上还需要其他的 API 函数来完成它的工作，为了隐藏这些 API，它一般只在壳的代码中用显式链接方式动态加载这些 API 函数:[3 个脱壳相关的重要函数介绍](http://bbs.pediy.com/showthread.php?s=&threadid=20230)

2. 解密原程序的各个区块(Section)的数据

壳出于保护原程序代码和数据的目的，一般都会加密原程序文件的各个区块。在程序执行时外壳将会对这些区块数据解密，以让程序能正常运行。 壳一般按区块加密的，那么在解密时也按区块解密，并且把解密的区块数据按照区块的定义放在合适的内存位置。

如果加壳时用到了压缩技术，那么在解密之前还有一道工序，当然是解压缩。这也是一些壳的特色之一，比如说原来的程序文件未加壳时 1～2M 大小，加壳后反而只有几百 K。

3. 重定位

文件执行时将被映像到指定内存地址中，这个初始内存地址称为基地址（ImageBase）。当然这只是程序文件中声明的，程序运行时能够保证系统一定满足其要求吗？

对于 EXE 的程序文件来说，Windows 系统会尽量满足。例如某 EXE 文件的基地址为 0x400000，而运行时 Windows 系统提供给程序的基地址也同样是 0x400000。在这种情况下就不需要进行地址“重定位”了。由于不需要对 EXE 文件进行“重定位”，所以加壳软件把原程序文件中用于保存重定位信息的区块干脆也删除了，这样使得加壳后的文件更加小巧。有些工具提供“Wipe Reloc”的功能，其实就是这个作用。

不过对于 DLL 的动态链接库文件来说，Windows 系统没有办法保证每一次 DLL 运行时提供相同的基地址。这样“重定位”就很重要了，此时壳中也需要提供进行“重定位”的代码，否则原程序中的代码是无法正常运行起来的。从这点来说，加壳的 DLL 比加壳的 EXE 更难修正。

4. HOOK-API

程序文件中的输入表的作用是让 Windows 系统在程序运行时提供 API 的实际地址给程序使用。在程序的第一行代码执行之前，Windows 系统就完成了这个工作。

壳一般都修改了原程序文件的输入表，然后自己模仿 Windows 系统的工作来填充输入表中相关的数据。在填充过程中，外壳就可填充 HOOK-API 的代码的地址，这样就可间接地获得程序的控制权。

5. 跳转到程序原入口点（OEP）

从这个时候起壳就把控制权交还给原程序了，一般的壳在这里会有明显的一个“分界线”。但现在的猛壳己没这界限了，壳里有肉，肉里有壳。

![bbs.pediy.com/upload/bbs/unpackfaq/loadpack.gif](http://bbs.pediy.com/upload/bbs/unpackfaq/loadpack.gif)

### 3. 压缩引擎

各类加壳软件，其压缩算法一般不是自己实现的，大多是调用其他的压缩引擎。目前压缩引擎种类比较多，不同的压缩引擎有不同特点，如一些对图像压缩效果好，一些对数据压缩效果好。而加壳软件选择压缩引擎有一个特点，在保证压缩比的条件下，压缩速度慢些关系不是太大，但解压速度一定要快，这样加了壳的 EXE 文件运行起来速度才不会受太大的影响。例如下面几个压缩引擎就能满足这要求：

1. `aPLib` 压缩引擎 http://www.ibsensoftware.com/，这个库对于低于 64K 的文件压缩效果较好，速度较快。
2. `JCALG1` 压缩引擎，相对于 `aPlib`，`JCALG1` 对于大文件效果好些。
3. `LZMA` 压缩引擎 http://www.7-zip.org/zh-cn/sdk.html，`LZMA` 是 `7-Zip` 程序中 `7z` 格式 的默认压缩算法，压缩率很高。

## 第四课 常见压缩壳与加密壳

加壳软件按照其加壳目的和作用，可分为两类：一是压缩（Packers），二是保护（Protectors）。压缩这类壳主要目的是减小程序体积，如 `ASPacK`、`UPX` 和 `PECompact` 等。另一类是保护程序，用上了各种反跟踪技术保护程序不被调试、脱壳等，其加壳后的体积大小不是其考虑的主要因素，如 `ASProtect`、`Armadillo`、`EXECryptor` 等。随着加壳技术的发展，这两类软件之间的界线越来越模糊，很多加壳软件除具有较强的压缩性能，同时也有了较强的保护性能。

目前流行的一些壳可以参考这里：[`www.pediy.com/tools/packers.htm`](http://www.pediy.com/tools/packers.htm)

### 1.常用压缩壳介绍

#### 1). ASPacK

主页：http://www.aspack.com/

`ASPack` 是款 Win32 可执行文件压缩软件，可压缩 Windows 32 位可执行文件（`.exe`）以及库文件（`.dll`、`.ocx`），文件压缩比率高达 40%～70%。

![bbs.pediy.com/upload/bbs/unpackfaq/aspack.gif](http://bbs.pediy.com/upload/bbs/unpackfaq/aspack.gif)

#### 2). UPX

主页：http://upx.sourceforge.net/

`UPX` 是一个以命令行方式操作的可执行文件经典免费压缩程序，压缩算法自己实现，速度极快。（开源）

#### 3). PECompact

主页：http://www.bitsum.com/

`PECompact` 同样也是一款能压缩可执行文件的工具（支持 `EXE`、`DLL`、`SCR`、`OCX` 等文件）。相比同类软件，`PECompact` 提供了多种压缩项目的选择，用户可以根据需要确定哪些内部资源需要压缩处理。同时，该软件还提供了加解密的插件接口功能。



![PECompact](http://bbs.pediy.com/upload/bbs/unpackfaq/PECompact.gif)

## 2. 加密保护壳介绍

为了保护自己的软件不轻易被他人“借鉴”，有必要对软件进行一些加密保护，而这方面目前己有成熟的专业加密软件可选择。但不要太依赖壳的保护，大多数壳是可以被攻破的，还是在自身保护上下些功夫。加密软件比较多，但在强度与兼容性上做的好的并不多，这里向大家介绍几款常见的。

现在壳的发展一个趋势就是虚拟机保护，利用虚拟机保护后，能大大提高强度，因此建议尽可能使用此类技术保护软件。如 `Themida`，`WinLicense`，`EXECryptor` 等壳带有虚拟机保护功能，因此得用好其 `SDK`。

### 1. ASProtect

主页：`http://www.aspack.com/`

`ASProtect` 是一款非常强大的 `Windows` 32 位保护工具，这 4、5 年来，其一直在更新进步。其开发者是俄国人 `Alexey Solodovnikov`。它拥有压缩、加密、反跟踪代码、反-反汇编代码、`CRC` 校验和花指令等保护措施。它使用 `Blowfish`、`Twofish`、`TEA` 等强劲的加密算法，还用 `RSA1024` 作为注册密钥生成器。它还通过 `API` 钩子（`API hooks`，包括 `Import hooks`（`GPA hook`）和 `Export hooks`）与加壳的程序进行通信。甚至用到了多态变形引擎（`Polymorphic Engine`）。反 `Apihook` 代码（`Anti-Apihook Code`）和 `BPE32` 的多态变形引擎（`BPE32` 的 `Polymorphic Engine`）。并且 `ASProtect` 为软件开发人员提供 `SDK`，实现加密程序内外结合。

![ASProtect](http://bbs.pediy.com/upload/bbs/unpackfaq/ASProtect.gif)

`ASProtect SKE` 系列己采用了部分虚拟机技术，主要是在 `Protect Original EntryPoint` 与 `SDK` 上。保护过程中建议大量里使用 `SDK`，`SDK` 使用请参考其帮助文档，在使用时注意 `SDK` 不要嵌套，并且同一组标签用在同一个子程序段里。`ASProtect` 使用相当的简单，打开被保护的 `EXE`/`DLL` 文件后，选上保护的选项。再单击菜单 `Modes`，单击 `Add Mode` 按钮，将 `Is this Mode Avtive` 选上，最后，单击 `Protection` 标签，对软件进行保护即可。`ASProtect` 加壳过程中也可外挂用户自己写的 `DLL` 文件，方法是在上图中的 `External Options` 选项加上目标 `DLL` 即可。这样，用户可以在 `DLL` 加入自己的反跟踪代码，以提高软件的反跟踪能力。

**强度评介：** 由于 `ASProtect` 名气太大，研究它的人很多，因此很容易被脱壳，不推荐使用。

### 2. Armadillo 加密壳

`Armadillo` 也称穿山甲，是一款应用面较广的壳。可以运用各种手段来保护你的软件，同时也可以为软件加上种种限制，包括时间、次数，启动画面等等！很多商用软件采用其加壳。`Armadillo` 对外发行时有 `Public`，`Custom` 两个版本。`Public` 是公开演示的版本，`Custom` 是注册用户拿到的版本。只有 `Custom` 才有完整的功能，`Public` 版有功能限制，没什么强度，不建议采用。

![Armadillo](http://bbs.pediy.com/upload/bbs/unpackfaq/image003.gif)

**强度评介：** `Armadillo` 中比较强大的保护选项是 `Nanomites` 保护（即 `CC` 保护），用的好能提高强度，其他选项没什么强度。

### 3. EXECryptor 加密壳

`EXECryptor` 也是一款猛壳，可能由于兼容性等原因，采用其保护的商业软件不是太多。这款壳的特点是 `Anti-Debug` 做的比较隐蔽，另外就是采用了虚拟机保护一些关键代码。

![EXECryptor](http://bbs.pediy.com/upload/bbs/unpackfaq/image005.gif)

**强度评介：** 用好 `EXECryptor` 虚拟机保护功能，将关键敏感代码用虚拟机保护起来，能提高强度。`EXECryptor` 壳能脱的人很多，但对付其虚拟机代码的人不多。

### 4. Themida 加密壳

`Themida` 是 `Oreans` 的一款商业壳，官方链接：`www.oreans.com`。`Themida 1.1` 以前版本带驱动，稳定性有些影响。`Themida` 最大特点就是其虚拟机保护技术，因此在程序中擅用 `SDK`，将关键的代码让 `Themida` 用虚拟机保护起来。`Themida` 最大的缺点就是生成的软件有些大。`WinLicense` 这款壳和 `Themida` 是同一公司的一个系列产品，`WinLicense` 主要多了一个协议，可以设定使用时间，运行次数等功能，两者核心保护是一样的。

![Themida](http://bbs.pediy.com/upload/bbs/unpackfaq/image007.gif)

**强度评介：** 用好其虚拟机保护功能，将关键敏感代码用虚拟机保护起来，能提高强度。

### 5. VMProtect

`VMProtect` 是一款纯虚拟机保护软件，官方链接：`www.VMProtect.ru`。它是当前最强的虚拟机保护软件，经 `VMProtect` 处理过的代码，至今还没有人公开宣称能还原。

但也有缺点，就是会影响程序速度，因此在一些对速度要求很高的场合就不适合用了。`VMProtect 1.22.3` 之前是免费版，可以支持 `EXE`，`DLL` 等文件。更高版本需要购买，其支持驱动的保护。现在流行的做法，先用 `VMProtect` 将你的核心代码处理一下，再选用一款兼容性好的壳保护。

#### (1). 关键代码自己定位

`VMProtect` 并没有提供使用说明，必须告诉 `VMProtect` 你要加密的代码具体地址，这对使用者有一定的要求，至少要懂一些跟踪技术，可以用调试器，如 `OllyDbg` 跟踪到程序需要保护的地址，然后添加地址到 `VMProtect`。

在这以一个记事本程序为例来演示一下使用方法。

运行 `VMProtect` 后，打开 `NOTEPAD.EXE` 文件。单击 `Dump` 标签，输入要加密的起始地址，光标来到要加密代码起始地址后，点击菜单“`project/new procedure`”,会出现一个新的项目，如下图。

![VMProtect](http://bbs.pediy.com/upload/bbs/unpackfaq/vm.gif)

需要处理其他地址时，请依次操作。

注意事项：

1. 用 `VMProtect` 处理，请多测试，如果不稳定，请调整被保护代码的范围。
2. `VMProtect` 对双线程支持不是太好，请同一次仅处理一个线程内的代码。

#### (2). 用 SDK 标记代码

`VMProtect v1.2` 以上支持 `SDK` 了，可以编程时插入一个标记，然后在加密时，`VMProtect` 会认出这些标记，并在有标记的地方进行保护。在程序源码中，用这对标签将一些核心代码包含，编译成 `EXE` 文件。然后用 `VMProtect` 打开 `EXE`，单击“`Project`”菜单下的“`New procedure`”或者单击工具栏中的“`New procedure`”按钮，在弹出的添加地址窗口中会自动将 `SDK` 定义代码的地址填上。然后在 `VMProtect` 的“`Options`”窗口中设置相应的选项，最后单击工具栏中的“`Compilation (F9)`”按钮，便可对目标软件进行保护。经 `VMProtect` 处理过的软件，可以继续用 `Asprotect`, `Themida` 等加壳软件进一步保护。

`VMProtect` 是当前最强的虚拟机保护软件，经过 `VMProtect` 处理的软件基本是没法分析原程序思路的，关键是用好，一定要将程序关键代码进行处理。另外，经虚拟机处理代码效率会降低，因此一些对效率要求比较高的代码就不要用 `VMProtect` 进行处理。

`Delphi` 中的标记模式

代码:

```
asm
    db $EB,$10,'VMProtect begin',0    //标记开始处.
end;

//想保护的程序代码
asm
    db $EB,$0E,'VMProtect end',0      //标记结束处.
end;
```

`VC` 的 `VMProtect` 的宏：

代码:

```
#define VMBEGIN
__asm //标记开始处.
{
  _emit 0xEB
  _emit 0x10
  _emit 0x56
  _emit 0x4D
  _emit 0x50
  _emit 0x72
  _emit 0x6F
  _emit 0x74
  _emit 0x65
  _emit 0x63
  _emit 0x74
  _emit 0x20
  _emit 0x62
  _emit 0x65
  _emit 0x67
  _emit 0x69
  _emit 0x6E
  _emit 0x00
}
//想保护的程序代码

#define VMEND
__asm //标记结束处.
{
  _emit 0xEB
  _emit 0x0E
  _emit 0x56
  _emit 0x4D
  _emit 0x50
  _emit 0x72
  _emit 0x6F
  _emit 0x74
  _emit 0x65
  _emit 0x63
  _emit 0x74
  _emit 0x20
  _emit 0x65
  _emit 0x6E
  _emit 0x64
  _emit 0x00
}
```

`VMProtect` 使用相关文档：

[VMProtect 配合 PESpin 的 SDK 加密的另一方法](http://bbs.pediy.com/showthread.php?s=&threadid=20324) 作者: `acafeel`

[VMProtect SDK+ASProtect SDK 混合编程[代码演示]](http://bbs.pediy.com/showthread.php?s=&threadid=20317) 作者: `Anskya`

[VMProtect 与 ASProtect 在 VC 中的 SDK 编程](http://bbs.pediy.com/showthread.php?s=&threadid=37477) 作者: `glucose`

## 3. 小结

更多与壳有关的描述参考：

[纵横间谁能相抗—论壳与加壳技术](http://bbs.pediy.com/showthread.php?s=&threadid=7945) 作者: `forgot`

### 常见加密壳官方站点

`ASProtect` [`www.aspack.com/`](http://www.aspack.com/)

`ACProtect` [`www.ultraprotect.com/`](http://www.ultraprotect.com/)

`Armadillo` [`www.siliconrealms.com`](http://www.siliconrealms.com/srt-news.shtml)

`EXECryptor` [`www.strongbit.com/`](http://anonymouse.org/cgi-bin/anon-www.cgi/http://www.strongbit.com/)

`Obsidium` [`www.obsidium.de/`](http://www.obsidium.de/)

`PESpin` [`pespin.w.interia.pl/`](http://pespin.w.interia.pl/)

`VMProtect` [`www.polytech.ural.ru/`](http://www.polytech.ural.ru/)

`Xtreme-Protector` [`www.oreans.com/xprotector/`](http://www.oreans.com/xprotector/)

`Themida`/`WinLicense` [`www.oreans.com`](http://www.oreans.com/downloads.php)

本站的工具栏目：[`www.pediy.com/tools/packers.htm`](http://www.pediy.com/tools/packers.htm)

## 第五课 文件类型分析

拿到一个壳，第一步就是用相关工具分析一下是什么壳，然后就可心中有数地跟踪分析。文件分析工具有 [PEID](http://www.pediy.com/tools/unpacker.htm)，[FileInfo](http://www.pediy.com/tools/unpacker.htm) 等。

### 1. PEiD

`PEiD` 的 `GUI` 界面操作非常方便直观。它的原理是利用查特征串搜索来完成识别工作的。各种开发语言都有固定的启动代码部分，利用这点就可识别出是何种语言编编译的。同样，不同的壳也有其特征码，利用这点就可识别是被何种壳所加密。`PEiD` 提供了一个扩展接口文件 `userdb.txt`，用户可以自定义一些特征码，这样就可识别出新的文件类型。

有些外壳程序为了欺骗 `PEiD` 等文件识别软件，会伪造启动代码部分，例如将入口代码改成与 `Visual C++ 6.0` 所编程程序入口处类似代码，即可达到欺骗目的。所以，文件识别工具所给出的结果只是个参考，文件是否被加壳处理过，还得跟踪分析程序代码才可得知。可参考这个文档了解如何伪装：[让侦测工具把壳识别为 VC++7.0 的源代码](http://www.pediy.com/bbshtml/BBS6/pediy6545.htm)。目前 `Hying` 的壳 `PE-Armor` 伪装能力是最强的：

![PE-Armor](http://bbs.pediy.com/upload/bbs/unpackfaq/hying.gif)

`PEiD` 分析不出类型的文件就报告是“`Nothing found *`”，如出现这情况一般都是未知壳或新版的壳。

下面 `PEiD` 识别出这个软件是用 `Asprotect 1.2x` 加的壳。

![PEiD Example](http://bbs.pediy.com/upload/bbs/unpackfaq/peid.gif)

### 2. FileInfo

`FileInfo`（简称 `Fi`）另一款不错的文件检测工具。`Fi` 运行时是 `DOS` 界面，在 `DOS` 窗口中运行程序相当不便，建议采用下面的技巧：

1. 用鼠标将文件拖到 `Fi` 主文件上。
2. 将 `Fi` 快捷方放进 `Windows` 的 `SendTo` 文件夹里.以后要分析某文件，只需右击“发送到”功能就可打开 `Fi`。

`FileInfo` 升级慢，其识别库不能自定义。而 `PEiD` 升级比较频繁，用户也可自定义特征码，因此 `PEiD` 用的比较普遍。

有时，`FileInfo` 和 `PEID` 会报“`PE Win GUI`”，`Win GUI` 就是 `Windows` 图形用户界面程序统称，表明程序可能没加壳。但不排除也有加壳的可能性，下图是一个 `ExeCryptor 2.2x` 的壳，`FileInfo` 报“`*PE Win GUI *section* ??`”，其不能识别出来。识别信息中带了个问号，表明 `FI` 对给出的结果不是太肯定。

![FileInfo Example](http://bbs.pediy.com/upload/bbs/unpackfaq/FileInfo.gif)

## 第六课 寻找 OEP

一般的压缩壳，如 `Aspack` 等都有 [专用的脱壳机](http://www.pediy.com/tools/unpacker.htm)。而加密壳（如 `ASProtect`,`Armadillo`）一般很少有脱壳机，必须手工脱壳。手工脱壳一般情况是分三步：一是查找程序的真正入口点（`OEP`）；二是抓取内存映像文件；三是输入表重建。（当然现在的加密壳复杂些，要考虑更多的东西）



# OEP 简介

OEP 是 `Original Entry Point` 缩写，即程序加壳前的真正的入口点。

外壳初始化的现场环境（各寄存器值）与原程序的现场环境是相同的。加壳程序初始化时保存各寄存器的值，外壳执行完毕，会恢复各寄存器内容。其代码形式一般如下：

```
PUSHFD ; 将标志寄存器入栈保存

PUSHAD ; push eax, ecx, edx, ebx, esp, ebp, esi, edi

…… ; 外壳代码部分

POPAD ; pop edi, esi, ebp, esp, ebx, edx, ecx, eax

POPFD ; 恢复标志寄存器

JMP OEP ;

OEP: …… ; 解压后的程序原代码
```

为了讲述方便，本节用 `UPX` 加壳的 `Win98` 记事本来演示。首先用 `PEid` 查看加壳前的记事本：

![`bbs.pediy.com/upload/bbs/unpackfaq/notepad1.gif`](http://bbs.pediy.com/upload/bbs/unpackfaq/notepad1.gif)

`PEid` 显示 `Notepad.exe` 程序是用 `Microsoft Visual C++ 6.0` 编译的，接下来用 `UPX` 来加壳，方法是开个 `DOS` 窗口，用命令 `upx notepad.exe`。如下图所示：

![`bbs.pediy.com/upload/bbs/unpackfaq/upx.gif`](http://bbs.pediy.com/upload/bbs/unpackfaq/upx.gif)

这时再用 `PEid` 查看加壳的文件，`PEid` 会给出如下信息：`UPX 0.89.6 - 1.02 / 1.05 - 1.24 -> Markus & Laszlo`

![`bbs.pediy.com/upload/bbs/unpackfaq/peid2.gif`](http://bbs.pediy.com/upload/bbs/unpackfaq/peid2.gif)

`UPX` 的壳可以用 `UPX.exe` 自身来脱，命令是：`upx -d 文件名`。一些变种的 `UPX` 壳用 `UPX.EXE` 自身脱不了，这时可以试试 [UPX ShellEx](http://www.pediy.com/tools/PACK/Packers/UPX/UPXShellEx.rar) 这款工具。

[![`bbs.pediy.com/images/attach/rar.gif`](http://bbs.pediy.com/images/attach/rar.gif) **本节实例下载：notepad.upx.rar**](http://bbs.pediy.com/upload/bbs/unpackfaq/notepad.upx.rar)

脱壳前建议用 `PE` 工具 [LordPE](http://www.pediy.com/tools/PE_tools/Lordpe/LPE-DLX.rar) 打开目标文件查看一下区块，以尽可能地多了解一些信息，对脱壳有帮助，如下图：

![`bbs.pediy.com/upload/bbs/unpackfaq/upxsection.gif`](http://bbs.pediy.com/upload/bbs/unpackfaq/upxsection.gif)

## 1.根据跨段指令寻找 OEP

推荐用 `Ollydbg` 来调试脱壳，比 `SoftICE` 和 `TRW2000` 方便多了。运行 `Ollydbg`，点击菜单“选项/调试设置”，将第一次暂停设在 `WinMain` 函数上。再用 `Ollydbg` 打开实例 `notepad.upx.exe` 就可中断在外壳的入口点处了：

![`bbs.pediy.com/upload/bbs/unpackfaq/upx_od1.gif`](http://bbs.pediy.com/upload/bbs/unpackfaq/upx_od1.gif)

上图相关代码如下：

```
0040E8C0 > 60 **pushad** //一开始 Ollydbg 就会中断这行，这个就是外壳的入口点,注意这个 pushad 指令
```

绝大多数加壳程序在被加密的程序中加上一个或多个段，所以依据跨段的转移指令（`JMP`）就可找到真正的入口点，此时就会有 `POPAD`/`POPFD` 指令出现。`UPX` 用了一次跨段的转移指令（`JMP`），在跳到 `OEP` 处会看到虚拟地址的值有一个突变，此时就能确定 `OEP` 了。

`UPX` 壳比较简单，大家不必要跟踪去找这个跨段的转移指令，中断 `WinMain` 后，只需要在 `Ollydbg` 里往下翻屏，就会发现这个跨段转移指令：

![`bbs.pediy.com/upload/bbs/unpackfaq/upx_oep.gif`](http://bbs.pediy.com/upload/bbs/unpackfaq/upx_oep.gif)

上图相关代码如下：

```
0040EA0E 61 **popad** //注意这里的 popad 指令，和开始的 pushad 对应

0040EA0F - E9 B826FFFF jmp 004010CC //这里跳到 OEP,将光标移到这，按 F4 执行到这行
```

这一句 `0040EA0F jmp 004010CC` 就是跳到 `OEP` 的指令，执行到这，`UPX` 外壳己将程序解压完毕，并模拟 `Windows` 加载器的将原始程序加载到内存，`004010CC` 就是映射到内存目标程序的入口点，此时就可抓取内存映像文件了。

## 2.根据堆栈平衡原理找 OEP

这个堆栈平衡原理其找 `OEP` 原理这篇文档描述的比较详细：[**寻找真正的入口（OEP）--广义 ESP 定律** 作者:Lenus](http://www.pediy.com/bbshtml/BBS6/pediy6083.htm)

操作方法：多数壳在运行到 `OEP` 的时候 `ESP=0012FFC4`，这就是说程序的第一句是对 `0012FFC0` 进行写入操作，只要在 `0012FFC0` 下硬件写入断点（命令行里键入 `HW 12FFC0`），我们就能停在 `OEP` 的第二句处。

**注意：** 并不是所有程序加载时，`ESP` 的值是 `0012FFC4`，这个值是什么由操作系统决定，将 `SizeOfStackCOmmit` 改大 `ESP` 就会变，这是因为操作系统从这个页向上找一个足够大地方当作 `stack` 了（感谢 `forgot` 解释）。你只要记住你系统加载时的 `ESP` 值，对其设断即可。

用 `OllyDBG` 重新加载实例程序 `notepad.upx.exe`，在命令行下硬件写断点：

![`bbs.pediy.com/upload/bbs/unpackfaq/hwod.gif`](http://bbs.pediy.com/upload/bbs/unpackfaq/hwod.gif)

按 `F9` 执行程序，就会中断在 `OEP` 第二行：

![`bbs.pediy.com/upload/bbs/unpackfaq/upx_oep2.gif`](http://bbs.pediy.com/upload/bbs/unpackfaq/upx_oep2.gif)

此时如果将光标向上移，会发现第一句代码变乱了：

```
004010C7 000D 0A000055 add [5500000A], cl

004010CD 8BEC mov ebp, esp
```

这是因为 `Ollydbg` 将数据当汇编代码来分析了，你可以按 `Ctrl＋ALT＋向上光标键` 将当前显示的代码向上滚动一个字节就可看到正确的汇编代码了：

```
004010CC 55 push ebp

004010CD 8BEC mov ebp, esp //中断在这行

004010CF 83EC 44 sub esp, 44

004010D2 56 push esi

004010D3 FF15 E4634000 call [4063E4] ; kernel32.GetCommandLineA
```

中断后，别忘点击菜单“调试/硬件断点/”打开硬件断点面板，将刚才的硬件断点删除。

![`bbs.pediy.com/upload/bbs/unpackfaq/upx-bchw.gif`](http://bbs.pediy.com/upload/bbs/unpackfaq/upx-bchw.gif)

小知识： [**硬件断点的原理** 作者:Lenus](http://bbs.pediy.com/showthread.php?threadid=10829)

## 3.根据编译语言特点找 OEP

各类语言编译的文件入口点都有一些规律，可以这利用这点来寻找入口点。

### 1）Delphi 程序

执行程序，用 `LordPE`（或 `Prodump`）选 `dump(full)` 脱壳，存为 `dump.exe`。接着用 `Hex Workshop` 打开 `dump.exe`，搜索文本“`runtime`”，搜到后，向前查找离“`runtime`”最近的十六进制数字“`55 8B EC`”，数字所在的地址就是程序的 `OEP`。

### 2）Visual C 程序

可以利用 `Visual C` 启动部分的几个函数 `GetCommandLineA(W)`、`GetVersion`、`GetModuleHandleA(W)`、`GetStartupInfoA(W)` 等来定位程序的 `OEP`。

常见的各类编译语言的入口汇编代码都要熟悉，因为一些加密强壳会偷 `OEP` 处的代码到壳里，一般情况各编译语言入口代码都相同，到时只需要直接引用相关程序的入口代码，这给我们恢复代码带来方便。

## 4.用内存断点找 OEP

`Author`：`Lenus`

`From`: [`www.popbase.net`](http://www.popbase.net) & [`www.pediy.com`](http://www.pediy.com)

`E-mail`：`Lenus_M@163.com`

---

### 1.前言

发现论坛中很多兄弟在询问：什么是二次内存断点，三次内存断点。还有很多人对内存断点的原理不是很明白。其实只要懂得壳是如何解压代码的，那么就完全可以按自己的喜欢来下断。

本文要解决的问题是：

1. 什么是内存断点？
2. 如何在寻找 `OEP` 时使用内存断点。
3. 内存断点的局限性。

### 2.内存断点寻找 OEP 的原理

i.首先，在 `OD` 中内存断点，硬件断点和普通断点（`F2` 下断）是有本质区别的。硬件断点等效与 `SoftICE` 命令 `bpm`，他的中断要用到 `DR0-DR7` 的调试寄存器，也就是说 `OD` 通过这些 `DR0-DR7` 的调试寄存器来判断是否断下。

普通断点（`F2` 下断）等效于 `bpx`，他是在所执行的代码的当前地址的一个字节修改为 `CC`（`int3`）。当程序运行到 `int3` 的时候就会产生一个异常，而这个异常将交给 `OD` 处理，把这个异常的 `regEIP-1` 以后就正好停在了需要的中断的地方（这个根据系统不同会不一样），同时 `OD` 在把上面的 `int3` 修改回原来的代码。

而内存断点基本上使用的是对代码使用的保护属性来实现中断。

内存断点分为：内存访问断点，内存写入断点。

我们知道，在程序运行的时候会有 3 种基本的状态产生：读取，写入，执行。

```
004AE242     A1 00104000       mov eax,dword ptr ds:[004AE24C]            //004AE24C 处的内存读取

004AE247     A3 00104000       mov dword ptr ds:[004AE24C],eax            //004AE24C 处的内存写入

004AE24C     83C0 01          add eax,1                                   //004AE24C 处的内存执行
```

那么我们应该如何中断在上面的几行呢？

1. 当我们对 `004AE24C` 下内存访问断点的时候，可以中断在 `004AE242` 也可以中断在 `004AE247`。
2. 当我们对 `004AE24C` 下内存写入断点的时候，只能中断在 `004AE247`。
3. 当我们对 `004AE24C` 下内存访问断点的时候，能中断在 `004AE24C`。

到这里你可能不明白了，为什么内存访问断点能中断在 `004AE247` 这一句对 `004AE24C` 的写入，而且还能中断在 `004AE24C` 的执行呢？

其实很简单，我们只要仔细体会一下“内存访问”这四个字的含义遍可以知道，当我们对 `004AE24C` 进行读取的时候需要“访问”他吧，当我对 `004AE24C` 进行写入的时候也需要“访问”他吧！！当然我们要执行内存地址 `004AE24C` 的代码的时候也是还是要“访问”他的！

所以我们不难得出下面的结论：

1. 内存写入中断的地方，一定是也可以用内存访问中断。
2. 内存执行的地方，也可以用内存访问中断。

如果这时你认为，那么内存写入岂不是没用了。呵呵~那我要告诉你当然不是，如果你想快速的准确的定位到 `004AE247` 这一行的时候，那么他就大有作用了！

总结一下：内存断点不修改改原代码，不会像普通断点那样因为修改代码被程序校验而导致中断失败；对于区段的访问只是区域大了一点，其原理和上面分析的三行代码是一样的。

ii.如何使用内存断点来寻找 `OEP` 呢？

要回答这个问题首先要回答这一个问题：壳是如何解压代码的？

正如我们知道的，壳如果要把原来加密或压缩的代码运行起来就必须要解压和解密原来的代码。而这一个过程我们难道不能将他看做是对代码段（`code` 段）的写入吗？好了，解压完毕了。我们要从壳代码的区段 `JMP` 到原来的代码段的时候，难道不正是对代码段（`code` 段）的执行吗？

理清了上面的关系就好办了，那么如果载入 `OD` 后，我们直接对 `code` 段下内存访问断点的时候，一定会中断在壳对 `code` 段的写入的代码的上面，就像上面的 `004AE247` 的这一行。而如果当他把 `code` 段的代码全部解压解密完毕了以后，`JMP` 到 `OEP` 的时候，我们是不是还可以停在 `OEP` 的代码上面呢？而且每按下 `F9` 都会中断，因为这时 `code` 段在执行中哦！

相信很多人到这里已经明白了，为什么在教程中到达了某一个时候，某一行的时候。牛人们就叫我们对 `code` 段下内存访问断点了吧。

而如果你还要继续问我为什么一定要到那个地方才可以下断呢？我难道不可以一开始就下断吗？

正入我上面所说的，如果你在前面下断很可能壳对 `code` 段还没解压完毕呢，这时如果你不停的按 `F9`，你将会看到 `OD` 的下方不断的在提示你“对 `401000` 写入中断” “对 `401002` 写入中断”“对 `401004` 写入中断”.......如果你不介意按 `F9` 到他把正个 `code` 段写完的话，我除了同情你的“`F9`”以外，没什么其他的意见！

那么我们就没有别更快一点的办法了吗？

有的！那就是我们呼之欲出的两次内存断点办法。

怎么理解两次内存断点呢？

让我来做一个假设吧，假设我是一个壳的作者。一个 `EXE` 文件的有 `code` 段，`data` 段，`rsrc` 段.....依次排列在你的内存空间中，那么我会怎么解码呢？呵呵~我比较笨一点，我会先将 `code` 段解码，然后再将 `data` 段解压，接着是 `rsrc` 段......那么聪明的你不难发现，只要你在 `data` 断或者 `rsrc` 段下内存访问断点，那么中断的时候 `code` 段就已经解压完毕了。这时我们再对 `code` 段下内存反问断点，不就可以到达 `OEP` 了吗？

这里注意上面虽然下了两次内存访问断点，但是本质是不一样的，目的也是不一样的。

1. 对 `data` 段下内存访问断点而中断是因为内存写入中断，目的是断在对对 `data` 段的解压时，这时壳要对 `data` 段写数据，但是 `code` 段已经解压完毕。
2. 对 `code` 段下内存访问断点而中断是因为内存执行中断，目的当然就是寻找 `OEP` 了。

总结一下：如果我们知道壳在什么地方对 `code` 段解压完毕我们就可以使用内存断点，找到 `OEP`。如果不知道，那么我们就依靠 2 次内存断点去找，如果还不行就用多次内存断点。总之明白了原理在多次的内存断点其实都一样。从这个过程中我们了解的是壳在对区段解码的顺序！

iii.实战

说了这么多，我想大家都越越欲试了吧。

好吧，来弄一个猛壳怎么样：

**注：本节实例有些难度，不适合新手练习，新手可以跳过这个实例的学习，等找到合适的实例会补充上来的**

![`bbs.pediy.com/images/attach/rar.gif`](http://bbs.pediy.com/images/attach/rar.gif) [**本节实例下载：unpackme.rar**](http://bbs.pediy.com/upload/bbs/unpackfaq/unpackme.rar)

这个壳是一个 `hying` 的旧版，我们用他来实验一下我们内存断点法。

`OD` 载入以后来到这里

```
0040D000 u>  56                      push esi               //这里

0040D001     52                      push edx

0040D002     51                      push ecx

0040D003     53                      push ebx

0040D004     55                      push ebp

0040D005     E8 15010000            call unpackme.0040D11F
```

根据跟过一次的经验我们将先设置，除 `int3` 异常以外忽略其他异常，`SHIFT+F9`



### 内存断点法脱壳示例

![bbs.pediy.com/upload/bbs/unpackfaq/codeoep.gif](http://bbs.pediy.com/upload/bbs/unpackfaq/codeoep.gif)

```assembly
003725B1    64:8925 0000000>      mov     fs:[0], esp  
003725B8    CC                    int3  
003725B9     90                   nop                  //到这里  
003725BA     8BCD                mov ecx,ebp  
```

然后再设置除“除零”异常外，忽略其他异常。`SHIFT+F9`

![bbs.pediy.com/upload/bbs/unpackfaq/codeoep1.gif](http://bbs.pediy.com/upload/bbs/unpackfaq/codeoep1.gif)

```assembly
00372660     F7F3               div ebx                //到这里  
00372662     90                nop  
```

下面是很多的单步异常，太麻烦我们不管他，现在开始用内存断点的方法（记得将所有异常忽略）。

对 `code` 段下内存访问断点，希望他已经解压完毕。方法是按 `ALT+M` 键打开内存窗口，在 `.code` 段按 `F2` 设断：

![bbs.pediy.com/upload/bbs/unpackfaq/codeoep2.gif](http://bbs.pediy.com/upload/bbs/unpackfaq/codeoep2.gif)

`SHIFT+F9` 执行：

```assembly
0040D19D     A4                movs byte ptr es:[edi],byte ptr ds:[esi]     //还没解完呢  
0040D19E     B3 02             mov bl,2  
```

对 `data` 段下内存“写入”断点，试试看他是不是要写 `data` 段。

```assembly
00372712     F3:A4             rep movs byte ptr es:[edi],byte ptr ds:[esi]    //断到这里  
00372714     5E                pop esi  
```

下面再对 `code` 段下内存访问断点。`F9`

```assembly
00372855     8907              mov dword ptr ds:[edi],eax              ; SHELL32.DragFinish  //这里是对 IAT 加密  
                                                                   //的地方了！！！  
00372857     5A                pop edx  
00372858     0FB642 FF         movzx eax,byte ptr ds:[edx-1]  
0037285C     03D0              add edx,eax  
0037285E     42                inc edx  
0037285F     83C7 04           add edi,4  
00372862     59                pop ecx  
00372863   ^ E2 A9             loopd short 0037280E  
00372865   ^ E9 63FFFFFF       jmp 003727CD  
0037286A     8BB5 93060000     mov esi,dword ptr ss:[ebp+693]           //到这里下断 F2  
```

现在如果再对 `data` 下访问断点已经是没用了。这时应该格外的小心。

我们现在就想既然这一段是对 `code` 解码的，那么我们就绕过他吧！

到 `0037286A` 下断 `F2`，然后清除内存断点！！！！

`F9` 以后停在这里，继续对 `code` 下内存访问断点。

看看左下角还在解码，哎~真是麻烦！

```assembly
003728E1    /EB 1D             jmp short 00372900  
003728E3    |25 FFFFFF7F       and eax,7FFFFFFF  
003728E8    |0385 83060000     add eax,dword ptr ss:[ebp+683]  
003728EE    |2B85 8F060000     sub eax,dword ptr ss:[ebp+68F]  
003728F4    |8BDE              mov ebx,esi  
003728F6    |2BD8              sub ebx,eax  
003728F8    |8958 FC           mov dword ptr ds:[eax-4],ebx           //停在这里  
003728FB    |83C7 08           add edi,8  
003728FE   ^|EB DB             jmp short 003728DB  
00372900    \\64:FF35 30000000  push dword ptr fs:[30]               //清除内存断点以后到这里下断，F9  
```

又是一段解码的代码，再次使用上面的办法手动跳出去。

现在继续对 `code` 段下内存访问断点！！`F9` 以后到达这里。

```assembly
004010CC     FFD7               call edi                                ; unpackme.004010CE   //OEP 哦  
004010CE     58                pop eax  
004010CF     83EC 44           sub esp,44  
004010D2     56                push esi  
004010D3     90                nop  
004010D4     E8 B518F7FF       call 0037298E  
004010D9     8BF0              mov esi,eax  
```

呵呵~虽然不是我们熟悉的 `OEP`，但是地址是没错了，况且根据我们的步骤，我可以很肯定的说这是 `code` 段的第一次“执行”中断！

所以这就是 `OEP` 了。

总结一下：当我们在寻找 `OEP` 的时候，要多次对 `code` 下断“赌”一“赌”他解压完毕，如果不是就对别的段试试~如果程序跑飞了，那就没办法了，重来呗~其实说起来要赌的是：当 `data` 段，`idata` 段，`rsrc` 段摆在你的面前，你会好好“珍惜”那个段，不过还好上天还会给我们从来一次的机会（`ctrl+F2` ^_^），那么我们会对那个不会跑飞的段说 3 个字----“先断你”如果非要在上面加一个次数，我希望是“一次内存断点就好了”

### 内存断点的局限性

#### 情况 1：UPX 壳

是不是什么壳都可以用内存中断啊？

不是每个都可以的，一些像 `UPX` 和 `ASPACK` 就不行。

为什么？

呵呵~follow me！

我们来看看 `UPX` 的壳

首先，他的壳代码在 `UPX1` 段。

这里是他要跳到 `OEP` 的地方

```assembly
0040ED4F    /77 11            ja short NOTEPAD_.0040ED62      
0040ED51    |01C3            add ebx,eax  
0040ED53    |8B03            mov eax,dword ptr ds:[ebx]  
0040ED55    |86C4            xchg ah,al  
0040ED57    |C1C0 10         rol eax,10                   //在解码  
0040ED5A    |86C4            xchg ah,al  
0040ED5C    |01F0            add eax,esi  
0040ED5E    |8903            mov dword ptr ds:[ebx],eax  
0040ED60   ^|EB E2           jmp short NOTEPAD_.0040ED44  
0040ED62    \\24 0F          and al,0F  
0040ED64     C1E0 10         shl eax,10  
0040ED67     66:8B07         mov ax,word ptr ds:[edi]  
0040ED6A     83C7 02         add edi,2  
0040ED6D   ^ EB E2           jmp short NOTEPAD_.0040ED51        //回跳解码  
0040ED6F     61                popad  
0040ED70   - E9 5723FFFF      jmp NOTEPAD_.004010CC             //跳到 OEP  
```

我们看到他在对 `code` 段解压完毕的时候马上就 `JMP` 到 `OEP` 去了，那么我们根本就来不及使用内存断点的办法。

你可能说，我可以在

```assembly
0040ED6F     61                popad //这一句下断然后使用啊  
```

呵呵~~当然可以，不过你把花在下内存断点的时间，多按下几次 `F8` 不更好？！

也就是说当一个壳如果他在 `JMP` 到 `OEP` 前的一行代码仍在都在对 `code` 段解压，那么我们就不能再使用这种办法了！

或者说我们没必要使用内存断点更贴切一点！

#### 情况 2：OEP 处有 stolen code 的代码

对于一些在 `OEP` 处有 stolen code 的代码

我们来看看一个 `OEP`

```assembly
0049E2F4 u>  55                push ebp                     //OEP  
0049E2F5     8BEC              mov ebp,esp  
0049E2F7     83C4 F4           add esp,-0C  
0049E2FA     B8 BCE04900       mov eax,unpack.0049E0BC  
0049E2FF     E8 048CF6FF       call unpack.00406F08                   //这里调用子程序  
0049E304     A1 B8FE4900       mov eax,dword ptr ds:[49FEB8]  
0049E309     50                push eax  
0049E30A     6A 00             push 0  
0049E30C     68 1F000F00       push 0F001F  
0049E311     E8 E68EF6FF       call <jmp.&kernel32.OpenFileMappingA>  //API  
0049E316     A3 60194A00       mov dword ptr ds:[4A1960],eax  
0049E31B     833D 60194A00 00  cmp dword ptr ds:[4A1960],0  
```

这个软件在被 `PESPIN` 加壳了以后这些全被偷掉了！

也就是说，壳在模拟 `OEP` 代码的时候必然会执行

```assembly
0049E2FF     E8 048CF6FF       call unpack.00406F08  //这一步  
```

而这个地方是 `call` 向 `code` 段的。如果我们使用内存访问断点，那么就停在这个子程序的地方

```assembly
00406F08     50                push eax                          //会停在这里  
00406F09     6A 00             push 0  
00406F0B     E8 F8FEFFFF       call <jmp.&kernel32.GetModuleHandleA>  
00406F10     BA 04F14900       mov edx,unpack.0049F104  
00406F15     52                push edx  
```

这里既不是处理 stolen code 的地方，也不是 `FOEP` 的地方。这就会对我们的判断产生误导。

当然你可以 `alt+F9` 返回到壳处理 stolen 的地方，然后用内存断点，或者按几下 `F8` 到达 `FOEP` 处，但试问如果你拿到一个未知的壳的时候又怎么知道应该这么处理呢？

还有其他一些情况留给大家总结吧！

在下的砖已抛出，各位的玉不久矣。

--------------------------------------------------

### 总结

好了说了很多，大家应该对内存断点的办法有了全面的了解，如果了解了内存断点的原理就不难明白他的使用方法，不难明白为什么有写壳不能使用内存断点的办法，其实任何的一种办法都需要经验的积累。相信如果大家在回答开篇的 3 个问题，已经不难了。

大家可以结合原理再好好的体会一下《手动脱壳进阶第八篇 Skvp1.32》这篇文章。

## 第七课 Dump 内存映像

外壳程序解压还原后就会跳到 `OEP` 处执行，此时内存映像文件是己解压的程序。这时就可抓取内存映像文件了（该过程称为 `Dump`）。当然不一定非要在程序原入口点抓取，只要能保证内存映像文件是己还原的就行了。

有关 `Dump` 技术的原理大家可以参考：[浅谈脱壳中的 Dump 技术 作者:Lenus](http://bbs.pediy.com/showthread.php?threadid=17624)

继续上一个实例 `notepad.upx.exe`，到 `OEP` 后就可以 `Dump` 取内存映像文件：

```assembly
004010CC    55             push    ebp  
004010CD    8BEC           mov     ebp, esp  
004010CF    83EC 44        sub     esp, 44  
```

运行 `LordPE.EXE`，点击 `Options`，默认选项如下：

![bbs.pediy.com/upload/bbs/unpackfaq/upx-lordpe.gif](http://bbs.pediy.com/upload/bbs/unpackfaq/upx-lordpe.gif)

默认选上“Full dump:paste header from disk”，`PE` 头的信息直接从磁盘文件获得。设置好后，在 `LordPE` 的进程窗口选择 `notepad.upx.exe`，点击右键，执行“dump full”菜单命令。如图：

![bbs.pediy.com/upload/bbs/unpackfaq/upxlordpedump.gif](http://bbs.pediy.com/upload/bbs/unpackfaq/upxlordpedump.gif)

将内存抓取的文件另存为 `dumped.exe`，此时程序还不能运行，接下来就是重建输入表。

## 第八课 重建输入表

在脱壳中输入表处理是很关键的一个环节，因此要求脱壳者对 `PE` 格式中的输入表概念非常清楚。在磁盘文件中，`PE` 文件的输入表结构如下图所示：

![bbs.pediy.com/upload/bbs/unpackfaq/IID.gif](http://bbs.pediy.com/upload/bbs/unpackfaq/IID.gif)

图 8.1 磁盘文件中的输入表

`PE` 文件运行时，`Windows` 系统加载器首先搜索 `OriginalFirstThunk`，如果存在，装载程序迭代搜索数组中的每个指针，找到每个 `IMAGE_IMPORT_BY_NAME` 结构所指向的输入函数的地址，然后用函数入口地址来替代由 `FirstThunk` 指向的 `IMAGE_THUNK_DATA` 数组里的元素值（即用真实的函数地址填充到 `IAT` 里）。因当 `PE` 文件装载内存后准备执行时，上图己转换成这种情况了：

![bbs.pediy.com/upload/bbs/unpackfaq/IAT.gif](http://bbs.pediy.com/upload/bbs/unpackfaq/IAT.gif)

图 8.2 `PE` 文件装载内存后的输入表



此时输入表中其它部分就不重要了，程序依靠 `IAT` 提供的函数地址就可正常运行（图 8.2 红圈部分）。如果程序加壳了，那壳自己模仿 Windows 装载器的工作来填充 `IAT` 中相关的数据，此时内存中就一张 `IAT` 表，输入表的其他部分是不存在的（当然不是绝对的，也有不少壳，如 `Aspack` 等，内存中会出现完整的输入表结构），如图 8.3 所示。

`![bbs.pediy.com/upload/bbs/unpackfaq/iatpack.gif](http://bbs.pediy.com/upload/bbs/unpackfaq/iatpack.gif)`

图 8.3 外壳加载程序后的内部 `IAT`

输入表重建就是根据图 8.3 这张 `IAT` 恢复整个输入表的结构（即图 8.1 这个结构），`ImpREC` 这款工具就是这个功能。

一些压缩壳，填充 `IAT` 过程中没做什么手脚，用 `ImpREC` 工具可以直接重建输入表。而一些加密壳为了防止输入表被还原，就在 `IAT` 加密上大作文章，此时壳填充 `IAT` 里的不是实际的 `API` 地址，而是填充壳中用来 `HOOK-API` 的外壳代码的地址。这样壳中的代码一旦完成了加载工作，在进入原程序的代码之后，仍然能够间接地获得程序的控制权。 因为程序总是需要与系统打交道，与系统交道的途径是 `API`，而 `API` 的地址已经替换成了壳的 `HOOK-API` 的地址，那程序每一次与系统打交道，都会让壳的代码获得一次控制权，这样壳可以进行反跟踪继续保护软件，同时也可完成某些特殊的任务。所以`重建输入表的关键是获得没加密的 IAT`，一般的做法是跟踪加壳程序对 `IAT` 处理过程，修改相关指令，不让外壳加密 `IAT`。

`UPX`、`ASPack` 等加壳保护的壳没加密 `IAT`，而 `ASProtect`、`tElock` 等加密保护的壳都对 `IAT` 进行了加密处理。这篇先来简单的，即 `UPX` 壳。用 `OD` 打开上面的 `notepad.upx.exe` 实例，运行到 `OEP`。（实际跟踪过程中，不一定要到 `OEP`，只要外壳处理完 `IAT` 就可）然后如下操作：

1. 运行 `ImportREC`，在下拉列表框中选择 `notepad.upx.exe` 进程，如图：

`![bbs.pediy.com/upload/bbs/unpackfaq/ImportREC1.gif](http://bbs.pediy.com/upload/bbs/unpackfaq/ImportREC1.gif)`

2. 上面己得知 `notepad.upx.exe` 的 `OEP` 地址是 `4010CC`，则在左下角 `OEP` 处填入 `OEP` 的 `RVA` 值，这里填上 `10CC`。点击“`IAT AutoSearch`”按钮，让其自动检测 `IAT` 偏移和大小，如出现下图表示 `ImportREC` 自己找到 `IAT` 的地址与大小了，即 `IAT` 地址：`000062E0`，大小 `248`。

`![bbs.pediy.com/upload/bbs/unpackfaq/ImportREC2.gif](http://bbs.pediy.com/upload/bbs/unpackfaq/ImportREC2.gif)`

如果 `ImportREC` 没找到 `IAT` 偏移，则必须手工填入 `IAT` 偏移和大小（`IAT` 偏移手动获得以后再讲述）。

3. 点击“`Get Import`”按钮，让其分析 `IAT` 结构得到基本信息，如下图所示：

`![bbs.pediy.com/upload/bbs/unpackfaq/ImportREC3.gif](http://bbs.pediy.com/upload/bbs/unpackfaq/ImportREC3.gif)`

4. 如发现某个 `DLL` 显示"valid :NO" ，按"Show Invalids"按钮将分析所有的无效信息，在 `Imported Function Found` 栏中点击鼠标右键，选择"Trace Level1 (Disasm)"，再按"Show Invalids"按钮。如果成功，可以看到所有的 `DLL` 都为"valid:YES"字样；

5. 再次刷新"Show Invalids"按钮查看结果，如仍有无效的地址，继续手动用右键的 `Level 2` 或 `3` 修复；

6. 如还是出错，可以利用" Invalidate function(s)"、"Delete thunk(s)"、编辑 `Import` 表（双击函数）等功能手动修复。

7. 开始修复已脱壳的程序。选择 `Add new section` (缺省是选上的) 来为 `Dump` 出来的文件加一个 `Section`(虽然文件比较大，但避免了许多不必要的麻烦) 。

8. 单击"Fix Dump"按钮，并选择刚在前面己 `Dump` 出来的文件。如修复的文件名是"Dump.exe"，它将创建一个"Dump\_.exe"，此外 `OEP` 也被修正。

经过这些步骤，这个 `UPX` 壳己成功脱掉。此时再用 `PEID` 查一下脱壳后的程序 `dumped\_.exe`，会显示是“Microsoft Visual C++ 6.0 SPx Method 1”，如下图所示：

`![bbs.pediy.com/upload/bbs/unpackfaq/peid3.gif](http://bbs.pediy.com/upload/bbs/unpackfaq/peid3.gif)`

再用 `LordPE` 查看脱壳后的输入表：

`![bbs.pediy.com/upload/bbs/unpackfaq/importupx.gif](http://bbs.pediy.com/upload/bbs/unpackfaq/importupx.gif)`

从上图可以看出，输入表己正确修复，此时脱壳后的文件己能成功运行了。

## 第九课 手动确定 IAT 的地址与大小

在第八课中讲到，点击 `ImportREC` 的“`IAT AutoSearch`”按钮，一般情况下 `ImportREC` 可以自动识别出 `IAT` 地址与大小。但如果不能自动识别，就必须手动确定 `IAT` 地址与大小，然后将 `IAT` 的 `RVA` 与 `Size` 填进 `ImportREC`，点击“`Get Import`”按钮就可得到输入表。

还是用上一节实例演示，用 `OD` 打开 `notepad.upx.exe`，来到 `OEP` 处：

`![bbs.pediy.com/upload/bbs/unpackfaq/9-iat.gif](http://bbs.pediy.com/upload/bbs/unpackfaq/9-iat.gif)`

随便找一个 `API` 函数调用语句，如：

```
004010D3    FF15 E4634000   call    [4063E4]           ; kernel32.GetCommandLineA
```

其中地址 `4063E4` 就是 `IAT` 中的一部分，在数据窗口下命令：`D 4063E4`，显示如下：

`![bbs.pediy.com/upload/bbs/unpackfaq/9-iat2.gif](http://bbs.pediy.com/upload/bbs/unpackfaq/9-iat2.gif)`

上图每一组数据都是指向一个 `API` 函数，如 `8D 2C 81 7C` 就是地址：`7C812C8D`，在 `OD` 里按 `Ctrl＋G`，输入 `7C812C8D` 跳到这个地址就会发现是 `kernel32.GetCommandLineA` 函数：

`![bbs.pediy.com/upload/bbs/unpackfaq/9-iat5.gif](http://bbs.pediy.com/upload/bbs/unpackfaq/9-iat5.gif)`

`![bbs.pediy.com/upload/bbs/unpackfaq/9-iat6.gif](http://bbs.pediy.com/upload/bbs/unpackfaq/9-iat6.gif)`

`IAT` 是一块连续排列的数据，因此在数据窗口向上翻屏，直到出现 `00` 数据，寻找 `IAT` 起始地址：

`![bbs.pediy.com/upload/bbs/unpackfaq/9-iat3.gif](http://bbs.pediy.com/upload/bbs/unpackfaq/9-iat3.gif)`

然后向下翻屏，寻找 `IAT` 结束地址：

`![bbs.pediy.com/upload/bbs/unpackfaq/9-iat4.gif](http://bbs.pediy.com/upload/bbs/unpackfaq/9-iat4.gif)`

为了直观些，你也可以这样让数据窗口直接显示这些 `API` 函数，以确定 `IAT` 是否正确，在数据窗口点击鼠标右键：

`![bbs.pediy.com/upload/bbs/unpackfaq/9-iat7.gif](http://bbs.pediy.com/upload/bbs/unpackfaq/9-iat7.gif)`

调整显示格式后的数据窗口：

`![bbs.pediy.com/upload/bbs/unpackfaq/9-iat8.gif](http://bbs.pediy.com/upload/bbs/unpackfaq/9-iat8.gif)`

这样就直观了，`IAT` 中每组数据指向一个 `API` 函数，各 `DLL` 之间是以 `000000` 分开的。

`IAT 范围：0x4062E4～0x406524 ，大小为 0x406524-0x4062E4=0x240`

如果 `IAT` 加密了，此时 `IAT` 中的地址不是指向系统 `DLL` 中的 `API` 函数了，可能指向外壳。这就十分有必要找到外壳处理 `IAT` 的代码了，前面己说过，外壳加载时，会模拟 Windows 加载器，向 `IAT` 里填充当前操作系统 `API` 函数的实际地址。所以，在 `IAT` 里设个内存写断点，就可中断到这段代码处。

重新加载 `notepad.upx.exe`，在 `IAT` 某个地址下内存写断点，这里选择 `0x4062E4` 这个地址设内存写断点，先在数据窗口下命令：`D 4062E4`

`![bbs.pediy.com/upload/bbs/unpackfaq/9-iat9.gif](http://bbs.pediy.com/upload/bbs/unpackfaq/9-iat9.gif)`

然后选择一个地址，点击鼠标右键，下“内存写断点”。

`![bbs.pediy.com/upload/bbs/unpackfaq/9-iat10.gif](http://bbs.pediy.com/upload/bbs/unpackfaq/9-iat10.gif)`

此时只要有数据写入 `4062E4` 地址处，`OD` 就会中断，按 `F9` 运行 `OD`，会中断这里：

```
0040E96D    > /8A02           mov     al, [edx]
0040E96F    . |42            inc     edx
0040E970    . |8807          mov     [edi], al
0040E972    . |47            inc     edi
0040E973    . |49            dec     ecx
0040E974    .^\\75 F7         jnz     short 0040E96D
```

这段还不是处理 `IAT`，按 `F9` 继续执行程序，会中断这里：

```
0040E9E9    > /8A07           mov     al, [edi]
0040E9EB    . |47            inc     edi
0040E9EC    . |08C0          or      al, al
0040E9EE    .^|74 DC         je      short 0040E9CC
0040E9F0    . |89F9          mov     ecx, edi
0040E9F2    . |57            push    edi                    // 函数名字符串   
0040E9F3    . |48            dec     eax
0040E9F4    .  F2:AE         repne   scas byte ptr es:[edi]
0040E9F6    . |55            push    ebp                    // DLL 模块句柄
0040E9F7    .  FF96 A4EC0000 call    [esi+ECA4]             ;   kernel32.GetProcAddress
0040E9FD    . |09C0          or      eax, eax
0040E9FF    . |74 07        je      short 0040EA08
0040EA01    . |8903          mov     [ebx], eax             // EBX 指向 IAT,将取得的 API 地址填充进 IAT
0040EA03    . |83C3 04       add     ebx, 4                 // 指向下一个地址
0040EA06    .^\\EB E1        jmp     short 0040E9E9
0040EA08    >  FF96 A8EC0000 call    [esi+ECA8]
```

上面这段就是 `UPX` 外壳填充 `IAT` 的全过程，感兴趣的，动态跟踪一下就明白了。这里用 `GetProcAddress` 函数获得函数地址：

```
FARPROC GetProcAddress(
  HMODULE hModule,    // DLL 模块句柄
  LPCSTR lpProcName   // 函数名
);
```

## 第十课 DLL 文件脱壳

[DLL 文件脱壳](http://bbs.pediy.com/showthread.php?t=61334)

## 第十一课 优化与自校验去除

[1.浅谈程序脱壳后的优化](http://bbs.pediy.com/showthread.php?s=&threadid=28402)

[2.常见自校检分析实例](http://bbs.pediy.com/showthread.php?s=&threadid=28298)

## 第十一课 Armadillo 壳

### 辅助工具

相应工具下载页面：`http://www.pediy.com/tools/unpacker.htm`

*   `Armadillo find protected`：检测 `Armadillo` 版本号及其保护类型
*   `dilloDIE`：一款脱 `Armadillo 4.xx` 的脱壳机
*   `ArmaGUI`：`Armadillo 3.xx/4.xx` 脱壳机
*   `ArmInline`：辅助处理 `Armadillo` 壳，其中 `Code Splicing` 修复的很完美。输入表混淆修复感觉有些问题，建议手工修复。

### 手动脱壳教学

*   [试玩 armadillo3.50a 一点心得](http://www.pediy.com/bbshtml/BBS6/pediy6837.htm) （基础知识）
*   [Armadillo COPYMEMEII 之 DUMP 的一个 LOADPE 小插件](http://www.pediy.com/bbshtml/BBS6/pediy6443.htm) （利用插件 `Dump` 取 `COPYMEMEII` 保护的程序）
*   [Armadillo 3.6 主程序脱壳](http://www.pediy.com/bbshtml/BBS6/pediy6444.htm) （`Dump` 取 `COPYMEMEII` 保护的程序另一方法）
*   [Patch 修复 Armadillo 的 IAT 乱序](http://bbs.pediy.com/showthread.php?threadid=19202) （处理 `IAT` 乱序）
*   [Armadillo 中 code splicing 的几种处理方法](http://bbs.pediy.com/showthread.php?threadid=18796) （手工修复 `Code Splicing`）
*   [Armadillo 客户版 Code Splicing+Import](http://bbs.pediy.com/showthread.php?threadid=17253) （利用 `ArmInline` 工具修复 `Code Splicing`）
*   [浅谈 Armadillo V.3.75 与 V.3.78 的保护](http://www.pediy.com/bbshtml/BBS6/pediy6906.htm) （`Nanomites` 原理概念，即 `CC` 保护）
*   [Blaze Media Pro5.05 脱壳＋基本修复 CC(int3)＋破解](http://www.pediy.com/bbshtml/BBS6/pediy6499.htm) （修复 `CC`）
*   [Armadillo V4.40 主程序脱壳](http://bbs.pediy.com/showthread.php?threadid=18215) （目前处理 `CC` 很好的一个方法）

带 key 脱壳：

`http://bbs.pediy.com/showthread.php?threadid=23174`

## 第十二课 ASProtect 2.x 壳



# 下载：VolX 的 Aspr2.XX_unpacker_v1.0

[VolX 的 Aspr2.XX_unpacker_v1.0](http://bbs.pediy.com/showthread.php?s=&threadid=38143)

VolX 这个脚本其实是一个 ASProtect 脱壳机了，支持 Asprotect 1.32, 1.33, 1.35, 1.4, 2.0, 2.1, 2.11, 2.2beta, 2.2, 2.3。

## 环境准备

1. Ollydbg 1.1
2. Odbgscript 1.47 或以上的版本（OD 插件，一般集成 OD 都有）
3. Import Reconstructor

## 操作步骤

运行 `Ollydbg` 打开目标程序，单击 OD 菜单“插件/Odbgscript/运行脚本”，打开 VolX 这个脚本：`Aspr2.XX_unpacker_v1.0SC.osc`。

然后按着脚本提示操作即可。

脚本跑完，会在目标软件的目录生成一个 `de_ 文件名.exe`。OD 的记录窗口（按 `Alt` + `L`）打开显示了 `OEP`、`IAT` 等信息，如：

```
            IAT 的地址 = 00405000

            IAT 的相对地址 = 00005000

            IAT 的大小 = 000000BC

00A70042   断点位于 00A70042

            OEP 的地址 = 00401D2B

            OEP 的相对地址 = 00001D2B
```

接下来，你再用 `Import Reconstructor` 修复输入表即可，最后 `Fix Dump` 时选中 `de_ 文件名.exe`。

## 参考文章

一般情况下，用 VolX 的脚本就能脱 Asprotect 了。如果你要手工脱壳，如下文章可参考：

> 引用:
> - [nspack3.5 主程序脱壳分析(Aspr SKE 2.X)](http://bbs.pediy.com/showthread.php?s=&threadid=19313) (输入表)
> - [Asprotect SKE 2.2 的 Advanced Import protection 保护技术](http://bbs.pediy.com/showthread.php?s=&threadid=23771) (输入表)
> - [莱鸟脱 Aspr2.11 SKE+修复 stolen](http://bbs.pediy.com/showthread.php?s=&threadid=22277) (Dump 法对付 stolen OEP)
> - [ASProtect.SKE.2.11 stolen code 解密](http://bbs.pediy.com/showthread.php?s=&threadid=24437) (ASProtect 2.11 的 stolen 与 SDK 修复技术)
> - [ASProtect_SKE_2.3Beta_Build0319 stolen code 分析](http://bbs.pediy.com/showthread.php?s=&threadid=24774) (ASProtect 2.22/2.23 的 stolen 与 SDK 修复技术)
> - [ASProtect SKE 2.2 SDK 中的 API 修复](http://bbs.pediy.com/showthread.php?s=&threadid=28790)

## 附 ASProtect 脱壳机

- ASProtect 1.2 以前版本，可以用 [CASPR v1.10](http://www.pediy.com/tools/unpack/CASPR/Caspr110.zip)
- ASProtect 1.2x，可以用 [Stripper 2.07ht](http://www.pediy.com/tools/unpack/ASProtect/stripper/stripper_v207ht.rar)
- ASProtect 1.3～ASProtect 2.0，可以用 [Stripper 2.11](http://www.pediy.com/tools/unpack/ASProtect/stripper/stripper211.zip)
- ASProtect SKE 2.11 以前版本，可以用 [stripper v2.13 (last beta)](http://www.pediy.com/tools/unpack/ASProtect/stripper/stripper_v213b9.rar)
- ASProtect SKE 2.2/2.3 关注...
