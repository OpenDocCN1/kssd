
# [原创]用16进制编辑器编写一个DLL文件

- 【文章标题】: 用16进制编辑器编写一个DLL文件
- 【文章作者】: newjueqi
- 【作者邮箱】: zengjiansheng1@126.com
- 【作者QQ号】: 190678908（sdf ）
- 【软件名称】: Hello
- 【软件大小】: 4K
- 【编写语言】: 机器码
- 【使用工具】: WinHex
- 【操作平台】: xpSp2
- 【作者声明】: 手写DLL一方面为了体会一下最早期程序员一个二进制一个二进制地写程序的感觉，另一方面加强对 PE文件的认识，失误之处敬请诸位大侠赐教!

## PE文件格式概述

PE文件格式是Windows下可执行文件的格式，熟悉PE文件将对操作系统和编译器工作原理的深刻理解。

本人之所以选择手写DLL而不是手写EXE主要原因是DLL文件比起EXE文件多了输出表和重定位表的处理，手写DLL可以对ＰＥ的理解更深刻，有关手写ＥＸＥ的可参考dncwbc大大写的文章：<http://bbs.pediy.com/showthread.php?t=48590>

本次手写DLL是输出一个函数显示一个对话框，对话框标题是“hello”，内容是“hello,pediy!!!”。由以上可知道，DLL文件涉及到的区块如下

