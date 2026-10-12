
# 一次 PE 文件之旅

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-29.htm`](https://www.pediy.com/kssd/pediy09/pediy09-29.htm)

最近老是盯着 PE 文件看，看它有什么秘密在里面，于是就产生了这样一个想法：构造一个 PE 头，构造自己的区段表，接着在`.text`中填一些机器码，当然还要构造自己的输入表，我想通过这种方式来实现一个程序。下面就是我的操作过程：

```
(-),首先构造一个 PE 头

-> DOS Header

  e_magic:     0x5A4D
  e_cblp:      0x0000
  e_cp:        0x0000
  e_crlc:      0x0000
  e_cparhdr:   0x0000
  e_minalloc:  0x0000
  e_maxalloc:  0x0000
  e_ss:        0x0000
  e_sp:        0x0000
  e_csum:      0x0000
  e_ip:        0x0000
  e_cs:        0x0000
  e_lfarlc:    0x0000
  e_ovno:      0x0000
  e_res:       0x0000000000000000
  e_oemid:     0x0000
  e_oeminfo:   0x0000
  e_res2:      0x0000000000000000000000000000000000000000
  e_lfanew:    0x00000040

-> File Header

  Machine:             0x014C  (I386)
  NumberOfSections:    0x0001
  TimeDateStamp:       0x45B9CD97  (GMT: Fri Jan 26 09:44:55 2007)
  PointerToSymbolTable: 0x00000000
  NumberOfSymbols:     0x00000000
  SizeOfOptionalHeader: 0x00E0
  Characteristics:     0x010F
                      (RELOCS_STRIPPED)
                      (EXECUTABLE_IMAGE)
                      (LINE_NUMS_STRIPPED)
                      (LOCAL_SYMS_STRIPPED)
                      (32BIT_MACHINE)

-> Optional Header

  Magic:                      0x010B  (HDR32_MAGIC)     ；书上说一般都是这个值
  MajorLinkerVersion:         0x05
  MinorLinkerVersion:         0x0C  -> 5.12             ；好多程序都是这个值
  SizeOfCode:                 0x00000200                ；通常情况下和.text 块的大小匹配
  SizeOfInitializedData:      0x00000200
  SizeOfUninitializedData:    0x00000000
  AddressOfEntryPoint:        0x00001000
  BaseOfCode:                 0x00001000
  BaseOfData:                 0x00002000
  ImageBase:                  0x00400000
  SectionAlignment:           0x00001000               ；这个值是最小的了
  FileAlignment:              0x00000200               ；这个也是
  MajorOperatingSystemVersion: 0x0004
  MinorOperatingSystemVersion: 0x0000  -> 4.00
  MajorImageVersion:          0x0000
  MinorImageVersion:          0x0000  -> 0.00
  MajorSubsystemVersion:      0x0004
  MinorSubsystemVersion:      0x0000  -> 4.00
  Win32VersionValue:          0x00000000               ；Reserved，以上都是常见的值
  SizeOfImage:                0x00003000               //内存镜象大小
  SizeOfHeaders:              0x00000200        ；从文件的开头到第一节的原始数据的偏移量
  CheckSum:                   0x00000000
  Subsystem:                  0x0002  (WINDOWS_GUI)
  DllCharacteristics:         0x0000
  SizeOfStackReserve:         0x00100000               ；以下的数据都是程序中常见的
  SizeOfStackCommit:          0x00001000
  SizeOfHeapReserve:          0x00100000
  SizeOfHeapCommit:           0x00001000
  LoaderFlags:                0x00000000
  NumberOfRvaAndSizes:        0x0000000F

  DataDirectory (F)         RVA         Size
  -------------             ---------- ----------
  ExportTable               0x00000000 0x00000000
  ImportTable               0x00002000 0x0000003C
  Resource                  0x00000000 0x00000000
  Exception                 0x00000000 0x00000000
  Security                  0x00000000 0x00000000
  Relocation                0x00000000 0x00000000
  Debug                     0x00000000 0x00000000
  Copyright                 0x00000000 0x00000000
  GlobalPtr                 0x00000000 0x00000000
  TLSTable                  0x00000000 0x00000000
  LoadConfig                0x00000000 0x00000000
  BoundImport               0x00000000 0x00000000
  IAT                       0x00000000 0x00000000
  DelayImport               0x00000000 0x00000000
  COM                       0x00000000 0x00000000
  Reserved                  0x00000000 0x00000000
