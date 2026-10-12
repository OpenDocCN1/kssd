
# Adobe PDF LibTiff Integer Overflow CVE-2010-0188 初探

> 原文：[`www.pediy.com/kssd/pediy12/109316.html`](https://www.pediy.com/kssd/pediy12/109316.html)

CVE-2010-0188 漏洞初探

## 一. TIFF 文件格式介绍

### 1、图像文件头（Image File Header 简称 IFH）：
![](img/9e92430824c922e02c1d19c7977af59c.jpg)
IFH 结构描述(图 1)

IFH 数据结构包含 3 个成员共计 8 个字节，`Byte order` 成员可能是“MM”(0x4d4d)或“II”(0x4949)，0x4d4d 表示该 TIFF 图是 Motoral 整数格式 0x4949 表示该图是 Intel 整数格式；`Version` 成员总是包含十进制 42(0x2a)；第三个成员是 IFD 相对文件开始处的偏移量。

### 2、图像文件目录（Image File Directory 简称 IFD）：
![](img/0461d8179ac610a06fe6cda362ec8c3d.jpg)
IFD 及 DE 结构描述(图 2)

`IFD` 是 TIF 图中最重要的数据结构，它包含了一个 TIF 文件中最重要的信息，一个 TIF 图可能有多个 `IFD`，这说明文件中有多个图像，每个 `IFD` 标识 1 个图像的基本属性。`IFD` 结构中包含了三类成员，`Directory Entry Count` 指出该结构里面有多少个目录入口；接下来就是 N 个线性排列的 `DE` 序列，数量不定（这就是为什么称 TIF 格式文件为可扩充标记的文件，甚至用户可以添加自定义的标记属性），每个 `DE` 标识了图像的某一个属性；最后就是一个偏移量，标识下一个文件目录相对于文件开始处的位置，当然，如果该 TIF 文件只包含了一幅图像，那么就只有一个 `IFD`，显然，这个偏移量就等于 0；

### 3、目录入口（Directory Entry 简称 DE）：
共 12 个字节，见图 2。简单说，一个 `DE` 就是一幅图像的某一个属性。例如图像的大小、分辨率、是否压缩、像素的行列数等。其中：`tag` 成员是该属性的编号，在图像文件目录中，它是按照升序排列的。属性是用数据来表示的，那么 `type` 就是代表着该数据的类型，TIF 官方指定的有 5 种数据类型。`type=1` 就是 BYTE 类型（8 位无标记整数）、`type=2` 是 ASCII 类型（7 位 ASCII 码加 1 位二进制 0）、`type=3` 是 SHORT 类型（16 位无标记整数）、`type=4` 是 LONG 类型（32 位无标记整数）、`type=5` 是 RATIONAL 类型（2 个 LONG，第一个是分子，第二个是分母）。`length` 成员是数据的数量而不是数据类型的长度。第 4 个成员 `valueOffset` 很重要，它是 `tag` 标识的属性代表的变量值相对文件开始处的偏移量。如果变量值占用的空间小于 4 个字节，那么该值就存放在 `valueOffset` 中即可，没必要再另外指向一个地方了。

## 二. 分析漏洞产生的原因：

分析样本发现，此漏洞与 CVE-2006-3459 类似，都是由于 `DotRange` 属性引起的。

![](img/d0e1548d625aa1330ce7d4e0862d9681.jpg)

`DotRange` 一般为两个值，即 `DotRange[0]` 和 `DotRange[1]`。`DotRange` 标签是一个目录项结构，12 字节的数据定义了该标签的 TAG、数据类型，数据长度以及值偏移。通常情况下，`DotRange` 的目录项结构是这个样子的：

TAG Type Count Value/Offset
0x0150 0x0003/0x0001 0x00000002 0xAAAAAAAA

`DotRange` 的 TAG 值为 0x0150，数据类型或为 SHORT 型，或为 BYTE 型，数据长度如前所述，不能超过 2，而数据偏移则指出了 `DotRange[0]` 和 `DotRange[1]` 在文件中的位置。然而，如果我们此时恶意的将 `DotRange` 目录项中的 `Count` 字段设置为任意大于 2 的值，那会出现什么情况呢？事实证明，这就是 `DotRange` 漏洞利用最原始的想法，`Libtiff` 根据 `Count` 的值，读入 `DotRange` 数据。如果攻击者精心构造了 `Count` 值，并在文件的特殊位置提供了恶意代码的话，那读入的多余数据将覆盖 `Libtiff` 的栈结构，引起缓冲区溢出，最终导致 shellcode 的执行。

程序执行的流程图：
![](img/8e942eb35416b802b072d6be483b1d7e.jpg)

`TIFFReadDirectory()` 函数在 `switch-case` 中，在遇到 `TIFFTAG_PAGENUMBER`、`TIFFTAG_HALFTONEHINTS`、`TIFFTAG_YCBCRSUBSAMPLING` 和 `TIFFTAG_DOTRANGE` 标签时进入 `TIFFFetchShortPair()` 的处理流程。`TIFFFetchShortPair()` 函数，这里没有判断 `Count` 的大小，从而产生了此漏洞。

![](img/6da9e49093d6ac1209f9b64bfc193f2a.jpg)

安装补丁之后，`TIFFFetchShortPair()` 增加了判读，当 `Count` 小于等于就跳过报错代码继续正常执行，当 `Count` 大于 2 时就报错(unexpect count ……)：
![](img/f1319d02c7c2b32dac9316135b73fc4f.jpg)

`TIFFFetchShortPair()` 其调用的是 `TIFFFetchShortArray()`。正常情况，`Count≤2`，流程调用的是函数中(`dir->tdir_count <= 2`)的分支；而恶意时，`Count>2`，执行流程又转入了 `TIFFFetchData()`。

`TIFFFetchShortArray()` 函数调用 `TIFFFetchData()` 处:
![](img/7a105c96cde1ce829cf4cfab9deb5557.jpg)

在 `TIFFFetchData()` 函数中，`tif->tif_size` 为文件大小，`dir->tdir_offset` 为 `DotRange` 数据的起始位置，`cc` 是数据真实长度，那 `dir->tdir_offset + cc` 也就是 `DotRange` 数据的结束位置，如果数据的结束位置超过了打开 TIFF 文件的大小，那也将会输出错误信息。这一点在构造恶意 TIFF 时非常重要，`DotRange` 标签中的 `Count` 值并不是大于 2 的任意数值都可以，要保证恶意数据的结束位置≤TIFF 文件尺寸，否则就算更改了数值，也可能因位置验证错误而覆盖失败。

`TIFFFetchData()` 函数调用了 `memcpy()` 处：
![](img/7c9284100038087348f8e2a8f1c67148.jpg)

缓冲区溢出的发生处是 `\_TIFFmemcpy(cp, tif->tif_base + dir->tdir_offset, cc);`。`\_TIFFmemcpy` 包装了一个 `memcpy`（见图 3），将从 `tif->tif_base + dir->tdir_offset`，即 `DotRange` 数据起始处开始的 `cc` 字节的数据拷贝到 `cp` 指定的内存。`cc` 是恶意构造的、验证后的 `Count` 值，而 `cp` 指向的正是 `TIFFFetchShortPair()` 中的栈变量 `uint16 v[4]` 的基地址。

从上面的分析可知，恶意数据覆盖的是 `TIFFFetchShortPair()` 的栈帧结构。只要覆盖了 `TIFFFetchShortPair()` 的返回地址，让其跳转到我们的 shellcode 执行就可以想做你想做的任何事了。实际覆盖情况如下图：

`TIFFFetchData()` 函数执行前堆栈情况:
![](img/662a961215d36daf6580304c81c7be5e.jpg)

`TIFFFetchData()` 函数调用 `memcpy()` 函数后的堆栈情况，与上图相互比较，可以看出返回地址被修改了:
![](img/88cd292a50f84257949b7cd8f4edc74b.jpg)

调试到这里，产生了一个疑问：此漏洞和 CVE-2006-3459 如出一辙，而且 `libtiff` 早已通过增加判断修复了此漏洞，为什么 Adobe 公司当初没有修复此漏洞呢？

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || `poc.rar`[解压密码：pediy] |
