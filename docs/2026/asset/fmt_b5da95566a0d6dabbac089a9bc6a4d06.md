
# 漏洞分析——ASX to MP3 Converter本地代码执行漏洞

Windows漏洞利用开发教程 Part 4：SEH

本文是针对ASX to MP3 Converter进行的漏洞分析

## 背景概述

本来是浏览学长博客[ASX TO MP3本地代码执行漏洞](https://bbs.kanxue.com/elink@bf6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4K9r3g2J5k6h3W2K6K9K6m8K6K9r3I4Q4x3X3g2@1L8%4m8Q4x3V1k6H3L8%4y4@1i4K6u0r3x3U0l9I4y4W2)9J5k6o6p5I4i4K6u0V1x3o6b7`.)感觉这个洞比较好入手，思路也比较清晰，就想着简单复现一下，但是在实际分析之中发现与博客原文写的有较大出入，于是自己一路分析上去，想搞清楚差异的地方探究真正的漏洞原因。

实验环境与原博中一样：

Asx to MP3 Converter 3.1.2（原博中写的是3.0.0，但提供的下载链接是3.1.2，是否是分析出现差异的原因？）

Windows xp sp3

Windbg

IDA pro

OD

软件下载地址：258K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2W2P5s2m8D9L8$3W2@1i4K6u0V1k6r3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2M7s2m8K6i4K6u0r3j5U0x3I4j5e0R3@1k6e0M7&6k6o6V1&6y4o6q4V1z5o6V1K6x3K6k6T1y4U0M7H3z5r3g2X3y4e0u0S2x3U0m8Q4x3X3c8m8f1#2S2@1L8@1#2b7x3@1y4G2L8Y4k6W2M7Y4c8W2M7W2)9#2k6U0x3I4x3U0q4Q4x3X3g2W2P5r3f1`.

### SEH攻击原理简述（详情可参考0day2）

为了保证系统在遇到错误时不至于崩溃，仍能够健壮稳定地继续运行下去，Windows会对运行在其中的程序提供一次补救的机会来处理错误，这种机制称为Windows异常处理机制。S.E.H即异常处理结构体(Structure Exception Handler)S.E.H链表指针和异常处理函数句柄，共计8个字节。

S.E.H结构体存放在系统栈中，线程初始化时，会自动向栈中安装S.E.H作为默认的异常处理。当异常发生时，操作系统会中断程序，并首先从T.E.B的0字节偏移处取出距离栈顶最近的S.E.H中的异常处理函数进行处理。

![image-20200417205721152](https://bbs.kanxue.com/upload/attach/202210/832012_RTEHTU9XPA2R4KP.jpg)

利用思路基本是：

1. 通过溢出覆盖栈中SEH，将指向下一条记录的指针覆盖为shellcode地址；
2. 触发一个SEH；
3. 应用程序处理SEH时调用了指令，将指向下一条记录的指针作为EIP内容；
4. 跳到shellcode地址，执行shellcode。

### 原博中的分析思路复现

利用poc生成一个畸形m3u文件，播放器打开文件，程序崩溃

PoC:

|  |  |
| --- | --- |
| 1  2  3  4  5 | `poc` `=` `"\x41"` `*` `50000`    `rst` `=` `open``(``"exploit.m3u"``,``'w'``)`  `rst.write(poc)`  `rst.close();` |

原文理想中的情况应该是：

1.使用windbg到达漏洞现场，使用kb发现堆栈调用全是系统dll的调用，无法回溯漏洞发生前的关键函数，于是更改思路；

2.想到该漏洞是一个文件格式的poc，那么在主程序中很有可能会通过`fopen`调用这个poc打开，以读取其中的文件从而触发文件格式的漏洞，于是想办法通过`fopen`函数来定位到漏洞发生前的函数；

3.在IDA Pro中找到5个`fopen`的字符串函数，在这几个地址处用windbg打断点；

4.发现在加载漏洞文件后windbg调用了`fopen`两次，第一次正常，在第二次时将畸形的文件格式读入到缓冲区中，而此时没有对文件的长度进行任何检查，而是直接读入缓冲区；

5.继续运行再调用后续函数时，返回地址被覆盖，导致出现一个错误，调出SEH，而SEH已经被覆盖，最终程序结束；

最终得出结论：***由于对于`filename`的长度检查不严格，导致直接作为参数回调到外层函数中，因此当外层函数结束时，程序返回到一个不可读的地址，从而触发了异常处理流程，因为文件畸形内容，导致了SEH指针被覆盖，程序可控。***

我按照此思路复现实际遇到的情况：

1.使用windbg达到漏洞现场，发现可以回溯漏洞发生前的关键函数，断在了`00430402`的位置

![image-20200416171026626](https://bbs.kanxue.com/upload/attach/202210/832012_NEMQ6AAYSW4AYT7.jpg)

2.在IDA中同样搜寻到5个`fopen`的调用并在windbg中对应打了断点，但是在实际调用中加载漏洞文件后windbg调用了`fopen`仅1次而不是2次，之后又会断在`00430402`

![image-20200417230817139](https://bbs.kanxue.com/upload/attach/202210/832012_PRASRVW5BFP68GF.jpg)

3.使用OllyDbg分析程序崩溃位置也与原博不同，原博程序是在执行到最后`ret 4`时，跳转地址被覆盖为`414141`不可读出现错误，实际调试过程中并未执行到该函数段最后，而是在过程中读写错误崩溃的，具体原因是：

```
0042BD1D |mov ecx,dword ptr ss:[esp+0xAA48] ；堆栈 ss:[000DBFB4]=41414141
0042BD26 |push ecx ；堆栈000D1568 41414141 AAAA
004303E9 | mov ebx,dword ptr ss:[esp+0x10] ；堆栈 ss:[000D1568]=41414141
004303FE | mov esi,ebx ；ebx=41414141
00430402 |rep movs dword ptr es:[edi],dword ptr ds:[esi] ；ds:[esi]=[41414141]=???
```

在执行`rep movs`操作中由于`414141`地址不可读导致出错，并且此时SEH已然被覆盖了

![image-20200417234729856](https://bbs.kanxue.com/upload/attach/202210/832012_M3WWSJ6NP2KQTPS.jpg)

#### 此时的疑问

1.既然没有第二次`fopen`依然崩溃，问题可能并不出在`fopen`操作，那么问题出在哪？

2.从图中可知SEH已经被覆盖，那么是在之前何处覆盖的？操作是什么？

此时我发现不止有我对此质疑，网上另一篇帖子也无法复现，和我的情况完全相同，为我提供了下一步的思路

[漏洞分析——ASX to MP3 Converter本地代码执行漏洞](https://bbs.kanxue.com/elink@154K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2P5h3g2K6j5i4N6&6k6i4u0Q4x3X3g2Y4K9i4c8Z5N6h3u0Q4x3X3g2A6L8#2)9J5c8U0t1H3x3e0W2Q4x3V1j5H3y4#2)9J5c8U0l9^5i4K6u0r3i4K6t1#2c8e0k6Q4x3U0g2n7b7#2)9J5y4e0S2r3i4K6t1#2c8e0k6Q4x3U0g2n7y4q4)9J5y4e0W2q4i4K6t1#2c8e0g2Q4x3U0f1^5z5q4)9J5y4e0R3$3i4K6t1#2c8e0k6Q4x3U0f1&6c8g2)9J5y4e0V1H3i4K6t1#2c8e0u0Q4x3U0f1^5x3q4)9J5y4e0V1@1i4K6t1#2c8e0u0Q4x3U0f1^5x3q4)9J5y4e0V1@1b7g2y4j5i4K6u0V1N6r3!0Q4x3X3c8y4f1o6y4Q4x3X3c8o6L8$3&6$3k6i4u0@1k6i4u0Q4x3U0g2q4y4W2)9J5y4e0W2o6i4K6t1#2b7f1y4Q4x3U0g2q4y4g2)9J5y4e0W2o6i4K6t1#2b7U0m8Q4x3U0g2q4y4q4)9J5y4f1u0n7i4K6t1#2b7e0y4Q4x3U0g2q4y4#2)9J5y4f1p5H3i4K6t1#2z5o6q4Q4x3U0g2q4y4W2)9J5y4e0R3&6i4K6t1#2b7e0N6Q4x3U0g2q4z5q4)9J5y4f1p5I4i4K6t1#2z5p5y4Q4x3U0g2q4y4W2)9J5y4f1u0o6i4K6t1#2z5p5k6Q4x3U0g2q4y4W2)9J5y4f1t1@1i4K6t1#2z5f1g2Q4x3V1j5`.)

### 分析原因

从崩溃地址`00430402`开始向上分析，追溯这一堆A的由来，首先断点依然在`fopen`处，虽然已经感觉问题不在这里，但还是看一下

![image-20200418000156687](https://bbs.kanxue.com/upload/attach/202210/832012_DZ8GM3MNS7ZRWA2.jpg)

从地址`428B46`调用`fopen`，在`428B72`处调用`freed`读入文件的值，之后一直到`fclose`都没有任何异常数据，所以漏洞点并不在这部分代码中

继续向下调试，发现在`42B62B`位置调用`MSA2Mfil.Playlist_FindNextItem`后就开始有异常数据，可以判断问题出在该函数内部，且在模块列表中发现该函数是在程序自带的Dll库中，是`MSA2Mfilter03.dll`

![image-20200416181606697](https://bbs.kanxue.com/upload/attach/202210/832012_PZWGPZYBS685U7G.jpg)

同时使用IDA查看`MSA2Mfilter03.dll`中的代码，在Functions中确实有`Playlist_FindNextItem`函数

![image-20200416190947842](https://bbs.kanxue.com/upload/attach/202210/832012_TG7BSQ9HJ4H4QWS.jpg)

在OD中也进入该dll逐步调试，在`10008D55`位置第一次出异常数据，在IDA中对应发现在函数`sub_10008D20`中无论条件如何，最终都会输出该异常数据

![image-20200418004152406](https://bbs.kanxue.com/upload/attach/202210/832012_Y22VTESGBSVXQMV.jpg)

![image-20200418003654169](https://bbs.kanxue.com/upload/attach/202210/832012_C79ZH54M5RNRU8M.jpg)

![image-20200418003726435](https://bbs.kanxue.com/upload/attach/202210/832012_YD2ETQ46YSP9F9G.jpg)

![image-20200418092057997](https://bbs.kanxue.com/upload/attach/202210/832012_TMRTSPJ9AMFXP83.jpg)

继续向下走，依旧是在该dll中，发现在地址`1000D3C3`处为溢出点，SEH就是在此处被覆盖掉的

![image-20200418005703356](https://bbs.kanxue.com/upload/attach/202210/832012_VESCXSAF9JUCUX9.jpg)

ida中查看REP MOVS循环，其实是使用函数`strcpy`完成循环复制，问题就出现在`strcpy`函数。

![image-20200416222010065](https://bbs.kanxue.com/upload/attach/202210/832012_GRYJMZV45S98HFS.jpg)

![image-20200418090251250](https://bbs.kanxue.com/upload/attach/202210/832012_85HB9RJ29RXF6XY.jpg)

此时我们就可以简单计算一下从覆盖的起点（由图可知D159C）到最近一个SEH（由图可知DBFA4）有多远了，它们相减是43528，看来poc设置50000还是有考虑的，经过多次测算调试，poc中设置43483个A（前面一串路径占了部分位置），后面的八个字符即正好是SEH的位置，后面写利用脚本可以使用该位置。

![image-20200416224706440](https://bbs.kanxue.com/upload/attach/202210/832012_YZG558GWQA5EU3J.jpg)

![image-20200416224017391](https://bbs.kanxue.com/upload/attach/202210/832012_UZEE34R76NZSMPK.jpg)

![image-20200416224157265](https://bbs.kanxue.com/upload/attach/202210/832012_PQE6FXCZUNTJ9N7.jpg)

看到这里有没有朋友发现奇怪的地方，在第二部分原博复现：我实际的操作第3步中分析了程序不像原博中一样执行到了最后而是在中间断开的原因，起端就是如下图这个位置堆栈`DBFB4`的非法数据`mov`进了`ecx`，最终导致内存读写错误，而`DBFB4`的非法数据从何而来我们从上图就可以一清二楚

![image-20200417102251074](https://bbs.kanxue.com/upload/attach/202210/832012_9XDEFXZYVSUCDST.jpg)

如果我们想要复现原博中的情况，让它执行到最后再跳转错误，只需要在构造poc时，中间多垫几层，在`DBFB4`位置输入一个合法地址就可以了，此时poc应该为

|  |  |
| --- | --- |
