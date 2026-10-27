
# 可移植的可执行文件格式全接触（附注释）

PE文件格式系列译文之二

原著：Randy Kath (微软开发者网络技术组)
翻译：ah007（沈忠平）

【说明：本译文的所有大小标题序号都是译者添加，以方便大家阅读。圆圈内的数字是注释的编号，注释全部译自网络。另外，本系列译文之一中已有的注释这里就不再重复了。所有注释仅供参考，如有不妥之处，敬请原谅！——译者】

## 一、摘要

Windows NT 3.1版操作系统引进了一种叫做可移植的可执行(PE)文件格式的新文件格式。尽管《可移植的可执行文件格式规范》的内容相当含糊，但公众已可得到了；并且它也已被包括在我们的微软开发者网络CD（其中的：Windows NT文件格式规范->规范->规范和战略）当中。

不过，对开发者来说，仅此规范一文并不足以提供足够的信息来让他们对PE文件格式的理解变得容易，哪怕是更合理一点。本文档的目的就是用来解决这个问题的。从本文档之中，你可找到整个PE文件格式的完整的解释，还有所有必须的结构体的描述以及演示怎样使用这个信息的源代码例子。

出现在本文中的所有源代码例子都是从一个叫`PEFILE.DLL`的动态链接库中例举出来的。我写出这个DLL的目的就是为了发现包含在一个PE文件中的重要信息。这个DLL和它的源代码也被包含在这个CD当中作为PE文件例子程序的一部分；你可以自由地将这个DLL使用在你自己的应用程序之中。同样，你也可以自由地取得这些源代码并在它的基础之上为你的任何目的去构建（程序）。在本文的末尾，你会找到一个从`PEFILE.DLL`中导出的简短的函数列表，以及怎样使用它们的解释。我想，你会发现利用这些函数会让你对PE文件格式的理解要容易一些。

## 二、介绍

最近Windows操作系统的家族得到了微软®Windows NT操作系统的加入给开发环境带来了很多的变化，并且也给应用程序本身带来了不小的变化。比较重要的变化之一就是对可移植的可执行（PE）文件格式的引入。新的PE文件格式主要源自于UNIX操作系统常用的COFF(Common Object File Format，通用目标文件格式)规范。不过，为了保持对以前各版的MS-DOS®和 Windows操作系统的兼容性，PE文件格式也保留了大家过去比较熟悉的MS-DOS的MZ头。

在本文中，PE文件格式将使用从头到尾的方式来解释。文中将依照你通读文件的内容时文件的每个组成部分出现的顺序来一一讨论它们，开始时是头部并沿着你曾走过的路线全程直到结束。

许多单个的文件组成部分的定义来自于`WINNT.H`文件中，这是一个包含在Windows NT的微软Win32™软件开发工具箱(SDK)中的文件。在这个文件中，你能找到被用来表示文件中各个组成部分、每个文件头和数据目录等的结构类型定义。而其他方面，在文件中`WINNT.H`缺少对文件结构的足够的定义。在这些方面，我决定定义我自己的、能被用来访问文件数据的新结构。你将会发现这些结构被定义在`PEFILE.H`文件中，而这个文件就是用来建立`PEFILE.DLL`的。全套的`PEFILE.DLL`开发文件包含在PEFILE实例应用这一节中。

作为`PEFILE.DLL`实例代码的补充，伴随此文的还有一个单独的、名为`EXEVIEW.EXE`的基于Win32的实例程序。此实例的建立基于以下两个目的：第一，我需要一种能检验`PEFILE.DLL`功能的方法，在一些情况下这种检验要求同时能查看多个文件——也就是说多重查看的支持。第二，领会PE文件格式的许多工作都和能交互地看见数据有关。例如，要搞清输入地址名字表是怎样的结构，我得同时查看`.idata`节的节头、输入映象文件的数据目录、可选头、以及实际的`.idata`节的节身等等。`EXEVIEW.EXE`就是查看上述信息的最佳人选。

不再罗嗦，我们马上开始。

## 三、PE文件的结构

