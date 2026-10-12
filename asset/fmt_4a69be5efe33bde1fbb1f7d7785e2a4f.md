
# 七、PE 的可选头

```c
} IMAGE_FILE_HEADER, *PIMAGE_FILE_HEADER;  

 #define IMAGE_SIZEOF_FILE_HEADER             20         //定义一个常量  
```

注意：文件头结构的大小被方便地定义在这个包含文件之中。这使得取得这个结构的大小非常简单，但我发觉对这个结构本身而言使用 `sizeof` 函数会更加简单，因为这样除了要记住 `IMAGE_FILE_HEADER`（映像文件的文件头）这个结构名之外，我就不必再去记住 `IMAGE_SIZEOF_FILE_HEADER`（映像文件的文件头大小）这个常量的名字了。另一方面，要记住所有结构的名字（对我来说）实际上已经具有挑战性了，特别是这些结构中没有一个是除了在 `WINNT.H` 这个包含文件中有记载外，在别的地方还有记载的。

## PE 文件头信息

PE 文件中的信息本来就是那些让系统或者是应用程序决定怎样处理此文件的高级信息。其中的第一个域（`Machine`）是用来指出此可执行文件是设计运行在何种机器之上的，例如：DEC® Alpha, MIPS R4000, Intel® x86, 或者其他的处理器等。系统在进一步处理其余的文件数据之前可以利用这个信息来确定该怎样来处理这个文件。

### 特性（Characteristics）

特性（`Characteristics`）域定义了文件的特殊特性。例如：考虑一下怎样为一个可执行文件管理单独的调试文件。将调试信息从 PE 文件中剥离出去并将其储存在给调试器使用的一个调试文件(`.DBG`)中是可能的。要做到这点，调试器需要知道是不是要在一个单独的文件中查找调试信息，以及调试信息是否已被从文件中剥离等等。一个调试器也可以通过深入可执行文件的内部寻找调试信息来找出它所需的东西，但为了避免调试器必须搜寻整个文件，一个指明文件已经被剥离(`IMAGE_FILE_DEBUG_STRIPPED`)的文件特性被创造进来了。这样调试器就可通过查看 PE 文件头来快速地确定调试信息是否就在文件中了。

`WINNT.H` 文件中定义了好几个其他的标示文件头信息的标志，它比上例中所标出的方式要多得多。我将把这个问题留给读者作为查找这些标志的练习，以让你们看看它们之中是否还有有意义的东西。它们就位于 `WINNT.H` 文件中、上面所述的 `IMAGE_FILE_HEADER` 结构之后。

### 节数（NumberOfSections）

PE 文件头结构中另一个有用的项就是节数（`NumberOfSections`）域。事实上你需要知道文件有多少个节----更准确地说是，多少个节头和节身----以便容易地提取出信息。每一个节头和节身都依次地被放到了文件当中，所以需要节的数量来确定节头和节身到哪里结束。下面的函数可从 PE 文件头中提取出节的数量：

`PEFILE.C`

```c
 int   WINAPI NumOfSections (
     LPVOID    lpFile)
 {
     /* 文件头中标示的节的数量。 */
     return (int)((PIMAGE_FILE_HEADER)
                 PEFHDROFFSET (lpFile))->NumberOfSections);
 }
```

正如你所见到的，`PEFHDROFFSET` 以及其他的宏都可以很方便的得到。

## 可执行文件中此后的 224 个字节构成 PE 的可选头

尽管它的名字叫“可选头”，但我可以肯定在 PE 可执行文件中它并不是一个可选的项。指向可选头的指针可以用 `OPTHDROFFSET` 宏来获得：

`PEFILE.H`

```c
 #define OPTHDROFFSET(a) ((LPVOID)((BYTE *)a           + \
     ((PIMAGE_DOS_HEADER)a)->e_lfanew + SIZE_OF_NT_SIGNATURE + \
     sizeof (IMAGE_FILE_HEADER)))
```

可选头中包含和可执行映像文件有关的大多数有用的信息，比如：初始栈大小、程序入口点位置、优先的基地址、操作系统版本号、节对齐信息、等等。`IMAGE_OPTIONAL_HEADER`（可选头）结构描绘的可选头如下所示：

`WINNT.H`

