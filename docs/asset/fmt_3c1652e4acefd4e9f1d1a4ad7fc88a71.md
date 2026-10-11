
```assembly
004DF6A6   .  8B45 FC       mov     eax, [ebp-4]  
004DF6A9   .  C780 4C020000>mov     dword ptr [eax+24C], 1  
004DF6B3   >  33C0         xor     eax, eax  
004DF6B5   .  5A         pop     edx  
004DF6B6   .  59         pop     ecx  
004DF6B7   .  59         pop     ecx  
004DF6B8   .  64:8910    mov     fs:[eax], edx  
004DF6BB   .  68 12F74D00  push    004DF712  
004DF6C0   >  8D45 CC    lea     eax, [ebp-34]  
004DF6C3   .  BA 03000000 mov     edx, 3  
004DF6C8   .  E8 5F4CF2FF call    0040432C  
004DF6CD   .  8D45 D8    lea     eax, [ebp-28]  
004DF6D0   .  E8 334CF2FF call    00404308  
004DF6D5   .  8D45 DC    lea     eax, [ebp-24]  
004DF6D8   .  E8 2B4CF2FF call    00404308  
004DF6DD   .  8D45 E0    lea     eax, [ebp-20]  
004DF6E0   .  E8 234CF2FF call    00404308  
004DF6E5   .  8D45 E4    lea     eax, [ebp-1C]  
004DF6E8   .  E8 1B4CF2FF call    00404308  
004DF6ED   >  8D45 E8    lea     eax, [ebp-18]  
004DF6F0   .  E8 134CF2FF call    00404308  
004DF6F5   .  8D45 EC    lea     eax, [ebp-14]  
004DF6F8   .  E8 0B4CF2FF call    00404308  
004DF6FD   >  8D45 F0    lea     eax, [ebp-10]  
004DF700   .  BA 02000000 mov     edx, 2  
004DF705   .  E8 224CF2FF call    00404308  
004DF70A   .  C3         retn  
004DF70B   .^ E9 1C45F2FF  jmp     00403C2C  
004DF710   .^ EB AE     jmp     short 004DF6C0  
004DF712   .  8BE5       mov     esp, ebp  
004DF714   .  5D         pop     ebp  
004DF715   .  C3         retn  
```

================ `004DF5B2` `. ` `E8 514AFFFF` `call` `004D4008` ================

