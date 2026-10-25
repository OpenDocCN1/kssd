
# 屏幕录像专家 V6.0 Build20070123 算法分析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-41.htm`](https://www.pediy.com/kssd/pediy09/pediy09-41.htm)

> 【文章标题】: 屏幕录像专家 V6.0 Build20070123 算法分析

> 【文章作者】: `the0crat`

> 【作者邮箱】: `the0crat.cn_at_gmail.com`

> 【作者主页】: `http://the0crat.blogcn.com`

> 【生产日期】: `20070205`

> 【软件名称】: 屏幕录像专家 V6.0 Build20070123

> 【保护方式】: 注册码

> 【编写语言】: `Borland C++ 1999`

> 【使用工具】: `OD`

> 【作者声明】: 本文仅供研究学习，本人对因这篇文章而导致的一切后果，不承担任何法律责任。本文中的不足之处请各位多多指教

> 【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

---

## 详细过程

无壳，常规的断点一路追到这里，即验证过程

```asm
0043942C  /.  55            push    ebp
0043942D  |.  8BEC          mov     ebp, esp
0043942F  |.  81C4 C0FEFFFF add     esp, -140
00439435  |.  53            push    ebx
00439436  |.  56            push    esi
00439437  |.  57            push    edi
00439438  |.  8985 3CFFFFFF mov     dword ptr [ebp-C4], eax
0043943E  |.  B8 60065100   mov     eax, 00510660
00439443  |.  E8 B4560A00   call    004DEAFC
00439448  |.  66:C785 50FFF>mov     word ptr [ebp-B0], 8
00439451  |.  8D45 FC       lea     eax, dword ptr [ebp-4]
00439454  |.  E8 CB83FCFF   call    00401824
00439459  |.  8BD0          mov     edx, eax
0043945B  |.  FF85 5CFFFFFF inc     dword ptr [ebp-A4]
00439461  |.  8B8D 3CFFFFFF mov     ecx, dword ptr [ebp-C4]
00439467  |.  8B81 E4020000 mov     eax, dword ptr [ecx+2E4]
0043946D  |.  E8 56270700   call    004ABBC8
00439472  |.  8D55 FC       lea     edx, dword ptr [ebp-4]
00439475  |.  8D45 F8       lea     eax, dword ptr [ebp-8]
00439478  |.  8B0A          mov     ecx, dword ptr [edx]
0043947A  |.  51            push    ecx
0043947B  |.  E8 A483FCFF   call    00401824
00439480  |.  8BD0          mov     edx, eax
00439482  |.  FF85 5CFFFFFF inc     dword ptr [ebp-A4]
00439488  |.  8B8D 3CFFFFFF mov     ecx, dword ptr [ebp-C4]
0043948E  |.  8B81 DC020000 mov     eax, dword ptr [ecx+2DC]
00439494  |.  E8 2F270700   call    004ABBC8
00439499  |.  8D55 F8       lea     edx, dword ptr [ebp-8]
0043949C  |.  8B0A          mov     ecx, dword ptr [edx]
0043949E  |.  51            push    ecx
0043949F  |.  8B85 3CFFFFFF mov     eax, dword ptr [ebp-C4]
004394A5  |.  50            push    eax
004394A6  |.  E8 2D0D0000   call    0043A1D8
004394AB  |.  83C4 0C       add     esp, 0C
004394AE  |.  3C 01         cmp     al, 1
004394B0  |.  0F94C2        sete    dl
004394B3  |.  83E2 01       and     edx, 1
004394B6  |.  8D45 F8       lea     eax, dword ptr [ebp-8]
004394B9  |.  52            push    edx
004394BA  |.  BA 02000000   mov     edx, 2
004394BF  |.  FF8D 5CFFFFFF dec     dword ptr [ebp-A4]
004394C5  |.  E8 9A110B00   call    004EA664
004394CA  |.  FF8D 5CFFFFFF dec     dword ptr [ebp-A4]              ; |
004394D0  |.  8D45 FC       lea     eax, dword ptr [ebp-4]         ; |
004394D3  |.  BA 02000000   mov     edx, 2                          ; |
004394D8  |.  E8 87110B00   call    004EA664                        ; \\屏录专家.004EA664
004394DD  |.  59            pop     ecx
004394DE  |.  84C9          test    cl, cl
004394E0  |.  74 48         je      short 0043952A                  ;  判断注册码是不是现有版本的注册码
004394E2  |.  66:C785 50FFF>mov     word ptr [ebp-B0], 14
004394EB  |.  BA 14035100   mov     edx, 00510314
004394F0  |.  8D45 F4       lea     eax, dword ptr [ebp-C]
004394F3  |.  E8 340F0B00   call    004EA42C
004394F8  |.  FF85 5CFFFFFF inc     dword ptr [ebp-A4]
004394FE  |.  8B00          mov     eax, dword ptr [eax]
00439500  |.  E8 ABD20600   call    004A67B0
00439505  |.  FF8D 5CFFFFFF dec     dword ptr [ebp-A4]
0043950B  |.  8D45 F4       lea     eax, dword ptr [ebp-C]
0043950E  |.  BA 02000000   mov     edx, 2
00439513  |.  E8 4C110B00   call    004EA664
00439518  |.  8B8D 40FFFFFF mov     ecx, dword ptr [ebp-C0]
0043951E  |.  64:890D 00000>mov     dword ptr fs:[0], ecx
00439525  |.  E9 920C0000   jmp     0043A1BC
0043952A  |>  6A 14         push    14                              ; /Arg3 = 00000014
0043952C  |.  6A 00         push    0                                 ; |Arg2 = 00000000
0043952E  |.  8D85 FCFEFFFF lea     eax, dword ptr [ebp-104]          ; |
00439534  |.  50            push    eax                                 ; |Arg1
00439535  |.  E8 6E520A00   call    004DE7A8                          ; \\屏录专家.004DE7A8
0043953A  |.  83C4 0C       add     esp, 0C
0043953D  |.  33FF          xor     edi, edi
0043953F  |.  6A 14         push    14                              ; /Arg3 = 00000014
00439541  |.  6A 00         push    0                                 ; |Arg2 = 00000000
00439543  |.  8D85 14FFFFFF lea     eax, dword ptr [ebp-EC]           ; |
00439549  |.  50            push    eax                                 ; |Arg1
0043954A  |.  E8 59520A00   call    004DE7A8                          ; \\屏录专家.004DE7A8
0043954F  |.  83C4 0C       add     esp, 0C
00439552  |.  8D95 E4FEFFFF lea     edx, dword ptr [ebp-11C]
00439558  |.  6A 14         push    14                              ; /Arg3 = 00000014
0043955A  |.  6A 00         push    0                                 ; |Arg2 = 00000000
0043955C  |.  52            push    edx                                 ; |Arg1
0043955D  |.  E8 46520A00   call    004DE7A8                          ; \\屏录专家.004DE7A8
00439562  |.  83C4 0C       add     esp, 0C
00439565  |.  66:C785 50FFF>mov     word ptr [ebp-B0], 20
0043956E  |.  8D45 F0       lea     eax, dword ptr [ebp-10]
00439571  |.  E8 AE82FCFF   call    00401824
00439576  |.  8BD0          mov     edx, eax
00439578  |.  FF85 5CFFFFFF inc     dword ptr [ebp-A4]
0043957E  |.  8B8D 3CFFFFFF mov     ecx, dword ptr [ebp-C4]
00439584  |.  8B81 DC020000 mov     eax, dword ptr [ecx+2DC]
0043958A  |.  E8 39260700   call    004ABBC8
0043958F  |.  8D45 F0       lea     eax, dword ptr [ebp-10]
00439592  |.  E8 B5B9FCFF   call    00404F4C                          ;  取用户名
00439597  |.  57            push    edi
00439598  |.  8BF8          mov     edi, eax
0043959A  |.  33C0          xor     eax, eax
0043959C  |.  83C9 FF       or      ecx, FFFFFFFF
0043959F  |.  F2:AE         repne   scas  byte ptr es:[edi]
004395A1  |.  F7D1          not     ecx
004395A3  |.  2BF9          sub     edi, ecx
004395A5  |.  8DB5 E4FEFFFF lea     esi, dword ptr [ebp-11C]
004395AB  |.  87F7          xchg    edi, esi
004395AD  |.  8BD1          mov     edx, ecx
004395AF  |.  8BC7          mov     eax, edi
004395B1  |.  C1E9 02       shr     ecx, 2
004395B4  |.  8D45 F0       lea     eax, dword ptr [ebp-10]
004395B7  |.  F3:A5         rep     movs  dword ptr es:[edi], dword p>
004395B9  |.  8BCA          mov     ecx, edx
004395BB  |.  BA 02000000   mov     edx, 2
004395C0  |.  83E1 03       and     ecx, 3
004395C3  |.  F3:A4         rep     movs  byte ptr es:[edi], byte ptr>
004395C5  |.  5F            pop     edi
004395C6  |.  FF8D 5CFFFFFF dec     dword ptr [ebp-A4]
004395CC  |.  E8 93100B00   call    004EA664
004395D1  |.  66:C785 50FFF>mov     word ptr [ebp-B0], 2C
004395DA  |.  8D45 EC       lea     eax, dword ptr [ebp-14]
004395DD  |.  E8 4282FCFF   call    00401824
004395E2  |.  8BD0          mov     edx, eax
004395E4  |.  FF85 5CFFFFFF inc     dword ptr [ebp-A4]
004395EA  |.  8B8D 3CFFFFFF mov     ecx, dword ptr [ebp-C4]
```



