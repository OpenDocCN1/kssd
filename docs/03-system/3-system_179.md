
# PE 学习

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-662.htm`](https://www.pediy.com/kssd/pediy08/pediy8-662.htm)

Luevelsmeyer 的 `pe.txt` 总算是看完了，感谢 `ah007` 的翻译。

最后译者说道精华在于附录中,我略作修改重建一个文件，但不管怎么调，都无法运行。

一开始还以为是什么地方没有写对，可运行附的 `hello.exe` 也是报错。

看来用 `0x20` 对齐在 `XP` 下是不行的。这篇附录中就是太老了...

我来说说我改写的过程吧。

### 1. 先写一段简单的代码

```
;MASMPlus 代码模板 - 普通的 Windows 程序代码

.386

.Model Flat, StdCall

Option Casemap :None

Include user32.inc

Include kernel32.inc

IncludeLib user32.lib

IncludeLib kernel32.lib

.data

MsgBoxText db "Hello.",0

MsgCaption db "Test",0

.CODE

START:

  invoke MessageBox, NULL,addr MsgBoxText, addr MsgCaption, MB_OK

  invoke ExitProcess,NULL

END START
```

顺便提一下，我用的是 `MASMPlus`，可以在 `http://www.aogosoft.com/` 下载。

### 2. 编译完之后用 OD 看一看

```
00401000 >/$  6A 00       push    0                ; /Style = MB_OK|MB_APPLMODAL
00401002  |.  68 07304000    push    00403007           ; |Title = "Test"
00401007  |.  68 00304000    push    00403000           ; |Text = "Hello."
0040100C  |.  6A 00       push    0                ; |hOwner = NULL
0040100E  |.  E8 07000000    call    <jmp.&user32.MessageBoxA>     ; \MessageBoxA
00401013  |.  6A 00       push    0                ; /ExitCode = 0
00401015  \.  E8 06000000    call    <jmp.&kernel32.ExitProcess>     ; \ExitProcess
0040101A   $- FF25 08204000  jmp     dword ptr [<&user32.MessageBoxA>>  ;  user32.MessageBoxA
00401020   .- FF25 00204000  jmp     dword ptr [<&kernel32.ExitProces>  ;  kernel32.ExitProcess
```

不错，以后代码照这个抄就可以了。

发现用到了两个 `DLL` 的函数，稍微复杂了一点。

再看看原文中的代码：

```
    6A 00          ; push       0x00000000
    68 d0 01 10 00     ; push       offset _written
    6A 0D          ; push       0x0000000d
    68 c0 01 10 00     ; push       offset hello_string
    6A F5          ; push       0xfffffff5
    2E FF 15 28 02 10 00  ; call       dword ptr cs:__imp___GetStdHandle@4
    50           ; push       eax
    2E FF 15 24 02 10 00  ; call       dword ptr cs:__imp___WriteConsoleA@20
    C3           ; ret
```

注意两个 `call` 对应的机器码：`E8` 和 `2E` 的不同，看来 `E8` 后面跟的是一个偏移量。

```
0040101A   $- FF25 08204000  jmp     dword ptr [<&user32.MessageBoxA>>  ;  user32.MessageBoxA
```

这条命令算是跳板的跳板。不想这样写，还是用 `2E FF 15`。

估算一下，代码长度在 `0x20` 之内。

### 3. 建一个 Hello.exe，用 WinHex 来改

#### a） 先大约有个思路

- 文件对齐 `0x00 00 02 00`
- 区段对齐 `0x00 00 10 00`
- `ImageBase` `0x00 40 00 00`
- 只要两个段，一个代码段(Raw：`0x200` / Rva：`0x1000`)，把上面的代码放进去就可以了
- 一个数据段(Raw：`0x400` / Rva：`0x2000`)，一上来先放两个字符串，然后是输入目录
- 这样看来最多 `0x600` 个字节就可以搞定了，打开 `WinHex`，加这么多的 `0`

然后就可以开始改了。

#### b） Dos Stub 这部分很简单，只要有 MZ，然后设一下 e_lfanew 就可以了。

