
# 荣创-房屋交易中介管理软件 1.05

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-242.htm`](https://www.pediy.com/kssd/pediy08/pediy8-242.htm)

【文章标题】: 某个房屋交易中介管理软件的注册码算法分析  

【文章作者】: 石羽  

【作者 QQ 号】: `67212777`  

【软件名称】: 荣创-房屋交易中介管理软件 1.05  

【软件大小】: `3298KB`  

【下载地址】: 自己搜索下载  

【加壳方式】: 无  

【保护方式】: 注册码  

【编写语言】: `Borland Delphi 4.0 - 5.0`  

【使用工具】: `OllyICE`  

【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

--------------------------------------------------------------------------------

## 详细过程

1、使用 `OllyICE` 载入文件，查找字符“软件已经注册成功”，双击来到 `006041D2`，我们向上看在 `00604140` 下个断点，然后 `F9` 运行。

进入软件单击“帮助---软件注册”，软件成功被断下，单步运行。

```
00604140  /.  55          push    ebp               *断点*
00604141  |.  8BEC        mov     ebp, esp
00604143  |.  83C4 D4     add     esp, -2C
00604146  |.  53          push    ebx
00604147  |.  56          push    esi
00604148  |.  33C9        xor     ecx, ecx
0060414A  |.  894D DC     mov     [ebp-24], ecx
0060414D  |.  894D F0     mov     [ebp-10], ecx
00604150  |.  894D F4     mov     [ebp-C], ecx
00604153  |.  894D F8     mov     [ebp-8], ecx
00604156  |.  894D FC     mov     [ebp-4], ecx
00604159  |.  8BD8        mov     ebx, eax
0060415B  |.  33C0        xor     eax, eax
0060415D  |.  55          push    ebp
0060415E  |.  68 86426000 push    00604286
00604163  |.  64:FF30     push    dword ptr fs:[eax]
00604166  |.  64:8920     mov     fs:[eax], esp
00604169  |.  8D45 FC     lea     eax, [ebp-4]
0060416C  |.  E8 7FEEFFFF call    00602FF0
00604171  |.  8B55 FC     mov     edx, [ebp-4]                     *取得用户号
00604174  |.  8D83 50030000 lea     eax, [ebx+350]
0060417A  |.  E8 99FEDFFF call    00404018
0060417F  |.  8D55 F8     lea     edx, [ebp-8]
00604182  |.  A1 14F96300 mov     eax, [63F914]
00604187  |.  8B00        mov     eax, [eax]
00604189  |.  E8 A6F5FFFF call    00603734
0060418E  |.  8B45 F8     mov     eax, [ebp-8]
00604191  |.  E8 FEEEFFFF call    00603094                     *跟进
```

--------------------------------------------------------------------------------

```
00603094  /$  55          push    ebp
00603095  |.  8BEC        mov     ebp, esp
00603097  |.  6A 00       push    0
00603099  |.  6A 00       push    0
0060309B  |.  6A 00       push    0
0060309D  |.  53          push    ebx
0060309E  |.  8945 FC     mov     [ebp-4], eax
006030A1  |.  8B45 FC     mov     eax, [ebp-4]
006030A4  |.  E8 5F13E0FF call    00404408
006030A9  |.  33C0        xor     eax, eax
006030AB  |.  55          push    ebp
006030AC  |.  68 09316000 push    00603109
006030B1  |.  64:FF30     push    dword ptr fs:[eax]
006030B4  |.  64:8920     mov     fs:[eax], esp
006030B7  |.  8D45 F4     lea     eax, [ebp-C]
006030BA  |.  E8 31FFFFFF call    00602FF0
006030BF  |.  8B45 F4     mov     eax, [ebp-C]           *将用户号存入 EAX 中
006030C2  |.  8D55 F8     lea     edx, [ebp-8]
006030C5  |.  E8 46FFFFFF call    00603010              * 跟进
```

---------------------------------------------------------------------------------

```
00603010  /$  55          push    ebp
00603011  |.  8BEC        mov     ebp, esp
00603013  |.  83C4 F4     add     esp, -0C
00603016  |.  53          push    ebx
00603017  |.  56          push    esi
00603018  |.  8BF2        mov     esi, edx
0060301A  |.  8945 FC     mov     [ebp-4], eax
0060301D  |.  8B45 FC     mov     eax, [ebp-4]
00603020  |.  E8 E313E0FF call    00404408
00603025  |.  33C0        xor     eax, eax
00603027  |.  55          push    ebp
00603028  |.  68 87306000 push    00603087
0060302D  |.  64:FF30     push    dword ptr fs:[eax]
00603030  |.  64:8920     mov     fs:[eax], esp
00603033  |.  8B45 FC     mov     eax, [ebp-4]
00603036  |.  E8 91FDFFFF call    00602DCC           *关键 CALL，跟进。可以直接下断点在这。
```

------------------------------------------------------------------------------------------

```
00602DCC  /$  55          push    ebp
00602DCD  |.  8BEC        mov     ebp, esp
00602DCF  |.  83C4 F8     add     esp, -8
00602DD2  |.  53          push    ebx
00602DD3  |.  56          push    esi
00602DD4  |.  33D2        xor     edx, edx
00602DD6  |.  8955 F8     mov     [ebp-8], edx
00602DD9  |.  8945 FC     mov     [ebp-4], eax
00602DDC  |.  8B45 FC     mov     eax, [ebp-4]
00602DDF  |.  E8 2416E0FF call    00404408
00602DE4  |.  33C0        xor     eax, eax
00602DE6  |.  55          push    ebp
00602DE7  |.  68 A62E6000 push    00602EA6
00602DEC  |.  64:FF30     push    dword ptr fs:[eax]
00602DEF  |.  64:8920     mov     fs:[eax], esp
00602DF2  |.  8B55 FC     mov     edx, [ebp-4]
00602DF5  |.  B8 C02E6000 mov     eax, 00602EC0            ;  -
00602DFA  |.  E8 4117E0FF call    00404540
00602DFF  |.  85C0        test    eax, eax
00602E01  |.  76 1F       jbe     short 00602E22
00602E03  |>  8D55 FC     /lea    edx, [ebp-4]
00602E06  |.  B9 01000000 |mov    ecx, 1
00602E0B  |.  92          |xchg   eax, edx
00602E0C  |.  E8 8B16E0FF |call   0040449C          *将用户号全部转换成数字，其实就是把用户号中的“-”去掉了。
00602E11  |.  8B55 FC     |mov    edx, [ebp-4]
00602E14  |.  B8 C02E6000 |mov    eax, 00602EC0           ;  -
00602E19  |.  E8 2217E0FF |call   00404540
00602E1E  |.  85C0        |test   eax, eax
00602E20  |.^ 77 E1       \\ja     short 00602E03
00602E22  |>  8B45 FC     mov     eax, [ebp-4]
00602E25  |.  E8 2A14E0FF call    00404254         *取得转换后的用户号位数
00602E2A  |.  83F8 08     cmp     eax, 8           *和 8 比较
00602E2D  |.  74 04       je      short 00602E33    *相等跳
00602E2F  |.  33DB        xor     ebx, ebx
00602E31  |.  EB 58       jmp     short 00602E8B
00602E33  |>  8D55 F8     lea     edx, [ebp-8]
00602E36  |.  8B45 FC     mov     eax, [ebp-4]
00602E39  |.  E8 166AE0FF call    00409854
00602E3E  |.  33C9        xor     ecx, ecx
00602E40  |.  33F6        xor     esi, esi
00602E42  |.  B8 06000000 mov     eax, 6               *EAX=6
00602E47  |>  8B55 F8     /mov    edx, [ebp-8]           *将转换后的用户号存入 EDX
00602E4A  |.  0FB65402 FF |movzx  edx, byte ptr [edx+eax-1]    *从用户号第 6 位数字向前取 ASC 值
00602E4F  |.  83EA 30     |sub    edx, 30               *取到的 ASC 值减去 30
00602E52  |.  03F2       |add    esi, edx             *算后的值累加到 ESI 中
00602E54  |.  03C9       |add    ecx, ecx             *将 ECX 原有的值乘以 2
00602E56  |.  8D0C89     |lea    ecx, [ecx+ecx*4]       *将 ECX 的值再乘以 5
00602E59  |.  03D1       |add    edx, ecx             *将 ECX+EDX 存入 EDX 中
00602E5B  |.  8BCA       |mov    ecx, edx             *将 EDX 中值存入 ECX
00602E5D  |.  48         |dec    eax                *eax=eax-1
00602E5E  |.  85C0       |test   eax, eax
00602E60  |.^ 75 E5     \\jnz     short 00602E47
```

其实这段只是把用户号的前六位数字倒转过来再转成十六进制值（A），并把前六位数字累加存入 `ESI` 中。

```
00602E62  |.  33D2        xor     edx, edx             *将 EDX 清零
00602E64  |.  B8 08000000 mov     eax, 8               *EAX=8
00602E69  |>  03D2       /add    edx, edx
00602E6B  |.  8D1492     |lea    edx, [edx+edx*4]
```



```
00602E6E  |.  8B5D F8       |mov     ebx, [ebp-8]  

