
## IMAGE_OPTIONAL_HEADER32

`IMAGE_OPTIONAL_HEADER` equ `<IMAGE_OPTIONAL_HEADER32>`

第 1 个 16 位字是 `Magic`，总是 `0x010b`。

下面 2 个字节是连接器的版本号 `MajorLinkerVersion` 和 `MinorLinkerVersion`，这些值都不可靠，不能总是妥当的反映连接器版本。有些连接器不设置该域。

下面 3 个 `longwords`(32 位) 指定执行代码尺寸 `SizeOfCode`，初始化数据尺寸 `SizeOfInitializedData`，所谓的数据段 "data segment"，未初始化数据尺寸 `SizeOfUninitializedData`，所谓的 "bss segment"。这些数值也不可靠。

往下一个 32 位的 `RVA` 是入口点的偏移量 `AddressOfEntryPoint`。执行从此开始。

下面 2 个 32 位是可执行代码 `BaseOfCode` 和初始化数据 `BaseOfData` 的 `RVAs`。我们对它没有兴趣，因为可以通过节来查看更可靠的信息。非初始化数据没有 `RVA`。

下面是一个 32 位值 `ImageBase`，作为整个文件的优先加载地址，包括所有头在内。该值总是 64KB 的倍数，文件已经被连接器重定位，如果文件能够真正加载到这个地址，加载器不必重定位文件。如果另一个映像已经被加载到那个地址，则优先地址不可使用。这种情况下，映像被加载到其他地址，需要重定位。如果映像是 DLL，还有更多结果，因为 "bound imports" 不再有效，需要对使用 DLL 的执行文件进行修正。参见 `import directory`。

下面 2 个 32 位是当映像文件加载后，PE 文件的节在内存内的对齐 `SectionAlignment`，以及在文件内的对齐 `FileAlignment`。一般文件对齐是 512，节对齐是 4096。

下面 2 个 16 位的字是期望的操作系统版本 `MajorOperatingSystemVersion` 和 `MinorOperatingSystemVersion`。

下面 2 个 16 位的字是期望的可执行文件版本 `MajorImageVersion` 和 `MinorImageVersion`。许多连接器不正确设置这些信息。

下面 2 个 16 位的字是期望的子系统版本 `MajorSubsystemVersion` 和 `MinorSubsystemVersion`。这个必须是 Win32 版本或者 POSIX 版本。该版本需要正确提供，因为它被检查并使用。如果程序是 Win32-GUI 并运行在 NT4，子系统版本不是 4.0，对话框不是 3D 效果。

然后是 `Win32VersionValue`，32 位。大部分情况下是 0。

下面是 32 位的映像需要的内存数量 `SizeOfImage`。是所有的头和节的总和，如果节已经对齐。它是给加载器的线索，需要多少页加载映像。

下面一个是 32 位的所有头的总和，包括数据目录和节头。`SizeOfHeaders`。它也是从文件开始到第一节的偏移量。

然后是 32 位的校验码 `CheckSum`。对当前版本的 NT，只校验映像是否是 NT 驱动程序。对于其他可执行文件类型，不必提供这个码，可能为 0。

然后是 16 的子系统 `Subsystem`，表明在什么系统上运行：

- `IMAGE_SUBSYSTEM_NATIVE` (1) 执行文件不需要子系统，用于驱动程序。
- `IMAGE_SUBSYSTEM_WINDOWS_GUI` (2) 映像是 Win32 图形程序可以打开控制台。
- `IMAGE_SUBSYSTEM_WINDOWS_CUI` (3) 映像是 Win32 控制台程序，可以得到缺省控制台。
- `IMAGE_SUBSYSTEM_OS2_CUI` (5) 映像是 OS/2 控制台，程序是 OS/2 格式。
- `IMAGE_SUBSYSTEM_POSIX_CUI` (7) 映像使用 POSIX 控制台子系统。

Windows 95 可执行文件总是使用 Win32 subsystem，于是合法值是 2 和 3。

下面是 16 位 `DllCharacteristics`，表明是否是 DLL。如果 0 位置 1，DLL 被通知进程结合。位 1 置 1，DLL 被通知线程脱离。位 2 置 1，DLL 被通知线程结合。位 3 置 1，DLL 被通知进程脱离。

下面 4 个 32 位预留堆栈大小 `SizeOfStackReserve`，提交的堆栈大小 `SizeOfStackCommit`，预留的堆的大小 `SizeOfHeapReserve` 和提交的堆的大小 `SizeOfHeapCommit`。

预留数量是地址空间不是真实的 RAM，程序启动时，提交的数量是真正的分配的内存。这个值也是堆和栈根据需要增长的一个数量。