```
00000000   4D 5A 00 00 00 00 00 00  00 00 00 00 00 00 00 00   MZ..............
00000010   00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   ................
00000020   00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   ................
00000030   00 00 00 00 00 00 00 00  00 00 00 00 40 00 00 00   ............@...
```

#### c） NT Header 部分这样看肯定一头雾水...

```
00000040   50 45 00 00 4C 01 02 00  00 00 00 00 00 00 00 00   PE..L...........
00000050   00 00 00 00 E0 00 02 01  0B 01 00 00 20 00 00 00   ....?...... ...
00000060   D0 00 00 00 00 00 00 00  00 10 00 00 00 10 00 00   ?..............
00000070   00 20 00 00 00 00 40 00  00 10 00 00 00 02 00 00   . ....@.........
00000080   04 00 00 00 00 00 00 00  04 00 00 00 00 00 00 00   ................
00000090   00 30 00 00 00 02 00 00  00 00 00 00 02 00 00 00   .0..............
000000A0   00 00 10 00 00 10 00 00  00 00 10 00 00 10 00 00   ................
000000B0   00 00 00 00 10 00 00 00  00 00 00 00 00 00 00 00   ................
000000C0   20 20 00 00 B0 00 00 00  00 00 00 00 00 00 00 00     ..?..........
000000D0   00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   ................
000000E0   00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   ................
000000F0   00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   ................
00000100   00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   ................
00000110   00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   ................
00000120   00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   ................
00000130   00 00 00 00 00 00 00 00  2E 63 6F 64 65 00 00 00   .........code...
00000140   00 00 00 00 00 10 00 00  20 00 00 00 00 02 00 00   ........ .......
00000150   00 00 00 00 00 00 00 00  00 00 00 00 20 00 00 60   ............ ..\`
00000160   2E 64 61 74 61 00 00 00  00 00 00 00 00 20 00 00   .data........ ..
00000170   C0 00 00 00 00 04 00 00  00 00 00 00 00 00 00 00   ?..............
00000180   00 00 00 00 40 00 00 C0                               ....@..
```

具体说一下：

`Signature` `0x4550` ；直接在右边输入 `PE` 就可以了

文件头部分：

- `Machine` `0x14c` ；`Intel 386`
- `NumberOfSections` `0x2` ；2 个区段
- `TimeDateStamp` `0x0`
- `PointerToSymbolTable` `0x0`
- `NumberOfSymbols` `0x0`
- `SizeOfOptionalHeader` `0xe0` ；可选头大小，永远是 `0xe0`
- `Characteristics` `0x102` ；具体看 `pe` 说明

----------------------------------------

可选头部分：

- `Magic` `0x10b` ；32-位
- `MajorLinkerVersion` `0x0`
- `MinorLinkerVersion` `0x0`
- `SizeOfCode` `0x20` ；随便写 `0x0~0xFF FF FF FF` 都可以
- `SizeOfInitializedData` `0xd0` ；随便写 `0x0~0xFF FF FF FF` 都可以
- `SizeOfUninitializedData` `0x0`
- `AddressOfEntryPoint` `0x1000` ；入口，这里就是代码段开头处的 `Rva`
- `BaseOfCode` `0x1000` ；代码段开头处的 `Rva`
- `BaseOfData` `0x2000` ；数据段开头处的 `Rva`
- `ImageBase` `0x400000` ；写代码是要注意把 `Base` 加上
- `SectionAlignment` `0x1000` ；区段对齐
- `FileAlignment` `0x200` ；文件对齐
- `MajorOperatingSystemVersion` `0x4` ；`NT 4.0`
- `MinorOperatingSystemVersion` `0x0`
- `MajorImageVersion` `0x0`
- `MinorImageVersion` `0x0`
- `MajorSubsystemVersion` `0x4` ；`Win32 4.0`
- `MinorSubsystemVersion` `0x0`
- `Reserved1` `0x0`
- `SizeOfImage` `0x3000` ；请看下面的话
- `SizeOfHeaders` `0x200` ；文件中第一个区段的偏移量

