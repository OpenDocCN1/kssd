
# For 菜鸟文章：PE 文件格式, qduwg 翻译

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-748.htm`](https://www.pediy.com/kssd/pediy07/pediy7-748.htm)

PE 文件格式

（翻译：QduWg，原作 LUEVELSMEYER）

说明：希望本文能够对初级入门 CRACKER 有一定帮助，翻译存在疏漏或者不准确，希望来信指出。感谢您的指导！感谢看雪为我们提供这个交流平台，让我们技术与时俱进！！

## 前言

`PE` ("portable executable") 文件格式是针对 MS `windows NT`, `windows 95` and `win32s` 的可执行二进制代码 (`DLLs` and `programs`)。在 `windows NT` 内, 驱动程序也是这个格式。也可以用于对象文件和库。

这个格式是 Microsoft 设计的，并在 1993 经过 `TIS` (tool interface standard) 委员会 (Microsoft, Intel, Borland, Watcom, IBM 等) 标准化了的。它基于在 UNIX 和 VMS 上运行的对象文件和可执行文件的 `COFF` "common object file format" 格式。

`win32 SDK` 包括一个头文件 `<winnt.h>` 包括对 PE 格式的定义。我将提及成员名和定义。你也可能发现 `DLL` 文件 `"imagehelp.dll"` 非常有用。它是 NT 的一部分，但文档很少。它的一些函数在 `"Developer Network"` 被描述。

## 总览

在 PE 文件的开始，我们可以发现 MSDOS 执行部分 (`"stub"`); 这使得任何一个 PE 文件是有效的 DOS 执行文件。在 `DOS-stub` 之后是 32 位的魔数 `0x00004550` (`IMAGE_NT_SIGNATURE`). 然后是一个 `COFF` 格式的文件头，指明在何种机器上运行，多少个节在里面，连接的时间，是否是可执行文件或者 DLL 等。DLL 和可执行文件的区别：DLL 不能够启动，只可以被其他可执行文件使用，一个可执行文件不能够连接到另一个可执行文件。

接着，我们看到一个可选文件头 `optional header` (虽然叫“可选”，它实际上一直存在）。

`COFF` 把可选文件头用于库，不用于目标文件。这里告诉我们文件如何被调入：起始地址，预留堆栈数，数据段尺寸。

一个有趣的部分是尾巴上的数组数据目录 `data directories`，这些目录包含指向节内数据的指针。例如，如果文件有输出目录，可以在数组成员 `IMAGE_DIRECTORY_ENTRY_EXPORT` 内发现一个指针指向那个目录（目录描述结构－> `THUNKDATA` 结构－> `BYNAME` 结构）。他将指向一个节。

在头的后面是节头，实际上，节的内容就是真正需要运行一个程序所需要的东西，所有的头和目录成员就是帮你找到它。每个节有几个标志：对齐，包含的数据类型（初始化数据等），是否可以共享等，及数据自身。多数节含有一个或多个通过“可选头”内的数据目录项引用的目录。没有目录类型的内容是初始化数据或者可执行代码。（节是物理意义上的内容组织，目录是逻辑意义上的内容组织，两者互相配合才能找到需要的东西。节是存储内容的地方，区域的安排，目录是如何对里面东西进行查找，目的是寻找里面的内容）

```
+-------------------+
| DOS-stub          |
+-------------------+
| file-header       |
+- - - - - - - - - -+
| optional header   |
|- - - - - - - - - -|
|                   |
| data directories  |
|                   |
+-------------------+
|                   |
| section headers   |
|                   |
+-------------------+
|                   |
| section 1         |
|                   |
+-------------------+
|                   |
| section 2         |
|                   |
+-------------------+
|                   |
| ...               |
|                   |
+-------------------+
|                   |
| section n         |
|                   |
+-------------------+
```

## DOS-stub and Signature

\----------------------

DOS STUB 的概念在 16 位 WINDOWS 可执行文件内就已经被熟知了，STUB 是用于 OS/2 可执行文件，自解压文档和其他程序。对于 PE 文件，它是 DOS2 兼容可执行文件，总是包含 100 字节内容，输出一个错误信息：比如 `"this program needs windows NT"`.

你认识到一个 DOSSTUB 通过验证 DOS-header，就是一个 `IMAGE_DOS_HEADER` 结构，前两个字节必须使 `"MZ"`（有一个定义针对这个 WORD，`IMAGE_DOS_SIGNATURE` ）。你通过尾部的 `'e_lfanew'` 给出的偏移量所确定的签名区别一个 PE 文件。对于 PE 文件，它是一个 32 位，按照 8 字节对齐边界。其值 `0x00004550` 由 `IMAGE_NT_SIGNATURE` 定义.

`IMAGE_NT_HEADERS STRUCT`

  `Signature`             `DWORD`                        ?

  `FileHeader`            `IMAGE_FILE_HEADER`            <>

  `OptionalHeader`        `IMAGE_OPTIONAL_HEADER32`      <>

`IMAGE_NT_HEADERS ENDS`

## 文件头 File Header

\-----------

`IMAGE_FILE_HEADER STRUCT`

  `Machine`               `WORD`    ?

  `NumberOfSections`      `WORD`    ?

  `TimeDateStamp`         `DWORD`   ?

  `PointerToSymbolTable`  `DWORD`   ?

  `NumberOfSymbols`       `DWORD`   ?

  `SizeOfOptionalHeader`  `WORD`    ?

  `Characteristics`       `WORD`    ?

`IMAGE_FILE_HEADER ENDS`

要得到 `IMAGE_FILE_HEADER`, 确认 DOS 头的前 2 个字节 `"MZ"`，然后找到 `'e_lfanew'` 成员，然后从文件开始跳过许多字节，验证你找到的签名。文件头作为一个 `IMAGE_FILE_HEADER` 结构, 就从它后面开始。从上到下描述其成员。

第 1：`Machine`, 16 位值，指明可执行文件所需要的系统。已知合法值如下：

    `IMAGE_FILE_MACHINE_I386`

    `0x014c`  Intel 80386 处理器。

    `0x014d`  Intel 80486 处理器

    `0x014e`  Pentium 处理器

    `0x0160`  R3000 (MIPS) 处理器

    `IMAGE_FILE_MACHINE_R3000` (`0x162`) R3000 (MIPS) 处理器

    `IMAGE_FILE_MACHINE_R4000` (`0x166`) R4000 (MIPS) 处理器

    `IMAGE_FILE_MACHINE_R10000` (`0x168`) R10000 (MIPS) 处理器

    `IMAGE_FILE_MACHINE_ALPHA` (`0x184`) DEC Alpha AXP 处理器

    `IMAGE_FILE_MACHINE_POWERPC` (`0x1F0`) IBM Power PC 处理器

第 2：`NumberOfSections`, 16 位值，它是跟随于头后面的节数。我们在后面讨论。

第 3：`TimeDateStamp`，32 位值, 文件创建的时间。可以通过该值区分不同的文件版本。时间戳用于绑定输入目录，后面讲到。有些连接器设置该值为荒唐的值。

第 3：`PointerToSymbolTable` 和 `NumberOfSymbols`，都是 32 位的。用于调试信息。一般都是 0。

第 4：`SizeOfOptionalHeader`，16 位，是 `IMAGE_OPTIONAL_HEADER` 的尺寸. 可以用它确认 PE 文件结构的正确性。

第 5：`Characteristics`，16 位值，包括一个标志集合，多数只对目标文件和库有效。

    `Bit 0` (`IMAGE_FILE_RELOCS_STRIPPED`) 如果文件内没有重定位信息该位置 1。这里指的是每个节内的重定位信息。不用于可执行文件，可执行文件的重定位信息在后面提到的 `base relocation` 目录。

    `Bit 1` (`IMAGE_FILE_EXECUTABLE_IMAGE`) 如果文件是可执行的则置 1，例如不是一个目标文件或者库文件。如果连接器试图创建可执行文件，但由于某种原因失败了，也置 1。

    `Bit 2` (`IMAGE_FILE_LINE_NUMS_STRIPPED`) 如果行数信息剥离，置 1，对可执行文件无效。

    `Bit 3` (`IMAGE_FILE_LOCAL_SYMS_STRIPPED`) 如果没有本地符号信息该位置 1。对可执行文件无效。

    `Bit 4` (`IMAGE_FILE_AGGRESIVE_WS_TRIM`) 如果操作系统被假定通过页换出抢占式修剪进程的工作集（进程使用的内存数），该位置 1。

    `Bits 7` (`IMAGE_FILE_BYTES_REVERSED_LO`) 和 `15` (`IMAGE_FILE_BYTES_REVERSED_HI`) 如果文件的的 `endianess` 不是机器期望的，则置 1，于是读之前必须交换字节。对可执行文件不可靠。

    `Bit 8` (`IMAGE_FILE_32BIT_MACHINE`) 如果机器被期望是 32 位机器，置 1。

    `Bit 9` (`IMAGE_FILE_DEBUG_STRIPPED`) 如果没有调试信息在文件内，置 1。对可执行文件无效。

    `Bit 10` (`IMAGE_FILE_REMOVABLE_RUN_FROM_SWAP`) 如果程序不能够从可移动媒体比如软盘或光驱，置 1。操作系统建议拷贝文件到交换文件然后执行。

    `Bit 11` (`IMAGE_FILE_NET_RUN_FROM_SWAP`) 如果不能够才网络运行，置 1。操作系统建议拷贝文件到交换文件然后执行。

    `Bit 12` (`IMAGE_FILE_SYSTEM`) 如果文件是类似驱动程序的系统文件，置 1。对执行文件无效。

    `Bit 13` (`IMAGE_FILE_DLL`) 如果文件是 DLL，置 1.

    `Bit 14` (`IMAGE_FILE_UP_SYSTEM_ONLY`) 如果文件不是为多处理器设计的，置 1  。

## 相对虚拟地址 Relative Virtual Addresses

\--------------------------

PE 格式使用所谓的 `RVA`。用于描述内存地址，如果你不知道基地址的话。需要你加上基地址得到线性地址。基地址是 PE 映像加载的地址。例如：假如可执行文件加载到 `0x400000`，可执行文件的 `RVA` 是 `0x1560`. 有效执行起始地址为 `0x401560`. 如果被加载到 `0x100000`, 则执行起始位置在 `0x101560`.

情况变的复杂起来，由于节不必按照加载的映像那样对齐。例如，节一般按照 512 字节对齐。加载的映像可能按照 4096 字节对齐。参看 `'SectionAlignment'` and `'FileAlignment'`  。对齐的意思就是地址值＝对齐长度的倍数。

于是为了找到一个特殊的 `RVA` 指向的信息，你必须计算偏移量好象文件被加载一样。假如知道执行起点在 `RVA 0x1560`, 想从这反汇编代码。要找到文件内的地址，你必须找出在 RAM 内的按照 4096 对齐的节，`".code"` 节自内存 `RVA 0x1000` 开始，16384 字节长，你知道 `RVA 0x1560` 的偏移量在那个节内是 `0x560`. 找出节在文件内按照 512 字节对齐，且 `".code"` 从 `0x800` 开始，那么在文件内的代码执行起点是 `0x800+0x560=0xd60`。

然后反汇编，并发现一个存取地址 `0x1051d0` 处的变量. 线性地址在加载执行文件时重新分配，并给出优先加载地址。你发现优先加载地址是 `0x100000`, 于是我们处理 `RVA 0x51d0`. 这是一个开始于 `RVA 0x5000` 的数据区，2048 字节长。它开始于文件偏移量 `0x4800`. 变量可以在文件偏移量 `0x4800+0x51d0-0x5000=0x49d0` 处发现。

## 可选头 Optional Header

\---------------

紧跟在文件头的后面是 `IMAGE_OPTIONAL_HEADER`，尽管名字是可选，实际一直存在。包含关于如何精确处理 PE 文件的信息。从上到下介绍成员。

`IMAGE_OPTIONAL_HEADER32 STRUCT`

  `Magic`                                 `WORD`       ?

  `MajorLinkerVersion`                    `BYTE`       ?

  `MinorLinkerVersion`                    `BYTE`       ?

  `SizeOfCode`                            `DWORD`      ?

  `SizeOfInitializedData`                 `DWORD`      ?

  `SizeOfUninitializedData`               `DWORD`      ?

  `AddressOfEntryPoint`                   `DWORD`      ?

  `BaseOfCode`                            `DWORD`      ?

  `BaseOfData`                            `DWORD`      ?

  `ImageBase`                             `DWORD`      ?

  `SectionAlignment`                      `DWORD`      ?

  `FileAlignment`                         `DWORD`      ?

  `MajorOperatingSystemVersion`           `WORD`       ?

  `MinorOperatingSystemVersion`           `WORD`       ?

  `MajorImageVersion`                     `WORD`       ?

  `MinorImageVersion`                     `WORD`       ?

  `MajorSubsystemVersion`                 `WORD`       ?

  `MinorSubsystemVersion`                 `WORD`       ?

  `Win32VersionValue`                     `DWORD`      ?

  `SizeOfImage`                           `DWORD`      ?

  `SizeOfHeaders`                         `DWORD`      ?

  `CheckSum`                              `DWORD`      ?

  `Subsystem`                             `WORD`       ?

  `DllCharacteristics`                    `WORD`       ?

  `SizeOfStackReserve`                    `DWORD`      ?

  `SizeOfStackCommit`                     `DWORD`      ?

  `SizeOfHeapReserve`                     `DWORD`      ?

  `SizeOfHeapCommit`                      `DWORD`      ?

  `LoaderFlags`                           `DWORD`      ?

  `NumberOfRvaAndSizes`                   `DWORD`      ?

  `DataDirectory`                         `IMAGE_DATA_DIRECTORY` `IMAGE_NUMBEROF_DIRECTORY_ENTRIES` `dup(<> )`
