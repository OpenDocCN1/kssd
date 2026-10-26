
# [原创]屏幕录像专家V6.0 Build20070123 算法分析

【文章标题】: 屏幕录像专家V6.0 Build20070123 算法分析

【文章作者】: the0crat

【作者邮箱】: the0crat.cn_at_gmail.com

【作者主页】: fa9K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4c8Z5k6e0m8U0M7X3q4@1i4K6u0W2j5X3I4G2k6$3y4F1i4K6u0W2j5$3!0E0

【生产日期】: 20070205

【软件名称】: 屏幕录像专家V6.0 Build20070123

【保护方式】: 注册码

【编写语言】: Borland C++ 1999

【使用工具】: `OD`

【作者声明】: 本文仅供研究学习，本人对因这篇文章而导致的一切后果，不承担任何法律责任。本文中的不足之处请各位多多指教

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

--------------------------------------------------------------------------------

## 【详细过程】

无壳，常规的断点一路追到这里，即验证过程

```
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
004394CA  |.  FF8D 5CFFFFFF dec     dword ptr [ebp-A4]               ; |
004394D0  |.  8D45 FC       lea     eax, dword ptr [ebp-4]           ; |
004394D3  |.  BA 02000000   mov     edx, 2                           ; |
004394D8  |.  E8 87110B00   call    004EA664                         ; \屏录专家.004EA664
004394DD  |.  59            pop     ecx
004394DE  |.  84C9          test    cl, cl
004394E0  |.  74 48         je      short 0043952A                   ;  判断注册码是不是现有版本的注册码
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
0043952A  |>  6A 14         push    14                               ; /Arg3 = 00000014
0043952C  |.  6A 00         push    0                                ; |Arg2 = 00000000
0043952E  |.  8D85 FCFEFFFF lea     eax, dword ptr [ebp-104]         ; |
00439534  |.  50            push    eax                              ; |Arg1
00439535  |.  E8 6E520A00   call    004DE7A8                         ; \屏录专家.004DE7A8
0043953A  |.  83C4 0C       add     esp, 0C
0043953D  |.  33FF          xor     edi, edi
0043953F  |.  6A 14         push    14                               ; /Arg3 = 00000014
00439541  |.  6A 00         push    0                                ; |Arg2 = 00000000
00439543  |.  8D85 14FFFFFF lea     eax, dword ptr [ebp-EC]          ; |
00439549  |.  50            push    eax                              ; |Arg1
0043954A  |.  E8 59520A00   call    004DE7A8                         ; \屏录专家.004DE7A8
0043954F  |.  83C4 0C       add     esp, 0C
00439552  |.  8D95 E4FEFFFF lea     edx, dword ptr [ebp-11C]
00439558  |.  6A 14         push    14                               ; /Arg3 = 00000014
0043955A  |.  6A 00         push    0                                ; |Arg2 = 00000000
0043955C  |.  52            push    edx                              ; |Arg1
0043955D  |.  E8 46520A00   call    004DE7A8                         ; \屏录专家.004DE7A8
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
00439592  |.  E8 B5B9FCFF   call    00404F4C                         ;  取用户名
00439597  |.  57            push    edi
00439598  |.  8BF8          mov     edi, eax
0043959A  |.  33C0          xor     eax, eax
0043959C  |.  83C9 FF       or      ecx, FFFFFFFF
0043959F  |.  F2:AE         repne   scas byte ptr es:[edi]
004395A1  |.  F7D1          not     ecx
004395A3  |.  2BF9          sub     edi, ecx
004395A5  |.  8DB5 E4FEFFFF lea     esi, dword ptr [ebp-11C]
004395AB  |.  87F7          xchg    edi, esi
004395AD  |.  8BD1          mov     edx, ecx
004395AF  |.  8BC7          mov     eax, edi
004395B1  |.  C1E9 02       shr     ecx, 2
004395B4  |.  8D45 F0       lea     eax, dword ptr [ebp-10]
004395B7  |.  F3:A5         rep     movs dword ptr es:[edi], dword p>
004395B9  |.  8BCA          mov     ecx, edx
004395BB  |.  BA 02000000   mov     edx, 2
004395C0  |.  83E1 03       and     ecx, 3
004395C3  |.  F3:A4         rep     movs byte ptr es:[edi], byte ptr>
004395C5  |.  5F            pop     edi
004395C6  |.  FF8D 5CFFFFFF dec     dword ptr [ebp-A4]
004395CC  |.  E8 93100B00   call    004EA664
004395D1  |.  66:C785 50FFF>mov     word ptr [ebp-B0], 2C
004395DA  |.  8D45 EC       lea     eax, dword ptr [ebp-14]
004395DD  |.  E8 4282FCFF   call    00401824
004395E2  |.  8BD0          mov     edx, eax
004395E4  |.  FF85 5CFFFFFF inc     dword ptr [ebp-A4]
004395EA  |.  8B8D 3CFFFFFF mov     ecx, dword ptr [ebp-C4]
004395F0  |.  8B81 F0020000 mov     eax, dword ptr [ecx+2F0]
004395F6  |.  E8 CD250700   call    004ABBC8
004395FB  |.  8D45 EC       lea     eax, dword ptr [ebp-14]
004395FE  |.  E8 49B9FCFF   call    00404F4C                         ;  取机器码
00439603  |.  57            push    edi
00439604  |.  8BF8          mov     edi, eax
00439606  |.  33C0          xor     eax, eax
00439608  |.  83C9 FF       or      ecx, FFFFFFFF
0043960B  |.  F2:AE         repne   scas byte ptr es:[edi]
0043960D  |.  F7D1          not     ecx
0043960F  |.  2BF9          sub     edi, ecx
00439611  |.  8DB5 FCFEFFFF lea     esi, dword ptr [ebp-104]
00439617  |.  87F7          xchg    edi, esi
```
