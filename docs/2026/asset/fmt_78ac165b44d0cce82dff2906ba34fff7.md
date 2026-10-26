
# 注册算法分析

```assembly
0042F690  |.  7D 12         jge     short AXE.0042F6A4               ;  大于等于4,跳

0042F692  |.  68 2C415100   push    AXE.0051412C                     ;  abcd

0042F697  |.  8D4C24 28     lea     ecx, dword ptr [esp+28]

0042F69B  |.  E8 9D450800   call    AXE.004B3C3D

0042F6A0  |.  8B4424 24     mov     eax, dword ptr [esp+24]          ;  注册名长度小于4,则注册名+'abcd' -> eax

0042F6A4  |>  8B48 F8       mov     ecx, dword ptr [eax-8]           ;  注册名长度 -> ecx

0042F6A7  |.  8A58 02       mov     bl, byte ptr [eax+2]             ;  取注册名的第3位字符码 -> bl(ebx低位)

0042F6AA  |.  8A10          mov     dl, byte ptr [eax]               ;  取注册名的第1位字符码 -> dl(edx低位)

0042F6AC  |.  0FBE4401 FF   movsx   eax, byte ptr [ecx+eax-1]        ;  取注册名的最后位字符码 -> eax

0042F6B1  |.  0FBEFB        movsx   edi, bl                          ;  edi = bl;

0042F6B4  |.  0FBED2        movsx   edx, dl                          ;  edx = dl;

0042F6B7  |.  03C7          add     eax, edi                         ;  eax := eax + edi;

0042F6B9  |.  03C2          add     eax, edx                         ;  eax := eax + edx;

0042F6BB  |.  0FAFC1        imul    eax, ecx                         ;  eax := eax * ecx;

0042F6BE  |.  25 FFFFFF8F   and     eax, 8FFFFFFF                    ;  eax := eax and $8FFFFFFF;

0042F6C3  |.  79 07         jns     short AXE.0042F6CC               ;  为0? 跳!

0042F6C5  |.  48            dec     eax

0042F6C6  |.  0D 000000F0   or      eax, F0000000

0042F6CB  |.  40            inc     eax

0042F6CC  |>  81FE 00000010 cmp     esi, 10000000                    ;  esi 比较 $10000000

0042F6D2  |.  76 05         jbe     short AXE.0042F6D9               ;  小于或等于,跳

0042F6D4  |.  05 00000010   add     eax, 10000000                    ;  大于: eax:= eax + $10000000;

0042F6D9  |>  35 D7139A03   xor     eax, 39A13D7                     ;  eax := eax xor $39A13D7

0042F6DE  |.  33C9          xor     ecx, ecx                         ;  ecx 清零 ecx := 0;

0042F6E0  |.  3BF0          cmp     esi, eax                         ;  esi 比较 eax,『『这里相等便OK』』

0042F6E2  |.  C74424 14 FFF>mov     dword ptr [esp+14], -1

0042F6EA  |.  0F94C1        sete    cl                               ;  cl 设置1

0042F6ED  |.  8BF1          mov     esi, ecx                         ;  注意: esi := ecx;

0042F6EF  |.  8D4C24 24     lea     ecx, dword ptr [esp+24]

0042F6F3  |.  E8 69410800   call    AXE.004B3861

0042F6F8  |.  8BC6          mov     eax, esi                         ;  eax := esi; //最终结果

0042F6FA  |.  8B4C24 0C     mov     ecx, dword ptr [esp+C]

0042F6FE  |.  64:890D 00000>mov     dword ptr fs:[0], ecx

0042F705  |.  5F            pop     edi

0042F706  |.  5E            pop     esi

0042F707  |.  5B            pop     ebx

0042F708  |.  83C4 0C       add     esp, 0C

0042F70B  |.  C2 0C00       retn    0C

0042F70E  |>  8D4C24 24     lea     ecx, dword ptr [esp+24]

0042F712  |.  C74424 14 FFF>mov     dword ptr [esp+14], -1

0042F71A  |.  E8 42410800   call    AXE.004B3861

0042F71F  |.  8B4C24 0C     mov     ecx, dword ptr [esp+C]

0042F723  |.  5F            pop     edi

0042F724  |.  5E            pop     esi

0042F725  |.  33C0          xor     eax, eax

0042F727  |.  64:890D 00000>mov     dword ptr fs:[0], ecx

0042F72E  |.  5B            pop     ebx

0042F72F  |.  83C4 0C       add     esp, 0C

0042F732  \.  C2 0C00       retn    0C
```

通过上面的分析，我们知道了其注册算法为：读入注册名和注册码，先将注册码转换为16进制数，放入`esi`中；然后读入注册名，取注册名的第3位字符，放入`edi`中，第1位字符放入`edx`中，注册名的最后位字符放入`eax`中，然后将这三个字符码相加，结果再与`$8FFFFFFF`做`and`运算，得到的结果再异或`$39A13D7`后得到最终结果，放入`eax`中。用这个结果比较`esi`中的注册码，相等，就注册成功了！

## 可用 Key

注册名: `aCaFeeL`

注册码: `60429493`

注册机的算法在多台机器上使用不同注册名通过验证，但没有时间再优化它了，应该不存在bug的问题，附件中为我打包后的注册机源代码。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [AXE341.gif](javascript:void(0);)

  （31.72kb，219次下载）

- [keygen.for.AXE.3.41.src.rar](javascript:void(0);)

  （6.28kb，58次下载）
