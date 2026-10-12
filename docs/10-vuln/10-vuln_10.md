
# Perl 黑客编程

> 原文：[`www.pediy.com/kssd/pediy10/103566.html`](https://www.pediy.com/kssd/pediy10/103566.html)

作者：riusksk(泉哥)  
主页：http://riusksk.blogbus.com  
邮箱：riusksk@qq.com  
声明：我承认这标题有点拉风，如果你看它不爽，可以改成《Perl exploit programming》  
本文已发表于《黑客防线》

## 目录

一． Perl 简介  
二． Windows 下的 Perl 环境搭建  
三． 基础知识  
四． 数组  
五． 条件与循环语句  
六． 用户数据获取  
七． 文件读写  
八． 子程序  
九． LWP 模块  
十． 套接字 Socket  
十一． 编写攻击代码  
（1） XSS exploit  
（2） SQL Injection Exploit  
（3） 远程文件包含  
（4） LFI2RCE Exploit  
（5） 本地溢出  
（6） 远程溢出  
十二． 结论

## 一． Perl 简介

Perl 是 Practical Extraction and Report Language 的编写，它是由 Larry Wall 设计的，主要用于 UNIX 环境下编程，当然目前它也是只支持 Windows 系统的，本文主要也是讲在 Windows 下的 Perl 编程，并侧重于黑客安全方面。Perl 如脚本语言一般，无需经编译器来编译才能运行代码，应用起来也很简便，特别是用它来写 exploit 尤为流行。据统计，目前世界上有 70%的 exploit 是用 Perl 来编写的。在国内，由安全焦点出版的《网络渗透技术》一书提到的主要编程语言中，主要有 perl,c,asm 三种，书中提到在阅读此书的预备知识中就要求熟悉 C，Perl,Asm，由此可见,perl 语言在国内安全方面的应用也是占有一席之地的。在国外著名漏洞公布站点 milw0rm 中，我们也可以经常看到用 perl 编写的 exploit，无论是在脚本攻击中，还是在系统漏利用中，特别是在溢出攻击中，都可以经常见到它的身影。除此之外，有些安全人士也用它来编写 Fuzzing 工具，用于挖掘漏洞。另外，本文讲述的只是 Perl 语言中的冰山一角，如果读者想深入学习该语言的话，建议学习一下官方文档或者其它编程书籍。

附件下载：

上传的附件

| ![](img/2674941de574f12fa6a3e90d2eb3468a.jpg) || Perl.doc |
