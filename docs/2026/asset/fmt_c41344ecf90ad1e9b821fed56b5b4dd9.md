
```
       lea     edx, [esp+2D0h+NumberOfBytesRead]

.text:004012D1 8D 84 24 D4 00 00 00                    lea     eax, [esp+2D0h+Buffer]

.text:004012D8 52                                      push    edx             ; lpNumberOfBytesRead

.text:004012D9 68 00 01 00 00                          push    100h            ; nSize

.text:004012DE 50                                      push    eax             ; lpBuffer

.text:004012DF 57                                      push    edi             ; lpBaseAddress

.text:004012E0 53                                      push    ebx             ; hProcess

.text:004012E1 FF D5                                   call    ebp ; ReadProcessMemory ; 读取内存数据

.text:004012E3 8B 84 24 D4 00 00 00                    mov     eax, [esp+2D0h+Buffer]

.text:004012EA 3B C7                                   cmp     eax, edi

.text:004012EC 0F 84 B6 00 00 00                       jz      loc_4013A8

.text:004012F2

.text:004012F2                         loc_4012F2:                             ; CODE XREF: _main+1DCj

.text:004012F2 8D 4C 24 28                             lea     ecx, [esp+2D0h+NumberOfBytesRead]

.text:004012F6 8D 94 24 D4 00 00 00                    lea     edx, [esp+2D0h+Buffer]

.text:004012FD 51                                      push    ecx             ; lpNumberOfBytesRead

.text:004012FE 68 00 01 00 00                          push    100h            ; nSize

.text:00401303 52                                      push    edx             ; lpBuffer

.text:00401304 50                                      push    eax             ; lpBaseAddress

.text:00401305 53                                      push    ebx             ; hProcess

.text:00401306 FF D5                                   call    ebp ; ReadProcessMemory ; 读取内存数据

.text:00401308 8B 84 24 E4 00 00 00                    mov     eax, [esp+2D0h+nSize]

.text:0040130F 8B 0E                                   mov     ecx, [esi]

.text:00401311 3B C1                                   cmp     eax, ecx        ; 检测读取到的LUID值是否是指定的LUID值，如果不是，则比较地址，如果当前地址小于目标地址，则继续读取内存，查找LUID

.text:00401313 75 0E                                   jnz     short loc_401323

.text:00401315 8B 8C 24 E8 00 00 00                    mov     ecx, [esp+2D0h+lpBaseAddress]

.text:0040131C 8B 46 04                                mov     eax, [esi+4]

.text:0040131F 3B C8                                   cmp     ecx, eax

.text:00401321 74 7C                                   jz      short loc_40139F

.text:00401323

.text:00401323                         loc_401323:                             ; CODE XREF: _main+1C3j

.text:00401323 8B 84 24 D4 00 00 00                    mov     eax, [esp+2D0h+Buffer]

.text:0040132A 3B C7                                   cmp     eax, edi

.text:0040132C 75 C4                                   jnz     short loc_4012F2

.text:0040132E 68 34 91 40 00                          push    offset aSpecificLuidNo ; "Specific LUID NOT found\n"

.text:00401333 E8 9E 04 00 00                          call    _puts

.text:00401338 83 C4 04                                add     esp, 4

.text:0040133B E9 D0 00 00 00                          jmp     loc_401410

.text:00401340                         ; ---------------------------------------------------------------------------

.text:00401340

.text:00401340                         loc_401340:                             ; CODE XREF: _main+9Fj

.text:00401340 83 F8 02                                cmp     eax, 2

.text:00401343 75 2F                                   jnz     short loc_401374

.text:00401345 C7 44 24 10 1C 00 00 00                 mov     [esp+2CCh+var_2BC], 1Ch

.text:0040134D C7 44 24 0C 04 00 00 00                 mov     [esp+2CCh+var_2C0], 4

.text:00401355 E9 AB FE FF FF                          jmp     loc_401205

.text:0040135A                         ; ---------------------------------------------------------------------------

.text:0040135A

.text:0040135A                         loc_40135A:                             ; CODE XREF: _main+96j

.text:0040135A 83 F9 06                                cmp     ecx, 6

.text:0040135D 75 15                                   jnz     short loc_401374

.text:0040135F C7 44 24 10 20 00 00 00                 mov     [esp+2CCh+var_2BC], 20h

.text:00401367 C7 44 24 0C 01 00 00 00                 mov     [esp+2CCh+var_2C0], 1

.text:0040136F E9 91 FE FF FF                          jmp     loc_401205

.text:00401374                         ; ---------------------------------------------------------------------------

.text:00401374

.text:00401374                         loc_401374:                             ; CODE XREF: _main+1F3j

.text:00401374                                                                 ; _main+20Dj

.text:00401374 50                                      push    eax

.text:00401375 51                                      push    ecx

.text:00401376 68 04 91 40 00                          push    offset aUndefinedOsVer ; "[Undefined OS version]  Major: %d Minor"...

.text:0040137B E8 BD 04 00 00                          call    sub_40183D

.text:00401380 68 B0 91 40 00                          push    offset aEchoPressAnyKe ; "echo Press any Key to Continue ... & pa"...

.text:00401385 E8 B6 03 00 00                          call    _system

.text:0040138A 83 C4 10                                add     esp, 10h

.text:0040138D 53                                      push    ebx             ; hObject

.text:0040138E FF 15 38 80 40 00                       call    ds:CloseHandle

.text:00401394 5F                                      pop     edi

.text:00401395 33 C0                                   xor     eax, eax

.text:00401397 5B                                      pop     ebx

.text:00401398 81 C4 C4 02 00 00                       add     esp, 2C4h

.text:0040139E C3                                      retn

.text:0040139F                         ; ---------------------------------------------------------------------------

.text:0040139F

.text:0040139F                         loc_40139F:                             ; CODE XREF: _main+1D1j

.text:0040139F 39 BC 24 D4 00 00 00                    cmp     [esp+2D0h+Buffer], edi

.text:004013A6 75 0F                                   jnz     short loc_4013B7

.text:004013A8

.text:004013A8                         loc_4013A8:                             ; CODE XREF: _main+19Cj

.text:004013A8 68 34 91 40 00                          push    offset aSpecificLuidNo ; "Specific LUID NOT found\n"

.text:004013AD E8 24 04 00 00                          call    _puts

.text:004013B2 83 C4 04                                add     esp, 4

.text:004013B5 EB 59                                   jmp     short loc_401410

.text:004013B7                         ; ---------------------------------------------------------------------------

.text:004013B7

.text:004013B7                         loc_4013B7:                             ; CODE XREF: _main+256j

.text:004013B7 8B 54 24 18                             mov     edx, [esp+2D0h+var_2B8]

.text:004013BB 33 F6                                   xor     esi, esi

.text:004013BD B9 40 00 00 00                          mov     ecx, 40h

.text:004013C2 BF C0 BB 40 00                          mov     edi, offset unk_40BBC0

.text:004013C7 66 8B B4 14 E6 00 00 00                 mov     si, word ptr [esp+edx+2D0h+nSize+2]

.text:004013CF 8D 84 14 E4 00 00 00                    lea     eax, [esp+edx+2D0h+nSize]

.text:004013D6 8B 94 14 E8 00 00 00                    mov     edx, [esp+edx+2D0h+lpBaseAddress]

.text:004013DD 33 C0                                   xor     eax, eax

.text:004013DF F3 AB                                   rep stosd

.text:004013E1 8D 44 24 28                             lea     eax, [esp+2D0h+NumberOfBytesRead]

.text:004013E5 50                                      push    eax             ; lpNumberOfBytesRead

.text:004013E6 56                                      push    esi             ; nSize

.text:004013E7 68 C0 BB 40 00                          push    offset unk_40BBC0 ; lpBuffer

.text:004013EC 52                                      push    edx             ; lpBaseAddress

.text:004013ED 53                                      push    ebx             ; hProcess

.text:004013EE FF D5                                   call    ebp ; ReadProcessMemory ; 读取内存获取密码

.text:004013F0 56                                      push    esi

.text:004013F1 68 C0 BB 40 00                          push    offset unk_40BBC0

.text:004013F6 FF 54 24 34                             call    [esp+2D8h+var_2A4] ; 解密密码

.text:004013FA 68 C0 BB 40 00                          push    offset unk_40BBC0

.text:004013FF 68 F8 90 40 00                          push    offset aIS      ; "密码: %S\n\n"

.text:00401404 E8 34 04 00 00                          call    sub_40183D      ; 显示密码

.text:00401409 8B 7C 24 40                             mov     edi, [esp+2D8h+var_298]

.text:0040140D 83 C4 08                                add     esp, 8

.text:00401410

.text:00401410                         loc_401410:                             ; CODE XREF: _main+1EBj

.text:00401410                                                                 ; _main+265j

.text:00401410 8B 44 24 10                             mov     eax, [esp+2D0h+var_2C0]

.text:00401414 8B 4C 24 24                             mov     ecx, [esp+2D0h+var_2AC]

.text:00401418 40                                      inc     eax

.text:00401419 3B C1                                   cmp     eax, ecx        ; 检测是否已经查找完所枚举出来的LUID

.text:0040141B 89 44 24 10                             mov     [esp+2D0h+var_2C0], eax

.text:0040141F 0F 82 8A FE FF FF                       jb      loc_4012AF

.text:00401425 8B 74 24 30                             mov     esi, [esp+2D0h+var_2A0]

.text:00401429 8B 6C 24 34                             mov     ebp, [esp+2D0h+var_29C]

.text:0040142D

.text:0040142D                         loc_40142D:                             ; CODE XREF: _main+153j

.text:0040142D 53                                      push    ebx             ; hObject

.text:0040142E FF 15 38 80 40 00                       call    ds:CloseHandle

.text:00401434 8B 4C 24 20                             mov     ecx, [esp+2D0h+var_2B0]

.text:00401438 51                                      push    ecx

.text:00401439 FF 54 24 20                             call    [esp+2D4h+var_2B4]

.text:0040143D 8B 3D 18 80 40 00                       mov     edi, ds:FreeLibrary

.text:00401443 55                                      push    ebp             ; hLibModule

.text:00401444 FF D7                                   call    edi ; FreeLibrary

.text:00401446 56                                      push    esi             ; hLibModule

.text:00401447 FF D7                                   call    edi ; FreeLibrary

.text:00401449 83 7C 24 14 01                          cmp     [esp+2D4h+var_2C0], 1

.text:0040144E 75 1A                                   jnz     short loc_40146A

.text:00401450 8B 35 14 80 40 00                       mov     esi, ds:GetModuleHandleA

.text:00401456 68 EC 90 40 00                          push    offset aBcrypt_dll ; "bcrypt.dll"

.text:0040145B FF D6                                   call    esi ; GetModuleHandleA

.text:0040145D 50                                      push    eax             ; hLibModule

.text:0040145E FF D7                                   call    edi ; FreeLibrary

.text:00401460 68 D4 90 40 00                          push    offset aBcryptprimitiv ; "bcryptprimitives.dll"

.text:00401465 FF D6                                   call    esi ; GetModuleHandleA

.text:00401467 50                                      push    eax             ; hLibModule

.text:00401468 FF D7                                   call    edi ; FreeLibrary

.text:0040146A

.text:0040146A                         loc_40146A:                             ; CODE XREF: _main+2FEj

.text:0040146A 68 A4 90 40 00                          push    offset aEchoPressAny_0 ; "echo Press any Key to EXIT ... & pause "...

.text:0040146F E8 CC 02 00 00                          call    _system

.text:00401474 83 C4 04                                add     esp, 4

.text:00401477 33 C0                                   xor     eax, eax
```
