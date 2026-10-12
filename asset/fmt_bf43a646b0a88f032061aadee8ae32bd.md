
`#define IMAGE_DIRECTORY_ENTRY_EXPORT` `0`

// 输入目录

`#define IMAGE_DIRECTORY_ENTRY_IMPORT` `1`

// 资源目录

`#define IMAGE_DIRECTORY_ENTRY_RESOURCE` `2`

// 异常目录

`#define IMAGE_DIRECTORY_ENTRY_EXCEPTION` `3`

// 安全目录

`#define IMAGE_DIRECTORY_ENTRY_SECURITY` `4`

// 基址重定位表

`#define IMAGE_DIRECTORY_ENTRY_BASERELOC` `5`

// 调试目录

`#define IMAGE_DIRECTORY_ENTRY_DEBUG` `6`

// 描述字符串

`#define IMAGE_DIRECTORY_ENTRY_COPYRIGHT` `7`

// 机器值(MIPS GP)

`#define IMAGE_DIRECTORY_ENTRY_GLOBALPTR` `8`

// TLS（线程本地存储）⑥目录

`#define IMAGE_DIRECTORY_ENTRY_TLS` `9`

// 载入配置目录

`#define IMAGE_DIRECTORY_ENTRY_LOAD_CONFIG` `10`

每个数据目录基本上都是一个被定义为`IMAGE_DATA_DIRECTORY`（数据目录）的结构体。并且尽管各个数据目录项本身都是相同的，但每一个特定的目录类型都是完全唯一的。每个已定义的数据目录的定义将在本文后面的“预定义的节”这一节中讲解。

`WINNT.H`

```
typedef struct _IMAGE_DATA_DIRECTORY {
    ULONG   VirtualAddress;       //虚拟地址
    ULONG   Size;                 //大小
} IMAGE_DATA_DIRECTORY, *PIMAGE_DATA_DIRECTORY;
```

每个数据目录项都指定了相应目录的大小和相对虚拟地址。要定位一个特定的目录，你可先通过可选头中的数据目录数组来确定它的相对地址。然后，利用这个相对地址来确定这个目录位于哪个节中。一旦你确定了在哪个节包含了这个目录，那个节的节头就被用来寻找这个数据目录的准确的文件偏移量的位置。

因此，要找到一个数据目录，首先你需要了解节的知识，这就是我们下一步将要讲解的内容。一个介绍如何定位数据目录的例子将附在这个讨论内容的后面。

## 八、PE 文件的节

PE 文件规范由本文前面已定义的各种头和一个叫做节的一类东西所组成。节包含着文件中的内容，包括：代码、数据、资源和其他可执行的信息。每个节都有一个头和一个身（原始数据）。节头将在下面讲述，但是节身却没有一个固定的文件结构。只要节头中填有能够解读数据的足够的信息，节身几乎可以被链接器组织为任何它所想组织的形式。

### （一）、节头

在 PE 文件格式中，各个节头紧接在可选头之后依次地排列。每个节头都是 40 字节，并且相互之间没有任何填充。节头被定义为下面的这种结构形式：

`WINNT.H`

```
#define IMAGE_SIZEOF_SHORT_NAME           8       //定义一个常量
typedef struct _IMAGE_SECTION_HEADER {
    UCHAR   Name[IMAGE_SIZEOF_SHORT_NAME];         //名字数组
    union {                                       //共用体标志
        ULONG   PhysicalAddress;                   //物理地址
        ULONG   VirtualSize;                       //虚拟大小
    } Misc;
    ULONG   VirtualAddress;                        //虚拟地址
    ULONG   SizeOfRawData;                         //原始数据的大小
    ULONG   PointerToRawData;                      //原始数据指针
    ULONG   PointerToRelocations;                  //重定位指针
    ULONG   PointerToLinenumbers;                  //行数指针
    USHORT  NumberOfRelocations;                   //重定位数目
    USHORT  NumberOfLinenumbers;                   //行数数目
    ULONG   Characteristics;                       //特征
} IMAGE_SECTION_HEADER, *PIMAGE_SECTION_HEADER;
```

你该怎样来找到一个特定节的节头信息呢？因为节头没有按照一种特定的顺序被依次地组织起来，所以节头就必须用名字来定位。下面的函数将显示在给定节的名字时如何从一个 PE 映像文件中检索到它的节头：

`PEFILE.C`