```
004395F0  |. 8B81 F0020000 mov     eax, dword ptr [ecx+2F0]
004395F6  |. E8 CD250700   call    004ABBC8
004395FB  |. 8D45 EC       lea     eax, dword ptr [ebp-14]
004395FE  |. E8 49B9FCFF   call    00404F4C                              ; 取机器码
00439603  |. 57            push    edi
00439604  |. 8BF8          mov     edi, eax
00439606  |. 33C0          xor     eax, eax
00439608  |. 83C9 FF       or      ecx, FFFFFFFF
0043960B  |. F2:AE         repne   scas byte ptr es:[edi]
0043960D  |. F7D1          not     ecx
0043960F  |. 2BF9          sub     edi, ecx
00439611  |. 8DB5 FCFEFFFF lea     esi, dword ptr [ebp-104]
00439617  |. 87F7          xchg    edi, esi
00439619  |. 8BD1          mov     edx, ecx
0043961B  |. 8BC7          mov     eax, edi
0043961D  |. C1E9 02       shr     ecx, 2
00439620  |. 8D45 EC       lea     eax, dword ptr [ebp-14]
00439623  |. F3:A5         rep     movs dword ptr es:[edi], dword p>
00439625  |. 8BCA          mov     ecx, edx
00439627  |. BA 02000000   mov     edx, 2
0043962C  |. 83E1 03       and     ecx, 3
0043962F  |. F3:A4         rep     movs byte ptr es:[edi], byte ptr>
00439631  |. 5F            pop     edi
00439632  |. FF8D 5CFFFFFF dec     dword ptr [ebp-A4]
00439638  |. E8 27100B00   call    004EA664
0043963D  |. 8D85 FCFEFFFF lea     eax, dword ptr [ebp-104]
00439643  |. 8D95 E4FEFFFF lea     edx, dword ptr [ebp-11C]
00439649  |. 8985 2CFFFFFF mov     dword ptr [ebp-D4], eax
0043964F  |. 8995 30FFFFFF mov     dword ptr [ebp-D0], edx
00439655  |. 33DB          xor     ebx, ebx                              ; 计数器清零
00439657  |. 8DB5 14FFFFFF lea     esi, dword ptr [ebp-EC]
0043965D  |> 8B8D 30FFFFFF /mov     ecx, dword ptr [ebp-D0]
00439663  |. 8B95 2CFFFFFF |mov     edx, dword ptr [ebp-D4]
00439669  |. 8A01          |mov     al, byte ptr [ecx]                  ; 取用户名左第一个字符
0043966B  |. 3202          |xor     al, byte ptr [edx]                  ; 与机器码左第一个字符异或记 a
0043966D  |. 83C4 F8       |add     esp, -8
00439670  |. 8806          |mov     byte ptr [esi], al                    ; |
00439672  |. 0FBE0E        |movsx   ecx, byte ptr [esi]                   ; |
00439675  |. 898D C8FEFFFF |mov     dword ptr [ebp-138], ecx              ; |
0043967B  |. DB85 C8FEFFFF |fild    dword ptr [ebp-138]                   ; |放入浮点寄存器
00439681  |. DD1C24        |fstp    qword ptr [esp]                       ; |
00439684  |. E8 33970A00   |call    004E2DBC                              ; \\屏录专家.004E2DBC
00439689  |. 83C4 08       |add     esp, 8
0043968C  |. 899D C4FEFFFF |mov     dword ptr [ebp-13C], ebx
00439692  |. DB85 C4FEFFFF |fild    dword ptr [ebp-13C]                   ; 取计数器的值入浮点寄存器记 b
00439698  |. DEC9          |fmulp   st(1), st                             ; a*b
0043969A  |. 89BD C0FEFFFF |mov     dword ptr [ebp-140], edi
004396A0  |. DB85 C0FEFFFF |fild    dword ptr [ebp-140]
004396A6  |. DEC1          |faddp   st(1), st                             ; 再加 edi
004396A8  |. E8 37970A00   |call    004E2DE4                              ; 值放入 eax
004396AD  |. 8BF8          |mov     edi, eax                              ; 值放入 edi，用以累加求和
004396AF  |. 43            |inc     ebx                                   ; 计数器+1
004396B0  |. 46            |inc     esi
004396B1  |. FF85 2CFFFFFF |inc     dword ptr [ebp-D4]                    ; 指针移动到机器码下一位
004396B7  |. FF85 30FFFFFF |inc     dword ptr [ebp-D0]                    ; 指针移动到用户名下一位，用户名末尾用空字符填充
004396BD  |. 83FB 14       |cmp     ebx, 14                               ; 遍历完 20 位机器码
004396C0  |.^ 7C 9B        \jl     short 0043965D
004396C2  |. 81C7 39300000 add     edi, 3039                               ; 循环算出的值再加 3039h
004396C8  |. 8D95 14FFFFFF lea     edx, dword ptr [ebp-EC]
004396CE  |. 57            push    edi                                     ; /Arg3
004396CF  |. 68 89035100   push    00510389                                ; |Arg2 = 00510389 ASCII "%d"
004396D4  |. 52            push    edx                                     ; |Arg1
004396D5  |. E8 2E7C0A00   call    004E1308                                ; \\把它的十进制数转换成字符串，得到 x
004396DA  |. 83C4 0C       add     esp, 0C
004396DD  |. 66:C785 50FFF>mov     word ptr [ebp-B0], 38
004396E6  |. 8D45 E8       lea     eax, dword ptr [ebp-18]
004396E9  |. E8 3681FCFF   call    00401824
004396EE  |. 8BD0          mov     edx, eax
004396F0  |. FF85 5CFFFFFF inc     dword ptr [ebp-A4]
004396F6  |. 8B8D 3CFFFFFF mov     ecx, dword ptr [ebp-C4]
004396FC  |. 8B81 E4020000 mov     eax, dword ptr [ecx+2E4]
00439702  |. E8 C1240700   call    004ABBC8
00439707  |. 8D45 E8       lea     eax, dword ptr [ebp-18]
0043970A  |. E8 3DB8FCFF   call    00404F4C                                ; 取输入的注册码
0043970F  |. 57            push    edi
00439710  |. 8BF8          mov     edi, eax
00439712  |. 33C0          xor     eax, eax
00439714  |. 83C9 FF       or      ecx, FFFFFFFF
00439717  |. F2:AE         repne   scas byte ptr es:[edi]
00439719  |. F7D1          not     ecx
0043971B  |. 2BF9          sub     edi, ecx
0043971D  |. 8DB5 CCFEFFFF lea     esi, dword ptr [ebp-134]
00439723  |. 87F7          xchg    edi, esi
00439725  |. 8BD1          mov     edx, ecx
00439727  |. 8BC7          mov     eax, edi
00439729  |. C1E9 02       shr     ecx, 2
0043972C  |. 8D45 E8       lea     eax, dword ptr [ebp-18]
0043972F  |. F3:A5         rep     movs dword ptr es:[edi], dword p>
00439731  |. 8BCA          mov     ecx, edx
00439733  |. BA 02000000   mov     edx, 2
00439738  |. 83E1 03       and     ecx, 3
0043973B  |. F3:A4         rep     movs byte ptr es:[edi], byte ptr>
0043973D  |. 5F            pop     edi
0043973E  |. FF8D 5CFFFFFF dec     dword ptr [ebp-A4]
00439744  |. E8 1B0F0B00   call    004EA664
00439749  |. 33DB          xor     ebx, ebx                              ; 计数器清零
0043974B  |. 8D85 CCFEFFFF lea     eax, dword ptr [ebp-134]
00439751  |. 8985 2CFFFFFF mov     dword ptr [ebp-D4], eax
00439757  |. 8DB5 14FFFFFF lea     esi, dword ptr [ebp-EC]                 ; ////////////////////////////*
0043975D  |> 8B95 2CFFFFFF /mov     edx, dword ptr [ebp-D4]                 ; //此循环将输入的注册码前五个字符每个-14h 然后与 x 比较
00439763  |. 0FBE06        |movsx   eax, byte ptr [esi]
00439766  |. 0FBE0A        |movsx   ecx, byte ptr [edx]                   ; 取输入的注册码的左第一个字符
00439769  |. 83C1 EC       |add     ecx, -14                              ; -20
0043976C  |. 3BC1          |cmp     eax, ecx                              ; 判断与 x 的相应位是否相同
0043976E  |. 0F85 80000000 |jnz     004397F4                              ; 不同则失败
00439774  |. 83FB 03       |cmp     ebx, 3                                ; 计数器比较，判断是否到注册码的第四位
00439777  |. 75 6A         |jnz     short 004397E3                        ; 不是则跳
00439779  |. 81C7 444D0000 |add     edi, 4D44                             ; x+=4d44h
0043977F  |. 89BD C8FEFFFF |mov     dword ptr [ebp-138], edi
00439785  |. DB85 C8FEFFFF |fild    dword ptr [ebp-138]                   ; 放入浮点寄存器
0043978B  |. DC0D C4A14300 |fmul    qword ptr [43A1C4]                   ; *3.14
00439791  |. DB2D CCA14300 |fld     tbyte ptr [43A1CC]
00439797  |. DEC9          |fmulp   st(1), st                             ; *0.1594896331738437120
00439799  |. E8 46960A00   |call    004E2DE4                              ; 整数部分放入 eax
0043979E  |. 8BF8          |mov     edi, eax
004397A0  |. 8BC7          |mov     eax, edi
004397A2  |. B9 A0860100   |mov     ecx, 186A0
```



