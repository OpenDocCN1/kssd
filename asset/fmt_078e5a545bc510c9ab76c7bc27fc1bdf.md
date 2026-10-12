
# Konvertor 3.13

> 原文：[`www.pediy.com/kssd/pediy05/pediy50120.htm`](https://www.pediy.com/kssd/pediy05/pediy50120.htm)

`Armadillo 2.5x` 加壳的 `Konvertor 3.13` 脱壳

crack by fwnl

软件说明:是一个图片、音乐、文字及影音格式文件的转换程序，它总共可以转换 135 种文件格式，包含 25 种音乐格式、4 种影音格式、105 种图片格式及 1 种文字格式。

软件下载:`http://www.onlinedown.net/konvertor.htm`

破解工具:`trw2000`, `Import REConstructor v1.4.2+`, `winhex10.45`, `Peditor 1.7`。

首先要谢谢 `hying`, `leo_cyl`, `yahoo007`, `电神`等大哥，因为这篇文章中的内容是按他的研究成果来写的.

特别是 `yahoo007` 大哥给了我不少帮助，也要谢谢 `hying` 大哥给我的 `Konvertor 3.12A` 的输入表.

由于最后输入表的问题并没有完美解决，所以还希望各位说说手动修复输入表的方法和技巧，因为这方面的资料真是太少了，这也是我写这篇文章的初衷之一吧，谢谢

## 一. 找 oep

### 方法一:

这种方法是按 `leo_cyl` 上次的脱文来找的，原理请看 `leo_cyl` 写的 `Armadillo 2.52` 的脱文（强烈建义脱壳前看看，并搞清楚原理)

首先用 `trw2000` 载入主程序,下 `bpx createprocessa`. `F5` 返回程序被中断，下 `Pmodule` 返回程序中
然后 `Ctrl+↓` 向下看

```
0187:00909B06  CALL    `KERNEL32!WaitForDebugEvent`  //看到这个就不远了
0187:00909B0C  TEST    EAX,EAX
0187:00909B0E  JZ      NEAR 0090A128
0187:00909B14  MOV     DWORD [EBP+FFFFFD70],00911468
.............
.............
```

注意下面的这几句格式，在别的加壳程序中也是差不多的

```
0187:00909CB7  MOV     ECX,[EBP+FFFFFE74]
0187:00909CBD  CMP     DWORD [ECX+0C],C0000005
0187:00909CC4  JNZ     NEAR 00909E89
0187:00909CCA  MOV     EDX,[EBP+FFFFFE74]
0187:00909CD0  CMP     DWORD [EDX+5C],BYTE +00    //下断在这里
0187:00909CD4  JZ      00909CE2
0187:00909CD6  MOV     DWORD [EBP+FFFFFD70],009114E8
```

于是 `bpx 909cd0`, `f5` 返回，程序中断在 `00909CD0` 下 `dd edx+18`，再按一下 `F5` 可看见 oep-> `474e06`

### 方法二:

这种方法是按 `电神` 大哥说的方法来找的:

首先用 `trw2000` 载入主程序, `bpx VirtualProtect`,在最后一次被中断后，下 `Pmodule` 返回程序，然后 `F10` 单步走，当走到遇上的第 2 个 `call` 时,`F8` 进入，然后 `F10` 单步走，直到走到 `call edi`,
`edi` 的值就是 oep-> `474e06`

注意:开始 `bpx virtualprotect` 时，其中会中断很多次，这个程序在我机子上是中断 91 次,
所以我就是从第 91 中断后开始 `F10` 单步走的.

## 二. 初步脱壳

