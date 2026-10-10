

# 【Anti Virus 专题】1.2 - 4.PE 结构、SEH 相关知识掌握

> 原文：[`www.pediy.com/kssd/pediy10/86452.html`](https://www.pediy.com/kssd/pediy10/86452.html)

1.2 4.PE 结构、SEH 相关知识掌握  
  
  今天这节课程中，我们来讲解一下部分 PE 结构，以及 SEH\[Structured Exception Handling(结构化异常处理)\]的知识。因为它在我们病毒编写中也是非常重要的。想象一下，如果我们连我们的被感染对象的文件格式都不熟悉的话，我们如何感染？ 那么在上几节课程中，我们在文章中已经穿插了部分 PE 结构的内容，今天我们就来再深入的熟悉一下 PE 结构以及 SEH 的相关知识，为我们后面的文章做好铺垫。  
  
  
1. 抛弃链接器, 手工定义 PE 结构程序:  
  
  当然我不喜欢很传统的给大家罗列一堆 PE 结构，然后来给大家讲解结构成员，我还是喜欢实际的的编写。所以这节课程，我们就通过编写一个自定义 PE 结构的程序来让大家充分的了解 PE 结构。（实际上在 fasm 中也就是抛弃 fasm 所有的宏，然后以二进制数据形式编译)  
  
  好了，我们了解下我们的 PE 结构组成部分。  
  
  |----------------|  
  |IMAGE\_DOS\_HEADER|  ------ IMAGE\_DOS\_HEADER  
  |----------------|  
  |     Dos STUB   |  ------ 16 位 DOS 代码  
  |                |  
  |----------------|   
  |                |  ------ Signature  
  |                |   
  |IMAGE\_NT\_HEADERS|  ------ IMAGE\_FILE\_HEADER  
  |                |   
  |                |  ------ IMAGE\_OPTIONAL\_HEADER32  
  |----------------|   
  |                |  
  | SECTION\_HEADER |  
  |                |  ------ 节表结构  
  |----------------|  
  |                |  
  |    SECTION     |  ------ 节  
  |                |  
  |----------------|  
  OK，了解了以上后，我们就来手工的定义一个 PE 文件结构的程序，我们仅让它拥有一个节。那么接下来我们来一步一步的构造整个文件结构，首先是 IMAGE\_DOS\_HEADER, 然后是 Dos STUB\[要不要都行\]，喜欢的就加上吧，...........一直到 Section。  
  
  我们在开始之前要定义几个常量，方便后续的添加。首先是我们的基地址。我们定义为   
        Image\_Base = 00400000h  
  然后是我们内存对齐值我们设置为 1000h，以及我们的节映射到内存基于基地址的偏移 我们也设置为 1000h。那么我们定义  
   Code\_Base = 1000h  
  
  好了，接下来我们就开始一步步的定义文件结构， 首先是 IMAGE\_DOS\_HEADER 结构。  
  
struct IMAGE\_DOS\_HEADER   
  e\_magic           dw      ?  
  e\_cblp            dw      ?  
  e\_cp              dw      ?  
  e\_crlc            dw      ?  
  e\_cparhdr         dw      ?  
  e\_minalloc        dw      ?  
  e\_maxalloc        dw      ?  
  e\_ss              dw      ?  
  e\_sp              dw      ?  
  e\_csum            dw      ?  
  e\_ip              dw      ?  
  e\_cs              dw      ?  
  e\_lfarlc          dw      ?  
  e\_ovno            dw      ?  
  e\_res             dw   4 dup(?)  
  e\_oemid           dw      ?  
  e\_oeminfo         dw      ?  
  e\_res2            dw  10 dup(?)  
  e\_lfanew          dd      ?  
ends  
  
  
  IMAGE\_DOS\_HEADER 结构中最重要的两个成员是 e\_magic,和 e\_lfanew。前者是两个字节的“MZ”标识符，后者是 4 个字节的指向 IMAGE\_NT\_HEADERS 结构的文件偏移。所以其余成员我们可以以 0 代替。定义如下  
  
  