PE文件格式被组织为一个线性的数据流。开始的是MS-DOS头，然后是实模式的程序根，再就是PE文件签名，紧随其后的便是PE文件头和可选头。在这之后，出现的是所有的节头，再跟着的就是所有节的节身。文件常以一些其它方面的杂项信息，包括重定位信息、符号表信息、行数信息以及字串表数据等作为结尾。所有这些都可以通过查看图1中的图象信息更轻松地被消化吸收。

![](http://bbs.pediy.com/upload/2006/32/image/01.gif)

图1. 一个可移植的可执行文件映像的结构

我们将从MS-DOS文件头结构开始讲解。以后，PE文件结构的每个组成部分的讨论都将按照它在文件中出现的顺序来进行。这其中的很多讨论是基于那个演示如何到达文件中特定信息的实例代码来的。所有的实例代码都是从`PEFILE.C`文件，也就是`PEFILE.DLL`的源模块①中提取的。这些实例中的每一个都利用了Windows NT中最酷的特性之一——内存镜像文件②。内存镜像文件允许使用取消指向的简单指针来访问包含在文件中的数据。实例中的每个程序都使用内存镜像文件来访问PE文件中的数据。

注意：请参看本文的最后一节以讨论如何使用`PEFILE.DLL`文件。

## 四、MS-DOS（实模式）头

如上所述，PE文件的第一个组成部分是MS-DOS头。MS-DOS头不是PE文件格式新发明的。它就是那个大约从MS-DOS操作系统第二版就已有的MS-DOS头。在PE文件格式的开头完整地保留这个同样的结构的主要原因就是：以便在你试图将创建的文件载入到Windows 3.1或以前、或者是MS-DOS 2.0或以后的各版本上时，操作系统能够读取文件并明白这是不兼容的。换句话说，在你试图在MS-DOS 6.0版之上运行Windows NT的可执行文件时，你能得到：“此文件不能运行在DOS模式之下。”这样的信息。如果不将MS-DOS头包括在PE文件格式的第一部分，操作系统将只会尝试载入文件失败并给出一些完全无用的东西，比如：“认不出指定的文件名是内部还是外部命令，是可操作的程序还是批处理文件。”等等。

MS-DOS头占据PE文件的头64(0x40)个字节。反映它的内容的一个结构如下所述：

`WINNT.H`

```c
typedef struct _IMAGE_DOS_HEADER {  // DOS下的.EXE文件头
     USHORT e_magic;         // 魔数
     USHORT e_cblp;          // 文件最后一页的字节数
     USHORT e_cp;            // 文件的页数
     USHORT e_crlc;          // 重定位
     USHORT e_cparhdr;       // 段中头的大小
     USHORT e_minalloc;      // 需要的最少额外段
     USHORT e_maxalloc;      // 需要的最多额外段
     USHORT e_ss;            // 初始的(相对的)SS寄存器值
     USHORT e_sp;            // 初始的SP寄存器值
     USHORT e_csum;          // 校验和
     USHORT e_ip;            // 初始的IP寄存器值
     USHORT e_cs;            // 初始的(相对的)CS寄存器值
     USHORT e_lfarlc;        // 重定位表在文件中的地址
     USHORT e_ovno;          // 交叠数
     USHORT e_res[4];        // 保留字
     USHORT e_oemid;         // OEM识别符(用于e_oeminfo成员)
     USHORT e_oeminfo;       // OEM信息; e_oemid中指定的
     USHORT e_res2[10];      // 保留字
     LONG   e_lfanew;        // 新exe头在文件中的地址
} IMAGE_DOS_HEADER, *PIMAGE_DOS_HEADER;
```

第一个域，`e_magic`，是所谓的魔数，这个域被用来鉴定一个MS-DOS兼容文件的类型。所有MS-DOS兼容的可执行文件都把这个值设为0x54AD，代表ASCII码字符MZ。正因为这个原因MS-DOS头有时也被称为MZ头。其它的很多域对MS-DOS操作系统很重要，但对Windows NT系统，这个结构中实际上只另有一个重要的域，即最后一个域，`e_lfanew`，它是一个4字节的、PE文件头被定位到的文件中的偏移量。我们必须使用这个偏移量来定位文件中的文件头。对Windows NT系统的PE文件而言，PE文件头在和MS-DOS头之间仅相隔实模式的根程序之后很快就出现了。

## 五、实模式根程序

实模式根程序指的是当可执行文件被载入后MS-DOS实际运行的程序。对于一个实际的MS-DOS可执行映像文件而言，应用程序就从这里开始执行。对于后来的操作系统，包括Windows，OS/2®，和Windows NT等系统来说，一个MS-DOS根程序放在这里只是为了代替实际的应用程序运行的。典型的情况下，程序只输出一行文字，比如：“此程序需要微软Windows v3.1或更高的版本支持。”当然，那些创建此应用程序的人可以将他们喜欢的任何根放在这儿，也就是说你可能经常看到诸如：“你不能在OS/2系统上运行Windows NT应用程序，很明显的这不可能。”之类的东西。

在我们为Windows 3.1版构建一个应用程序时，链接器会将一个缺省的叫做`WINSTUB.EXE`的根程序链接到你的可执行文件之中。你可以通过用你自己的有效的基于MS-DOS的程序替换掉原`WINSTUB`程序并用`STUB`模块定义声明将它指定给链接器的方法来覆盖缺省的链接器行为。为Windows NT开发的应用程序在链接可执行文件时也可以通过使用`-STUB:`链接器选项来实现同样的功能。

## 六、PE文件头和签名

PE文件头可以通过MS-DOS头中的`e_lfanew`(新exe头在文件中的地址)域来索引定位。`e_lfanew`域只是提供在文件中的偏移量，因此要加上文件的内存镜像基址才能确定实际的内存镜像地址。例如：下面的宏③包含在`PEFILE.H`源文件中：

`PEFILE.H`

```c
#define NTSIGNATURE(a) ((LPVOID)((BYTE *)a +    \
                         ((PIMAGE_DOS_HEADER)a)->e_lfanew))
```

当我操纵PE文件信息时，我发现有好几个我需要的文件中的位置经常被用到。因为它们只是针对文件中的偏移量，所以用宏来实现是比较容易的，因为宏的表现要比函数的好很多。

注意：这个宏是检索PE文件签名的位置的，而不是检索PE文件头的偏移量的。从Windows和OS/2的可执行文件开始，`.EXE`文件将被给出文件签名用来指定预定的目标操作系统。对于Windows NT的PE文件格式，这个签名就在PE文件头结构的前面出现。在各个版本的Windows和OS/2系统中，签名常常在文件头的第一个word单元中。同样，对于PE文件格式，Windows NT系统也使用一个DWORD来定义签名。

不管是何种类型的可执行文件，上面展示的宏都会返回其文件签名出现的偏移量。所以根据它是不是一个Windows NT文件的不同情况，文件头要么是在DWORD的签名之后，要么就在WORD的签名之处开始。为解决这个易混淆的问题，我写出了`ImageFileType`这个函数（见下面）来返回映像文件的类型：

`PEFILE.C`

```c
DWORD  WINAPI ImageFileType (
     LPVOID    lpFile)
{
     /* DOS文件签名先出现。 */
     if (*(USHORT *)lpFile == IMAGE_DOS_SIGNATURE)
         {
         /* 从DOS头开始确定PE文件头的位置。 */
         if (LOWORD (*(DWORD *)NTSIGNATURE (lpFile)) ==
                                 IMAGE_OS2_SIGNATURE ||
             LOWORD (*(DWORD *)NTSIGNATURE (lpFile)) ==
                              IMAGE_OS2_SIGNATURE_LE)
             return (DWORD)LOWORD(*(DWORD *)NTSIGNATURE (lpFile));
         else if (*(DWORD *)NTSIGNATURE (lpFile) ==
                             IMAGE_NT_SIGNATURE)
             return IMAGE_NT_SIGNATURE;
         else
             return IMAGE_DOS_SIGNATURE;
         }
     else
         /* 未知的文件类型。 */
         return 0;
}
```

上面的代码很快地就显示出了`NTSIGNATURE`宏变得多么有用。这个宏使得比较不同文件的类型变得非常容易并为每个特定文件类型返回一个合适的类型。`WINNT.H`中定义的四种不同的文件类型分别是：

`WINNT.H`

```c
#define IMAGE_DOS_SIGNATURE             0x5A4D      // MZ
#define IMAGE_OS2_SIGNATURE             0x454E      // NE
#define IMAGE_OS2_SIGNATURE_LE          0x454C      // LE
#define IMAGE_NT_SIGNATURE              0x00004550  // PE00
```

开始时你会觉得奇怪：Windows的文件类型怎么没有出现在这个列表中呢。但是，只要你稍加研究，原因是很清楚的：Windows的可执行文件和OS/2的可执行文件之间除了操作系统的版本规格不同之外真的没有区别，这两种操作系统共享相同的可执行文件结构。

请将我们的注意力再次转回到Windows NT的PE文件格式上来，我们发现一旦我们找到了文件签名的位置，PE文件（头）就跟在此后的四个字节之后。下一个宏是用来识别PE文件头的：

`PEFILE.C`

```c
#define PEFHDROFFSET(a) ((LPVOID)((BYTE *)a +  \
     ((PIMAGE_DOS_HEADER)a)->e_lfanew + SIZE_OF_NT_SIGNATURE))
```

这个宏和前面的一个宏的唯一区别在于这个宏加入了常量`SIZE_OF_NT_SIGNATURE`。说起来悲哀，这个常量不是`WINNT.H`中所定义的，但却是我在`PEFILE.H`中为一个DWORD的大小而定义的。

因为我们已经知道PE文件头的位置，我们只需简单地将这个位置赋值给如下例所示的一个结构就能检测出这个头中的数据了：

```c
PIMAGE_FILE_HEADER   pfh;
pfh = (PIMAGE_FILE_HEADER)PEFHDROFFSET (lpFile);
```

在这个例子中，`lpFile`代表一个指向内存镜像的可执行文件的基址的指针，那里就有内存镜像文件的入口。没有文件的I/O需要执行，只需将`pfh`指针原来的对访问文件中的信息的指向取消就行了。PE文件头结构的定义如下：

`WINNT.H`

```c
typedef struct _IMAGE_FILE_HEADER {
     USHORT  Machine;                 //机器
     USHORT  NumberOfSections;        //节数
     ULONG   TimeDateStamp;           //时间日期戳
     ULONG   PointerToSymbolTable;    //符号表指针
     ULONG   NumberOfSymbols;         //符号数
     USHORT  SizeOfOptionalHeader;    //可选头的大小
     USHORT  Characteristics;         //特性
} IMAGE_FILE_HEADER, *PIMAGE_FILE_HEADER;

#define IMAGE_SIZEOF_FILE_HEADER             20         //定义一个常量
```

注意：文件头结构的大小被方便地定义在这个包含文件之中。这使得取得这个结构的大小非常简单，但我发觉对这个结构本身而言使用`sizeof`函数会更加简单，因为这样除了要记住`IMAGE_FILE_HEADER`（映像文件的文件头）这个结构名之外，我就不必再去记住`IMAGE_SIZEOF_FILE_HEADER`（映像文件的文件头大小）这个常量的名字了。另一方面，要记住所有结构的名字（对我来说）实际上已经具有挑战性了，特别是这些结构中没有一个是除了在`WINNT.H`这个包含文件中有记载外，在别的地方还有记载的。

PE文件中的信息本来就是那些让系统或者是应用程序决定怎样处理此文件的高级信息。其中的第一个域（`Machine`）是用来指出此可执行文件是设计运行在何种机器之上的，例如：DEC® Alpha, MIPS R4000, Intel® x86, 或者其他的处理器等。系统在进一步处理其余的文件数据之前可以利用这个信息来确定该怎样来处理这个文件。
