
```
:00471974 03D9     add ebx, ecx
:00471976 81C3A41D0F00 add ebx, 000F1DA4
:0047197C 40     inc eax
:0047197D 4A     dec edx
:0047197E 75F0   jne 00471970

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

:00471968(C)

:00471980 8D85F0FCFFFF lea eax, dword ptr [ebp+FFFFFCF0]
:00471986 8D95FFFCFFFF lea edx, dword ptr [ebp+FFFFFCFF]
:0047198C E85F24F9FF call 00403DF0
:00471991 8B85F0FCFFFF mov eax, dword ptr [ebp+FFFFFCF0]
:00471997 50     push eax
:00471998 8D95ECFCFFFF lea edx, dword ptr [ebp+FFFFFCEC]
:0047199E 8BC3   mov eax, ebx
:004719A0 E8E374F9FF call 00408E88 <---用 xxxx 算出正确的注册码;
执行完上面这条语句后,EDX 中就是
正确的注册码,我的是：5944406
:004719A5 8B95ECFCFFFF mov edx, dword ptr [ebp+FFFFFCEC]
:004719AB 58     pop eax
:004719AC E8AB25F9FF call 00403F5C <---判断 yyyy 与上面用 xxxx 算出的
注册码是否相等；
:004719B1 750D   jne 004719C0 <---不等就跳走;
:004719B3 80BD00FFFFFF61 cmp byte ptr [ebp+FFFFFF00], 61
:004719BA 7204   jb 004719C0
:004719BC C645FF01 mov [ebp-01], 01

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

:0047191F(C), :004719B1(C), :004719BA(C)

:004719C0 33C0   xor eax, eax <---可爱的 EAX 标志被置 0，就 OVER 了
:004719C2 5A     pop edx
:004719C3 59     pop ecx
:004719C4 59     pop ecx
:004719C5 648910 mov dword ptr fs:[eax], edx
:004719C8 68E5194700 push 004719E5

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

:004719E3(U)

:004719CD 8D85ECFCFFFF lea eax, dword ptr [ebp+FFFFFCEC]
:004719D3 BA04000000 mov edx, 00000004
:004719D8 E81322F9FF call 00403BF0
:004719DD C3     ret

--------------------------------------------------------------------------------

由 `call 00402A20` 跟入：此 Call 判断注册码是否为 `xxxx-yyyy` 的形式.

:00402A20 53     push ebx
:00402A21 56     push esi
:00402A22 57     push edi
:00402A23 89C6   mov esi, eax
:00402A25 89D7   mov edi, edx
:00402A27 31C9   xor ecx, ecx
:00402A29 8A0F   mov cl, byte ptr [edi]
:00402A2B 47     inc edi
:00402A2C 57     push edi
:00402A2D 31D2   xor edx, edx
:00402A2F 8A16   mov dl, byte ptr [esi]
:00402A31 46     inc esi
:00402A32 4A     dec edx
:00402A33 781B   js 00402A50
:00402A35 8A06   mov al, byte ptr [esi] <---将 AL 赋值'2D',也就是符号'-';
:00402A37 46     inc esi
:00402A38 29D1   sub ecx, edx
:00402A3A 7E14   jle 00402A50

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

:00402A4E(U)

:00402A3C F2     repnz
:00402A3D AE     scasb <---循环依次取输入的注册码与 AL 中的'-'比较
:00402A3E 7510   jne 00402A50 <---注册码中没有'-'符就跳走；
:00402A40 89CB   mov ebx, ecx
:00402A42 56     push esi
:00402A43 57     push edi
:00402A44 89D1   mov ecx, edx
:00402A46 F3     repz
:00402A47 A6     cmpsb
:00402A48 5F     pop edi
:00402A49 5E     pop esi
:00402A4A 7409   je 00402A55
:00402A4C 89D9   mov ecx, ebx
:00402A4E EBEC   jmp 00402A3C

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

:00402A33(C), :00402A3A(C), :00402A3E(C)

:00402A50 5A     pop edx
:00402A51 31C0   xor eax, eax
:00402A53 EB05   jmp 00402A5A
```

整理一下我的注册码为：`liangs-5944406` `4dd466d0e1e0cc123afbe504da2462ca`
