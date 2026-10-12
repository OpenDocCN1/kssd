
```
00402C6F    6A 40           push    40
00402C71    33C0           xor     eax, eax
00402C73    889D FCFCFFFF mov     byte ptr [ebp-304], bl
00402C79    59            pop     ecx
00402C7A    8DBD FDFCFFFF lea     edi, dword ptr [ebp-303]
00402C80    F3:AB         rep     stos dword ptr es:[edi]
00402C82    66:AB         stos    word ptr es:[edi]
00402C84    AA            stos    byte ptr es:[edi]
00402C85    8D85 FCFCFFFF lea     eax, dword ptr [ebp-304]
00402C8B    50            push    eax
00402C8C    E8 7F210100   call    00414E10
00402C91    6A 11         push    11
00402C93    8D85 FCFCFFFF lea     eax, dword ptr [ebp-304]
00402C99    68 800D4500   push    00450D80            ; ASCII "PortExplorer.exe"
00402C9E    50            push    eax
00402C9F    E8 7CE80200   call    00431520
00402CA4    83C4 10       add     esp, 10
00402CA7    6A 0A         push    0A
00402CA9    53            push    ebx
00402CAA    53            push    ebx
00402CAB    8D85 FCFCFFFF lea     eax, dword ptr [ebp-304]
00402CB1    50            push    eax
00402CB2    68 780D4500   push    00450D78            ; ASCII "open"
00402CB7    53            push    ebx
00402CB8    FF15 9CB24300 call    dword ptr [43B29C]     ; SHELL32.ShellExecuteA
00402CBE    A1 58F54500   mov     eax, dword ptr [45F558]
00402CC3    3BC3          cmp     eax, ebx
00402CC5    8B35 3CB44300 mov     esi, dword ptr [43B43C]    ; user32.SendMessageA
00402CCB    74 0E         je      short 00402CDB
00402CCD    53            push    ebx
00402CCE    68 65050000   push    565
00402CD3    68 11010000   push    111
00402CD8    50            push    eax
00402CD9    FFD6          call    esi
00402CDB    A1 00F54500   mov     eax, dword ptr [45F500]
00402CE0    3BC3          cmp     eax, ebx
00402CE2    74 5F         je      short 00402D43
00402CE4    53            push    ebx
00402CE5    53            push    ebx
00402CE6    6A 10         push    10
00402CE8    50            push    eax
00402CE9    FFD6          call    esi
00402CEB    EB 56         jmp     short 00402D43
00402CED    6A 10         push    10
00402CEF    B8 E0794600   mov     eax, 004679E0         ; ASCII "Error!"
00402CF4    50            push    eax
00402CF5    50            push    eax
00402CF6    53            push    ebx
00402CF7    FF15 F0B34300 call    dword ptr [43B3F0]     ; user32.MessageBoxA
00402CFD    EB 44         jmp     short 00402D43
00402CFF    8B7D 08       mov     edi, dword ptr [ebp+8]
00402D02    8B35 A4B24300 mov     esi, dword ptr [43B2A4]    ; user32.SetDlgItemTextA
00402D08    68 E08A4600   push    00468AE0          ; ASCII "Please enter your unlock code"
00402D0D    68 6F040000   push    46F
00402D12    57            push    edi
00402D13    893D 44EE4800 mov     dword ptr [48EE44], edi
00402D19    FFD6          call    esi
00402D1B    68 E08B4600   push    00468BE0          ; ASCII "&Ok"
00402D20    68 6D040000   push    46D
00402D25    57            push    edi
00402D26    FFD6          call    esi
00402D28    FF15 0CB24300 call    dword ptr [43B20C]
00402D2E    50            push    eax
00402D2F    E8 9CE90200   call    004316D0
00402D34    59            pop     ecx
00402D35    C705 40EE4800 0 mov     dword ptr [48EE40], 8
00402D3F    EB 12         jmp     short 00402D53
00402D41    33DB          xor     ebx, ebx
00402D43    53            push    ebx
00402D44    FF75 08       push    dword ptr [ebp+8]
00402D47    891D 44EE4800 mov     dword ptr [48EE44], ebx
00402D4D    FF15 10B44300 call    dword ptr [43B410]     ; user32.EndDialog
00402D53    5F            pop     edi
00402D54    5E            pop     esi
00402D55    33C0          xor     eax, eax
00402D57    5B            pop     ebx
00402D58    C9            leave
00402D59    C2 1000       retn    10
```

进入

