
```
004C6BAE   .  8D4C24 18     lea     ecx, [esp+18]

004C6BB2   .  E8 69B9F3FF   call    00402520

004C6BB7   .  8D5C24 60     lea     ebx, [esp+60]

004C6BBB   .  8BF8          mov     edi, eax                         ;  用户名"snake"

004C6BBD   .  E8 5EE0FFFF   call    004C4C20                         ;  算法call，由用户名生成4位的字符串
```

```
004C4C20  /$  81EC 04010000 sub     esp, 104

004C4C26  |.  A1 00156900   mov     eax, [691500]

004C4C2B  |.  56            push    esi

004C4C2C  |.  8D7424 04     lea     esi, [esp+4]

004C4C30  |.  898424 040100>mov     [esp+104], eax

004C4C37  |.  E8 64FEFFFF   call    004C4AA0                         ;  生成长字符串

004C4C3C  |.  8A07          mov     al, [edi]

004C4C3E  |.  33C9          xor     ecx, ecx

004C4C40  |.  84C0          test    al, al

004C4C42  |.  B2 20         mov     dl, 20

004C4C44  |.  74 09         je      short 004C4C4F

004C4C46  |>  8A4439 01     /mov     al, [ecx+edi+1]                 ;  计算用户名长度

004C4C4A  |.  41            |inc     ecx

004C4C4B  |.  84C0          |test    al, al

004C4C4D  |.^ 75 F7         \jnz     short 004C4C46

004C4C4F  |>  33C0          xor     eax, eax

004C4C51  |.  85C9          test    ecx, ecx

004C4C53  |.  7E 08         jle     short 004C4C5D

004C4C55  |>  021438        /add     dl, [eax+edi]                   ;  分别取用户名的ASCII值进行运算

004C4C58  |.  40            |inc     eax

004C4C59  |.  3BC1          |cmp     eax, ecx

004C4C5B  |.^ 7C F8         \jl      short 004C4C55

004C4C5D  |>  0FB6C2        movzx   eax, dl                          ;  运算结果

004C4C60  |.  8A4404 04     mov     al, [esp+eax+4]                  ;  al='O'

004C4C64  |.  33F6          xor     esi, esi

004C4C66  |.  85C9          test    ecx, ecx

004C4C68  |.  8803          mov     [ebx], al

004C4C6A  |.  A2 1C466D00   mov     [6D461C], al

004C4C6F  |.  B2 40         mov     dl, 40

004C4C71  |.  7E 0C         jle     short 004C4C7F

004C4C73  |>  8A043E        /mov     al, [esi+edi]                   ;  分别取用户名的ASCII值进行运算

004C4C76  |.  F6EA          |imul    dl

004C4C78  |.  46            |inc     esi

004C4C79  |.  3BF1          |cmp     esi, ecx

004C4C7B  |.  8AD0          |mov     dl, al

004C4C7D  |.^ 7C F4         \jl      short 004C4C73

004C4C7F  |>  0FB6D2        movzx   edx, dl                          ;  运算结果

004C4C82  |.  8A4414 04     mov     al, [esp+edx+4]                  ;  al='U'

004C4C86  |.  8843 01       mov     [ebx+1], al

004C4C89  |.  A2 1D466D00   mov     [6D461D], al

004C4C8E  |.  33C0          xor     eax, eax

004C4C90  |.  85C9          test    ecx, ecx

004C4C92  |.  B2 80         mov     dl, 80

004C4C94  |.  5E            pop     esi

004C4C95  |.  7E 08         jle     short 004C4C9F

004C4C97  |>  2A1438        /sub     dl, [eax+edi]                   ;  分别取用户名的ASCII值进行运算

004C4C9A  |.  40            |inc     eax

004C4C9B  |.  3BC1          |cmp     eax, ecx

004C4C9D  |.^ 7C F8         \jl      short 004C4C97

004C4C9F  |>  8A0C0C        mov     cl, [esp+ecx]                    ;  cl='F'

004C4CA2  |.  0FB6C2        movzx   eax, dl

004C4CA5  |.  8A0404        mov     al, [esp+eax]                    ;  al='C'

004C4CA8  |.  8843 02       mov     [ebx+2], al

004C4CAB  |.  A2 1E466D00   mov     [6D461E], al

004C4CB0  |.  884B 03       mov     [ebx+3], cl                      ;  最终结果为

004C4CB3  |.  880D 1F466D00 mov     [6D461F], cl                     ;  006D461C  4F 55 43 46 串"OUCF"

004C4CB9  |.  8B8C24 000100>mov     ecx, [esp+100]

004C4CC0  |.  E8 49260A00   call    0056730E

004C4CC5  |.  81C4 04010000 add     esp, 104

004C4CCB  \.  C3            retn
```

