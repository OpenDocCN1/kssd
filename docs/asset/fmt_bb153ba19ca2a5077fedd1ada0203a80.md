
# `lib` 文件格式分析，以及从 `lib` 文件提取 `obj` 的思路和源码

> 原文：[`www.pediy.com/kssd/pediy10/63585.html`](https://www.pediy.com/kssd/pediy10/63585.html)

最近在学习 `ida`，有时候需要从 `lib` 文件里面提取 `sig`，方便我们查看函数的名称。但是手工的办法只能一个个 `obj` 提取，虽然可以使用批处理，但是偶尔碰到错误的 `coff` 文件，还是需要人手工干预，所以写了这么一个小工具，顺带学习了一下 `lib` 文件的结构。

`lib` 文件最开头，由 8 个字符串开始，值为“`!<arch>\n`”。接下来由 4 个不同类型的区段组成（不是四个区段），依次排列下来是 `First Section`（一个）, `Second Section`（一个）, `Long Section`（一个）, `Obj Section`（包含有多少个 `obj` 文件，就有多少个 `obj` section）。

区段头都是以这样的一个结构体开始。

```c
typedef struct  
{  
  char Name[16];      // 名称  
  char Time[12];      // 时间  
  char UserID[6];     // 用户 ID  
  char GroupID[6];    // 组 ID  
  char Mode[8];       // 模式  
  char Size[10];      // 长度  
  char EndOfHeader[2];// 结束符  
} SectionHeader;  
```

要从文件提取 `obj` 文件，我们一般只需要关注第二个区段。这个里面保存了如下的信息：

```c
typedef struct 
{ 
  unsigned long ObjNum;        // Obj Sec 的数量 
  unsigned long ObjOffset[x];  // 每一个 Obj Sec 的偏移 
  unsigned long SymbolNum;     // 库中符号的数量 
  unsigned short SymbolIdx[n]; // 符号在 ObjOffset 表中的索引 
  char StrTable[m];           // 符号名称字符串表 
} SecondSec; 
```

但是我们却不能直接去读取他，因为

`ObjOffset[x];  // 每一个 Obj Sec 的偏移`

长度是不固定的，取决于

`ObjNum;        // Obj Sec 的数量`

而且，`first section` 的长度也是不固定的。这怎么办呢？

我的解决办法如下：

`first section` 和 `second section` 都以

```c
char szSecName[16] = {'/', 0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20,  0x20,0x20, 0x20, 0x20, 0x20, 0x20, 0x20, 0x20};
```

这样的形式开头。

我就定义了一个 `char`，一个个字节去读取，遇到`'/'`就判断后面是不是 15 个 `0x20`，如果读了 2 次这样的结构，那么说明到了 `second section`，马上读后面的内容。这就是 `ObjNum`，也就是 `obj` 的数量。

然后去申请一块内存，用来存储 `ObjOffset[x];  // 每一个 Obj Sec 的偏移`

`ULONG* m_offset = new ULONG[ObjNum]`，然后从文件里面将 `obj` 偏移读取到这个数组。

请注意，这些偏移值，并不是由小到大排列的，所以得自己排列一次。

用第二个偏移减去第一个偏移，得到的就是第一个 `obj` section 的大小（不是 `obj` 文件的大小）。因为每个 `obj` section 前面还有一个 `SectionHeader` 结构体，所以 `obj` 文件真正的偏移是：

读出来的偏移 `offset + sizeof(SectionHeader)`，从这里开始读，才是 `obj` 文件的首地址。

下面的事情就简单了，一点点的写到各个文件，即可。

`From:武汉科锐`

代码在下面，没有使用多线程，所以 `obj` 文件过多的时候，运行的时候会稍慢，嘿嘿。有不正确的地方，或者有更好的算法，请多多指教！

[`img.jpg.name/twdwsdtjztwwthhy...wzwtrwtrrj.rar`](http://img.jpg.name/twdwsdtjztwwthhyztswytttwrrwzwtrwtrrj.rar) 

![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸") 多谢分享，刚好要用到，转一份本地

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `jpg.name_219139254211_2008-4-23_18-22-21_6614126_12665.rar`  
|
