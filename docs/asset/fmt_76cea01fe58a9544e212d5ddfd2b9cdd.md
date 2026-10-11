
```
00646FEB   |.  BE 7CA47F00     mov esi,PDFEdit.007FA47C                         ;  ASCII "WAC Server"  

00646FF0   |.  EB 67           jmp short PDFEdit.00647059  

00646FF2   |>  68 78A47F00     push PDFEdit.007FA478                            ; /Arg2 = 007FA478 ASCII "PPO"  

00646FF7   |.  55             push ebp                                         ; |Arg1  

00646FF8   |.  E8 2A20E9FF    call PDFEdit.004D9027                            ; \\PDFEdit.004D9027  

00646FFD   |.  83C4 08        add esp,8  

00647000   |.  85C0           test eax,eax  

00647002   |.  75 07          jnz short PDFEdit.0064700B  

00647004   |.  BE 64A47F00    mov esi,PDFEdit.007FA464                         ;  ASCII "PDF Page Organizer"  

00647009   |.  EB 4E          jmp short PDFEdit.00647059  

0064700B   |>  68 60A47F00    push PDFEdit.007FA460                            ; /Arg2 = 007FA460 ASCII "PPP"  

00647010   |.  55             push ebp                                         ; |Arg1  

00647011   |.  E8 1120E9FF    call PDFEdit.004D9027                            ; \\PDFEdit.004D9027  

00647016   |.  83C4 08        add esp,8  

00647019   |.  85C0           test eax,eax  

0064701B   |.  75 07          jnz short PDFEdit.00647024  

0064701D   |.  BE 48A47F00    mov esi,PDFEdit.007FA448                         ;  ASCII "PDF Page Organizer Pro"  

00647022   |.  EB 35          jmp short PDFEdit.00647059  

00647024   |>  68 D0267D00    push PDFEdit.007D26D0                            ; /Arg2 = 007D26D0 ASCII "PED"  

00647029   |.  55             push ebp                                         ; |Arg1  

0064702A   |.  E8 F81FE9FF    call PDFEdit.004D9027                            ; \\PDFEdit.004D9027  

0064702F   |.  83C4 08        add esp,8  

00647032   |.  85C0           test eax,eax  

00647034   |.  75 07          jnz short PDFEdit.0064703D  

00647036   |.  BE 00277D00    mov esi,PDFEdit.007D2700                         ;  ASCII "PDF Editor"  

0064703B   |.  EB 1C          jmp short PDFEdit.00647059  

0064703D   |>  68 44A47F00    push PDFEdit.007FA444                            ; /Arg2 = 007FA444 ASCII "FRP"  

00647042   |.  55             push ebp                                         ; |Arg1  

00647043   |.  E8 DF1FE9FF    call PDFEdit.004D9027                            ; \\PDFEdit.004D9027  

00647048   |.  83C4 08        add esp,8  

0064704B   |.  BE 30A47F00    mov esi,PDFEdit.007FA430                         ;  ASCII "Foxit Reader Pro"  

00647050   |.  85C0           test eax,eax  

00647052   |.  74 05          je short PDFEdit.00647059  

00647054   |.  BE 20A47F00    mov esi,PDFEdit.007FA420                         ;  ASCII "Foxit License"  

00647059   |>  8BBC24 0C140000 mov edi,dword ptr ss:\[esp+140C\]  

00647060   |.  8B1D 54BD8100  mov ebx,dword ptr ds:\[<&KERNEL32.GetPrivateProfileSt>;  kernel32.GetPrivateProfileStringA  

00647066   |.  57             push edi                                         ; /IniFileName  

00647067   |.  8D8424 0C010000 lea eax,dword ptr ss:\[esp+10C\]                 ; |  

0064706E   |.  68 00010000    push 100                                         ; |BufSize = 100 (256.)  

00647073   |.  50             push eax                                         ; |ReturnBuffer  

00647074   |.  68 18258000    push PDFEdit.00802518                            ; |Default = ""  

00647079   |.  68 1CA47F00    push PDFEdit.007FA41C                            ; |Key = "SN"  

0064707E   |.  56             push esi                                         ; |Section  

0064707F   |.  FFD3           call ebx                                         ; \\GetPrivateProfileStringA  

00647081   |.  8A8424 08010000 mov al,byte ptr ss:\[esp+108\]  

00647088   |.  84C0           test al,al  

0064708A   |.  75 20          jnz short PDFEdit.006470AC  

0064708C   |.  57             push edi                                         ; /IniFileName  

0064708D   |.  8D8C24 0C010000 lea ecx,dword ptr ss:\[esp+10C\]                 ; |  

00647094   |.  68 00010000    push 100                                         ; |BufSize = 100 (256.)  

00647099   |.  51             push ecx                                         ; |ReturnBuffer  

0064709A   |.  BE 20A47F00    mov esi,PDFEdit.007FA420                         ; |ASCII "Foxit License"  

0064709F   |.  68 18258000    push PDFEdit.00802518                            ; |Default = ""  

006470A4   |.  68 1CA47F00    push PDFEdit.007FA41C                            ; |Key = "SN"  

006470A9   |.  56             push esi                                         ; |Section => "Foxit License"  

006470AA   |.  FFD3           call ebx                                         ; \\GetPrivateProfileStringA  

006470AC   |>  57             push edi  

006470AD   |.  8D9424 0C100000 lea edx,dword ptr ss:\[esp+100C\]  

006470B4   |.  68 00040000    push 400  

006470B9   |.  52             push edx  

006470BA   |.  68 18258000    push PDFEdit.00802518  

006470BF   |.  68 14A47F00    push PDFEdit.007FA414                            ;  ASCII "Modules"  

006470C4   |.  56             push esi  

006470C5   |.  FFD3           call ebx  ; \\GetPrivateProfileStringA  

006470C7   |.  57             push edi  

006470C8   |.  8D8424 0C060000 lea eax,dword ptr ss:\[esp+60C\]  

006470CF   |.  68 00010000    push 100  

006470D4   |.  50             push eax  

006470D5   |.  68 18258000    push PDFEdit.00802518  

006470DA   |.  68 0CA47F00    push PDFEdit.007FA40C                            ;  ASCII "Users"  

006470DF   |.  56             push esi  

006470E0   |.  FFD3           call ebx  ; \\GetPrivateProfileStringA  

006470E2   |.  57             push edi  

006470E3   |.  8D8C24 0C050000 lea ecx,dword ptr ss:\[esp+50C\]  

006470EA   |.  68 00010000    push 100  

006470EF   |.  51             push ecx  

006470F0   |.  68 18258000    push PDFEdit.00802518  

006470F5   |.  68 00A47F00    push PDFEdit.007FA400                            ;  ASCII "Licensee"  

006470FA   |.  56             push esi  

006470FB   |.  FFD3           call ebx  ; \\GetPrivateProfileStringA  

006470FD   |.  57             push edi  

006470FE   |.  8D9424 0C030000 lea edx,dword ptr ss:\[esp+30C\]  

00647105   |.  68 00010000    push 100  

0064710A   |.  52             push edx  

0064710B   |.  68 18258000    push PDFEdit.00802518  

00647110   |.  68 F4A37F00    push PDFEdit.007FA3F4                            ;  ASCII "ExpireDate"  

00647115   |.  56             push esi  

00647116   |.  FFD3           call ebx  ; \\GetPrivateProfileStringA  

00647118   |.  57             push edi  

00647119   |.  8D8424 0C070000 lea eax,dword ptr ss:\[esp+70C\]  

00647120   |.  68 00010000    push 100  

00647125   |.  50             push eax  

00647126   |.  68 18258000    push PDFEdit.00802518  

0064712B   |.  68 ECA37F00    push PDFEdit.007FA3EC                            ;  ASCII "Sign"  

00647130   |.  56             push esi  

00647131   |.  FFD3           call ebx  ; \\GetPrivateProfileStringA  

00647133   |.  57             push edi  

00647134   |.  8D8C24 0C040000 lea ecx,dword ptr ss:\[esp+40C\]  

0064713B   |.  68 00010000    push 100  

00647140   |.  51             push ecx  

00647141   |.  68 18258000    push PDFEdit.00802518  

00647146   |.  68 E0A37F00    push PDFEdit.007FA3E0                            ;  ASCII "LicenseDate"  

0064714B   |.  56             push esi  

0064714C   |.  FFD3           call ebx  ; \\GetPrivateProfileStringA  

0064714E   |.  57             push edi  

0064714F   |.  8D9424 0C020000 lea edx,dword ptr ss:\[esp+20C\]  

00647156   |.  68 00040000    push 400
```
