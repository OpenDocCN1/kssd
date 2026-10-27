
# [原创]国庆PE总复习(1-7)合集

本来打算国庆去上海观看世博会最后精彩去的，因为有些事要做，所以就没去了，![](https://bbs.kanxue.com/view/img/face/16.gif)还是在家呆着吧，给大家重温一下PE文件的知识，国庆七天每天都会更新，我会从最基础的PE理论再结合编程给大家详细讲解，参考书籍主要两本：看雪《加密与解密》第三版，老罗《Windows下32位汇编程序设计》第二版，如果大家看了我的PE文件详解能给大家一点点小的帮助，我想国庆的努力也没白费，呵呵~~其实论坛上，这样的文章也讲了不少，这里给大家复习复习，熟能生巧嘛！！如果您对PE文件已经很熟悉了，就可以直接跳过了总复习一，后面的会更加精彩，呵呵，免得浪费你保贵的时间~~![](https://bbs.kanxue.com/view/img/face/41.gif)

PE格式是Windows下最常用的可执行文件格式，在DOS时代COM文件是最早的也是结构最简单的可执行文件，COM文件中仅包含可执行代码，没有附带任何“支持性”数据，所以，第一句执行指令必须安排在文件头部：再就是没有重定位的信息，这样代码中不能有跨段操作数据的指令，造成代码和数据，甚至包括堆栈只能限制在同一个64KB的段中，由于这个原因，DOS系统中又定义了一种可执行文件---EXE文件，EXE文件在代码的前面加了一个文件头，文件头中包括各种说明数据，如文件入口，堆栈位置，重定位表等，操作系统根据文件头的信息将代码部分装入内存，根据重定位表修正代码，最后在设置好堆栈后从文件头中指定的入口开始执行。

当Windows3.X出现的时候，可执行文件中出现了32位代码，程序运行时转到保护模式之前需要在实模式下做一些初始化，这样实模式的16位代码必须和32位代码一起放在可执行文件中，旧的DOS可执行文件格式无法满足需要，所以Windows3.X执行文件使用新的LE格式的可执行文件(Linear executable/线性可执行文件)，Window9x中的VxD程序也是使用LE格式，因为这些驱动程序中也同时包括16位和32位代码。

而在Windows 9x，Windows NT,Windows 2000下，纯32位可执行文件都使用微软设计的一种新格式——PE格式(Portable Executable File Format/可移值的执行体）。

在学习的同时建议用`Stud_PE`工具配合！！！

## PE基本概念

PE文件的框架结构如下图所示：

![](https://bbs.kanxue.com/upload/attach/201010/288797_7hfthow58p3is8z.jpg)

PE文件使用的是一个平面地址空间，所有代码和数据都被合并在一起，组成一个很大的结构。文件的内容被分割为不同的区块（Section)，区块中包含代码或数据，各个区块按页边界来对齐，区块没有大小限制，是一个连续结构，每个块都有它自己在内存中的一套属性，比如：这个块是否包含代码，是否只读或可读/写等。

认识PE文件不是作为单一内存映射文件被装入内存。Windows加载器（又称PE装载器）遍历PE文件并决定文件的哪一部分被映射，这种映射方式是将文件较高的偏移位置映射到较高的内存地址中。当磁盘文件一旦被装入内存中，磁盘上的数据结构布局和内存中的数据结构布局是一致的。这样如果在磁盘的数据结构中寻找一些内容，那么几乎都能在被装入到内存中找到相同的信息。但数据之间的相对位置可能改变，其某项的偏移地址可能区别于原始的偏移地置，不管怎样，所有表现出来的信息都允许从磁盘文件偏移到内存偏移的转换，如图所示：

![](https://bbs.kanxue.com/upload/attach/201010/288797_ccsebmgls6vndrp.jpg)

从图中可以看出映射方式为：低位到低位，高位到高位的方式

这里一定要强调一点：`RVA`（虚拟地址），`FileOffset`（文件偏移地址），`VA`（虚拟地址），`ImageBase`（基地址）,入口点（`Entry Point`）

`RVA`（`Relative Virtual Address`）的缩写，相对虚拟地址，这是一个“相对”地址，PE文件的各种数据结构中涉及地址的字段大部分都是以`RVA`表示的。

`FileOffset`（文件偏移地址）

`FileOffset`是当PE文件储存在磁盘上时，各数据的地址称做为文件偏移地址（`File Offset`）。文件偏移地址从PE文件的第一个字节开始计数，起始值为0

`VA`（虚拟地址）

`VA`程序访问存储器所使用的逻辑地址称为虚拟地址（Virtual Address），又称为内存偏移地址

`ImageBase`（基地址）

`ImageBase`文件执行时将被映射到指定内存地址中，这个初始内存地址称为基地址（`ImageBase`）。这个值是由PE文件决定的，按照默认设置。Visual C++建立的EXE文件的基地址为`00400000h`,DLL文件基地址是`10000000h`，但可以改变这个地址，有很多种方法，以后在慢慢给大家介绍，书上说的是在链接程序的`/BASE`选项中改为你的程序的入口函数名称就行了。

`Entry Point`（入口点）

`Entry Point`是指PE文件执行时的入口点（`Entry Point`）。也就是说，程序在执行时的第一行代码的地址应该就是这个值。大家用`LoadPE`工具就可以查看PE文件的区段表！

## 汇编中虚拟地址(`VRA`)与文件偏移地址(`FileOffset`)的相互转换:

+---------+---------+---------+---------+---------+---------+---------+--------------------+
| 段名称    虚拟地址   虚拟大小   物理地址   物理大小    标志
|+---------+---------+---------+---------+---------+---------+----------+--------------------+
|   Name      VOffset     VSize     ROffset     RSize       Flags
|+---------+---------+---------+---------+---------+---------+-----------+--------------------+
||   .text    00001000    00000092   00000400   00000200   60000020
||  .rdata   00002000    000000F6   00000600   00000200   40000040
||   .data    00003000    0000018E   00000800   00000200   C0000040
||   .rsrc    00004000    000003A0   00000A00   00000400   C0000040
|+---------+---------+---------+---------+---------+---------+-----------+----------------------+

将`RVA`转换为`File Offset`，给大家一个非常经典的公式：

设：`VK`为相对虚拟地址`RVA`与文件偏移地址`File Offset`的差值

```
VA=ImageBase+RVA
File Offset = RVA ---VK
File Offset = VA---ImageBase---VK
```

大家如果不想自己去算，也可以用`LoadPE`中的转换工具来计算，如图所示：

![](https://bbs.kanxue.com/upload/attach/201010/288797_wzm5fqq58wu2gir.jpg)

## DOS文件头和DOS块

PE文件中还包括一个标准的DOS可执行文件部分，如图17.1中左边的①所示，这看上去有些奇怪，但是这对于可执行文件的向下兼容性来说却是不可缺少的。

但是这种方法也存在一个问题，假如一个PE格式的可执行文件在Windows中执行，那没有任何异常，因为Windows能够识别PE文件头并正确装入，但如果将PE文件放入DOS执行，那么DOS系统肯定无法识别PE文件头，假如PE文件的头部不包括一个DOS部分的话，那么按照前面介绍的规则，PE文件头的数据会被DOS系统作为代码装入并执行，这种操作几乎可以肯定会让系统立刻挂起。

为了避免这种情况，PE文件的头部包括了一个标准的DOS MZ格式的可执行部分，这样万一在DOS下执行一个PE文件，系统可以将文件解释为DOS下的.exe可执行格式，并执行DOS部分的代码。

一般来说，DOS部分的执行代码只是简单地显示一个`"This program cannot be run in DOS mode."`就退出了，这段简单的代码是编译器自动生成的。

PE文件中的DOS部分由MZ格式的文件头和可执行代码部分组成，可执行代码被称为“DOS块”（DOS stub）。MZ格式的文件头由`IMAGE_DOS_HEADER`结构定义：

```
IMAGE_DOS_HEADER STRUCT

 e_magic WORD ? ；DOS可执行文件标记，为“MZ”

 e_cblp WORD ?

 e_cp WORD ?

 e_crlc WORD ?

 e_cparhdr WORD ?

 e_minalloc WORD ?

 e_maxalloc WORD ?

 e_ss WORD ? ；DOS代码的初始化堆栈段

 e_sp WORD ? ；DOS代码的初始化堆栈指针

 e_csum WORD ?

 e_ip WORD ? ；DOS代码的入口IP

 e_cs WORD ? ；DOS代码的入口CS

 e_lfarlc WORD ?

 e_ovno WORD ?

 e_res WORD  4 dup(?)

 e_oemid WORD ?

 e_oeminfo WORD ?

 e_res2 WORD 10 dup(?)

 e_lfanew DWORD ? ；指向PE文件头

IMAGE_DOS_HEADER ENDS
```

DOS文件头的前面部分并不陌生，第一个字段`e_magic`被定义成字符`"MZ"`（在`Windows.inc`文件中已经预定义为`IMAGE_DOS_SIGNATURE`）作为识别标志，后面的一些字段指明了入口地址、堆栈位置和重定位表位置等。

其中我们还要关心的是`e_lfanew`这个字段，`e_lfanew`字段是真正PE文件头的相对偏移（`RVA`），其指出真正PE头的文件偏移位置，它占用四个字节，位于文件开始偏移`3Ch`字节中。

分析如图所示：

![](https://bbs.kanxue.com/upload/attach/201010/288797_djq1fyh2ydljjbp.jpg)

从图中我们可以看到`e_lfanew`的值为`000000c0`，也就是说`000000c0`处是我们的PE文件头的位置。

## PE文件头

紧跟在DOS stub的是PE文件头，从DOS文件头的`e_lfanew`字段（文件偏移`003ch`)得到真正PE文件头位置后，现在来看看它的定义，PE文件头是由`IMAGE_NT_HEADERS`结构定义的：

```
IMAGE_NT_HEADERS STRUCT

Signature DWORD ? ；PE文件标识

FileHeader    IMAGE_FILE_HEADER    <>

OptionalHeader   IMAGE_OPTIONAL_HEADER32 <>

IMAGE_NT_HEADERS ENDS
```

PE文件头的第一个双字是一个标志，它被定义为`00004550h`，也就是字符`"P"`、`"E"`加上两个0，这也是“PE”这个称呼的由来，大部分的文件属性由标志后面的`IMAGE_FILE_HEADER`和`IMAGE_OPTIONAL_HEADER32`结构来定义，从名称看，似乎后面的这个PE文件表头结构是可选的（Optional），但实际上这个名称是名不符实的，因为它总是存在于每个PE文件中。

`Signature`字段：

在一个有效的PE文件里，`Signature`字段被设置为`00004550h`。ASCII码字符是`"PE00"`,

`#define IMAGE_NT_SIGNATURE`定义了这个信息。

```
#define IMAGE_NT_SIGNATURE   0x00004550
```

`"PE\0\0"`字段是PE文件头的开始，DOS头部的`e_lfanew`字段正是指向`"PE\0\0"`。

### IMAGE_FILE_HEADER结构

```
IMAGE_FILE_HEADER STRUCT

Machine WORD ? ；0004h - 运行平台

NumberOfSections WORD ? ；0006h - 文件的节数目

TimeDateStamp DWORD ? ；0008h - 文件创建日期和时间

PointerToSymbolTable DWORD ? ；000ch - 指向符号表（用于调试）

NumberOfSymbols DWORD ? ；0010h - 符号表中的符号数量（用于调试）

SizeOfOptionalHeader WORD ? ；0014h - IMAGE_OPTIONAL_HEADER32结构的长度

Characteristics WORD ? ；0016h - 文件属性

IMAGE_FILE_HEADER ENDS
```

`Machine`字段

用来指定文件的运行平台，常见的定义值见表17.1所示。Windows可以运行在Intel和SUN等几种不同的硬件平台上，不同平台指令的机器码是不同的，为不同平台编译的可执行文件显然无法通用。如果Windows检测到这个字段指定的适用平台与当前的硬件平台不兼容，它将拒绝装入这个文件。

`NumberOfSections`字段

指出文件中存在的节的数量（如图17.1中的④所示），同样，节表的数量（如图17.1中的③所示）也等于节的数量。

`TimeDateStamp`字段

编译器创建此文件的时间，它的数值是从1969年12月31日下午4：00开始到创建时间为止的总秒数。

`PointerToSymbolTable`和`NumberOfSymbols`字段

这两个字段并不重要，它们与调试用的符号表有关。

`SizeOfOptionalHeader`字段

紧接在当前结构下面的`IMAGE_OPTIONAL_HEADER32`结构的长度，这个值等于`00e0h`。

`Characteristics`字段

属性标志字段，它的不同数据位定义了不同的文件属性，具体内容如表17.2所示，这是一个很重要的字段，不同的定义将影响系统对文件的装入方式，比如，当位13为1时，表示这是一个DLL文件，那么系统将使用调用DLL入口函数的方式调用文件入口，否则的话，表示这是一个普通的可执行文件，系统直接跳到入口处执行。对于普通的可执行PE文件，这个字段的值一般是`010fh`，而对于DLL文件来说，这个字段的值一般是`210eh`。

如图所示：

![](https://bbs.kanxue.com/upload/attach/201010/288797_v7mgwylngq3289c.jpg)

### IMAGE_OPTIONAL_HEADER32结构

定义`IMAGE_OPTIONAL_HEADER32`结构的本意在于让不同的开发者能够在PE文件头中使用自定义的数据，这就是结构名称中“Optional”一词的由来，但实际上`IMAGE_FILE_HEADER`结构不足以用来定义PE文件的属性，反而在这个“可选”的部分中有着更多的定义数据，对于读者来说，可以完全不必考虑这两个结构的区别在哪里，只要把它们当成是连在一起的“PE文件头结构”就可以了。

```
IMAGE_OPTIONAL_HEADER32 STRUCT

Magic WORD ? ;0018h 107h＝ROM Image,10Bh=exe Image

MajorLinkerVersion BYTE ? ;001ah 链接器版本号

MinorLinkerVersion BYTE ? ;001bh

SizeOfCode DWORD ? ;001ch 所有含代码的节的总大小

SizeOfInitializedData DWORD? ;0020h所有含已初始化数据的节的总大小

SizeOfUninitializedData DWORD ? ;0024h 所有含未初始化数据的节的大小

AddressOfEntryPoint DWORD ? ;0028h 程序执行入口RVA

BaseOfCode DWORD ? ;002ch 代码的节的起始RVA

BaseOfData DWORD ? ;0030h 数据的节的起始RVA

ImageBase DWORD ? ;0034h 程序的建议装载地址

SectionAlignment DWORD ? ;0038h 内存中的节的对齐粒度

FileAlignment DWORD ? ;003ch 文件中的节的对齐粒度

MajorOperatingSystemVersion WORD ? ;0040h 操作系统主版本号

MinorOperatingSystemVersion WORD ? ;0042h 操作系统副版本号

MajorImageVersion WORD ? ;0044h可运行于操作系统的最小版本号

MinorImageVersion WORD ? ;0046h

MajorSubsystemVersion WORD ?;0048h 可运行于操作系统的最小子版本号

MinorSubsystemVersion WORD ? ;004ah
```
