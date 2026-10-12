
# PE 结构与节表详解

## 目录表与导入信息
是不是所有的目录表都要关心呢？其实要把这些目录表都研究清楚是个很大的课题，对于我们这个程序，只需关心第 2 个元素，导入目录，它标识了我们的程序从其他模块导入的函数信息。因为我们要显示一个消息框，所以要导入 `user32.dll` 库中的 `MessageBoxA` 函数。程序要正常退出，又要导入 `kernel32.dll` 库中的 `ExitProcess` 函数。因此需要构造这个目录表。然而上面已说明每个目录是一个 `IMAGE_DATA_DIRECTORY` 结构，该结构具有两个成员，第一个成员表示目录表的起始 RVA 地址，第二个成员表示目录表的长度。我们将把这个目录表构造到 `.rdata` 段中，所以暂时先不填写。但是要留出空位来，为了记住该位置，我们先都填写为 `a`，即：“aaaaaaaa”，“aaaaaaaa”。注意因为要文件对齐，所以其余的统统添零直到地址 `1a7h` 处。此时完成的代码如图 7 所示：

![](img/9f2d238ba92858bd708a36d1451dc8d0.jpg)

图 7 完成 PE 结构中的 PE 头部分

## 节表结构
接下来完成各段头部，又称为节表。一个程序中用到的所有代码、资源、全局数据等信息分布在各个节中，而各个节的信息，如节加载位置，节大小，节属性等信息都由紧跟 PE 头之后的节表所指出。它实际上就是紧挨着 PE 头的一个结构数组，该数组成员的数目由 `file header` (`IMAGE_FILE_HEADER`) 结构中 `NumberOfSections` 域的域值来决定。节表结构又命名为 `IMAGE_SECTION_HEADER`。

我们这里有 3 个段，`.text` (代码段)，`.rdata` (只读数据段)，`data` (全局变量数据段)。每段是一个 `IMAGE_SECTION_HEADER` 结构，具有 10 个成员。`IMAGE_SECTION_HEADER` 结构定义如下：

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

首先我们来完成 `.text` 段。

- **成员 1（Name）**：8 个字节，表示该节的名称，我们这里该节的名字为 `.text`，那么此值应该是他的 ASKII 码值应该为“2E74657874000000”。
- **成员 2（VirtualSize）**：4 个字节，表示该节数据映射到内存后所占字节数。在这里是指有效代码所占的字节数。稍后我们将把程序的执行代码指令写入到文件中，总共有多少字节的指令需要那时计算，我们也可以提前将代码准备好并计算出长度将其填写。笔者已经计算完毕，将写入 `26h` 个字节的代码。所以此时仍然将此值设为“26000000”。我们也可以填写 `26h` 经过内存对齐后的值，即“00100000”。
- **成员 3（VirtualAddress）**：4 个字节，表示在 `.text` 段映射到内存中的起始地址，那么这个值如何得来呢？我们知道 `.text` 是紧跟 PE 结构后的，然后整个 PE 头结构映射到内存后占的大小为 `1000h`（因为 PE 头本身结构小于 `1000h` 个字节，而经过内存对齐后便为 `1000h)`，那么此值便得到了，为 `0x00001000`，因此此处填写“00100000”。这个时候我们已经可以完成前面遗留的一个问题，再填写 `IMAGE_OPTIONAL_HEADER32` 结构的第七个成员的时候，它实际上是程序的入口地址，当时已经讲解此入口地址实际就是 `.text` 段的起始地址，所以可以将此处的“aaaaaaaa”更改为“00100000”。

> **提示**：程序的入口地址并不一定就代码段 `.text` 的起始位置。读者只需知道 `IMAGE_OPTIONAL_HEADER32` 结构的第 7 个成员所表示的含义是程序的入口地址。它通常是由编译器生成的。因为我们没有使用编译器，而是要手工打造一个可执行程序，所以所有的成员值都要自己来安排。只要按照 PE 结构的要求，安排合理即可。因此我们可以把代码的起始地址随意安排什么地方，只要安排的那个地址刚好又是我们保存的程序执行代码的入口即可。我们为了方便将其放在 `.text` 段的起始地址处。

- **成员 4（SizeOfRawData）**：4 个字节，表示 `.text` 段在文件中所占的大小。因为我们的实际代码只有 `26h` 个字节，那么这个值可以填写“26000000”，也可以填写此值经过文件对齐后的值即 `200h`，所以也可以填写此值为“00020000”。
- **成员 5（PointerToRawData）**：4 个字节，表示 `.text` 段在文件中的起始地址，上面已经计算过 PE 文件的总长度为 `400h`，他实际上也就是 `.text` 的起始偏移地址，此值为“00040000”。
- **成员 6（PointerToRelocations）、7（PointerToLinenumbers）、8（NumberOfRelocations）、9（NumberOfLinenumbers）**：均占 4 个字节，都仅用于目标文件，我们这里用零来填充。
- **成员 10（Characteristics）**：4 个字节。包含标记以指示节属性，比如节是否含有可执行代码、初始化数据、未初始数据，是否可写、可读等。这个值实际上是二进制位进行或运算得到的值。各二进制位表示的意义如下：

