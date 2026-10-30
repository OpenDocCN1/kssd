
```
0:005> db 11e078
0011e078 3b 9f c0 73 3b 9f c0 73-bc 4f c0 73 3e 52 c0 73 ;..s;..s.O.s>R.s
0011e088 7b 53 c0 73 5c 75 c0 73-5c 75 c0 73 78 1e 17 00 {S.s\u.s\u.sx...
0011e098 78 1e 17 00 00 00 00 00-50 01 20 01 00 00 01 00 x.......P. .....
0011e0a8 00 00 00 00 78 7e 17 00-00 00 00 00 00 00 00 00 ....x~..........
0011e0b8 05 00 09 00 ae 01 08 00-e8 e0 11 00 00 00 00 00 ................
0011e0c8 00 00 00 00 00 00 00 00-00 00 00 00 00 00 00 00 ................
0011e0d8 00 00 00 00 00 00 00 00-05 00 05 00 a5 01 08 00 ................
0011e0e8 38 e1 11 00 00 00 00 00-00 00 00 00 00 00 00 00 8...............
0:005> dds 11e078
0011e078 73c09f3b iccvid!ExpandCodeBook32
0011e07c 73c09f3b iccvid!ExpandCodeBook32
0011e080 73c04fbc iccvid!DrawKey32
0011e084 73c0523e iccvid!DrawSmooth32
0011e088 73c0537b iccvid!DrawInter32
0011e08c 73c0755c iccvid!DetailCodeFromRGBa8888
0011e090 73c0755c iccvid!DetailCodeFromRGBa8888
0011e094 00171e78
0011e098 00171e78
0011e09c 00000000
0011e0a0 01200150 xpsp2res+0x1e0150
0:005> db 2676af8
02676af8 00 00 00 68 01 60 01 20-00 10 10 00 00 10 00 00 ...h.`. ........
02676b08 00 00 00 60 01 60 20 00-00 00 11 00 00 10 41 41 ...`.` .......AA
02676b18 41 41 41 41 41 41 41 41-41 41 11 00 00 10 41 41 AAAAAAAAAA....AA
02676b28 41 41 41 41 41 41 41 41-41 41 11 00 00 10 41 41 AAAAAAAAAA....AA
02676b38 41 41 41 41 41 41 41 41-41 41 11 00 00 10 41 00 AAAAAAAAAA....A.
02676b48 69 64 78 31 10 00 00 00-30 30 64 63 10 00 00 00 idx1....00dc....
02676b58 04 00 00 00 68 00 00 00-00 00 00 00 00 00 00 00 ....h...........
02676b68 00 00 00 00 00 00 00 00-00 00 00 00 00 00 00 00 ................
```

第二个参数`0x2676af8`处的数据是不是特别眼熟，就是上面的CVID数据，看来第二个参数确实是源数据，b(￣▽￣)d。而第一个参数那里保存的是一些函数地址，不知道有什么用(后来发现这里的一块空间应该是用于存储数据的)。

# 4.3 漏洞函数代码分析

接下来我是IDA+Windbg同步进行分析的，由于Windbg的结果贴上来实在太冗杂了，所以直接贴上IDA中的代码分析结果：

```c
int __stdcall CVDecompress(ULONG data, int src, int length, int a4, int a5, int a6, int a7)
{
// [COLLAPSED LOCAL DECLARATIONS. PRESS KEYPAD CTRL-"+" TO EXPAND]
data_ = data;
v8 = *(data + 36);
if...
result = 0;
if ( length >= 0x20 )
{
src_ = src;
BYTE1(result) = *(src + 1);
LOBYTE(result) = *(src + 2);
v11 = *(src + 3) | (result << 8); // 这里得到的还是length, 0x68
if ( length < v11 || (HIBYTE(length) = *src, ULongSub(v11, 10u, &remain_len) < 0) )// 这里做了一个运算0x68-10，减去了头部的长度看是否还大于0
{
LABEL_33:
result = 0;
}
else
{
HIBYTE(nStrip) = *(src_ + 8);
strip = (src_ + 10); // 加上了头部的长度，现在指向strip 1的起始位置
strip_idx = 0;
strip_ = strip;
strip__ = strip;
LOBYTE(nStrip) = *(strip - 1); // 这里是strip的个数
nStrip_ = nStrip;
if ( nStrip )
{
v32 = 0;
do
{
if ( remain_len < 0x16 )
break;
HIBYTE(v14) = strip[1];
LOBYTE(v14) = strip[2];
strip_len = strip[3] | (v14 << 8);
if ( remain_len < strip_len )
break;
if ( *strip == 0x10 || *strip == 0x11 )// 检查strip ID
{
if ( ULongSub(strip_len, 0xCu, &data) < 0 )// 这里又减去了strip header的长度 12个字节
// data的位置存储的是差值
goto LABEL_33;
HIBYTE(bottom_y) = strip[8];
HIBYTE(top_y) = strip[4];
LOBYTE(bottom_y) = strip[9];
LOBYTE(top_y) = strip[5];
v17 = bottom_y - top_y;
LOWORD(v17) = *(data_ + 46) * v17;
src = v17;
if ( v32 && !HIBYTE(length) && *strip == 0x11 )// 当遍历到第二个strip的时候，条件都满足，进入if块
{
qmemcpy((*(data_ + 28) + v32), (*(data_ + 28) + v32 - 0x2000), 0x2000u);// 异常发生在这里
strip = strip_;
}
chunk = strip__ + 12; // 这里又加上了strip header的长度，目前指向了chunk
chunk_ = strip + 12;
*(data_ + 56) = v32 + *(data_ + 32);
chunk__ = strip + 12;
*(data_ + 60) = a7;
while ( data >= 4 )
{
HIBYTE(v20) = chunk_[1];
LOBYTE(v20) = chunk_[2];
chunk_size = chunk_[3] | (v20 << 8);
chunk_size_ = chunk_size;
if ( data < chunk_size ) // 验证chunk_size的大小
break;
switch... // 这里在检查chunk ID,根据chunk ID处理chunk数据
chunk_ = &chunk__[chunk_size_];
v22 = 1;
chunk += chunk_size_;
chunk__ += chunk_size_;
if ( chunk_size_ > 1 )
v22 = chunk_size_;
data -= v22;
}
a6 += a7 * src;
++strip_idx;
v32 += 0x2000;
}
strip__ += strip_len; // 接着处理下一个strip
remain_len -= strip_len;
strip += strip_len;
strip_ = strip;
}
while ( strip_idx < nStrip_ );
}
result = 1;
}
}
return result;
}
```

上述代码理解可以看下面这张图：

![图片描述](upload/attach/202107/600394_WNCUAGC2VAAZBJ3.png)

每次遇到strip ID是`0x1100`的时候，都会进行数据的复制操作，每次复制的字节数为`0x2000`，这里并没有判断堆中是否能够容纳这么多的数据，查看一下堆块的大小：

```
0:005> !heap -p -a edi
address 00173e78 found in
_HEAP @ a0000
HEAP_ENTRY Size Prev Flags UserPtr UserSize - state
00171e70 0c01 0000 [01] 00171e78 06000 - (busy)
? wmploc+93
```

可以看到这个堆的大小是`0x6000`，也就是说复制三次数据就满了，第四次再复制的时候就会导致堆溢出。o(*￣▽￣*)ブ

# 4.4 小总结

### 4.4.1 关于cvid压缩格式

目前看来我对于CVID格式的理解是没什么问题的，chunk是strip的一部分，strip1后面跟着strip2，以此类推

### 4.4.2 如何触发漏洞

1. CVID数据长度就按照数据总长度设置，需要大于`0x20`
2. 包含大于3个，且ID为`0x1100`的strip

# 5. 本文Windbg学习总结

1. `!heap`：查看堆相关信息
使用`!heap -?`查看帮助信息
`!heap -p`：查看所有可能的handle值
`!heap -p -h HANDLE`：查看对应HANDLE的详细分配信息
`!heap -p -a ADDR`：显示具体某个堆块的详细信息

2. `dt STRUCTURE ADDR`：按照STRUCTURE结构显示ADDR处结构体各成员值

3. `!gflag [-? | flags]`指令用于堆调试，重点标志：htc, hfc, hpc, hpa

4. `sx`：用于控制被调试程序发生某异常或特定事件时，调试器要采取的动作
`sxe ld:ModuleName` 在首次加载ModuleName的时候中断。

# 6. 总结

这次漏洞分析花费的时间主要集中在前面对于堆溢出相关知识的复习，以及AVI格式的学习上。一旦对于格式有所了解，使用`!gflag`定位到漏洞函数后，代码分析就很简单了。

`!gflag`和`!heap`命令在堆溢出的漏洞分析上帮助很大；如何使用Windbg，在加载DLL之后设置断点也是新接触到的一个知识点。

对于该漏洞本身，在刚看到漏洞介绍时，我只以为是在单次strcpy调用时复制的数据超过了目标空间大小，但实际调试时发现是多次同样大小的数据复制最终超过了目标空间大小，虽然原理相同，但这里需要判断的其实应该是复制的次数而不是大小了。

# 7. 参考资料

1. 《漏洞战争》

2. <a href="elink@e20K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6A6L8X3c8T1k6#2)9J5k6h3W2F1k6X3!0Q4x3V1k6V1L8$3y4Q4x3V1j5I4i4K6u0V1j5$3!0E0L8h3!0F1i4K6u0V1j5$3#2V1M7#2)9J5k6h3S2@1L8h3H3`.">Common WinDbg Commands (Thematically Grouped)</a>

