

# For 菜鸟文章：PE 文件格式,qduwg 翻译

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-748.htm`](https://www.pediy.com/kssd/pediy07/pediy7-748.htm)

PE 文件格式  
（翻译：QduWg，原作 LUEVELSMEYER）  
说明：希望本文能够对初级入门 CRACKER 有一定帮助，翻译存在疏漏或者不准确，希望来信指出。感谢您的指导！感谢看雪为我们提供这个交流平台，让我们技术与时俱进！！  
前言：  
PE("portable executable")文件格式是针对 MS windows NT, windows 95 and  
win32s 的可执行二进制代码(DLLs and programs) 。在 windows NT 内, 驱动程序也是这个格式。也可以用于对象文件和库。  
  
这个格式是 Microsoft 设计的，并在 1993 经过 TIS (tool interface standard)委员会(Microsoft, Intel, Borland, Watcom, IBM 等)标准化了的。它基于在 UNIX 和 VMS 上运行的对象文件和可执行文件的 COFF"common object file format"格式。  
  
win32 SDK 包括一个头文件<winnt.h>包括对 PE 格式的定义。我将提及成员名和定义。你也可能发现 DLL 文件"imagehelp.dll" 非常有用。它是 NT 的一部分，但文档很少。它的一些函数在"Developer Network"被描述。  
  
总览：  
在 PE 文件的开始，我们可以发现 MSDOS 执行部分("stub"); 这使得任何一个 PE 文件是有效的 DOS 执行文件。在 DOS-stub 之后是 32 位的魔数 0x00004550(IMAGE\_NT\_SIGNATURE).然后是一个 COFF 格式的文件头，指明在何种机器上运行，多少个节在里面，连接的时间，是否是可执行文件或者 DLL 等。DLL 和可执行文件的区别：DLL 不能够启动，只可以被其他可执行文件使用，一个可执行文件不能够连接到另一个可执行文件。  
  
接着，我们看到一个可选文件头 optional header(虽然叫“可选”，它实际上一直存在）。  
COFF 把可选文件头用于库，不用于目标文件。这里告诉我们文件如何被调入：起始地址，预留堆栈数，数据段尺寸。  
  
一个有趣的部分是尾巴上的数组数据目录 data directories，这些目录包含指向节内数据的指针。例如，如果文件有输出目录，可以在数组成员 IMAGE\_DIRECTORY\_ENTRY\_EXPORT 内发现一个指针指向那个目录（目录描述结构－>THUNKDATA 结构－>BYNAME 结构）。他将指向一个节。  
  
在头的后面是节头，实际上，节的内容就是真正需要运行一个程序所需要的东西，所有的头和目录成员就是帮你找到它。每个节有几个标志：对齐，包含的数据类型（初始化数据等），是否可以共享等，及数据自身。多数节含有一个或多个通过“可选头”内的数据目录项引用的目录。没有目录类型的内容是初始化数据或者可执行代码。（节是物理意义上的内容组织，目录是逻辑意义上的内容组织，两者互相配合才能找到需要的东西。节是存储内容的地方，区域的安排，目录是如何对里面东西进行查找，目的是寻找里面的内容）  
    +-------------------+  
    | DOS-stub          |  
    +-------------------+  
    | file-header       |  
    +- - - - - - - - - -+  
    | optional header   |  
    |- - - - - - - - - -|  
    |                   |  
    | data directories  |  
    |                   |  
    +-------------------+  
    |                   |  
    | section headers   |  
    |                   |  
    +-------------------+  
    |                   |  
    | section 1         |  
    |                   |  
    +-------------------+  
    |                   |  
    | section 2         |  
    |                   |  
    +-------------------+  
    |                   |  
    | ...               |  
    |                   |  
    +-------------------+  
    |                   |  
    | section n         |  
    |                   |  
    +-------------------+  
DOS-stub and Signature  
\----------------------  
DOS STUB 的概念在 16 位 WINDOWS 可执行文件内就已经被熟知了，STUB 是用于 OS/2 可执行文件，自解压文档和其他程序。对于 PE 文件，它是 DOS2 兼容可执行文件，总是包含 100 字节内容，输出一个错误信息：比如"this program needs windows NT".  
你认识到一个 DOSSTUB 通过验证 DOS-header，就是一个 IMAGE\_DOS\_HEADER 结构，前两个字节必须使"MZ"（有一个定义针对这个 WORD，IMAGE\_DOS\_SIGNATURE ）。你通过尾部的'e\_lfanew' 给出的偏移量所确定的签名区别一个 PE 文件。对于 PE 文件，它是一个 32 位，按照 8 字节对齐边界。其值 0x00004550 由 IMAGE\_NT\_SIGNATURE 定义.  
  
IMAGE\_NT\_HEADERS STRUCT  
  Signature         DWORD                   ?  
  FileHeader        IMAGE\_FILE\_HEADER       <>  
  OptionalHeader    IMAGE\_OPTIONAL\_HEADER32 <>  
IMAGE\_NT\_HEADERS ENDS  
  
文件头 File Header  
\-----------  
IMAGE\_FILE\_HEADER STRUCT  
  Machine               WORD    ?  
  NumberOfSections      WORD    ?  
  TimeDateStamp         DWORD   ?  
  PointerToSymbolTable  DWORD   ?  
  NumberOfSymbols       DWORD   ?  
  SizeOfOptionalHeader  WORD    ?  
  Characteristics       WORD    ?  
IMAGE\_FILE\_HEADER ENDS  
  
要得到 IMAGE\_FILE\_HEADER,确认 DOS 头的前 2 个字节"MZ"，然后找到'e\_lfanew'成员，然后从文件开始跳过许多字节，验证你找到的签名。文件头作为一个 IMAGE\_FILE\_HEADER 结构,就从它后面开始。从上到下描述其成员。  
  
第 1：Machine, 16 位值，指明可执行文件所需要的系统。已知合法值如下：  
    IMAGE\_FILE\_MACHINE\_I386   
    0x014c  Intel 80386 处理器。  
    0x014d  Intel 80486 处理器  
    0x014e  Pentium 处理器  
    0x0160  R3000 (MIPS)处理器  
    IMAGE\_FILE\_MACHINE\_R3000 (0x162) R3000 (MIPS)处理器  
    IMAGE\_FILE\_MACHINE\_R4000 (0x166) R4000 (MIPS)处理器  
    IMAGE\_FILE\_MACHINE\_R10000 (0x168)R10000 (MIPS)处理器  
    IMAGE\_FILE\_MACHINE\_ALPHA (0x184) DEC Alpha AXP 处理器  
    IMAGE\_FILE\_MACHINE\_POWERPC (0x1F0)IBM Power PC 处理器  
第 2：NumberOfSections,16 位值，它是跟随于头后面的节数。我们在后面讨论。  
第 3：TimeDateStamp，32 位值,文件创建的时间。可以通过该值区分不同的文件版本。时间戳用于绑定输入目录，后面讲到。有些连接器设置该值为荒唐的值。  
第 3：PointerToSymbolTable 和 NumberOfSymbols，都是 32 位的。用于调试信息。一般都是 0。  
第 4：SizeOfOptionalHeader，16 位，是 IMAGE\_OPTIONAL\_HEADER 的尺寸.可以用它确认 PE 文件结构的正确性。  
第 5：Characteristics，16 位值，包括一个标志集合，多数只对目标文件和库有效。  
    Bit 0 (IMAGE\_FILE\_RELOCS\_STRIPPED) 如果文件内没有重定位信息该位置 1。这里指的是每个节内的重定位信息。不用于可执行文件，可执行文件的重定位信息在后面提到的 base relocation 目录。  
    Bit 1 (IMAGE\_FILE\_EXECUTABLE\_IMAGE) 如果文件是可执行的则置 1，例如不是一个目标文件或者库文件。如果连接器试图创建可执行文件，但由于某种原因失败了，也置 1。  
    Bit 2 (IMAGE\_FILE\_LINE\_NUMS\_STRIPPED)如果行数信息剥离，置 1，对可执行文件无效。  
    Bit 3 (IMAGE\_FILE\_LOCAL\_SYMS\_STRIPPED) 如果没有本地符号信息该位置 1。对可执行文件无效。  
    Bit 4 (IMAGE\_FILE\_AGGRESIVE\_WS\_TRIM) 如果操作系统被假定通过页换出抢占式修剪进程的工作集（进程使用的内存数），该位置 1。  
    Bits 7 (IMAGE\_FILE\_BYTES\_REVERSED\_LO)和 15(IMAGE\_FILE\_BYTES\_REVERSED\_HI) 如果文件的的 endianess 不是机器期望的，则置 1，于是读之前必须交换字节。对可执行文件不可靠。  
    Bit 8 (IMAGE\_FILE\_32BIT\_MACHINE) 如果机器被期望是 32 位机器，置 1。  
    Bit 9 (IMAGE\_FILE\_DEBUG\_STRIPPED)如果没有调试信息在文件内，置 1。对可执行文件无效。  
    Bit 10 (IMAGE\_FILE\_REMOVABLE\_RUN\_FROM\_SWAP) 如果程序不能够从可移动媒体比如软盘或光驱，置 1。操作系统建议拷贝文件到交换文件然后执行。  
    Bit 11 (IMAGE\_FILE\_NET\_RUN\_FROM\_SWAP) 如果不能够才网络运行，置 1。操作系统建议拷贝文件到交换文件然后执行。  
    Bit 12 (IMAGE\_FILE\_SYSTEM) 如果文件是类似驱动程序的系统文件，置 1。对执行文件无效。  
    Bit 13 (IMAGE\_FILE\_DLL) 如果文件是 DLL，置 1.  
    Bit 14 (IMAGE\_FILE\_UP\_SYSTEM\_ONLY) 如果文件不是为多处理器设计的，置 1 。  
  
相对虚拟地址 Relative Virtual Addresses  
\--------------------------  
PE 格式使用所谓的 RVA。用于描述内存地址，如果你不知道基地址的话。需要你加上基地址得到线性地址。基地址是 PE 映像加载的地址。例如：假如可执行文件加载到 0x400000，可执行文件的 RVA 是 0x1560. 有效执行起始地址为 0x401560.如果被加载到 0x100000,则执行起始位置在 0x101560.  
  
情况变的复杂起来，由于节不必按照加载的映像那样对齐。例如，节一般按照 512 字节对齐。加载的映像可能按照 4096 字节对齐。参看'SectionAlignment' and 'FileAlignment' 。对齐的意思就是地址值＝对齐长度的倍数。  
  
于是为了找到一个特殊的 RVA 指向的信息，你必须计算偏移量好象文件被加载一样。假如知道执行起点在 RVA 0x1560, 想从这反汇编代码。要找到文件内的地址，你必须找出在 RAM 内的按照 4096 对齐的节，".code"节自内存 RVA 0x1000 开始，16384 字节长，你知道 RVA 0x1560 的偏移量在那个节内是 0x560.找出节在文件内按照 512 字节对齐，且".code"从 0x800 开始，那么在文件内的代码执行起点是 0x800+0x560=0xd60。  
然后反汇编，并发现一个存取地址 0x1051d0 处的变量.线性地址在加载执行文件时重新分配，并给出优先加载地址。你发现优先加载地址是 0x100000,于是我们处理 RVA 0x51d0. 这是一个开始于 RVA 0x5000 的数据区，2048 字节长。它开始于文件偏移量 0x4800.变量可以在文件偏移量 0x4800+0x51d0-0x5000=0x49d0 处发现。  
  
可选头 Optional Header  
\---------------  
紧跟在文件头的后面是 IMAGE\_OPTIONAL\_HEADER，尽管名字是可选，实际一直存在。包含关于如何精确处理 PE 文件的信息。从上到下介绍成员。  
IMAGE\_OPTIONAL\_HEADER32 STRUCT  
  Magic                         WORD       ?  
  MajorLinkerVersion            BYTE       ?  
  MinorLinkerVersion            BYTE       ?  
  SizeOfCode                    DWORD      ?  
  SizeOfInitializedData         DWORD      ?  
  SizeOfUninitializedData       DWORD      ?  
  AddressOfEntryPoint           DWORD      ?  
  BaseOfCode                    DWORD      ?  
  BaseOfData                    DWORD      ?  
  ImageBase                     DWORD      ?  
  SectionAlignment              DWORD      ?  
  FileAlignment                 DWORD      ?  
  MajorOperatingSystemVersion   WORD       ?  
  MinorOperatingSystemVersion   WORD       ?  
  MajorImageVersion             WORD       ?  
  MinorImageVersion             WORD       ?  
  MajorSubsystemVersion         WORD       ?  
  MinorSubsystemVersion         WORD       ?  
  Win32VersionValue             DWORD      ?  
  SizeOfImage                   DWORD      ?  
  SizeOfHeaders                 DWORD      ?  
  CheckSum                      DWORD      ?  
  Subsystem                     WORD       ?  
  DllCharacteristics            WORD       ?  
  SizeOfStackReserve            DWORD      ?  
  SizeOfStackCommit             DWORD      ?  
  SizeOfHeapReserve             DWORD      ?  
  SizeOfHeapCommit              DWORD      ?  
  LoaderFlags                   DWORD      ?  
  NumberOfRvaAndSizes           DWORD      ?  
  DataDirectory                 IMAGE\_DATA\_DIRECTORY IMAGE\_NUMBEROF\_DIRECTORY\_ENTRIES dup(<> )  
IMAGE\_OPTIONAL\_HEADER32 ENDS  
IMAGE\_OPTIONAL\_HEADER  equ  <IMAGE\_OPTIONAL\_HEADER32>  
  
第 1 个 16 位字是'Magic'，总是 0x010b.  
  
下面 2 个字节是连接器的版本号'MajorLinkerVersion'和'MinorLinkerVersion'，这些值都不可靠，不能总是妥当的反映连接器版本。有些连接器不设置该域。  
  
下面 3 个 longwords(32 位)指定执行代码尺寸('SizeOfCode'),初始化数据尺寸  
'SizeOfInitializedData', 所谓的数据段"data segment", 未初始化数据尺寸  
'SizeOfUninitializedData',所谓的"bss segment".这些数值也不可靠。  
  
往下一个 32 位的 RVA.是入口点的偏移量。('AddressOfEntryPoint').执行从此开始。  
  
下面 2 个 32 位是可执行代码('BaseOfCode')和初始化数据('BaseOfData')的 RVAs 我们对它没有兴趣，因为可以通过节来查看更可靠的信息。非初始化数据没有 RVA。  
  
下面是一个 32 位值，ImageBase'作为整个文件的优先加载地址，包括所有头在内。该值总是  
64KB 的倍数，文件已经被连接器重定位，如果文件能够真正加载到这个地址，加载器不必重定位文件。如果另一个映像已经被加载到那个地址，则优先地址不可使用。这种情况下，映像被加载到其他地址，需要重定位。如果映像是 DLL，还有更多结果，因为"bound imports"不再有效，需要对使用 DLL 的执行文件进行修正。参见'import directory' 。  
  
下面 2 个 32 位是当映像文件加载后，PE 文件的节在内存内的对齐，'SectionAlignment', 以及在文件内的对齐'FileAlignment'. 一般文件对齐是 512，节对齐是 4096.  
  
下面 2 个 16 位的字是期望的操作系统版本，'MajorOperatingSystemVersion'和 'MinorOperatingSystemVersion'。   
  
下面 2 个 16 位的字是期望的可执行文件版本，'MajorImageVersion'和  
'MinorImageVersion'. 许多连接器不正确设置这些信息。  
  
下面 2 个 16 位的字是期望的子系统版本，'MajorSubsystemVersion 和 MinorSubsystemVersion. 这个必须是 Win32 版本或者 POSIX 版本。该版本需要正确提供，因为它被检查并使用。如果程序是 Win32-GUI 并运行在 NT4,子系统版本不是 4.0,对话框不是 3D 效果。  
  
然后是 Win32VersionValue，32 位。大部分情况下是 0。  
  
下面是 32 位的映像需要的内存数量'SizeOfImage'.是所有的头和节的总和，如果节已经对齐。它是给加载器的线索，需要多少页加载映像。  
  
下面一个是 32 位的所有头的总和，包括数据目录和节头。'SizeOfHeaders'.它也是才文件开始到第一节的偏移量。  
  
然后是 32 位的校验码'CheckSum'.对当前版本的 NT，只校验映像是否是 NT 驱动程序。对于其他可执行文件类型，不必提供这个码，可能为 0。  
  
然后是 16 的子系统 Subsystem'表明在什么系统上运行：  
    IMAGE\_SUBSYSTEM\_NATIVE (1)执行文件不需要子系统，用于驱动程序。  
    IMAGE\_SUBSYSTEM\_WINDOWS\_GUI (2)映像是 Win32 图形程序可以打开控制台      
    IMAGE\_SUBSYSTEM\_WINDOWS\_CUI (3)映像是 Win32 控制台程序，可以得到缺省控制台。  
    IMAGE\_SUBSYSTEM\_OS2\_CUI (5)映像是 OS/2 控制台，程序是 OS/2 格式。      
    IMAGE\_SUBSYSTEM\_POSIX\_CUI (7)映像使用 POSIX 控制台子系统  
Windows 95 可执行文件总是使用 Win32 subsystem,于是合法值是 2 和 3。  
  
下面是 16 位，DllCharacteristics,表明是否是 DLL,如果 0 位置 1，DLL 被通知进程结合。位 1 置 1，DLL 被通知线程脱离。位 2 置 1，DLL 被通知线程结合。位 3 置 1，DLL 被通知进程脱离。  
  
下面 4 个 32 位预留堆栈大小'SizeOfStackReserve',提交的堆栈大小'SizeOfStackCommit',预留的堆的大小'SizeOfHeapReserve'和提交的的堆的的大小'SizeOfHeapCommit'.  
  
预留数量是地址空间不是真实的 RAM，程序启动时，提交的数量是真正的分配的内存。这个值也是堆和栈根据需要增长的一个数量。  
例如：一个程序预留 1 MB 的堆并提交的堆时 64KB,该堆就从 64KB 开始，并保证可以加大到 1MB.堆将以 64KB 块增长。该堆在这里是主要堆，默认堆。一个进程可以创建多个堆如果需要的话。栈是第一个线程的栈，进程可以创建许多线程，每个都有自己的堆栈，DLLs 没有栈或者堆，于是在其映像内该值被忽略。  
  
下面是 32 位的 LoaderFlags, 没有用。  
  
然后是 32 位的 NumberOfRvaAndSizes,在随后的目录内的有效项目数。最好使用  
IMAGE\_NUMBEROF\_DIRECTORY\_ENTRIES，即 16。  
  
下面是具有 IMAGE\_NUMBEROF\_DIRECTORY\_ENTRIES(16)个成员的 IMAGE\_DATA\_DIRECTORYs 结构数组.  
  
IMAGE\_DATA\_DIRECTORY STRUCT  
  VirtualAddress    DWORD      ?  
  isize             DWORD      ?  
IMAGE\_DATA\_DIRECTORY ENDS  
  
每个目录描述了节内特定信息位置，32 bits RVA VirtualAddress 和尺寸 32 bit,各个成员索引如下（括号内为索引值）：  
    IMAGE\_DIRECTORY\_ENTRY\_EXPORT (0)输出符号目录用于 DLL      
    IMAGE\_DIRECTORY\_ENTRY\_IMPORT (1)输入符号目录      
    IMAGE\_DIRECTORY\_ENTRY\_RESOURCE (2)资源目录      
    IMAGE\_DIRECTORY\_ENTRY\_EXCEPTION (3)异常目录      
    IMAGE\_DIRECTORY\_ENTRY\_SECURITY (4)安全目录      
    IMAGE\_DIRECTORY\_ENTRY\_BASERELOC (5)重定位表      
    IMAGE\_DIRECTORY\_ENTRY\_DEBUG (6)调试目录  
    IMAGE\_DIRECTORY\_ENTRY\_COPYRIGHT (7)描述版权串      
    IMAGE\_DIRECTORY\_ENTRY\_GLOBALPTR (8)机器值      
    IMAGE\_DIRECTORY\_ENTRY\_TLS (9)Thread local storage 目录  
    IMAGE\_DIRECTORY\_ENTRY\_LOAD\_CONFIG (10)Load configuration 目录      
    IMAGE\_DIRECTORY\_ENTRY\_BOUND\_IMPORT (11)Bound import directory 目录      
    IMAGE\_DIRECTORY\_ENTRY\_IAT (12)Import Address Table 输入地址表目录  
  
例如，如果我们找到索引 7 的 2 个值 0x12000 和 33, 加载地址是 0x10000, 我们知道版权数据是在 0x10000+0x12000，版权字数为 33。如果一个特定类型的目录没有被使用，地址和尺寸都为 0。  
  
节目录 Section directories  
\-------------------  
  
节包含 2 个部分：节头 IMAGE\_SECTION\_HEADER，节数据。在数据目录之后，我们看到一个具有 NumberOfSections 个节头成员的数组，按 RVA 排序。  
  
节头包括：  
IMAGE\_SECTION\_HEADER STRUCT  
    Name1 db IMAGE\_SIZEOF\_SHORT\_NAME dup(?)  
    union Misc  
        PhysicalAddress dd  ?  
        VirtualSize dd      ?  
    ends  
    VirtualAddress dd       ?  
    SizeOfRawData dd        ?  
    PointerToRawData dd     ?  
    PointerToRelocations dd ?  
    PointerToLinenumbers dd ?  
    NumberOfRelocations dw  ?  
    NumberOfLinenumbers dw  ?  
    Characteristics dd      ?  
IMAGE\_SECTION\_HEADER ENDS  
  
  
IMAGE\_SIZEOF\_SHORT\_NAME(8)个字节的数组，组成节的名字。如果所有 8 个字节被用掉，没有 0 做为结尾。典型的名字如".data"或者".text"或者".bss". 没有必要前导'.',可以是是"CODE"或 "IAT" .注意名字不全部跟节内容有关。一个".code"节可能或没有可能包括可执行代码，可能只包括输入地址表。可能包含代码和地址表和初始化数据。要找到在节内的信息，必须通过“可选头”内的数据目录查找他们。不要依赖名字，不要假定节的原始数据起始于节的开始。  
  
下面一个成员是'PhysicalAddress'和'VirtualSize'的 32 位联合体. 在目标文件，该地址是内容被重定位的地址，在可执行文件内是内容的尺寸。实际上该域好像没有被使用，有的链接器填入尺寸有的链接器填入地址，有的链接器填入 0。  
下一个成员是'VirtualAddress',32 位，保存当节的数据加载入内存时的 RVA。  
然后是 32 位'SizeOfRawData',是四舍五入到下个 FileAlignment'倍数的大小。  
下一个是'PointerToRawData',32 位，它是从文件起始到节数据的偏移量。如果是 0,节数据不包含在文件内，在加载时被确定。  
然后是'PointerToRelocations'，32 位，和'PointerToLinenumbers'，32 位，'NumberOfRelocations'，16 位，'NumberOfLinenumbers'，16 位.所有这些信息仅仅用于目标文件。可执行文件有一个特殊的基准重定位目录，如果存在行号信息，一般包含在特殊目的的调试段或其他。  
最后一个是 32 位的'Characteristics',它是一组标志描述如何处理节的内存。  
    bit 5 (IMAGE\_SCN\_CNT\_CODE)，置 1，节内包含可执行代码。      
    bit 6 (IMAGE\_SCN\_CNT\_INITIALIZED\_DATA)置 1，节内包含的数据在执行前是确定的。      
    bit 7 (IMAGE\_SCN\_CNT\_UNINITIALIZED\_DATA) 置 1，本节包含未初始化的数据，执行前即将被初始化为 0。一般是 BSS.  
    bit 9 (IMAGE\_SCN\_LNK\_INFO) 置 1，节内不包含映象数据除了注释，描述或者其他文档外，是一个目标文件的一部分，可能是针对链接器的信息。比如哪个库被需要。  
    bit 11 (IMAGE\_SCN\_LNK\_REMOVE) 置 1，在可执行文件链接后，作为文件一部分的数据被清除。  
    bit 12 (IMAGE\_SCN\_LNK\_COMDAT) 置 1，节包含公共块数据，是某个顺序的打包的函数。  
    bit 15 (IMAGE\_SCN\_MEM\_FARDATA) 置 1，不确定。  
    bit 17 (IMAGE\_SCN\_MEM\_PURGEABLE) 置 1，节的数据是可清除的。  
    bit 18 (IMAGE\_SCN\_MEM\_LOCKED) 置 1，节不可以在内存内移动。  
    bit 19 (IMAGE\_SCN\_MEM\_PRELOAD)置 1， 节必须在执行开始前调入。  
    Bits 20 to 23 指定对齐。一般是库文件的对象对齐。  
    bit 24 (IMAGE\_SCN\_LNK\_NRELOC\_OVFL) 置 1， 节包含扩展的重定位。  
    bit 25 (IMAGE\_SCN\_MEM\_DISCARDABLE) 置 1，进程开始后节的数据不再需要。  
    bit 26 (IMAGE\_SCN\_MEM\_NOT\_CACHED) 置 1，节的 数据不得缓存。  
    bit 27 (IMAGE\_SCN\_MEM\_NOT\_PAGED) 置 1，节的 数据不得交换出去。  
    bit 28 (IMAGE\_SCN\_MEM\_SHARED) 置 1，节的数据在所有映象例程内共享，如 DLL 的初始化数据。  
    bit 29 (IMAGE\_SCN\_MEM\_EXECUTE) 置 1，进程得到“执行”访问节内存。  
    bit 30 (IMAGE\_SCN\_MEM\_READ) 置 1，进程得到“读出”访问节内存。  
    bit 31 (IMAGE\_SCN\_MEM\_WRITE)置 1，进程得到“写入”访问节内存。  
  
在节头后面我们看到节自身，他们在文件内按照'FileAlignment'字节数对齐。即在可选头和每个节的后面将添加 0。节按照他们的 RVAs 排序.当加载到 RAM,节的对齐按照 SectionAlignment。  
  
例如：一个可选头在偏移量 981 处结束，文件对齐为 512，第一个节起始于 1024。你可以借助'PointerToRawData'或者'VirtualAddress'找到节，不必用对齐找节。（节头哪里去了？此段有问题！！）  
  
    +-------------------+  
    | DOS-stub          |  
    +-------------------+  
    | file-header       |  
    +-------------------+  
    | optional header   |  
    |- - - - - - - - - -|  
    |                   |----------------+  
    | data directories  |                |  
    |                   |                |  
    |(RVAs to direc-    |-------------+  |  
    |tories in sections)|             |  |  
    |                   |---------+   |  |目录在节内的相对地址  
    |                   |         |   |  |  
    +-------------------+         |   |  |  
    |                   |-----+   |   |  |  
    | section headers   |     |   |   |  |  
    | (RVAs to section  |--+  |   |   |  |节边界的相对地址  
    |  borders)         |  |  |   |   |  |  
    +-------------------+<-+  |   |   |  |  
    |                   |     | <-+   |  |  
    | section data 1    |     |       |  |节数据  
    |                   |     | <-----+  |  
    +-------------------+<----+          |  
    |                   |                |  
    | section data 2    |                |  
    |                   | <--------------+  
    +-------------------+  
  
每个节都有一个节头，每个数据目录将指向一个节（几个数据目录可能指向同一个节），有的节可能没有数据目录指向它。  
  
节的原始数据 Sections' raw data  
\------------------  
概要-------  
所有的节调入内存后按照'SectionAlignment'对齐，FileAlignment'是节在文件内对齐字节数。 节由节头内的项目来描述，可以通过'PointerToRawData'在文件内找到节，在内存内通过'VirtualAddress'找到节，长度是'SizeOfRawData'.  
根据他们包含的内容，有几种节。一般至少有一个数据目录指向的内容保存在一个节内。  
  
代码节 code section  
\------------  
本节至少含有一个标志位'IMAGE\_SCN\_CNT\_CODE', 'IMAGE\_SCN\_MEM\_EXECUTE' and  
'IMAGE\_SCN\_MEM\_READ'的集合，并且“可选头”的成员'AddressOfEntryPoint'指向这个节内的某个地方。“可选头”的成员'BaseOfCode'将指向这个节的开始，如果把非代码放在代码之前，也有可能指向后面某个地方。一般除了可执行代码，没有其他东西，一般只有一个代码节。一般的名字如：".text", ".code", "AUTO"。  
  
数据节 data section  
\------------  
本节包含初始化过的静态变量，例如 "static int i = 5;".他含有这些位'IMAGE\_SCN\_CNT\_INITIALIZED\_DATA','IMAGE\_SCN\_MEM\_READ'及'IMAGE\_SCN\_MEM\_WRITE' 等.有的链接器把常量数据放在没有可写标志位的节内。如果部分数据是共享的，或者有其他特性的话，节将包含更多的特征位集。节一般在'BaseOfData'到'BaseOfData'+'SizeOfInitializedData'的范围内.典型名字如：'.data', '.idata', 'DATA'。  
  
bss section  
\-----------  
还有未初始化的数据，例如"static int k;"该节的'PointerToRawData'为 0，表明其内容不在文件内，特征位'IMAGE\_SCN\_CNT\_UNINITIALIZED\_DATA'指明所有内容必须在加载时间置 0。这意味着有节头，但没有节在文件内，该节被加载器创建，并且包含全 0 字节。其长度是'SizeOfUninitializedData'.典型名字如'.bss', 'BSS'。  
  
有的节数据没有被数据目录指向，其内容和结果被编译器支持，不是被链接器支持。堆栈段和堆段不是可执行文件的节，但是被加载器创建。其大小为 optional header 内的 stacksize 和 heapsize   
.  
  
版权 copyright  
\---------  
开始于一个简单的目录'IMAGE\_DIRECTORY\_ENTRY\_COPYRIGHT'.其内容是一个版权或者一个非 0 结尾的串，象"Gonkulator control application, copyright (c) 1848 Hugendubel & Cie".  
该串被使用命令行或者描述文件提供给链接器。该串不是必须的，可以被舍弃。他不是可写的，实际上程序不必访问他。链接器将找出是否有一个可以舍弃的不可写的段，创建一个名字为'.descr'的段。然后把串填入该段，让版权目录指针指向他。该节的特征字'IMAGE\_SCN\_CNT\_INITIALIZED\_DATA' 必须置 1。  
  
输出符号 exported symbols  
\----------------  
下面一个简单的事情是输出目录'IMAGE\_DIRECTORY\_ENTRY\_EXPORT'. 该目录是 DLL 内的典型目录。他包括输出函数的入口点，输出对象的地址等。该节必须是初始化数据和可读的。不可以是可废弃的，因为进程可能在运行时间调用"GetProcAddress()"找出函数的入口。该节一般称为'.edata' . 它一般被并入其他节。  
IMAGE\_EXPORT\_DIRECTORY STRUCT  
  Characteristics           DWORD      ?  
  TimeDateStamp             DWORD      ?  
  MajorVersion              WORD       ?  
  MinorVersion              WORD       ?  
  nName                     DWORD      ?  
  nBase                     DWORD      ?  
  NumberOfFunctions         DWORD      ?  
  NumberOfNames             DWORD      ?  
  AddressOfFunctions        DWORD      ?  
  AddressOfNames            DWORD      ?  
  AddressOfNameOrdinals     DWORD      ?  
IMAGE\_EXPORT\_DIRECTORY ENDS  
  
输出表结构('IMAGE\_EXPORT\_DIRECTORY')包含一个头和输出数据，即符号名，序号及到入口点的偏移。  
首先，是一个 32 位的'Characteristics'，一般为 0。然后是 32 位的'TimeDateStamp',输出表创建的时间，并非总是有效的，有些链接器置 0。然后是 2 个 16 位的版本信息 MajorVersion 和 MinorVersion，一般置 0.  
  
下面是 32 位的'Name'，这是一个指向以 0 结尾的字符串的 RVA，名字是必须的，防止 DLL 文件改名。  
然后是 32 位的'Base'.下一个是 32 位的输出项目总数'NumberOfFunctions'.除了序数外，也可能以名字输出。下面一个是 32 位的输出名字的总数'NumberOfNames'.在多数情况，每个输出项有一个确切的名字，将以名字使用它。但一个项目可能有多个名字，或者没有名字，这种情况只能够以序号来访问，不赞成只以序号输出，带来维护问题。  
  
下面 32 位是 AddressOfFunctions，是输出项目列表的 RVA，它指向具有'NumberOfFunctions'个数元素的数组。每个成员是输出函数或变量的 RVA。该列表有两个怪事，输出的 RVA 可能是 0，他没有被使用。第二如果 RVA 指向包含输出目录的节，这是一个转发输出。转发输出是一个指向另一个文件输出的指针。这样就使用另一个文件内的被指向的输出。输出序号是 AddressOfFunctions 数组的索引＋上面提到的 BASE 值。多数情况下，'Base'＝1,意思是第一个输出的序号为 1，第二个是 2。  
  
下面是指向其成员为指向符号名的 32 位 RVA 的数组'AddressOfNames',和一个指向 16 位序数的数组的 32 位 RVA AddressOfNameOrdinals'. 两个数组都含有'NumberOfNames'个元素.符号名可能全部丢失，所以'AddressOfNames'是 0. 否则被指向的数组并行运行，'AddressOfNames'数组包含指向 0 结尾的输出名的 RVAs。名字都按照字母顺序保存，便于高效检索名字。根据 PE 规范，AddressOfNameOrdinals 数组含有相应名字的序数，但实际上含有 AddressOfFunctions 数组的索引。序数＝BASE+INDEX。  
  
上述 3 表的示意图：  
  
    AddressOfFunctions  函数地址指向一个地址数组  
           |  
           |  
           v  
    exported RVA with ordinal 'Base'  //带有序号的输出函数的 RVA  
    exported RVA with ordinal 'Base'+1  
    ...  
    exported RVA with ordinal 'Base'+'NumberOfFunctions'-1  
  
    AddressOfNames                  AddressOfNameOrdinals  
           |                              |  
           |                              |  
           v                              v  
    RVA to first name           <-> Index of export for first name  
    RVA to second name          <-> Index of export for second name  
    ...                             ...  
    RVA to name 'NumberOfNames' <-> Index of export for name 'NumberOfNames'  
名字地址指向名字 RVA 数组                “名字序数地址”指向名字序数数组  
  
要按照序数找出输出符号：减去'Base'得到索引，跟随 AddressOfFunctions RVA 找到输出数组，用索引定位输出符号 RVA 所在元素。如果没有指向输出节，你就完成任务了。否则，如果指向一个描述输出 DLL 名字和序数的串，你必须查找被转发的输出项。  
要按照名字找出输出符号，沿着 AddressOfNames'RVA，找到指向包含输出名字的 RVAs 的数组，其中每个 RVA 指向一个名字。搜索名字，使用名字的索引在 AddressOfNameOrdinals 数组得到与相应名字对应的 16 位数（按照 PE 规范，此数值为输出函数的序号，需要减去 BASE 得到索引）。根据经验，这就是索引，不是序号，不必减去 BASE，直接使用该索引在'AddressOfFunctions'数组找出输出函数的的 RVA 或者是一个指向转发串。  
  
输入表 imported symbols  
\----------------  
当编译器发现一个调用的函数在其他可执行文件（一般是 DLL）内，它将不知道这个情况，简单输出一个调用符号的指令，其地址由链接器添上。链接器使用输入库查询哪个 DLL 哪个符号被引入。为所有引入的符号产生 STUB，每个由一个跳转指令构成，STUB 是真正的调用目标，这些跳转指令真正跳到一个从地址表内取出的地址。在复杂的应用中，当"\_\_declspec(dllimport)" 被使用的时候，编译器知道函数被引入，并输出一个对输入表内的地址的调用。无论如何，DLL 内的函数地址总是必须的，来自输出 DLL 的输出目录的地址被提供给加载器，加载器知道哪个符号需要查找，通过搜索输入目录修正地址。下面是一个例子：  
  
一个带有/不带有声明\_\_declspec(dllimport)的调用如下：  
  
源程序：  
        int symbol(char \*);  
        \_\_declspec(dllimport) int symbol2(char\*);  
        void foo(void)  
        {  
            int i=symbol("bar");  
            int j=symbol2("baz");  
        }  
      
汇编程序：  
        ...  
        call \_symbol                 ; 无 declspec(dllimport)  
        ...  
        call \[\_\_imp\_\_symbol2\]        ; 有 declspec(dllimport)  
        ...  
第一种情况，没有声明，编译器不知道'\_symbol'在一个 DLL，于是链接器必须提供此函数。因为函数不在那里，它将提供一个 STUB 函数对应输入符号，作为一个间接跳转。所有的输入 STUB 的集合称作转换区或跳板，因为跳到那里为的是跳到另一个地方。该跳板典型的位于代码区，不是输入目录的一部分。每个函数的 STUB 是一个跳转到 DLL 的实际函数的 JUMP，跳板区看起来如下：  
    \_symbol:        jmp  \[\_\_imp\_\_symbol\]  
    \_other\_symbol:  jmp  \[\_\_imp\_\_other\_\_symbol\]  
    ...  
这就是意味着，如果你使用不带声明的输入符号，链接器就产生包含间接跳转的跳板区，如果指定了声明"\_\_declspec(dllimport)", 编译器就自己做这个工作，跳板区是不必要的。它也意味着，如果你引入变量或其他东西，你必须指定"\_\_declspec(dllimport)",因为带有 JMP 的 STUB 只适合于函数，不适合于变量等。  
  
在任何情况下，符号的地址存在于一个地址'\_\_imp\_x'.所有这些地址组成所谓的输入地址表，通过 DLL 的引入库提供给链接器。输入地址表如下：  
   \_\_imp\_\_symbol:   0xBEFFA100  
   \_\_imp\_\_symbol2:  0x40100  
   \_\_imp\_\_symbol3:  0x300100  
   ...  
  
输入地址表是数据目录的一部分，被 IMAGE\_DIRECTORY\_ENTRY\_IAT 目录指针指向。一些链接器不设置这个目录项，照样工作，显然加载器可以不使用目录 IMAGE\_DIRECTORY\_ENTRY\_IAT 就可以解决问题。链接器不知道这个表内的地址，链接器插入哑元（即函数名的 RVAs），在加载时，加载器用输出 DLL 的输出目录修补这些哑元。注意这是 C 规范，有其他程序编译环境不需要输入库。他们需要产生输入地址表，C 编译器倾向于使用输入库，方便于它们的链接器。  
  
我们要看一下一个输入目录如何建立，于是加载器才可以使用。  
输入目录必须驻留在一个节，该节是初始化的和可读的，输入目录是一个 IMAGE\_IMPORT\_DESCRIPTORs 数组，每个 DLL 都使用一个元素，该列表中止于一个全 0 的数组元素。  
一个 IMAGE\_IMPORT\_DESCRIPTOR 是一个具有如下元素的结构：  
  
IMAGE\_IMPORT\_DESCRIPTOR STRUCT  
    union  
        Characteristics dd      ?  
        OriginalFirstThunk dd   ?  
   ends  
    TimeDateStamp dd    ?  
    ForwarderChain dd   ?  
    Name1 dd            ?  
    FirstThunk dd       ?  
IMAGE\_IMPORT\_DESCRIPTOR ENDS  
  
    OriginalFirstThunk 32 位 RVA ，指向一个以 0 结尾的 IMAGE\_THUNK\_DATAs 数组。每个成员描述一个输入函数。  
    TimeDateStamp  32 位时间戳。  
    ForwarderChain 输入函数第一个转发链的 32 位索引  
    Name 指向 DLL 名字的 32 位 RVA。          
    FirstThunk 指向 IMAGE\_THUNK\_DATAs 数组的 32 位 RVA，它是输入地址表的一部分，将发生变化。Thunk 的意思就是查找，虚实变换的意思。  
  
IMAGE\_THUNK\_DATA32 STRUCT  
    union u1  
        ForwarderString dd  ?  
        Function dd         ?  
        Ordinal dd          ?  
        AddressOfData dd    ?  
    ends  
IMAGE\_THUNK\_DATA32 ENDS  
  
以下是 VC98，WINNT.h 内的定义：  
typedef struct \_IMAGE\_THUNK\_DATA64 {  
    union {  
        PBYTE  ForwarderString;  
        PDWORD Function;  
        ULONGLONG Ordinal;  
        PIMAGE\_IMPORT\_BY\_NAME  AddressOfData;  
    } u1;  
} IMAGE\_THUNK\_DATA64;  
typedef IMAGE\_THUNK\_DATA64 \* PIMAGE\_THUNK\_DATA64;  
  
于是每个 IMAGE\_IMPORT\_DESCRIPTOR 成员给出输出 DLL 的名字，2 个 RVAs 指向 IMAGE\_THUNK\_DATAs 数组,数组的最后一个成员全部填充 0 表示结束。每个 IMAGE\_THUNK\_DATA 成员有一个指向 IMAGE\_IMPORT\_BY\_NAME 的 RVA，用以描述输入函数。有趣的是这些数组并行运行，他们指向相同的  
IMAGE\_IMPORT\_BY\_NAMEs（按名输入）.  
  
这是 IMAGE\_IMPORT\_DESCRIPTOR 必须的内容:  
  
     OriginalFirstThunk      FirstThunk  
            |                    |  
            |                    |  
            |                    |  
            V                    V  
            0-->    func1     <--0  
            1-->    func2     <--1  
            2-->    func3     <--2  
            3-->    foo       <--3  
            4-->    mumpitz   <--4  
            5-->    knuff     <--5  
            6-->0            0<--6      /\* 最后的 RVA 是 0 \*/  
  
IMAGE\_IMPORT\_BY\_NAME STRUCT  
    Hint dw     ?  
    Name1 db    ?  
IMAGE\_IMPORT\_BY\_NAME ENDS  
  
中间的名字是要讨论的 IMAGE\_IMPORT\_BY\_NAMEs.每个包括一个 16 位的 hint，一个没有确定字节数的 name，hint 是一个输出 DLL 函数名字表（见前面输出目录）的索引，索引处的名字被测试，如果不匹配，则用二进制搜索来找到名字。有的链接器只是简单置 1，或者其他任意数字。只是导致第一次搜索企图失败，然后强迫使用二进制搜索名字。  
如果想从 DLL "knurr"查找输入函数"foo"的信息,首先数据目录内找到 IMAGE\_DIRECTORY\_ENTRY\_IMPORT 项，得到 RVA,在 raw section data 找到那个地址，现在我们有了一个数组 IMAGE\_IMPORT\_DESCRIPTORs. 通过检查由 NAME 指向的的字符串，得到与 DLL "knurr"有关的数组成员。当你发现了正确的 IMAGE\_IMPORT\_DESCRIPTOR,沿着它的 OriginalFirstThunk 得到一个指针指向一个 IMAGE\_THUNK\_DATAs 数组;检查每个 RVAs 找到"foo".  
  
为什么有两个指针列表指向 IMAGE\_IMPORT\_BY\_NAMEs？因为运行时，程序不需要输入函数名，需要的是地址，这里是地址表再次出现的地方，加载器在每个涉及的 DLL 输出目录内查找输入符号，使用 DLL 入口点的线性地址替换掉 FirstThunk 列表内的 IMAGE\_THUNK\_DATA 元素。记住带有标号"\_\_imp\_\_symbol 的地址列表、被数据目录项  
IMAGE\_DIRECTORY\_ENTRY\_IAT 指向的输入地址表，也确切地被'FirstThunk'指向. (假如输入来自几个不同的 DLL，输入地址表由所有 DLL 的'FirstThunk'数组构成，目录项 IMAGE\_DIRECTORY\_ENTRY\_IAT 可能丢失，输入依然工作的很好。'OriginalFirstThunk'数组保持不变，可以通过它查找输入名的的原始列表。  
  
现在输入已经用正确线性地址打了补丁，如下所示：  
     OriginalFirstThunk      FirstThunk  
            |                    |  
            |                    |  
            |                    |  
            V                    V  
            0-->    func1        0-->  exported func1  
            1-->    func2        1-->  exported func2  
            2-->    func3        2-->  exported func3  
            3-->    foo          3-->  exported foo  
            4-->    mumpitz      4-->  exported mumpitz  
            5-->    knuff        5-->  exported knuff  
            6-->0            0<--6  
  
这是最简单情况的基本结构，下面学习输入目录。  
  
首先，IMAGE\_THUNK\_DATA 数组内的 IMAGE\_ORDINAL\_FLAG 的最高位可以置 1，这种情况没有符号名信息在列表内，符号纯粹用序号引入。可以通过检查 IMAGE\_THUNK\_DATA.低字得到序号。通过序号引入的方式不提倡，以名字引入更安全，因为输出序号可能因输出 DLL 的版本不同而改变。  
第二,存在所谓的绑定输入"bound imports".考虑加载器的工作，当一个可执行文件需要运行一个 DLL 内的函数时，加载器加载 DLL，找到输出目录，查找函数的 RVA，计算函数的入口点，然后把所谓的地址补入 FirstThunk'列表内。考虑到程序员是聪明的，为不发生冲突的 DLL 提供唯一的优先加载地址，我们假定函数入口点总是一样的，在链接时他们可以被计算然后补入'FirstThunk'列表，这就是所谓的绑定输入。  
  
必须注意一点：用户的 DLL 可能是不同的版本或者需要重定位 DLL，这样就作废先前补入的'FirstThunk'列表，这种情况下，加载器依然能够遍历'OriginalFirstThunk'列表，找出输入符号并重新补入'FirstThunk'列表内.加载器知道这是必须的，如果 a) DLL 版本不相符，b)DLL 必须重定位。  
  
确定是否有重定位，对加载器来说不是问题，但是如何知道版本的不同？这是 IMAGE\_IMPORT\_DESCRIPTOR 中的'TimeDateStamp'出现的地方，如果是 0，输入表没有绑定，加载器必须总是修复入口点，否则输入表被绑定，'TimeDateStamp'必须与 DLL 的文件头的时间戳相符，如果不相符，加载器假定执行文件绑定了一个错误的 DLL，将重新补入输入列表。//输入列表就是 FirstThunk 所指的地址列表。  
  
还有一个怪事就是输入表内的转发器"forwarders"。一个 DLL 可以输出一个没有在本 DLL 内定义的符号，而是从另一个 DLL 引入的，这样的符号成为被转发的，现在，通过检查没有包含真正入口点的 DLL 的时间戳，很明显不能确定入口点是否有效。所以被转发的符号的入口点必须总是被修复，在输入列表内，必须被找到被转发符号的输入，加载器可以补入它。这个靠'ForwarderChain'.这是 thunk 列表的索引，在索引位置的输入是一个被转发的输出，在'FirstThunk'列表的该位置的内容是下一个转发的输出索引，直到－1 为止，表明没有转发了。如果根本没有转发，'ForwarderChain' 置-1 。  
这是所谓的老式绑定。我们总结一下：  
  
我假定你已经找到了 IMAGE\_DIRECTORY\_ENTRY\_IMPORT，沿着它找到输入目录，在某个节内。现在你在 IMAGE\_IMPORT\_DESCRIPTORs 数组的开始了，最后一个是全 0。要解码一个 IMAGE\_IMPORT\_DESCRIPTORs, 首先找到'Name',循着 RVA 可以找到输出的 DLL 名字。下面决定是否输入绑定与否了。如果时间戳非 0，说明绑定。如果绑定了，现在通过比较时间戳检查是否 DLL 版本匹配。现在循着'OriginalFirstThunk'的 RVA 到达 IMAGE\_THUNK\_DATA 数组，顺着往下走，每个成员是一个指向 IMAGE\_IMPORT\_BY\_NAME 结构的 RVA，除非高位置 1 表示没有名字，只有序号可用。循着这个 RVA,跳过 2 个字节（hint), 现在我们得到一个以 0 结尾的引入函数名了。  
如果输入绑定了，要找到提供的入口点地址，沿着'FirstThunk'并行遍历'OriginalFirstThunk'数组;数组成员是入口点的线性地址。  
  
有的链接器设置 IMAGE\_IMPORT\_DESCRIPTOR 内的'OriginalFirstThunk'为 0，只创建'FirstThunk'数组，非常明显，这个输入目录不能发现，因为无法找到函数名，必要的用于修复输入输入的信息丢失了。这种情况下，必须沿着'FirstThunk'数组得到输入符号名，永远得不到预先打过补丁的入口地址。  
  
最后一个关于输入目录的新式绑定。当用这个的时候，'TimeDateStamp'设置为全 1，没有转发链 forwarderchain;所有引入符号的地址被修补，无论是否被转发。依然需要 DLL 的版本，需要区别转发符号和普通符号。为了这个目的，IMAGE\_DIRECTORY\_ENTRY\_BOUND\_IMPORT 目录被建立，不在节内，而是在头内，在节头的后面，第一节的前面。这个目录告诉我们，对应每个用到的 DLL，另外哪个 DLL 有转发输出。  
该结构是一个 IMAGE\_BOUND\_IMPORT\_DESCRIPTOR, 组成如下：  
  
IMAGE\_BOUND\_IMPORT\_DESCRIPTOR STRUCT  
    TimeDateStamp dd    ?  
    OffsetModuleName dw ?  
    NumberOfModuleForwarderRefs dw ?  
IMAGE\_BOUND\_IMPORT\_DESCRIPTOR ENDS  
  
DLL 的 32 位'TimeDateStamp' ;  
16 位的'OffsetModuleName',从目录的开始到 DLL 名字的偏移量。  
16 位的'NumberOfModuleForwarderRefs'给出该 DLL 用于转发的 DLL 个数。  
  
紧跟这个结构，是'IMAGE\_BOUND\_FORWARDER\_REF's 结构，告诉你本 DLL 转发的 DLL 的名字和版本。该结构如下:  
  
IMAGE\_BOUND\_FORWARDER\_REF STRUCT  
    TimeDateStamp dd    ?  
    OffsetModuleName dw ?  
    Reserved dw         ?  
IMAGE\_BOUND\_FORWARDER\_REF ENDS  
  
32 位时间戳 'TimeDateStamp';  
16 位的'OffsetModuleName', 被转发的 DLL 名字的偏移量。  
16 位保留未用的。  
  
跟随'IMAGE\_BOUND\_FORWARDER\_REF's 后面的是下一个'IMAGE\_BOUND\_IMPORT\_DESCRIPTOR'。该列表以全 0 结束。  
  
现在你有一个新绑定的输入目录，加载所有的 DLL，使用目录指针 IMAGE\_DIRECTORY\_ENTRY\_BOUND\_IMPORT 找到 IMAGE\_BOUND\_IMPORT\_DESCRIPTOR, 扫描它检查时间戳是否与加载的 DLL 相符，如果不的话，修复输入目录内的 FirstThunk'数组内容即函数地址。  
  
  
资源 resources  
\---------  
资源比如对话框，菜单，图标等等保存在由 IMAGE\_DIRECTORY\_ENTRY\_RESOURCE 指向的数据目录内. 那是一个至少含有一个位集合'IMAGE\_SCN\_CNT\_INITIALIZED\_DATA'和'IMAGE\_SCN\_MEM\_READ'的节。   
资源的根基是一个'IMAGE\_RESOURCE\_DIRECTORY';它包含几个'IMAGE\_RESOURCE\_DIRECTORY\_ENTRY'项 ，每个指向一个'IMAGE\_RESOURCE\_DIRECTORY'.这样你得到一个'IMAGE\_RESOURCE\_DIRECTORY'树。 'IMAGE\_RESOURCE\_DIRECTORY\_ENTRY'是树叶，指向实际的资源数据。  
层次是这样的，一个目录是根，指向一些目录，每一个针对一个资源类型。这些目录指向子目录，每个含有一个名字或者 ID，并指向一个为该资源提供的语言目录。对于每个语言你会发现资源入口，它指向数据。  
下面是没有指向数据的树：  
                           (root)  
                              |  
             +----------------+------------------+  
             |                |                  |  
            menu            dialog             icon  
             |                |                  |  
       +-----+-----+        +-+----+           +-+----+----+  
       |           |        |      |           |      |    |  
    "main"      "popup"   0x10   "maindlg"    0x100 0x110 0x120  
       |            |       |      |           |      |    |   
   +---+-+          |       |      |           |      |    |  
   |     |     default   english   default    def.   def.  def.  
german english  
  
IMAGE\_RESOURCE\_DIRECTORY STRUCT  
    Characteristics dd      ?  
    TimeDateStamp dd        ?  
    MajorVersion dw         ?  
    MinorVersion dw         ?  
    NumberOfNamedEntries dw ?  
    NumberOfIdEntries dw    ?  
IMAGE\_RESOURCE\_DIRECTORY ENDS  
  
IMAGE\_RESOURCE\_DIRECTORY 结构包括：  
32 位没有用到的'Characteristics';  
32 位'TimeDateStamp'给出资源创建时间。  
16 位'MajorVersion'和 16 位'MinorVersion', 维护资源版本。  
16 位'NumberOfNamedEntries，带名字的项目个数和 16 位'NumberOfIdEntries'，ID 项目个数.  
  
紧接在该结构后面的是'NumberOfNamedEntries'+'NumberOfIdEntries'个结构。他们是 'IMAGE\_RESOURCE\_DIRECTORY\_ENTRY'目录项,他们可能指向下一个 IMAGE\_RESOURCE\_DIRECTORY'或者实际的资源数据。  
  
IMAGE\_RESOURCE\_DIRECTORY\_ENTRY 结构：  
  
IMAGE\_RESOURCE\_DIRECTORY\_ENTRY STRUCT  
    union  
        rName  RECORD NameIsString:1,NameOffset:31  
        Name1 dd ?         //名字是名字，串是串，各不相同的。  
        Id dw ?  
    ends  
    union  
        OffsetToData dd ?  
        rDirectory  RECORD DataIsDirectory:1,OffsetToDirectory:31  
    ends  
IMAGE\_RESOURCE\_DIRECTORY\_ENTRY ENDS  
  
32 位资源标识或者其描述的目录  
32 位数据偏移量或者下个子目录的偏移量。  
  
ID 的意义依赖与在树内的层次，ID 可能是一个数字（高位清 0）或者名字（高位置 1），如果是名字，低 31 位是从资源节的原始数据开始到名字的偏移量。名字是 16 位长度，以宽字符结尾，不是 0。  
如果在根目录下，如果 ID 是数字，是资源类型：  
    1: cursor  
    2: bitmap  
    3: icon  
    4: menu  
    5: dialog  
    6: string table  
    7: font directory  
    8: font  
    9: accelerators  
    10: unformatted resource data  
    11: message table  
    12: group cursor  
    14: group icon  
    16: version information  
任何其他数字都是自定义的，任何带类型名字的资源类型都是自定义的。更深一层的话，ID 是资源 ID 或者资源名字。  
如果再深入一层，ID 必须是数字，它是特定资源实例的语言 ID，例如，可以具有不同本地化的语言的对话框。他们使用统一的资源 ID，系统会选择基于线程的地域加载对话框，反过来反应用户的区域设置。如果对于线程本地没有资源发现，系统首先试图发现中立语言为资源的区域。如果还不能够发现，带有最小语言 ID 的实例将被使用。解码语言 ID，拆分为主语言 ID 和子语言 ID，使用宏 PRIMARYLANGID() 和 SUBLANGID(),分别是 0－9，10－15。其值定义在"winresrc.h".  
语言资源只被加速键，对话框，菜单，串表支持，其他资源类型必须是 LANG\_NEUTRAL/SUBLANG\_NEUTRAL.  
  
要找出资源目录的下级目录是否是另一个目录，检查偏移量的高位，如果置 1，其余 31 位是从资源原始数据开始到下一个目录的偏移量。格式还是 IMAGE\_RESOURCE\_DIRECTORY 后面跟着 IMAGE\_RESOURCE\_DIRECTORY\_ENTRYs 项目.  
  
如果高位清 0，偏移量是从节开始到资源原始数据描述结构（一个 IMAGE\_RESOURCE\_DATA\_ENTRY）的偏移量。一个 IMAGE\_RESOURCE\_DATA\_ENTRY 包括  
IMAGE\_RESOURCE\_DATA\_ENTRY STRUCT  
    OffsetToData dd ?  
    Size1 dd        ?  
    CodePage dd     ?  
    Reserved dd     ?  
IMAGE\_RESOURCE\_DATA\_ENTRY ENDS  
  
32 位'OffsetToData' 从资源节开始到原始数据的偏移量，32 位数据大小 Size，32 位'CodePage'  
和 32 保留。  
原始数据格式依赖于资源类型，任何字符串资源都是 UNICODE 格式。  
  
重定位 relocations  
\-----------  
最后一个数据目录是重定位，基重定位目录被 IMAGE\_DIRECTORY\_ENTRY\_BASERELOC 指向，典型的包括一个自己的节，名字是".reloc"以及 IMAGE\_SCN\_CNT\_INITIALIZED\_DATA,IMAGE\_SCN\_MEM\_DISCARDABLE 和  
IMAGE\_SCN\_MEM\_READ 位集合。  
  
如果映象没有被加载器调入优先地址，则该数据被加载器使用。这种情况下给链接器填入的地址无效了。加载器必须修复用于静态变量的绝对地址，串等。  
  
IMAGE\_BASE\_RELOCATION STRUCT  
    VirtualAddress dd   ?  
    SizeOfBlock dd      ?  
IMAGE\_BASE\_RELOCATION ENDS  
重定位目录是一系列块，每个块包括重定位信息针对 4K 的映象。一个块起始于一个  
'IMAGE\_BASE\_RELOCATION'结构，包括 32 位的'VirtualAddress'和 32 位的'SizeOfBlock'.随后是实际的重定位数据每个都是 16 位的。'VirtualAddress'是本块要应用的基地址。'SizeOfBlock' 是整个块的大小。后面的重定位的数目是('SizeOfBlock'-sizeof(IMAGE\_BASE\_RELOCATION))/2，重定位信息以 VirtualAddress'＝0 的 IMAGE\_BASE\_RELOCATION 结构结束。  
  
每个 16 位重定位信息包括低 12 位的重定位位置和高 4 位的重定位类型。要得到重定位的 RVA,IMAGE\_BASE\_RELOCATION'的'VirtualAddress'需要加上 12 位位置偏移量. 类型是下列之一：  
  
    IMAGE\_REL\_BASED\_ABSOLUTE (0) 使块按照 32 位对齐，位置为 0。  
    IMAGE\_REL\_BASED\_HIGH (1) 高 16 位必须应用于偏移量所指高字 16 位。  
    IMAGE\_REL\_BASED\_LOW (2)  低 16 位必须应用于偏移量所指低字 16 位。  
    IMAGE\_REL\_BASED\_HIGHLOW (3) 全部 32 位应用于所有 32 位。.  
    IMAGE\_REL\_BASED\_HIGHADJ (4) 需要 32 位，高 16 位位于偏移量，低 16 位位于下一个偏移量数组元素，组合为一个带符号数，加上 32 位的一个数，然后加上 8000 然后把高 16 位保存在偏移量的 16 位域内。  
    IMAGE\_REL\_BASED\_MIPS\_JMPADDR (5)        Unknown  
    IMAGE\_REL\_BASED\_SECTION (6)        Unknown  
    IMAGE\_REL\_BASED\_REL32 (7)        Unknown  
  
举例：  
    0x00004000      (32 bits, starting RVA)  
    0x00000010      (32 bits, size of chunk)  
    0x3012          (16 bits reloc data)  
    0x3080          (16 bits reloc data)  
    0x30f6          (16 bits reloc data)  
    0x0000          (16 bits reloc data)  
    0x00000000      (next chunk's RVA)  
    0xff341234  
第一块描述重定位起始于 RVA 0x4000 长度 16 字节。因为头用去 8 字节，一个重定位用 2 个字节，总共（16－8)/2＝4 个重定位，第一个重定位被用于 0x4012，下一个重定位于 4080,第三个定位于 0x40f6. 最后一个无用。最后一个结束。  
  
附录：Appendix: hello world  
\---------------------  
The program will be the equivalent of  
  
    #include <stdio.h>  
    int main(void)  
    {  
        puts(hello,world);  
        return 0;  
    }  
  
用 Win32 函数代替 C runtime:  
    #define STD\_OUTPUT\_HANDLE -11UL  
    #define hello "hello, world\\n"  
    \_\_declspec(dllimport) unsigned long \_\_stdcall  
    GetStdHandle(unsigned long hdl);  
    \_\_declspec(dllimport) unsigned long \_\_stdcall  
    WriteConsoleA(unsigned long hConsoleOutput,  
                    const void \*buffer,  
                    unsigned long chrs,  
                    unsigned long \*written,  
                    unsigned long unused  
                    );  
  
    static unsigned long written;  
    void startup(void)  
    {  
        WriteConsoleA(GetStdHandle(STD\_OUTPUT\_HANDLE),hello,sizeof(hello)-1,&written,0);  
        return;  
    }  
  
汇编语言  
    startup:  
                ; 参数 WriteConsole(), 逆向  
    6A 00                     push      0x00000000  
    68 ?? ?? ?? ??            push      offset \_written  
    6A 0D                     push      0x0000000d  
    68 ?? ?? ?? ??            push      offset hello  
                ; 参数 GetStdHandle()  
    6A F5                     push      0xfffffff5  
    2E FF 15 ?? ?? ?? ??      call      dword ptr cs:\_\_imp\_\_GetStdHandle@4  
                ; result is last parameter for WriteConsole()  
    50                        push      eax  
    2E FF 15 ?? ?? ?? ??      call      dword ptr cs:\_\_imp\_\_WriteConsoleA@20  
    C3                        ret         
  
    hello:  
    68 65 6C 6C 6F 2C 20 77 6F 72 6C 64 0A   "hello, world\\n"  
    \_written:  
    00 00 00 00  
  
下面是链接器：  
我们需要找出函数 WriteConsoleA() and GetStdHandle().他们在"kernel32.dll". 此为输入库部分。我们现在生成可执行文件。问号在后面被找出。  
  
DOS-stub, 起始于 0x0，0x40 字节:  
    00 | 4d 5a 00 00 00 00 00 00 00 00 00 00 00 00 00 00  
    10 | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  
    20 | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  
    30 | 00 00 00 00 00 00 00 00 00 00 00 00 40 00 00 00  
只是一个头，前面带有签字"MZ"后面是 e\_lfanew 指针。然后是 PE 签名，起始于 0x40，0x4 字节:  
         50 45 00 00  
  
下面是文件头，起始于 0x44,0x14 字节:  
    Machine                     4c 01       ; i386  
    NumberOfSections            02 00       ; code and data  
    TimeDateStamp               00 00 00 00 ; who cares?  
    PointerToSymbolTable        00 00 00 00 ; unused  
    NumberOfSymbols             00 00 00 00 ; unused  
    SizeOfOptionalHeader        e0 00       ; constant  
    Characteristics             02 01       ; executable on 32-bit-machine  
  
下面是 optional header,起始于 0x58，0x60 bytes long:  
    Magic                       0b 01       ; constant  
    MajorLinkerVersion          00          ; I'm version 0.0 :-)  
    MinorLinkerVersion          00          ;  
    SizeOfCode                  20 00 00 00 ; 32 bytes of code  
    SizeOfInitializedData       ?? ?? ?? ?? ; yet to find out  
    SizeOfUninitializedData     00 00 00 00 ; we don't have a BSS  
    AddressOfEntryPoint         ?? ?? ?? ?? ; yet to find out  
    BaseOfCode                  ?? ?? ?? ?? ; yet to find out  
    BaseOfData                  ?? ?? ?? ?? ; yet to find out  
    ImageBase                   00 00 10 00 ; 1 MB, chosen arbitrarily  
    SectionAlignment            20 00 00 00 ; 32-bytes-alignment  
    FileAlignment               20 00 00 00 ; 32-bytes-alignment  
    MajorOperatingSystemVersion  04 00      ; NT 4.0  
    MinorOperatingSystemVersion  00 00      ;  
    MajorImageVersion           00 00       ; version 0.0  
    MinorImageVersion           00 00       ;  
    MajorSubsystemVersion       04 00       ; Win32 4.0  
    MinorSubsystemVersion       00 00       ;  
    Win32VersionValue           00 00 00 00 ; unused?  
    SizeOfImage                 ?? ?? ?? ?? ; yet to find out  
    SizeOfHeaders               ?? ?? ?? ?? ; yet to find out  
    CheckSum                    00 00 00 00 ; not used for non-drivers  
    Subsystem                   03 00       ; Win32 console  
    DllCharacteristics          00 00       ; unused (not a DLL)  
    SizeOfStackReserve          00 00 10 00 ; 1 MB stack  
    SizeOfStackCommit           00 10 00 00 ; 4 KB to start with  
    SizeOfHeapReserve           00 00 10 00 ; 1 MB heap  
    SizeOfHeapCommit            00 10 00 00 ; 4 KB to start with  
    LoaderFlags                 00 00 00 00 ; unknown  
    NumberOfRvaAndSizes         10 00 00 00 ; constant  
  
计划 2 个节，一个是代码，一个是数据（包括数据，常量和输入目录），没有重定位，没有其他资源。没有 BSS 段。变量'written'填入初始化数据。节对齐和文件对齐都是 32 字节。  
现在建立数据目录，开始于 0xb8，0x80 字节长。只有引入目录被使用。  
    Address        Size  
    00 00 00 00    00 00 00 00         ; IMAGE\_DIRECTORY\_ENTRY\_EXPORT (0)  
    ?? ?? ?? ??    ?? ?? ?? ??         ; IMAGE\_DIRECTORY\_ENTRY\_IMPORT (1)  
    00 00 00 00    00 00 00 00         ; IMAGE\_DIRECTORY\_ENTRY\_RESOURCE (2)  
    00 00 00 00    00 00 00 00         ; IMAGE\_DIRECTORY\_ENTRY\_EXCEPTION (3)  
    00 00 00 00    00 00 00 00         ; IMAGE\_DIRECTORY\_ENTRY\_SECURITY (4)  
    00 00 00 00    00 00 00 00         ; IMAGE\_DIRECTORY\_ENTRY\_BASERELOC (5)  
    00 00 00 00    00 00 00 00         ; IMAGE\_DIRECTORY\_ENTRY\_DEBUG (6)  
    00 00 00 00    00 00 00 00         ; IMAGE\_DIRECTORY\_ENTRY\_COPYRIGHT (7)  
    00 00 00 00    00 00 00 00         ; IMAGE\_DIRECTORY\_ENTRY\_GLOBALPTR (8)  
    00 00 00 00    00 00 00 00         ; IMAGE\_DIRECTORY\_ENTRY\_TLS (9)  
    00 00 00 00    00 00 00 00         ; IMAGE\_DIRECTORY\_ENTRY\_LOAD\_CONFIG (10)  
    00 00 00 00    00 00 00 00         ; IMAGE\_DIRECTORY\_ENTRY\_BOUND\_IMPORT (11)  
    00 00 00 00    00 00 00 00         ; IMAGE\_DIRECTORY\_ENTRY\_IAT (12)  
    00 00 00 00    00 00 00 00         ; 13  
    00 00 00 00    00 00 00 00         ; 14  
    00 00 00 00    00 00 00 00         ; 15  
  
下面是节头，首先是代码节，包括上述汇编，32 字节长，开始于 0x138，长度 x28：  
    Name            2e 63 6f 64 65 00 00 00     ; ".code"  
    VirtualSize         00 00 00 00             ; unused  
    VirtualAddress      ?? ?? ?? ??             ; yet to find out  
    SizeOfRawData       20 00 00 00             ; size of code  
    PointerToRawData    ?? ?? ?? ??             ; yet to find out  
    PointerToRelocations 00 00 00 00            ; unused  
    PointerToLinenumbers 00 00 00 00            ; unused  
    NumberOfRelocations  00 00                  ; unused  
    NumberOfLinenumbers  00 00                  ; unused  
    Characteristics     20 00 00 60             ; code, executable, readable  
  
第二个节包括数据，起始于 0x160，长度 x28：  
    Name            2e 64 61 74 61 00 00 00     ; ".data"  
    VirtualSize         00 00 00 00             ; unused  
    VirtualAddress      ?? ?? ?? ??             ; yet to find out  
    SizeOfRawData       ?? ?? ?? ??             ; yet to find out  
    PointerToRawData    ?? ?? ?? ??             ; yet to find out  
    PointerToRelocations 00 00 00 00            ; unused  
    PointerToLinenumbers 00 00 00 00            ; unused  
    NumberOfRelocations  00 00                  ; unused  
    NumberOfLinenumbers  00 00                  ; unused  
    Characteristics     40 00 00 c0             ; initialized, readable, writeable  
  
下一个是 0x188,由于节必须按照 32 字节对齐，所有填 0 到 0x1a0:  
    00 00 00 00 00 00       ; padding  
    00 00 00 00 00 00  
    00 00 00 00 00 00  
    00 00 00 00 00 00  
现在第一节代码节来到了，开始于 0x1a0，0x20 字节长。  
    6A 00                    ; push      0x00000000  
    68 ?? ?? ?? ??           ; push      offset \_written  
    6A 0D                    ; push      0x0000000d  
    68 ?? ?? ?? ??           ; push      offset hello\_string  
    6A F5                    ; push      0xfffffff5  
    2E FF 15 ?? ?? ?? ??     ; call      dword ptr cs:\_\_imp\_\_GetStdHandle@4  
    50                       ; push      eax  
    2E FF 15 ?? ?? ?? ??     ; call      dword ptr cs:\_\_imp\_\_WriteConsoleA@20  
    C3                       ; ret         
由于前面节的长度，不需要填 0。数据节接着出现：起始于 0x1c0:  
    68 65 6C 6C 6F 2C 20 77 6F 72 6C 64 0A  ; "hello, world\\n"  
    00 00 00                                ; padding to align \_written//为什么要对齐？？文件对齐而已。  
    00 00 00 00                             ; \_written  
  
现在剩下的是输入目录了，引入 2 个函数从"kernel32.dll",紧随 2 个变量。首先对齐到 32 个字节，文件对齐。  
    00 00 00 00 00 00 00 00 00 00 00 00     ; padding  
  
IMAGE\_IMPORT\_DESCRIPTOR 从 0x1e0 开始:只有一个 DLL 所以只有一个数组元素。  
    OriginalFirstThunk      ?? ?? ?? ??     ; yet to find out  
    TimeDateStamp           00 00 00 00     ; unbound  
    ForwarderChain          ff ff ff ff     ; no forwarders  
    Name                    ?? ?? ?? ??     ; yet to find out  
    FirstThunk              ?? ?? ?? ??     ; yet to find out  
  
我们需要结束输入目录用 0 填充，0x1f4:  
    OriginalFirstThunk      00 00 00 00     ; terminator  
    TimeDateStamp           00 00 00 00     ;  
    ForwarderChain          00 00 00 00     ;  
    Name                    00 00 00 00     ;  
    FirstThunk              00 00 00 00     ;  
  
现在剩下 DLL 名字和 2 个 thunks,及 thunk-data,函数名。DLL 名字从 0x208 开始:  
    6b 65 72 6e 65 6c 33 32 2e 64 6c 6c 00  ; "kernel32.dll"  
    00 00 00                                ; padding to 32-bit-boundary  
originalfirstthunk 数组，2 个成员,起始于 0x218:  
    AddressOfData   ?? ?? ?? ??             ; RVA to function name "WriteConsoleA"  
    AddressOfData   ?? ?? ?? ??             ; RVA to function name "GetStdHandle"  
                    00 00 00 00             ; terminator  
firstthunk 含有同样的列表，起始于 0x224:  
(\_\_imp\_\_WriteConsoleA@20, at 0x224)  
    AddressOfData   ?? ?? ?? ??             ; RVA to function name "WriteConsoleA"  
(\_\_imp\_\_GetStdHandle@4, at 0x228)  
    AddressOfData   ?? ?? ?? ??             ; RVA to function name "GetStdHandle"  
                    00 00 00 00             ; terminator  
现在只剩下 2 个函数名在 IMAGE\_IMPORT\_BY\_NAME.从 0x230.  
    01 00                                      ; ordinal, need not be correct  
    57 72 69 74 65 43 6f 6e 73 6f 6c 65 41 00  ; "WriteConsoleA"  
    02 00                                      ; ordinal, need not be correct  
    47 65 74 53 74 64 48 61 6e 64 6c 65 00     ; "GetStdHandle"  
  
下面填充到 0x260:  
    00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 ; padding  
    00  
  
\------------  
现在结束，我们知道所有字节偏移量，我们可以实施修补地址和尺寸了。  
DOS-header, starting at 0x0:  
    00 | 4d 5a 00 00 00 00 00 00 00 00 00 00 00 00 00 00  
    10 | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  
    20 | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  
    30 | 00 00 00 00 00 00 00 00 00 00 00 00 40 00 00 00  
signature, starting at 0x40:  
        50 45 00 00  
file-header, starting at 0x44:  
    Machine                     4c 01       ; i386  
    NumberOfSections            02 00       ; code and data  
    TimeDateStamp               00 00 00 00 ; who cares?  
    PointerToSymbolTable        00 00 00 00 ; unused  
    NumberOfSymbols             00 00 00 00 ; unused  
    SizeOfOptionalHeader        e0 00       ; constant  
    Characteristics             02 01       ; executable on 32-bit-machine  
  
optional header, starting at 0x58:  
    Magic                       0b 01       ; constant  
    MajorLinkerVersion          00          ; I'm version 0.0 :-)  
    MinorLinkerVersion          00          ;  
    SizeOfCode                  20 00 00 00 ; 32 bytes of code  
    SizeOfInitializedData       a0 00 00 00 ; data section size  
    SizeOfUninitializedData     00 00 00 00 ; we don't have a BSS  
    AddressOfEntryPoint         a0 01 00 00 ; beginning of code section  
    BaseOfCode                  a0 01 00 00 ; RVA to code section  
    BaseOfData                  c0 01 00 00 ; RVA to data section  
    ImageBase                   00 00 10 00 ; 1 MB, chosen arbitrarily  
    SectionAlignment            20 00 00 00 ; 32-bytes-alignment  
    FileAlignment               20 00 00 00 ; 32-bytes-alignment  
    MajorOperatingSystemVersion  04 00      ; NT 4.0  
    MinorOperatingSystemVersion  00 00      ;  
    MajorImageVersion           00 00       ; version 0.0  
    MinorImageVersion           00 00       ;  
    MajorSubsystemVersion       04 00       ; Win32 4.0  
    MinorSubsystemVersion       00 00       ;  
    Win32VersionValue           00 00 00 00 ; unused?  
    SizeOfImage                 c0 00 00 00 ; sum of all section sizes//没有包含头的尺寸，跟前面所说矛盾了。  
    SizeOfHeaders               a0 01 00 00 ; offset to 1st section  
    CheckSum                    00 00 00 00 ; not used for non-drivers  
    Subsystem                   03 00       ; Win32 console  
    DllCharacteristics          00 00       ; unused (not a DLL)  
    SizeOfStackReserve          00 00 10 00 ; 1 MB stack  
    SizeOfStackCommit           00 10 00 00 ; 4 KB to start with  
    SizeOfHeapReserve           00 00 10 00 ; 1 MB heap  
    SizeOfHeapCommit            00 10 00 00 ; 4 KB to start with  
    LoaderFlags                 00 00 00 00 ; unknown  
    NumberOfRvaAndSizes         10 00 00 00 ; constant  
  
data directories, starting at 0xb8:  
    Address        Size  
    00 00 00 00    00 00 00 00         ; IMAGE\_DIRECTORY\_ENTRY\_EXPORT (0)  
    e0 01 00 00    6f 00 00 00         ; IMAGE\_DIRECTORY\_ENTRY\_IMPORT (1)  
    00 00 00 00    00 00 00 00         ; IMAGE\_DIRECTORY\_ENTRY\_RESOURCE (2)  
    00 00 00 00    00 00 00 00         ; IMAGE\_DIRECTORY\_ENTRY\_EXCEPTION (3)  
    00 00 00 00    00 00 00 00         ; IMAGE\_DIRECTORY\_ENTRY\_SECURITY (4)  
    00 00 00 00    00 00 00 00         ; IMAGE\_DIRECTORY\_ENTRY\_BASERELOC (5)  
    00 00 00 00    00 00 00 00         ; IMAGE\_DIRECTORY\_ENTRY\_DEBUG (6)  
    00 00 00 00    00 00 00 00         ; IMAGE\_DIRECTORY\_ENTRY\_COPYRIGHT (7)  
    00 00 00 00    00 00 00 00         ; IMAGE\_DIRECTORY\_ENTRY\_GLOBALPTR (8)  
    00 00 00 00    00 00 00 00         ; IMAGE\_DIRECTORY\_ENTRY\_TLS (9)  
    00 00 00 00    00 00 00 00         ; IMAGE\_DIRECTORY\_ENTRY\_LOAD\_CONFIG (10)  
    00 00 00 00    00 00 00 00         ; IMAGE\_DIRECTORY\_ENTRY\_BOUND\_IMPORT (11)  
    00 00 00 00    00 00 00 00         ; IMAGE\_DIRECTORY\_ENTRY\_IAT (12)  
    00 00 00 00    00 00 00 00         ; 13  
    00 00 00 00    00 00 00 00         ; 14  
    00 00 00 00    00 00 00 00         ; 15  
  
section header (code), starting at 0x138:  
    Name            2e 63 6f 64 65 00 00 00     ; ".code"  
    VirtualSize         00 00 00 00             ; unused  
    VirtualAddress      a0 01 00 00             ; RVA to code section  
    SizeOfRawData       20 00 00 00             ; size of code  
    PointerToRawData    a0 01 00 00             ; file offset to code section  
    PointerToRelocations 00 00 00 00            ; unused  
    PointerToLinenumbers 00 00 00 00            ; unused  
    NumberOfRelocations  00 00                  ; unused  
    NumberOfLinenumbers  00 00                  ; unused  
    Characteristics     20 00 00 60             ; code, executable, readable  
  
section header (data), starting at 0x160:  
    Name            2e 64 61 74 61 00 00 00     ; ".data"  
    VirtualSize         00 00 00 00             ; unused  
    VirtualAddress      c0 01 00 00             ; RVA to data section  
    SizeOfRawData       a0 00 00 00             ; size of data section  
    PointerToRawData    c0 01 00 00             ; file offset to data section  
    PointerToRelocations 00 00 00 00            ; unused  
    PointerToLinenumbers 00 00 00 00            ; unused  
    NumberOfRelocations  00 00                  ; unused  
    NumberOfLinenumbers  00 00                  ; unused  
    Characteristics     40 00 00 c0             ; initialized, readable, writeable  
  
(padding)  
    00 00 00 00 00 00       ; padding  
    00 00 00 00 00 00  
    00 00 00 00 00 00  
    00 00 00 00 00 00  
  
code section, starting at 0x1a0:  
    6A 00                    ; push      0x00000000  
    68 d0 01 10 00           ; push      offset \_written  
    6A 0D                    ; push      0x0000000d  
    68 c0 01 10 00           ; push      offset hello\_string  
    6A F5                    ; push      0xfffffff5  
    2E FF 15 28 02 10 00     ; call      dword ptr cs:\_\_imp\_\_GetStdHandle@4  
    50                       ; push      eax  
    2E FF 15 24 02 10 00     ; call      dword ptr cs:\_\_imp\_\_WriteConsoleA@20  
    C3                       ; ret         
  
data section, beginning at 0x1c0:  
    68 65 6C 6C 6F 2C 20 77 6F 72 6C 64 0A  ; "hello, world\\n"  
    00 00 00                                ; padding to align \_written  
    00 00 00 00                             ; \_written  
padding:  
    00 00 00 00 00 00 00 00 00 00 00 00     ; padding  
IMAGE\_IMPORT\_DESCRIPTOR, starting at 0x1e0:  
    OriginalFirstThunk      18 02 00 00     ; RVA to orig. 1st thunk  
    TimeDateStamp           00 00 00 00     ; unbound  
    ForwarderChain          ff ff ff ff     ; no forwarders  
    Name                    08 02 00 00     ; RVA to DLL name  
    FirstThunk              24 02 00 00     ; RVA to 1st thunk  
terminator (0x1f4):  
    OriginalFirstThunk      00 00 00 00     ; terminator  
    TimeDateStamp           00 00 00 00     ;  
    ForwarderChain          00 00 00 00     ;  
    Name                    00 00 00 00     ;  
    FirstThunk              00 00 00 00     ;  
The DLL name, at 0x208:  
    6b 65 72 6e 65 6c 33 32 2e 64 6c 6c 00  ; "kernel32.dll"  
    00 00 00                                ; padding to 32-bit-boundary  
original first thunk, starting at 0x218:  
    AddressOfData   30 02 00 00             ; RVA to function name "WriteConsoleA"  
    AddressOfData   40 02 00 00             ; RVA to function name "GetStdHandle"  
                    00 00 00 00             ; terminator  
first thunk, starting at 0x224:  
    AddressOfData   30 02 00 00             ; RVA to function name "WriteConsoleA"  
    AddressOfData   40 02 00 00             ; RVA to function name "GetStdHandle"  
                    00 00 00 00             ; terminator  
IMAGE\_IMPORT\_BY\_NAME, at byte 0x230:  
    01 00                                      ; ordinal, need not be correct  
    57 72 69 74 65 43 6f 6e 73 6f 6c 65 41 00  ; "WriteConsoleA"  
IMAGE\_IMPORT\_BY\_NAME, at byte 0x240:  
    02 00                                      ; ordinal, need not be correct  
    47 65 74 53 74 64 48 61 6e 64 6c 65 00     ; "GetStdHandle"  
(padding)  
    00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 ; padding  
    00  
First unused byte: 0x260  
  
\--------------  
  
因为是 32 字节的节对齐，所以在 windows 98 下该程序不工作，可以在 NT 下工作，要想在 WIN98 下工作，必须插入许多 0 并修改 RVA。  
  
QduWg 翻译，2005 年 12 月 5 日埃塞  
后记：为了让刚刚入门的朋友对 PE 文件结构有一个比较清晰的了解，所以我翻译了此文。 

 4dd466d0e1e0cc123afbe504da2462ca