例如：一个程序预留 1 MB 的堆并提交的堆时 64KB，该堆就从 64KB 开始，并保证可以加大到 1MB。堆将以 64KB 块增长。该堆在这里是主要堆，默认堆。一个进程可以创建多个堆如果需要的话。栈是第一个线程的栈，进程可以创建许多线程，每个都有自己的堆栈，DLLs 没有栈或者堆，于是在其映像内该值被忽略。

下面是 32 位的 `LoaderFlags`，没有用。

然后是 32 位的 `NumberOfRvaAndSizes`，在随后的目录内的有效项目数。最好使用 `IMAGE_NUMBEROF_DIRECTORY_ENTRIES`，即 16。

下面是具有 `IMAGE_NUMBEROF_DIRECTORY_ENTRIES` (16) 个成员的 `IMAGE_DATA_DIRECTORYs` 结构数组。

```
IMAGE_DATA_DIRECTORY STRUCT
  VirtualAddress    DWORD      ?
  isize             DWORD      ?
IMAGE_DATA_DIRECTORY ENDS
```

每个目录描述了节内特定信息位置，32 bits `RVA VirtualAddress` 和尺寸 32 bit，各个成员索引如下（括号内为索引值）：

- `IMAGE_DIRECTORY_ENTRY_EXPORT` (0) 输出符号目录用于 DLL
- `IMAGE_DIRECTORY_ENTRY_IMPORT` (1) 输入符号目录
- `IMAGE_DIRECTORY_ENTRY_RESOURCE` (2) 资源目录
- `IMAGE_DIRECTORY_ENTRY_EXCEPTION` (3) 异常目录
- `IMAGE_DIRECTORY_ENTRY_SECURITY` (4) 安全目录
- `IMAGE_DIRECTORY_ENTRY_BASERELOC` (5) 重定位表
- `IMAGE_DIRECTORY_ENTRY_DEBUG` (6) 调试目录
- `IMAGE_DIRECTORY_ENTRY_COPYRIGHT` (7) 描述版权串
- `IMAGE_DIRECTORY_ENTRY_GLOBALPTR` (8) 机器值
- `IMAGE_DIRECTORY_ENTRY_TLS` (9) Thread local storage 目录
- `IMAGE_DIRECTORY_ENTRY_LOAD_CONFIG` (10) Load configuration 目录
- `IMAGE_DIRECTORY_ENTRY_BOUND_IMPORT` (11) Bound import directory 目录
- `IMAGE_DIRECTORY_ENTRY_IAT` (12) Import Address Table 输入地址表目录

例如，如果我们找到索引 7 的 2 个值 `0x12000` 和 33，加载地址是 `0x10000`，我们知道版权数据是在 `0x10000+0x12000`，版权字数为 33。如果一个特定类型的目录没有被使用，地址和尺寸都为 0。

## 节目录 Section directories

节包含 2 个部分：节头 `IMAGE_SECTION_HEADER`，节数据。在数据目录之后，我们看到一个具有 `NumberOfSections` 个节头成员的数组，按 `RVA` 排序。

节头包括：

```
IMAGE_SECTION_HEADER STRUCT
    Name1 db IMAGE_SIZEOF_SHORT_NAME dup(?)
    union Misc
        PhysicalAddress dd  ?
        VirtualSize dd      ?
    ends
    VirtualAddress dd       ?
    SizeOfRawData dd        ?
    PointerToRawData dd     ?
    PointerToRelocations dd ?
    PointerToLinenumbers dd ?
    NumberOfRelocations dw  ?
    NumberOfLinenumbers dw  ?
    Characteristics dd      ?
IMAGE_SECTION_HEADER ENDS
```

`IMAGE_SIZEOF_SHORT_NAME` (8) 个字节的数组，组成节的名字。如果所有 8 个字节被用掉，没有 0 做为结尾。典型的名字如 `.data` 或者 `.text` 或者 `.bss`。没有必要前导 `.`，可以是 `CODE` 或 `IAT`。注意名字不全部跟节内容有关。一个 `.code` 节可能或没有可能包括可执行代码，可能只包括输入地址表。可能包含代码和地址表和初始化数据。要找到在节内的信息，必须通过“可选头”内的数据目录查找他们。不要依赖名字，不要假定节的原始数据起始于节的开始。

下面一个成员是 `PhysicalAddress` 和 `VirtualSize` 的 32 位联合体。在目标文件，该地址是内容被重定位的地址，在可执行文件内是内容的尺寸。实际上该域好像没有被使用，有的链接器填入尺寸有的链接器填入地址，有的链接器填入 0。

下一个成员是 `VirtualAddress`，32 位，保存当节的数据加载入内存时的 `RVA`。

然后是 32 位 `SizeOfRawData`，是四舍五入到下个 `FileAlignment` 倍数的大小。

下一个是 `PointerToRawData`，32 位，它是从文件起始到节数据的偏移量。如果是 0，节数据不包含在文件内，在加载时被确定。