DOS\_HEADER:  
.e\_magic           db    'MZ'  
.e\_cblp            dw      0  
.e\_cp              dw      0  
.e\_crlc            dw      0  
.e\_cparhdr         dw      0  
.e\_minalloc        dw      0  
.e\_maxalloc        dw      0  
.e\_ss              dw      0  
.e\_sp              dw      0  
.e\_csum            dw      0  
.e\_ip              dw      0  
.e\_cs              dw      0  
.e\_lfarlc          dw      0  
.e\_ovno            dw      0  
.e\_res             dw   4 dup(0)  
.e\_oemid           dw      0  
.e\_oeminfo         dw      0  
.e\_res2            dw  10 dup(0)  
.e\_lfanew          dd  NT\_HEADERS  
;-------------------------------------  
Dos\_Stub:  
  mov ah, 4ch  
  int 21h  
;------------------------------------  
  
NT\_HEADERS:  
  
  
  OK,我们定义完 DOS\_HEADER,我们来定义 NT\_HEADERS 结构。  
  
\[1\]:  
  
struct IMAGE\_NT\_HEADERS  
  Signature        dd  ?  
  FileHeader        IMAGE\_FILE\_HEADER  
  OptionalHeader      IMAGE\_OPTIONAL\_HEADER32  
ends  
  
  1. 首先第一个成员是 Signature4 个字节 PE 文件标志，它被定义为 00004550h。ASCII 码表示法则为 dd “PE”。  
  2. 紧接着是我们的 FileHeader 成员。  
  
\[2\]:  
  
  
struct IMAGE\_FILE\_HEADER  
  Machine      dw  ?  
  NumberOfSections      dw  ?  
  TimeDateStamp    dd  ?  
  PointerToSymbolTable  dd  ?  
  NumberOfSymbols    dd  ?  
  SizeOfOptionalHeader  dw  ?  
  Characteristics    dw  ?  
ends  
  
    
  (1): 第一个 2 字节成员 Machine,表示运行平台。我们一般设置为 14ch,表示 Intel I386。  
  (2): 第二个 2 字节成员 NumberOfSections,表示我们的节数量，这个程序中我们设置一个 Section，所以我们填写为 1。  
  (3): 3 - 5 , 4 字节成员因为没有什么用，所以我们设置为 0.TimeDateStamp(文件创建时间) ，PointerToSymbolTable(COFF 符号表的文件偏移量)，NumberOfSymbols(符号表符号数量)。  
  (4): SizeOfOptionalHeader 成员表示的是下面 IMAGE\_OPTIONAL\_HEADER32 结构的字节大小。我们用默认的 IMAGE\_OPTIONAL\_HEADER32 字节大小，0E0h。  
  (5): Characteristics 成员表示的是文件的属性。我们普通的 EXE 文件一般是 010fh。  
      
  
\[3:\]  
  接下来是我们的 OptionalHeader 结构成员。  
  
struct IMAGE\_OPTIONAL\_HEADER32  
  Magic        dw  ?  
  MajorLinkerVersion    db  ?  
  MinorLinkerVersion    db  ?  
  SizeOfCode      dd  ?  
  SizeOfInitializedData    dd  ?  
  SizeOfUninitializedData    dd  ?  
  AddressOfEntryPoint    dd  ?  
  BaseOfCode      dd  ?  
  BaseOfData      dd  ?  
  ImageBase      dd  ?  
  SectionAlignment    dd  ?  
  FileAlignment      dd  ?  
  MajorOperatingSystemVersion  dw  ?  
  MinorOperatingSystemVersion  dw  ?  
  MajorImageVersion               dw       ?  
  MinorImageVersion               dw       ?  
  MajorSubsystemVersion           dw       ?  
  MinorSubsystemVersion           dw       ?  
  Win32VersionValue               dd      ?  
  SizeOfImage                     dd      ?  
  SizeOfHeaders                   dd      ?  
  CheckSum                        dd      ?  
  Subsystem                       dw       ?  
  DllCharacteristics              dw       ?  
  SizeOfStackReserve              dd      ?  
  SizeOfStackCommit               dd      ?  
  SizeOfHeapReserve               dd      ?  
  SizeOfHeapCommit                dd      ?  
  LoaderFlags                     dd      ?  
  NumberOfRvaAndSizes             dd      ?  
  DataDirectory      IMAGE\_DATA\_DIRECTORY  
  rb       sizeof.IMAGE\_DATA\_DIRECTORY\*15  
