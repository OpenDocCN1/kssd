
```
0042B227   |.  0FBE440D 8F     |movsx eax,byte ptr ss:[ebp+ecx-71]           ;  取假码第2位
0042B22C   |.  03D0            |add edx,eax
0042B22E   |.  8B4D C4         |mov ecx,[local.15]
0042B231   |.  0FBE440D 90     |movsx eax,byte ptr ss:[ebp+ecx-70]           ;  取假码第3位
0042B236   |.  33D0            |xor edx,eax
0042B238   |.  8B4D C4         |mov ecx,[local.15]
0042B23B   |.  0FBE840D 78FFFF>|movsx eax,byte ptr ss:[ebp+ecx-88]           ;  取常串第3位
0042B243   |.  33D0            |xor edx,eax
0042B245   |.  52              |push edx                                     ; /Arg1
0042B246   |.  E8 25E0FFFF     |call x.00429270                              ; \x.00429270
0042B24B   |.  59              |pop ecx
0042B24C   |.  B9 1A000000     |mov ecx,1A
0042B251   |.  99              |cdq
0042B252   |.  F7F9            |idiv ecx
0042B254   |.  83C2 41         |add edx,41                                   ;  运算结果
0042B257   |.  8B45 C4         |mov eax,[local.15]
0042B25A   |.  0FBE4C05 99     |movsx ecx,byte ptr ss:[ebp+eax-67]           ;  取假码第12位
0042B25F   |.  3BD1            |cmp edx,ecx                                  ;  比较
0042B261   |.  74 06           |je short x.0042B269                          ;  必须跳
0042B263   |.  C645 D7 00      |mov byte ptr ss:[ebp-29],0
0042B267   |.  EB 09           |jmp short x.0042B272
0042B269   |>  FF45 C4         |inc [local.15]                               ;  计数器
0042B26C   |.  837D C4 0A      |cmp [local.15],0A                            ;  循环8次
0042B270   |.^ 7C A7           \jl short x.0042B219
0042B272   |>  807D D7 00      cmp byte ptr ss:[ebp-29],0
0042B276   |.  0F84 D2000000   je x.0042B34E
0042B27C   |.  C745 C0 1800000>mov [local.16],18
0042B283   |.  66:C745 E8 0800 mov word ptr ss:[ebp-18],8
0042B289   |.  837D C0 28      cmp [local.16],28
0042B28D   |.  7D 54           jge short x.0042B2E3
0042B28F   |>  8B55 C0         /mov edx,[local.16]
0042B292   |.  0FBE8415 79FFFF>|movsx eax,byte ptr ss:[ebp+edx-87]           ;  取假码第2位
0042B29A   |.  B9 06000000     |mov ecx,6
0042B29F   |.  99              |cdq
0042B2A0   |.  F7F9            |idiv ecx
0042B2A2   |.  8BCA            |mov ecx,edx
0042B2A4   |.  8B45 C0         |mov eax,[local.16]
0042B2A7   |.  0FBE9405 7AFFFF>|movsx edx,byte ptr ss:[ebp+eax-86]           ;  取假码第3位
0042B2AF   |.  D3E2            |shl edx,cl
0042B2B1   |.  8B45 C0         |mov eax,[local.16]
0042B2B4   |.  0FBE8C05 7BFFFF>|movsx ecx,byte ptr ss:[ebp+eax-85]           ;  取假码第4位
0042B2BC   |.  0BD1            |or edx,ecx
0042B2BE   |.  52              |push edx                                     ; /Arg1
0042B2BF   |.  E8 ACDFFFFF     |call x.00429270                              ; \x.00429270
0042B2C4   |.  59              |pop ecx
0042B2C5   |.  B9 1A000000     |mov ecx,1A
0042B2CA   |.  99              |cdq
0042B2CB   |.  F7F9            |idiv ecx
0042B2CD   |.  80C2 61         |add dl,61                                    ;  运算结果
0042B2D0   |.  8B45 C0         |mov eax,[local.16]
0042B2D3   |.  889405 4CFFFFFF |mov byte ptr ss:[ebp+eax-B4],dl              ;  生成新的字符串
0042B2DA   |.  FF45 C0         |inc [local.16]                               ;  计数器
0042B2DD   |.  837D C0 28      |cmp [local.16],28                            ;  循环16次
0042B2E1   |.^ 7C AC           \jl short x.0042B28F
0042B2E3   |>  C685 74FFFFFF 5>mov byte ptr ss:[ebp-8C],5A
0042B2EA   |.  C685 75FFFFFF 5>mov byte ptr ss:[ebp-8B],59
0042B2F1   |.  C745 BC 1800000>mov [local.17],18
0042B2F8   |.  66:C745 E8 0800 mov word ptr ss:[ebp-18],8
0042B2FE   |.  837D BC 28      cmp [local.17],28
0042B302   |.  7D 4A           jge short x.0042B34E
0042B304   |>  8B55 BC         /mov edx,[local.17]
0042B307   |.  0FBE8415 4CFFFF>|movsx eax,byte ptr ss:[ebp+edx-B4]           ;  新字符串第1位
0042B30F   |.  C1E0 04         |shl eax,4
0042B312   |.  8B55 BC         |mov edx,[local.17]
0042B315   |.  0FBE8C15 4DFFFF>|movsx ecx,byte ptr ss:[ebp+edx-B3]           ;  新字符串第2位
0042B31D   |.  D1F9            |sar ecx,1
0042B31F   |.  33C1            |xor eax,ecx
0042B321   |.  50              |push eax                                     ; /Arg1
0042B322   |.  E8 49DFFFFF     |call x.00429270                              ; \x.00429270
0042B327   |.  59              |pop ecx
0042B328   |.  B9 1A000000     |mov ecx,1A
0042B32D   |.  99              |cdq
0042B32E   |.  F7F9            |idiv ecx
0042B330   |.  83C2 41         |add edx,41                                   ;  运算结果
0042B333   |.  8B45 BC         |mov eax,[local.17]
0042B336   |.  0FBE4405 90     |movsx eax,byte ptr ss:[ebp+eax-70]           ;  取假码第25位
0042B33B   |.  3BD0            |cmp edx,eax                                  ;  比较
0042B33D   |.  74 06           |je short x.0042B345                          ;  必须跳
0042B33F   |.  C645 D7 00      |mov byte ptr ss:[ebp-29],0
0042B343   |.  EB 09           |jmp short x.0042B34E
0042B345   |>  FF45 BC         |inc [local.17]                               ;  计数器
0042B348   |.  837D BC 28      |cmp [local.17],28                            ;  循环16次
0042B34C   |.^ 7C B6           \jl short x.0042B304
0042B34E   |>  0FBE55 9A       movsx edx,byte ptr ss:[ebp-66]                ;  取假码第11位
0042B352   |.  83FA 59         cmp edx,59                                    ;  与‘Y’比较
0042B355   |.  74 04           je short x.0042B35B                           ;  跳
0042B357   |.  C645 D7 00      mov byte ptr ss:[ebp-29],0
0042B35B   |>  8A45 D7         mov al,byte ptr ss:[ebp-29]                   ;  验证标志位
0042B35E   |.  50              push eax
0042B35F   |.  FF4D F4         dec [local.3]
0042B362   |.  8D45 08         lea eax,[arg.1]
0042B365   |.  BA 02000000     mov edx,2
0042B36A   |.  E8 A58B0D00     call x.00503F14
0042B36F   |.  58              pop eax
0042B370   |.  8B55 D8         mov edx,[local.10]
0042B373   |.  64:8915 0000000>mov dword ptr fs:[0],edx
0042B37A   |>  5F              pop edi
0042B37B   |.  5E              pop esi
0042B37C   |.  8BE5            mov esp,ebp
0042B37E   |.  5D              pop ebp
0042B37F   \.  C3              retn
```

