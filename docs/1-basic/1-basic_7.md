

# 取 FS:\[0\]的一个小技巧

> 原文：[`www.pediy.com/kssd/pediy12/130147.html`](https://www.pediy.com/kssd/pediy12/130147.html)

代码:

```
004F45A5 >  1E              push    ds
004F45A6    0FA0            push    fs
004F45A8    1F              pop     ds
004F45A9    33C0            xor     eax, eax
004F45AB    8B00            mov     eax, dword ptr \[eax\]
004F45AD    1F              pop     ds

```

有一点点迷惑作用,做个笔记,以后待用![](img/2aa97d64a7e0b211d2f8f207eb382aa7.jpg "酷")