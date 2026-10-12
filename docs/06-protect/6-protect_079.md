
# 某驱动源码(逆向)

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-507.htm`](https://www.pediy.com/kssd/pediy09/pediy09-507.htm)

> `dump_wmimmc.sys` 版本 997 这东西没什么可说的了

> 花了几天的时间看了一下,把部分逆向源码发上来,请高手指点

> ///////////////////////////////////////////////////

> 另外有一个问题想请教,`IDA` 分析一些函数出现这样问题 如

```
.text:000106E0 sub_106E0       proc near
.text:000106E0                push    ebp
.text:000106E1                mov     ebp, esp
.text:000106E3                sub     esp, 8
.text:000106E6                jmp     near ptr 35457h
.text:000106EB                db 0B2h
.text:000106EC                dd 4432DF36h
.text:000106F0                dd 52C62406h
.text:000106F4                db 99h, 0CFh, 0F5h
........................................
```

> 请问为什么会这样?有什么解决办法?

`4dd466d0e1e0cc123afbe504da2462ca`