```
BOOL    WINAPI GetSectionHdrByName (
    LPVOID                  lpFile,
    IMAGE_SECTION_HEADER    *sh,
    char                    *szSection)
{
    PIMAGE_SECTION_HEADER   psh;
    int                     nSections = NumOfSections (lpFile);
    int                     i;
    if ((psh = (PIMAGE_SECTION_HEADER)SECHDROFFSET (lpFile)) !=
        NULL)
        {
        /* 按名字寻找节 */
        for (i=0; i<nSections; i++)
            {
            if (!strcmp (psh->Name, szSection))
                {
                /* copy data to header */
                CopyMemory ((LPVOID)sh,
                            (LPVOID)psh,
                            sizeof (IMAGE_SECTION_HEADER));
                return TRUE;
                }
            else
                psh++;
            }
        }
    return FALSE;
}
```

此函数先是简单地通过`SECHDROFFSET`宏来定位第一节的节头。然后，函数就在每个节中循环，用它所寻找的节的名字来和每个节的名字一一比较，直到它找到了正确的名字。当找到那个节后，函数将从内存镜像文件中复制所需的数据到函数所传进来的结构体中。此时`IMAGE_SECTION_HEADER`（节头）结构体中的各个域就可以从这个结构体中直接访问了。

### （二）、节头的域

*   `Name`（名字）。每个节头都有一个最长可达 8 个字符的名字域，其第一个字符必须为句点。（这里的描述不准确----译者）
*   `PhysicalAddress`（物理地址）或 `VirtualSize`（虚拟大小）。第二个域是一个共用体域，目前未使用。
*   `VirtualAddress`（虚拟地址）。此域标示的是进程中的、节被载入到的地址空间中的虚拟地址。通过使用此域中的值并加上可选头结构中的映像文件基址（`ImageBase`）域中的虚拟地址就可以得到实际的地址。不过请记住：如果这个映像文件代表的是一个 DLL 文件，那么就无法保证 DLL 文件能被载入到所要求的映像文件基址（`ImageBase`）位置。因此，一旦此文件被载入到一个进程当中时，实际的映像文件基址（`ImageBase`）值应该使用 `GetModuleHandle`（取得模块句柄）函数逐个程序地纠正。
*   `SizeOfRawData`（原始数据大小）。此域指出节身的、和文件对齐（`FileAlignment`）相关的大小。节身的实际大小将小于或等于文件中的文件对齐（`FileAlignment`）的某个倍数。映像文件一旦被载入到一个进程的地址空间之中时，节身的大小就会小于或等于节对齐（`SectionAlignment`）的某个倍数。
*   `PointerToRawData`（原始数据指针）。这是一个位于文件中的节身的位置的偏移量。
*   `PointerToRelocations`（重定位指针），`PointerToLinenumbers`（行数指针），`NumberOfRelocations`（重定位数），`NumberOfLinenumbers`（行数数）。在 PE 文件格式中，这些域都没有使用。
*   `Characteristics`（特征）。 定义节的特征。这些值在 `WINNT.H` 文件和本 CD 的可移植的可执行格式规范一节中均可找到。

    | 值 | 定义 |
    | :--- | :--- |
    | `0x00000020` | 代码节 |
    | `0x00000040` | 已初始化数据节 |
    | `0x00000080` | 未初始化数据节 |
    | `0x04000000` | 不可缓存节 |
    | `0x08000000` | 不可分页节 |
    | `0x10000000` | 共享节 |
    | `0x20000000` | 可执行节 |
    | `0x40000000` | 可读节 |
    | `0x80000000` | 可写节 |

### （三）、定位数据目录

数据目录存在于它们相应节的节身之中。典型地，数据目录是节身当中的第一个结构体，但这并不是出于必须的原因。也正因为这个原因，你得同时从节头和可选头二者之中检索信息来定位一个特定的数据目录。

为使这个过程容易一点，特写出下列的函数来定位 `WINNT.H` 中定义的任意一个数据目录：

`PEFILE.C`