3. <a href="elink@b39K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5c8X3y4Z5k6h3&6W2N6q4)9J5c8U0M7&6x3e0M7@1y4e0l9H3">AVI文件格式详解</a>

4. <a href="elink@ebdK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0N6h3I4@1K9h3#2W2k6r3W2S2i4K6u0W2j5%4S2Q4x3V1k6E0K9i4u0J5L8%4u0Q4x3V1k6U0K9h3&6W2M7r3q4C8i4K6u0W2N6s2S2@1">cinepak文档</a>

# 0. 前言

这篇文章可以分为两大部分，前半部分根据《漏洞战争》的代码，在调试的过程中重新复习了一遍堆结构相关的知识，并学习了Windbg提供的堆调试选项；后半部分是对CVE-2010-2553分析，包括AVI格式分析以及漏洞函数代码分析。

在学习过程中发现了书中对AVI格式的介绍存在一些问题，同时对于溢出漏洞也有了更深的理解。

# 1. 堆溢出相关知识

之前一直在看栈溢出漏洞，这次选择堆溢出，之后也会各个漏洞轮流分析，加强一下记忆。

由于有一段时间没看过这部分内容，再加上《漏洞战争》中使用的是Windbg这个我不太熟悉的工具，所以决定完整的跟一遍前面的“堆溢出原理”小结，同时回顾一下自己在学习0day时总结的堆溢出漏洞笔记。