```c
 typedef struct _IMAGE_OPTIONAL_HEADER {
     //
     //标准域
     //
     USHORT  Magic;                    //魔数
     UCHAR   MajorLinkerVersion;       //链接器主版本号
     UCHAR   MinorLinkerVersion;       //链接器小版本号
     ULONG   SizeOfCode;               //代码大小
     ULONG   SizeOfInitializedData;    //已初始化数据大小
     ULONG   SizeOfUninitializedData;  //未初始化数据大小
     ULONG   AddressOfEntryPoint;      //入口点地址
     ULONG   BaseOfCode;              //代码基址
     ULONG   BaseOfData;              //数据基址
     //
     //NT 增加的域
     //
     ULONG   ImageBase;                //映像文件基址
     ULONG   SectionAlignment;         //节对齐
     ULONG   FileAlignment;            //文件对齐
     USHORT  MajorOperatingSystemVersion;//操作系统主版本号
     USHORT  MinorOperatingSystemVersion;//操作系统小版本号
     USHORT  MajorImageVersion;        //映像文件主版本号
     USHORT  MinorImageVersion;        //映像文件小版本号
     USHORT  MajorSubsystemVersion;    //子系统主版本号
     USHORT  MinorSubsystemVersion;    //子系统小版本号
     ULONG   Reserved1;               //保留项 1
     ULONG   SizeOfImage;             //映像文件大小
     ULONG   SizeOfHeaders;           //所有头的大小
     ULONG   CheckSum;               //校验和
     USHORT  Subsystem;              //子系统
     USHORT  DllCharacteristics;      //DLL 特性
     ULONG   SizeOfStackReserve;       //保留栈的大小
     ULONG   SizeOfStackCommit;        //指定栈的大小
     ULONG   SizeOfHeapReserve;        //保留堆的大小
     ULONG   SizeOfHeapCommit;         //指定堆的大小
     ULONG   LoaderFlags;             //加载器标志
     ULONG   NumberOfRvaAndSizes;      //RVA 的数量和大小
     IMAGE_DATA_DIRECTORY DataDirectory  [IMAGE_NUMBEROF_DIRECTORY_ENTRIES];   //数据目录数组
 } IMAGE_OPTIONAL_HEADER, *PIMAGE_OPTIONAL_HEADER;
```

正如你所见到的，这个结构中的域的列表相当长。为了不使你对所有这些域的描述感到厌烦，我将只讨论其中有用的域----也就是说，那些对探索 PE 文件格式的背景有用的域。

## （一）、标准域

首先，请注意一下：这个结构是被分成“标准域”和“NT 增加的域”（两组）的。所谓标准域就是那些常见于通用目标文件格式(`COFF`)中的项，大多数 UNIX 可执行文件都使用这些项。尽管标准域保留了 `COFF` 中定义的名字，但 Windows NT 实际上是为了别的目的来使用其中的一些域的，因此这些域使用别的名字可能会表达得更好。

*   `Magic`（魔数）。我没能追查到此域的实际用途。就 `EXEVIEW.EXE` 这个例子程序而言，此值为 `0x010B` 或 `267`。
*   `MajorLinkerVersion`（链接器主版本号）, `MinorLinkerVersion`（链接器小版本号）。指出链接此映像文件的链接器的版本号。初期的、和 Windows NT 的 build 438 版一起发行的 Windows NT 软件开发工具箱[`Software Development Kit (SDK)`]中，包含的链接器版本为 `2.39` (`2.27` 十六进制)。
*   `SizeOfCode`（代码大小）。可执行文件代码的大小。
*   `SizeOfInitializedData`（已初始化数据大小）。 已初始化数据的大小。
*   `SizeOfUninitializedData`（未初始化数据大小）。未初始化数据的大小。
*   `AddressOfEntryPoint`（入口点地址）。在标准域当中，入口点地址域对 PE 文件格式是最有用的。此域指出了应用程序入口点的位置以及，也许对系统黑客更重要的，输入地址表 (`Import Address Table`，`IAT`)结束的位置。下面的函数演示了怎样从可选头中检索出 Windows NT 可执行映像文件的入口点。

    `PEFILE.C`

    ```c
        LPVOID  WINAPI GetModuleEntryPoint (
            LPVOID    lpFile)
        {
            PIMAGE_OPTIONAL_HEADER   poh;
            poh = (PIMAGE_OPTIONAL_HEADER)OPTHDROFFSET (lpFile);
            if (poh != NULL)
                return (LPVOID)poh->AddressOfEntryPoint;
            else
                return NULL;
        }
    ```

*   `BaseOfCode`（代码基址）。 已载入的映像文件中代码（`.text`节）的相对偏移量。
*   `BaseOfData`（数据基址）。 已载入的映像文件中未初始化数据（`.bss`节）的相对偏移量。

## （二）、Windows NT 增加的域

Windows NT 的 PE 文件格式中新增的增加域提供了加载器对许多 Windows NT 特有的处理行为的支持。下面是对这些域的一个小结。

*   `ImageBase`（映像文件基址）。是将可执行文件镜像到的、一个进程④地址空间中的优先的基地址。Windows NT 的微软 Win32 SDK 将这个地址默认为 `0x00400000`，但是你可以用`-BASE:`链接器开关来改变这个默认值。
*   `SectionAlignment`（节对齐）。每个节都被依次地载入一个进程的地址空间，开始时的是 `ImageBase`（映像文件基址）。 节对齐（`SectionAlignment`）则指出了在已被载入后一个节能拥有的最小的空间数量----也就是说，节是按照节对齐（`SectionAlignment`）数的边界来对齐的。

    节对齐数不能小于页的大小(在 x86 平台上目前是 `4096` 字节)并且必须是 Windows NT 的虚拟地址管理器（virtual memory manager）⑤的状态所设定的页大小的倍数。 `4096` 字节是 x86 链接器默认的，但此值能使用`-ALIGN:` 链接器开关来重新设定。

