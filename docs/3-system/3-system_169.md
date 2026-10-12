
# 手动打造 97 字节 PE

> 原文：[`www.pediy.com/kssd/pediy11/125328.html`](https://www.pediy.com/kssd/pediy11/125328.html)

![](img/54832798bb5b4483bba617ede5347012.jpg)

DOS 头部

PE 头和 DOS 头重合,DOS 头部指向 PE 头的位置正好在选项头的节对齐位置,节对齐和文件对齐都设为 `4`

`0x8--0x1c` 之间是 PE 文件头

![](img/93b9b333504ad2a8ae04bd753ee0b3a7.jpg)

下面是重点,选项头部分

![](img/266942c15b454863ffc140d2d358abca.jpg)

![](img/f6e1860fc3af72a75168ebdb36df17ab.jpg)

代码:

```
68 0c 00 40 00   ;push 0x0040000c(imageBase+0xc)  也就是 c 处的字符串 user32.dll   (0x0040000c = 0x00400000 + 0x0c, imageBase 默认 0x00400000)

e8 53 1d 40 7c   ;call LoadLibraryA(0x7c401d5c)
PE 装载完后,Kernel32.dll 会加载所以可以直接调用,此指令后地址为 imageBase+0x28(0x00400028)
LoadLibraryA 地址为 0x7C801D7B 减去 0x00400028 得到 0x7C401D53

6a 00            ;push 0
eb 1c            ;jmp  0x00400048  (0x00400048 - 0x0040002c = 0x1c)   

6a 00            ;push 0
eb 10            ;jmp  0x0040005c  (0x0040005c - 0x0040004c = 0x10)

6a 00            ;push 0
eb d0            ;jmp  0x00400030

6a 00            ;push 0
eb 1c            ;jmp  0x00400050

e8 95079577      ;call MessageBoxA (此地址算法和 LoadLibraryA 同理)
c3               ;ret

```

`97.rar`
