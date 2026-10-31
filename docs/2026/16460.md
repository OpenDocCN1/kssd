
# [分享]浅谈如何学习linux

## 一. 为什么要学linux?

当然最重要是爱好和兴趣！如果你这种必要学，或者根本不喜欢，请不要浪费时间，你学也学不好！

## 二. 起步

你应该为自己创造一个学习linux的环境--在电脑上装一个linux或unix

如何选择版本：北美用`redhat`,欧洲用`SuSE`,桌面`mandrake`较多，而`debian`是技术最先进的linux开发人员中用`debian`的最多，其次是`redhat`,从全球linux各应用领域市场份额来看无疑`redhat`是最多的，此外还有很多出名的发行版本，不再列举。

对于初学linux的人来说，我建议是使用`redhat`,原因如下：

1）现在很多书都是以`redhat`为例讲的，为了与书本协调一致

2）周围的人都用`redhat`,交流比较方便

3）`redhat`应用范围广，有典型性和代表性

4）它易于使用和安装，我们没有必要把时间浪费在“装系统”上而应集中精力学习最有用的东西。

【注】现在觉得RH很死板，AS,ES等用在服务器上或许不错，Personal desktop用`mandrake`,`debian`,`suse`都不错，笔者现在用`Mandrake`,因为她长得漂亮:)

如果你并不打算深入学习linux，而是有诸如适应北京市政府办公平台迁移到linux上这种需要，那么中软，红旗等中文linux是不错的选择

我强烈建议：自己亲自动手把linux装到你的硬盘上，你必须学会独立安装linux系统的技能，对于现在的版本来说，其实跟装WinXP一样简单

从此现在开始，请不要以windows的工作方式来考虑问题，应该尝试挖掘linux身上的“天才unix”的气质。

## 三. 进阶

掌握至少50个以上的常用命令

理解shell管道"`|`",文件流重定向"`>`"及追加"`>>`"等

熟悉`Gnome`/`KDE`等X-windows桌面环境操作

掌握`.tgz` `.rpm` `.biz`等软件包的常用安装方法

学习添加外设，安装设备驱动程序（比如modem)

熟悉`Grub`/`Lilo`引导器及简单的修复操作

熟悉系统固有目录的名称及公用

学会用`mount`命令访问其他文件系统

了解`vi`,`gcc`,`gdb`等常用编辑器，编译器，调试器

学习linux环境下的简单组网

建议：买一本不需要太厚的linux教材，大致可以满足要求（现在的书越来越多了，还带很多图，我当时可没这么多书）

## 四. 高级应用

澄清一些概念：linux的普通操作与真正的系统管理不能相提并论，后者需要很多知识

### 1. 英语

即使你不学linux，我也强烈建议你学好英文[U.S.english]。因为实质上计算机语言就是英文和字符,所谓的多国语言只是外部包装。你必须能无障碍的阅读大量的英文技术文档在搜索引擎找到的英文网站和网页中熟练的检索最好能有用英文直接交流的能力，摆脱了这个障碍，你的学习和理解速度就能快很多，你就有机会拉开和别人的差距。

### 2. shell[sed/awk]

`shell`是命令解释器，是内核与用户界面交流通道，`shell`写的小脚本有点类似于win下的`.bat`但`shell`比`.bat`强大的多，`shell`不只是解释命令，更是一种编程语言，有时候几百行的c用`shell`几十行就能代替完成工作，因为`shell`的工作方式建立在系统已有的众多应用程序之上

这也是CS中的一个重要思想。此外，`shell`可以实现工作自动化，这个概念也比较重要`sed`,`awk`用来处理文本，历来很常用。

### 3. Perl/PHP

漂亮的脚本， CGI的首选，比ASP好，应用面很广

### 4. C\C++

`C`、`C++`是linux/unix的核心语言，系统代码都是C写的

### 5. ASM

系统底层及内核，硬件，设备驱动程序，嵌入式开发都需要。//走核心路线的话，`c\c++`,`asm`最重要

### 6. Java,Python,Tcl,XML

## 五、系统管理篇

在熟悉linux的基础上还需要掌握至少一种unix。我首推`Solaris`，其次是`FreeBSD`比如运营级系统一般是`Solaris+Oracle/DB2`之类的学习`apache`、`ssh`、`sendmail/Qmail`、`proftp/vsftp`、`Samba`、`Squid`、`MySQL/PostgreSQL/Oracle`、`Bind`等各种应用服务器的构架及电子商务的应用熟悉`TCP/IP`协议族，学习诸如`apache+php+proftp+mysql+quota`的实现以及大型局域网，分布式集群等各种企业级应用解决方案熟悉多用户管理，数据库管理，文件系统，逻辑存储管理，日志分析，备份与灾难数据修复系统补丁，内核升级，以及在此基础上的防火墙构架等以保障系统安全在内的各种系统管理技能。

我觉得，如果在此基础上再掌握路由/交换设备便是一个不错的系统管理员

各种基于linux的解决方案可参考相关书籍和文献，必要时用google或各大linux站点站内搜索引擎寻找最新文档，以避免错误和漏洞有几本技术大全和技术内幕我认为都是这方面不错的书

## 六、深入学习linux

我个人理解的读linux内核需要的基础：在此之前,希望先把应用层的东西学一下,那样会比较好理解

1. `C`

