
# [原创]我眼中的数据对齐

我眼中的数据对齐

作者：溟初

声明：这篇文章是用word写成的，里面含有一些表格、流程图什么的，不易贴出，在此仅列一下目录，感兴趣者可以下载察看！

在看雪论坛关于`memcpy`的帖子（[`memcpy`应该怎样写，同时庆祝新版开张](http://bbs.pediy.com/showthread.php?threadid=14128)）中谈及了数据对齐的话题，重新唤起了我对它的思考（以前一直都似懂非懂，逐渐淡忘了^\_^），纯属个人见解，请批评指正！

## 一、什么是数据对齐？请看官方的解释：

## 二、数据对齐随处理器和编译器的不同而不同，处理好数据对齐可以提高程序的时空效率（节省空间，提高效率），特别是在汇编程序中。

### 1、在处理器方面：

### 2、在编译器方面（以微软的 `ml`、`link` 为例）：

- （1）全局变量：
- （2）函数参数和局部变量：
- （3）结构体字段对齐（在移植程序时需要特别考虑，写网络程序好像也很重要）：
- （4）指令对齐：
- （5）段的对齐类型：
- （6）PE文件中的 `SectionAlignment`、`FileAlignment`：

## 三、尝试分析 `\VC98\CRT\SRC\PLATFORM\memcpy.asm`（VC6.0安装盘上，vc7.0在安装盘 `crt\src\intel\` 下，你也可以从网上找到）

### 1、覆盖的情况：`dst>src && dst < src + len` 时，如下图：

### 2、`memcpy.asm` 代码分析。

## 四、结束语：

这篇心得终于完成了，感觉自己的表达能力还是很有限，请批评指正，我的e_mail：hejiwen2001@sohu.com，如果它能给你带来帮助，我将很欣慰！，至少没有白忙活J

## Reference

1、《The art of assemle language》    来自 c11K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4K6L8g2)9J5k6i4W2W2j5h3S2Q4x3X3g2F1k6i4c8Q4x3V1j5`.

2、《windows核心编程》                来自 d11K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3W2F1k6X3!0^5j5g2)9J5k6h3y4G2L8g2)9J5c8R3`.`.

3、《INTEL 80386 PROGRAMMER'S REFERENCE MANUAL 1986 》    来自 54eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8#2M7X3g2U0i4K6u0W2j5X3W2F1k6$3S2#2j5g2)9J5k6h3y4G2L8g2)9J5c8R3`.`.

4、cf1K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2V1M7X3g2S2L8h3u0J5L8$3!0C8i4K6u0W2j5$3!0E0i4K6u0r3M7$3!0D9L8$3W2K6N6q4)9J5c8X3q4J5j5$3S2A6N6X3g2Q4x3V1j5J5x3o6l9@1i4K6u0r3x3e0u0Q4x3V1j5I4x3W2)9J5c8U0x3^5z5q4)9J5k6h3q4K6M7s2R3`.

5、574K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6F1j5$3A6Q4x3X3g2$3K9h3y4H3i4K6u0W2L8X3g2@1i4K6u0r3j5$3!0#2M7Y4y4W2i4K6u0r3K9r3g2H3i4K6u0r3K9s2g2A6j5X3W2S2L8Y4W2#2P5h3q4F1i4K6u0r3x3o6c8Q4x3X3b7K6i4K6u0W2K9s2c8E0

6、8f6K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2K6k6r3&6Q4x3X3g2E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6D9K9h3u0J5j5i4u0&6

还有其他一些相关的网络资源，在此一一谢过[附件:AlignInMyOpinionPDF.rar](https://bbs.pediy.com/upload/2005/41/files/aligninmyopinionpdf.rar)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2018-8-6 09:29 被kanxue编辑，原因：