ends  
  
  
  
  (1): 首先第一个成员.Magic 是一个标记，一般我们普通的可执行文件，这个标记值为 010Bh。  
  (2): 下面两个 1 字节 MajorLinkerVersion 一般为 0.它表示为我们的链接程序的版本号。  
  (3): SizeOfCode 表示代码段的大小。这个大小值是代码段对齐后的值。这里我们设置的文件对齐值是 200h，所以我们这里填写为 200h。（也就是 512byte)。  
  (4): SizeOfInitializedData 表示初始化数据的字节大小，不包括代码段。我们这里填写为 0。  
  (5): SizeOfUninitializedData 表示未初始化数据的字节大小。即我们常说的.bbs 节，这里我们没用。所以填写 0。  
  (6): AddressOfEntryPoint 成员指向我们程序的 OEP，这个值是一个 RVA 地址，相对于我们基地址的偏移。这里我们把这个值设置为我们定义的 Code 节的偏移。由于我们 Code 段节我们设置它映射内存的虚拟地址是 1000h,所以我们需要通过这个 1000h - Code 段文件偏移  = 相对于 Code 段的偏移。 通过这个相对偏移 + 文件偏移 = RVA 地址。我们把这个相对偏移定义为 Rva\_Del = Code\_Base - @Code。  
  (7): BaseOfCode 表示代码段起始 RVA 地址。我们直接用 Code\_Base 即可。  
  (8): BaseOfData 表示数据段的起始 RVA 地址。我们可以设置为 0.  
  (9): ImageBase 表示我们的基地址。我们这里设置我们上面定义好的 Image\_Base。  
  (10): MajorOperatingSystemVersion 和 MinorOperatingSystemVersion 表示的是操作系统的主版本号，和次版本好，我们一般用不上。所以这两个成员可以设置为 0.  
  (11): MajorImageVersion, MinorImageVersion 表示用户自定义主、次版本号。对我们来说没用，设置为 0.  
  (12): MajorSubsystemVersion 和 MinorSubsystemVersion 表示最低子系统版本号，我们通常设置 MajorSubsystemVersion 为 4。  
  (13): Win32VersionValue 通常是保留值，一般为 0.  
  (14): SizeOfImage 表示映像文件载入内存后的总大小。我们这里一个节 + 我们的 PE 头部。我们设置为 2000h  
  (15): SizeOfHeaders 表示 size(DosHeader + DosStub + NtHeader + Section Header)  
  (16): CheckSum 效验和值。一般 EXE 文件为 0.  
  (17): Subsystem 表示子系统值。 2 GUI, 3 CUI。  
  (18): DllCharacteristics 指定 DLLMain 函数何时被调用。默认 0.  
  (19): SizeOfStackReserve 在 exe 文件中默认的线程堆栈保留大小。这里我们设置为 10000h,因为它在需要时才被提交。  
  (20): SizeOfStackCommit 表示开始委派的堆栈大小。默认是 4KB。  
  (21): SizeOfHeapReserve 表示在 exe 文件中默认的堆空间保留大小，同样我们设置为 10000h。  
  (22): SizeOfHeapCommit 表示开始委派的堆空间大小。默认是 4KB。  
  (23): LoaderFlags 默认为 0。  
  (24): NumberOfRvaAndSizes 表示下面数据目录段的成员数量。10h   
  (25): DataDirectory 数据目录表。表中成员为 16 个 IMAGE\_DATA\_DIRECTORY 结构的成员。我们不需要任何结构，这里全部填 0.  
  
