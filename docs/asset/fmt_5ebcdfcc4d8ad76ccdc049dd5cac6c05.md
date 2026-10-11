
# [算法进阶]TMPGEnc v2.58 注册算法分析-基于堆栈的简单虚拟机 v1.0

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-553.htm`](https://www.pediy.com/kssd/pediy09/pediy09-553.htm)

### 1. History

这个目标是 05 年 5.1 前后，看雪论坛的好好学习找我的，我断断续续搞了 1 年半，最后终于做出 keygen 了。

本还想进一步研究的，可惜没有太多时间，连台电脑都还买不起。本想等买了电脑来庆祝的，现在想申请 CUG I [免试]，感觉拿不出像样的东西，暂且拿出来混个精华，希望不辱 CUG 的名号。这个教材就当作 1.0 版本吧，如果以后再有研究再修改。

### 2. 虚拟机

虚拟机主要可分为 基于堆栈的虚拟机和 基于寄存器的虚拟机。

`TMPGEnc` 进入虚拟机的代码为：

![](img/842fb856b9fbef7f2546caa1b83738fb.jpg)

VM Engine 的代码为：

![](img/fb1208b7a3faebb96636e692f0b980fd.jpg)
![](img/c10786ac5d425962d7445fc4f20d5a2f.jpg)

![](img/7e93309fb26fb056c9722f99538a0780.jpg)

![](img/895dca882892d666c4e8b1f6ea59d8fb.jpg)

![](img/2c5d8103361a469e90c83081d493da28.jpg)

![](img/78e64f1d17608dd6e483c30ff5b84d11.jpg)

![](img/16ec92b8184d86fb5683e4c23b750d35.jpg)

![](img/b7e5d375573858cbe4f6043a7e293e43.jpg)

![](img/dd75bc22575f81761e7bfe01e80b7f02.jpg)

![](img/b605bb8ca2b8cd016ac6f65704581b37.jpg)

![](img/7b5da4701a012d4cf35a67fcb8e63cde.jpg)

![](img/699e4a180f8a0db9e3cdcef854777562.jpg)

![](img/3018e7d33a2932a13a1082259ddb5fde.jpg)

![](img/893a3a8ca4d69282e9384a109282698a.jpg)

![](img/dcdc60d00c6c99a4b5f0c7cd2c24769d.jpg)

![](img/7ec68cbaff05000ed46b852a272f3223.jpg)

![](img/7f92f9d922c4976706a6aaa9fde5bd74.jpg)

### 3. 反汇编 VM 代码

基本思路 i 就是将 VM 代码反汇编出来，然后看这个给予堆栈的虚拟机的代码（不是 X86 汇编了）。

我写的一个很简单的反汇编的代码如下：

```
代码:
```