```
LPVOID  WINAPI ImageDirectoryOffset (
        LPVOID    lpFile,
        DWORD     dwIMAGE_DIRECTORY)
{
    PIMAGE_OPTIONAL_HEADER    poh;
    PIMAGE_SECTION_HEADER     psh;
    int                       nSections = NumOfSections (lpFile);
    int                       i = 0;
    LPVOID                    VAImageDir;
    /* 一直到(NumberOfRvaAndSizes-1)都必须为 0。 */
    if (dwIMAGE_DIRECTORY >= poh->NumberOfRvaAndSizes)
        return NULL;
    /* 检索出节头和可选头的偏移量。 */
    poh = (PIMAGE_OPTIONAL_HEADER)OPTHDROFFSET (lpFile);
    psh = (PIMAGE_SECTION_HEADER)SECHDROFFSET (lpFile);
    /* 定位映像文件目录的相对虚拟地址。 */
    VAImageDir = (LPVOID)poh->DataDirectory
                            [dwIMAGE_DIRECTORY].VirtualAddress;
    /* 定位包含映像文件目录的节。 */
    while (i++<nSections)
        {
        if (psh->VirtualAddress <= (DWORD)VAImageDir &&
            psh->VirtualAddress +
              psh->SizeOfRawData > (DWORD)VAImageDir)
            break;
        psh++;
        }
    if (i > nSections)
        return NULL;
    /* 返回映像文件输入目录的偏移量。 */
    return (LPVOID)(((int)lpFile +
                    (int)VAImageDir. psh->VirtualAddress) +
                    (int)psh->PointerToRawData);
}
```

此函数开始先验证所求数据目录项的数字。然后检索可选头和第一个节头的指针。从可选头开始算起，函数确定数据目录的虚拟地址，接着它利用这个值来确定数据目录位于哪一个节身之中。一旦合适的节身被识别出来之后，就可通过转换数据目录与文件中的一个特定地址的相对虚拟地址来找到数据目录的特定位置。

### （四）、预定义的节

一个 Windows NT 的应用程序典型情况下会有九个预定义的节，它们的名字分别为：`.text`、`.bss`、`.rdata`、`.data`、`.rsrc`、`.edata`、`.idata`、`.pdata` 以及 `.debug` 等。有些应用程序并不需要所有这么多的节，而另一些则会还需定义更多的节以适合它们的特殊需要。这种行为与 MS-DOS 以及 Windows 3.1 版下的代码段和数据段是很相似的。事实上，一个应用程序定义一个独一无二的节的方法就是利用标准编译器的用来给代码段和数据段命名的指令，或者是利用名字段编译器的-NT 选项----而这正是 Windows 3.1 版下应用程序定义独一无二的代码段和数据段名字时所使用的同样的方法。

下面是在典型的 Windows NT PE 文件常见的、一些比较有意义的节的讨论。

#### 1.可执行代码节：.text

Windows 3.1 版和 Windows NT 之间的一个区别就在于：在 Windows NT 中默认的行为就是将所有的代码段（在 Windows 3.1 版中它们就是这样被引用的）合并为一个单独的、名为`.text`的节。因为 Windows NT 使用一种基于页的虚拟内存管理系统，所以将代码分开为几个截然不同的代码段就没有任何优势了。因此，使用一个大的代码段对操作系统和应用程序开发者俩者来说都更易于管理。

`.text` 节也含有先前提到的入口点。IAT 也存在于`.text` 节中、模块入口点正前面。（将 IAT 放在`.text` 节中是明智的，因为这个表实际上就是一系列的跳转指令，跳往的特定位置就是已改正的地址。）当 Windows NT 的可执行映像文件被载入到一个进程的地址空间后，IAT 就被用每个输入函数的物理地址的位置改正过来。为了找到`.text` 节中的 IAT，加载器只是定位一下模块的入口点，并完全的依赖 IAT 就在入口点前面出现这个事实。又因为每个入口都是同样的大小，所以在这个表中往回走来找到它的开始处就容易了。

#### 2.数据节：.bss, .rdata, .data

`.bss` 节代表的是应用程序的未初始化的数据，包括一个函数或者源模块中所有声明为静态的变量。

`.rdata` 节代表的是只读数据，比如：字符串、常量以及调试目录信息等。

所有其他的变量（除了自动变量，它只出现在栈中）都存储在`.data` 节中。这些基本上都是应用程序和模块的全局变量。

#### 3.资源节：.rsrc

`.rsrc` 节包含一个模块的资源信息。像大多数其他节一样它也以一个资源目录结构开始，但是这个节的数据又进一步构造为一个资源树。如下所示的 `IMAGE_RESOURCE_DIRECTORY`（资源目录）结构体，形成了树的根和节点。

`WINNT.H`

```
typedef struct _IMAGE_RESOURCE_DIRECTORY {
    ULONG   Characteristics;         //特征
    ULONG   TimeDateStamp;           //时间日期戳
    USHORT  MajorVersion;            //主版本号
```
