
# Petite IAT 分析+去效验

> 原文：[`www.pediy.com/kssd/pediy10/95509.html`](https://www.pediy.com/kssd/pediy10/95509.html)

**【文章标题】**: Petite IAT 分析+去效验

**【文章作者】**: `yangjt`

**【作者邮箱】**: `yangjietao123@163.com`

**【作者 QQ 号】**: `325002492`

**【软件名称】**: 国家药品审评中心受理品种数据库搜索专家

**【作者声明】**: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

----------------------------------------

## 【详细过程】

我这里只分析 IAT 的还原的过程，如果想看其他部分的分析请访问

`http://bbs.pediy.com/showthread.php?t=74701`

这样 Petite 的分析就全了……

```assembly
0049A10F >  B8 00A04900     mov     eax, 0049A000////////////////EP
0049A114   6A 00           push    0
0049A116   68 73354800     push    00483573        //到这里下硬件执行断点
0049A11B   64:FF35 0000000>push    dword ptr fs:[0]
0049A122   64:8925 0000000>mov     dword ptr fs:[0], esp
```

然后运行

中断到这里

```assembly
00483573   E8 4F000000     call    004835C7
00483578   D9D7            fst     edi                           ; 非法使用寄存器
0048357A   ^ 71 AC         jno     short 00483528
0048357C   ^ E0 B9         loopdne short 00483537
```

`F7` 进去

```assembly
004835C7   33C0            xor     eax, eax
004835C9   5E              pop     esi
004835CA   64:8B18         mov     ebx, dword ptr fs:[eax]
004835CD   8B1B            mov     ebx, dword ptr [ebx]
004835CF   8D63 DA         lea     esp, dword ptr [ebx-26]
004835D2   8B6B 08         mov     ebp, dword ptr [ebx+8]
004835D5   8B6D F8         mov     ebp, dword ptr [ebp-8]
004835D8   8D8E DA020000   lea     ecx, dword ptr [esi+2DA]
004835DE   894B 04         mov     dword ptr [ebx+4], ecx//F4 到这一行，命令行下断点 he ecx
004835E1   64:891D 0000000>mov     dword ptr fs:[0], ebx//这里替换 SEH
```

到这里了

`Shift+F9` 继续运行

又中断到这里

```assembly
00483852   33C0            xor     eax, eax
00483854   64:8B18         mov     ebx, dword ptr fs:[eax]
00483857   8B1B            mov     ebx, dword ptr [ebx]
00483859   8D63 AE         lea     esp, dword ptr [ebx-52]
0048385C   61              popad
0048385D   833E 00         cmp     dword ptr [esi], 0
00483860   ^ 0F84 ACFDFFFF  je      00483612
```

到这里以后 `Ctrl+S` 查找命令序列

```assembly
pushad
jmp     eax
```

找到以后在他的下一行下断点

比如在这里是这样

```assembly
0048360D   F3:AA           rep     stos byte ptr es:[edi]
0048360F   60              pushad//找到这里
00483610   FFE0            jmp     eax
00483612   5B              pop     ebx   //这里下硬件执行断点
```

再往下就是输入表处理模块

以下是我对该程序的输入表处理模块的分析

```assembly
004836AB  > /833E 00       cmp     dword ptr [esi], 0              ;  还有 DLL 需要处理吗？
004836AE  . |0F84 15020000 je      004838C9        //记录下这里 je 到的地址
004836B4  . |51             push    ecx
004836B5  . |51             push    ecx
004836B6  . |FF95 A4070000 call    dword ptr [ebp+7A4]             ;  得到 Dll Image Base
004836BC  . |85C0           test    eax, eax
004836BE  . |75 11          jnz     short 004836D1                 ;  没有装载吗？
004836C0  . |83EC 04        sub     esp, 4
004836C3  . |FF95 90070000 call    dword ptr [ebp+790]
004836C9  . |85C0           test    eax, eax
004836CB  . |0F84 E6000000 je      004837B7
004836D1  > |8BF8           mov     edi, eax
004836D3  . |0340 3C        add     eax, dword ptr [eax+3C]       ;  找到 PE 头
004836D6  . |8B40 78       mov     eax, dword ptr [eax+78]
004836D9  . |FF7438 18     push    dword ptr [eax+edi+18]
004836DD  . |8B4C38 24     mov     ecx, dword ptr [eax+edi+24]
004836E1  . |03CF          add     ecx, edi
004836E3  . |51             push    ecx                             ;  IMAGE_EXPORT_DIRECTORY
004836E4  . |8B4C38 20     mov     ecx, dword ptr [eax+edi+20]
004836E8  . |03CF          add     ecx, edi
004836EA  . |51             push    ecx
004836EB  . |FF7438 10     push    dword ptr [eax+edi+10]
004836EF  . |FF7438 14     push    dword ptr [eax+edi+14]
004836F3  . |8B4438 1C     mov     eax, dword ptr [eax+edi+1C]
004836F7  . |03C7          add     eax, edi
004836F9  . |50             push    eax
004836FA  . |56             push    esi
004836FB  . |8B36           mov     esi, dword ptr [esi]
004836FD  . |03F5          add     esi, ebp
004836FF  > |8B06           mov     eax, dword ptr [esi]            ;  IAT Table
00483701  . |85C0           test    eax, eax                        ;  是不是空的？
00483703  . |0F84 85000000 je      0048378E                         ;  擦屁股走人……
00483709  . |79 2F         jns     short 0048373A
0048370B  . |0FBAE0 1E     bt      eax, 1E
0048370F  . |72 29         jb      short 0048373A
00483711  . |0FB7C0        movzx   eax, ax
00483714  . |2B4424 0C     sub     eax, dword ptr [esp+C]
00483718  . |0F82 B2000000 jb      004837D0
0048371E  . |3B4424 08     cmp     eax, dword ptr [esp+8]
00483722  . |0F83 A8000000 jnb     004837D0
00483728  . |C1E0 02       shl     eax, 2
0048372B  . |034424 04     add     eax, dword ptr [esp+4]
0048372F  . |8B00          mov     eax, dword ptr [eax]
00483731  . |03C7          add     eax, edi
00483733  . |8906          mov     dword ptr [esi], eax
00483735  . |83C6 04       add     esi, 4
00483738  .^|EB C5         jmp     short 004836FF
0048373A  > |03C6          add     eax, esi                          ;  Import AscII
0048373C  . |50             push    eax       ;这里记录下 eax，后面修复的时候用
0048373D  . |50             push    eax                           ;  Which function?
0048373E  . |57             push    edi                           ;  Which dll?
0048373F  . |FF95 94070000 call    dword ptr [ebp+794]             ;  GetProcAddress
00483745  . |85C0           test    eax, eax
00483747  . |0F84 82000000 je      004837CF                         ;  大概不会失败吧……
0048374D  . |FF4C24 28     dec     dword ptr [esp+28]              ;  ??随机偷窃……多多益善
00483751  . |7D 1F         jge     short 00483772                  ;  该偷不该偷？这里强行跳转一切就 OK 了……
00483753  . |8B5424 24     mov     edx, dword ptr [esp+24]         ;  要写到哪里？
00483757  . |C602 E9       mov     byte ptr [edx], 0E9             ;  jmp
0048375A  . |2BC2          sub     eax, edx
0048375C  . |83E8 05       sub     eax, 5
0048375F  . |8942 01       mov     dword ptr [edx+1], eax          ;  Stolen IAT Address
00483762  . |8BC2          mov     eax, edx
00483764  . |83C2 05       add     edx, 5                          ;  下一个位置
00483767  . |895424 24     mov     dword ptr [esp+24], edx         ;  ESP+24 是下一个要写被偷 IAT 的位置
0048376B  . |83E2 07       and     edx, 7                          ;  隔多少个 API 再偷？
0048376E  . |895424 28     mov     dword ptr [esp+28], edx         ;  判断是否应该偷的标志
00483772  > |8906          mov     dword ptr [esi], eax            ;  填充源程序 IAT Table
00483774  . |873C24        xchg    dword ptr [esp], edi            ;  好了……你已经作废了，我们要的是 Dll Image Base
00483777  . |83C9 FF       or      ecx, FFFFFFFF                   ;  ？？？
0048377A  . |33C0          xor     eax, eax
0048377C  . |F2:AE         repne   scas byte ptr es:[edi]          ;  多少字？
0048377E  . |FD            std                               ;  反向胡搞
0048377F  . |F7D1          not     ecx                           ;  这才对……
00483781  . |4F            dec     edi                           ;  定位到结尾
00483782  . |F3:AA         rep     stos byte ptr es:[edi]          ;  抹掉 Import AscII
00483784  . |5F            pop     edi                           ;  edi = Current Dll
00483785  . |FC            cld                               ;  正向
```



```
00483786   . |83C6 04       add     esi, 4                     ; 搞下一个
00483789   .^|E9 71FFFFFF   jmp     004836FF
0048378E   > |5E            pop     esi                          ; 搞完一个 Dll 了
0048378F   . |83C4 18       add     esp, 18                      ; 清理堆栈
00483792   . |8B16          mov     edx, dword ptr [esi]
00483794   . |03D5          add     edx, ebp
00483796   . |8D83 10010000 lea     eax, dword ptr [ebx+110]
0048379C   . |8B4C24 04     mov     ecx, dword ptr [esp+4]
004837A0   > |833A 00       cmp     dword ptr [edx], 0
004837A3   . |74 12         je      short 004837B7
004837A5   . |3B1A          cmp     ebx, dword ptr [edx]         ; 这些代码的作用是解密 OEP ==膜拜下作者
004837A7   . |8318 00       sbb     dword ptr [eax], 0           ; 不是 MessageBox 则-1
004837AA   . |390A          cmp     dword ptr [edx], ecx         ; ??
004837AC   . |8318 00       sbb     dword ptr [eax], 0
004837AF   . |83C2 04       add     edx, 4                     ; 找下一个
004837B2   . |C108 03       ror     dword ptr [eax], 3
004837B5   .^|EB E9         jmp     short 004837A0
004837B7   > |C706 00000000 mov     dword ptr [esi], 0
004837BD   . |5F            pop     edi
004837BE   . |83C9 FF       or      ecx, FFFFFFFF
004837C1   . |33C0          xor     eax, eax
004837C3   . |F2:AE         repne   scas byte ptr es:[edi]
004837C5   . |8BCF          mov     ecx, edi
004837C7   . |83C6 04       add     esi, 4                     ; 下一个 Dll
004837CA   .^\\E9 DCFEFFFF  jmp     004836AB
004837CF   >  59            pop     ecx
```

如果你有兴趣知道怎么搞得你可以看看过程……如果你只想完美修复 IAT 改一个 `jmp` 就可以了

很明显，修改以后就过不了后面的 OEP 解码了……所以这次处理的程序只能用于 IAT 修复

修改一下

```
00483751   . |7D 1F        jge     short 00483772
```

这个地方，改成 `jmp` 这样就不会偷了

```
00483772
```

断在这一行，取消断点……此时 `esi` 是 IAT 的起始地址。命令行 `dd esi`

看 `004836AE` 这一行 `je` 到的地址，记录我这里是 `je` 到 `004838C9`

在 `004838C9` 这一行 `F4` 然后复制一份 IAT 的备份出来

这里是我的

备份：

```
38 C2 DC 77 8B 79 DA 77 27 6C DA 77 8D BB DC 77 52 78 DA 77 BB 7A DA 77 F4 E9 DA 77 0C F0 DA 77
E7 EA DA 77 00 00 00 00 CF 65 17 5D 00 00 00 00 1F 94 EF 77 4C 7B EF 77 16 C0 EF 77 7D 73 F0 77
CD D5 F1 77 14 8E EF 77 D4 72 F0 77 AE D6 F1 77 56 6A EF 77 1A A2 EF 77 97 D9 EF 77 71 5A EF 77
F1 7C EF 77 79 7C EF 77 55 A1 EF 77 A5 61 EF 77 9B 86 EF 77 AF 8F EF 77 5B 8F EF 77 11 E6 EF 77
C7 D4 EF 77 EC 8E EF 77 DB 5E EF 77 28 8B EF 77 EE 8B EF 77 5F 6E EF 77 EF 61 EF 77 29 5E EF 77
77 5D EF 77 A1 6A EF 77 1C EF EF 77 25 8D EF 77 CE EC EF 77 C1 61 EF 77 5A 6F F0 77 FA D3 EF 77
4F BA EF 77 1B 82 EF 77 07 68 F2 77 0A 70 EF 77 E0 5F EF 77 70 5B EF 77 79 6F EF 77 23 AD EF 77
FF DC EF 77 FA 6B EF 77 00 00 00 00 2D FF 92 7C D7 06 81 7C F8 C0 80 7C BF 50 83 7C 6F 17 80 7C
74 A8 80 7C B5 99 80 7C B0 94 93 7C 37 05 93 7C 7B D3 81 7C F1 0E 81 7C E8 8D 83 7C 48 CD 80 7C
35 49 84 7C AA 45 86 7C EF D6 81 7C 87 4B 81 7C 93 CC 81 7C A8 2F 81 7C 37 CD 80 7C D9 2F 81 7C
C4 00 93 7C 98 0F 81 7C 56 2C 81 7C 84 9B 80 7C F1 9A 80 7C 19 9F 80 7C 0C 8A 83 7C 30 A5 80 7C
A1 9E 80 7C 6F BD 80 7C 17 D1 80 7C FE A3 80 7C 78 34 83 7C BD 2F 81 7C 95 14 82 7C 2E 98 80 7C
F2 1E 80 7C A9 2A 81 7C 1A 1E 80 7C 12 CB 81 7C 92 41 95 7C AF AC 80 7C 1D 1C 83 7C 17 0B 81 7C
54 5D 83 7C 47 28 81 7C 16 2F 81 7C C3 2C 81 7C E0 97 80 7C FF 08 83 7C 65 9C 80 7C 59 24 81 7C
77 37 81 7C B9 4C 83 7C 3F 2E 81 7C 09 BD 80 7C 72 67 83 7C 06 E9 80 7C 8C E8 80 7C B5 A4 80 7C
9C 39 81 7C 46 20 83 7C BC 22 83 7C 61 23 83 7C E1 26 81 7C 2E 0C 81 7C 27 0E 81 7C 12 18 80 7C
9E DE 80 7C E0 10 92 7C B1 13 93 7C 91 9F 80 7C 2D 9A 80 7C B1 01 81 7C 00 10 92 7C 78 F7 82 7C
CF 99 80 7C 85 08 83 7C 32 97 83 7C A8 C1 80 7C F7 28 83 7C B7 A0 80 7C 30 25 80 7C 4C 0D 83 7C
8B 99 80 7C 66 98 80 7C 30 FE 92 7C 1A 98 80 7C 06 98 80 7C 7B 1D 80 7C 7E AC 80 7C D0 97 80 7C
43 C8 85 7C 41 BB 80 7C A9 60 83 7C C3 60 83 7C 93 0B 83 7C 41 B7 80 7C 40 AE 80 7C B9 FF 80 7C
22 FF 80 7C 37 CD 80 7C 29 BF 80 7C 55 A0 80 7C A5 1B 82 7C 7A 12 81 7C CD FD 80 7C CF FC 80 7C
41 4D 83 7C 56 BE 80 7C ED 2A 86 7C A1 BE 80 7C 48 C3 82 7C 28 1A 80 7C E7 9B 80 7C 29 16 80 7C
7E 2B 81 7C 79 38 81 7C B1 4E 83 7C 77 EE 80 7C DC 15 81 7C E3 14 82 7C 22 28 81 7C AD 1E 83 7C
35 14 82 7C FE 4F 83 7C 4A 93 80 7C 6F B5 80 7C 98 9C 80 7C 74 A1 80 7C CA 10 83 7C 46 24 80 7C
95 DE 80 7C 92 4B 81 7C 21 FE 92 7C 00 00 00 00 39 4B 0F 77 F0 48 0F 77 65 33 11 77 BB 6B 0F 77
A2 4B 0F 77 FD 4C 0F 77 35 4C 0F 77 1B 4C 0F 77 80 48 0F 77 00 00 00 00 D7 1F FE 5E 00 00 00 00
E8 11 61 7D 00 00 00 00 91 6C D5 77 A6 8F D2 77 27 BF D5 77 28 8E D1 77 C5 77 D2 77 E1 BD D5 77
FD AA D2 77 AB AE D2 77 E7 C2 D3 77 07 95 D2 77 40 A3 D2 77 B8 96 D1 77 C8 98 D2 77 12 B1 D2 77
EA E7 D2 77 A0 97 D2 77 3D 9E D2 77 5B F2 D2 77 0E 97 D1 77 5D EE D3 77 AD A8 D1 77 FF EB D3 77
5E EA D2 77 BA 14 D3 77 1C EF D2 77 96 D8 D2 77 C8 F1 D4 77 8B F1 D4 77 6B 21 D3 77 1B AF D2 77
D9 9E D2 77 A9 E4 D2 77 11 12 D3 77 C6 B3 D2 77 F1 F4 D2 77 F3 D5 D2 77 42 00 D3 77 7D A9 D2 77
94 00 D3 77 7E C1 D2 77 08 C9 D2 77 6C 99 D2 77 7A 15 D3 77 23 98 D2 77 ED 42 D2 77 55 96 D2 77
F3 99 D2 77 28 8E D1 77 1F 8F D2 77 B2 DE D2 77 C7 03 D3 77 C3 37 D2 77 4E 4A D2 77 22 78 D2 77
13 93 D2 77 28 9B D3 77 9C B1 D2 77 5F F4 D2 77 7A 97 D2 77 7B 1F D3 77 B4 90 D2 77 C7 86 D1 77
9D 86 D1 77 9D C2 D2 77 42 8C D1 77 3E D3 D2 77 FF 97 D2 77 9C 8F D1 77 6C D0 D3 77 22 B2 D2 77
0E 1B D3 77 2E 8C D1 77 49 98 D2 77 A3 89 D2 77 86 B0 D2 77 9A B0 D2 77 51 02 D3 77 77 E5 D3 77
72 9F D2 77 12 D3 D2 77 F6 E8 D2 77 EA 07 D5 77 75 A2 D5 77 05 5B D5 77 02 C7 D3 77 E5 A5 D5 77
30 99 D2 77 5D 94 D1 77 42 A0 D2 77 2A F9 D2 77 D5 98 D2 77 2F 9C D2 77 78 8E D1 77 11 90 D2 77
C2 F3 D2 77 00 F1 D4 77 AB 8E D1 77 00 00 D3 77 4F F9 D2 77 E8 C2 D2 77 0F 91 D2 77 DA 94 D1 77
5E C3 D2 77 8E 90 D2 77 19 97 D2 77 7A C3 D2 77 D5 8F D2 77 08 7C D2 77 44 99 D2 77 B0 C8 D2 77
2B 8D D1 77 D2 D1 D2 77 4C BE D5 77 D9 FD D3 77 66 97 D2 77 2B 77 D2 77 F6 8B D1 77 BD FB D2 77
4E 97 D2 77 5A CA D2 77 FD 8F D2 77 E9 8F D2 77 21 90 D1 77 F9 02 D5 77 3C 47 D2 77 67 F9 D1 77
0B F2 D4 77 B2 FA D4 77 BD 1A D3 77 C4 D2 D2 77 56 AF D2 77 9E B2 D2 77 6B F5 D2 77 89 C6 D3 77
6E 43 D2 77 9D D3 D2 77 E0 9D D2 77 60 9B D2 77 00 00 00 00 54 89 6C 76 42 87 6A 76 3F 82 69 76
91 90 6C 76 E3 88 6C 76 40 91 6A 76 99 74 69 76 D6 33 69 76 90 71 68 76 A6 57 69 76 A4 4D 69 76
72 5A 69 76 00 00 00 00 57 37 F8 72 40 4D F7 72 5F 66 F8 72 00 00 00 00 10 7C 33 76 63 25 32 76
9F 30 32 76 00 00 00 00 44 D0 9A 76 60 D0 9A 76 EA F6 9A 76 E7 31 9E 76 AC 00 9E 76 2A 56 9F 76
1F 5F A0 76 F3 A2 9D 76 71 AB A2 76 C1 A9 A2 76 5A 57 9F 76 1D C8 A8 76 C5 56 9C 76 B6 FC 9D 76
F2 87 9C 76 80 5E 9B 76 00 00 00 00 6A 09 CA 74
```

重新载入程序

命令行下断点 `he eip`，然后 `F9` 再 `F8` 就到 OEP 了。这个时候转到刚才的 IAT 起始地址

把备份的 IAT 还原回去，直接 `Dump`。

这样 `Dump` 以后就可以删除 `Petite` 加壳以后生成的垃圾区段了……就是最后面一个没有名字的区段。嗯……就是这样

打开 `ImportREC` 获得程序输入表，去掉添加新节选项

现在轮到在 `0048373C` 时记录的 `eax` 派上用场了，把这个 `eax` 取整下……比如我这里是 `004822D8`，那我们就用 `00482200`

转成程序的 `Rav`，就是减掉基址 `00400000` 把得数填到 `ImportREC` 的新输入表信息里的 `RVA`，然后修复转存……搞定……测试程序……什么，还不能运行？肯定有自效验了，嗯……这个程序的校验比较简单，大小效验，这里引用 `KuNgBiM` 所说的方法

原帖见 `http://bbs.pediy.com/showthread.php?t=15444`

`【去自校验过程】`

重新打开 `Ollydbg`，载入刚刚我们脱壳修复优化后的“`dumped_.exe`”文件(这里我采用的是默认脱壳文件名)

根据以往的经验，软件校验无非就是采用校验文件名和大小作为依据，那么我们大胆猜想到肯定使用了下面这条语句：

```
cmp eax,30FF2
```

(`30FF2` 的十进制就是原文件大小 `200,690` 字节)

现脱壳文件为 `497,664` 字节，也就是说现在代码应该替换为：

```
cmp eax,79800
```

所以用 `Ctrl+S` 搜索命令“`cmp eax,30FF2`”：



`\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\\`

这里我们搜索`cmp eax, 3A516`，进到它上面那个`Call`里

```
0045CE13      B8 16A50300   mov     eax, 3A516
0045CE18      C3            retn
0045CE19      90            nop
```

写入这两句代码然后复制到可执行文件，好了……程序终于可以运行了……

--------------------------------------------------------------------------------

【经验总结】

好像跑题了，`rz`……别的程序有自校验的话一般也是此类……好了……就这样了……搞的还算比较完美

……现在文件的大小和加壳前已经是一样的了……就这一点我可以打包票……

--------------------------------------------------------------------------------

【版权声明】：`转载请注明作者并保持文章的完整，谢谢！`

2009 年 08 月 11 日 16:09:21
