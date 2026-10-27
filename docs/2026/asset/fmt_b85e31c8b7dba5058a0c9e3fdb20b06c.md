
`WriteConsoleA(GetStdHandle(STD_OUTPUT_HANDLE),hello,sizeof(hello)-1,&written,0);`

`return;`

现在我将笨拙的将它汇编出来：

```
startup:

                ; WriteConsole()的参数, 反向的

    6A 00                     push      0x00000000

    68 ?? ?? ?? ??            push      offset _written

    6A 0D                     push      0x0000000d

    68 ?? ?? ?? ??            push      offset hello

                ; GetStdHandle()的参数

    6A F5                     push      0xfffffff5

    2E FF 15 ?? ?? ?? ??      call      dword ptr cs:__imp__GetStdHandle@4

                ; 结果是WriteConsole()的参数

    50                        push      eax

    2E FF 15 ?? ?? ?? ??      call      dword ptr cs:__imp__WriteConsoleA@20

    C3                        ret

    hello:

    68 65 6C 6C 6F 2C 20 77 6F 72 6C 64 0A   "hello, world\n"

    _written:

    00 00 00 00
```

以上就是编译的部分。任何人都能做到这点。从现在起让我们扮演起链接器的角色，这会非常有趣 :-)

我需要先找出函数`WriteConsoleA()`和`GetStdHandle()`。碰巧它们都在`kernel32.dll`中。（这是“输入库”部分。）

现在我开始做可执行文件。问号代表待定的值；它们将在以后被修正。

首先是`DOS`-根，开始于`0x0`，有`0x40`字节长：

```
    00 | 4d 5a 00 00 00 00 00 00 00 00 00 00 00 00 00 00

    10 | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00

    20 | 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00

    30 | 00 00 00 00 00 00 00 00 00 00 00 00 40 00 00 00
```

正如你所见到的，这不是真正的`MS-DOS`程序。它只是一个开始部分有`MZ`签名的头和紧跟在头后面的`e_lfanew`指针，没有任何代码。这是因为它并非打算运行于`MS-DOS`之上；它之所以在这里只是因为规范的需要。

然后是`PE`签名，开始于`0x40`，有`0x4`字节长：

```
        50 45 00 00
```

现在到了文件头，开始于`0x44`，有`0x14`字节长：

```
    Machine                     4c 01       ; i386

    NumberOfSections            02 00       ; 代码段和数据段

    TimeDateStamp               00 00 00 00 ; 谁管它？

    PointerToSymbolTable        00 00 00 00 ; 未用

    NumberOfSymbols             00 00 00 00 ; 未用

    SizeOfOptionalHeader        e0 00       ; 常量

    Characteristics             02 01       ; 32位机器上的可执行文件
```

接着是可选头，开始于`0x58`，有`0x60`字节长：

```
    Magic                       0b 01       ; 常量

    MajorLinkerVersion          00          ; 我是 0.0 版:-)

    MinorLinkerVersion          00          ;

    SizeOfCode                  20 00 00 00 ; 32字节代码

    SizeOfInitializedData       ?? ?? ?? ?? ; 待找出

    SizeOfUninitializedData     00 00 00 00 ; 我们没有BSS节

    AddressOfEntryPoint         ?? ?? ?? ?? ; 待定

    BaseOfCode                  ?? ?? ?? ?? ; 待定

    BaseOfData                  ?? ?? ?? ?? ; 待定

    ImageBase                   00 00 10 00 ; 1 MB, 随意选

    SectionAlignment            20 00 00 00 ; 32字节对齐

    FileAlignment               20 00 00 00 ; 32字节对齐

    MajorOperatingSystemVersion  04 00      ; NT 4.0

    MinorOperatingSystemVersion  00 00      ;

    MajorImageVersion           00 00       ;0.0版

    MinorImageVersion           00 00       ;

    MajorSubsystemVersion       04 00       ; Win32 4.0

    MinorSubsystemVersion       00 00       ;

    Win32VersionValue           00 00 00 00 ; 未使用?

    SizeOfImage                 ?? ?? ?? ?? ; 待定

    SizeOfHeaders               ?? ?? ?? ?? ; 待定

    CheckSum                    00 00 00 00 ; 非驱动不用

    Subsystem                   03 00       ; Win32控制台

    DllCharacteristics          00 00       ; 未用 (不是一个DLL)

    SizeOfStackReserve          00 00 10 00 ; 1 MB栈

    SizeOfStackCommit           00 10 00 00 ; 开始时4 KB

    SizeOfHeapReserve           00 00 10 00 ; 1 MB堆

    SizeOfHeapCommit            00 10 00 00 ; 开始时4 KB

    LoaderFlags                 00 00 00 00 ; 未知

    NumberOfRvaAndSizes         10 00 00 00 ; 常量
```

