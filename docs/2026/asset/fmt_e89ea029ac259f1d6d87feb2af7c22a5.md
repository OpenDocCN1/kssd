
```
004041CC    . |8D5424 14          lea edx, dword ptr ss:[esp+14]

004041D0    . |8D4424 3C          lea eax, dword ptr ss:[esp+3C]

004041D4    . |52                 push edx                               ; /pBufSize

004041D5    . |8B5424 14          mov edx, dword ptr ss:[esp+14]         ; |

004041D9    . |8D4C24 2C          lea ecx, dword ptr ss:[esp+2C]         ; |

004041DD    . |50                 push eax                               ; |Buffer

004041DE    . |51                 push ecx                               ; |pValueType

004041DF    . |6A 00              push 0                                 ; |Reserved = NULL

004041E1    . |68 80A24400        push CHMUnpac.0044A280                 ; |ValueName = "Version"

004041E6    . |52                 push edx                               ; |hKey

004041E7    . |C74424 2C FF000000 mov dword ptr ss:[esp+2C], 0FF         ; |

004041EF    . |FF15 04B04300      call dword ptr ds:[<&ADVAPI32.RegQuery>; \RegQueryValueExA

004041F5    . |85C0               test eax, eax

004041F7    . |75 3D              jnz short CHMUnpac.00404236

004041F9    . |8B4424 10          mov eax, dword ptr ss:[esp+10]

004041FD    . |50                 push eax                               ; /hKey

004041FE    . |FF15 30B04300      call dword ptr ds:[<&ADVAPI32.RegClose>; \RegCloseKey

00404204    . |33DB               xor ebx, ebx

00404206    .  8D7C24 3C          lea edi, dword ptr ss:[esp+3C]                                   ；edi指向试练码

0040420A    >  8A4F 01            mov cl, byte ptr ds:[edi+1]                                     ；获取后一字符

0040420D    .  51                 push ecx                                                         ；F7跟进

0040420E    .  E8 1DFBFFFF        call CHMUnpac.00403D30

00404213    .  8A17               mov dl, byte ptr ds:[edi]                                     ；获取前一字符

00404215    .  52                 push edx

00404216    .  0FBEE8             movsx ebp, al                                                 ；刚转换后的字符存入ebp

00404219    .  E8 12FBFFFF        call CHMUnpac.00403D30

0040421E    .  0FBEC0             movsx eax, al                                                   ；扩展存入eax

00404221    .  03E8               add ebp, eax                                                      ;ebp=ebp+eax

00404223    .  83C4 08            add esp, 8                                                         ;esp=esp+8

00404226    .  83FD 3D            cmp ebp, 3D                                                        ;ebp=3d吗

00404229    .  75 0B              jnz short CHMUnpac.00404236                          ；不等，就OVER了，改，让它等于0x3D                                                                                              注册码改为s7t6u5v4w3x2y1z0

0040422C    .  83C7 02            add edi, 2

0040422F    .  83FB 08            cmp ebx, 8

00404232    .^ 7C D6              jl short CHMUnpac.0040420A

00404234    .  EB 5C              jmp short CHMUnpac.00404292

00404236    >  8D4C24 10          lea ecx, dword ptr ss:[esp+10]

0040423A    .  51                 push ecx

0040423B    .  8BCE               mov ecx, esi

0040423D    .  E8 EE060000        call CHMUnpac.00404930

00404242    .  68 38A44400        push CHMUnpac.0044A438

00404247    .  8D5424 18          lea edx, dword ptr ss:[esp+18]

0040424B    .  50                 push eax

0040424C    .  52                 push edx

0040424D    .  C68424 50010000 04 mov byte ptr ss:[esp+150], 4

00404255    .  E8 DBB80100        call CHMUnpac.0041FB35

0040425A    .  8B00               mov eax, dword ptr ds:[eax]

0040425C    .  8D8E AC000000      lea ecx, dword ptr ds:[esi+AC]

00404262    .  50                 push eax

00404263    .  C68424 48010000 05 mov byte ptr ss:[esp+148], 5

0040426B    .  E8 B9B70100        call CHMUnpac.0041FA29

00404270    .  8D4C24 14          lea ecx, dword ptr ss:[esp+14]

00404274    .  C68424 44010000 04 mov byte ptr ss:[esp+144], 4

0040427C    .  E8 1FB60100        call CHMUnpac.0041F8A0

00404281    .  8D4C24 10          lea ecx, dword ptr ss:[esp+10]

00404285    .  C68424 44010000 01 mov byte ptr ss:[esp+144], 1

0040428D    .  E8 0EB60100        call CHMUnpac.0041F8A0

00404292    >  8D4424 10          lea eax, dword ptr ss:[esp+10]

00404296    .  50                 push eax                               ; /pHandle

00404297    .  68 88A24400        push CHMUnpac.0044A288                 ; |Subkey = "SOFTWARE\YBSoft\CHMUnpacker"

0040429C    .  68 02000080        push 80000002                          ; |hKey = HKEY\_LOCAL\_MACHINE

004042A1    .  FF15 08B04300      call dword ptr ds:[<&ADVAPI32.RegOpenK>; \RegOpenKeyA

004042A7    .  85C0               test eax, eax

004042A9    .  0F85 41010000      jnz CHMUnpac.004043F0

004042AF    .  8D4C24 14          lea ecx, dword ptr ss:[esp+14]

004042B3    .  8D5424 3C          lea edx, dword ptr ss:[esp+3C]                                                          ;edx指向试练码

004042B7    .  51                 push ecx                               ; /pBufSize

004042B8    .  8B4C24 14          mov ecx, dword ptr ss:[esp+14]         ; |

004042BC    .  8D4424 2C          lea eax, dword ptr ss:[esp+2C]         ; |

004042C0    .  52                 push edx                               ; |Buffer

004042C1    .  50                 push eax                               ; |pValueType

004042C2    .  6A 00              push 0                                 ; |Reserved = NULL

004042C4    .  68 80A24400        push CHMUnpac.0044A280                 ; |ValueName = "Version"

004042C9    .  51                 push ecx                               ; |hKey

004042CA    .  C74424 2C FF000000 mov dword ptr ss:[esp+2C], 0FF         ; |

004042D2    .  FF15 04B04300      call dword ptr ds:[<&ADVAPI32.RegQuery>; \RegQueryValueExA

004042D8    .  85C0               test eax, eax

004042DA    .  0F85 10010000      jnz CHMUnpac.004043F0

004042E0    .  8B5424 10          mov edx, dword ptr ss:[esp+10]

004042E4    .  52                 push edx                               ; /hKey

004042E5    .  FF15 30B04300      call dword ptr ds:[<&ADVAPI32.RegClose>; \RegCloseKey

004042EB    .  33ED               xor ebp, ebp

004042ED    .  8D7C24 3D          lea edi, dword ptr ss:[esp+3D]                                      ；edi指向试练码

004042F1    >  8A47 FF            mov al, byte ptr ds:[edi-1]                                         ；eax取第一字符

004042F4    .  3C 61              cmp al, 61                                                           ；eax<61码

004042F6    .  7C 04              jl short CHMUnpac.004042FC                                           ；小于则跳

004042F8    .  2C 3D              sub al, 3D                                                           ；al=al-3d

004042FA    .  EB 0A              jmp short CHMUnpac.00404306

004042FC    >  3C 41              cmp al, 41                                                           ;al<41吗

004042FE    .  7C 04              jl short CHMUnpac.00404304

00404300    .  2C 37              sub al, 37                                                            ；小于则跳

00404302    .  EB 02              jmp short CHMUnpac.00404306

00404304    >  2C 30              sub al, 30                                                             ；al=al-30

00404306    >  8AD8               mov bl, al                                                              ;bl=al

00404308    .  8A07               mov al, byte ptr ds:[edi]                                               ;al取下一位字符

0040430A    .  50                 push eax

0040430B    .  E8 20FAFFFF        call CHMUnpac.00403D30                                       ；进入这个call进行字符变换

00404310    .  0FBEC8             movsx ecx, al                                                 ；变换后的字符扩展存入ecx

00404313    .  0FBED3             movsx edx, bl                                                   ;bl扩展存入edx
```