```

## （二）区段表

下面到块表了(剩下那些字段全部填 0 了)。

```
Name    VSize    VAddress    RSize    Roffset    characterstics
.TEXT   526      1000        200      200        60000020
.RDATA  592      2000       200       400        E0000040
```

接着就是我自己的代码了，当然是最简单的！我把数据也放到`.RDATA`区段。这些代码很多我是借住 OD 查看得来的(先写代码，在 OD 打开)。

```
'pediy'       5B0------004021B0
'http://pediy.com'       5C1------004021C1

PUSH 0                                 6A00
push  'http://pediy.com'              68 C1214000   //指向 http://pediy.com
PUSH  'pediy'                          68 B0214000
push  0         6A00
call  MessageBoxA                     E8 07000000   //也就是向前跳 7 个字节吧
push  0                               6A00
call  ExitProcess                     E8 06000000
jmp   MessageBoxA                     FF25 B0204000  //跳到 IMAGE_THUNK_DATA
jmp   ExitProcess                     FF25 B8204000
```

## （三）导入表

程序中引入了两个函数，这就得为它构造两个 IID。

```
MessageBoxA    44c--->204c----0040204c
ExitProcess    466--->2066--->00402066

User32.dll     45A--->205A--->0040205A
Kelnel32.dll   474--->2074--->00402074

IMAGE_THUNK_DATA (把它放在了地址 4B0，4B8 处)
B0200000 00000000   以 0 结束
B8200000 00000000

IMAGE_IMPORT_DIRECTORY
OrignFirstThunk      TimeDateStamp    ForWarderChain     Name1      FirstThunk
00000000    00000000  00000000    5a200000    B0200000
00000000    00000000  00000000    74200000    B8200000
```

## （四）保存与测试

将这些数据保存为`pediy.exe`，结果见附件，都是我自己构造的数据，你可以用二进制编辑工具打开。再次感谢 `dummy`。

`SizeOfImage` --> `3000`

这样就可以加载了。

对了，你的程序使用 thunk 有问题，对照改一下就行。楼上的厉害，一眼就看出来了．．．程序运行出错！用 OD 载入成这样了：

```
00401000 > $  6A 00          push    0
00401002  .  68 B0214000    push    004021B0            ;  ASCII "http://pediy.com"
00401007  .  68 C1214000    push    004021C1            ;  ASCII "pediy"
0040100C  .  6A 00          push    0
0040100E  .  E8 07000000    call    0040101A
00401013  .  6A 00          push    0
00401015  .  E8 06000000    call    00401020
0040101A  $- FF25 4C204000  jmp     dword ptr [40204C]           //这里出错
00401020  $- FF25 66204000  jmp     dword ptr [402066]
```

`//这里出错！`

应该是这里错！

```
jmp   MessageBoxA                      FF25 4C204000  //跳到 IMAGE_THUNK_DATA
jmp   ExitProcess                      FF25 66204000
```

原来是 THUNK 的 RVA 搞错了，改成：

```
jmp   MessageBoxA                      FF25 Ｂ０204000
jmp   ExitProcess                      FF25 Ｂ８204000
```

，程序终于可以正常运行了！感谢 `dummy`！！，还有：

```
PUSH  'pediy'                          68 B0214000
push  'http://pediy.com'              68 C1214000   //指向 http://pediy.com
```

push 反了，和我的本意相反，改过来就行了，呵呵！

![](img/768c8e05987a6949d1bb5f7378212ee0.jpg "狂笑")

`4dd466d0e1e0cc123afbe504da2462ca`
