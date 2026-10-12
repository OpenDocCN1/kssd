
### 汇编代码分析

```
0042612B  |.  E8 00090100   |call    winimp32.00436A30  
00426130  |.  83C4 0C       |add     esp, 0C  
00426133  |.  B8 D8DA4400   |mov     eax, winimp32.0044DAD8         ;  ASCII "Options"  
00426138  |.  83C6 04       |add     esi, 4  
0042613B  |.  E8 9768FFFF   |call    winimp32.0041C9D7  
00426140  |.  89F8          |mov     eax, edi  
00426142  |.  83FF 02       |cmp     edi, 2  
00426145  |.^ 7C BC         |\jl     short winimp32.00426103  
00426147  |.  C605 52134500>mov     byte ptr [451352], 1  
0042614E  |.  B8 01000000   mov     eax, 1  
00426153  |>  C9            leave  
00426154  |.  5F            pop     edi  
00426155  |.  5E            pop     esi  
00426156  |.  5A            pop     edx  
00426157  |.  59            pop     ecx  
00426158  |.  5B            pop     ebx  
00426159  \\.  C3           retn  
```

可能大家也都注意到了：

```
0042605A      8A8F 10114500 mov     cl, byte ptr [edi+451110]
```

这里的 `byte ptr [edi+451110]`，我们顺着来到 `00451110` 后，会发现这是一张字符码对应表格，长度为：(4x4)x16 = 256，格式如下：

```
00451110   00 01 02 03 04 05 06 07 08 09 0A 0B 0C 0D 0E 0F  ...
..
00451120   10 11 12 13 14 15 16 17 18 19 1A 1B 1C 1D 1E 1F   
00451130   20 21 22 23 24 25 26 27 28 29 2A 2B 2C 2D 2E 2F   !"#$%&'()*+,-./
00451140   30 31 32 33 34 35 36 37 38 39 3A 3B 3C 3D 3E 3F  0123456789:;<=>?
00451150   40 61 62 63 64 65 66 67 68 69 6A 6B 6C 6D 6E 6F  @abcdefghijklmno
00451160   70 71 72 73 74 75 76 77 78 79 7A 5B 5C 5D 5E 5F  pqrstuvwxyz[]^\_
00451170   60 61 62 63 64 65 66 67 68 69 6A 6B 6C 6D 6E 6F  `abcdefghijklmno
00451180   70 71 72 73 74 75 76 77 78 79 7A 7B 7C 7D 7E 7F  pqrstuvwxyz{|}~
00451190   80 81 82 83 84 85 86 87 88 89 8A 8B 8C 8D 8E 8F    琋桻 S 梀 rX 沍絔
004511A0   90 91 92 93 94 95 96 97 98 99 9A 9B 9C 9D 9E 9F  `渃 eCg 焛辦痭^pM
004511B0   A0 A1 A2 A3 A4 A5 A6 A7 A8 A9 AA AB AC AD AE AF  rNⅲぅΗī?嗳釂
004511C0   B0 B1 B2 B3 B4 B5 B6 B7 B8 B9 BA BB BC BD BE BF  氨渤吹斗腹夯冀究
004511D0   C0 C1 C2 C3 C4 C5 C6 C7 C8 C9 CA CB CC CD CE CF  懒旅呐魄壬仕掏蜗
004511E0   D0 D1 D2 D3 D4 D5 D6 D7 D8 D9 DA DB DC DD DE DF  醒矣哉肿刭谯茌捱
004511F0   E0 E1 E2 E3 E4 E5 E6 E7 E8 E9 EA EB EC ED EE EF  噌忏溴骁栝觌祉铒
00451200   F0 F1 F2 F3 F4 F5 F6 F7 F8 F9 FA FB FC FD FE FF  瘃蝮趱鲼鈱鉐??
```

通过分析上面的这张表格，我们会发现，这张表格的主要作用便是：不管你输入的注册名字符是否为大写字母，他都通过这张表格将注册名全部转换为小写字母。

而 `00426042`～`00426065` 处的循环算法我用 `delphi+kol/mck` 源码表示，则为如下形式：

```
 for i:=0 to 3 do sum[i] := $00; //初始化变量 sum
 Texb := 0;
 for i:=1 to length(RegName) do
   begin
     if (ord(RegName[i])=$20) or (ord(RegName[i])=$2E)
     then Texb := (Texb and $3)
     else begin
             sum[Texb] := sum[Texb]  + key[ord(RegName[i]) and $FF];
             Texb := Texb + 1;
             Texb := (Texb and $3);
           end;
   end;
 sumAll := int2hex(sum[3],2)+int2hex(sum[2],2)+int2hex(sum[1],2)+int2hex(sum[0],2);
 tempA  := Hex2Int('$' + sumAll);        //得到的前部分的结果(RegName -> TempA)
```

上面的算法举例来说就是：

比如我输入的注册名为：`aCaFeeL`，则查找到上面我给出来的‘字符码对应表格’中相应的 `ASCII` 码就是：`61 63 61 66 65 65 6C`；`sum[0]=61`,`sum[1]=63`,`sum[2]=61`,`sum[3]=66`；然后又开始从 `sum[0]` 开始累加，即：`sum[0]=61+65=C6`,`sum[1]=63+65=C8`,`sum[2]=61+6C=CD`，`sum[3]=66`；则结果在内存窗口中为：`sum[0]=C6 sum[1]=C8 sum[2]=CD sum[3]=66`，故实际为十六进制的：`$66CDC8C6` 数字，暂且命名为 `tempA` 吧！