\[4:\]  
  接下来是节表结构。  
struct IMAGE\_SECTION\_HEADER  
    Name1 rb 8  
    union  
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
ends  
  
  (1):首先第一个 8 字节成员表示节名称。  
  (2):VirtualSize 表示节映射内存的大小。我们这里填写为 1000h。  
  (3):VirtualAddress 表示节空间映射到内存基于基地址的偏移。这里我们填写我们之前定义好的 Code\_Base。  
  (4):SizeOfRawData 表示节在磁盘空间中的大小。我们通过定义节代码指令的最后偏移 - 节起始偏移即可求出。  
  (5): PointerToRawData 表示节所在文件中的物理文件偏移。我们直接用我们定义的@Code 标号即可。  
  (6): PointerToRelocations PointerToLinenumbers NumberOfRelocations 没什么用处直接填充 0.  
  (7): Characteristics 表示节属性，这个属性很重要，它决定了节空间映射到内存后的内存空间属性。我们这里用 E0000020（可读可写可执行)  
  
  
  
  
  OK.到这里我们的结构就完成了，然后就是在我们的代码段偏移中写代码了。我们仅写一个 ret 指令。  
  
OK。我们来看下我们最后的程序代码。  
  

代码:

```
;-----------------------------------------
; Format Pe File  By:XFish
;------------------------------------------
 Code\_Base   = 1000h
 File\_Algin  = 200h
 Rva\_Del     = Code\_Base - @Code
 Image\_Base  = 00400000h
;\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_
DOS\_HEADER:
.e\_magic    db  'MZ'
.e\_cblp           dw      0
.e\_cp             dw      0
.e\_crlc           dw      0
.e\_cparhdr        dw      0
.e\_minalloc       dw      0
.e\_maxalloc       dw      0
.e\_ss             dw      0
.e\_sp             dw      0
.e\_csum           dw      0
.e\_ip             dw      0
.e\_cs             dw      0
.e\_lfarlc           dw      0
.e\_ovno           dw      0
.e\_res            dw   4 dup(0)
.e\_oemid          dw      0
.e\_oeminfo        dw      0
.e\_res2           dw  10 dup(0)
.e\_lfanew         dd  NT\_HEADERS



Dos\_Stub:
  mov ah, 4ch
  int 21h
  
  

NT\_HEADERS:
.Signature    dd  'PE'
;++          ;IMAGE\_FILE\_HEADER
.Machine    dw  14ch     ;.Inten I386
.NumberOfSections   dw  1
.TimeDateStamp    dd  0
.PointerToSymbolTable  dd  0
.NumberOfSymbols  dd  0  
.SizeOfOptionalHeader  dw  0E0h
.Characteristics  dw  010fh    ;.exe Signature
;--
;++          ;IMAGE\_OPTIONAL\_HEADER32
.Magic      dw  10Bh   ;.exe file
.MajorLinkerVersion  db  0
.MinorLinkerVersion  db  0
.SizeOfCode    dd  File\_Algin
.SizeOfInitializedData  dd  0
.SizeOfUninitializedData dd  0
.AddressOfEntryPoint  dd  @Code + Rva\_Del ;oep
.BaseOfCode    dd  Code\_Base  ;Code Section Rva
.BaseOfData     dd  0    ;Data Section Rva
.ImageBase    dd  Image\_Base
.SectionAlignment  dd  Code\_Base  ;Section Mem Align
.FileAlignment    dd  File\_Algin  ;Section Disk Align
.MajorOperSystemVersion dw  0    ;system version 主
.MinorOperSystemVersion dw  0    ;system version 次
.MajorImageVersion  dw  0    ;user version   主
.MinorImageVersion  dw  0    ;user version   次
.MajorSubsystemVersion  dw  4
.MinorSubsystemVersion  dw  0
.Win32VersionValue  dd  0    ;Reserved 0
.SizeOfImage    dd  2000h
.SizeOfHeaders    dd  File\_Algin  ;DosHeader + DosStub + NtHeader + Section Header
.CheckSum    dd  0
.SubSystem    dw  2    ;Gui
.DllCharacteristics  dw  0
.SizeOfStackReserve  dd  10000h    
.SizeOfStackCommit  dd  1000h    ;Stack = 4kb
.SizeOfHeapReserve  dd  10000h
.SizeOfHeapCommit  dd  1000h    ;Heap = 4kb
.LoaderFlags    dd  0
.NumberOfRvaAndSizes  dd  10h    ;16
.Export                 rd   2
.Import      rd  2
.Misc\_Section    rd  28
  

\_SECTION\_HEADER:
.Name1      db  '.code', 0, 0, 0
.VirtualSize    dd  1000h
.VirtualAddress    dd  Code\_Base
.SizeOfRawData    dd  @Code\_End - @Code
.PointerToRawData  dd  @Code
.PointerToRelocations   dd   0
.PointerToLinenumbers   dd   0
.NumberOfRelocations    dw   0
.NumberOfLinenumbers    dw   0
.Characteristics  dd  0E0000020h

align  200h
;------------------------------------

@Code:

  ret
  

@Code\_End:

```

  这里留个问题给大家想想，此代码编译完后程序体积是多少字节呢？  
  
  好了，通过以上的编码，相信你对 PE 结构已经很熟悉了。那么我们继续往下吧，我们来学习 SEH 技术的利用。  
  
  
