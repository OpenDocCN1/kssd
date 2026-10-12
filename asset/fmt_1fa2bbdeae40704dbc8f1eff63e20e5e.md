
# PDF 文件分析辅助工具

> 原文：[`www.pediy.com/kssd/pediy11/125416.html`](https://www.pediy.com/kssd/pediy11/125416.html)

## 基本信息
题目：PDF 文件分析辅助工具  
写者：Cryin'  
操作系统：Windows XP  
开发平台：VC++ 6.0

## 概述
看到国外的一款 PDF 文件分析辅助工具 `PDF Dissector`，参考【1】。看其演示效果相当诱人。就想着自己能否也做一个 PDF 分析的辅助工具。可惜 VC 处理数据比较麻烦，再加上能力有限！所以就做一个对我个人平时分析 PDF 文件有些许帮助的小工具吧，所以就将分析 PDF 文件时一般会用到的小工具集成一体，做成一个工具。当然现在还在完善中。

## 功能介绍
这个工具的代码均为自己编写，其中有以下一些功能：

### 1. Hex 编辑器
未完善，暂且可以无视，如图：  
![](img/993ae1ffb161a6684f1f3d4b867d6983.jpg)

### 2. PE 分析工具
其中包括 PE 格式分析以及反汇编，本程序的反汇编功能均采用 Pvdasm 反汇编引擎，目前正在完善中，暂且可以无视，如图：  
![](img/ff56e4c8126cfe448c897c0215ebf1a6.jpg)  
![](img/6b9acc8a8e839a596067ab1aefe43f3c.jpg)

### 3. Shellcode 辅助工具
其中包括 shellcode 反汇编以及机器码提取，如图：  
![](img/867b16ffc4d6451dfcd42f373e52df20.jpg)  
![](img/c8902f1ab51a91971201bbb7d2f89add.jpg)

### 4. 简单的加密解密(XOR)工具
包括字节、字、双字 XOR、递减、递加 XOR 功能，并生成加密文件。如图  
![](img/6cded1d9632ef0a680dc0204a4860e24.jpg)

### 5. PDF 分析辅助工具
这也是最重要要做的工具，目前包括 PDF 文件检测、提取 OBJ、Stream、Info、JavaScript 等信息、主要是检测文件是否符合 ADOBE 规范、以及是否可能为 Exploit 文件，效果还行，待完了再添加 zlib 功能进去压缩与解压缩 JS 代码。目前正在完善中，如图  
![](img/972afecb7910bfaec4336a4d9a53a222.jpg)

## 结束语
写这个工具纯属为了误乐与学习。大牛见笑了！目前程序正在慢慢完善，暂时欢迎大家试用。如果有什么意见或建议。欢迎与我交流。  
Blog：[`hi.baidu.com/justear/blog`](http://hi.baidu.com/justear/blog)

## 下载
`PDFTear.rar`

## 参考
【1】[`www.zynamics.com/dissector.html`](http://www.zynamics.com/dissector.html)

12 月 6 日更新版：  
[`bbs.pediy.com/showthread.php?t=126125`](http://bbs.pediy.com/showthread.php?t=126125)
