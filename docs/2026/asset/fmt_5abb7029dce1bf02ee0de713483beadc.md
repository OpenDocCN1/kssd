
# [原创]delphi控件ImageEn2.26算法分析

**【文章标题】**: delphi控件ImageEn2.26算法分析

**【文章作者】**: 8713007

**【软件名称】**: ImageEn2.26

**【软件大小】**: 12M

**【下载地址】**: 自己搜索下载

**【加壳方式】**: 无壳

**【保护方式】**: 序列号

**【编写语言】**: delphi

**【使用工具】**: W32dsm8.93+,OD

**【操作平台】**: winxp

**【软件介绍】**: 一款非常优秀的图像控件，能实现多种图像各式的操作

**【作者声明】**: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

--------------------------------------------------------------------------------

## 【详细过程】

ImageEn2.26是一款非常优秀的软件，安装后运行安装目录的`setup.exe`，勾选I have the serial Number,填入用户名`SDlingying`，序列号`12345678901234`，点击Next出现Invalid Serial Number,使用`W32dsm8.93+`反汇编，查找字符串，发现在`00447B09`处，利用`OD`载入，在其上的`00447A7B`下断，F9运行，勾选I have the serial Number,填入用户名`SDlingying`，序列号`12345678901234`，点击Next，程序被断下

```
00447A7B   .  8B45 FC       mov     eax, [ebp-4]             ;  段在这里
00447A7E   .  8B80 E4020000 mov     eax, [eax+2E4]
00447A84   .  8B10          mov     edx, [eax]
00447A86   .  FF92 B4000000 call    [edx+B4]                 ;  setup.00420F5C
00447A8C   .  84C0          test    al, al
00447A8E   .  0F84 84000000 je      00447B18
00447A94   .  8D55 E0       lea     edx, [ebp-20]
00447A97   .  8B45 FC       mov     eax, [ebp-4]
00447A9A   .  8B80 DC020000 mov     eax, [eax+2DC]
00447AA0   .  E8 C3E2FDFF   call    00425D68                 ;  取得用户名
00447AA5   .  837D E0 00    cmp     dword ptr [ebp-20], 0    ;  比较是否位空
00447AA9   .  74 6D         je      short 00447B18
00447AAB   .  6A 12         push    12                       ;  12压栈
00447AAD   .  68 73010000   push    173                      ;  173压栈
00447AB2   .  8D55 DC       lea     edx, [ebp-24]
00447AB5   .  8B45 FC       mov     eax, [ebp-4]
00447AB8   .  8B80 DC020000 mov     eax, [eax+2DC]
00447ABE   .  E8 A5E2FDFF   call    00425D68                 ;  取得用户名字长度入eax
00447AC3   .  8B45 DC       mov     eax, [ebp-24]            ; |
00447AC6   .  8D55 E5       lea     edx, [ebp-1B]            ; |[ebp-1b]入edx此处为0013f40d
00447AC9   .  B9 12000000   mov     ecx, 12                  ; |12入Ecx
00447ACE   .  E8 89040000   call    00447F5C                 ; \关键call，F7跟进
00447AD3   .  C645 F7 00    mov     byte ptr [ebp-9], 0
00447AD7   .  8D55 D8       lea     edx, [ebp-28]
00447ADA   .  8B45 FC       mov     eax, [ebp-4]
00447ADD   .  8B80 E0020000 mov     eax, [eax+2E0]
00447AE3   .  E8 80E2FDFF   call    00425D68
00447AE8   .  8B45 D8       mov     eax, [ebp-28]
00447AEB   .  E8 64C3FBFF   call    00403E54
00447AF0   .  8BD0          mov     edx, eax
00447AF2   .  8D45 E5       lea     eax, [ebp-1B]
00447AF5   .  E8 220AFCFF   call    0040851C
00447AFA   .  85C0          test    eax, eax
00447AFC   .  74 1A         je      short 00447B18           ;  爆破改这里
00447AFE   .  6A 00         push    0                        ; /Arg1 = 00000000
00447B00   .  66:8B0D A87C4>mov     cx, [447CA8]             ; |
00447B07   .  33D2          xor     edx, edx                 ; |
00447B09   .  B8 B47C4400   mov     eax, 00447CB4            ; |ASCII "Invalid serial number"
00447B0E   .  E8 C9E4FFFF   call    00445FDC                 ; \setup.00445FDC
00447B13   .  E9 52010000   jmp     00447C6A
```

///////////////////////////////////////////////////F7跟进后来到这里