(2): SEH 技术的利用。  
  
  关于 seh 技术，很久之前 Hume 就总结过了，所以我在这里不想过多总结了。大家看可以在这个地址中看下 Hume 的文章。  
    http://www.pediy.com/bbshtml/bbs4/kanxue310.htm  
  
  我要讲解下关于 seh 技术的利用，例如如何在我们的汇编自程序当中利用技巧，方便的排错。  
  
  
  传统的 SEH 的框架我想我就不多说了。  
  如下:  
  push    SEH\_Handler  
  push    dword ptr fs:\[0\]  
  mov  \[fs:0\], esp  
\`    
大家看如下代码:  
  
代码:

```
        call  .SetupSEH  
.ErrorHandler:
  mov  esp, \[esp+8\]  ;\[esp+8\],指向 lpseh。lpseh 是我们 EXCEPTION\_REGISTRATION 结构的地址。
  xor  eax, eax
  pop  dword \[fs:eax\]  ;取出 EXCEPTION\_REGISTRATION->prev ;前一个\_EXCEPTION\_REGISTRATION 结构
  pop  ecx    ;修正堆栈
  ret      ;ExitThread
  
.SetupSEH:
  xor  edx, edx
  push  dword \[fs:edx\]
  mov  \[fs:edx\], esp
  int  3

```

  这段代码是如果异常发生，则我们的异常过程将 esp 堆栈指针修复，然后恢复\[fs:0\]的成员。然后修正堆栈，退出线程。我们平时封装子程序的时候或者是写线程处理过程的时候完全可以利用以上代码方便的封装。  
  
  OK，写一段我们自定义 PE 文件结构的异常处理过程代码。  
代码:

```

;-----------------------------------------
; Format Pe File  By:XFish
;------------------------------------------
 include 'win32a.inc'
 Code\_Base   = 1000h
 File\_Algin  = 200h
 Rva\_Del     = Code\_Base - @Code
 Image\_Base  = 00400000h
;\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_
DOS\_HEADER:
.e\_magic    db  'MZ'
.e\_cblp           dw      0
.e\_cp             dw      0
.e\_crlc           dw      0
.e\_cparhdr        dw      0
.e\_minalloc       dw      0
.e\_maxalloc       dw      0
.e\_ss             dw      0
.e\_sp             dw      0
.e\_csum           dw      0
.e\_ip             dw      0
.e\_cs             dw      0
.e\_lfarlc           dw      0
.e\_ovno           dw      0
.e\_res            dw   4 dup(0)
.e\_oemid          dw      0
.e\_oeminfo        dw      0
.e\_res2           dw  10 dup(0)
.e\_lfanew         dd  NT\_HEADERS