## 1.1 代码基本调试

注：*书中该实验环境是Win7简体中文旗舰版，我使用的版本不同，结果并没有触发异常，这里也要分析一下原因*

**实验环境：WinXP SP3**

实验代码：

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21 | `#include <windows.h>`  `#include <stdio.h>`    `int` `main () {`  `HANDLE hHeap;`  `char` `*``heap;`  `char` `str``[]` `=` `"AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA"``;`    `hHeap` `=` `HeapCreate(HEAP_GENERATE_EXCEPTIONS,` `0x1000``,` `0xffff``);`  `__asm` `int` `3`    `heap` `=` `(char` `*``)HeapAlloc(hHeap,` `0``,` `0x10``);`  `printf(``"heap addr: 0x%08x\n"``, heap);`    `strcpy(heap,` `str``);`  `HeapFree(hHeap,` `0``, heap);`    `HeapDestroy(hHeap);`    `return` `0``;`  `}` |

程序在`int 3`中断并进入调试器之后，刚刚执行完`HeapCreate`，起始地址为`0x3a0000`，根据0day中学到的知识，偏移`0x178`的位置是空表索引区：

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9 | `0``:``000``> dd` `3a0178`  `003a0178`  `003a0688` `003a0688` `003a0180` `003a0180`  `003a0188`  `003a0188` `003a0188` `003a0190` `003a0190`  `003a0198`  `003a0198` `003a0198` `003a01a0` `003a01a0`  `003a01a8`  `003a01a8` `003a01a8` `003a01b0` `003a01b0`  `003a01b8`  `003a01b8` `003a01b8` `003a01c0` `003a01c0`  `003a01c8`  `003a01c8` `003a01c8` `003a01d0` `003a01d0`  `003a01d8`  `003a01d8` `003a01d8` `003a01e0` `003a01e0`  `003a01e8`  `003a01e8` `003a01e8` `003a01f0` `003a01f0` |

可以看到`0x003a0688`那里就是`Freelist[0]`中唯一一个空的堆块：

|  |  |
| --- | --- |