```
004397A7  |.  99          |cdq
004397A8  |.  F7F9        |idiv    ecx                     ;  除以 186A0h
004397AA  |.  8BFA        |mov     edi, edx                ;  取余数，记 d
004397AC  |.  33C0        |xor     eax, eax
004397AE  |.  8985 38FFFFFF |mov     dword ptr [ebp-C8], eax         ;  累加器清零
004397B4  |.  33D2        |xor     edx, edx
004397B6  |.  8D85 CCFEFFFF |lea     eax, dword ptr [ebp-134]
004397BC  |>  0FBE08      |/movsx   ecx, byte ptr [eax]     ;  注册码的左一个字符
004397BF  |.  018D 38FFFFFF ||add      dword ptr [ebp-C8], ecx        ;  累加
004397C5  |.  42          ||inc      edx
004397C6  |.  40          ||inc      eax                     ;  注册码指针顺移
004397C7  |.  83FA 13     ||cmp      edx, 13                 ;  循环 20 次
004397CA  |.^ 7C F0       |\jl       short 004397BC
004397CC  |.  8B85 38FFFFFF |mov     eax, dword ptr [ebp-C8]
004397D2  |.  B9 0A000000  |mov     ecx, 0A
004397D7  |.  99          |cdq
004397D8  |.  F7F9        |idiv    ecx                     ;  累加器的值/10
004397DA  |.  83C2 30     |add     edx, 30                 ;  余数加 30h，记 c
004397DD  |.  8995 38FFFFFF |mov     dword ptr [ebp-C8], edx
004397E3  |>  43          |inc     ebx                     ;  计数器+1
004397E4  |.  FF85 2CFFFFFF |inc     dword ptr [ebp-D4]          ;  注册码指针指向下一位
004397EA  |.  46          |inc     esi                     ;  真注册码指针指向下一位
004397EB  |.  83FB 05     |cmp     ebx, 5
004397EE  |.^ 0F8C 69FFFFFF \jl      0043975D                ;  *////////////////////////////
004397F4  |>  83FB 05     cmp     ebx, 5
004397F7  |.  0F8C 68090000 jl      0043A165
004397FD  |.  0FBE85 DFFEFF>movsx   eax, byte ptr [ebp-121]
00439804  |.  3B85 38FFFFFF cmp     eax, dword ptr [ebp-C8]
0043980A  |.  74 09       je      short 00439815           ;  c 等于注册码末位的 ascii 则进入下一步
0043980C  |.  83F8 41     cmp     eax, 41
0043980F  |.  0F8C 50090000 jl      0043A165                ;  或者也可以是 c 大于 41h
00439815  |>  8BC7        mov     eax, edi                  ;  取 d
00439817  |.  B9 0A000000  mov     ecx, 0A
0043981C  |.  99          cdq
0043981D  |.  F7F9        idiv    ecx                       ;  d/10，得余数
0043981F  |.  0FBE841D CCFE>movsx   eax, byte ptr [ebp+ebx-134]     ;  取注册码第 6 位
00439827  |.  83C0 BF     add     eax, -41                  ;  -41h
0043982A  |.  2BC2        sub     eax, edx                  ;  -刚才的余数
0043982C  |.  85C0        test    eax, eax
0043982E  |.  74 09       je      short 00439839           ;  等于 0 则成功
00439830  |.  83F8 07     cmp     eax, 7
00439833  |.  0F85 E1080000 jnz     0043A11A                ;  或者也可以等于 7
```

### 算法总结

```
sum=0;

for(int i=0;i<20;i++){
    sum+=i*(用户名[i] xor 机器码[i])
}

sum=ascii(decimal(sum+0x3039));

a=个位数字(取余((int((sum+0x4D44)*3.14*0.1594896331738437120))/0x186A0))

b=注册码每个字符 ascii 累加值的个位数字+0x30
```

### 注册成功要同时满足的条件

1. `sum[n]` <==> `注册码[n]-0x14` `n` 前五位要相同
2. `b` 等于注册码末位字符的 `ascii` 或者 `b` 大于 `0x41`
3. 注册码第 6 位 - `41h` - `a` 等于 `0` 或者 `7`

### 机器码与用户名

- 机器码: `60279011771522222222`
- 用户名: `the0crat`
- 注册码: `FFHLMI234567890abcde`

`4dd466d0e1e0cc123afbe504da2462ca`
