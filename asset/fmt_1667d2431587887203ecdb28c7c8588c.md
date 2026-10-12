
# 【翻】SHELLCODE 解密

> 原文：[`www.pediy.com/kssd/pediy10/69385.html`](https://www.pediy.com/kssd/pediy10/69385.html)

原文：[`www.enderunix.org/docs/en/sc-en.txt`](http://www.enderunix.org/docs/en/sc-en.txt)

引用: 目

前序 ---------------------------------------------------------------------------------------------------------- 1

SHELLCODE 何物？------------------------------------------------------------------------------------- 2

系呼叫 ---------------------------------------------------------------------------------------------------- 3

生介程式的 SHELLCODE -------------------------------------------------------------------------- 12

後 -------------------------------------------------------------------------------------------------------- 19

-------------------------------------------------------------------------------------------------------- 19

文 -------------------------------------------------------------------------------------------------------- 19

引用: FORE WORD

## 前序

In our previous paper, Buffer Overflows Demystified, we told you that there will be more papers on these subjects. We kept our promise. Here is the second paper from the same series. The paper is about the fundamentals of shellcode design and totally Linux 2.2 on IA-32 specific. The base principles apply to all architectures, whereas the details might obviously not.

在前文「溢出解密」提到，撰更多有主的文章。者保持承，前所此系列的第二篇文章。 文有 SHELLCODE 於 IA-32 架中 Linux 2.2 系核心上的基知。 用到所有架上的基原理皆是如此，本文不再次述那些本知道的。

To understand what's going on, some C and assembly knowledge is required. Virtual Memory, some Operating Systems essentials, like, for example, how a process is laid out in memory will be helpful. You MUST know what a setuid binary is, and of course you need to be able to at least use UNIX systems. If you have an experience of gdb/cc, that is something really really good. Keep 「IA-32 Intel Architecture Software Developer's Manual Volume 1: Basic Architecture" at hand. You can get it from here.

想解接下要作什，需要一些於 C Asm 的知。 有作系的要，亦同於上。 例：程序如何作於中其分。 者必知道什是 `setuid` 二制案，然也必操作 Unix 系。 假若曾有於 `gdb`/`cc` 工具的使用，那然更好。 最後忘常注官方手：「IA-32 Intel Architecture Software Developer's Manual Volume 1: Basic Architecture」，可於下列址得手。

Recent versions of the paper can be found [here](http://www.enderunix.org/documents/en/sc-en.txt).

可於下列[址](http://www.enderunix.org/documents/en/sc-en.txt)前版本的文。

P.S. 於「溢出解密」篇的翻可能要晚，因有友先跟我要其他 Windows 的新手溢出文件的翻，麻各位大大先照[原文版](http://www.enderunix.org/docs/en/bof-eng.txt)的的原始！

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || SHELLCODE 解密.rar |
