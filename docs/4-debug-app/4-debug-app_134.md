
# 发现 LordPE 一个 bug

> 原文：[`www.pediy.com/kssd/pediy11/114733.html`](https://www.pediy.com/kssd/pediy11/114733.html)

用 `LordPE` 来重建附件中的 `exe` 程序，`LordPE` 直接崩溃掉了！~

查输入表的时候 也崩溃。。。

大体跟了一下发现是在

`REALIGN.DLL` 中的

```
1000164E   .  F3:A5         rep     movs dword ptr es:[edi], dword ptr [esi]
```

崩溃的

还有一处是 主模块的

```
0040D9B2  |> \\394F 10      cmp     dword ptr [edi+10], ecx
```

崩溃的

上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || `测试.rar` |