```
00447F5C  /$  55            push    ebp
00447F5D  |.  8BEC          mov     ebp, esp
00447F5F  |.  83C4 F4       add     esp, -0C
00447F62  |.  53            push    ebx
00447F63  |.  56            push    esi
00447F64  |.  57            push    edi
00447F65  |.  8955 F8       mov     [ebp-8], edx             ;  edx值(0013f40d)入ebp-8
00447F68  |.  8945 FC       mov     [ebp-4], eax             ;  用户名入ebp-4
00447F6B  |.  8B7D 08       mov     edi, [ebp+8]             ;  173入edi
00447F6E  |.  8B5D 0C       mov     ebx, [ebp+C]             ;  12入ebx
00447F71  |.  4B            dec     ebx                      ;  ebx-1
00447F72  |.  85DB          test    ebx, ebx
00447F74  |.  7C 38         jl      short 00447FAE           ;  小于0跳走
00447F76  |.  43            inc     ebx                      ;  ebx+1
00447F77  |.  C745 F4 00000>mov     dword ptr [ebp-C], 0
00447F7E  |.  BE D8AA4400   mov     esi, 0044AAD8            ;  0044AAD8入esi
00447F83  |>  8B45 FC       /mov     eax, [ebp-4]            ;  用户名入eax
00447F86  |.  E8 05BDFBFF   |call    00403C90                ;  用户名长度入eax
00447F8B  |.  50            |push    eax
00447F8C  |.  8B45 F4       |mov     eax, [ebp-C]            ;  ebp-c入eax(首次为0)
00447F8F  |.  5A            |pop     edx
00447F90  |.  8BCA          |mov     ecx, edx                ;  edx入ecx
00447F92  |.  99            |cdq                             ;  edx清0
00447F93  |.  F7F9          |idiv    ecx                     ;  eax除以ecx,商入eax，余数入edx
00447F95  |.  42            |inc     edx                     ;  edx+1
00447F96  |.  8B45 FC       |mov     eax, [ebp-4]            ;  用户名入eax
00447F99      8A4410 FF     mov     al, [eax+edx-1]          ;  取用户名第edx出的字符
00447F9D  |.  8B16          |mov     edx, [esi]              ;  取[esi]处的值入edx
00447F9F  |.  8B4D F8       |mov     ecx, [ebp-8]            ;  13f40d入ecx
00447FA2  |.  880411        |mov     [ecx+edx], al           ;  将字符放在13f40d+edx处
00447FA5  |.  FF45 F4       |inc     dword ptr [ebp-C]       ;  ebp-c处的值+1
00447FA8  |.  83C6 04       |add     esi, 4                  ;  esi+4
00447FAB  |.  4B            |dec     ebx                     ;  ebx-1
00447FAC  |.^ 75 D5         \jnz     short 00447F83          ;  循环
```

程序以上算法是将用户名前18位取出按照不同的位次重新组合，生成一个新的字符串。用户不足18位从头再取
生成字符串放在`ebp-c`处，我们设这个字符串为`X`,循环完成后来到这里

```
00447FAE  |> \8B5D>mov     ebx, [ebp+C]           ;  12入ebx
00447FB1 >|.  4B   dec     ebx                    ;  ebx-1
00447FB2  |.  85DB test    ebx, ebx
00447FB4  |.  7C 2>jl      short 00447FDF         ;  小于0跳走
00447FB6  |.  43   inc     ebx                    ;  ebx+1
00447FB7  |.  8B45>mov     eax, [ebp-8]           ;  X入eax
00447FBA  |>  33D2 /xor     edx, edx              ;  edx清0
00447FBC  |.  8A10 |mov     dl, [eax]             ;  依次取X的每个字符入dl
00447FBE  |.  0FB7>|movzx   ecx, di               ;  di入ecx
00447FC1  |.  C1E9>|shr     ecx, 8                ;  ecx shr 8
00447FC4  |.  33D1 |xor     edx, ecx              ;  edx xor ecx
00447FC6  |.  8810 |mov     [eax], dl             ;  将得到的值代替原来的字符串
00447FC8  |.  33D2 |xor     edx, edx              ;  edx清0
00447FCA  |.  8A10 |mov     dl, [eax]             ;  将计算后的值入dl
00447FCC  |.  66:0>|add     di, dx                ;  di and dx
00447FCF  |.  66:6>|imul    dx, di, 0CE6D         ;  dx=di imul 0ce6d
00447FD4  |.  66:8>|add     dx, 58BF              ;  dx and 58bf
00447FD9  |.  8BFA |mov     edi, edx              ;  edx入edi
00447FDB  |.  40   |inc     eax                   ;  eax+1
00447FDC  |.  4B   |dec     ebx                   ;  ebx-1
00447FDD  |.^ 75 D>\jnz     short 00447FBA        ;  循环
```

///////////////////////////////////////以上程序将`X`进行一番运算，设`X`的值变为`Y`

```
00447FDF  |> \8B5D>mov     ebx, [ebp+C]           ;  12入ebx
00447FE2 >|.  4B   dec     ebx                    ;  ebx-1
00447FE3  |.  85DB test    ebx, ebx
00447FE5 >|.  7C 1>jl      short 00448004         ;  小于0跳走
00447FE7  |.  43   inc     ebx                    ;  ebx+1
00447FE8  |.  8B4D>mov     ecx, [ebp-8]           ;  Y入ecx
00447FEB  |>  33C0 /xor     eax, eax              ;  eax清0
00447FED  |.  8A01 |mov     al, [ecx]             ;  依次取Y的每位字符Ascii入al
00447FEF  |.  BE 3>|mov     esi, 31               ;  31入esi
00447FF4  |.  33D2 |xor     edx, edx
00447FF6 >|.  F7F6 |div     esi                   ;  eax 除以esi商入eax余数入edx
00447FF8  |.  8A82>|mov     al, [edx+44AAA4]      ;  44AAA4处是一个字符串表
00447FFE  |.  8801 |mov     [ecx], al             ;  取edx+44AAA4处的值代替Y原来的字符
00448000  |.  41   |inc     ecx                   ;  ecx+1
00448001  |.  4B   |dec     ebx                   ;  ebx-1
```
