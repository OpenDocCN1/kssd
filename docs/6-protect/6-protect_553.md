

# 【专题六】软件保护壳技术专题

> 原文：[`www.pediy.com/kssd/pediy10/66575.html`](https://www.pediy.com/kssd/pediy10/66575.html)

壳专题  
****1 ****壳的原理****  
  
****1.1 ****必要的知识****  
  
****1.1.1 ****汇编语言教程****  
  
没有风的汇编语言专题，简短实用。可以省下不少汇编教程书籍的银子了。  
[`bbs.pediy.com/showthread.php?t=64480&tcatid=73`](http://bbs.pediy.com/showthread.php?t=64480&tcatid=73)  
看雪论坛上有很多关于汇编的话题足够初学的朋友学习了  
利用 GOOGLE 很方便的检索一下  
site:pediy.com 汇编  
site:pediy.com intitle:汇编  
site:pediy.com incontent:汇编  
以上几句在 google.com 提交会列出很多。  
  
汇编其实是最简单的编程语言。只要弄清楚 8 个常用的寄存器使用方式，弄清楚一个栈结构.弄清楚 10+条指令，最后弄清楚地址和内存就可以写程序了。其中要记忆的东西非常少。至于什么标志寄存器和中断什么的。在写程序当中熟悉就可以了。  
而要学习任何高级语言,要记住的东西绝对不止这些。  
个人很喜欢 TASM。但是 BORLAND 一直没有后文了。为了能适应潮流不得不换了 MASM。  
****1.1.2 ****PE 文件格式分析****  
  
PE 结构方面，看雪论坛上有很多精彩的帖子。这里收集一下。  
[`bbs.pediy.com/archive/index.php?t-16568.html`](http://bbs.pediy.com/archive/index.php?t-16568.html)  
[`bbs.pediy.com/showthread.php?s=&threadid=22892`](http://bbs.pediy.com/showthread.php?s=&threadid=22892)  
[`www.pediy.com/document/PE/pecoff\_v8.rar`](http://www.pediy.com/document/PE/pecoff_v8.rar)  
[`www.pediy.com/document/PE.rar`](http://www.pediy.com/document/PE.rar)  
[`bbs.pediy.com/showthread.php?t=66180`](http://bbs.pediy.com/showthread.php?t=66180)  
(这本电子书可以做为 PE 格式进阶读物)  
  
同样在看雪论坛上查找  
site:pediy.com PE 结构  
会列出更对的对 PE 结构的讨论，这些足够初学的朋友学习。  
****1.2 ****对 PE 文件的操作****  
  
**[**1.2.1 **](http://bbs.pediy.com/showthread.php?p=467116)**[添加一个新节](http://bbs.pediy.com/showthread.php?p=467116)**  
**[`bbs.pediy.com/showthread.php?p=467116`](http://bbs.pediy.com/showthread.php?p=467116)  
  
**[**1.2.2 **](http://bbs.pediy.com/showthread.php?t=67098)**[加密引入表](http://bbs.pediy.com/showthread.php?t=67098)**  
**[`bbs.pediy.com/showthread.php?t=67098`](http://bbs.pediy.com/showthread.php?t=67098)  
  
[****1.2.3 ****处理 TLS 表****](http://bbs.pediy.com/showthread.php?t=67627)  
[`bbs.pediy.com/showthread.php?t=67627`](http://bbs.pediy.com/showthread.php?t=67627)  
  
**[**1.2.4 **](http://bbs.pediy.com/showthread.php?t=68262)**[加密资源节](http://bbs.pediy.com/showthread.php?t=68262)**  
**[`bbs.pediy.com/showthread.php?t=68262`](http://bbs.pediy.com/showthread.php?t=68262)  
  
**1.2.5 处理引出表**  
[`bbs.pediy.com/showthread.php?t=80406`](http://bbs.pediy.com/showthread.php?t=80406)  
  
****1.3 ****壳主体的编写****  
  
****[1.3.1 反静态反汇编技术](http://bbs.pediy.com/showthread.php?t=68486)**  
**[`bbs.pediy.com/showthread.php?t=68486`](http://bbs.pediy.com/showthread.php?t=68486)  
  
****[1.3.2 反调试器技术](http://bbs.pediy.com/showthread.php?t=71113)**  
**[`bbs.pediy.com/showthread.php?t=71113`](http://bbs.pediy.com/showthread.php?t=71113)  
  
**[1.3.3 反汇编引擎的构建](http://bbs.pediy.com/showthread.php?t=74414)**  
[`bbs.pediy.com/showthread.php?t=74414`](http://bbs.pediy.com/showthread.php?t=74414)  
  
**[1.3.4 重定位模块的构建](http://bbs.pediy.com/showthread.php?t=77514)**  
[`bbs.pediy.com/showthread.php?t=77514`](http://bbs.pediy.com/showthread.php?t=77514)  
  
**[1.3.5 PE Loader 的构建](http://bbs.pediy.com/showthread.php?t=83669)**  
[`bbs.pediy.com/showthread.php?t=83669`](http://bbs.pediy.com/showthread.php?t=83669)  
  
**[1.3.6 变形引擎的构建](http://bbs.pediy.com/showthread.php?t=95879)**  
[`bbs.pediy.com/showthread.php?t=95879`](http://bbs.pediy.com/showthread.php?t=95879)  
  
**[1.3.7 代码乱序引擎的构建](http://bbs.pediy.com/showthread.php?t=96640)**  
[`bbs.pediy.com/showthread.php?t=96640`](http://bbs.pediy.com/showthread.php?t=96640)  
  
**[1.3.8 代码替换引擎的构建](http://bbs.pediy.com/showthread.php?t=96644)**  
[`bbs.pediy.com/showthread.php?t=96644`](http://bbs.pediy.com/showthread.php?t=96644)  
  
**[1.3.9 再谈引入表保护](http://bbs.pediy.com/showthread.php?t=101394)**  
[`bbs.pediy.com/showthread.php?t=101394`](http://bbs.pediy.com/showthread.php?t=101394)  
  
**[1.3.10 识别函数与数据区域](http://bbs.pediy.com/showthread.php?t=102977)**  
[`bbs.pediy.com/showthread.php?t=102977`](http://bbs.pediy.com/showthread.php?t=102977)  
  
**[1.3.11 使用数学运算式随机生成花指令](http://bbs.pediy.com/showthread.php?t=103229)**  
[`bbs.pediy.com/showthread.php?t=103229`](http://bbs.pediy.com/showthread.php?t=103229)  
  
**[1.3.12 构建壳使用的 x86 汇编器](http://bbs.pediy.com/showthread.php?t=103757)**  
[`bbs.pediy.com/showthread.php?t=103757`](http://bbs.pediy.com/showthread.php?t=103757)  
  
**1.3.13 虚拟机保护技术**  
  
****2 ****壳源码分析****  
  
****2.1 ****yC1.3 代码分析****  
  
****2.2 ****UPX3.03 代码分析****  
  
****3 ****成果展示****  
  
**玩命的壳**