Dos\_Stub:
  mov ah, 4ch
  int 21h
  
  

NT\_HEADERS:
.Signature    dd  'PE'
;++          ;IMAGE\_FILE\_HEADER
.Machine    dw  14ch     ;.Inten I386
.NumberOfSections   dw  1
.TimeDateStamp    dd  0
.PointerToSymbolTable  dd  0
.NumberOfSymbols  dd  0  
.SizeOfOptionalHeader  dw  0E0h
.Characteristics  dw  010fh    ;.exe Signature
;--
;++          ;IMAGE\_OPTIONAL\_HEADER32
.Magic      dw  10Bh   ;.exe file
.MajorLinkerVersion  db  0
.MinorLinkerVersion  db  0
.SizeOfCode    dd  File\_Algin
.SizeOfInitializedData  dd  0
.SizeOfUninitializedData dd  0
.AddressOfEntryPoint  dd  @Code + Rva\_Del ;oep
.BaseOfCode    dd  Code\_Base  ;Code Section Rva
.BaseOfData     dd  0    ;Data Section Rva
.ImageBase    dd  Image\_Base
.SectionAlignment  dd  Code\_Base  ;Section Mem Align
.FileAlignment    dd  File\_Algin  ;Section Disk Align
.MajorOperSystemVersion dw  0    ;system version 主
.MinorOperSystemVersion dw  0    ;system version 次
.MajorImageVersion  dw  0    ;user version   主
.MinorImageVersion  dw  0    ;user version   次
.MajorSubsystemVersion  dw  4
.MinorSubsystemVersion  dw  0
.Win32VersionValue  dd  0    ;Reserved 0
.SizeOfImage    dd  @Code\_End - @Code + 1000h
.SizeOfHeaders    dd  File\_Algin  ;DosHeader + DosStub + NtHeader + Section Header
.CheckSum    dd  0
.SubSystem    dw  2    ;Gui
.DllCharacteristics  dw  0
.SizeOfStackReserve  dd  10000h    
.SizeOfStackCommit  dd  1000h    ;Stack = 4kb
.SizeOfHeapReserve  dd  10000h
.SizeOfHeapCommit  dd  1000h    ;Heap = 4kb
.LoaderFlags    dd  0
.NumberOfRvaAndSizes  dd  10h    ;16
.Export                 rd   2
.Import      rd  2
.Misc\_Section    rd  28
  

\_SECTION\_HEADER:
.Name1      db  '.code', 0, 0, 0
.VirtualSize    dd  1000h
.VirtualAddress    dd  Code\_Base
.SizeOfRawData    dd  @Code\_End - @Code
.PointerToRawData  dd  @Code
.PointerToRelocations   dd   0
.PointerToLinenumbers   dd   0
.NumberOfRelocations    dw   0
.NumberOfLinenumbers    dw   0
.Characteristics  dd  0E0000020h

align  200h
;------------------------------------

@Code:
  db  0E8h    ;call opcode
  dd  SetupSEH - ErrorHandler  
ErrorHandler:
  db  8bh
  db  64h
  db  24h
  db  08h     ;mov esp, \[esp+8\]
  db  31h
  db  0c0h    ;xor eax, eax
  db  64h
  db  8fh
  db  00    ;pop dword \[fs:0\]
  db  59h    ;pop ecx
  ret
  
SetupSEH:
  db  31h
  db  0d2h    ;xor edx, edx
  db  64h    
  db  0ffh
  db  32h    ;push dword \[fs:0\]
  db  64h
  db  89h
  db  22h    ;mov dword \[fs:0\], esp
  db  0cdh    
  db  03h    ;int 3
  ret
  
@Code\_End:

```

  OK,今天的这篇文章就到这里了。我们下篇文章再见。以下是随文的程序和代码。。

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || bin.rar  
 |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || seh.rar  
 |