```
0041A250    8038 50       cmp     byte ptr [eax], 50      //再次判断注册开头和结尾是否为 PS 和 PE
0041A253    0F85 E40A0000 jnz     0041AD3D
0041A259    8078 01 53    cmp     byte ptr [eax+1], 53
0041A25D    0F85 DA0A0000 jnz     0041AD3D
0041A263    807C07 FE 50 cmp     byte ptr [edi+eax-2], 50
0041A268    0F85 CF0A0000 jnz     0041AD3D
0041A26E    807C07 FF 45 cmp     byte ptr [edi+eax-1], 45
0041A273    0F85 C40A0000 jnz     0041AD3D
0041A279    83C0 02       add     eax, 2
0041A27C    50            push    eax
0041A27D    8D85 FCFAFFFF lea     eax, dword ptr [ebp-504]
0041A283    50            push    eax
0041A284    FF15 18B24300 call    dword ptr [43B218]
0041A28A    80A43D F8FAFFFF and    byte ptr [ebp+edi-508], 0
0041A292    8D85 FCFAFFFF lea     eax, dword ptr [ebp-504]
0041A298    50            push    eax
0041A299    E8 92700100   call    00431330         //判断注册码的真正长度
0041A29E    3D 12010000   cmp     eax, 112         //真正的注码的位数，PS+112 位注册码+PE
0041A2A3    59            pop     ecx
0041A2A4    0F85 930A0000 jnz     0041AD3D
0041A2AA    8975 FC       mov     dword ptr [ebp-4], esi
0041A2AD    33C0          xor     eax, eax
0041A2AF    8D7D F1       lea     edi, dword ptr [ebp-F]
0041A2B2    66:AB         stos    word ptr es:[edi]
0041A2B4    8B45 FC       mov     eax, dword ptr [ebp-4]
0041A2B7    03C0          add     eax, eax
0041A2B9    8A8C05 FCFAFFFF mov    cl, byte ptr [ebp+eax-504]
0041A2C0    8A8405 FDFAFFFF mov    al, byte ptr [ebp+eax-503]
0041A2C7    6A 10         push    10
0041A2C9    8845 F1       mov     byte ptr [ebp-F], al
0041A2CC    8D45 F0       lea     eax, dword ptr [ebp-10]
0041A2CF    56            push    esi
0041A2D0    50            push    eax
0041A2D1    884D F0       mov     byte ptr [ebp-10], cl
0041A2D4    E8 387C0100   call    00431F11
0041A2D9    8B4D FC       mov     ecx, dword ptr [ebp-4]
0041A2DC    83C4 0C       add     esp, 0C
0041A2DF    FF45 FC       inc     dword ptr [ebp-4]
0041A2E2    817D FC 8900000 cmp    dword ptr [ebp-4], 89    //89=137
0041A2E9    88840D 00FEFFFF mov    byte ptr [ebp+ecx-200], al
0041A2F0    72 BB         jb      short 0041A2AD    //注册码转换为 137 位 16 进制数
0041A2F2    6A 20         push    20      //例 31＝1 32＝2 合并，转换成 12  存放堆栈 0012D230
0041A2F4    8D45 C0       lea     eax, dword ptr [ebp-40]
0041A2F7    50            push    eax
0041A2F8    8D85 00FEFFFF lea     eax, dword ptr [ebp-200]
0041A2FE    68 89000000   push    89
0041A303    50            push    eax
0041A304    E8 45FBFFFF   call    00419E4E    //对注册码进行第一次异或运算
0041A309    83C4 10       add     esp, 10
0041A30C    C745 FC FA00000 mov    dword ptr [ebp-4], 0FA  //常量 FA  ,第二次循环异或运算开始
0041A313    8A55 FC       mov     dl, byte ptr [ebp-4]    //将 FA 传递给 DL
0041A316    8D85 00FEFFFF lea     eax, dword ptr [ebp-200]  //再次出现注册异或后的注册码表
0041A31C    68 89000000   push    89            //89 再次出现，作为常量
0041A321    50            push    eax
0041A322    E8 54FBFFFF   call    00419E7B     //用 FA 参与注册表中的值异或运算
0041A327    FF4D FC       dec     dword ptr [ebp-4]    // FA－1＝F9，依次递减 1
0041A32A    3975 FC       cmp     dword ptr [ebp-4], esi
0041A32D    59            pop     ecx
0041A32E    59            pop     ecx
0041A32F    7F E2         jg      short 0041A313    //由 FA 开头依次递减 1 的异或注册码循环
0041A331    80A5 FCFCFFFF 0 and    byte ptr [ebp-304], 0
```