```assembly
004D4008  /$  55         push    ebp                       ; 来到这里  
004D4009  |.  8BEC       mov     ebp, esp  
004D400B  |.  83C4 F0    add     esp, -10  
004D400E  |.  53         push    ebx  
004D400F  |.  56         push    esi  
004D4010  |.  57         push    edi  
004D4011  |.  33C9       xor     ecx, ecx  
004D4013  |.  894D F4    mov     [ebp-C], ecx  
004D4016  |.  8955 F8    mov     [ebp-8], edx  
004D4019  |.  8945 FC    mov     [ebp-4], eax              ; (ASCII "lzq1973")  
004D401C  |.  8B45 FC    mov     eax, [ebp-4]  
004D401F  |.  E8 9407F3FF call    004047B8  
004D4024  |.  33C0       xor     eax, eax  
004D4026  |.  55         push    ebp  
004D4027  |.  68 AF404D00 push    004D40AF  
004D402C  |.  64:FF30    push    dword ptr fs:[eax]  
004D402F  |.  64:8920    mov     fs:[eax], esp  
004D4032  |.  8D45 F4    lea     eax, [ebp-C]  
004D4035  |.  BA C8404D00 mov     edx, 004D40C8           ; 字符串常量(ASCII "qwertyuiopasdfghjklzxcvbnm")  
004D403A  |.  E8 6103F3FF call    004043A0  
004D403F  |.  BE F57E0800 mov     esi, 87EF5               ; 常数（87EF5）赋给 ESI  
004D4044  |.  8B45 FC    mov     eax, [ebp-4]  
004D4047  |.  E8 7C05F3FF call    004045C8  
004D404C  |.  85C0       test    eax, eax                  ; 取用户名长度  
004D404E  |.  7E 37      jle     short 004D4087  
004D4050  |.  8945 F0    mov     [ebp-10], eax  
004D4053  |.  BB 01000000 mov     ebx, 1  
004D4058  |>  8B45 F4    /mov     eax, [ebp-C]             ; 常量赋给 EAX  
004D405B  |.  E8 6805F3FF |call     004045C8  
004D4060  |.  85C0       |test    eax, eax                ; 取字符串常量长度  
004D4062  |.  7E 1D      |jle     short 004D4081  
004D4064  |.  BA 01000000|mov     edx, 1  
004D4069  |.  8B4D FC    |.mov     ecx, [ebp-4]            ; 用户名赋给 ECX  
004D406C  |.  0FB64C19 FF||movzx   ecx, byte ptr [ecx+ebx-1]  ; 取当前字符  
004D4071  |.  8B7D F4    ||mov     edi, [ebp-C]  
004D4074  |.  0FB67C17 FF||movzx   edi, byte ptr [edi+edx-1]  
004D4079  |.  03CF       ||add     ecx, edi               ; 用户名首字符 16 进制＋当前常量的 16 进制（从左至右逐位取）  
004D407B  |.  03F1       ||add     esi, ecx               ; ESI＝ESI＋ECX  
004D407D  |.  42         ||inc     edx                    ; 递增  
004D407E  |.  48         ||dec     eax                    ; 递减  
004D407F  |.^ 75 E8      |\jnz     short 004D4069  
004D4081  |>  43         |inc     ebx                    ; 逐位取用户名  
004D4082  |.  FF4D F0    |dec     dword ptr [ebp-10]       ; 还剩几位  
004D4085  |.^ 75 D1      \jnz     short 004D4058  
004D4087  |>  8B55 F8    mov     edx, [ebp-8]  
004D408A  |.  8BC6       mov     eax, esi  
004D408C  |.  E8 074DF3FF call    00408D98  
004D4091  |.  33C0       xor     eax, eax  
004D4093  |.  5A         pop     edx  
004D4094  |.  59         pop     ecx  
004D4095  |.  59         pop     ecx  
004D4096  |.  64:8910    mov     fs:[eax], edx  
004D4099  |.  68 B6404D00 push    004D40B6  
004D409E  |>  8D45 F4    lea     eax, [ebp-C]  
004D40A1  |.  E8 6202F3FF call    00404308  
004D40A6  |.  8D45 FC    lea     eax, [ebp-4]  
004D40A9  |.  E8 5A02F3FF call    00404308  
004D40AE  \\.  C3        retn  
004D40AF   .^ E9 78FBF2FF jmp     00403C2C  
004D40B4   .^ EB E8     jmp     short 004D409E  
004D40B6   .  5F        pop     edi  
004D40B7   .  5E        pop     esi  
004D40B8   .  5B        pop     ebx  
004D40B9   .  8BE5      mov     esp, ebp  
004D40BB   .  5D        pop     ebp  
004D40BC   .  C3        retn  
```

------------------------------------------------------------------------

**【破解总结】**

名码比较，算法也简单，如下：

用户名各字符的 16 进制依次与字串（`qwertyuiopasdfghjklzxcvbnm`）的各字符 16 进制和累加后再加上常数（`87EF5`）后转为 10 进制就是注册码。

根据飘云老大的注册机教程，试了试，代码如下：

```
.如果 (编辑框 1.内容 ≠ "")  
    用户名长度 ＝ 取文本长度 (编辑框 1.内容)  
    字符串常量 ＝ “qwertyuiopasdfghjklzxcvbnm”  
    字符串常量长度 ＝ 取文本长度 (字符串常量)  
    .计次循环首 (用户名长度, i)  
        c ＝ 取代码 (编辑框 1.内容, i)  
        .计次循环首 (字符串常量长度, ii)  
            cc ＝ 取代码 (字符串常量, ii)  
            Sn ＝ Sn ＋ c ＋ cc  
        .计次循环尾 ()  
    .计次循环尾 ()  
.否则  
    信息框 (“请输入用户名”, 0, )  
.如果结束  
编辑框 2.内容 ＝ 到文本 (Sn ＋ 556789)  

.子程序 _按钮 2_ 被单击  

信息框 (“制作：lzq1973[PYG][CZG][OCN][DFCG][D.4s]”, 0, “感谢飘云老大”)  
```

收工，首次作算法注册机，痛快～～

------------------------------------------------------------------------

**【版权声明】** 本文纯属技术交流, 转载请注明作者信息并保持文章的完整, 谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`
