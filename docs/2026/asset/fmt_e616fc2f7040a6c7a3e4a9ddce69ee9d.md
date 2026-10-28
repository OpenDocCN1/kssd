
6. Walter Oney, "Programming the Microsoft Windows Driver Model. 2nd edition", Microsoft Press, 2003（这是一本很好的书，它强调的是即插即用驱动程序的开发，但这并不降低了它的重要性，因为驱动开发方面的基本理论都是通用的）

7. Art Baker, Jerry Lozano, "The Windows 2000 Device Driver Book, A Guide for Programmers, Second Edition", Prentice Hall, 2000，这也是一本好书，选题范围和本教程类似

8. Rajeev Nagar, "Windows NT File System Internals. A Developer's Guide", O'Reilly

9. Prasad Dabak, Sandeep Phadke, and Milind Borate, "Undocumented Windows NT", M&T Books, 1999 ，这本书里面包含了n多公开的文档中找不到的东西。

10. Gary Nebbett, " Windows NT-2000 Native API Reference", MacMillan Technical Publishing, 2000,这里也有一堆公开的文档中找不到的函数和数据结构的说明

11. Jeffrey Richter, "Programming Applications for Microsoft Windows. Fourth Edition", Microsoft Press, 1999，这本书和开发设备驱动程序没什么关系，但是也是一本很有趣的书。

这里列出的并不是全部，但这些似乎都有点儿看看的必要性。

注：本章的内容均来自于上文提到的KmdTut中文版，它是由由罗云彬和刘松翻译的。

# 第二章 驱动开发环境的搭建

## 2.1 Win32Asm 简介

`WIN32`程序运行在保护模式下的,所有的 `WIN32` 的应用程序运行在一个连续、平坦、巨大的 4GB 的空间中。

这同时也意味着您无须和段寄存器打交道，您可以用任意的段寄存器寻址任意的地址空间，这对于程序员来说是非常方便的，这也使得用32位汇编语言和用C语言一样方便。

## 2.2 Win32ASM编译器

`Win32ASM`的编译器最常用的有两种：Borland公司的`Tasm`和Microsoft的`Masm`，两种编译器各有自己的优缺点，看来使用哪一种编译器还是比较难选择的，但`CCDebuger`给了我们一个答案，他为`RadAsm`(下面有介绍)整合了供`MASM`使用的很全的SDK、DDK库，这些库、include文件、学习资料和其他工具还有`Masm9.0`版本整合形成了一个非常方便的版本：`RadAsm 汉化整合版`。这样一来，我们用汇编编程就象用VC一样方便了。

因为有了`RadAsm 汉化整合版`，所以就我个人而言，我推荐使用`Masm`作为`Win32ASM`的编译工具，但`Masm`和`Tasm`的宏语法有很多的不同，我的这个是以`Masm`格式写的。

## 2.3 关于DDK

开发驱动程序一个必备的东西就是`DDK`（设备驱动开发包），它跟我们在应用层常听到的`SDK`差不多，只不过一个是应用层的，一个是内核层的。

`RadAsm 汉化整合版`里整合的实际上是被称为`KmdKit`的`DDK`。

`KmdKit`的全称是Kernel Mode Driver development Kit for assembly language programmers，即内核模式驱动程序汇编开发包，`KmdKit`中包括了用宏汇编编译器开发驱动程序需要用到的所有东西。

你可以从`495K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4N6S2M7$3#2Q4x3X3g2J5N6g2)9J5c8R3`.`.`.或者`081K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3k6J5k6h3g2%4k6h3u0K6i4K6u0W2j5$3!0E0i4K6u0r3k6X3!0#2M7W2)9J5k6r3k6Q4x3V1j5`.下载到最新版本的`KmdKit`。

网上流传了一份`KmdKit`的使用教程：`KmdTut`中文版，它是由由罗云彬和刘松翻译的，大家可以百度一下，我以后的文章中也可能会多次引用里面的内容，这里说明一下。

## 2.4 关于IDE

`Win32ASM`常用的`IDE`有`Radasm`，`MASMPLUS`，`WINASM`，`EASYCode`等，前面我们已经说了，`RadAsm 汉化整合版`整合了我们驱动开发所需要的所有东西，非常方便，这里我们就使用它来进行代码编写工作，其它的`IDE`读者可以自行搜索了解。

`RadAsm`是一个非常好用的`IDE`，它不仅支持代码高亮，输入提示，还支持编译其它语言，如`C/C++`等，使用也非常简单，全中文的，基本摸索下就会使用了，它像使用`VC`一样方便，我这里就不啰嗦了，可以百度一下或者在这里下载：

[363K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3!0F1L8r3W2F1k6h3c8G2N6$3&6Q4x3X3g2F1k6i4c8Q4x3V1k6K6L8$3k6@1i4K6u0r3x3U0j5H3z5e0N6Q4x3X3g2Z5N6r3@1`.](https://bbs.kanxue.com/elink@53aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3!0F1L8r3W2F1k6h3c8G2N6$3&6Q4x3X3g2F1k6i4c8Q4x3V1k6K6L8$3k6@1i4K6u0r3x3U0j5H3z5e0N6Q4x3X3g2Z5N6r3@1`.)

注意： `RadAsm`汉化增强版适合于未安装`Masm32`及`Viusual C++`的用户，安装路径必须是分区根目录，最好不要在系统盘,`RadAsm`的使用可以参考这篇文章：[Win32ASM可视化编程](http://bbs.pediy.com/showthread.php?t=42232&highlight=)

### 本章小结：

搭建`W32ASM`的驱动开发环境，只需要一个步骤：[下载安装RadAsm汉化增强版](https://bbs.kanxue.com/elink@d03K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3!0F1L8r3W2F1k6h3c8G2N6$3&6Q4x3X3g2F1k6i4c8Q4x3V1k6K6L8$3k6@1i4K6u0r3x3U0j5H3z5e0N6Q4x3X3g2Z5N6r3@1`.)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [Windows NT结构简图.jpg](javascript:void(0);)

（88.50kb，7次下载）
