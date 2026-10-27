
# PE 文件结构

## `IMAGE_OPTIONAL_HEADER32` 结构

```
Win32VersionValue DWORD ? ;004ch 未用
SizeOfImage DWORD ? ;0050h 内存中整个PE映像尺寸
SizeOfHeaders DWORD ? ;0054h 所有头＋节表的大小
CheckSum DWORD ? ;0058h
Subsystem WORD ? ;005ch 文件的子系统
DllCharacteristics WORD ? ;005eh
SizeOfStackReserve DWORD ? ;0060h 初始化时的堆栈大小
SizeOfStackCommit DWORD ? ;0064h 初始化时实际提交的堆栈大小
SizeOfHeapReserve DWORD ? ;0068h 初始化时保留的堆大小
SizeOfHeapCommit DWORD ? ;006ch 初始化时实际提交的堆大小
LoaderFlags DWORD ? ;0070h 未用
NumberOfRvaAndSizes DWORD ? ;0074h 下面的数据目录结构的数量
DataDirectory    IMAGE_DATA_DIRECTORY 16 dup(<>) ;0078h
IMAGE_OPTIONAL_HEADER32 ENDS
```

这个结构的字段比较多，我就不一一介绍了，具体请参考《加密与解密》第三版第十章PE文件格式，下面介绍几个比较重要的：

### `AddressOfEntryPoint` 字段

指出文件被执行时的入口地址，这是一个RVA地址（RVA的含义在下一节中详细介绍）。如果在一个可执行文件上附加了一段代码并想让这段代码首先被执行，那么只需要将这个入口地址指向附加的代码就可以了。

### `ImageBase` 字段

指出文件的优先装入地址。也就是说当文件被执行时，如果可能的话，Windows优先将文件装入到由`ImageBase`字段指定的地址中，只有指定的地址已经被其他模块使用时，文件才被装入到其他地址中。链接器产生可执行文件的时候对应这个地址来生成机器码，所以当文件被装入这个地址时不需要进行重定位操作，装入的速度最快，如果文件被装载到其他地址的话，将不得不进行重定位操作，这样就要慢一点。

对于EXE文件来说，由于每个文件总是使用独立的虚拟地址空间，优先装入地址不可能被其他模块占据，所以EXE总是能够按照这个地址装入，这也意味着EXE文件不再需要重定位信息。对于DLL文件来说，由于多个DLL文件全部使用宿主EXE文件的地址空间，不能保证优先装入地址没有被其他的DLL使用，所以DLL文件中必须包含重定位信息以防万一。因此，在前面介绍的`IMAGE_FILE_HEADER`结构的`Characteristics`字段中，DLL文件对应的`IMAGE_FILE_RELOCS_STRIPPED`位总是为0，而EXE文件的这个标志位总是为1。

在链接的时候，可以通过对`link.exe`指定`/base:address`选项来自定义优先装入地址，如果不指定这个选项的话，一般EXE文件的默认优先装入地址被定为`00400000h`，而DLL文件的默认优先装入地址被定为`10000000h`。

### `SectionAlignment` 字段和 `FileAlignment` 字段

`SectionAlignment`字段指定了节被装入内存后的对齐单位。也就是说，每个节被装入的地址必定是本字段指定数值的整数倍。而`FileAlignment`字段指定了节存储在磁盘文件中时的对齐单位。

### `DataDirectory` 字段

这个字段可以说是最重要的字段之一，它由16个相同的`IMAGE_DATA_DIRECTORY`结构组成，虽然PE文件中的数据是按照装入内存后的页属性归类而被放在不同的节中的，但是这些处于各个节中的数据按照用途可以被分为导出表、导入表、资源、重定位表等数据块，这16个`IMAGE_DATA_DIRECTORY`结构就是用来定义多种不同用途的数据块的（如表17.4所示）。`IMAGE_DATA_DIRECTORY`结构的定义很简单，它仅仅指出了某种数据块的位置和长度。

```
IMAGE_DATA_DIRECTORY STRUCT
VirtualAddress DWORD ? ；数据的起始RVA
Size DWORD ? ；数据块的长度
IMAGE_DATA_DIRECTORY ENDS
```

一共有十六个`IMAGE_DATA_DIRECTORY`结构：

