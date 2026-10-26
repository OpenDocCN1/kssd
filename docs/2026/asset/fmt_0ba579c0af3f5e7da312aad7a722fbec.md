
```asm
004060A7   .  C2 2C00       retn    0x2C

004060AA   >  55            push    ebp

004060AB   .  56            push    esi

004060AC   .  33C0          xor     eax, eax

004060AE   .  57            push    edi

004060AF   .  884424 5C     mov     byte ptr [esp+0x5C], al

004060B3   .  EB 0B         jmp     short 004060C0

004060B5   >  8B5C24 14     mov     ebx, dword ptr [esp+0x14]

004060B9   .  8B4424 18     mov     eax, dword ptr [esp+0x18]

004060BD   .  8D49 00       lea     ecx, dword ptr [ecx]

004060C0   >  85C0          test    eax, eax

004060C2   .  7C 1D         jl      short 004060E1

004060C4   .  8B8B 88020000 mov     ecx, dword ptr [ebx+0x288]

004060CA   .  3BC1          cmp     eax, ecx

004060CC   .  7D 13         jge     short 004060E1

004060CE   .  3BC1          cmp     eax, ecx

004060D0   .  0F8D 58020000 jge     0040632E

004060D6   .  8B8B 7C020000 mov     ecx, dword ptr [ebx+0x27C]              ;  ！！！！！存有坐标参数

004060DC   .  8B2C81        mov     ebp, dword ptr [ecx+eax*4]

004060DF   .  EB 02         jmp     short 004060E3

004060E1   >  33ED          xor     ebp, ebp

004060E3   >  40            inc     eax

004060E4   .  85ED          test    ebp, ebp

004060E6   .  894424 18     mov     dword ptr [esp+0x18], eax

004060EA   .  0F84 28010000 je      00406218

004060F0   .  837D 08 01    cmp     dword ptr [ebp+0x8], 0x1

004060F4   .  74 07         je      short 004060FD

004060F6   .  807C24 50 01  cmp     byte ptr [esp+0x50], 0x1

004060FB   .^ 74 C3         je      short 004060C0

004060FD   >  66:8B8D 32400>mov     cx, word ptr [ebp+0x4032]

00406104   .  66:3B8D 30400>cmp     cx, word ptr [ebp+0x4030]

0040610B   .^ 73 B3         jnb     short 004060C0

0040610D   .  8B93 58020000 mov     edx, dword ptr [ebx+0x258]              ;  ！！！！！

00406113   .  807D 01 02    cmp     byte ptr [ebp+0x1], 0x2

00406117   .  8B3C95 283549>mov     edi, dword ptr [edx*4+0x493528]

0040611E   .  8BF7          mov     esi, edi

00406120   .  75 3D         jnz     short 0040615F

00406122   .  8BC2          mov     eax, edx

00406124   .  83F8 0A       cmp     eax, 0xA                                ;  Switch (cases A..C)

00406127   .  75 0D         jnz     short 00406136

00406129   .  8B3D 84354900 mov     edi, dword ptr [0x493584]               ;  Case A of switch 00406124

0040612F   .  A1 14354900   mov     eax, dword ptr [0x493514]

00406134   .  EB 22         jmp     short 00406158

00406136   >  83F8 0B       cmp     eax, 0xB

00406139   .  75 0D         jnz     short 00406148

0040613B   .  8B3D 88354900 mov     edi, dword ptr [0x493588]               ;  Case B of switch 00406124

00406141   .  A1 18354900   mov     eax, dword ptr [0x493518]

00406146   .  EB 10         jmp     short 00406158

00406148   >  83F8 0C       cmp     eax, 0xC

0040614B   .  75 12         jnz     short 0040615F

0040614D   .  8B3D 8C354900 mov     edi, dword ptr [0x49358C]               ;  Case C of switch 00406124

00406153   .  A1 1C354900   mov     eax, dword ptr [0x49351C]

00406158   >  99            cdq

00406159   .  2BC2          sub     eax, edx

//开火post

0040F10E    52              push    edx

0040F10F    C74424 48 00000>mov     dword ptr ss:[esp+48], 0

0040F117    8B94C6 E8380000 mov     edx, dword ptr ds:[esi+eax*8+38E8]

0040F11E    D98486 44390000 fld     dword ptr ds:[esi+eax*4+3944]

0040F125    8B84C6 E4380000 mov     eax, dword ptr ds:[esi+eax*8+38E4]

0040F12C    D95C24 20       fstp    dword ptr ss:[esp+20]

0040F130    8B4C24 20       mov     ecx, dword ptr ss:[esp+20]

0040F134    51              push    ecx

0040F135    52              push    edx

0040F136    8B5424 68       mov     edx, dword ptr ss:[esp+68]

0040F13A    50              push    eax

0040F13B    6A 05           push    5

0040F13D    8D4C24 58       lea     ecx, dword ptr ss:[esp+58]

0040F141    51              push    ecx

0040F142    6A 01           push    1

0040F144    52              push    edx

0040F145    8BCD            mov     ecx, ebp

0040F147    E8 542BFFFF     call    00401CA0

0040F14C    8BE8            mov     ebp, eax

0040F14E    33C0            xor     eax, eax

0040F150    66:8B86 B001000>mov     ax, word ptr ds:[esi+1B0]

0040F157    8D8E 8C360000   lea     ecx, dword ptr ds:[esi+368C]

0040F15D    50              push    eax

0040F15E    E8 5D3BFFFF     call    00402CC0

0040F163    85ED            test    ebp, ebp

0040F165    75 0C           jnz     short 0040F173

0040F167    5B              pop     ebx

0040F168    5F              pop     edi

0040F169    5D              pop     ebp

0040F16A    32C0            xor     al, al

0040F16C    5E              pop     esi

0040F16D    83C4 3C         add     esp, 3C

0040F170    C2 0C00         retn    0C

0040F173    6A 08           push    8

0040F175    E8 88A40000     call    <jmp.&MFC71.#operator new_762>

0040F17A    8B4C24 5C       mov     ecx, dword ptr ss:[esp+5C]

0040F17E    8928            mov     dword ptr ds:[eax], ebp

0040F180    8948 04         mov     dword ptr ds:[eax+4], ecx

0040F183    8B8E 40760000   mov     ecx, dword ptr ds:[esi+7640]

0040F189    83C4 04         add     esp, 4

0040F18C    85C9            test    ecx, ecx

0040F18E    74 0C           je      short 0040F19C

0040F190    53              push    ebx

0040F191    50              push    eax

0040F192    68 64040000     push    464                                    ; 消息号

0040F197    E8 0425FFFF     call    004016A0                               ; 开火 发消息(内部post)

0040F19C    833CBD 74414C00>cmp     dword ptr ds:[edi*4+4C4174], 1

0040F1A4    8BCE            mov     ecx, esi

0040F1A6    6A 01           push    1

0040F1A8    75 09           jnz     short 0040F1B3

0040F1AA    6A 0C           push    0C

0040F1AC    68 84384600     push    00463884                               ; ASCII "28.ogg"

0040F1B1    EB 07           jmp     short 0040F1BA

0040F1B3    6A 0E           push    0E

0040F1B5    68 7C384600     push    0046387C                               ; ASCII "25.ogg"

0040F1BA    E8 E1E7FFFF     call    0040D9A0

0040F1BF    837C24 14 28    cmp     dword ptr ss:[esp+14], 28

0040F1C4    8BBCBE C8000000 mov     edi, dword ptr ds:[esi+edi*4+C8]

0040F1CB    8B1D 4C174600   mov     ebx, dword ptr ds:[<&USER32.SetTimer>] ; user32.SetTimer

0040F1D1    0F8C A3000000   jl      0040F27A
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [`金蟾捕鱼.zip`](javascript:void(0);)

  （5.10kb，159次下载）