```
/*
    这里插段话，在原文的附录中，关于这两个字段是这样描述的。

    SizeOfImage                 c0 00 00 00 ; sum of all section sizes
    SizeOfHeaders               a0 01 00 00 ; offset to 1st section

    这个描述很不好，首先 all section 是包括 Headers 这个 Section 的
    但下面的 1st section，显然指的是 Headers 后的第一个区段。
    这两个 section 含义不一致。
    我就被误导了，一开始这里填了 0x2000，就没有通过。
    其实原文前面的解释比较清楚：
    ...它就是所有头和节的长度的总和。
*/
```

- `CheckSum` `0x0`
- `Subsystem` `0x2` ；`Win GUI`
- `DllCharacteristics` `0x0`
- `SizeOfStackReserve` `0x100000` ；就这样写吧
- `SizeOfStackCommit` `0x1000`
- `SizeOfHeapReserve` `0x100000`
- `SizeOfHeapCommit` `0x1000`
- `LoaderFlags` `0x0`
- `NumberOfRvaAndSizes` `0x10` ；总是这个值，下面有多少个目录

----------------------------------------

数据目录部分

`Directory 0`
- `Size` `0x0`
- `VirtualAddress` `0x0`

----------------------------------------

`Directory 1`
- `Size` `0xb0` ；随便写 `0x0~0xFF FF FF FF` 都可以
- `VirtualAddress` `0x2020` ；两个字符串 `0x20` 总够了吧

----------------------------------------

- 还有 14 个，全是 0

----------------------------------------

区段头

`Section 0`
- `Name` `.code` ；区段的名字
- `VirtualAddress` `0x1000` ；内存中的位置
- `SizeOfRawData` `0x20` ；随便写 `0x1~0x10 00` 都可以
- `PointerToRawData` `0x200` ；文件中的位置
- `PointerToRelocations` `0x0`
- `PointerToLinenumbers` `0x0`
- `NumberOfRelocations` `0x0`
- `NumberOfLinenumbers` `0x0`
- `Characteristics` `0x6000002` ；具体看 `pe` 说明

----------------------------------------

`Section 1`
- `Name` `.data` ；区段的名字
- `VirtualAddress` `0x2000` ；内存中的位置
- `SizeOfRawData` `0xc0` ；写大一点就可以了
- `PointerToRawData` `0x400` ；文件中的位置
- `PointerToRelocations` `0x0`
- `PointerToLinenumbers` `0x0`
- `NumberOfRelocations` `0x0`
- `NumberOfLinenumbers` `0x0`
- `Characteristics` `0xc000004` ；具体看 `pe` 说明

----------------------------------------



关于“随便写”的地方，都是 `size`。  

看来 `.code` 段无所谓 `size`，别超过段大小就行。  

`.data` 段就不行，不能少，但可以多，能多多少就要看文件的大小。  

`SizeOfCode`    0x20    ；随便写 0x0~0xFF FF FF FF 都可以  

`SizeOfInitializedData`    0xd0    ；随便写 0x0~0xFF FF FF FF 都可以  

`Directory 1`  

`Size`    0xb0    ；随便写 0x0~0xFF FF FF FF 都可以  

这 3 个有点夸张，但可以正常运行。  

c）现在不妨跳过 `.code` 段，先搞 `.data` 段  

```
00000400   50 45 D1 A7 CF B0 00 00  BF B4 B5 BD C4 E3 C1 CB   PE 学习..看到你了  

00000410   00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   ................  

00000420   5C 20 00 00 00 00 00 00  FF FF FF FF 7C 20 00 00   \ ......   | ..  

00000430   64 20 00 00 6C 20 00 00  00 00 00 00 FF FF FF FF   d ..l ......  

00000440   8C 20 00 00 74 20 00 00  00 00 00 00 00 00 00 00   ?..t ..........  

00000450   00 00 00 00 00 00 00 00  00 00 00 00 AC 20 00 00   ............?..  

00000460   00 00 00 00 AC 20 00 00  00 00 00 00 BC 20 00 00   ....?......?..  

00000470   00 00 00 00 BC 20 00 00  00 00 00 00 75 73 65 72   ....?......user  

00000480   33 32 2E 64 6C 6C 00 00  00 00 00 00 6B 65 72 6E   32.dll......kern  

00000490   65 6C 33 32 2E 64 6C 6C  00 00 00 00 00 00 00 00   el32.dll........  

000004A0   00 00 00 00 00 00 00 00  00 00 00 00 01 00 4D 65   ..............Me  

000004B0   73 73 61 67 65 42 6F 78  41 00 00 00 02 00 45 78   ssageBoxA.....Ex  

000004C0   69 74 50 72 6F 63 65 73  73 00 00 00 00 00 00 00   itProcess.......  

000004D0   00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   ................  

000004E0   00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   ................  

000004F0   00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   ................  
```