正如你所见，我计划只用2个节，一个用于代码，一个用于所有剩余的东西（数据、常量和输入目录等）。没有重定位和象资源之类其它东西。我也不用`BSS`节并将变量`written`放入已初始化数据。文件和`RAM`中的节对齐都是一样的（32字节）；这将有助于使任务简单，否则我就得来回地计算`RVA`很多次。

现在我们设置数据目录，开始于`0xb8`字节，有 `0x80`字节长:

```
       地址           大小

    00 00 00 00    00 00 00 00         ; IMAGE_DIRECTORY_ENTRY_EXPORT (0)

    ?? ?? ?? ??    ?? ?? ?? ??         ; IMAGE_DIRECTORY_ENTRY_IMPORT (1)

    00 00 00 00    00 00 00 00         ; IMAGE_DIRECTORY_ENTRY_RESOURCE (2)

    00 00 00 00    00 00 00 00         ; IMAGE_DIRECTORY_ENTRY_EXCEPTION (3)

    00 00 00 00    00 00 00 00         ; IMAGE_DIRECTORY_ENTRY_SECURITY (4)

    00 00 00 00    00 00 00 00         ; IMAGE_DIRECTORY_ENTRY_BASERELOC (5)

    00 00 00 00    00 00 00 00         ; IMAGE_DIRECTORY_ENTRY_DEBUG (6)

    00 00 00 00    00 00 00 00         ; IMAGE_DIRECTORY_ENTRY_COPYRIGHT (7)

    00 00 00 00    00 00 00 00         ; IMAGE_DIRECTORY_ENTRY_GLOBALPTR (8)

    00 00 00 00    00 00 00 00         ; IMAGE_DIRECTORY_ENTRY_TLS (9)

    00 00 00 00    00 00 00 00         ; IMAGE_DIRECTORY_ENTRY_LOAD_CONFIG (10)

    00 00 00 00    00 00 00 00         ; IMAGE_DIRECTORY_ENTRY_BOUND_IMPORT (11)

    00 00 00 00    00 00 00 00         ; IMAGE_DIRECTORY_ENTRY_IAT (12)

    00 00 00 00    00 00 00 00         ; 13

    00 00 00 00    00 00 00 00         ; 14

    00 00 00 00    00 00 00 00         ; 15
```

仅使用输入目录。

下一个使节头。首先我们做代码节的，代码节将包含前面所编的汇编语句。它有32字节长，所以代码节也就是这么长。节头从`0x138`处开始，有`0x28`字节长：

```
    Name            2e 63 6f 64 65 00 00 00     ; ".code"的ASCII码值

    VirtualSize         00 00 00 00             ; 未用

    VirtualAddress      ?? ?? ?? ??             ; 待定

    SizeOfRawData       20 00 00 00             ; 代码的大小

    PointerToRawData    ?? ?? ?? ??             ; 待定

    PointerToRelocations 00 00 00 00            ; 未用

    PointerToLinenumbers 00 00 00 00            ; 未用

    NumberOfRelocations  00 00                  ; 未用

    NumberOfLinenumbers  00 00                  ; 未用

    Characteristics     20 00 00 60             ; 代码节，可执行，可读
```

