
# ASPack 1.08.03

> 原文：[`www.pediy.com/kssd/pediy06/pediy6688.htm`](https://www.pediy.com/kssd/pediy06/pediy6688.htm)

用 esp 定律 15 秒脱 ASPack 1.08.03 -> Alexey Solodovnikov 破解分析

**软件名称**：

用 esp 定律 15 秒脱 ASPack 1.08.03 -> Alexey Solodovnikov

**软件简介**：

一个老壳了，不用偶多说了

**下载地址**：

本地下载

---

**破解作者**：

springkang\[DFCG\]

**破解工具**：

`OD`，`impr`

**破解目的**：

偶是一只小菜鸟，偶有一点心得，愿与大家分享

---

**\[破解过程\]**

**详细过程**：

```
00479000  >  60             PUSHAD           //停在这，直接下 0012ffc0   硬件写入-----dword 断点
00479001    E8 00000000     CALL ex602.00479006
00479006    5D             POP EBP
00479007    81ED 0A4A4400  SUB EBP,ex602.00444A0A
00479558    61             POPAD            //中断后停在这，
00479559    75 08          JNZ SHORT ex602.00479563
0047955B    B8 01000000    MOV EAX,1
00479560    C2 0C00        RETN 0C
00479563    50             PUSH EAX
00479564    C3             RETN           //飞向光明之颠(fly 语录)
0045568C    55             PUSH EBP        //到站了，dump 后用 impr 修复，一切 OK!
0045568D    8BEC           MOV EBP,ESP
0045568F    83C4 F4        ADD ESP,-0C
00455692    B8 74554500    MOV EAX,ex602.00455574
```

**破解小结**：

偶很菜，挑了个软柿子，不好意思总结了!

---

版权公有，人权私有。欢迎转载，注明作者!

`4dd466d0e1e0cc123afbe504da2462ca`
