
# 喜悦加密器 v1.0

> 原文：[`www.pediy.com/kssd/pediy03/forum1179.htm`](https://www.pediy.com/kssd/pediy03/forum1179.htm)

注册 喜悦加密器 v1.0   

by `Fpc[CCG]`   

tools: `softice`   

d/l: [csdn download](http://www.csdn.net/soft/openfile.asp?kind=1&id=12000)   

它的注册算法设计的太糟了。

```
00446168  8D9528FEFFFF          lea     edx, [ebp+$FFFFFE28]   

* Reference to control TForm3.Edit1 : TEdit   

|   

0044616E  8B83E4020000          mov     eax, [ebx+$02E4]   

|   

00446174  E8D3E0FDFF            call    0042424C              <- 读取名字   

00446179  8B8528FEFFFF          mov     eax, [ebp+$FFFFFE28]   

0044617F  0FB64438FF            movzx   eax, byte ptr [eax+edi]-$01      <- 顺序取字符   

00446184  0145FC                add     [ebp-$04], eax        <- 累加   

00446187  47                    inc     edi   

00446188  4E                    dec     esi   

00446189  75DD                  jnz     00446168   

0044618B  8D9524FEFFFF          lea     edx, [ebp+$FFFFFE24]   

* Reference to control TForm3.Edit2 : TEdit   

|   

00446191  8B83E8020000          mov     eax, [ebx+$02E8]   

|   

00446197  E8B0E0FDFF            call    0042424C              <- 读取注册码   

0044619C  8B8524FEFFFF          mov     eax, [ebp+$FFFFFE24]   

|   

004461A2  E88D1DFCFF            call    00407F34              <- string to int   

004461A7  8B55FC                mov     edx, [ebp-$04]        <- 累加和到 edx   

004461AA  81F240E20100          xor     edx, $0001E240        <- 异或   

004461B0  3BC2                  cmp     eax, edx              <- 比较   

004461B2  7507                  jnz     004461BB              <- 不正确到下面仍有机会   

004461B4  C6055098440001        mov     byte ptr [$449850], $01   

004461BB  8D9520FEFFFF          lea     edx, [ebp+$FFFFFE20]   

* Reference to control TForm3.Edit2 : TEdit   

|   

004461C1  8B83E8020000          mov     eax, [ebx+$02E8]   

|   

004461C7  E880E0FDFF            call    0042424C   

004461CC  8B8520FEFFFF          mov     eax, [ebp+$FFFFFE20]   

* Possible String Reference to: '5764804'           <- 作者留的万能注册码。   

|   

004461D2  BAE0624400            mov     edx, $004462E0   

|   

004461D7  E8B8DAFBFF            call    00403C94   

004461DC  7507                  jnz     004461E5   

004461DE  C6055098440001        mov     byte ptr [$449850], $01   

004461E5  803D5098440000        cmp     byte ptr [$449850], $00   

004461EC  751A                  jnz     00446208   

004461EE  33D2                  xor     edx, edx
```

需要在以后的版本中加强。

整理一下：

`Name: Fpc[CCG]`

`Code: 123102` `4dd466d0e1e0cc123afbe504da2462ca`