上来两个字符串，注意最后加 `'\\0'`。对应的 `Rva` 是 `0x00 40 20 00` 和 `0x00 40 20 08`  

从 `0x420` 开始是输入目录  

因为有 2 个 DLL，所以先留 3*0x14=0x3c 的地方，先把 4 个 `FF` 写好，以后找起来方便些，一一。  

`OriginalFirstThunk`    0x205c     


`TimeDateStamp`    0x0     ；  


`ForwarderChain`    0xffffffff   ；不中转  


`Name`    0x207c  

        `user32.dll`  

`FirstThunk`    0x2064  

\----------------------------------------  

`OriginalFirstThunk`    0x206c  

`TimeDateStamp`    0x0     ；  


`ForwarderChain`    0xffffffff   ；不中转  


`Name`    0x208c  

        `kernel32.dll`  

`FirstThunk`    0x2074  

\----------------------------------------  

全 0  

\----------------------------------------  

之后我放的是 4 组 `Thunk`，注意每组最后一个元素都是 0。  

这样上面两个 `OriginalFirstThunk` 和 `FirstThunk` 都可以定下来了。  

之后是两个 DLL 的名字，于是 `Name` 可以定下来了。  

然后是两个 `IMAGE_IMPORT_BY_NAME`  

`WORD` `Hint`    ；随便写一个  

`BYTE` `Name[1]`   ；函数名  

`MessageBoxA` 是 `user32.dll` 的。  

`ExitProcess` 是 `kernel32.dll` 的。  

这样上面的 `Thunk` 可以定了。  

好，数据段的大小也定了。回到上面去改一下。  

d）最后写代码。  

```
00000200   6A 00 68 00 20 40 00 68  08 20 40 00 6A 00 2E FF   j.h. @.h. @.j.. 

00000210   15 64 20 40 00 6A 00 2E  FF 15 74 20 40 00 00 00   .d @.j.. .t @...  

00000220   00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   ................  

00000230   00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00   ................  

00401000 >/$  6A 00         push    0                 ; /Style = MB_OK|MB_APPLMODAL  

00401002  |.  68 00204000    push    00402000            ; |Title = "PE 学?,B0,""  

00401007  |.  68 08204000    push    00402008            ; |Text = "?,B4,"?,BD,"你了"  

0040100C  |.  6A 00         push    0                 ; |hOwner = NULL  

0040100E  |.  2E:FF15 64204000>  call    dword ptr cs:[<&user32.MessageBoxA>  ; \MessageBoxA  

00401015  |.  6A 00         push    0  

00401017  \\.  2E:FF15 74204000>  call    dword ptr cs:[<&kernel32.ExitProcess>]  ;  kernel32.ExitProcess  
```

代码很简单，`00 20 40 00` 也就是 `00 40 20 00`，是第一个字符串的地址。  

要注意“高高低低”的规则。  

`64 20 40 00` 是 `00 40 20 64` 是 `user32.dll` 的 `FirstThunk`,指向的就是 `MessageBoxA`  

其他：  

  学习 `pe` 的时候，用 `c` 写了段读 `pe` 结构的代码，对于学习还是很有帮助的。  

  `winnt.h` 包含了 `pe` 结构的说明，但一开始 `include` 之后一排错。  

  原来还要 `include <window.h>`。  

终于结束了，打完收工... 

4dd466d0e1e0cc123afbe504da2462ca