*   `FileAlignment`（文件对齐）。是载入前，映像文件中信息块的最小的间隔尺寸。例如：链接器会用零填充节身（一个的节的原始数据）到文件中最近的文件对齐（`FileAlignment`）数的边界。前面提到的 `2.39` 版的链接器将按照 `0x200` 字节的间隔尺寸来对齐映像文件。此值被限定为 `512` 和 `65,535` 之间的一个 2 的幂的值。
*   `MajorOperatingSystemVersion`（操作系统主版本号）。指出 Windows NT 操作系统的主版本号，目前对于 Windows NT 1.0 版此值被设为 `1`。
*   `MinorOperatingSystemVersion`（操作系统小版本号）。指出 Windows NT 操作系统的小版本号，目前对于 Windows NT 1.0 版此值被设为 `0`。
*   `MajorImageVersion`（映像文件主版本号）。被用来指出应用程序的主版本号；对微软的 Excel 4.0 版，此值被设为 `4`。
*   `MinorImageVersion`（映像文件小版本号）。被用来指出应用程序的小版本号；对微软的 Excel 4.0 版，此值被设为 `0`。
*   `MajorSubsystemVersion`（子系统主版本号）。指出 Windows NT Win32 子系统的主版本号，目前对于 Windows NT 3.10 版来说此值被设为 `3`。
*   `MinorSubsystemVersion`（子系统小版本号）。指出 Windows NT Win32 子系统的小版本号，目前对于 Windows NT 3.10 版来说此值被设为 `10`。
*   `Reserved1`（保留项 1）。目的不清楚，目前系统不使用它，链接器将它设为 `0`。
*   `SizeOfImage`（映像文件大小）。指出在地址空间中为载入的可执行映像文件保留的地址空间数量。此值主要受节对齐（`SectionAlignment`）的值的影响。例如：假设一个系统的页为固定大小的 `4096` 字节。假如你有一个 `11` 个节的可执行文件，并且每节都小于 `4096` 字节，如果你按照 `65,536` 字节边界对齐，映像文件大小（`SizeOfImage`）域将被设为 `11 * 65,536 = 720,896` (`176` 页)。同样的文件在按 `4096` 字节边界对齐链接时映像文件大小（`SizeOfImage`）域的结果将为 `11 * 4096 = 45,056` (`11` 页) 。这是在每个节需求的内存数都小于一页时的简单例子。实际使用中，链接器会通过计算每个节的单独的大小来确定映像文件大小（`SizeOfImage`）的值。它首先确定一个节需要多少字节，并舍入到最近的下一个页边界的值，最后将页数也舍入到最近的节对齐（`SectionAlignment`）边界时的值。总数就是每个节单独需要的数量和。
*   `SizeOfHeaders`（所有头的大小）。此域指出使用多少文件空间来描述所有的文件头，包括 MS-DOS 头、PE 文件头、PE 可选头、和 PE 节头等。在文件中节身将从这个位置开始。
*   `CheckSum`（校验和）。一个校验和的值是用来在载入时验证可执行文件的。此值由链接器来设定和检验。设定此校验和值的算法是（微软）私人拥有的信息，不会被发表出来的。
*   `Subsystem`（子系统）。此域是用来为可执行文件鉴定目标子系统的。在 `WINNT.H` 文件中，每个可能的子系统值都被列在了可选头（`IMAGE_OPTIONAL_HEADER`）结构之后。
*   `DllCharacteristics`（DLL 特性）。是被用来指出一个 DLL 映像文件中是否包含进程入口点以及线程的初始化和终止等信息的一系列标志的。
*   `SizeOfStackReserve`（保留栈的大小）, `SizeOfStackCommit`（指定栈的大小）, `SizeOfHeapReserve`（保留堆的大小）, `SizeOfHeapCommit`（指定堆的大小）。这些域控制了为栈和缺省堆保留和指定的地址空间的数量。栈和堆两项的缺省值都是 `1` 页的指定值和 `16` 页的保留值。这些值也可以通过链接器的`-STACKSIZE:` 和`-HEAPSIZE:`开关来设定。
*   `LoaderFlags`（加载器标志）。告诉加载器是否在载入时中断、是否在载入时调试、或者是缺省设置，这些都是为了让它正常地运行。
*   `NumberOfRvaAndSizes`（RVA 的数量和大小）。此域标示跟在后面的数据目录（`DataDirectory`）数组的长度。特别需要注意的是：此域是用来标示这个数组的大小的，而不是数组中有效项的数量的。
*   `DataDirectory`（数据目录）。数据目录标出在文件中的什么地方能找到可执行文件的其他重要组成部分的信息。它除了位于可选头结构尾部的数据目录（`IMAGE_DATA_DIRECTORY`）结构之外没有别的东西。目前的 PE 文件格式定义了 `16` 个可能的数据目录，现在只使用了其中的 `11` 个。

## （三）、数据目录

正如 `WINNT.H` 定义的那样，数据目录有：

`WINNT.H`

```c
// 各个目录项
// 输出目录
```
