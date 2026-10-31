
# [原创] 如何分析一个stripped ELF？这款IDA插件来帮忙

![](upload/attach/202212/906228_P8YN4WEPQQ6QUCD.jpg)

这两天有闲心了就把去年写的几个脚本整合成了一个 IDA 插件，希望对在座各位有所帮助。

## 背景

分析 IOT 样本时经常会遇见被剥离符号的样本文件，此时如果直接进行分析则非常困难，所以需要还原其函数来帮助我们进行分析。

这里放两张图对比使用插件前后，还原前甚至可以说是完全无法进行分析的：

![](upload/attach/202212/906228_CE73DN3EDMU586A.jpg)

![](upload/attach/202212/906228_TVTX53KM2C3H23X.jpg)

## 原理

`SysNR-FuncFinder` 这款 IDA 插件使用的方法是通过 Linux System Call Table 进行还原（此处不进行详细讲解了，大家可以自行搜索 system call 的相关文章）

简单来说是分析不同架构的汇编代码，然后使用 `IDA Python` 提取系统调用号，最后对应不同架构调用号的对应函数名进行 IDA 函数的重命名即可。

## 示例

举例样本 `8D5B7B806E35F6AE0BA02E9CAB4C3E5F`，Linux 下使用 `file` 命令查看文件信息如下：

![](upload/attach/202212/906228_3YM4DURH8PHEE78.jpg)

拖进 IDA 查看可以发现，完全没有可读函数：

![](upload/attach/202212/906228_CE73DN3EDMU586A.jpg)

使用编写的插件可以自动依照框架对系统调用函数进行重命名（`version 1.0` 版本支持 4 种主流框架）所有通过 `NR` 调用的系统函数都能进行还原，此时再进行分析可以说是如鱼得水：

![](upload/attach/202212/906228_UYXDRPQ7XJKEWNW.jpg)

## 下载

大家直接进 github 下载即可，有兴趣的也可以阅读一下代码。

**[Github 链接](https://bbs.kanxue.com/elink@4c6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6i4f1r3g2S2j5$3g2Q4x3X3c8t1j5@1S2Q4x3V1k6e0P5i4y4z5f1W2)9J5k6p5k6#2L8X3y4r3K9h3&6V1k6i4t1`. )**

PS:（`IDA >=7.4` and `python3`）

![](https://bbs.kanxue.com/upload/attach/202212/906228_P8YN4WEPQQ6QUCD.jpg)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#病毒木马](https://bbs.kanxue.com/forum-4-1-6.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