然后是 `PointerToRelocations`，32 位，和 `PointerToLinenumbers`，32 位，`NumberOfRelocations`，16 位，`NumberOfLinenumbers`，16 位。所有这些信息仅仅用于目标文件。可执行文件有一个特殊的基准重定位目录，如果存在行号信息，一般包含在特殊目的的调试段或其他。

最后一个是 32 位的 `Characteristics`，它是一组标志描述如何处理节的内存。

- `bit 5 (IMAGE_SCN_CNT_CODE)`，置 1，节内包含可执行代码。
- `bit 6 (IMAGE_SCN_CNT_INITIALIZED_DATA)` 置 1，节内包含的数据在执行前是确定的。
- `bit 7 (IMAGE_SCN_CNT_UNINITIALIZED_DATA)` 置 1，本节包含未初始化的数据，执行前即将被初始化为 0。一般是 BSS。
- `bit 9 (IMAGE_SCN_LNK_INFO)` 置 1，节内不包含映象数据除了注释，描述或者其他文档外，是一个目标文件的一部分，可能是针对链接器的信息。比如哪个库被需要。
- `bit 11 (IMAGE_SCN_LNK_REMOVE)` 置 1，在可执行文件链接后，作为文件一部分的数据被清除。
- `bit 12 (IMAGE_SCN_LNK_COMDAT)` 置 1，节包含公共块数据，是某个顺序的打包的函数。
- `bit 15 (IMAGE_SCN_MEM_FARDATA)` 置 1，不确定。
- `bit 17 (IMAGE_SCN_MEM_PURGEABLE)` 置 1，节的数据是可清除的。
- `bit 18 (IMAGE_SCN_MEM_LOCKED)` 置 1，节不可以在内存内移动。
- `bit 19 (IMAGE_SCN_MEM_PRELOAD)` 置 1，节必须在执行开始前调入。
- `Bits 20 to 23` 指定对齐。一般是库文件的对象对齐。
- `bit 24 (IMAGE_SCN_LNK_NRELOC_OVFL)` 置 1，节包含扩展的重定位。
- `bit 25 (IMAGE_SCN_MEM_DISCARDABLE)` 置 1，进程开始后节的数据不再需要。
- `bit 26 (IMAGE_SCN_MEM_NOT_CACHED)` 置 1，节的数据不得缓存。
- `bit 27 (IMAGE_SCN_MEM_NOT_PAGED)` 置 1，节的数据不得交换出去。
- `bit 28 (IMAGE_SCN_MEM_SHARED)` 置 1，节的数据在所有映象例程内共享，如 DLL 的初始化数据。
- `bit 29 (IMAGE_SCN_MEM_EXECUTE)` 置 1，进程得到“执行”访问节内存。
- `bit 30 (IMAGE_SCN_MEM_READ)` 置 1，进程得到“读出”访问节内存。
- `bit 31 (IMAGE_SCN_MEM_WRITE)` 置 1，进程得到“写入”访问节内存。

在节头后面我们看到节自身，他们在文件内按照 `FileAlignment` 字节数对齐。即在可选头和每个节的后面将添加 0。节按照他们的 `RVAs` 排序。当加载到 RAM，节的对齐按照 `SectionAlignment`。

例如：一个可选头在偏移量 981 处结束，文件对齐为 512，第一个节起始于 1024。你可以借助 `PointerToRawData` 或者 `VirtualAddress` 找到节，不必用对齐找节。（节头哪里去了？此段有问题！！）

```
    +-------------------+
    | DOS-stub          |
    +-------------------+
    | file-header       |
    +-------------------+
    | optional header   |
    |- - - - - - - - - -|
    |                   |----------------+
    | data directories  |                |
    |                   |                |
    |(RVAs to direc-    |-------------+  |
    |tories in sections)|             |  |目录在节内的相对地址
    |                   |---------+   |  |
    |                   |         |   |  |
    +-------------------+         |   |  |
    |                   |-----+   |   |  |
    | section headers   |     |   |   |  |
    | (RVAs to section  |--+  |   |   |  |节边界的相对地址
    |  borders)          |  |  |   |   |  |
    +-------------------+<-+  |   |   |  |
    |                   |     |   |<-+  |
    | section data 1    |     |       |  |节数据
    |                   |     |<-----+  |
    +-------------------+<----+          |
    |                   |                |
    | section data 2    |                |
    |                   |<--------------+
    +-------------------+
```

每个节都有一个节头，每个数据目录将指向一个节（几个数据目录可能指向同一个节），有的节可能没有数据目录指向它。

## 节的原始数据 Sections' raw data

### 概要

所有的节调入内存后按照 `SectionAlignment` 对齐，`FileAlignment` 是节在文件内对齐字节数。节由节头内的项目来描述，可以通过 `PointerToRawData` 在文件内找到节，在内存内通过 `VirtualAddress` 找到节，长度是 `SizeOfRawData`。
