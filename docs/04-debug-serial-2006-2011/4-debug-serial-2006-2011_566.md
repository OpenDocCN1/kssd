
# 七巧板游戏 Tangram-7 V1.1 算法分析及算法注册机

> 原文：[`www.pediy.com/kssd/pediy11/107265.html`](https://www.pediy.com/kssd/pediy11/107265.html)

**【破文标题】** 七巧板游戏 Tangram-7 V1.1 算法分析及算法注册机

**【破文作者】** zaas\[PYG\]\[FCT\]

**【破解工具】** `OllyICE`，`PEiD` v0.94

**【破解平台】** `WinXP`

**【软件名称】** `Tangram-7` V1.1

**【更新时间】** 2010-2-11

**【软件类别】** 国外软件/游戏

**【软件语言】** 英文

**【应用平台】** `WinXP`/`2000`/`2003`/`Vista`

**【软件性质】** 共享(收费)软件

**【软件大小】** 7.8M

**【原版下载】** [`download.cnet.com/Tangram-7/3...-10973833.html`](http://download.cnet.com/Tangram-7/3000-18536_4-10973833.html)

**【保护方式】** 注册码

**【软件简介】** 七巧板是一种智力游戏，顾名思义，是由七块板组成的。而这七这块板可拼成许多图形(1600 种以上)，例如：三角形、平行四边形、不规则多边形、玩家也可以把它拼成各种人物、形象、动物、桥、房、塔等等，亦可是一些中、英文字母。七巧板的好处与用处简直是多不胜数

，以下是七巧板部分的好处与用处：形状概念、视觉分辨、认智技巧、视觉记忆、手眼协调、鼓励开放、扩散思考、创作机会。

无论在现代或古代，七巧板都是用以启发幼儿智力的良好伙伴。能够把幼儿对实物与形态之间的桥梁连接起来，培养幼儿的观察力、想像力、形状分析及创意逻辑上都有巨大的发展空间。

现在被家长们广泛采用来帮助小孩学习基本逻辑关系和数学概念。可以帮助孩子认识各种几何图形、数字、认识周长和面积的意义，了解毕氏定理。

七巧板还可以教导小朋友辨认颜色，引导小朋友领悟图形的分割与合成，进而增强小朋友的手部智能、耐性和观察力。亦可用以说故事，将数十幅七巧板图片连成一幅幅的连惯图画，即可当漫画般说故给小朋友听。先拼出数款猫、几款狗、一间屋，即可说出一美妙动人的故事。

**【破解声明】** 我是一只小菜鸟，偶得一点心得，愿与大家分享：）

---

**【破解内容】**

---

用 `PEiD` 查壳，`Microsoft Visual Studio .NET 2005 -- 2008 -> Microsoft Corporation` \*

输入错误的注册码，有提示”、“`Wrong key`”，但搜索字符串无结果。

下断的过程不再说了，有细心和耐心就够了，直接来到注册验证部分。

### Part1：注册码整理

```
00409080   $  81EC 24030000 sub     esp, 324
00409086   .  A1 04804100   mov     eax, dword ptr [418004]
0040908B   .  33C4          xor     eax, esp
0040908D   .  898424 200300>mov     dword ptr [esp+320], eax
00409094   .  53            push    ebx
00409095   .  56            push    esi
00409096   .  33DB          xor     ebx, ebx
00409098   .  33C9          xor     ecx, ecx
0040909A   .  33F6          xor     esi, esi
0040909C   .  8D6424 00     lea     esp, dword ptr [esp]
004090A0   >  8A86 90465000 mov     al, byte ptr [esi+504690]        ;  假码字符入 al
004090A6   .  3AC3          cmp     al, bl                          ;  结束了吗？
004090A8   .  0F84 9D000000 je      0040914B
004090AE   .  3C 30         cmp     al, 30                          ;  跟 0 比较
004090B0   .  72 04         jb      short 004090B6
004090B2   .  3C 39         cmp     al, 39                          ;  和 9 比较
004090B4   .  76 0A         jbe     short 004090C0
004090B6   >  8AD0          mov     dl, al                          ;  入 dl
004090B8   .  80EA 61       sub     dl, 61                          ;  -61
004090BB   .  80FA 05       cmp     dl, 5                           ;  和 5 比较
004090BE   .  77 0A         ja      short 004090CA                  ;  大于则跳也就是比较是否大于 f，大于则直接取下一个
004090C0   >  88840C 280100>mov     byte ptr [esp+ecx+128], al
004090C7   .  83C1 01       add     ecx, 1                          ;  计数器+1
004090CA   >  8A86 91465000 mov     al, byte ptr [esi+504691]        ;  下一字符
004090D0   .  3AC3          cmp     al, bl
004090D2   .  74 77         je      short 0040914B
004090D4   .  3C 30         cmp     al, 30
004090D6   .  72 04         jb      short 004090DC
004090D8   .  3C 39         cmp     al, 39
004090DA   .  76 0A         jbe     short 004090E6
004090DC   >  8AD0          mov     dl, al
004090DE   .  80EA 61       sub     dl, 61
004090E1   .  80FA 05       cmp     dl, 5
004090E4   .  77 0A         ja      short 004090F0
004090E6   >  88840C 280100>mov     byte ptr [esp+ecx+128], al
004090ED   .  83C1 01       add     ecx, 1
004090F0   >  8A86 92465000 mov     al, byte ptr [esi+504692]        ;  第三字符/////为什么不用循环呢。。。。
004090F6   .  3AC3          cmp     al, bl
004090F8   .  74 51         je      short 0040914B
004090FA   .  3C 30         cmp     al, 30
004090FC   .  72 04         jb      short 00409102
004090FE   .  3C 39         cmp     al, 39
00409100   .  76 0A         jbe     short 0040910C
00409102   >  8AD0          mov     dl, al
00409104   .  80EA 61       sub     dl, 61
00409107   .  80FA 05       cmp     dl, 5
0040910A   .  77 0A         ja      short 00409116
0040910C   >  88840C 280100>mov     byte ptr [esp+ecx+128], al
00409113   .  83C1 01       add     ecx, 1
00409116   >  8A86 93465000 mov     al, byte ptr [esi+504693]
0040911C   .  3AC3          cmp     al, bl
0040911E   .  74 2B         je      short 0040914B
00409120   .  3C 30         cmp     al, 30
00409122   .  72 04         jb      short 00409128
00409124   .  3C 39         cmp     al, 39
00409126   .  76 0A         jbe     short 00409132
00409128   >  8AD0          mov     dl, al
0040912A   .  80EA 61       sub     dl, 61
0040912D   .  80FA 05       cmp     dl, 5
00409130   .  77 0A         ja      short 0040913C
00409132   >  88840C 280100>mov     byte ptr [esp+ecx+128], al
00409139   .  83C1 01       add     ecx, 1
0040913C   >  83C6 04       add     esi, 4                          ;  一个 dword 为一组循环
0040913F   .  81FE 00010000 cmp     esi, 100
00409145   .^ 0F8C 55FFFFFF jl      004090A0
0040914B   >  83F9 1C       cmp     ecx, 1C                         ;  一共 1C 位注册码，注册码格式为 0-f（小写）
0040914E   .  74 19         je      short 00409169

```

注册码共 34 位，经以上处理去掉假码中分段的“-”

注册码的字符范围在“0-9”或者“a-f”之间

### Part2：假码整理

代码:



### 汇编代码段 1：序列号获取与预处理

```
00409150   .  5E             pop     esi
00409151   .  33C0           xor     eax, eax
00409153   .  5B             pop     ebx
00409154   .  8B8C24 200300> mov     ecx, dword ptr [esp+320]
0040915B   .  33CC           xor     ecx, esp
0040915D   .  E8 51220000    call    0040B3B3
00409162   .  81C4 24030000  add     esp, 324
00409168   .  C3             retn
00409169   >  8D8424 280200> lea     eax, dword ptr [esp+228]
00409170   .  50             push    eax
00409171   .  889C24 480100> mov     byte ptr [esp+148], bl
00409178   .  33F6           xor     esi, esi
0040917A   .  E8 21020000    call    004093A0                              ;  算法-》机器码字符串 A
0040917F   .  83C4 04        add     esp, 4
00409182   .  33C9           xor     ecx, ecx
00409184   .  EB 0A          jmp     short 00409190
00409186   .  8DA424 000000> lea     esp, dword ptr [esp]
0040918D   .  8D49 00        lea     ecx, dword ptr [ecx]
00409190   >  8A840C 280200> mov     al, byte ptr [esp+ecx+228]           ;  字符串 A 处理方式同假码
00409197   .  3AC3           cmp     al, bl
00409199   .  0F84 8C000000  je      0040922B
0040919F   .  3C 30          cmp     al, 30
004091A1   .  7C 04          jl      short 004091A7
004091A3   .  3C 39          cmp     al, 39
004091A5   .  7E 08          jle     short 004091AF
004091A7   >  3C 61          cmp     al, 61
004091A9   .  7C 0B          jl      short 004091B6
004091AB   .  3C 66          cmp     al, 66
004091AD   .  7F 07          jg      short 004091B6
004091AF   >  884434 28      mov     byte ptr [esp+esi+28], al
004091B3   .  83C6 01        add     esi, 1
004091B6   >  8A840C 290200> mov     al, byte ptr [esp+ecx+229]
004091BD   .  3AC3           cmp     al, bl
004091BF   .  74 6A          je      short 0040922B
004091C1   .  3C 30          cmp     al, 30
004091C3   .  7C 04          jl      short 004091C9
004091C5   .  3C 39          cmp     al, 39
004091C7   .  7E 08          jle     short 004091D1
004091C9   >  3C 61          cmp     al, 61
004091CB   .  7C 0B          jl      short 004091D8
004091CD   .  3C 66          cmp     al, 66
004091CF   .  7F 07          jg      short 004091D8
004091D1   >  884434 28      mov     byte ptr [esp+esi+28], al
004091D5   .  83C6 01        add     esi, 1
004091D8   >  8A840C 2A0200> mov     al, byte ptr [esp+ecx+22A]
004091DF   .  3AC3           cmp     al, bl
004091E1   .  74 48          je      short 0040922B
004091E3   .  3C 30          cmp     al, 30
004091E5   .  7C 04          jl      short 004091EB
004091E7   .  3C 39          cmp     al, 39
004091E9   .  7E 08          jle     short 004091F3
004091EB   >  3C 61          cmp     al, 61
004091ED   .  7C 0B          jl      short 004091FA
004091EF   .  3C 66          cmp     al, 66
004091F1   .  7F 07          jg      short 004091FA
004091F3   >  884434 28      mov     byte ptr [esp+esi+28], al
004091F7   .  83C6 01        add     esi, 1
004091FA   >  8A840C 2B0200> mov     al, byte ptr [esp+ecx+22B]
00409201   .  3AC3           cmp     al, bl
00409203   .  74 26          je      short 0040922B
00409205   .  3C 30          cmp     al, 30
00409207   .  7C 04          jl      short 0040920D
00409209   .  3C 39          cmp     al, 39
0040920B   .  7E 08          jle     short 00409215
0040920D   >  3C 61          cmp     al, 61
0040920F   .  7C 0B          jl      short 0040921C
00409211   .  3C 66          cmp     al, 66
00409213   .  7F 07          jg      short 0040921C
00409215   >  884434 28      mov     byte ptr [esp+esi+28], al
00409219   .  83C6 01        add     esi, 1
0040921C   >  83C1 04        add     ecx, 4
0040921F   .  81F9 00010000  cmp     ecx, 100
00409225   .^ 0F8C 65FFFFFF  jl      00409190
0040922B   >  33C0           xor     eax, eax
0040922D   .  83FE 1C        cmp     esi, 1C
00409230   .  0F85 47010000 jnz     0040937D
00409236   .  885C24 44      mov     byte ptr [esp+44], bl               ;  结尾设 0
0040923A   .  885C24 16      mov     byte ptr [esp+16], bl               ;  转换后得到--》字符串 B
```

此段获取序列号，并将序列号如假码般处理，即 `34` 位去掉每四位的 `-`，得到 `28` 位注册码

### Part3：序列号处理

代码:

```
0040923E   .  8BFF           mov     edi, edi
00409240   >  8A4C44 28      mov     cl, byte ptr [esp+eax*2+28]         ;  字符串 B 取字符入 cl
00409244   .  8AD1           mov     dl, cl
00409246   .  80EA 30        sub     dl, 30                               ;  -30
00409249   .  80FA 09        cmp     dl, 9                                ;  和 9 比较
0040924C   .  77 04          ja      short 00409252                       ;  分别字符与数字
0040924E   .  8ACA           mov     cl, dl
00409250   .  EB 03          jmp     short 00409255
00409252   >  80E9 57        sub     cl, 57                               ;  字母-57，数字-30
00409255   >  8A5444 29      mov     dl, byte ptr [esp+eax*2+29]          ;  下一位
00409259   .  8ADA           mov     bl, dl
0040925B   .  80EB 30        sub     bl, 30
0040925E   .  80FB 09        cmp     bl, 9
00409261   .  77 04          ja      short 00409267
00409263   .  8AD3           mov     dl, bl
00409265   .  EB 03          jmp     short 0040926A
00409267   >  80EA 57        sub     dl, 57
0040926A   >  C0E1 04        shl     cl, 4                                ;  上一字符处理后左移一位
0040926D   .  02CA           add     cl, dl                               ;  和这一字符组合
0040926F   .  8A5444 2A      mov     dl, byte ptr [esp+eax*2+2A]          ;  第三字符
00409273   .  884C04 08      mov     byte ptr [esp+eax+8], cl
00409277   .  8ACA           mov     cl, dl
00409279   .  80E9 30        sub     cl, 30
0040927C   .  80F9 09        cmp     cl, 9
0040927F   .  76 05          jbe     short 00409286
00409281   .  80EA 57        sub     dl, 57
00409284   .  8ACA           mov     cl, dl
00409286   >  8A5444 2B      mov     dl, byte ptr [esp+eax*2+2B]          ;  第四字符
0040928A   .  8ADA           mov     bl, dl
0040928C   .  80EB 30        sub     bl, 30
0040928F   .  80FB 09        cmp     bl, 9
00409292   .  77 04          ja      short 00409298
00409294   .  8AD3           mov     dl, bl
00409296   .  EB 03          jmp     short 0040929B
00409298   >  80EA 57        sub     dl, 57
0040929B   >  C0E1 04        shl     cl, 4
0040929E   .  02CA           add     cl, dl
004092A0   .  884C04 09      mov     byte ptr [esp+eax+9], cl
004092A4   .  83C0 02        add     eax, 2                               ;  计数器+2
004092A7   .  83F8 0E        cmp     eax, 0E                               ;  28/2=14
004092AA   .^ 7C 94          jl      short 00409240                       ;  转换后得到--》字符串 C
```

此段将机器码（序列号）每两位处理合成为一个字符，处理方法为：数字 - `30`，字母 - `57`

得到字符串 `C`，`28` 位机器码变为 `14` 位数值

### Part4：假码处理

代码:



### 假码处理

```
004092AC   .  33C0          xor     eax, eax
004092AE   .  884424 26     mov     byte ptr [esp+26], al
004092B2   >  8A8C44 280100>mov     cl, byte ptr [esp+eax*2+128]     ;  假码同上处理得到-->假码 B
004092B9   .  8AD1          mov     dl, cl
004092BB   .  80EA 30       sub     dl, 30
004092BE   .  80FA 09       cmp     dl, 9
004092C1   .  77 04         ja      short 004092C7
004092C3   .  8ACA          mov     cl, dl
004092C5   .  EB 03         jmp     short 004092CA
004092C7   >  80E9 57       sub     cl, 57
004092CA   >  8A9444 290100>mov     dl, byte ptr [esp+eax*2+129]
004092D1   .  8ADA          mov     bl, dl
004092D3   .  80EB 30       sub     bl, 30
004092D6   .  80FB 09       cmp     bl, 9
004092D9   .  77 04         ja      short 004092DF
004092DB   .  8AD3          mov     dl, bl
004092DD   .  EB 03         jmp     short 004092E2
004092DF   >  80EA 57       sub     dl, 57
004092E2   >  C0E1 04       shl     cl, 4
004092E5   .  02CA          add     cl, dl
004092E7   .  8A9444 2A0100>mov     dl, byte ptr [esp+eax*2+12A]
004092EE   .  884C04 18     mov     byte ptr [esp+eax+18], cl
004092F2   .  8ACA          mov     cl, dl
004092F4   .  80E9 30       sub     cl, 30
004092F7   .  80F9 09       cmp     cl, 9
004092FA   .  76 05         jbe     short 00409301
004092FC   .  80EA 57       sub     dl, 57
004092FF   .  8ACA          mov     cl, dl
00409301   >  8A9444 2B0100>mov     dl, byte ptr [esp+eax*2+12B]
00409308   .  8ADA          mov     bl, dl
0040930A   .  80EB 30       sub     bl, 30
0040930D   .  80FB 09       cmp     bl, 9
00409310   .  77 04         ja      short 00409316
00409312   .  8AD3          mov     dl, bl
00409314   .  EB 03         jmp     short 00409319
00409316   >  80EA 57       sub     dl, 57
00409319   >  C0E1 04       shl     cl, 4
0040931C   .  02CA          add     cl, dl
0040931E   .  884C04 19     mov     byte ptr [esp+eax+19], cl
00409322   .  83C0 02       add     eax, 2
00409325   .  83F8 0E       cmp     eax, 0E
00409328   .  ^ 7C 88       jl      short 004092B2
```

此段对假码的处理方法同序列号。我的疑问是，为什么作者不用模块化的方法写代码，徒增代码量尔。。。

### Part 5：真假码比较的过程

```
0040932A   .  B3 01         mov     bl, 1
0040932C   .  33F6          xor     esi, esi
0040932E   .  8BFF          mov     edi, edi
00409330   >  32C0          xor     al, al
00409332   .  83FE 0E       cmp     esi, 0E
00409335   .  8BCE          mov     ecx, esi
00409337   .  7D 13         jge     short 0040934C          ;  各位机器码（序列号的 ascii 相加），每次取的时候去掉第一位
00409339   .  8DA424 000000>lea     esp, dword ptr [esp]
00409340   >  02440C 08     add     al, byte ptr [esp+ecx+8]       ;  取字符串 C 字符相加入 al
00409344   .  83C1 01       add     ecx, 1                          ;  计数器+1
00409347   .  83F9 0E       cmp     ecx, 0E
0040934A   .  ^ 7C F4       jl      short 00409340
0040934C   >  0FB65434 08   movzx   edx, byte ptr [esp+esi+8]      ;  取字符串 C 字符入 edx
00409351   .  0FB6C8        movzx   ecx, al                         ;  al 入 ecx
00409354   .  C0E9 04       shr     cl, 4                           ;  右移一位
00409357   .  02C8          add     cl, al                          ;  和 al 相加
00409359   .  52            push    edx                               ;  push edx 保存的是字符串 C 首字母
0040935A   .  83E1 0F       and     ecx, 0F                          ;  去掉第二位
0040935D   .  8B048D 008D41>mov     eax, dword ptr [ecx*4+418D00]   ;  根据余数的不同，套用不同的算法
00409364   .  FFD0          call    eax                               ;  此 call 是算法的关键
00409366   .  83C4 04       add     esp, 4
00409369   .  3A4434 18     cmp     al, byte ptr [esp+esi+18]         ;  假码 B 字符和 al 比较
0040936D   .  0F94C1        sete    cl                                ;  爆破点
00409370   .  83C6 01       add     esi, 1                            ;  计数器+1
00409373   .  22D9          and     bl, cl                            ;  and 结果为 1 成功
00409375   .  83FE 0E       cmp     esi, 0E
00409378   .  ^ 7C B6       jl      short 00409330
0040937A   .  0FBEC3        movsx   eax, bl
0040937D   >  8B8C24 280300>mov     ecx, dword ptr [esp+328]
00409384   .  5E            pop     esi
00409385   .  5B            pop     ebx
00409386   .  33CC          xor     ecx, esp
00409388   .  E8 26200000   call    0040B3B3
0040938D   .  81C4 24030000 add     esp, 324
00409393   .  C3            retn
```

序列号的代码和假码的代码处理完毕，进行比较了。`sete cl` + `and bl, cl` 的处理结果：注册码一次为 0 则结果永远为 0，也即注册失败。。。

### Part 6：机器码怎么来的

跟进：

`0040917A   .  E8 21020000   call    004093A0          ;  算法-》机器码字符串 A`

代码:



```
004093A0  /$  81EC 58020000  sub     esp, 258
004093A6  |.  A1 04804100   mov     eax, dword ptr [418004]
004093AB  |.  33C4          xor     eax, esp
004093AD  |.  898424 540200>mov     dword ptr [esp+254], eax
004093B4  |.  8B8424 5C0200>mov     eax, dword ptr [esp+25C]
004093BB  |.  53            push    ebx
004093BC  |.  55            push    ebp
004093BD  |.  56            push    esi
004093BE  |.  57            push    edi
004093BF  |.  8D4C24 18     lea     ecx, dword ptr [esp+18]
004093C3  |.  51            push    ecx                       ; /pBufferSize
004093C4  |.  8D5424 60     lea     edx, dword ptr [esp+60]     ; |
004093C8  |.  52            push    edx                       ; |Buffer
004093C9  |.  894424 1C     mov     dword ptr [esp+1C], eax     ; |
004093CD  |.  C74424 20 040>mov     dword ptr [esp+20], 104     ; |
004093D5  |.  FF15 98514100 call    dword ptr [<&KERNEL32.GetCompute>; \GetComputerNameW
004093DB  |.  8D4424 5C     lea     eax, dword ptr [esp+5C]     ;  取得计算机名 unicode
004093DF  |.  50            push    eax                       ; /String
004093E0  |.  66:C74424 5C >mov     word ptr [esp+5C], 0        ; |
004093E7  |.  FF15 94514100 call    dword ptr [<&KERNEL32.lstrlenW>] ; \计算机名长度 2
004093ED  |.  83F8 0E       cmp     eax, 0E                   ;  计算机名长度小于 14 位
004093F0  |.  7C 14         jl      short 00409406
004093F2  |.  6A 0E         push    0E                        ; /n = E (14.)
004093F4  |.  8D4C24 60     lea     ecx, dword ptr [esp+60]     ; |
004093F8  |.  51            push    ecx                       ; |String2
004093F9  |.  8D5424 44     lea     edx, dword ptr [esp+44]     ; |
004093FD  |.  52            push    edx                       ; |String1
004093FE  |.  FF15 90514100 call    dword ptr [<&KERNEL32.lstrcpynW>>; \lstrcpynW
00409404  |.  EB 3B         jmp     short 00409441
00409406  |>  33D2          xor     edx, edx
00409408  |.  8D1C00        lea     ebx, dword ptr [eax+eax]    ;  ebx=len(name)*2
0040940B  |.  8D7C24 3C     lea     edi, dword ptr [esp+3C]
0040940F  |.  90            nop
00409410  |>  33C9          /xor    ecx, ecx
00409412  |.  85C0          |test   eax, eax
00409414  |.  7E 22         |jle     short 00409438
00409416  |.  8BF7          |mov     esi, edi
00409418  |>  8D2C11        |/lea    ebp, dword ptr [ecx+edx]
0040941B  |.  83FD 0E       ||cmp    ebp, 0E
0040941E  |.  7D 0E         ||jge    short 0040942E
00409420  |.  66:8B6C4C 5C  ||mov    bp, word ptr [esp+ecx*2+5C]   ;  取得 name 字符 1
00409425  |.  66:03EA       ||add    bp, dx
00409428  |.  66:03E9       ||add    bp, cx                    ;  +计数器
0040942B  |.  66:892E       ||mov    word ptr [esi], bp
0040942E  |>  83C1 01       ||add    ecx, 1                    ;  计数器+1
00409431  |.  83C6 02       ||add    esi, 2
00409434  |.  3BC8          ||cmp    ecx, eax
00409436  |.^ 7C E0         |\jl     short 00409418
00409438  |>  03D0          |add     edx, eax                  ;  eax=len(name)
0040943A  |.  03FB          |add     edi, ebx
0040943C  |.  83FA 0E       |cmp     edx, 0E                   ;  变为 14 位字符串 M
0040943F  |.^ 7C CF         \jl     short 00409410
00409441  |>  C64424 38 00  mov     byte ptr [esp+38], 0
00409446  |.  33D2          xor     edx, edx
00409448  |.  EB 06         jmp     short 00409450
0040944A  |   8D9B 00000000 lea     ebx, dword ptr [ebx]
00409450  |>  8A4414 3D     /mov    al, byte ptr [esp+edx+3D]     ;  取字符串 M 字符
00409454  |.  024414 3C     |add    al, byte ptr [esp+edx+3C]
00409458  |.  8AC8          |mov    cl, al
0040945A  |.  C0E8 04       |shr    al, 4                   ;  右移 4 位
0040945D  |.  0FBEC0        |movsx  eax, al
00409460  |.  83C0 FF       |add    eax, -1                 ;  -1
00409463  |.  80E1 0F       |and    cl, 0F                  ;  字符串 M 字符保留个位
00409466  |.  83F8 0E       |cmp    eax, 0E                 ;  Switch (cases 0..E)
00409469  |.  77 43         |ja     short 004094AE          ;  大于 E 则=0
0040946B  |.  FF2485 B89540>|jmp    dword ptr [eax*4+4095B8]
00409472  |>  B0 66         |mov    al, 66                  ;  Case E of switch 00409466
00409474  |.  EB 3A         |jmp    short 004094B0
00409476  |>  B0 65         |mov    al, 65                  ;  Case D of switch 00409466
00409478  |.  EB 36         |jmp    short 004094B0
0040947A  |>  B0 64         |mov    al, 64                  ;  Case C of switch 00409466
0040947C  |.  EB 32         |jmp    short 004094B0
0040947E  |>  B0 63         |mov    al, 63                  ;  Case B of switch 00409466
00409480  |.  EB 2E         |jmp    short 004094B0
00409482  |>  B0 62         |mov    al, 62                  ;  Case A of switch 00409466
00409484  |.  EB 2A         |jmp    short 004094B0
00409486  |>  B0 61         |mov    al, 61                  ;  Case 9 of switch 00409466
00409488  |.  EB 26         |jmp    short 004094B0
0040948A  |>  B0 39         |mov    al, 39                  ;  Case 8 of switch 00409466
0040948C  |.  EB 22         |jmp    short 004094B0
0040948E  |>  B0 38         |mov    al, 38                  ;  Case 7 of switch 00409466
00409490  |.  EB 1E         |jmp    short 004094B0
00409492  |>  B0 37         |mov    al, 37                  ;  Case 6 of switch 00409466
00409494  |.  EB 1A         |jmp    short 004094B0
00409496  |>  B0 36         |mov    al, 36                  ;  Case 5 of switch 00409466
00409498  |.  EB 16         |jmp    short 004094B0
0040949A  |>  B0 35         |mov    al, 35                  ;  Case 4 of switch 00409466
0040949C  |.  EB 12         |jmp    short 004094B0
0040949E  |>  B0 34         |mov    al, 34                  ;  Case 3 of switch 00409466
004094A0  |.  EB 0E         |jmp    short 004094B0
004094A2  |>  B0 33         |mov    al, 33                  ;  Case 2 of switch 00409466
004094A4  |.  EB 0A         |jmp    short 004094B0
004094A6  |>  B0 32         |mov    al, 32                  ;  Case 1 of switch 00409466
004094A8  |.  EB 06         |jmp    short 004094B0
004094AA  |>  B0 31         |mov    al, 31                  ;  Case 0 of switch 00409466
004094AC  |.  EB 02         |jmp    short 004094B0
004094AE  |>  B0 30         |mov    al, 30                  ;  Default case of switch 00409466
004094B0  |>  884414 1C     |mov    byte ptr [esp+edx+1C], al
004094B4  |.  0FBEC1        |movsx  eax, cl
004094B7  |.  83C0 FF       |add    eax, -1                 ;  Switch (cases 1..F)
004094BA  |.  83F8 0E       |cmp    eax, 0E
004094BD  |.  77 43         |ja     short 00409502
004094BF  |.  FF2485 F49540>|jmp    dword ptr [eax*4+4095F4]
004094C6  |>  B0 66         |mov    al, 66                  ;  Case F of switch 004094B7
004094C8  |.  EB 3A         |jmp    short 00409504
004094CA  |>  B0 65         |mov    al, 65                  ;  Case E of switch 004094B7
004094CC  |.  EB 36         |jmp    short 00409504
004094CE  |>  B0 64         |mov    al, 64                  ;  Case D of switch 004094B7
004094D0  |.  EB 32         |jmp    short 00409504
004094D2  |>  B0 63         |mov    al, 63                  ;  Case C of switch 004094B7
004094D4  |.  EB 2E         |jmp    short 00409504
004094D6  |>  B0 62         |mov    al, 62                  ;  Case B of switch 004094B7
004094D8  |.  EB 2A         |jmp    short 00409504
004094DA  |>  B0 61         |mov    al, 61                  ;  Case A of switch 004094B7
004094DC  |.  EB 26         |jmp    short 00409504
004094DE  |>  B0 39         |mov    al, 39                  ;  Case 9 of switch 004094B7
004094E0  |.  EB 22         |jmp    short 00409504
004094E2  |>  B0 38         |mov    al, 38                  ;  Case 8 of switch 004094B7
004094E4  |.  EB 1E         |jmp    short 00409504
004094E6  |>  B0 37         |mov    al, 37                  ;  Case 7 of switch 004094B7
004094E8  |.  EB 1A         |jmp    short 00409504
004094EA  |>  B0 36         |mov    al, 36                  ;  Case 6 of switch 004094B7
004094EC  |.  EB 16         |jmp    short 00409504
004094EE  |>  B0 35         |mov    al, 35                  ;  Case 5 of switch 004094B7
004094F0  |.  EB 12         |jmp    short 00409504
004094F2  |>  B0 34         |mov    al, 34                  ;  Case 4 of switch 004094B7
004094F4  |.  EB 0E         |jmp    short 00409504
004094F6  |>  B0 33         |mov    al, 33                  ;  Case 3 of switch 004094B7
004094F8  |.  EB 0A         |jmp    short 00409504
004094FA  |>  B0 32         |mov    al, 32                  ;  Case 2 of switch 004094B7
004094FC  |.  EB 06         |jmp    short 00409504
004094FE  |>  B0 31         |mov    al, 31                  ;  Case 1 of switch 004094B7
00409500  |.  EB 02         |jmp    short 00409504
00409502  |>  B0 30         |mov    al, 30                  ;  Default case of switch 004094B7
00409504  |>  884414 1D     |mov    byte ptr [esp+edx+1D], al
00409508  |.  83C2 02       |add    edx, 2
0040950B  |.  83FA 1C       |cmp    edx, 1C                 ;  28 位
0040950E  |.^ 0F8C 3CFFFFFF \jl     00409450
00409514  |.  8B4424 14     mov     eax, dword ptr [esp+14]
00409518  |.  8D4C24 1C     lea     ecx, dword ptr [esp+1C]
0040951C  |.  51            push    ecx                       ; /String
0040951D  |.  C600 00       mov     byte ptr [eax], 0         ; |
00409520  |.  33F6          xor     esi, esi                  ; |
00409522  |.  FF15 C0514100 call    dword ptr [<&KERNEL32.lstrlenA>] ; \lstrlenA
00409528  |.  85C0          test    eax, eax                  ;  取得长度 1c
0040952A  |.  7E 73         jle     short 0040959F
0040952C  |.  8B2D DC514100 mov     ebp, dword ptr [<&KERNEL32.lstrc>;  kernel32.lstrcatA
00409532  |.  8D5C24 1C     lea     ebx, dword ptr [esp+1C]
00409536  |.  83EB 01       sub     ebx, 1
00409539  |.  8DA424 000000>lea     esp, dword ptr [esp]
00409540  |>  8B4C24 14     /mov    ecx, dword ptr [esp+14]
00409544  |.  8A5433 01     |mov    dl, byte ptr [ebx+esi+1]
00409548  |.  8D4424 10     |lea    eax, dword ptr [esp+10]
0040954C  |.  8D7E 01       |lea    edi, dword ptr [esi+1]
0040954F  |.  50            |push   eax
00409550  |.  51            |push   ecx
00409551  |.  885424 18     |mov    byte ptr [esp+18], dl
00409555  |.  C64424 19 00  |mov    byte ptr [esp+19], 0
0040955A  |.  FFD5          |call   ebp
0040955C  |.  85F6          |test   esi, esi
0040955E  |.  74 2E         |je     short 0040958E
00409560  |.  8D5424 1C     |lea    edx, dword ptr [esp+1C]
00409564  |.  52            |push   edx                       ; /String
00409565  |.  FF15 C0514100 |call   dword ptr [<&KERNEL32.lstrlenA>>; \lstrlenA
0040956B  |.  83E8 01       |sub    eax, 1
0040956E  |.  3BF0          |cmp    esi, eax
00409570  |.  74 1C         |je     short 0040958E
00409572  |.  8BC7          |mov    eax, edi
00409574  |.  25 03000080   |and    eax, 80000003
00409579  |.  79 05         |jns    short 00409580
0040957B  |.  48            |dec    eax
0040957C  |.  83C8 FC       |or     eax, FFFFFFFC
0040957F  |.  40            |inc    eax
00409580  |>  75 0C         |jnz    short 0040958E
00409582  |.  8B4C24 14     |mov    ecx, dword ptr [esp+14]
00409586  |.  68 7C694100   |push   0041697C
0040958B  |.  51            |push   ecx
0040958C  |.  FFD5          |call   ebp
0040958E  |>  8D5424 1C     |lea    edx, dword ptr [esp+1C]
00409592  |.  52            |push   edx                       ; /String
00409593  |.  8BF7          |mov    esi, edi                  ; |
00409595  |.  FF15 C0514100 |call   dword ptr [<&KERNEL32.lstrlenA>>; \lstrlenA
0040959B  |.  3BF0          |cmp    esi, eax
0040959D  |.^ 7C A1         \jl    short 00409540
0040959F  |>  8B8C24 640200>mov    ecx, dword ptr [esp+264]
004095A6  |.  5F            pop     edi
004095A7  |.  5E            pop     esi
004095A8  |.  5D            pop     ebp
004095A9  |.  5B            pop     ebx
004095AA  |.  33CC          xor     ecx, esp
004095AC  |.  E8 021E0000   call    0040B3B3                  ;  四个一组加上“-”
004095B1  |.  81C4 58020000 add    esp, 258
004095B7  \.  C3            retn
```



机器码根据计算机名而来。根据计算机名的长度，计算机名的`ascii`码每次`+2`循环取够`28`位，然后根据尾数的不同取不同的字符，每四个一组加上`-`即为序列号（机器码）。

### Part 7：最终的算法

跟进：

```
0040935D   .  8B048D 008D41>mov     eax, dword ptr [ecx*4+418D00]    ;  根据余数的不同，套用不同的算法
00409364   .  FFD0          call    eax       
```

代码:

```
00409680   .  0FB64424 04   movzx   eax, byte ptr [esp+4]      ;  取字符串 C 字符？
00409685   .  35 8F000000   xor     eax, 8F                       ;  xor 8F
0040968A   .  C3            retn
```

根据`part5` `mod` `16`所得的余数`0-F`，分别不同的`Xor` `1F`，`2F`，....`9F`，`AA`,`BB`,`CC`,`DD`,`EE`,`FF`，得到一组数值，和假码经过相同处理得到的数值一一比对，如果全部一致则注册成功，否则注册失败。

### 【破解总结】

软件的爆破点在于:

```
0040936D   .  0F94C1        sete    cl                            ;  爆破点
```

`cl`的值。需要注意的是，当软件选择第`>7`的拼图后，软件会在不同位置做一次验证，验证不通过即使爆破显示注册成功了一样有限制。该位置在：

代码:

```
0040999D   .  0F94C1        sete    cl
004099A0   .  83C6 01       add     esi, 1
004099A3   .  22D9          and     bl, cl
004099A5   .  83FE 0E       cmp     esi, 0E
004099A8   .  ^ 7C B6       jl      short 00409960
```

如果要爆破需修改以上两处。

### 【算法总结】
软件并非明码比较。

第一步：软件去掉序列号和注册码中的`-`；

第二步：软件会把机器码和注册码每两位按照数字-&`h30`，字母-&`h57`的方式进行第一次处理：

处理后的数字`A`*`16`+处理后的数字`B`得到字符串`C`

第三步：

字符串`C`各位`ascii`相加，每次取的时候去掉第一位，根据`mod` `16`的余数选择不同的数字`XOR`

和同样处理后的注册码各位数值比较，全部相等则注册成功。否则失败。

综上所述，写出`VB`注册机。鉴于`VB`代码过长，不再附录。仅给出注册机。

![](img/6821a6175013046370981b6035d43ab4.jpg "使眼色")

### 【版权声明】
破文是学习的手记，兴趣是成功的源泉；本破文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!