| 位 | 标志 | 说明 |
| :--- | :--- | :--- |
| 5 | `IMAGE_SCN_CNT_CODE` | 置 1，节内包含可执行代码。 |
| 6 | `IMAGE_SCN_CNT_INITIALIZED_DATA` | 置 1，节内包含的数据在执行前是确定的。 |
| 7 | `IMAGE_SCN_CNT_UNINITIALIZED_DATA` | 置 1，本节包含未初始化的数据，执行前即将被初始化为 0。一般是 BSS. |
| 9 | `IMAGE_SCN_LNK_INFO` | 置 1，节内不包含映象数据除了注释，描述或者其他文档外，是一个目标文件的一部分，可能是针对链接器的信息。比如哪个库被需要。 |
| 11 | `IMAGE_SCN_LNK_REMOVE` | 置 1，在可执行文件链接后，作为文件一部分的数据被清除。 |
| 12 | `IMAGE_SCN_LNK_COMDAT` | 置 1，节包含公共块数据，是某个顺序的打包的函数。 |
| 15 | `IMAGE_SCN_MEM_FARDATA` | 置 1，不确定。 |
| 17 | `IMAGE_SCN_MEM_PURGEABLE` | 置 1，节的数据是可清除的。 |
| 18 | `IMAGE_SCN_MEM_LOCKED` | 置 1，节不可以在内存内移动。 |
| 19 | `IMAGE_SCN_MEM_PRELOAD` | 置 1， 节必须在执行开始前调入。 |
| 20-23 | - | 指定对齐。一般是库文件的对象对齐。 |
| 24 | `IMAGE_SCN_LNK_NRELOC_OVFL` | 置 1， 节包含扩展的重定位。 |
| 25 | `IMAGE_SCN_MEM_DISCARDABLE` | 置 1，进程开始后节的数据不再需要。 |
| 26 | `IMAGE_SCN_MEM_NOT_CACHED` | 置 1，节的 数据不得缓存。 |
| 27 | `IMAGE_SCN_MEM_NOT_PAGED` | 置 1，节的 数据不得交换出去。 |
| 28 | `IMAGE_SCN_MEM_SHARED` | 置 1，节的数据在所有映象例程内共享，如 DLL 的初始化数据。 |
| 29 | `IMAGE_SCN_MEM_EXECUTE` | 置 1，进程得到“执行”访问节内存。 |
| 30 | `IMAGE_SCN_MEM_READ` | 置 1，进程得到“读出”访问节内存。 |
| 31 | `IMAGE_SCN_MEM_WRITE` | 置 1，进程得到“写入”访问节内存。 |

在我们这里，因为这是代码段，所以 `bit 5` (`IMAGE_SCN_CNT_CODE`) 位要置 1，一般代码段都含有初始化数据，那么 `bit 6` (`IMAGE_SCN_CNT_INITIALIZED_DATA`) 位要置 1，又因为代码段的代码可以执行的，所以 `bit 29` (`IMAGE_SCN_MEM_EXECUTE`) 位要置 1，那么这 3 个二进制位进行或运算最终得到的二进制值为“00100000000000000000000001100000”，将其转换为十六进制值为 `0x20000060`，所以此处应该填写“60000020”。

到此整个 `.text` 头编写完毕，按照上面的方法，分别填写 `.rdata` 段和 `.data` 段。因为要文件对齐，所以后面的代码用零补齐，直到 `3ffh`。

## PE 加载器处理流程
PE 加载器根据节表加载程序的过程是这样的：读取 `IMAGE_FILE_HEADER` 的 `NumberOfSections` 域，得到文件中节的数目。读取 `IMAGE_OPTIONAL_HEADER32` 的 `SizeOfHeaders` 域值，将其作为节表的文件偏移，并以此定位节表。遍历整个结构数组检查各成员值。对于每个结构，读取 `PointerToRawData` 域值并定位到该文件偏移量。然后再读取 `SizeOfRawData` 域值来决定映射内存的字节数。将 `VirtualAddress` 域值加上 `ImageBase` 域值等于节起始的虚拟地址。然后把节映射进内存，并根据 `Characteristics` 域值设置属性。遍历整个数组，直至所有节都已处理完毕。

> **提示**：感染型病毒经常通过增加节来达到感染正常文件的目的。因为感染正常文件需要添加病毒代码，病毒最常用的方法是新建一个节，然后将病毒代码放置在新建的节中，然后修改程序入口地址使其指向病毒代码。这样程序运行以后首先运行的是病毒代码，等病毒代码运行完毕才会跳转到被感染程序的原始入口地址处执行。

最后的编写结果如图 8 所示：

![](img/70913bffc234d24fe27051ba20117a81.jpg)

