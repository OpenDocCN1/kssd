
# 文件格式之 PDF

> 原文：[`www.pediy.com/kssd/pediy11/104633.html`](https://www.pediy.com/kssd/pediy11/104633.html)

【文章标题】： 文件格式之 PDF

【文章作者】： youstar

【作者邮箱】： youstar@foxmail.com

【个人主页】： http://www.youstar.org.ru

【作者声明】： 只是一些很浅的认识，希望大家一起讨论。

【文章下载】： 代码与文章见附件

参考了网上的很多知识，作为自己整理学习，不足之处请大家理解与讨论。

## 目录

1、PDF 格式分析
1.1 header
1.2 Obj
1.3 cross reference table
1.4 trailer
1.5 文件尾
1.6 Obj 对象
2、VC 生成 PDF
2.1 前期准备
2.2 源码及效果
3、参考资料

## 正文

### 1、PDF 文件分析

PDF 文件主要包含以下几种结构：

#### 1.1 header

PDF 文件的第一行，格式如下：

`%PDF-1.1` //表示当前文件的版本为 1.1（目前最高版本是 1.7）

#### 1.2 Obj 文件体

PDF 中用到的对象，包括文本\图象\音乐\视频\字体\超连接\加密信息等，格式如下：

例：

```
1  0  obj   
//1 标示这个 obj 的序号，便于在 xref 中查找，0 是用来标示不同的 obj 现在都是 0，obj 是关键字
<<     // 从这里开始各行就是 obj 的属性了 以“/关键字 值”的形式来出现
/Type /Catalog           //obj 的类型是目录 catalog
/Outlines 2 0 R          //第二个 obj 是 outlines
/Pages 3 0 R             //第三个 obj 是 Pages
/OpenAction 7 0 R      // OpenAction 在第 7 个 obj
>>
endobj                       //一个 obj 结束
```

#### 1.3 cross reference table

```
xref   
  0   5   
  0000000000   65535   f   
  0000000009   00000   n   
  0000000074   00000   n   
  0000000120   00000   n   
  0000000179   00000   n 
```

`xref` 是 obj 的索引表，用来索引各个 obj 在文档中的起始位置，它的形式如下图，其中 0,211 分别代表这个 xref 的 obj 的起始和终止序号。下面的各行就代表各个 obj 在这个文档的起始位置，其中第一串字符（10 个）代表起始位置，中间的五个字符也是为了区分用的，只有当这个对象被删除的时候才有用,表示这个对象被删除后又被重新生成后的对象号，现在基本上全为 0。 后面的字母有两种：f 代表删除的，n 代表要使用的。

#### 1.4 trailer

```
trailer   
  <<   
  /Size   8   
  /Root   1   0   R   
  >>   
```

后面的 `trailer` 是对整个 `xref` 的摘要，说明里面有多少个 OBJ，读的时候从哪个 OBJ 开始解析，信息节点等，其中：

`/size` :这个 pdf 中总共使用了多少个对象  
`/root` :这个 pdf 文件的 catalog 对象的对象号,这是 pdf 中最顶层的对象，即 PDF 是组织对象号，此例中位 1 0 R

#### 1.5 文件尾

```
startxref   
  553   
  %%EOF  
```

因为一个文档中可以有多个 `xref`，所以这里要指明要从哪个 `xref` 开始进行解析这个文件。

`/startxref`: 后面的数字表示 `cross reference table` 的开始位置，里面的数字是 10 进制。

`/%%EOF` :文件结束符.

#### 1.6 OBJ 对象

Pdf 的 OBJ 中的对象有如下几种，现介绍如下：

(1)文件描述对象，用来描述这个文件的标题，作者，时间等
(2)组对象，也就是文档内容的起始结点。Type 为 Catalog
(3)页集合，里面聚合了大量的页对象。 Type 为 pages
(4)页对象，里面指明了当前页里面用到的字体，内容，活动对象，图片等。Type 为 page
(5)活动对象，有链接，文字，声音，电影等，Type 为 Annots
(6)图片对象， Type 为 XObject
(7)字体对象，Type 为 Font 。PDF 可以内置字体，所以即使目标机器上没有，只要它内置了，也可以正常地显示，不受影响。
(8)流对象。所有的二进制内容都是存在这个对象里面，文件流的常见的压缩方式是：图片的一些压缩算法，FlateDecode,ASCIIHexDecode,ASCII85Decode 等等，FlateDecode 事实上就是 ZIP 的压缩算法。流对象的长度是关连到一个长度对象上的`/Length  188 0 R` 这种形式。188 就是存这个长度的 OBJ 的顺序号。
(9)数字对象。例如：

```
188 0 obj
2538
endobj
```

里面只有一个数字，经常用来表示长度。

### 2、VC 生成 PDF

PDFLib 是用于创建 PDF 文档的开发库，提供了简单易用的 API，隐藏了创建 PDF 的复杂细节且不需要第 3 方软件的支持。以 VC 编辑一个控制应用台程序为例来生成一个 PDF。

#### 2.1 前期准备

先到 PDFLib 官网下载库文件，最新版为 pdflib8，但是生成的 pdf 有一个官网的网址，5.0 版本以下没有，链接地址：[`www.pdflib.com/download/pdflib-family/pdflib-8`](http://www.pdflib.com/download/pdflib-family/pdflib-8)建立工程后，将下载库中 `pdflib.h`,`pdflib.dll`,`pdflib.lib` 拷贝到工程目录。

#### 2.2 源码

参照官网提供的一些 sample，改编了一个，源代码如下：

```
#include "stdafx.h"
#include "pdflib.h"
#pragma comment(lib,"pdflib.lib")
int main()
{
  PDF *p;
   /* create a new PDFlib object */
    if ((p = PDF_new()) == (PDF *) 0)
    {
        printf("Couldn't create PDFlib object (out of memory)!\n");
        return(2);
    }
PDF_TRY(p)
  {
    /* This means we must check return values of load_font() etc. */
    PDF_set_parameter(p, "errorpolicy", "return");
    
    if (PDF_begin_document(p, "pediy.pdf", 0, "") == -1) 
    {
      printf("Error: %s\n", PDF_get_errmsg(p));
      return(2);
    }  
//     /* This line is required to avoid problems on Japanese systems */
//     PDF_set_parameter(p, "hypertextencoding", "host");
    PDF_set_info(p, "Creator", "pediy.c");
    PDF_set_info(p, "Author", "youstar");
    PDF_set_info(p, "Title", "Hello, pediy (C)");  
    PDF_begin_page_ext(p, a4_width, a4_height, "");  
    /* Change "host" encoding to "winansi" or whatever you need! */
    font = PDF_load_font(p, "Helvetica-Bold", 0, "host", "");
    if (font == -1) 
    {
      printf("Error: %s\n", PDF_get_errmsg(p));
      PDF_delete(p);
      return(2);
    }  
    PDF_setfont(p, font, 24);
    PDF_set_text_pos(p, 50, 700);
    PDF_show(p, "pediy");
    PDF_continue_text(p, "2010 better!");
    PDF_end_page_ext(p, "");  
    PDF_end_document(p, "");
    }
    PDF_CATCH(p) {
        printf("PDFlib exception occurred in hello sample:\n");
        printf("[%d] %s: %s\n",
      PDF_get_errnum(p), PDF_get_apiname(p), PDF_get_errmsg(p));
        PDF_delete(p);
        return(2);
    }
```

效果如下：

![](img/d0ef41af3d4a36691d4acaaf10cbfec1.jpg)

### 3、参考资料

【1】PDF 格式详解 http://blog.csdn.net/bobob/archive/2006/05/23/751381.aspx
【2】PDF 文件格式 http://ironsoft.cnblogs.com/archive/2006/01/05/311467.html
【3】PDFLib8 下载地址： http://www.pdflib.com/download/pdflib-family/pdflib-8

附件下载：

1、本文文章：文件格式之 PDF.pdf
2、工程源码：pediy.rar
3、库文件：pdflib.rar