![](https://bbs.kanxue.com/upload/attach/200810/193829_7woryhsjehjmc0j.jpg)

**表1**

本文的重点是上图所示的几个区块的构建，其它部分如果比较重要的会加说明，如果不重要的就略过，需要详细了解的请参考相关资料。

## DOS头（DOS Header）

那么，就从DOS部首开始分析，DOS部首是一个`IMAGE_DOS_HEADER`，结构如下：

```
typedef struct _IMAGE_DOS_HEADER {      // DOS .EXE header
    WORD   e_magic;                     // Magic number
    WORD   e_cblp;                      // Bytes on last page of file
    WORD   e_cp;                        // Pages in file
    WORD   e_crlc;                      // Relocations
    WORD   e_cparhdr;                   // Size of header in paragraphs
    WORD   e_minalloc;                  // Minimum extra paragraphs needed
    WORD   e_maxalloc;                  // Maximum extra paragraphs needed
    WORD   e_ss;                        // Initial (relative) SS value
    WORD   e_sp;                        // Initial SP value
    WORD   e_csum;                      // Checksum
    WORD   e_ip;                        // Initial IP value
    WORD   e_cs;                        // Initial (relative) CS value
    WORD   e_lfarlc;                    // File address of relocation table
    WORD   e_ovno;                      // Overlay number
    WORD   e_res[4];                    // Reserved words
    WORD   e_oemid;                     // OEM identifier (for e_oeminfo)
    WORD   e_oeminfo;                   // OEM information; e_oemid specific
    WORD   e_res2[10];                  // Reserved words
    LONG   e_lfanew;                    // File address of new exe header
} IMAGE_DOS_HEADER, *PIMAGE_DOS_HEADER;
```

`MS_DOS`头部我们关心的只有两个字段：`e_magic` 和 `e_lfanew`。`e_magic`需要被设置成`5A4Dh`，这个值有个宏是`IMAGE_DOS_SIGNATURE`，翻译成相应的ASCⅡ码就是“`MZ`”，是某位大牛名字的缩写（至于是哪位大牛，请翻看《加密与解密3》P266），另外一个关键的字段是`e_lfanew`，这个字段指出了真正PE文件头的偏移量，当PE加载器加载文件后，用基址加上`e_lfanew`的偏移量，就可得到PE文件的指针。

接着是DOS stud部分，在windows下用不到，预留80字节的空间。

现在来计算一下，`MS_DOS`头部共19个字段值，占64个字节，`80+64=144`转化为16进制就是`90`，所以ＰＥ文件头在整个文件里的偏移量为`90`，`e_lfanew`设置为`90 00 00 00`（ 因为Intel CPU 字符存储是低位在前，高位在后，要把`00 00 00 90`写成`90 00 00 00`）。

在DOS头部分，需要设置 `e_magic` 为`4D5A`, `e_lfanew` 为`90 00 00 00`。

完成后内容如下：

```
00000000   4D 5A 00 00 00 00 00 00  00 00 00 00 00 00 00 00   MZ..............
00000010   00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   ................
00000020   00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   ................
00000030   00 00 00 00 00 00 00 00  00 00 00 00 90 00 00 00   ............?..
00000040   00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   ................
00000050   00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   ................
00000060   00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   ................
00000070   00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   ................
00000080   00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   ................
```

## PE文件头（`IMAGE_NT_HEADERS`）

紧接着DOS文件头是PE文件头（PE Header ），是PE相关结构NT映像头（`IMAGE_NT_HEADERS`）的简称，内容如下:

```
typedef struct _IMAGE_NT_HEADERS {
    DWORD Signature;
    IMAGE_FILE_HEADER FileHeader;
    IMAGE_OPTIONAL_HEADER32 OptionalHeader;
} IMAGE_NT_HEADERS32, *PIMAGE_NT_HEADERS32;
```

`IMAGE_NT_HEADERS`结构中第一个字段`Signature`被设置成`00004550h`，转化成ASCⅡ码就是“`PE00`”，在`IMAGE_NT_SIGNATURE`中定义，这个字段标志着ＰＥ文件头的开始。

## `IMAGE_FILE_HEADER`（映像文件头）

`IMAGE_FILE_HEADER`（映像文件头）中包含了PE文件的一些基本信息，结构如下：

```
typedef struct _IMAGE_FILE_HEADER {
    WORD    Machine;
    WORD    NumberOfSections;
    DWORD   TimeDateStamp;
    DWORD   PointerToSymbolTable;
    DWORD   NumberOfSymbols;
    WORD    SizeOfOptionalHeader;
    WORD    Characteristics;
} IMAGE_FILE_HEADER, *PIMAGE_FILE_HEADER;
```

1. `Machine`：两个字节，可执行文件的目标CPU类型，几种常见的CPU类型如下表：

   ![](https://bbs.kanxue.com/upload/attach/200810/193829_cjau691ohee9u9d.gif)

   **表2**

   对于我们常用的Intel平台，我们选择设置为`014Ch`，在16进制编辑器里输入“`4C 01`”

2. `NumberOfSections`：两个字节，区块的数目，紧接着`IMAGE_NT_HEADERS`后面，从表1可看到我们选择创建了5个区块，所以应该设置成“`00 05`”， 在16进制编辑器里输入“`05 00`”。

3. `TimeDateStamp`：四个字节，文件的创建的时间，设置为“`00 00 00 00`”， 在16进制编辑器里输入“`00 00 00 00`”。

4. `PointerToSymbolTable`：四个字节，COFF符号表的文件偏移位置，设置为“`00 00 00 00`”， 在16进制编辑器里输入“`00 00 00 00`”。

5. `NumberOfSymbols`：四个字节，如果有COFF符号表，就表示符号表的数目，设置为“`00 00 00 00`”， 在16进制编辑器里输入“`00 00 00 00`”。

6. `SizeOfOptionalHeader`：两个字节，紧接着`IMAGE_FILE_HEADER`后面的`IMAGE_OPTIONAL_HEADER32`的数据大小，通常对于32位系统，通常设置为`00 E0h`，对于64位的PE32+文件，通常是`00 F0h`，设置为“`00 E0`”， 在16进制编辑器里输入“`E0 00`”。

7. `Characteristics`：两个字节，表示文件的属性，各个位的含有如下：

   - Bit 0：没有重定位信息的话就置1
   - Bit 1：是可执行文件就置1，表示不是一个目标文件或库文件。这里说明一下，所谓的可执行文件不单单指EXE文件，更包括了编译成功的文件。
   - Bit 2：行号信息被移去就置1。
   - Bit 3：符号信息被移去就置1。
   - Bit 4：应用程序可处理超过2G的地址时就置1。
   - Bit 7：处理器的低字节是相反的就置1。
   - Bit 8：目标平台是32位机器就置1。
   - Bit 9：如果没有调试信息在内就置1。
   - Bit 10：如果程序不能从移动磁盘或多媒体光盘执行就置1。
   - Bit 11：如果程序不能够从网络中运行就置1。
   - Bit 12：是系统文件就置1。
   - Bit 13：是DLL文件就置1。
   - Bit 14：如果文件只能在单处理器上运行就置1。
   - Bit 15：处理器的高字节是相反的就置1。

在这里，本人设置属性值为“`A1 8E`”, 分别置Bit 1，Bit 2，Bit 3，Bit 7，Bit 8，Bit 13，Bit 15为1，在16进制编辑器里输入“`8E A1`”。

经网友cnhnyu指点，本人设置`Characteristics`的属性值是有问题，PECOFF里建议把第7和第15位设置为0，特此提出来

好，现在的PE文件头部分的`Signature` 和`IMAGE_FILE_HEADER`部分的16进制源码如下：

```
00000090   50 45 00 00 4C 01 05 00  00 00 00 00 00 00 00 00   PE..L...........
000000A0   00 00 00 00 E0 00 8E A1                            ....?帯
```

## `IMAGE_OPTIONAL_HEADER32`（可选映像头）

紧接着`IMAGE_FILE_HEADER`就是`IMAGE_OPTIONAL_HEADER32`结构，注意，虽然名字是可选，但实际上是必须的。这个结构的定义如下：

```
typedef struct _IMAGE_OPTIONAL_HEADER {
    //
    // Standard fields.
    //
    WORD    Magic;
    BYTE    MajorLinkerVersion;
    BYTE    MinorLinkerVersion;
    DWORD   SizeOfCode;
    DWORD   SizeOfInitializedData;
    DWORD   SizeOfUninitializedData;
    DWORD   AddressOfEntryPoint;
    DWORD   BaseOfCode;
    DWORD   BaseOfData;
    //
    // NT additional fields.
    //
    DWORD   ImageBase;
    DWORD   SectionAlignment;
    DWORD   FileAlignment;
    WORD    MajorOperatingSystemVersion;
    WORD    MinorOperatingSystemVersion;
    WORD    MajorImageVersion;
    WORD    MinorImageVersion;
    WORD    MajorSubsystemVersion;
    WORD    MinorSubsystemVersion;
    DWORD   Win32VersionValue;
    DWORD   SizeOfImage;
    DWORD   SizeOfHeaders;
    DWORD   CheckSum;
    WORD    Subsystem;
    WORD    DllCharacteristics;
    DWORD   SizeOfStackReserve;
    DWORD   SizeOfStackCommit;
    DWORD   SizeOfHeapReserve;
    DWORD   SizeOfHeapCommit;
    DWORD   LoaderFlags;
    DWORD   NumberOfRvaAndSizes;
    IMAGE_DATA_DIRECTORY DataDirectory[IMAGE_NUMBEROF_DIRECTORY_ENTRIES];
} IMAGE_OPTIONAL_HEADER32, *PIMAGE_OPTIONAL_HEADER32;
```

一共是31个属性值，由此可见，这个结构的内容是多么的丰富！

1. `Magic`：两个字节，是一个标志字，如果是ROM文件（`0107h`），可执行文件（`010Bh`），或者是PE32+文件（`020Bh`）。设置为“`01 0B`”， 在16进制编辑器里输入“`0B 01`”。

2. `MajorLinkerVersion`：一个字节，链接程序的主版本号。不影响程序的执行，设置为“`00`”， 在16进制编辑器里输入“`00`”。

3. `MinorLinkerVersion`：一个字节，链接程序的主版本号。不影响程序的执行，设置为“`00`”， 在16进制编辑器里输入“`00`”。

4. `SizeOfCode`：四个字节，可执行代码的长度，不影响程序的执行，设置为“`00 00 00 00`”， 在16进制编辑器里输入“`00 00 00 00`”。

5. `SizeOfInitializedData`：四个字节，初始化数据的大小，不影响程序的执行，设置为“`00 00 00 00`”， 在16进制编辑器里输入“`00 00 00 00`”。

6. `SizeOfUnInitializedData`：四个字节，未初始化数据的大小，不影响程序的执行，设置为“`00 00 00 00`”， 在16进制编辑器里输入“`00 00 00 00`”。

7. `AddressOfEntryPoint`：四个字节，程序执行入口的RVA。对于DLL文件来说，这个入口点是在进程初始化和关闭时以及进程创建/违灭时被调用。在DLL中这个域能被设置为`0`，前面提到的通知消息都不接收。我们知道一般的PE文件都有一个代码区块，里面存放的是所有的可执行代码，这里程序执行入口也就是代码区块的首地址RVA，本人设置为`1000`，这里不知道这个值怎么来没关系，在讲到区块表时就会了解。设置为“`00 00 10 00`”， 在16进制编辑器里输入“`00 10 00 00`”。

8. `BaseOfCode`：四个字节，代码段起始RVA。这个值在这里和`AddressOfEntryPoint`一样，设置为“`00 00 10 00`”， 在16进制编辑器里输入“`00 10 00 00`”。

9. `BaseOfData`：四个字节，数据段的起始RVA。在区块表会有详细说明，现在设置为“`00 00 20 00`”， 在16进制编辑器里输入“`00 20 00 00`”。