- `IMAGE_DIRECTORY_ENTRY_EXPORT`：导出表
- `IMAGE_DIRECTORY_ENTRY_IMPORT`：导入表
- `IMAGE_DIRECTORY_ENTRY_RESOURCE`：资源
- `IMAGE_DIRECTORY_ENTRY_EXCEPTION`：异常（具体资料不详）
- `IMAGE_DIRECTORY_ENTRY_SECURITY`：安全（具体资料不详）
- `IMAGE_DIRECTORY_ENTRY_BASERELOC`：重定位表
- `IMAGE_DIRECTORY_ENTRY_DEBUG`：调试信息
- `IMAGE_DIRECTORY_ENTRY_ARCHITECTURE`：版权信息
- `IMAGE_DIRECTORY_ENTRY_GLOBALPTR`：具体资料不详
- `IMAGE_DIRECTORY_ENTRY_TLS`：Thread Local Storage
- `IMAGE_DIRECTORY_ENTRY_LOAD_CONFIG`：具体资料不详
- `IMAGE_DIRECTORY_ENTRY_BOUND_IMPORT`：具体资料不详
- `IMAGE_DIRECTORY_ENTRY_IAT`：导入函数地址表
- `IMAGE_DIRECTORY_ENTRY_DELAY_IMPORT`：具体资料不详
- `IMAGE_DIRECTORY_ENTRY_COM_DESCRIPTOR`：具体资料不详
- 未使用保留

PE文件中定位输出表，输入表和资源等重要数据时，就是从`IMAGE_DATA_DIRECTORY`结构开始的。

如图所示：

