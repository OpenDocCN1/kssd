
# ChmMaker 追注册码过程（明码比较）

> 原文：[`www.pediy.com/kssd/pediy12/131786.html`](https://www.pediy.com/kssd/pediy12/131786.html)

运行程序，点注册软件，输入注册码，提示“重新运行程序！”，显然是重启验证的类型。用 `PEID` 查壳，发现是 `ASPack 2.12 -> Alexey Solodovnikov`。

用 `ESP` 定律脱壳。

载入 `OD`：提示代码段加壳，是否分析，点否。

我们来到下面的地址：

```
0057D001  >  60               pushad
0057D002    E8 03000000      call    0057D00A
0057D007  - E9 EB045D45      jmp     45B4D4F7
0057D00C    55               push    ebp
0057D00D    C3               retn
0057D00E    E8 01000000      call    0057D014
0057D013    EB 5D            jmp     short 0057D072
0057D015    BB EDFFFFFF      mov     ebx, -13
0057D01A    03DD             add     ebx, ebp
0057D01C    81EB 00D01700    sub     ebx, 17D000
0057D022    83BD 22040000 0> cmp     dword ptr [ebp+422], 0
0057D029    899D 22040000    mov     dword ptr [ebp+422], ebx
0057D02F    0F85 65030000    jnz     0057D39A
0057D035    8D85 2E040000    lea     eax, dword ptr [ebp+42E]
0057D03B    50               push    eax
0057D03C    FF95 4D0F0000    call    dword ptr [ebp+F4D]
0057D042    8985 26040000    mov     dword ptr [ebp+426], eax
0057D048    8BF8             mov     edi, eax
0057D04A    8D5D 5E          lea     ebx, dword ptr [ebp+5E]
0057D04D    53               push    ebx
0057D04E    50               push    eax
0057D04F    FF95 490F0000    call    dword ptr [ebp+F49]
0057D055    8985 4D050000    mov     dword ptr [ebp+54D], eax
0057D05B    8D5D 6B          lea     ebx, dword ptr [ebp+6B]
0057D05E    53               push    ebx
0057D05F    57               push    edi
0057D060    FF95 490F0000    call    dword ptr [ebp+F49]
0057D066    8985 51050000    mov     dword ptr [ebp+551], eax
0057D06C    8D45 77          lea     eax, dword ptr [ebp+77]
0057D06F    FFE0             jmp     eax
0057D071    56               push    esi
0057D072    6972 74 75616C4> imul    esi, dword ptr [edx+74], 416C617>
0057D079    6C               ins     byte ptr es:[edi], dx
0057D07A    6C               ins     byte ptr es:[edi], dx
0057D07B    6F               outs    dx, dword ptr es:[edi]
0057D07C    6300             arpl    word ptr [eax], ax
0057D07E    56               push    esi
0057D07F    6972 74 75616C4> imul    esi, dword ptr [edx+74], 466C617>
0057D086    72 65            jb      short 0057D0ED
0057D088    65:008B 9D31050> add     byte ptr gs:[ebx+5319D], cl
0057D08F    000B             add     byte ptr [ebx], cl
```

单步走，发现到 `0057D002    E8 03000000      call    0057D00A`

首次 `ESP` 变红，复制 `ESP` 中的地址，下断 `hr 0012FFA4` 后，`F9` 运行，我们来到这里：

```
0057D3B0   /75 08           jnz     short 0057D3BA
0057D3B2   |B8 01000000     mov     eax, 1
0057D3B7   |C2 0C00         retn    0C
0057D3BA   \68 50FD4C00     push    004CFD50
0057D3BF    C3              retn
0057D3C0    8B85 26040000   mov     eax, dword ptr [ebp+426]
0057D3C6    8D8D 3B040000   lea     ecx, dword ptr [ebp+43B]
0057D3CC    51              push    ecx
0057D3CD    50              push    eax
0057D3CE    FF95 490F0000   call    dword ptr [ebp+F49]
0057D3D4    8985 55050000   mov     dword ptr [ebp+555], eax
0057D3DA    8D85 47040000   lea     eax, dword ptr [ebp+447]
0057D3E0    50              push    eax
0057D3E1    FF95 510F0000   call    dword ptr [ebp+F51]
0057D3E7    8985 2A040000   mov     dword ptr [ebp+42A], eax
0057D3ED    8D8D 52040000   lea     ecx, dword ptr [ebp+452]
0057D3F3    51              push    ecx
0057D3F4    50              push    eax
```

继续 `F8` 单步走，几步后我们来到这里：

```
004CFD50    55               push    ebp
004CFD51    8BEC             mov     ebp, esp
004CFD53    83C4 F0          add     esp, -10
004CFD56    53               push    ebx
004CFD57    B8 D0F94C00      mov     eax, 004CF9D0
004CFD5C    E8 A76BF3FF      call    00406908
004CFD61    8B1D 98304D00    mov     ebx, dword ptr [4D3098]           ; ChmMaker.004D4C34
004CFD67    8B03             mov     eax, dword ptr [ebx]
004CFD69    E8 F645F9FF      call    00464364
004CFD6E    8B03             mov     eax, dword ptr [ebx]
004CFD70    BA C8FD4C00      mov     edx, 004CFDC8
004CFD75    E8 F641F9FF      call    00463F70
004CFD7A    8B0D C42F4D00    mov     ecx, dword ptr [4D2FC4]           ; ChmMaker.004D51D8
004CFD80    8B03             mov     eax, dword ptr [ebx]
004CFD82    8B15 B4D44C00    mov     edx, dword ptr [4CD4B4]           ; ChmMaker.004CD500
004CFD88    E8 EF45F9FF      call    0046437C
004CFD8D    8B0D E8314D00    mov     ecx, dword ptr [4D31E8]           ; ChmMaker.004D51D0
```

这里应该就到程序的 `OEP` 了，脱壳，保存。双击运行，没问题。

用 `OD` 载入脱壳后的文件，查找字符串，`Ultra String Reference`，条目 `510`

```
Address=004CEF73
Disassembly=mov      edx, 004CF04C
Text String= - 未购买用户
```

双击跟进，找到关键部分下好断点，运行我们来到这里：

```
004CEEF2  |.  B1 01          mov     cl, 1
004CEEF4  |.  BA F8EF4C00    mov     edx, 004CEFF8                    ;  software\chmmaker  //打开注册表
004CEEF9  |.  8BC3           mov     eax, ebx
004CEEFB  |.  E8 84ABF9FF    call    00469A84
004CEF00  |.  8D4D F0        lea     ecx, dword ptr [ebp-10]
004CEF03  |.  BA 14F04C00    mov     edx, 004CF014                    ;  reguser   //注册的用户名
004CEF08  |.  8BC3           mov     eax, ebx
004CEF0A  |.  E8 3DADF9FF    call    00469C4C
004CEF0F  |.  8B55 F0        mov     edx, dword ptr [ebp-10]
004CEF12  |.  B8 E0514D00    mov     eax, 004D51E0                    ;  ASCII "\/?
004CEF17  |.  E8 FC55F3FF    call    00404518
004CEF1C  |.  8D4D EC        lea     ecx, dword ptr [ebp-14]
004CEF1F  |.  BA 24F04C00    mov     edx, 004CF024                    ;  regno     //保存的注册码
004CEF24  |.  8BC3           mov     eax, ebx
004CEF26  |.  E8 21ADF9FF    call    00469C4C
004CEF2B  |.  8B45 EC        mov     eax, dword ptr [ebp-14]          ;  这里出现注册码（真码）
004CEF2E  |.  50             push    eax
004CEF2F  |.  8D4D E8        lea     ecx, dword ptr [ebp-18]
004CEF32  |.  BA 34F04C00    mov     edx, 004CF034                    ;  chmmakerchina
004CEF37  |.  A1 E0514D00    mov     eax, dword ptr [4D51E0]
004CEF3C  |.  E8 97F2FCFF    call    0049E1D8
004CEF41  |.  8B55 E8        mov     edx, dword ptr [ebp-18]
004CEF44  |.  58             pop     eax
004CEF45  |.  E8 7659F3FF    call    004048C0
004CEF4A  |.  75 07          jnz     short 004CEF53
004CEF4C  |.  C605 DC514D00> mov     byte ptr [4D51DC], 1
004CEF53  |>  8BC3           mov     eax, ebx
004CEF55  |.  E8 DE47F3FF    call    00403738
004CEF5A  |.  803D DC514D00> cmp     byte ptr [4D51DC], 0
004CEF61  |.  75 27          jnz     short 004CEF8A                   ;  关键跳，跳了就挂！爆破直接改 jmp
004CEF63  |.  8D55 E4        lea     edx, dword ptr [ebp-1C]
004CEF66  |.  A1 D8514D00    mov     eax, dword ptr [4D51D8]
004CEF6B  |.  E8 7054F7FF    call    004443E0
004CEF70  |.  8D45 E4        lea     eax, dword ptr [ebp-1C]
004CEF73  |.  BA 4CF04C00    mov     edx, 004CF04C                    ;   - 未购买用户
004CEF78  |.  E8 0758F3FF    call    00404784
004CEF7D  |.  8B55 E4        mov     edx, dword ptr [ebp-1C]
004CEF80  |.  A1 D8514D00    mov     eax, dword ptr [4D51D8]
004CEF85  |.  E8 8654F7FF    call    00444410
004CEF8A  |>  33C0           xor     eax, eax
004CEF8C  |.  5A             pop     edx
004CEF8D  |.  59             pop     ecx
004CEF8E  |.  59             pop     ecx
004CEF8F  |.  64:8910        mov     dword ptr fs:[eax], edx
004CEF92  |.  68 B4EF4C00    push    004CEFB4
004CEF97  |>  8D45 E4        lea     eax, dword ptr [ebp-1C]
004CEF9A  |.  E8 2555F3FF    call    004044C4
004CEF9F  |.  8D45 E8        lea     eax, dword ptr [ebp-18]
004CEFA2  |.  BA 06000000    mov     edx, 6
004CEFA7  |.  E8 3C55F3FF    call    004044E8
004CEFAC  \\.  C3             retn
004CEFAD   .^ E9 1A4FF3FF    jmp     00403ECC
```



## 汇编代码

```
004CEFB2   .^ EB E3         jmp     short 004CEF97  

004CEFB4   .  5B           pop     ebx  

004CEFB5   .  8BE5         mov     esp, ebp  

004CEFB7   .  5D           pop     ebp  

004CEFB8   .  C3           retn  
```

## 注册信息

【注册名】：`sun152121`

【注册码】：`62DDF43AAEB12C7E087F495EB972713C`

## 爆破方法

软件如果爆破的话，将 `004CEF61 |. /75 27         jnz     short 004CEF8A ;` 改成 `004CEF61 |. /75 27    jmp     short 004CEF8A` 即可。

## 总结

这个软件是明码比较的软件，爆破也很简单。如果感兴趣也可以做个内存注册机。