如果学过潭浩强的大学教科书(除了编几个数学模型好像什么也做不了的那种)，建议再看一下《The C Programming Language》Second Edition这本圣经（如果想学缓冲区溢出，这点C的功力可能是不够的，还有<C陷阱与缺陷>,<C专家编程>,<C\C++深层探索>，反正经典书看多了是没有坏处的）

2. `asm` (AT&T语法，保护模式)

保护模式下的比较复杂，基本上每本讲内核的书都会有介绍有80x86 Intel语法的基础就行，

有兴趣可以看看Intel的官方网站

3. 数据结构（离散数学）

计算机专业的核心课程，重要性我就不说了

4. 操作系统原理

看懂这个再去读linux吧

5. 微机原理/组成原理（数字电路）

底层直接和硬件打交道，所以这个也要

6. 了解linux/unix

我想至少你要会操作吧，了解unix的API

7. 软件工程/编译原理

这方面的经典书去003K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4Z5K9h3&6S2i4K6u0V1M7s2g2T1i4K6u0W2j5$3!0E0i4@1f1#2i4K6S2r3i4@1q4r3i4@1f1@1i4@1u0n7i4@1p5#2i4@1f1$3i4K6V1H3i4K6W2o6i4@1f1#2i4K6R3^5i4@1t1H3i4@1f1#2i4@1u0q4i4K6R3^5i4@1f1#2i4@1p5@1i4K6W2m8i4@1f1$3i4K6R3^5i4K6V1I4i4@1f1^5i4@1p5%4i4K6R3&6i4@1f1#2i4@1u0q4i4K6V1%4b7g2m8g2c8g2!0q4z5q4!0n7c8W2)9&6z5g2!0q4y4#2!0m8y4#2)9^5c8q4!0q4y4q4!0n7z5g2!0m8y4W2!0q4y4q4!0n7z5q4)9^5c8q4!0q4z5g2)9^5x3q4)9^5x3W2!0q4y4g2)9&6x3q4)9^5z5q4!0q4y4g2)9^5z5q4)9&6c8q4!0q4y4g2!0m8c8q4!0m8y4W2!0q4z5q4)9^5x3q4)9^5y4g2!0q4c8W2!0n7b7#2)9^5b7#2!0q4z5q4!0n7c8W2)9&6z5q4!0q4y4W2)9&6z5q4!0m8c8W2!0q4y4g2)9^5y4g2)9^5z5q4!0q4y4#2)9&6b7#2)9^5b7W2!0q4y4#2)9&6b7#2)9^5b7W2g2b7c8g2!0q4y4q4!0n7z5g2)9^5b7W2!0q4y4#2!0n7x3g2!0n7b7W2!0q4y4#2)9&6b7g2)9^5y4q4!0q4y4g2)9&6x3q4!0m8y4#2!0q4x3#2)9^5x3q4)9^5x3R3`.

`APUE`: 《advanced　programming　in　the　unix　environment》，中文名《unix环境高级编程》，作者：W.Richard.Stevens（共有6本经典），尊为“圣经”。他的书为全世界黑客所拜读！

`UPE`:《unix　programming　environment》，中文名《unix编程环境》比较适合初学者的，深入浅出，其中有一些比较重要的思想。

另外,介绍linux下应用层编程的书也有很多,比如`<GNU\Linux编程指南>`,redhat,redflag出的书...........

经典书还有很多很多......

《操作系统：设计与实现》（交大的考研参考书目，讲Minix的unix操作系统设计)、4.4BSD操作系统设计与实现、昂莱氏unix源代码分析、Understanding the linux kernel (2nd)深入理解linux内核、linux设备驱动程序 (2nd)、linux内核源代码情景分析及《保护方式下的80386及其编程》(次书已绝版,只有超星的电子版,如果谁能搞到纸书,清一定告诉我一下,非常感激!!!)

在这里你能明白"黑客"及"haching"一词所代表的真正意义.希望我写的这篇文章能造就更多的黑客，而不是骇客

## 七、一些资源

82fK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3I4A6L8Y4g2^5k6X3!0J5N6h3#2Q4x3X3g2F1k6i4b7`. // 国内最高水平GNU站点

633K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4Z5K9h3&6S2N6h3&6A6P5q4)9J5k6h3&6W2N6q4)9J5y4X3&6T1M7%4m8Q4x3@1u0Q4x3U0k6F1j5Y4y4H3i4K6y4n7i4K6u0r3i4K6u0r3 unix、系统管理的不错

795K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3I4A6L8Y4g2^5k6h3c8W2L8W2)9J5k6h3y4G2L8g2)9J5y4X3&6T1M7%4m8Q4x3@1u0Q4x3U0k6F1j5Y4y4H3i4K6y4n7i4K6u0r3i4K6u0r3 下载软件不错

21bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3I4A6L8Y4g2^5j5h3W2V1i4K6u0W2j5$3!0E0i4K6u0W2j5$3^5`.

aa3K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3I4A6L8Y4g2^5M7$3W2J5i4K6u0W2L8%4u0Y4

其实还有很多，包括国外的

欢迎访问我的博客:[a1fK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0A6L8r3I4K6N6r3!0F1k6g2)9J5k6h3y4#2j5X3I4G2k6#2)9J5k6h3y4F1](https://bbs.kanxue.com/elink@c58K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0A6L8r3I4K6N6r3!0F1k6g2)9J5k6h3y4#2j5X3I4G2k6#2)9J5k6h3y4F1)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
