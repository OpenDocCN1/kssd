
# [翻译]Ghidra简介

原文链接：<a href="elink@f8dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0m8^5k6h3u0Q4x3X3g2F1k6i4c8Q4x3V1j5J5x3o6p5&6i4K6u0r3x3o6y4Q4x3V1k6Y4K9r3W2V1M7X3q4Q4x3X3c8S2i4K6u0V1M7i4g2A6j5$3E0Q4x3X3c8G2N6X3g2J5N6X3W2W2N6#2)9J5c8R3`.`.">Ghidra: A quick overview for the curious</a>

<a href="elink@961K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2Y4K9r3W2V1M7X3q4Q4x3X3c8K6M7X3g2Q4x3X3g2G2M7X3N6Q4x3V1j5`.">Ghidra</a>是<a href="elink@ed5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2F1M7$3q4Q4x3X3g2Y4L8%4k6Q4x3V1k6J5k6i4y4G2N6i4u0U0k6i4y4Q4x3V1k6W2N6X3g2J5P5h3!0F1k6g2)9J5c8X3N6Z5K9h3c8J5j5g2)9J5c8R3`.`.">由美国国家安全局（NSA）研究部门开发的</a>软件逆向工程（SRE）套件，用于支持网络安全任务。它最近被公开出来，我对此感到好奇，想看看它是什么样的。

我还没有查过是否有其他人已经发过了类似的概述文章，不过，我还是决定写这篇文章，为了自己和那些不想自行运行Ghidra而只是想简单了解一下的人。

我知道将Ghidra与IDA Pro进行比较是不公平的，但我还是忍不住要比较，因为我是IDA Pro的长期用户，而且它是我在逆向工程工具方面的唯一参照物。

这篇文章很长，将包含很多截图。我刚刚开始上手Ghidra，因此，我可能是错的，或者可能会提供不准确或不完整的信息，所以请原谅我。

## 目录

- 总体概述
- 什么是Ghidra
- 文件结构概述
- 处理器模块
- Ghidra的功能
- 项目管理
- 代码浏览器
- 符号树
- 反编译器
- 代码修补和十六进制查看器
- 图表视图
- 搜索功能
- 脚本功能
- 杂项功能
- 选项
- 其他截图
- 结论

# 总体概述

## 什么是Ghidra

![](upload/attach/201903/833800_ZTPXFXNZM63Q4P4.png)

> Ghidra是一个软件逆向工程（SRE）框架，包括一套功能齐全的高端软件分析工具，使用户能够在各种平台上分析编译后的代码，包括Windows、Mac OS和Linux。功能包括反汇编，汇编，反编译，绘图和脚本，以及数百个其他功能。Ghidra支持各种处理器指令集和可执行格式，可以在用户交互模式和自动模式下运行。用户还可以使用公开的API开发自己的Ghidra插件和脚本。

## 文件结构概述

我在解压缩的Ghidra安装档案中运行了`tree`命令。这是输出：