走过上面的循环运算后，我们继续 `F8` 键向下分析，来到这里后：

```
004260AA  |.  E8 A5130100   call    winimp32.00437454                ;  key2 -> eax
004260AF  |.  8B55 F4       mov     edx, dword ptr [ebp-C]           ;  key1 -> edx
004260B2  |.  8945 F8       mov     dword ptr [ebp-8], eax
004260B5  |.  81FA 00000001 cmp     edx, 1000000                     ;  edx = hex(key1)，不能小于 1000000
004260BB  |.  72 16         jb      short winimp32.004260D3
004260BD  |.  3D 00000001   cmp     eax, 1000000                     ;  eax = hex(key2)，不能小于 1000000
004260C2  |.  72 0F         jb      short winimp32.004260D3
004260C4  |.  89D0          mov     eax, edx                         ;  key1 -> eax(key1/key2 互换)
004260C6  |.  8B55 F8       mov     edx, dword ptr [ebp-8]           ;  key2 -> edx(key1/key2 互换)
004260C9  |.  E8 D9010000   call    winimp32.004262A7                ;  ×××算法 Call×××::F7 进入×××
004260CE  |.  3B45 FC       cmp     eax, dword ptr [ebp-4]           ;  eax 结果（tempB） = 上面的循环结果（tempA） ?
004260D1  |.  74 18         je      short winimp32.004260EB          ;  成功后保存注册相关的信息
004260D3  |>  6A 30         push    30                               ; /Style = MB_OK|MB_ICONEXCLAMATION|MB_APPLMODAL
```

我们知道了，`Key1`、`Key2` 的数值都必须大于 `$1000000` 才行，而满足上述条件后，我们便来到了后面部分的算法相关的 `call` 子程序了，这便是：

```
004260C9  |.  E8 D9010000   call    winimp32.004262A7
```

我们需要进入到这个子程序中一探究竟，按 `F7` 键进入后，来到了这里：

```
004262A7  /$  53            push    ebx
004262A8  |.  51            push    ecx
004262A9  |.  56            push    esi
004262AA  |.  89C1          mov     ecx, eax                         ;  key1 -> ecx
004262AC  |.  89D3          mov     ebx, edx                         ;  key2 -> ebx
004262AE  |.  8B15 64A74400 mov     edx, dword ptr [44A764]          ;  $E8B8D413 -> edx
004262B4  |.  E8 74FFFFFF   call    winimp32.0042622D                ;  ::F7 进入::(注意:只要 eax 的结果)
004262B9  |.  89C6          mov     esi, eax                         ;  (运算结果 eax)TempB -> (esi 下面要用)
004262BB  |.  89DA          mov     edx, ebx
004262BD  |.  BB 20000000   mov     ebx, 20
004262C2  |.  89C8          mov     eax, ecx                         ;  key1 -> eax
004262C4  |.  31C9          xor     ecx, ecx
004262C6  |.  E8 B3080100   call    winimp32.00436B7E                ;  key2 -> eax
004262CB  |.  31F0          xor     eax, esi                         ;  tempB = hex(key2) xor TempB(esi 上面算法中得到的值)
004262CD  |.  5E            pop     esi                              ;  如要注册成功，则必须: TempA = TempB
004262CE  |.  59            pop     ecx                              ;  即推出：Key2 = TempA xor TempB;
004262CF  |.  5B            pop     ebx
004262D0  \\.  C3           retn
```

通过上面的代码，我们看到后半部分的关键算法便在：

```
004262B4  |.  E8 74FFFFFF   call    winimp32.0042622D
```

的这个子 `call` 中，所以走到这里后，`F7` 键进入，来到了如下的地方：

```
0042622D  /$  53            push    ebx
0042622E  |.  51            push    ecx
0042622F  |.  56            push    esi
00426230  |.  57            push    edi
00426231  |.  C8 040000     enter   4, 0
00426235  |.  52            push    edx
00426236  |.  F645 F8 01    test    byte ptr [ebp-8], 1
0042623A  |.  75 07         jnz     short winimp32.00426243
0042623C  |.  B9 01000000   mov     ecx, 1
00426241  |.  EB 02         jmp     short winimp32.00426245
00426243  |>  89C1          mov     ecx, eax                         ;  ecx = eax = key1
00426245  |>  C745 FC 02000>mov     dword ptr [ebp-4], 2             ;  $2 -> [ebp-4]
0042624C  |.  89C7          mov     edi, eax                         ;  edi = eax = key1
0042624E  |.  EB 09         jmp     short winimp32.00426259
00426250  |>  D165 FC       /shl      dword ptr [ebp-4], 1           ;  [ebp-4] = [ebp-4] shl 1
00426253  |.  837D FC 00    |cmp     dword ptr [ebp-4], 0             ;  ? = 0
00426257  |.  74 46         |je      short winimp32.0042629F
00426259  |>  8B35 60A74400  mov     esi, dword ptr [44A760]          ;  $F527789F -> esi
0042625F  |.  89F8          |mov     eax, edi
00426261  |.  89FA          |mov     edx, edi                         ;  下面是大数运算
```
