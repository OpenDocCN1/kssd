
## 目标地址列表

`0041AD2F`   call    dword ptr [<&USER32.SetTimer>]    `USER32.SetTimer`      这是我们要找的目标

`0041B6A5`   call    dword ptr [<&USER32.SetTimer>]    `USER32.SetTimer`

`0041BF28`   call    dword ptr [<&USER32.SetTimer>]    `USER32.SetTimer`

`0042B4B7`   call    dword ptr [<&USER32.SetTimer>]    `USER32.SetTimer`

`0042DC12`   call    dword ptr [<&USER32.SetTimer>]    `USER32.SetTimer`

`0043014C`   call    dword ptr [<&USER32.SetTimer>]    `USER32.SetTimer`

`00433922`   call    dword ptr [<&USER32.SetTimer>]    `USER32.SetTimer`

`00453303`   call    dword ptr [<&USER32.SetTimer>]    `USER32.SetTimer`

这所以把上面两个列入目标,是因为只有这两个函数的周期比较可以(`300s`和`600s`)

### 第一个目标

双击第一个进入

```
00419ABE  |> \68 20144300   push    00431420                         ; /Timerproc = HooNetMe.00431420
00419AC3  |.  68 E0930400   push    493E0                            ; |Timeout = 300000. ms
00419AC8  |.  55            push    ebp                              ; |TimerID
00419AC9  |.  55            push    ebp                              ; |hWnd
00419ACA  |.  FF15 10E54500 call    dword ptr [<&USER32.SetTimer>]   ; \SetTimer
```

然后再来到`HooNetMe.00431420`

```
00431420   .  56            push    esi
00431421   .  E8 E4D10100   call    0044E60A
00431426   .  85C0          test    eax, eax
00431428   .  74 0B         je      short 00431435
0043142A   .  8B10          mov     edx, dword ptr [eax]
0043142C   .  8BC8          mov     ecx, eax
0043142E   .  FF52 74       call    dword ptr [edx+74]
00431431   .  8BF0          mov     esi, eax
00431433   .  EB 02         jmp     short 00431437
00431435   >  33F6          xor     esi, esi
00431437   >  F786 A0000000>test    dword ptr [esi+A0], A6000000
00431441   .  74 33         je      short 00431476
00431443   .  8D46 5C       lea     eax, dword ptr [esi+5C]
00431446   .  50            push    eax
00431447   .  E8 D44E0000   call    00436320                        ;该函数很可疑,进入看看
0043144C   .  83C4 04       add     esp, 4
0043144F   .  85C0          test    eax, eax
00431451   .  75 23         jnz     short 00431476                  ; 爆破点
00431453   .  817E 7C 20040>cmp     dword ptr [esi+7C], 420
0043145A   .  76 1A         jbe     short 00431476
0043145C   .  8BCE          mov     ecx, esi
0043145E   .  E8 DDA1FEFF   call    0041B640
00431463   .  8B4E 1C       mov     ecx, dword ptr [esi+1C]
00431466   .  6A 00         push    0                                ; /lParam = 0
00431468   .  6A 00         push    0                                ; |wParam = 0
0043146A   .  68 77040000   push    477                              ; |Message = MSG(477)
0043146F   .  51            push    ecx                              ; |hWnd
00431470   .  FF15 08E54500 call    dword ptr [<&USER32.PostMessageW>]; \PostMessageW
00431476   >  8B96 A4000000 mov     edx, dword ptr [esi+A4]
0043147C   .  52            push    edx                              ; /TimerID
0043147D   .  6A 00         push    0                                ; |hWnd = NULL
0043147F   .  FF15 04E54500 call    dword ptr [<&USER32.KillTimer>]  ; \KillTimer
00431485   .  5E            pop     esi
00431486   .  C2 1000       retn    10
```

进入`436320`