![](https://bbs.kanxue.com/upload/attach/201010/288797_vl0vabj975foy1h.jpg)

由于数据结构太多，这里只给出部分，具体请参考《加密与解密》第三版

![](https://bbs.kanxue.com/upload/attach/201010/288797_g12kmdl4bi05d64.jpg)

如图七所示，数据目录表位于`138---1b07h`之间，每个成员占8个字节，分别指向相关的结构，前面四个字节代表`VirtualAddress`（数据块的起始RVA），后面四个字节代表`Size`（数据块的长度），比如上面程序中没有输出表，有输入表，且在`00000140h`处，我们就可以得出此程序的输入表RVA为`00002090`，`Size`为`0000003Ch`。

上面得到的结果和用`LoadPE`查看的结果一样，呵呵，如图所示：

![](https://bbs.kanxue.com/upload/attach/201010/288797_7obd626bfgbofx0.jpg)

上面将PE文件结构的文件头介绍了一下，下面我们开始重点讲解PE中的各个区块

## 区块表

紧跟在`IMAGE_NT_HEADERS`后面的是区块表，它是一个`IMAGE_SECTION_HEADER`结构数组。每个`IMAGE_SECTION_HEADER`结构包含了它所关联区块的信息，如位置，长度，属性；该数组的数目由`IMAGE_NT_HEADERS.FileHeader.NumberOfSections`指出。

### `IMAGE_SECTION_HEADER` 结构

```
IMAGE_SECTION_HEADER STRUCT
  Name1 db IMAGE_SIZEOF_SHORT_NAME dup(?) ；8个字节的节区名称
  union Misc
  PhysicalAddress dd ?
  VirtualSize dd ? ；节区的尺寸
  ends
  VirtualAddress dd ? ；节区的RVA地址
  SizeOfRawData dd ? ；在文件中对齐后的尺寸
  PointerToRawData dd ? ；在文件中的偏移
  PointerToRelocations dd ? ；在OBJ文件中使用
  PointerToLinenumbers dd ? ；行号表的位置（供调试用）
  NumberOfRelocations dw ? ；在OBJ文件中使用
  NumberOfLinenumbers dw ? ；行号表中行号的数量
  Characteristics dd ? ；节的属性
IMAGE_SECTION_HEADER ENDS
```

### `Name1` 字段

这个字段的字段名原来应该是“`Name`”，但是这个名称和MASM中的关键字冲突，所以在定义的时候改为“`Name1`”，`Name1`字段定义了节的名称，字段的长度为8个字节。

PE文件中的节的名称是一个由ANSI字符组成的字符串，但并没有规定以0结束，如果节的名称字符串长度小于8个字节的话，后面以0补齐，但是字符串长度达到8个字节的话，后面就没有0字符了，所以在处理的时候要注意字符串的结束方式。

每个节的名称是惟一的，不能有同名的两个节，但是节的名称不代表任何含义，它仅仅是为了查看方便而设置的一个标记而已，可以选择任何名称甚至将它空着也可以，将包含代码的节命名为“DATA”或者将包含数据的节命名为“CODE”都是合法的。

各种编译器都以自己的方式对节进行命名，所以，在PE文件中可以看到各式各样的节名称，比如，在MASM32产生的可执行文件中，代码节被命名为“.text”；可读写的数据节被命名为“.data”；包含只读数据、导入表以及导出表的节被命名为“.rdata”；而资源节被命名为“.rsrc”等。但是在其他一些编译器中，导入表被单独放在“.idata”中；而代码节可能被命名为“.code”。

当从PE文件中读取需要的节时，不能以节的名称作为定位标准，正确的方法是按照`IMAGE_OPTIONAL_HEADER32`结构中的数据目录字段定位。

### `VirtualSize` 字段

代表节的大小，这是节的数据在没有进行对齐处理前的实际大小。

### `VirtualAddress` 字段

指出节被装载到内存中后的偏移地址，这是一个RVA地址。这个地址是按照内存页对齐的，它的数值总是`SectionAlignment`的值的整数倍。

### `PointerToRawData` 字段

指出节在磁盘文件中的所处的位置。这个数值是从文件头开始算起的偏移量。

### `SizeOfRawData` 字段

指出节在磁盘文件中所占的空间大小，这个数值等于`VirtualSize`字段的值按照`FileAlignment`的值对齐以后的大小。

依靠这4个字段的值，装载器就可以从PE文件中找出某个节（从`PointerToRawData`偏移开始的`SizeOfRawData`字节）的数据，并将它映射到内存中去（映射到从模块基地址开始偏移`VirtualAddress`的地方，并占用以`VirtualSize`的值按照页的尺寸对齐后的空间大小）。

### `Characteristics` 字段

这是节的属性标志字段，其中的不同数据位代表了不同的属性，具体的定义如表17.5所示，这些数据位组合起来描述了节的属性。

结合上面的讲解，对照下面的图示，看就会很清楚，如图所示：

![](https://bbs.kanxue.com/upload/attach/201010/288797_qb8jbzgghfe220x.jpg)

## 学习区块的重要概念

### 区块对齐值

区块的大小是要对齐的，有两种对齐值，一种用于磁盘文件内，另一种用于内存文件中。PE文件头指出了这两个值，它们可以不同。

PE文件头里`FileAlignment`定义了磁盘区块的对齐值，每一个区块从对齐值的倍数的偏移位置开始。而区块的实际代码或数据的大小不一定刚好就是这么多，所以不足的地方一般以`00h`来填充，这就是区块间的间隙。

PE文件头里`SectionAlignment`定义了内存中区块的对齐值。PE文件被映射到内存中，区块总是至少从一个页边界开始，也就是说，当一个PE文件映射到内存中，每个区块的第一个字节对应于某个内存页。

### 内存页的属性

对于磁盘映射文件来说，所有的页都是按照磁盘映射文件函数指定的属性设置的，但是装载可执行文件时，与节对应的内存页的属性要按照节的属性来设置。所以在同一属性模块的内存页中，从不同映射过来的内存页的属性是不同的。

### 节的偏移地址

节的起始地址在磁盘文件中要按照`IMAGE_OPTIONAL_HEADER32`结构的`FileAlignment`字段的值对齐，而被装载到内存中时是按照同一结构中的`SectionAlignment`字段的值来对齐，两者的值可能不同，所以一个节被装入内存后相对于文件头的偏移和在磁盘文件中的偏移量可能是不同的。

### 节的尺寸

对节的尺寸的处理有两个方面：首先是由于磁盘映像和内存映像中节对齐单位不同而造成的长度扩展；其次是对包含未初始化数据的节的处理。

对于未初始化数据来说，没必要为它们在磁盘文件中预留空间，只要在可执行文件被装载到内存中后为它们分配空间就可以了，所以包含未初始化数据的节在磁盘文件中的长度被定义为0，但是装载到内存中的地址和大小是被明确指定的。

### 不进行映射的节

有些节中包含的数据仅仅在装载的时候用到，当文件装载完毕时候，它们不会被递交到物理内存页。

今天就讲到这里吧，也有点累了，写文章很累的，用了三个多小时，不过没事，只要能给大家一点帮助，付出的就是值得的，明天咱们继续学习输入表，输出表，基址重定位，资源，我打算在两天之类，把PE资源的理论介绍完毕，后面几天我重点讲解PE方面的编程知识，这样理论与实践相结合，学起来会更清楚明白一点！！![](https://bbs.kanxue.com/view/img/face/1.gif)

格式经较乱，大家将就的看吧~~~不好意思了![](https://bbs.kanxue.com/view/img/face/16.gif)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

## 上传的附件

- [图1.jpg](javascript:void(0);)（22.69kb，4629次下载）
- [图2.jpg](javascript:void(0);)（19.02kb，5638次下载）
- [图3.jpg](javascript:void(0);)（14.64kb，4574次下载）
- [图4.jpg](javascript:void(0);)（75.55kb，4586次下载）
- [图5.jpg](javascript:void(0);)（55.64kb，4576次下载）
- [图6.jpg](javascript:void(0);)（136.30kb，4588次下载）
- [图7.jpg](javascript:void(0);)（50.65kb，4544次下载）
- [图8.jpg](javascript:void(0);)（45.66kb，4542次下载）
- [图9.jpg](javascript:void(0);)（79.36kb，4571次下载）
