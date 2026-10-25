
# PE 格式详细讲解（一）

> 原文：[`www.pediy.com/kssd/pediy12/98202.html`](https://www.pediy.com/kssd/pediy12/98202.html)

前几天发了一个 PE 信息查看器的小工具，本来想用那个获取邀请码的，可是觉得几率不是太大，于是再献上一篇教程，既是为了自己能获得邀请码，也是帮助那些想学习 PE 格式的人，让知识来源于网络再回归网络。

N 年没写文章了，不知道句子还能不能写通顺，最近正在看《软件加密技术内幕》，刚看完 PE 结构那部分内容，所以想起来写篇教程作为读书笔记，既可加强记忆又可帮助别人，何乐而不为呢。

好了，废话少说好戏正式上场，PE 是英文 Portable Executable（可移植的执行体）的缩写，从缩写可以看出它是跨平台的，即使在非 intel 的 CPU 上也能正常运行的。它是 Win32 环境自身所带的执行体文件格式。其实不光是 EXE 文件是 PE 格式，其它的一些重要文件，例如动态链接库文件(DLL),驱动文件(SYS)等也是 PE 格式的，所以学好 PE 格式是非常重要的，以下我把这类文件统称为 PE 文件。学习 PE 文件结构不仅可以使我们知道可执行文件是怎样运行的，也可以使我们了解一下 windows 操作系统的一些工作机制，精通 PE 是成为计算机高手的必经之路。

其实说白了 PE 文件格式就是一种文件组织的方式，里面对一些重要信息的存放做了一些规定，比如文件要运行，我们就得先知道入口地址，可是我们从哪去得到入口地址呢，我们必须把保存有入口地址信息的有关数据放在固定的位置，这样不管是哪个文件我们就能从那个固定位置取得入口地址，而这固定位置就是 PE 标准所规定的。现在我们就来正式学习这套标准，我觉得从整体到部分是个很好的学习方法，先从整体把握全局，然后再重点各个击破，逐渐深入，下面我就以这个思路来介绍 PE 文件格式，方便大家快速掌握。它总体上由五大部分组成：

1. DOS MZ header(DOS 头)
2. DOS stub
3. PE header(PE 头)
4. section table(节表)
5. section(各个节)

所有的 PE 文件必须以一个 DOS MZ header 开始，其实它是一个 `IMAGE_DOS_HEADER` 类型的结构，这个结构的定义我们可以在 `WINNT.H` 头文件找到，此结构中有两个重要的成员是我们必须知道的，第一个 `e_magic`，它是一个 `DWORD` 类型的变量，这个变量只有一个用处，就是当我们要判断一个文件是不是 PE 文件时，我们首先需要把这个变量的值与 `IMAGE_DOS_SIGNATURE` 比较，相等就完成了判断的第一步，当然后面我们还得判断一个标志，不等的话就说明当前文件不是 PE 文件。第二个成员就是非常重要的了，可以说只要它一出错我想这个文件就坏了，它就是 `e_lfanew`，这是一个 `LONG` 类型的变量，里面存放了 PE 头在这个文件中的偏移量，它是定位 PE 头的关键数据，知道这两个成员的意义后 DOS 头学习基本就完成了。接下来的 DOS stub 实际上是个 EXE,当当前系统不支持 PE 文件结构时它能输出一个错误提示“This program requires Windows”，不是很重要。接下来是重头戏 PE 头，PE 头是一个 `IMAGE_NT_HEADERS` 类型的结构，下面是这个结构在 `WINNT.H` 中的定义：

```c
typedef struct _IMAGE_NT_HEADERS {
    DWORD Signature;
    IMAGE_FILE_HEADER FileHeader;
    IMAGE_OPTIONAL_HEADER32 OptionalHeader;
} IMAGE_NT_HEADERS32, *PIMAGE_NT_HEADERS32;
```

它有三个数据成员，`Signature` 是一个标志变量，这个就是当我们判断一个文件是否是 PE 文件时第二步需要判断的，若这个值等于`"PE\0\0"`时就是一个 PE 文件，当然我们也可以像上面一样直接与 `IMAGE_NT_SIGNATURE` 这个常量比较，其实是等效的，因为 `IMAGE_NT_SIGNATURE` 是一个宏定义，它的实际值就是`"PE\0\0"`。第二个成员是一个 `IMAGE_FILE_HEADER` 类型的对象，通常我们叫它文件头，这个结构在头文件中的定义是：

```c
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

可以看出它有 7 个成员，下面我就对其中比较重要的成员做一说明，`NumberOfSections` 这个成员保存了节的数目，至于节是什么东东，后面再说，在这只要知道它的数目保存在文件头的 `NumberOfSections` 中就可以了，当我们要遍历节时需要访问这个对象。`PointerToSymbolTable`，`NumberOfSymbols` 这两个成员在调试时用的着，这里不做过多说明。`SizeOfOptionalHeader` 这个成员保存了 PE 头中 `OptionalHeader` 这个成员的大小，最后一个成员是一个关于文件的标记，即这个文件是 EXE 还是 DLL 文件。总的来说文件头中包含了 PE 文件的物理分布的有关信息，比较重要的就是第二个成员了。接下来我们来学习 PE 头的第三个成员 `OptionalHeader`，这是 PE 文件结构中最重要的一个部分，因为大部分重要的数据结构都得通过它去定位，可以说它保存了 PE 文件逻辑分布的信息，但是这部分也是学习 PE 结构过程中的一个难点，因为层次结构比较多，我们定位一个结构需要抽丝剥茧般一层层进行下去，在此过程还得保持头脑清醒，对变量类型还得有比较深层次的理解。好了，我们还是那个原则，开始不要先涉入过深，先对这个成员有个总体的印象，它是一个 `IMAGE_OPTIONAL_HEADER32` 类型的对象，这个结构的定义是：

```c
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

从这个结构的长度就可看出，它保存的信息不少，肯定很重要，在正式学习这个结构的成员前，我们还要理解一个重要的术语--RVA，RVA 代表相对虚拟地址。它是一个 `DWORD` 类型的数据，那 RVA 到底是什么呢?简单的说它是一个偏移量，这个偏移量是虚拟空间中相对于参考点的偏移，它的值就是偏移的大小，值得一说的是这个值只有当 PE 文件被 PE 文件装载器载入内存时才有作用，否则的话我们不能用这个值去直接定位与它相关的数据结构。如果在 PE 文件没有被 PE 装载器载入内存，而我们又想在 PE 文件中直接定位相关数据该怎么办呢，我们需要把这个值转换成文件偏移量，也就是相对于文件开头的偏移量，我们可以用 `RVA` 转换成 `Offset` 的函数（通常称为 `RVAToOffset`）去完成地址的相关转换，至于这个函数的实现可以从网上找到。好了，准备工作做好了，下面我们就进入正题吧。由于这个结构成员较多，有些也不是很常用，还有有些我也不是很明白，所以就不说了，下面我就对一些比较常用的一一进行介绍吧。还希望大家不要像我一样一知半解的，要有宁可错杀一千，不可放过一个的学习精神。

`AddressOfEntryPoint`：
PE 装载器准备运行的 PE 文件的第一个指令的 RVA。若您要改变整个执行的流程，可以将该值指定到新的 RVA，这样新 RVA 处的指令首先被执行。

`ImageBase`
PE 文件的优先装载地址。比如，如果该值是 `400000h`，PE 装载器将尝试把文件装到虚拟地址空间的 `400000h` 处。字眼"优先"表示若该地址区域已被其他模块占用，那 PE 装载器会选用其他空闲地址。

`SectionAlignment`：
内存中节对齐的粒度。例如，如果该值是 `4096` (`1000h`)，那么每节的起始地址必须是 `4096` 的倍数。若第一节从 `401000h` 开始且大小是 `10` 个字节，则下一节必定从 `402000h` 开始，即使 `401000h` 和 `402000h` 之间还有很多空间没被使用。

`FileAlignment`：
文件中节对齐的粒度。例如，如果该值是(`200h`),，那么每节的起始地址必须是 `512` 的倍数。若第一节从文件偏移量 `200h` 开始且大小是 `10` 个字节，则下一节必定位于偏移量 `400h`: 即使偏移量 `512` 和 `1024` 之间还有很多空间没被使用/定义。

`MajorSubsystemVersion`
`MinorSubsystemVersion` ：
win32 子系统版本。若 PE 文件是专门为 Win32 设计的，该子系统版本必定是 `4.0` 否则对话框不会有 3 维立体感。

`SizeOfImage`：
内存中整个 PE 映像体的尺寸。它是所有头和节经过节对齐处理后的大小。

`SizeOfHeaders`：
所有头+节表的大小，也就等于文件尺寸减去文件中所有节的尺寸。可以以此值作为 PE 文件第一节的文件偏移量。

`Subsystem`：
NT 用来识别 PE 文件属于哪个子系统。 对于大多数 Win32 程序，只有两类值: Windows GUI 和 Windows CUI (控制台)。

`DataDirectory`：
一 `IMAGE_DATA_DIRECTORY` 结构数组。每个结构给出一个重要数据结构的 RVA，比如引入地址表等。 可以告诉大家这个成员是 PE 结构的重中之重了，在这先给大家提个醒，让大家多看它两眼，加深对它的印象。

好了，到此为止，这个 PE 文件结构在总体划分上就只剩下节表和节没有介绍了，现在我就来说说节表和节吧，在正式介绍它之前，我先说一点它话，当然绝对是有助于你理解节表的，书大家都看过是吧，现在你想一想我们拿到一本新书时，做的第一件事是什么呢？首先我们翻开的是书的目录，然后从书的目录中去搜寻我们感兴趣的东西。可以打个形象的比喻，节表就相当于书的目录，而书中的各个章节就相当于 PE 文件结构中的节，通过目录我们能很快找到书中我们感兴趣的内容，同样通过节表我们很快能找到 PE 文件中的各个节。好了，有了定性的认识后我们还得继续升华，还得定量的学习它，这才是科学的学习方法。节表从数据结构的角度来说它是一个结构数组，所谓结构数组就是这个数组的每个成员都是同一种结构类型的变量。它们不光在逻辑上是连续的，在存储介质中也是连续的。同样我们也可在 `WINNT.H` 中找到这个结构的定义：

```c
typedef struct _IMAGE_SECTION_HEADER {
    BYTE    Name[IMAGE_SIZEOF_SHORT_NAME];
    union {
            DWORD   PhysicalAddress;
            DWORD   VirtualSize;
    } Misc;
    DWORD   VirtualAddress;
    DWORD   SizeOfRawData;
    DWORD   PointerToRawData;
    DWORD   PointerToRelocations;
    DWORD   PointerToLinenumbers;
    WORD    NumberOfRelocations;
    WORD    NumberOfLinenumbers;
    DWORD   Characteristics;
} IMAGE_SECTION_HEADER, *PIMAGE_SECTION_HEADER;
```

这个结构总共包含了 10 个成员变量，由于有些并不是很重要，所以我就对其中比较重要的做个介绍，如果你是那种追根究底的人，你自己完全可以继续查阅相关资料，了解一下其它成员的实际意义，不过要在你对我介绍的都掌握之后，学习把握主次是很重的，舍本逐末只会捡了芝麻丢了西瓜。好了，现在开始介绍吧。

`Name[IMAGE_SIZEOF_SHORT_NAME]`：
这个成员是一个字节型的数组，很明显它保存的是一个名字，我想你能猜到了吧，这就是节的名字，不过要注意一下，这个数组的上限是 `8`，最多只能保存 `8` 个字符，还有就是它不是一个 ASCIIZ 字符串，因为它不是以 null 结尾的。

`VirtualAddress`：
本节的 RVA（相对虚拟地址）。PE 装载器将节映射至内存时会读取本值，因此如果域值是 `1000h`，而 PE 文件装在地址 `400000h` 处，那么本节就被载到 `401000h`。

这个很重要，定位节时需要它。

`SizeOfRawData`：
经过文件对齐处理后节尺寸，PE 装载器提取本域值了解需映射入内存的节字节数。（译者注: 假设一个文件的文件对齐尺寸是 `0x200`，如果前面的 `VirtualSize` 域指示本节长度是 `0x388` 字节，则本域值为 `0x400`，表示本节是 `0x400` 字节长）。

`PointerToRawData`：
这是节基于文件的偏移量，PE 装载器通过本域值找到节数据在文件中的位置。

`Characteristics`：
包含标记以指示节属性，比如节是否含有可执行代码、初始化数据、未初始数据，是否可写、可读等。

基本上比较重要的成员都介绍了，我想你看了这么久肯定有点迷糊了，介绍了这么久的节表，那到底什么是节呢？还专门为它建立一个目录好像很重要似的。确实，可以说节才是 PE 内容的真正载体， 节其实我们可以想象成块，这样更形象点，因为一节就是一块数据，而且这块数据拥有共同的属性，比如是代码还是数据，是只读的还是读写的。学习节要牢牢记住几个字---共同的属性。就是说多个数据只要是具有共同属性我们就能把它放在同一节中，而不用去考虑这些数据是不是在逻辑上有什么关联。真正以数据之间逻辑关联建立的目录是 PE 头的第三个成员 `OptionalHeader`，我们已经稍微了解了下了。



好了，对 PE 文件结构的总体构造已经介绍完了，但学习 PE 文件结构的任务还远远没有完成，因为还有导入表，象 DLL 还有导出表，还有到底怎么去定位 PE 中的重要结构等都没有介绍。我只能对你说，不要着急，一口吃不成胖子，学习决非一日之寒，尤其是计算机的学习，它是一个系统的认知过程，知识有很强的连贯性。所以对于决心学好的计算机的人，我有一句良言相赠----找准目标，持之以恒，绝不放弃。上面介绍的内容我参考了《软件加密技术内幕》这本书，由于我也是第一次真正接触 PE，所以有些概念理解的不是很准确，希望大家多多包含，有时间大家可以看看原版。当然这只是这个 PE 教程的第一部分，如果大家不拿鸡蛋砸我的话，我还会写出后续部分。我的知识来自网络，所以我也要回馈网络，虽然现在水平还不是很高，但尽力了就行了。

附上 PE 信息查看器的下载地址，可以用这个小工具辅助学习，效果更好。

[`bbs.pediy.com/showthread.php?t=97970`](http://bbs.pediy.com/showthread.php?t=97970)

最近也研究 PE 文件，找到这个图片，最爱啊，

上传的附件

| ![](img/aabdec8de01ec04d2bc68959c61abb93.jpg) || `PE Format.pdf` |
| :--- | :--- |