## 汇编注册机算法部分源码

```
                .data

szStr                db        '1z1h+2a0n-0g8y\*9a1n|',0

szData                db        'ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789',0

                .code

GetRegKey        proc        hDlg:DWORD

                local        szReg[64]:BYTE,szTmp[32]:BYTE

                pushad

                invoke  RtlZeroMemory,addr szReg,sizeof szReg

                invoke  RtlZeroMemory,addr szTmp,sizeof szTmp

                invoke        GetTickCount

                lea        esi,szData

                lea        edi,szReg

                mov        ecx,32h

        @@:

                and        eax,1fh

                mov        dl,BYTE ptr [esi+eax]

                mov        BYTE ptr [edi],dl

                add        eax,3

                inc        edi

                dec        ecx

                jnz        @b

                lea        edi,szReg

                mov        BYTE ptr [edi+1],2dh

                xor        ebx,ebx

        @@:

                movsx        eax,BYTE ptr [edi+ebx]

                mov        ecx,0ah

                cdq

                idiv        ecx

                add        edx,30h

                mov        BYTE ptr [edi+2ch+ebx],dl

                inc        ebx

                cmp        ebx,6

                jl        @b

                mov        BYTE ptr [edi+28h],42h

                mov        BYTE ptr [edi+29h],53h

                mov        BYTE ptr [edi+2ah],4dh

                mov        BYTE ptr [edi+2bh],31h

                mov        BYTE ptr [edi+1],23h

                mov        BYTE ptr [edi+0ah],59h

                mov        BYTE ptr [edi+17h],37h

                lea        esi,szStr

                mov        ebx,2

        @@:

                movsx        edx,BYTE ptr [esi+ebx]
```