```
00436320  /$  8B4424 04     mov     eax, dword ptr [esp+4]
00436324  |.  53            push    ebx
00436325  |.  55            push    ebp
00436326  |.  56            push    esi
00436327  |.  8B50 14       mov     edx, dword ptr [eax+14]
0043632A  |.  8B18          mov     ebx, dword ptr [eax]
0043632C  |.  8B68 18       mov     ebp, dword ptr [eax+18]
0043632F  |.  8BCA          mov     ecx, edx
00436331  |.  33CB          xor     ecx, ebx
00436333  |.  8B58 08       mov     ebx, dword ptr [eax+8]
00436336  |.  81E1 AB793B57 and     ecx, 573B79AB
0043633C  |.  57            push    edi
0043633D  |.  33CA          xor     ecx, edx
0043633F  |.  8B50 0C       mov     edx, dword ptr [eax+C]
00436342  |.  33EA          xor     ebp, edx
00436344  |.  81F1 ECA89458 xor     ecx, 5894A8EC                                                                ;ecx = a[5] ^ ((a[0] ^ a[5]) & 0x573B79AB) ^ 0x5894A8EC
0043634A  |.  81E5 31908848 and     ebp, 48889031
00436350  |.  8B78 10       mov     edi, dword ptr [eax+10]
00436353  |.  33EA          xor     ebp, edx
00436355  |.  B8 ABAAAAAA   mov     eax, AAAAAAAB
0043635A  |.  81F5 64316121 xor     ebp, 21613164               ;ebp = a[3] ^ ((a[3] ^ a[6]) & 0x48889031) ^ 0x21613164
00436360  |.  8BF3          mov     esi, ebx
00436362  |.  0FAFE9        imul    ebp, ecx
00436365  |.  F7E5          mul     ebp
00436367  |.  33F7          xor     esi, edi
00436369  |.  81E6 77777777 and     esi, 77777777
0043636F  |.  8BC6          mov     eax, esi
00436371  |.  33F3          xor     esi, ebx                    ;esi = a[2] ^ ((a[4] ^ a[2]) & 0x77777777)
00436373  |.  33C7          xor     eax, edi
00436375  |.  5F            pop     edi
00436376  |.  D1EA          shr     edx, 1                      ;edx =
00436378  |.  35 159A4865   xor     eax, 65489A15               ;eax = a[4] ^ ((a[4] ^ a[2]) & 0x77777777) ^ 0x65489a15
0043637D  |.  2BC2          sub     eax, edx
0043637F  |.  03C1          add     eax, ecx
00436381  |.  33C9          xor     ecx, ecx
00436383  |.  F7D0          not     eax
00436385  |.  3BC6          cmp     eax, esi
00436387  |.  5E            pop     esi
00436388  |.  0F94C1        sete    cl
0043638B  |.  5D            pop     ebp
0043638C  |.  8BC1          mov     eax, ecx
0043638E  |.  5B            pop     ebx
0043638F  \.  C3            retn
```

该函数伪代码如下:

```c
int foo(int[8] a){
    //这里用到了a[0], a[2], a[3], a[4], a[5], a[6]
    if(~((a[5] ^ ((a[0] ^ a[5]) & 0x573B79AB) ^ 0x5894A8EC)
         + (a[4] ^ ((a[4] ^ a[2]) & 0x77777777) ^ 0x65489a15)
         - (a[5] ^ ((a[0] ^ a[5]) & 0x573B79AB) ^ 0x5894A8EC)
         * (a[3] ^ ((a[3] ^ a[6]) & 0x48889031) ^ 0x21613164)
         / 3) == (a[2] ^ ((a[4] ^ a[2]) & 0x77777777)))
         return 1;
         return 0;
}
```

由此可见，注册码还必须满足这里的条件。

### 第二个目标

双击第二个目标进入

```
0041AD23  |> \68 507D4200   push    00427D50                         ; /Timerproc = HooNetMe.00427D50
0041AD28  |.  68 C0270900   push    927C0                            ; |Timeout = 600000. ms
0041AD2D  |.  53            push    ebx                              ; |TimerID
0041AD2E  |.  53            push    ebx                              ; |hWnd
0041AD2F  |.  FF15 10E54500 call    dword ptr [<&USER32.SetTimer>]   ; \SetTimer
```

然后再来到`HooNetMe.00427D50`

```
00427D50   .  56            push    esi
00427D51   .  E8 B4680200   call    0044E60A
00427D56   .  85C0          test    eax, eax
00427D58   .  74 0B         je      short 00427D65
00427D5A   .  8B10          mov     edx, dword ptr [eax]
00427D5C   .  8BC8          mov     ecx, eax
00427D5E   .  FF52 74       call    dword ptr [edx+74]
00427D61   .  8BF0          mov     esi, eax
00427D63   .  EB 02         jmp     short 00427D67
00427D65   >  33F6          xor     esi, esi
00427D67   >  F786 A0000000>test    dword ptr [esi+A0], A6000000
00427D71   .  74 33         je      short 00427DA6
00427D73   .  8D46 5C       lea     eax, dword ptr [esi+5C]
00427D76   .  50            push    eax
00427D77   .  E8 44E10000   call    00435EC0                   ;该函数比较可以,进去查看后实际会发现就是serialNo_check2的功能
00427D7C   .  83C4 04       add     esp, 4
00427D7F   .  85C0          test    eax, eax
00427D81   .  75 23         jnz     short 00427DA6             ; 爆破点
00427D83   .  817E 7C E0030>cmp     dword ptr [esi+7C], 3E0    ;发注册码不合法消息,从而弹出窗口提示试用结束(估计应该还会先检测是否过试用期)
00427D8A   .  76 1A         jbe     short 00427DA6
00427D8C   .  8BCE          mov     ecx, esi
00427D8E   .  E8 AD38FFFF   call    0041B640
00427D93   .  8B4E 1C       mov     ecx, dword ptr [esi+1C]
00427D96   .  6A 00         push    0                                ; /lParam = 0
00427D98   .  6A 00         push    0                                ; |wParam = 0
00427D9A   .  68 77040000   push    477                              ; |Message = MSG(477)
```