00602E71  |.  0FB65C03 FF   |movzx   ebx, byte ptr [ebx+eax-1]  

00602E76  |.  83EB 30       |sub     ebx, 30  

00602E79  |.  03D3         |add     edx, ebx  

00602E7B  |.  48            |dec     eax  

00602E7C  |.  83F8 06       |cmp     eax, 6  

00602E7F  |.^ 75 E8        \jnz     short 00602E69  
```

这段和上面相同，只是把用户号的后二位数字倒转过来再转成十六进制存入 `EDX` 中。

```
00602E81  |.  3BF2         cmp     esi, edx               *比较 ESI 和 EDX，这里肯定相等  
00602E83  |.  74 04        je      short 00602E89         单步运行  
00602E85  |.  33DB        xor     ebx, ebx  
00602E87  |.  EB 02       jmp     short 00602E8B  
00602E89  |>  8BD9        mov     ebx, ecx  
00602E8B  |>  33C0        xor     eax, eax  
00602E8D  |.  5A          pop     edx  
00602E8E  |.  59          pop     ecx  
00602E8F  |.  59          pop     ecx  
00602E90  |.  64:8910     mov     fs:[eax], edx  
00602E93  |.  68 AD2E6000  push    00602EAD  
00602E98  |>  8D45 F8     lea     eax, [ebp-8]  
00602E9B  |.  BA 02000000  mov     edx, 2  
00602EA0  |.  E8 4311E0FF  call    00403FE8  
00602EA5  \\.  C3         retn  
```

------------------------------------------------------------------------

```
0060303B  |.  8BD8        mov     ebx, eax             *将 EAX 中计算后的值（A）存入 EBX 中  
0060303D  |.  85DB        test    ebx, ebx             *看 EBX 中值是否为 0               
0060303F  |.  75 09       jnz     short 0060304A       *不为 0 则跳转  
00603041  |.  8BC6        mov     eax, esi  
00603043  |.  E8 7C0FE0FF  call    00403FC4  
00603048  |.  EB 27       jmp     short 00603071  
0060304A  |>  2B1D 10EB6300  sub     ebx, [63EB10]     *EBX=EBX-ds:[0063eb10]中的值，ds:[0063eb10]=35A63C6  
00603050  |.  895D F4     mov     [ebp-C], ebx         *将 EBX 中值存入地址 1  
00603053  |.  33C0        xor     eax, eax             *将 EAX 清零  
00603055  |.  8945 F8     mov     [ebp-8], eax  
00603058  |.  DF6D F4     fild    qword ptr [ebp-C]     *将地址 1 中的值转换成十进制数值（B）  
0060305B  |.  DC0D 14EB6300  fmul    qword ptr [63EB14] *将数值（B）乘以 ds:[63eb14] =0.920561220000 得值（C）       
00603061  |.  E8 32FCDFFF  call    00402C98             
00603066  |.  8BD8        mov     ebx, eax  
00603068  |.  8BD6        mov     edx, esi  
0060306A  |.  8BC3        mov     eax, ebx  
0060306C  |.  E8 67FCFFFF  call    00602CD8             *计算注册码过程，跟进  
```
-----------------------------------------------------------------------------------------------------

```
00602CD8  /$  55          push    ebp  
00602CD9  |.  8BEC        mov     ebp, esp  
00602CDB  |.  33C9        xor     ecx, ecx  
00602CDD  |.  51          push    ecx  
00602CDE  |.  51          push    ecx  
00602CDF  |.  51          push    ecx  
00602CE0  |.  51          push    ecx  
00602CE1  |.  51          push    ecx  
00602CE2  |.  53          push    ebx  
00602CE3  |.  56          push    esi  
00602CE4  |.  57          push    edi  
00602CE5  |.  8955 F8     mov     [ebp-8], edx  
00602CE8  |.  8945 FC     mov     [ebp-4], eax  
00602CEB  |.  33C0        xor     eax, eax  
00602CED  |.  55          push    ebp  
00602CEE  |.  68 B02D6000  push    00602DB0  
00602CF3  |.  64:FF30     push    dword ptr fs:[eax]  
00602CF6  |.  64:8920     mov     fs:[eax], esp  
00602CF9  |.  8D45 F4     lea     eax, [ebp-C]  
00602CFC  |.  E8 C312E0FF  call    00403FC4  
00602D01  |.  33FF        xor     edi, edi  
00602D03  |.  BB 06000000  mov     ebx, 6             *EBX=6  
00602D08  |>  8B45 FC     /mov    eax, [ebp-4]         *将 ds:[ebp-4]中值（C）存入 EAX 中  
00602D0B  |.  B9 0A000000  |mov    ecx, 0A             *ECX=A  
00602D10  |.  33D2        |xor     edx, edx            *EDX 清零  
00602D12  |.  F7F1        |div     ecx                 *EAX 除以 ECX，商存入 EAX，余数存入 EDX 中  
00602D14  |.  8BF2        |mov     esi, edx            *将 EDX 中的值存到 ESI 中  
00602D16  |.  03FE        |add     edi, esi            *将 ESI 中的值累加到 EDI 中  
00602D18  |.  8D45 F0     |lea     eax, [ebp-10]      
00602D1B  |.  8D56 30     |lea     edx, [esi+30]       *EDX=ESI+30 (每一次算出的值对应的 ASC 字符就是注册码的前六位）  
00602D1E  |.  E8 4914E0FF  |call    0040416C  
00602D23  |.  8B55 F0     |mov     edx, [ebp-10]  
00602D26  |.  8D45 F4     |lea     eax, [ebp-C]  
00602D29  |.  E8 2E15E0FF  |call    0040425C  
00602D2E  |.  8B45 FC     |mov     eax, [ebp-4]         *将 ds:[ebp-4]中值（C）存入 EAX 中  
00602D31  |.  B9 0A000000  |mov     ecx, 0A             *ECX=A  
00602D36  |.  33D2        |xor     edx, edx            *EDX 清零  
00602D38  |.  F7F1        |div     ecx                 *EAX 除以 ECX，商存入 EAX，余数存入 EDX 中  
00602D3A  |.  8945 FC     |mov     [ebp-4], eax         *将 EAX 中值存入 DS：[ebp-4]中  
00602D3D  |.  4B          |dec     ebx                 *EBX=EBX-1  
00602D3E  |.^ 75 C8       \jnz    short 00602D08       
```

其实这段可以把值（C）的十进制后六位数字倒转过来就是六位注册码，并把后六位数字累加存入 `EDI` 中。

```
00602D40  |.  BB 02000000  mov     ebx, 2             *EBX=2  
00602D45  |>  8BC7        /mov    eax, edi             *将 EDI 中的值存入 EAX 中  
00602D47  |.  B9 0A000000  |mov    ecx, 0A             *ECX=A  
00602D4C  |.  33D2        |xor     edx, edx            *EDX 清零  
00602D4E  |.  F7F1        |div     ecx                 *EAX 除以 ECX，商存入 EAX，余数存入 EDX 中  
00602D50  |.  8BF2        |mov     esi, edx            *将 EDX 中的值存到 ESI 中  
00602D52  |.  8D45 EC     |lea     eax, [ebp-14]      
00602D55  |.  8D56 30     |lea     edx, [esi+30]       *EDX=ESI+30 (算出的值对应的 ASC 字符就是注册码的后二位）  
00602D58  |.  E8 0F14E0FF  |call    0040416C  
00602D5D  |.  8B55 EC     |mov     edx, [ebp-14]  
00602D60  |.  8D45 F4     |lea     eax, [ebp-C]  
00602D63  |.  E8 F414E0FF  |call    0040425C  
00602D68  |.  8BC7        |mov     eax, edi             *将 EDI 中的值存入 EAX 中          
00602D6A  |.  B9 0A000000  |mov     ecx, 0A             *ECX=A   
00602D6F  |.  33D2        |xor     edx, edx            *EDX 清零  
00602D71  |.  F7F1        |div     ecx                 *EAX 除以 ECX，商存入 EAX，余数存入 EDX 中  
00602D73  |.  8BF8        |mov     edi, eax            *将 EAX 中求得的商存入 EDI 中  
00602D75  |.  4B          |dec     ebx  
00602D76  |.^ 75 CD       \jnz    short 00602D45  
```

将求得的前面六位注册码与后二注册码连起来就是八位的注册码，中间在第四位后面加入一个“-”就是最后的注册码。

```
00602D78  |.  8D55 F4     lea     edx, [ebp-C]  
00602D7B  |.  B9 05000000  mov     ecx, 5  
00602D80  |.  B8 C82D6000  mov     eax, 00602DC8       ;  -      *将“-”的值存入 EAX 中  
00602D85  |.  E8 5A17E0FF  call    004044E4             *在注册码第四位后面加入“-”       
00602D8A  |.  8B45 F8     mov     eax, [ebp-8]  
00602D8D  |.  8B55 F4     mov     edx, [ebp-C]  
00602D90  |.  E8 8312E0FF  call    00404018             *如果要直接得到注册码可以直接在这里下个断点就行了，大概按三下 F9，EDX 中就是注册码。  
00602D95  |.  33C0        xor     eax, eax  
00602D97  |.  5A          pop     edx  
00602D98  |.  59          pop     ecx  
00602D99  |.  59          pop     ecx  
00602D9A  |.  64:8910     mov     fs:[eax], edx  
00602D9D  |.  68 B72D6000  push    00602DB7  
```



```
  00602DA2  |>  8D45 EC       lea     eax, [ebp-14]  

  00602DA5  |.  BA 03000000   mov     edx, 3  

  00602DAA  |.  E8 3912E0FF   call    00403FE8  

  00602DAF  \.  C3            retn  

  00602DB0   .^ E9 EB0BE0FF   jmp     004039A0  

  00602DB5   .^ EB EB         jmp     short 00602DA2  

  00602DB7   .  5F            pop     edi  

  00602DB8   .  5E            pop     esi  

  00602DB9   .  5B            pop     ebx  

  00602DBA   .  8BE5          mov     esp, ebp  

  00602DBC   .  5D            pop     ebp  

  00602DBD   .  C3            retn  

  ---------------------------------------------------------------------------------------------  

  00603071  |>  33C0          xor     eax, eax  

  00603073  |.  5A            pop     edx  

  00603074  |.  59            pop     ecx  

  00603075  |.  59            pop     ecx  

  00603076  |.  64:8910       mov     fs:[eax], edx  

  00603079  |.  68 8E306000   push    0060308E  

  0060307E  |>  8D45 FC       lea     eax, [ebp-4]  

  00603081  |.  E8 3E0FE0FF   call    00403FC4  

  00603086  \.  C3            retn  

  ------------------------------------------------------------------------------------------  

  006030CA  |.  8B45 F8       mov     eax, [ebp-8]  

  006030CD  |.  E8 FAFCFFFF   call    00602DCC  

  006030D2  |.  8BD8          mov     ebx, eax  

  006030D4  |.  8B45 FC       mov     eax, [ebp-4]  

  006030D7  |.  E8 F0FCFFFF   call    00602DCC  

  006030DC  |.  85C0          test    eax, eax  

  006030DE  |.  75 04         jnz     short 006030E4  

  006030E0  |.  33DB          xor     ebx, ebx  

  006030E2  |.  EB 0A         jmp     short 006030EE  

  006030E4  |>  3BC3          cmp     eax, ebx  

  006030E6  |.  75 04         jnz     short 006030EC  

  006030E8  |.  B3 01         mov     bl, 1  

  006030EA  |.  EB 02         jmp     short 006030EE  

  006030EC  |>  33DB          xor     ebx, ebx  

  006030EE  |>  33C0          xor     eax, eax  

  006030F0  |.  5A            pop     edx  

  006030F1  |.  59            pop     ecx  

  006030F2  |.  59            pop     ecx  

  006030F3  |.  64:8910       mov     fs:[eax], edx  

  006030F6  |.  68 10316000   push    00603110  

  006030FB  |>  8D45 F4       lea     eax, [ebp-C]  

  006030FE  |.  BA 03000000   mov     edx, 3  

  00603103  |.  E8 E00EE0FF   call    00403FE8  

  00603108  \.  C3            retn  

  -----------------------------------------------------------------------------------------------  

  00604196  |.  84C0          test    al, al  

  00604198  |.  74 1D         je      short 006041B7  

  0060419A  |.  8D55 F4       lea     edx, [ebp-C]  

  0060419D  |.  A1 14F96300   mov     eax, [63F914]  

  006041A2  |.  8B00          mov     eax, [eax]  

  006041A4  |.  E8 8BF5FFFF   call    00603734  

  006041A9  |.  8B55 F4       mov     edx, [ebp-C]  

  006041AC  |.  8D83 54030000 lea     eax, [ebx+354]  

  006041B2  |.  E8 61FEDFFF   call    00404018  

  006041B7  |>  8D55 F0       lea     edx, [ebp-10]  

  006041BA  |.  A1 14F96300   mov     eax, [63F914]  

  006041BF  |.  8B00          mov     eax, [eax]  

  006041C1  |.  E8 6EF5FFFF   call    00603734  

  006041C6  |.  8B45 F0       mov     eax, [ebp-10]  

  006041C9  |.  E8 C6EEFFFF   call    00603094  

  006041CE  |.  84C0          test    al, al  

  006041D0  |.  74 1F         je      short 006041F1  

  006041D2  |.  BA 9C426000   mov     edx, 0060429C                    ;  软件已经成功注册！  

  006041D7  |.  8B83 08030000 mov     eax, [ebx+308]  

  006041DD  |.  E8 863AE3FF   call    00437C68  

  006041E2  |.  33D2          xor     edx, edx  

  006041E4  |.  8B83 04030000 mov     eax, [ebx+304]  

  006041EA  |.  E8 6139E3FF   call    00437B50  

  006041EF  |.  EB 72         jmp     short 00604263  

--------------------------------------------------------------------------------
```

## 【经验总结】

第一次发破文，好多地方可能没讲明白，请大家多多包涵……

注册机我是用 `VB` 编的，而且 `VB` 编程也只懂些皮毛，很乱，所以就不贴出来找骂了。

## 【版权声明】

本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2006 年 07 月 25 日 11:01:54

`4dd466d0e1e0cc123afbe504da2462ca`
