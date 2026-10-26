
# 逆向分析

## 代码段 1

```
004DA400  |.  74 04         je      short 004DA406
004DA402  |.  33DB          xor     ebx, ebx
004DA404  |.  EB 02         jmp     short 004DA408
004DA406  |>  B3 01         mov     bl, 1
004DA408  |>  8B45 FC       mov     eax, [ebp-4]
004DA40B  |.  E8 8CDDFEFF   call    004C819C
004DA410  |.  84C0          test    al, al
004DA412  |.  74 02         je      short 004DA416
004DA414  |.  33DB          xor     ebx, ebx
004DA416  |>  33C0          xor     eax, eax
004DA418  |.  5A            pop     edx
004DA419  |.  59            pop     ecx
004DA41A  |.  59            pop     ecx
004DA41B  |.  64:8910       mov     fs:[eax], edx
004DA41E  |.  68 38A44D00   push    004DA438
004DA423  |>  8D45 E0       lea     eax, [ebp-20]
004DA426  |.  BA 08000000   mov     edx, 8
004DA42B  |.  E8 9492F2FF   call    004036C4
004DA430  \.  C3            retn
```

## 代码段 2

```
3、跟进call    004D9FEC 后是这样的。
004D9FEC  /$  55            push    ebp
004D9FED  |.  8BEC          mov     ebp, esp
004D9FEF  |.  83C4 EC       add     esp, -14
004D9FF2  |.  53            push    ebx
004D9FF3  |.  56            push    esi
004D9FF4  |.  57            push    edi
004D9FF5  |.  33DB          xor     ebx, ebx
004D9FF7  |.  895D EC       mov     [ebp-14], ebx
004D9FFA  |.  895D F4       mov     [ebp-C], ebx
004D9FFD  |.  8BD9          mov     ebx, ecx
004D9FFF  |.  8955 F8       mov     [ebp-8], edx
004DA002  |.  8945 FC       mov     [ebp-4], eax
004DA005  |.  8B45 FC       mov     eax, [ebp-4]
004DA008  |.  E8 BF99F2FF   call    004039CC
004DA00D  |.  8B45 F8       mov     eax, [ebp-8]
004DA010  |.  E8 B799F2FF   call    004039CC
004DA015  |.  33C0          xor     eax, eax
004DA017  |.  55            push    ebp
004DA018  |.  68 A8A04D00   push    004DA0A8
004DA01D  |.  64:FF30       push    dword ptr fs:[eax]
004DA020  |.  64:8920       mov     fs:[eax], esp
004DA023  |.  33F6          xor     esi, esi
004DA025  |.  8BD3          mov     edx, ebx
004DA027  |.  8B45 F8       mov     eax, [ebp-8]
004DA02A  |.  E8 19F7FFFF   call    004D9748                         ;  第一次这个call跟进取后，会根据固定字符来判断到底赋予
什么值，我这里固定字符是“CB”，对应的值就是“7CD”也许每个电脑都有不同的值吧。第二次跟进这个call的时候，同样是固定字符串“CB
”但它对应的值却取了“1F37”。就是刚才那个7CD（1997）反过来1F37（7991）。
004DA02F  |.  8945 F0       mov     [ebp-10], eax
004DA032  |.  8D55 F4       lea     edx, [ebp-C]
004DA035  |.  8B45 FC       mov     eax, [ebp-4]                     ;  用户名
004DA038  |.  E8 6FC1F2FF   call    004061AC
004DA03D  |.  8B45 F4       mov     eax, [ebp-C]                     ;  变成大写了。
004DA040  |.  E8 D397F2FF   call    00403818                         ;  位数
004DA045  |.  8BD0          mov     edx, eax
004DA047  |.  85D2          test    edx, edx
004DA049  |.  7E 20         jle     short 004DA06B
004DA04B  |.  B9 01000000   mov     ecx, 1                           ;  计数器
004DA050  |>  8B45 F4       /mov     eax, [ebp-C]                    ;  循环开始了，用的是大写的用户名“BINBINBIN”
004DA053  |.  8A4408 FF     |mov     al, [eax+ecx-1]                 ;  逐个取用户名
004DA057  |.  3C 20         |cmp     al, 20                          ;  和“20”比较，看看是不是空格
004DA059  |.  74 0C         |je      short 004DA067
004DA05B  |.  25 FF000000   |and     eax, 0FF
004DA060  |.  0FAF45 F0     |imul    eax, [ebp-10]                   ;  ASCII码值（16进制）乘以固定值7CD（1997）
004DA064  |.  48            |dec     eax                             ;  -1
004DA065  |.  03F0          |add     esi, eax                        ;  累加。最后的值在esi
004DA067  |>  41            |inc     ecx
004DA068  |.  4A            |dec     edx
004DA069  |.^ 75 E5         \jnz     short 004DA050
004DA06B  |>  8D55 EC       lea     edx, [ebp-14]
004DA06E  |.  8BC6          mov     eax, esi                         ;  结果转到eax
004DA070  |.  E8 87C4F2FF   call    004064FC                         ;  转变成10进制
004DA075  |.  8B4D EC       mov     ecx, [ebp-14]                    ;  第一次得到的值是1300038。第二次调用得到的值是5202132
004DA078  |.  8B45 08       mov     eax, [ebp+8]
004DA07B  |.  BA C0A04D00   mov     edx, 004DA0C0                    ;  “RKS-”固定字符串。将作为注册码的前面部分
004DA080  |.  E8 DF97F2FF   call    00403864
004DA085  |.  33C0          xor     eax, eax
004DA087  |.  5A            pop     edx
004DA088  |.  59            pop     ecx
004DA089  |.  59            pop     ecx
004DA08A  |.  64:8910       mov     fs:[eax], edx
004DA08D  |.  68 AFA04D00   push    004DA0AF
004DA092  |>  8D45 EC       lea     eax, [ebp-14]
004DA095  |.  E8 0A96F2FF   call    004036A4
004DA09A  |.  8D45 F4       lea     eax, [ebp-C]
004DA09D  |.  BA 03000000   mov     edx, 3
004DA0A2  |.  E8 1D96F2FF   call    004036C4
004DA0A7  \.  C3            retn
```

## 算法总结

1、首先是一个固定字符“`CB`”，这个值也许每个人装了软件后不同，但我的无论怎么弄都是这个字符。

2、根据这个固定字符得到“`7CD`”（第一次）或者“`1F37`”（第二次）。

3、用户名变大写后，逐个取用户名的ASCII值（16进制），乘以固定值`7CD`（`1F37`），再-1。然后累加在`ESI`中。

4、将刚才`ESI`中的值转成10进制，在和“`RKS-`”固定字符串连接就是注册码了。

5、注册码有两个的，随便一个都可以注册成功。注册信息保存在根目录的“`CB.ini`”里面

用户名：`binbinbin`

注册码：`RKS-1300038`（或者`RKS-5202132`）

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
