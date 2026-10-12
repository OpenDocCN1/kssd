
# 使用 OllyDbg 快速脱壳

> 原文：[`www.pediy.com/kssd/pediy05/pediy50413.htm`](https://www.pediy.com/kssd/pediy05/pediy50413.htm)

使用 `OllyDbg` 快速脱壳

`dOSKEY lEE`

目标：采用 `ASPACK`、`UPX` 加壳的 `NOTEPAD.EXE`

工具：`OllyDbg` 1.09 英文版、`DUMP` 插件、`PEditor`

系统：`Win98SE`

关键词： 脱壳、`OllyDbg`、`OD`、`DUMP`、`PUSHAD`、`POPAD`

## 预备知识

大多数壳都有一个共同的特点。在壳准备开始解压时都要执行 `PUSHAD`，当壳解压完时都要调用 `POPAD`。到底 `PUSHAD` 和 `POPAD` 是什么干什么用的呢？其实 `PUSHAD` 是用来将所有普通寄存器顺序进栈的指令，`POPAD` 是所有普通寄存器顺序出栈指令。`POPAD` 的出栈顺序和 `PUSHAD` 相反。壳为了保护寄存器，便在解压前将所有寄存器进栈保护起来，当解压完成后又将寄存器出栈，恢复其原貌，并将 `IP` 设置为原程序的 `OEP`。这样我们就可以通过这个特点快速脱掉多种软件的壳。

## ASPACK 篇

先用 `ASPACK` 将 `NOTEPAD.EXE` 加壳。用 `OllyDbg`（以下简称 `OD`）载入。看见光标停在壳的入口处。

```
0040D001 >  60                 PUSHAD ;壳的入口。准备开始解压，保护寄存器
0040D002    E8 03000000        CALL     NOTEPAD.0040D00A
……
```

我们不管它，直接向下翻页找 `POPAD` 指令。在 `40D3AF` 处找到 `POPAD`

```
……
0040D3AF    61                 POPAD ;解压完成，恢复寄存器
0040D3B0    75 08              JNZ      SHORT NOTEPAD.0040D3BA
0040D3B2    B8 01000000        MOV      EAX, 1
0040D3B7    C2 0C00           RETN     0C
0040D3BA    68 CC104000        PUSH     NOTEPAD.004010CC ;返回到原程序 OEP 处
0040D3BF    C3                 RETN
……
```

选定 `40D3AF` 这一行，`F4` 运行到此处。在这里说明壳已经完成解压工作。并且返回到原程序的入口处。`F8` 单步到 `4010CC`，这里便是原程序的 `OEP`。用 `DUMP` 插件直接 `DUMP` 出来就可以了（在 `DUMP` 时注意将入口点改为 `10CC`，即 `4010CC-400000=10CC`，`400000` 是映象基地址）。文件大小是 `77059` 字节，用 `PEditor` 重建 `PE` 头便可以了。未压缩的文件大小是 `53248` 字节，脱壳后的文件大小是 `60930` 字节。

## UPX 篇

用 `UPX` 将 `NOTEPAD.EXE` 加壳，然后用 `OD` 载入。停在 `PUSHAD` 处，用脱 `ASPACK` 同样的方法，向下翻页找 `POPAD`。

```
……
0040E9FE    61                 POPAD
0040E9FF - E9 C826FFFF         JMP      NOTEPAD.004010CC
……
```

下面的 `JMP` 就是跳转到程序的 `OEP` 处。`F4` 到 `40E9FF` 处，`F8` 单步一下，来到 `OEP` 处，`DUMP` 出来。`DUMP` 文件的大小是 `65536` 字节，直接就可以运行。为了完美，用 `PEditor` 重建 `PE` 头。那么脱壳后的文件大小是 `60293` 字节。

## 后记

用上面说的方法，很多种壳都可以快速的手动脱掉。如果你没有 `OD` 的 `DUMP` 插件，可以到新论坛的下载区找。如果实在没有，也可以直接停在 `OEP` 处用 `PEDump` 来 `DUMP`。很久没有写东西了。这一篇是写给初学者练手的。其实壳也是软件，再怎么复杂都有可能被脱下来。祝你好运。

另外，转载时请保持本文的完整。 `4dd466d0e1e0cc123afbe504da2462ca`