```
├───Configurations
│ └───Public_Release
│ ├───data
│ └───lib
├───Extensions
├───Features
│ ├───Base
│ │ ├───data
│ │ │ ├───formats
│ │ │ ├───parserprofiles
│ │ │ ├───stringngrams
│ │ │ ├───symbols
│ │ │ │ ├───win32
│ │ │ └───win64
│ │ │ └───typeinfo
│ │ │ ├───generic
│ │ │ ├───mac_10.9
│ │ │ └───win32
│ │ │ └───msvcrt
│ │ ├───ghidra_scripts
│ │ └───lib
│ ├───BytePatterns
│ │ ├───data
│ │ │ └───test
│ │ ├───ghidra_scripts
│ │ └───lib
│ ├───ByteViewer
│ │ ├───data
│ │ └───lib
│ ├───DebugUtils
│ │ └───lib
│ ├───Decompiler
│ │ ├───ghidra_scripts
│ │ ├───lib
│ │ └───os
│ │ ├───linux64
│ │ ├───osx64
│ │ └───win64
│ ├───DecompilerDependent
│ │ ├───data
│ │ └───lib
│ ├───FileFormats
│ │ ├───data
│ │ │ ├───android
│ │ │ ├───crypto
│ │ │ └───languages
│ │ │ └───Dalvik
│ │ ├───ghidra_scripts
│ │ └───lib
│ ├───FunctionGraph
│ │ ├───data
│ │ └───lib
│ ├───FunctionGraphDecompilerExtension
│ │ └───lib
│ ├───FunctionID
│ │ ├───data
│ │ ├───ghidra_scripts
│ │ └───lib
│ ├───GhidraServer
│ │ ├───data
│ │ │ └───yajsw-stable-12.12
│ │ │ ├───doc
│ │ │ ├───lib
│ │ │ │ ├───core
│ │ │ │ │ ├───commons
│ │ │ │ │ ├───jna
│ │ │ │ │ ├───netty
│ │ │ │ │ └───yajsw
│ │ │ │ └───extended
│ │ │ │ ├───abeille
│ │ │ │ ├───commons
│ │ │ │ ├───cron
│ │ │ │ ├───glazedlists
│ │ │ │ ├───groovy
│ │ │ │ ├───jgoodies
│ │ │ │ ├───keystore
│ │ │ │ ├───regex
│ │ │ │ ├───velocity
│ │ │ │ ├───vfs-dbx
│ │ │ │ ├───vfs-webdav
│ │ │ │ └───yajsw
│ │ │ └───templates
│ │ ├───lib
│ │ └───os
│ │ ├───linux64
│ │ ├───win32
│ │ └───win64
│ ├───GnuDemangler
│ │ ├───ghidra_scripts
│ │ └───lib
│ ├───GraphFunctionCalls
│ │ └───lib
│ ├───MicrosoftCodeAnalyzer
│ │ └───lib
│ ├───MicrosoftDemangler
│ │ └───lib
│ ├───MicrosoftDmang
│ │ └───lib
│ ├───PDB
│ │ ├───lib
│ │ ├───os
│ │ │ └───win64
│ │ └───src
│ │ └───pdb
│ │ ├───cpp
│ │ └───headers
│ ├───ProgramDiff
│ │ └───lib
│ ├───Python
│ │ ├───data
│ │ │ └───jython-2.7.1
│ │ ├───ghidra_scripts
│ │ └───lib
│ ├───Recognizers
│ │ └───lib
│ ├───SourceCodeLookup
│ │ └───lib
│ └───VersionTracking
│ ├───data
│ ├───ghidra_scripts
│ └───lib
├───Framework
│ ├───DB
│ │ └───lib
│ ├───Demangler
│ │ └───lib
│ ├───Docking
│ │ ├───data
│ │ └───lib
│ ├───FileSystem
│ │ └───lib
│ ├───Generic
│ │ ├───data
│ │ └───lib
│ ├───Graph
│ │ └───lib
│ ├───Help
│ │ └───lib
│ ├───Project
│ │ ├───data
│ │ └───lib
│ ├───SoftwareModeling
│ │ ├───data
│ │ │ └───languages
│ │ └───lib
│ └───Utility
│ └───lib
├───Processors
│ ├───6502
│ │ └───data
│ │ └───languages
│ ├───68000
│ │ ├───data
│ │ │ ├───languages
│ │ │ └───manuals
│ │ └───lib
│ ├───6805
│ │ └───data
│ │ └───languages
│ ├───8051
│ │ ├───data
│ │ │ ├───languages
│ │ │ │ └───old
│ │ │ └───manuals
│ │ └───ghidra_scripts
│ ├───8085
│ │ └───data
│ │ └───languages
│ ├───AARCH64
│ │ ├───data
│ │ │ ├───languages
│ │ │ └───patterns
│ │ └───lib
│ ├───ARM
│ │ ├───data
│ │ │ ├───languages
│ │ │ │ └───old
│ │ │ ├───manuals
│ │ │ └───patterns
│ │ └───lib
│ ├───Atmel
│ │ ├───data
│ │ │ ├───languages
│ │ │ └───manuals
│ │ └───lib
│ ├───CR16
│ │ └───data
│ │ ├───languages
│ │ └───manuals
│ ├───DATA
│ │ ├───data
│ │ │ └───languages
│ │ ├───ghidra_scripts
│ │ └───lib
│ ├───JVM
│ │ ├───data
│ │ │ ├───languages
│ │ │ └───manuals
│ │ └───lib
│ ├───MIPS
│ │ ├───data
│ │ │ ├───languages
│ │ │ ├───manuals
│ │ │ └───patterns
│ │ └───lib
│ ├───PA-RISC
│ │ └───data
│ │ ├───languages
│ │ ├───manuals
│ │ └───patterns
│ ├───PIC
│ │ ├───data
│ │ │ ├───languages
│ │ │ └───manuals
│ │ ├───ghidra_scripts
│ │ └───lib
│ ├───PowerPC
│ │ ├───data
│ │ │ ├───languages
│ │ │ │ └───old
│ │ │ ├───manuals
│ │ │ └───patterns
│ │ └───lib
│ ├───Sparc
│ │ ├───data
│ │ │ ├───languages
│ │ │ ├───manuals
│ │ │ └───patterns
│ │ └───lib
│ ├───TI_MSP430
│ │ └───data
│ │ ├───languages
│ │ └───manuals
│ ├───Toy
│ │ ├───data
│ │ │ └───languages
│ │ │ └───old
│ │ │ └───v01stuff
│ │ └───lib
│ ├───x86
│ │ ├───data
│ │ │ ├───languages
│ │ │ │ └───old
│ │ │ ├───manuals
│ │ │ └───patterns
│ │ └───lib
│ └───Z80
│ └───data
│ ├───languages
│ └───manuals
└───Test
└───IntegrationTest
└───lib
```