这里我是用的 `hying` 大哥的方法，因为 `leo_cyl` 上次的脱文中的意思我还没有搞明白，那篇文中谈到的几个关键点在这个程序中我没找到 : (

下面的事就是要把 `hying` 大哥说的一段代码写入到程序里

```
6804000000 push    PAGE_READWRITE
6800100000 push    MEM_COMMIT
6800005800 push    580000h  <-用 peditor 看程序的 size of image ,我这里看到是 57f000,大一点就写
6800000000 push    0                                             580000h
xxxxxxxxxx call    VirtualAlloc   <-申请内存
        mov    edi,eax
        mov    ecx,57e000     <- size of image-1000h= 57e000
        mov    esi,401000h    <-基址加 1000h，有的程序不是这个值
        rep    movsb
```

载入程序,按一下 `F10`，看看当前的地址,我的是 `90b7d9`, 于是 `d 90b7d9`, `Alt+↓` 向下看
看到 `910F64` 处内存有一片 `00 00 00 00 00`

重新载入程序.`bpx VirtualAlloc`,`Pmodule` 返回程序中，`code on`,可是看到
`Call KERNEL32! VirtualAlloc` 前的代码，我机子上是 `FF1528019100`

然后,再重新载入程序, `d 910F64` 再 `u 910F64`,在内存窗口开始填入 `680400000068001000006800005800`
`6800000000FF1528019100`,可看见下面内容, 再 `A 910f7e` 回车写入 `mov edi,eax` 就可知代码为 `89c7`
后面的也是这样,最后可见代码窗口为下面这样:

```
0187:00910F64 6804000000      PUSH    DWORD 04
0187:00910F69 6800100000      PUSH    DWORD 01000
0187:00910F6E 6800005800      PUSH    DWORD 00580000
0187:00910F73 6800000000      PUSH    DWORD 00
0187:00910F78 FF1528019100    CALL    `KERNEL32!VirtualAlloc`
0187:00910F7E 89C7            MOV     EDI,EAX
0187:00910F80 B900E05700      MOV     ECX,0057E000
0187:00910F85 BE00104000      MOV     ESI,00401000
0187:00910F8A F3A4            REP MOVSB
0187:00910F8C 0000            ADD     [EAX],AL
```

用 `Peditor 1.7` 打开原加壳程序，点 `FLC`,在 `virtual Address` 填入 `910f64`,点 `do!` 可知 `offset(hex)` 为 `3BF64`

用 `winhex` 打开原加壳程序，从 `3BF64` 开始写入
`6804000000680010000068000058006800000000FF152801910089C7B900E05700BE00104000F3A4`
保存后用 `trw2000` 打开这个被修改的程序，`bpx 474e06` ,中断后 `r eip 910f64` ,然后 `f10` 单步走,走到 `00910F7E`
时记下 `eax` 的值,我机子上是 `5560000`,继续走到 `910f8c` 时,下命令 `w 5560000 5ADE000 c:\ppp.bin`
(5560000+57E000=5ADE000)

然后 `r eip 474e06`,`F5` 返回，关闭程序

这时我们载入原加壳程序，`bpx 474e06`, `f5` 返回中断,`pedump` 脱出程序 `test.exe`

用 `winhex` 分别打开 `test.exe` 和 `ppp.bin`,在 `test.exe` 的 `offset 1000` 处开始，用 `ppp.bin` 替换下面的数据,
替换的长度就是 `ppp.bin` 的长度，然后保存.

## 三. 修复输入表

这部分对我来说太难了,因为工具并不能完全解决.我的方法是:

在 `Import REConstructor v1.4.2+` 的 `Attach to an Active Process` 窗口中选取加壳程序的进程,有 2 个进程选上面的一个,然后在下方的 oep 处填入 `00074e06`,点 `IAI AutoSearch`,再点 `Get Imports`,然后点 `Auto Trace`,然后点看还有多少地址是 `NO`,剩下的事就是手动找这些函数了.

`hying` 大哥说过
```
CreateEventW
GetStringTypeA
```
如果发现这两个函数连续在一起的话，上面一个该改为 `GetStringTypeW`

手工修复修入表后，程序应能运行了，好像也就不用注册了，可是注册部分在壳中，壳脱了注册也就不用了

这程序的输入表我是手工找到一部分，然后参照 `hying` 大哥给我的 `Konvertor 3.12A` 的输入表修复的

真希望那位能说说手工输复输入表，因为没有谁具体说过，都是很抽象的，向我这样菜鸟根本搞不明白,
我看了很多资料也只看到，`BPX GetVersion` ,然后在 `u` 那些未解决的地址，再看看上面或下面，找出正确的函数.但是并不是所有的都能这样解决.

因为是初次接触 `Armadillo` 的壳，所以中文错误和不足之处可能不少，希望各高手指正，那样就太谢谢了

:)

fwnl

2000.5.5

长沙  `4dd466d0e1e0cc123afbe504da2462ca`