图 8 完成各个节表

## 编写段实体部分
至此，我们已经完成了 PE 头结构的编写。为了让我们写的程序可以运行，我们还要完成 `.text` (代码段)，`.rdata` (只读数据段)，`data` (全局变量数据段) 三个段的实体部分。

首先编写 `.text` 段，他紧接着 PE 结构后面。前面已经说过，`.text` 段中存放所有可执行的指令代码（机器码）。我们可以通过先编写汇编指令（调用 `MessageBoxA` 和 `ExitProcess` 两个函数），然后反汇编出机器代码抄到这里就可以了。我们的程序功能是弹出一个消息框，这需要用到 `MessageBoxA` 函数，当用户单击确定以后程序要退出，这又需要用到 `ExitProcess` 函数。这两个函数调用的汇编代码如下：

```asm
push    0          ; MessageBoxA 的第四个参数，即消息框的风格，这里传入 0。
push    ？？？？   ; 第三个参数，消息框的标题字符串所在的地址，需要计算。
push    ？？？？   ; 第二个参数，消息框的内容字符串所在的地址，需要计算。
push    0          ; 第一个参数，消息框所属窗口句柄，这里填 0。
call    ？？？？ ；调用 MessageBoxA，实际是跳转到该函数的跳转指令所在地址
push    0       ；ExitProcess 函数的参数，程序退出码，传入 0.
call    ？？？？ ；调用 ExitProcess，实际是跳转到该函数的跳转指令所在地址
jmp    ？？？？ ；跳转到 MessageBoxA 的真正地址处。
jmp    ？？？？ ；跳转到 ExitProcess 的真正地址处。
```

首先计算 `MessageBoxA` 两个字符串参数的地址，实际上是两个字符串，“消息框”和“HelloWorld ！”。这两个串需要保存到文件中，我们设计将其存放在 `.data` (全局变量数据段)。这个段位于 `.rdata` 段之后，那么可以计算它的起始内存地址，PE 头 `1000h`，`.text` 段只有 `26h` 字节，内存对齐后为 `1000h`，`.data` (只读数据段) 是准备用来完成导入表的段，它肯定也不会超过 `1000h`，所以对齐后应为 `1000h`，因此紧随其后的 `.rdata` 的起始内存地址应该在偏移为：

`1000h + 1000h + 1000h = 3000h` 处，程序的基址为 `400000h`，故此得到 `.rdata` 的绝对内存地址为：

`0x00400000 + 0x00003000 = 0x00403000`。我们将“消息框”字符串放于此处，该字符串占 7 个字符，那么紧随其后的“Hello World !”字符串的地址应该为 `0x00403000 + 7 = 0x00403007`。

所以修正以上汇编代码为：

```asm
push    0          ; MessageBoxA 的第四个参数，即消息框的风格，这里传入 0。
push    0x403000   ; 第三个参数，消息框的标题字符串所在的地址。
push    0x403007   ; 第二个参数，消息框的内容字符串所在的地址。
push    0          ; 第一个参数，消息框所属窗口句柄，这里填 0。
call    ？？？？ ；调用 MessageBoxA，实际是跳转到该函数的跳转指令所在地址。
push    0       ；ExitProcess 函数的参数，程序退出码，传入 0.
call    ？？？？ ；调用 ExitProcess，实际是跳转到该函数的跳转指令所在地址。
jmp    ？？？？ ；跳转到 MessageBoxA 的真正地址处。
jmp    ？？？？ ；跳转到 ExitProcess 的真正地址处。
```

其次需要计算 `MessageBoxA` 和 `ExitProcess` 两个函数所在地址，这个需要完成 `.rdata` 段的导入表才可以得到。所以首先用 `200h` 个 `00` 将 `.text` 段填充，待完成 `.rdata` 段后再返过来完成它。

接下来完成 `.rdata` 段，这个段非常重要，也非常繁琐。因为我们要手工打造导入表。通常导入表是由编译器生成的，其生成规则遵循 `IMAGE_IMPORT_DESCRIPTOR` 结构。`IMAGE_IMPORT_DESCRIPTOR` 结构的定义如下：

```c
typedef struct _IMAGE_IMPORT_DESCRIPTOR {
    union {
        DWORD   Characteristics;         // 0 for terminating null import descriptor
        DWORD   OriginalFirstThunk;      // RVA to original unbound IAT (PIMAGE_THUNK_DATA)
    };
    DWORD   TimeDateStamp;               // 0 if not bound,
                                         // -1 if bound, and real date\time stamp
                                         //     in IMAGE_DIRECTORY_ENTRY_BOUND_IMPORT (new BIND)
                                         // O.W. date/time stamp of DLL bound to (Old BIND)
    DWORD   ForwarderChain;              // -1 if no forwarders
    DWORD   Name;
    DWORD   FirstThunk;                  // RVA to IAT (if bound this IAT has actual addresses)
} IMAGE_IMPORT_DESCRIPTOR；
```
