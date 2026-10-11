
```
:0040349A 83F85A                 cmp eax, 0000005A

:0040349D 7F03                   jg 004034A2

:0040349F 83C020                 add eax, 00000020
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  
|:00403498(C), :0040349D(C)  
|

```
:004034A2 0FBE540C0C             movsx edx, byte ptr [esp+ecx+0C]

:004034A7 3BD0                   cmp edx, eax
                                  ==== >逐位比较比较前 4 位！
                                  ==== >[esp+0C]=3611
                                  ==== >[esi]=1234
        ★★★★      一、           ==== >所以注册码的前 4 位是：3611

:004034A9 0F8519010000           jne 004035C8
                                  ==== >跳则 OVER！

:004034AF 41                     inc ecx
:004034B0 83F904                 cmp ecx, 00000004
:004034B3 7CDC                   jl 00403491
:004034B5 8B44240C               mov eax, dword ptr [esp+0C]
                                  ==== >EAX=31313633

:004034B9 33DB                   xor ebx, ebx
:004034BB 8D0440                 lea eax, dword ptr [eax+2*eax]
                                  ==== >EAX=31313633 * 3=9393A299

:004034BE C1E002                 shl eax, 02
                                  ==== >EAX=9393A299 SHL 02=4E4E8A64   得出运算 6-9 位注册码的参数！

:004034C1 89442410               mov dword ptr [esp+10], eax
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:004034E8(C)  
|

```
:004034C5 8D7C1C10               lea edi, dword ptr [esp+ebx+10]
:004034C9 8A07                   mov al, byte ptr [edi]
:004034CB 50                     push eax
:004034CC 53                     push ebx
:004034CD 43                     inc ebx
:004034CE E82D010000             call 00403600
                                  ==== >对 64 8A 4E 4E 进行运算得出一组新值 B2 45 13 09

:004034D3 83C408                 add esp, 00000008
:004034D6 33C9                   xor ecx, ecx
:004034D8 8AC8                   mov cl, al
:004034DA 51                     push ecx
:004034DB E810020000             call 004036F0
                                  ==== >对 B2 45 13 09 进行运算得出一组新值 32 65 33 37

:004034E0 83C404                 add esp, 00000004
:004034E3 8807                   mov byte ptr [edi], al
                                  ==== >[edi]=2e37        保存结果

:004034E5 83FB04                 cmp ebx, 00000004
:004034E8 7CDB                   jl 004034C5
                                  ==== >循环 4 次

:004034EA 33C9                   xor ecx, ecx
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:0040350F(C)  
|

```
:004034EC 0FBE440E05             movsx eax, byte ptr [esi+ecx+05]
:004034F1 83F841                 cmp eax, 00000041
:004034F4 7C08                   jl 004034FE
:004034F6 83F85A                 cmp eax, 0000005A
:004034F9 7F03                   jg 004034FE
:004034FB 83C020                 add eax, 00000020
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  
|:004034F4(C), :004034F9(C)  
|

```
:004034FE 0FBE540C10             movsx edx, byte ptr [esp+ecx+10]
:00403503 3BD0                   cmp edx, eax
                                  ==== >逐位比较比较 6-9 位！
                                  ==== >[esp+10]=2e37
                                  ==== >[esi+05]=5678
        ★★★★      二、           ==== >所以注册码的 6-9 位是：2e37

:00403505 0F85C8000000           jne 004035D3
                                  ==== >跳则 OVER！

:0040350B 41                     inc ecx
:0040350C 83F904                 cmp ecx, 00000004
:0040350F 7CDB                   jl 004034EC
:00403511 8B442410               mov eax, dword ptr [esp+10]
                                  ==== >EAX=37336532

:00403515 33DB                   xor ebx, ebx
:00403517 3344240C               xor eax, dword ptr [esp+0C]
                                  ==== >EAX=37336532 XOR 31313633=06025301

:0040351B 83C002                 add eax, 00000002
                                  ==== >EAX=06025301 + 00000002=06025303

:0040351E 0FAF44240C             imul eax, dword ptr [esp+0C]
                                  ==== >EAX=06025303 * 31313633=268C2B99

:00403523 03442410               add eax, dword ptr [esp+10]
                                  ==== >EAX=268C2B99 + 37336532=5DBF90CB   得出运算 11-14 位注册码的参数！

:00403527 89442414               mov dword ptr [esp+14], eax
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00403541(C)  
|

```
:0040352B 8D7C1C14               lea edi, dword ptr [esp+ebx+14]
:0040352F 43                     inc ebx
:00403530 0FBE07                 movsx eax, byte ptr [edi]
:00403533 50                     push eax
:00403534 E8B7010000             call 004036F0
                                  ==== >对 CB 90 BF 5D 进行运算得出一组新值 6B 31 31 73

:00403539 83C404                 add esp, 00000004
:0040353C 8807                   mov byte ptr [edi], al
                                  ==== >[edi]=k11s        保存结果

:0040353E 83FB04                 cmp ebx, 00000004
:00403541 7CE8                   jl 0040352B
                                  ==== >循环 4 次

:00403543 33C9                   xor ecx, ecx
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00403564(C)  
|

```
:00403545 0FBE440E0A             movsx eax, byte ptr [esi+ecx+0A]
:0040354A 83F841                 cmp eax, 00000041
:0040354D 7C08                   jl 00403557
:0040354F 83F85A                 cmp eax, 0000005A
:00403552 7F03                   jg 00403557
:00403554 83C020                 add eax, 00000020
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  
|:0040354D(C), :00403552(C)  
|

```
:00403557 0FBE540C14             movsx edx, byte ptr [esp+ecx+14]
:0040355C 3BD0                   cmp edx, eax
                                  ==== >逐位比较比较后 4 位！
                                  ==== >[esp+14]=k11s
                                  ==== >[esi+0A]=90AB
        ★★★★      三、           ==== >所以注册码的 11-14 位是：k11s

:0040355E 757E                   jne 004035DE
                                  ==== >跳则 OVER！

:00403560 41                     inc ecx
:00403561 83F904                 cmp ecx, 00000004
:00403564 7CDF                   jl 00403545
:00403566 8B442410               mov eax, dword ptr [esp+10]
                                  ==== >EAX=37336532

:0040356A 33FF                   xor edi, edi
:0040356C 0FAF442414             imul eax, dword ptr [esp+14]
                                  ==== >EAX=37336532 * 7331316B=BD6BDDE6

:00403571 40                     inc eax
:00403572 0FAF44240C             imul eax, dword ptr [esp+0C]
                                  ==== >EAX=BD6BDDE6 * 31313633=2D82EF05

:00403577 03442414               add eax, dword ptr [esp+14]
                                  ==== >EAX=2D82EF05 + 7331316B=A0B42070   得出运算最后位注册码的参数！

:0040357B 89442418               mov dword ptr [esp+18], eax
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00403595(C)  
|

```
:0040357F 8D5C3C18               lea ebx, dword ptr [esp+edi+18]
:00403583 47                     inc edi
:00403584 0FBE03                 movsx eax, byte ptr [ebx]
:00403587 50                     push eax
:00403588 E863010000             call 004036F0
                                  ==== >对 70 20 B4 A0 进行运算得出一组新值 70 31 34 31

:0040358D 83C404                 add esp, 00000004
:00403590 8803                   mov byte ptr [ebx], al
                                  ==== >[ebx]=p141        保存结果

:00403592 83FF04                 cmp edi, 00000004
:00403595 7CE8                   jl 0040357F
                                  ==== >循环 4 次

:00403597 33C9                   xor ecx, ecx
```