```
004C6BC2   .  6A 04         push    4

004C6BC4   .  8D4424 30     lea     eax, [esp+30]

004C6BC8   .  50            push    eax

004C6BC9   .  8D4C24 28     lea     ecx, [esp+28]

004C6BCD   .  E8 1EBAF3FF   call    004025F0

004C6BD2   .  C64424 74 0F  mov     byte ptr [esp+74], 0F

004C6BD7   .  50            push    eax

004C6BD8   .  8D4C24 14     lea     ecx, [esp+14]

004C6BDC   .  E8 1FBBF3FF   call    00402700

004C6BE1   .  C64424 74 0E  mov     byte ptr [esp+74], 0E

004C6BE6   .  8D4C24 2C     lea     ecx, [esp+2C]

004C6BEA   .  E8 41B8F3FF   call    00402430

004C6BEF   .  8B4C24 10     mov     ecx, [esp+10]                    ;  取注册码前4位

004C6BF3   .  8B41 F4       mov     eax, [ecx-C]

004C6BF6   .  50            push    eax

004C6BF7   .  8D4C24 14     lea     ecx, [esp+14]

004C6BFB   .  C64424 68 00  mov     byte ptr [esp+68], 0

004C6C00   .  8BF3          mov     esi, ebx                         ;  取由用户名生成的字符串

004C6C02   .  E8 19B9F3FF   call    00402520

004C6C07   >  8A10          mov     dl, [eax]                        ;  逐位比较相等则验证通过

004C6C09   .  8A1E          mov     bl, [esi]

004C6C0B   .  8ACA          mov     cl, dl

004C6C0D   .  3AD3          cmp     dl, bl

004C6C0F   .  75 1E         jnz     short 004C6C2F                   ;  关键跳转 nop

004C6C11   .  84C9          test    cl, cl

004C6C13   .  74 16         je      short 004C6C2B

004C6C15   .  8A50 01       mov     dl, [eax+1]

004C6C18   .  8A5E 01       mov     bl, [esi+1]

004C6C1B   .  8ACA          mov     cl, dl

004C6C1D   .  3AD3          cmp     dl, bl

004C6C1F   .  75 0E         jnz     short 004C6C2F                   ;  关键跳转 nop

004C6C21   .  83C0 02       add     eax, 2

004C6C24   .  83C6 02       add     esi, 2

004C6C27   .  84C9          test    cl, cl

004C6C29   .^ 75 DC         jnz     short 004C6C07

004C6C2B   >  33C0          xor     eax, eax

004C6C2D   .  EB 04         jmp     short 004C6C33                   ;  中间4位注册码为任意值

004C6C2F   >  1BC0          sbb     eax, eax

004C6C31   .  1BC5          sbb     eax, ebp

004C6C33   >  85C0          test    eax, eax

004C6C35   .  0F85 CF000000 jnz     004C6D0A
```

......

## 【汇编注册机算法部分源码】

```
.data

szTable                db        "ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ01"

                db        "23456789ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789ABCDEFGHIJKLMNOPQRST"

                db        "UVWXYZ0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789ABCDEFGHIJKL"

                db        "MNOPQRSTUVWXYZ0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789ABCD",0

.data?

szName                dd        32 dup (?)

szSerial        db        16 dup (?)

szTmp                db        8  dup (?)

nCon                dd        ?

.code

GetRegKey        proc        hDlg

        pushad

        invoke        GetDlgItemText,hDlg,IDC_NAME,addr szName,sizeof szName

        .if        eax

                mov        nCon,eax

                lea        esi,szTable

                mov        dl,[esi+eax]

                mov        [szSerial+3],dl

                xor        edx,edx

                lea        edi,szName

        @@:

                add        dl,[edi+eax-1]

                dec        eax

                jnz        @b

                cmp        dl,80h

                ja        @1

                mov        ecx,80h

                sub        ecx,edx

                jmp        @2

        @1:

                mov        ecx,180h

                sub        ecx,edx

        @2:

                mov        dl,[esi+ecx]

                mov        [szSerial+2],dl

                mov        ecx,nCon

                xor        ebx,ebx

                mov        dl,40h

        @@:

                mov        al,[edi+ebx]

                imul        dl

                mov        dl,al

                inc        ebx

                dec        ecx

                jnz        @b

                mov        al,[esi+edx]

                mov        [szSerial+1],al

                xor        edx,edx

                mov        ecx,nCon

        @@:

                sub        dl,[edi+ecx-1]

                dec        ecx

                jnz        @b

                cmp        dl,20h

                ja        @3

                mov        ecx,20h

                sub        ecx,edx
```