第二节将包含数据。节头开始于`0x160`处，有`0x28`字节长：

```
    Name            2e 64 61 74 61 00 00 00     ; ".data"的ASCII码值

    VirtualSize         00 00 00 00             ; 未用

    VirtualAddress      ?? ?? ?? ??             ; 待定

    SizeOfRawData       ?? ?? ?? ??             ; 待定

    PointerToRawData    ?? ?? ?? ??             ; 待定

    PointerToRelocations 00 00 00 00            ; 未用

    PointerToLinenumbers 00 00 00 00            ; 未用

    NumberOfRelocations  00 00                  ; 未用

    NumberOfLinenumbers  00 00                  ; 未用

    Characteristics     40 00 00 c0             ; 已初始化的，可读，可写
```

下一个字节位于`0x188`处，但节需要按32字节（的倍数）对齐（因为我是这样选择的），所以我们需要添一些（0）字节直到`0x1a0`处：

```
    00 00 00 00 00 00       ; 填充的

    00 00 00 00 00 00

    00 00 00 00 00 00

    00 00 00 00 00 00
```

现在第一节，就是上面所汇编的代码节，“到”了。它开始于`0x1a0`处，有`0x20`字节长：

```
    6A 00                    ; push      0x00000000

    68 ?? ?? ?? ??           ; push      offset _written

    6A 0D                    ; push      0x0000000d

    68 ?? ?? ?? ??           ; push      offset hello_string

    6A F5                    ; push      0xfffffff5

    2E FF 15 ?? ?? ?? ??     ; call      dword ptr cs:__imp__GetStdHandle@4

    50                       ; push      eax

    2E FF 15 ?? ?? ?? ??     ; call      dword ptr cs:__imp__WriteConsoleA@20

    C3                       ; ret
```

因为这一节的长度（刚好32字节），在下一节（数据节）前我们不需要填充任何字节。下一节到了，从`0x1c0`处开始：

```
    68 65 6C 6C 6F 2C 20 77 6F 72 6C 64 0A  ; "hello, world\n"的ASCII码值

    00 00 00                                ; 填充几个0以和_written对齐

    00 00 00 00                             ; _written
```

现在剩下的只有输入目录了。本文件将从`"kernel32.dll"`库中输入2个函数，输入目录将从本节的变量后面立即开始。首先我们先将上面的数据按32字节对齐：

```
    00 00 00 00 00 00 00 00 00 00 00 00     ; 填充的
```

在`0x1e0`处开始输入描述（`IMAGE_IMPORT_DESCRIPTOR`):

```
    OriginalFirstThunk      ?? ?? ?? ??     ; 待定

    TimeDateStamp           00 00 00 00     ; 未绑定

    ForwarderChain          ff ff ff ff     ; 无中转

    Name                    ?? ?? ?? ??     ; 待定

    FirstThunk              ?? ?? ?? ??     ; 待定
```

我们需要用一个0字节项来结束输入目录（我们现在位于`0x1f4`):

```
    OriginalFirstThunk      00 00 00 00     ; 结束符号

    TimeDateStamp           00 00 00 00     ;

    ForwarderChain          00 00 00 00     ;

    Name                    00 00 00 00     ;

    FirstThunk              00 00 00 00     ;
```

现在只剩下`DLL`名字，还有2个换长，以及换长数据和函数名字了。但现在我们真的很快就要完成了。

`DLL`名字，以0结尾，开始于`0x208`处:

```
    6b 65 72 6e 65 6c 33 32 2e 64 6c 6c 00  ; "kernel32.dll"的ASCII码值

    00 00 00                                ; 填充到32位边界
```

原始第一个换长，开始于`0x218`处:
