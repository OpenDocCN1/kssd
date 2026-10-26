
```asm
004472D0  /.  55            push    ebp                              ;  按钮事件代码开始处

004472D1  |.  8BEC          mov     ebp, esp

004472D3  |.  33C9          xor     ecx, ecx

004472D5  |.  51            push    ecx

004472D6  |.  51            push    ecx

004472D7  |.  51            push    ecx

004472D8  |.  51            push    ecx

004472D9  |.  51            push    ecx

004472DA  |.  51            push    ecx

004472DB  |.  53            push    ebx

004472DC  |.  56            push    esi

004472DD  |.  8BD8          mov     ebx, eax

004472DF  |.  33C0          xor     eax, eax

004472E1  |.  55            push    ebp

004472E2  |.  68 8B744400   push    0044748B

004472E7  |.  64:FF30       push    dword ptr fs:[eax]

004472EA  |.  64:8920       mov     dword ptr fs:[eax], esp

004472ED  |.  8D55 F4       lea     edx, dword ptr [ebp-C]

004472F0  |.  8B83 E0010000 mov     eax, dword ptr [ebx+1E0]

004472F6  |.  E8 0D6FFDFF   call    0041E208                         ;  取试炼码

004472FB  |.  837D F4 00    cmp     dword ptr [ebp-C], 0             ;  试炼码是否为空？

004472FF  |.  0F84 5B010000 je      00447460                         ;  为空则跳向失败

00447305  |.  8D55 F0       lea     edx, dword ptr [ebp-10]

00447308  |.  8B83 DC010000 mov     eax, dword ptr [ebx+1DC]

0044730E  |.  E8 F56EFDFF   call    0041E208                         ;  取用户名

00447313  |.  837D F0 00    cmp     dword ptr [ebp-10], 0            ;  用户名是否为空？

00447317  |.  0F84 43010000 je      00447460                         ;  为空则跳向失败

0044731D  |.  8D55 F4       lea     edx, dword ptr [ebp-C]

00447320  |.  8B83 E0010000 mov     eax, dword ptr [ebx+1E0]

00447326  |.  E8 DD6EFDFF   call    0041E208                         ;  取试炼码

0044732B  |.  8B45 F4       mov     eax, dword ptr [ebp-C]           ;  试炼码地址入EAX

0044732E  |.  E8 65FAFBFF   call    00406D98                         ;  试炼码由字符串转为对应16进制

00447333  |.  8945 EC       mov     dword ptr [ebp-14], eax          ;  放入[ebp-14]

00447336  |.  DB45 EC       fild    dword ptr [ebp-14]               ;  将整数试炼码16进制装入st(0)

00447339  |.  E8 8AB6FBFF   call    004029C8                         ;  将st(0)值以整数保存到eax

0044733E  |.  8945 FC       mov     dword ptr [ebp-4], eax

00447341  |.  8D55 F8       lea     edx, dword ptr [ebp-8]

00447344  |.  8B83 DC010000 mov     eax, dword ptr [ebx+1DC]

0044734A  |.  E8 B96EFDFF   call    0041E208

0044734F  |.  33F6          xor     esi, esi                         ;  esi清零

00447351  |.  8B45 F8       mov     eax, dword ptr [ebp-8]           ;  用户名地址入EAX

00447354  |.  E8 6BC8FBFF   call    00403BC4                         ;  取用户名长度,设为len

00447359  |.  85C0          test    eax, eax

0044735B  |.  7E 13         jle     short 00447370

0044735D  |.  BA 01000000   mov     edx, 1                           ;  edx=1

00447362  |>  8B4D F8       /mov     ecx, dword ptr [ebp-8]          ;  用户名地址传入ecx

00447365  |.  0FB64C11 FF   |movzx   ecx, byte ptr [ecx+edx-1]       ;  用户名逐位扩展送入ecx

0044736A  |.  03F1          |add     esi, ecx                        ;  esi=esi+ecx,通过循环esi内保存用户名ASCII值之和，设为sum

0044736C  |.  42            |inc     edx                             ;  edx增1

0044736D  |.  48            |dec     eax                             ;  eax减1

0044736E  |.^ 75 F2         \jnz     short 00447362                  ;  循环次数取决于用户名长度

00447370  |>  8975 EC       mov     dword ptr [ebp-14], esi

00447373  |.  DB45 EC       fild    dword ptr [ebp-14]               ;  将整数sum装入st(0)

00447376  |.  E8 4DB6FBFF   call    004029C8                         ;  将st(0)值以整数保存到eax

0044737B  |.  8BD0          mov     edx, eax                         ;  edx=eax=sum

0044737D  |.  C1E0 07       shl     eax, 7                           ;  eax值左移7位

00447380  |.  03C2          add     eax, edx                         ;  eax=eax+edx

00447382  |.  8BF0          mov     esi, eax                         ;  eax 值传入esi，查看eax值10进制就是真正注册码了

00447384  |.  3B75 FC       cmp     esi, dword ptr [ebp-4]           ;  esi值与[ebp-4]内试炼码的16进制相比较

00447387  |.  0F85 C9000000 jnz     00447456                         ;  不等则跳向失败，爆破点

0044738D  |.  B2 01         mov     dl, 1

0044738F  |.  A1 28634400   mov     eax, dword ptr [446328]

00447394  |.  E8 8BF0FFFF   call    00446424

00447399  |.  8BF0          mov     esi, eax

0044739B  |.  B1 01         mov     cl, 1

0044739D  |.  BA A0744400   mov     edx, 004474A0                    ;  \software\lighttek\melody

004473A2  |.  8BC6          mov     eax, esi

004473A4  |.  E8 73F1FFFF   call    0044651C

004473A9  |.  8D55 F4       lea     edx, dword ptr [ebp-C]

004473AC  |.  8B83 DC010000 mov     eax, dword ptr [ebx+1DC]

004473B2  |.  E8 516EFDFF   call    0041E208

004473B7  |.  8B4D F4       mov     ecx, dword ptr [ebp-C]

004473BA  |.  BA C4744400   mov     edx, 004474C4                    ;  username

004473BF  |.  8BC6          mov     eax, esi

004473C1  |.  E8 1AF3FFFF   call    004466E0

004473C6  |.  8D55 F4       lea     edx, dword ptr [ebp-C]

004473C9  |.  8B83 E0010000 mov     eax, dword ptr [ebx+1E0]

004473CF  |.  E8 346EFDFF   call    0041E208

004473D4  |.  8B45 F4       mov     eax, dword ptr [ebp-C]

004473D7  |.  E8 BCF9FBFF   call    00406D98

004473DC  |.  8945 EC       mov     dword ptr [ebp-14], eax

004473DF  |.  DB45 EC       fild    dword ptr [ebp-14]

004473E2  |.  83C4 F8       add     esp, -8

004473E5  |.  DD1C24        fstp    qword ptr [esp]

004473E8  |.  9B            wait

004473E9  |.  BA D8744400   mov     edx, 004474D8                    ;  usercode

004473EE  |.  8BC6          mov     eax, esi

004473F0  |.  E8 1BF4FFFF   call    00446810

004473F5  |.  8BC6          mov     eax, esi

004473F7  |.  E8 8CF0FFFF   call    00446488

004473FC  |.  8BC6          mov     eax, esi

004473FE  |.  E8 31BAFBFF   call    00402E34

00447403  |.  8D55 F4       lea     edx, dword ptr [ebp-C]

00447406  |.  8B83 DC010000 mov     eax, dword ptr [ebx+1DC]

0044740C  |.  E8 F76DFDFF   call    0041E208

00447411  |.  8B4D F4       mov     ecx, dword ptr [ebp-C]

00447414  |.  8D45 E8       lea     eax, dword ptr [ebp-18]

00447417  |.  BA EC744400   mov     edx, 004474EC                    ;  registered for

0044741C  |.  E8 EFC7FBFF   call    00403C10

00447421  |.  8B55 E8       mov     edx, dword ptr [ebp-18]

00447424  |.  A1 10FE4400   mov     eax, dword ptr [44FE10]

00447429  |.  8B00          mov     eax, dword ptr [eax]

0044742B  |.  8B80 A8020000 mov     eax, dword ptr [eax+2A8]

00447431  |.  E8 026EFDFF   call    0041E238

00447436  |.  A1 10FE4400   mov     eax, dword ptr [44FE10]

0044743B  |.  8B00          mov     eax, dword ptr [eax]

0044743D  |.  8B80 A4020000 mov     eax, dword ptr [eax+2A4]

00447443  |.  33D2          xor     edx, edx

00447445  |.  E8 1A6DFDFF   call    0041E164

0044744A  |.  A1 B0074500   mov     eax, dword ptr [4507B0]

0044744F  |.  E8 1C49FEFF   call    0042BD70

00447454  |.  EB 0A         jmp     short 00447460

00447456  |>  B8 04754400   mov     eax, 00447504                    ;  invalide registration name or code. try again.

0044745B  |.  E8 E000FFFF   call    00437540

00447460  |>  33C0          xor     eax, eax

00447462  |.  5A            pop     edx

00447463  |.  59            pop     ecx

00447464  |.  59            pop     ecx

00447465  |.  64:8910       mov     dword ptr fs:[eax], edx

00447468  |.  68 92744400   push    00447492

0044746D  |>  8D45 E8       lea     eax, dword ptr [ebp-18]

00447470  |.  E8 D3C4FBFF   call    00403948

00447475  |.  8D45 F0       lea     eax, dword ptr [ebp-10]

00447478  |.  BA 02000000   mov     edx, 2

0044747D  |.  E8 EAC4FBFF   call    0040396C

00447482  |.  8D45 F8       lea     eax, dword ptr [ebp-8]

00447485  |.  E8 BEC4FBFF   call    00403948

0044748A  \.  C3            retn
```

## 五、算法总结

程序算法非常简单，正所谓提高菜鸟信心之练手绝佳对象。君不见中国男足热身老找柬埔寨、老挝等球队？找感觉啊

1、求出用户名ASCII值之和，设为`sum`；

2、`sum`左移7位，然后与`sum`相加；

3、上面值转为10进制，即为注册码。

## 六、keymake汇编注册机代码
