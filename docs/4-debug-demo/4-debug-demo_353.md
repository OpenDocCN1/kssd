
# Windows Commander 4.52 破解分析

> 原文：[`www.pediy.com/kssd/pediy03/forum639.htm`](https://www.pediy.com/kssd/pediy03/forum639.htm)

## 前言

破解 `Windows Commander 4.52`。

看了精华论坛 II（感谢看雪及各位大侠的精彩文章）中 `JOHNSON(C-Pen)` 大侠破解 `wincmd4.5` 的文章，手有些发痒，于是就下载了 `WinCmd4.52` 版练习练习。

**所用工具：**
- `Trw2000 1.22` (注册版)
- `UltraEdit 8.0`

## 一、脱壳

使用 `FileInfo 2.43` 检测发现是使用 `ASPACK2.1` 加的壳，但使用 `UnAsPack1.0.9.1` 自动脱壳无效（Why???，I don't know.）。

按照 `JOHNSON` 大侠的方法，用 `Trw2K` 载入 `WinCMD32.exe`，下命令：

```
pnewsec<Enter>
```

耐心地等一会儿，等 `trw2K` 跳出后，下命令：

```
makepe<Enter>
```

这样，就在 `WinCmd` 的目录下生成了一个 `newpe.exe` 文件，它就是脱壳后的文件。

## 二、去除 CRC 检验

程序发现自身被修改后，会提示有病毒，然后退出。

用 `MessageBoxa` 断点拦截后，找不到跳转点，这是和 `4.5` 版不同之处，和 `4.0` 版倒是一样（不知作者是出于什么心理）。

估计一下，从程序启动完成到弹出有病毒提示大约为 5 秒钟，因此，重新载入，下断点：

```
bpx settimer<Enter>
```

运行程序，拦下后，用 `pmodule` 命令返回，可以看到以下代码。如果没有看到，可以继续运行，重复上面动作，一定能找到：

```assembly
015F:004EF70C  MOV       EBX,EAX
015F:004EF70E  CMP       BYTE [EBX+051F],00
015F:004EF715  JNZ       004EF737      ; 关键的跳转，改成 JMP 就没问题了
015F:004EF717  MOV       BYTE [EBX+051F],01
015F:004EF71E  PUSH      BYTE +00
015F:004EF720  PUSH      DWORD 1388    ; 1388 即十进制的 5000，即 5000 毫秒
015F:004EF725  PUSH      DWORD 0541
015F:004EF72A  MOV       EAX,EBX
015F:004EF72C  CALL      00421AB8
015F:004EF731  PUSH      EAX
015F:004EF732  CALL      `USER32!SetTimer`
```

如此改了之后，程序还会不定时的自动退出。还有陷阱，问题是它没有提示，如何找到它呢？猜想不是使用的时钟，大概是在执行具体功能时检测的。

试一下，删除一个文件，结果自动退出。OK，有办法了。载入后，下断点：

```
bpx messageboxa
```

全速运行，找一个没有的文件删除，被拦下后，用 `pmodule`（这个命令太 COOL 了）返回，`F10` 步步跟踪，看到了如下代码：

```assembly
015F:004F3BA0  JMP       NEAR [ECX+00545298]
015F:004F3BA6  SUB       EAX,23B9
015F:004F3BAB  JZ        004F3BB7      ; 关键的跳转，改成 JMP 就 OK
015F:004F3BAD  MOV       EAX,01
015F:004F3BB2  CALL      004044FC      ; 此 CALL 将退出程序
015F:004F3BB7  CALL      0044CD54
015F:004F3BBC  XOR       EAX,EAX
015F:004F3BBE  POP       EDX
015F:004F3BBF  POP       ECX
015F:004F3BC0  POP       ECX
```

## 三、去除 NAG

反复看了 `johnson` 大侠的文章 `XX` 遍，还好找到了。

可让我省了很多很多的力气，要先假造 `wincmd.key` 文件。

用 `createfilea` 断点拦截，拦下后，`d eax` 看一下内存中的数据，有 `Wincmd.key` 字眼就找到了。

```assembly
015F:004D944B  LEA       EDX,[EBP+FFFFFE60]
015F:004D9451  MOV       ECX,10
015F:004D9456  CALL      00402A24
015F:004D945B  JZ        004D945F      ; 此处此处可改成 JMP，其实不跳也行，跳了会显示乱码，
015F:004D945D  XOR       EBX,EBX       ；EBX 为注册标志，盯住它，没错
015F:004D945F  MOV       EAX,[EBP+FFFFFEFC]
015F:004D9465  CALL      00402CBC
015F:004D946A  MOV       EAX,[EBP+FFFFFEF8]
015F:004D9470  CALL      00402CBC
015F:004D9475  MOV       EAX,[EBP+FFFFFEF4]
015F:004D947B  CALL      00402CBC
015F:004D9480  MOV       [0054F6CC],BL
015F:004D9486  MOV       EAX,[0054F6C8]
015F:004D948B  NEG       EAX
015F:004D948D  IMUL      EAX,EAX,BYTE +25
015F:004D9490  MOV       [0054F6C8],EAX
```

将注册标志存入内存处为：

```assembly
015F:004D9451  MOV       ECX,10
015F:004D9456  CALL      00402A24      ；再次比较 KEY 文件
015F:004D945B  JZ        004D945F
015F:004D945D  XOR       EBX,EBX       ；EBX 被清则表示未注册
015F:004D945F  MOV       EAX,[EBP+FFFFFEFC]
015F:004D9465  CALL      00402CBC
015F:004D946A  MOV       EAX,[EBP+FFFFFEF8]
015F:004D9470  CALL      00402CBC
015F:004D9475  MOV       EAX,[EBP+FFFFFEF4]
015F:004D947B  CALL      00402CBC
015F:004D9480  MOV       [0054F6CC],BL  ; 存入注册标志此处不管有无 KEY 文件都会走到
015F:004D9486  MOV       EAX,[0054F6C8]
015F:004D948B  NEG       EAX
015F:004D948D  IMUL      EAX,EAX,BYTE +25
```

因此，我们不必假造 `KEY` 文件，只要让上面的 `BL` 值为 `1` 不就行了么？

我找了下一处：

```assembly
015F:004D8DBD  JNZ       004D8DE0
015F:004D8DBF  LEA       EAX,[EBP+FFFFFEE2]
015F:004D8DC5  PUSH      EAX
015F:004D8DC6  LEA       EDX,[EBP+FFFFFA70]
015F:004D8DCC  MOV       CX,0400
015F:004D8DD0  MOV       EAX,ESI
015F:004D8DD2  CALL      0043CF10
015F:004D8DD7  MOV       EAX,ESI
015F:004D8DD9  CALL      0043CEA0
015F:004D8DDE  JMP       SHORT 004D8DEB
015F:004D8DE0  XOR       EBX,EBX
015F:004D8DE2  MOV       WORD [EBP+FFFFFEE2],00
015F:004D8DEB  CMP       WORD [EBP+FFFFFEE2],80
015F:004D8DF4  SETZ      [0054F6CD]
015F:004D8DFB  MOV       BYTE [EBP+FFFFFEE1],01
015F:004D8E02  TEST      BL,BL           ；在此行上动手脚 改成 mov bl,01
015F:004D8E04  JZ        NEAR 004D90F4   ；在此行上动手脚 改成 jmp 004d90f4，再补一个 NOP
015F:004D8E0A  LEA       EAX,[EBP+FFFFF608]
015F:004D8E10  PUSH      EAX
015F:004D8E11  CALL      `KERNEL32!GetTimeZoneInformation`
015F:004D8E16  INC       EAX
015F:004D8E17  JZ        004D8E35
```

另外再把 `015F:004D945D` 处改成 2 个 `NOP`。

当然具体改法很多，自己决定吧！

这样，运行程序没有 `NAG` 了，但标题栏显示为 `Not Registered`，不管它了。当然也可改掉，不过我很懒啦，反正已经达到目的了，至此为止！

通过分析，可以知道：
- 第一次比较 `KEY` 文件为判断是否注册；
- 第二次比较 `KEY` 文件为判断是否要出 `NAG`。

## 结语

这篇破文总算是写完了，花了我很长时间，虽然没有什么新意，但也很累。

看了网上那么多大侠的文章，我也把自己的一点体会写出来，算占一点回报吧。

如有遗漏可来 `E-mail` 联系，没有时间作全面充分的测试（现在的软件陷阱多多）。

再见吧！各位好运!!!!!

`written by mjing` `E-mail：mjing@wx88.net` `2001-2-19` `4dd466d0e1e0cc123afbe504da2462ca`