可以看出这个项目非常有条理。深入挖掘，我注意到Ghidra已经包含了许多组件的源代码：

- 如果你搜索`\* -src.zip`，会出现很多源代码文件。
- PDB插件源代码
- 源代码形式的200多个Java脚本
- 等等

我提到了源代码，因为在撰写本文时，Ghidra的GitHub仓库仍然不包含源代码，它的内容如下：

> 此仓库是完整开源版本的预备发布区。请放心，我们正在努力使这里的软件可用。在此期间，您可以在您的软件逆向工程（SRE）工作中使用Ghidra，开发自己的脚本和插件，并仔细阅读首次公开发行版本中发布的超过一百万行的Java和Sleigh代码。该版本可以从我们的项目主页下载。请考虑查看我们的贡献者指南，了解如何在项目可用时参与此开源项目。

## 处理器模块

在撰写本文时，Ghidra支持以下处理器模块：

- 6502
- 68000
- 6805
- 8051
- 8085
- AARCH64
- ARM
- Atmel
- CR16
- DATA
- JVM
- MIPS
- PA-RISC
- PIC
- PowerPC
- Sparc
- TI_MSP430
- Toy
- x86
- Z80

它们位于`C:\ghidra_9.0\Ghidra\Processors`。

处理器模块似乎是数据驱动的。它们的一些插件/扩展方面是用Java编写和实现的。例如，您可以在此处找到x86模块的一些源代码组件：`C:\ghidra_9.0\Ghidra\Processors\x86\lib\x86-src.zip`。

处理器模块的可编程部分包含“重定位解码器”、“文件格式解码器”、“分析插件”等。

```
├───app
│ ├───plugin
│ │ └───core
│ │ └───analysis
│ └───util
│ └───bin
│ └───format
│ ├───coff
│ │ └───relocation
│ └───elf
│ ├───extend
│ └───relocation
└───feature
└───fid
└───hash
```

有趣的是，处理器模块参考了外部工具（即IDA Pro）中相应的处理器模块：

```
<language_definitions>
<language processor="6502"
endian="little"
size="16"
variant="default"
version="1.0"
slafile="6502.sla"
processorspec="6502.pspec"
id="6502:LE:16:default">
<description>6502 Microcontroller Family</description>
<compiler name="default" spec="6502.cspec" id="default"/>
<external_name tool="IDA-PRO" name="m6502"/>
<external_name tool="IDA-PRO" name="m65c02"/>
</language>
```

# Ghidra的功能

Ghidra功能齐全。它包括强大的代码浏览器，图形查看器，反编译器，数百个脚本，各种搜索工具，撤消/重做支持，协同工作服务器，程序差异比较工具等。由于Ghidra是巨大的，我无法涵盖每一个功能。那么，我将关注那些经验丰富的逆向工程师所认为的基础功能，也就是最重要和最有用的功能。

## 项目管理

在Ghidra中，任何东西都是一个项目。与IDA不同，您不会使用输入文件开始逆向工程会话，而是从创建项目开始。在第一次运行时没有项目，您将看到此对话框：

![](upload/attach/201903/833800_QZ9SVQHNXDYUVRV.png)

在本文中，我将对我的开源的Wizmo工具（可在<a href="elink@167K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6D9j5h3I4D9L8%4g2K6P5o6R3$3i4K6u0r3g2$3W2F1g2r3!0G2L8s2y4Q4x3V1k6@1M7X3g2W2i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3g2$3W2*7L8h3)9`.">此处</a>找到）进行逆向工程。如果你想跟着我一起使用Ghidra，请下载<a href="elink@e28K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6D9j5h3I4D9L8%4g2K6P5o6R3$3i4K6u0r3g2$3W2F1g2r3!0G2L8s2y4Q4x3V1k6J5j5i4N6Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6i4K9i4A6E0L8#2)9J5c8V1u0u0e0V1q4d9d9f1g2e0i4K6u0r3g2$3W2*7L8h3)9H3x3K6l9$3x3U0l9I4z5g2)9J5k6i4A6A6M7l9`.`.">二进制文件</a>。

首先创建一个名为“Wizmo”的项目，然后导入“WizmoConsole.exe”程序：

![](upload/attach/201903/833800_SRQRTQ237DKD67S.png)

导入文件后，将显示导入结果摘要对话框：

![](upload/attach/201903/833800_YPGZAN4ST8TVPZM.png)

按“确定”后，您将看到代码浏览器窗口，并将被询问是否要开始分析该文件：

![](upload/attach/201903/833800_YKEUD5QDB5BGKZZ.png)

您可以随后从“分析”菜单中分析或重新分析该文件：
