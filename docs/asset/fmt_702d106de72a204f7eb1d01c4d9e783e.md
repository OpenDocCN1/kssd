
# 128 位 AES 加密算法图解

> 原文：[`www.pediy.com/kssd/pediy10/90722.html`](https://www.pediy.com/kssd/pediy10/90722.html)

128 位 AES 加密算法图解

![](img/94642e64613a09fa67e7829d67ad5802.jpg)

由于 `AES` 算法比较复杂，单凭一个图很难说明细节，大家只能把图作为学习参考吧。

简单解释一下：

`AddRoundKey` 为 异或

`SubBytes` 为 查表替换（线性替换）

`ShiftRows` 为 按字节循环左移

`MixColumns` 为 矩阵乘法（这个没法讲了，大家翻翻大学的线性代数吧）

另外说一句，这个是 `AES` 算法的原型。计算机中使用的算法是优化后的，时间复杂度要小很多。但是空间要求需要 `4K`。就是把 4 步合成 1 步，查一个很大的表。

前几天做过一个 `DES` 的。链接：

http://bbs.pediy.com/showthread.php?t=90593
