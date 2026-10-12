
```
00584489    8D95 50FFFFFF         lea edx,dword ptr ss:[ebp-B0]  

0058448F    E8 844CE8FF          call dumped_.00409118  

00584494    8B85 50FFFFFF        mov eax,dword ptr ss:[ebp-B0]  

0058449A    8A10                mov dl,byte ptr ds:[eax]  

0058449C    8B45 FC            mov eax,dword ptr ss:[ebp-4]  

0058449F    05 80000000         add eax,80  

005844A4    E8 7303E8FF         call dumped_.0040481C  

005844A9    8B45 FC            mov eax,dword ptr ss:[ebp-4]  

005844AC    8B80 80000000       mov eax,dword ptr ds:[eax+80]  

005844B2    E8 014DE8FF         call dumped_.004091B8  

005844B7    8BD8                mov ebx,eax  

005844B9    8B45 FC            mov eax,dword ptr ss:[ebp-4]  

005844BC    8958 0C            mov dword ptr ds:[eax+C],ebx  

005844BF    D1FB                sar ebx,1  

005844C1    79 03              jns short dumped_.005844C6  

005844C3    83D3 00            adc ebx,0  

005844C6    85DB                test ebx,ebx  

005844C8    75 12              jnz short dumped_.005844DC  

005844CA    8B45 FC            mov eax,dword ptr ss:[ebp-4]  

005844CD    8B80 7C060000      mov eax,dword ptr ds:[eax+67C]  

005844D3    B2 01              mov dl,1  

005844D5    E8 4EF2EBFF        call dumped_.00443728  

005844DA    EB 0C              jmp short dumped_.005844E8  

005844DC    E8 E7FFE7FF        call dumped_.004044C8  

005844E1    EB 05              jmp short dumped_.005844E8  

005844E3    E8 E0FFE7FF        call dumped_.004044C8  

005844E8    33C0                xor eax,eax                            ; 异或，eax=001D5200  

005844EA    5A                  pop edx  

005844EB    59                  pop ecx  

005844EC    59                  pop ecx  

005844ED    64:8910             mov dword ptr fs:[eax],edx  

005844F0    68 A8455800         push dumped_.005845A8  

005844F5    8B45 FC            mov eax,dword ptr ss:[ebp-4]  

005844F8    8B80 9C030000      mov eax,dword ptr ds:[eax+39C]  

005844FE    33D2                xor edx,edx  

00584500    E8 57F8EFFF        call dumped_.00483D5C  

00584505    8B45 FC            mov eax,dword ptr ss:[ebp-4]  

00584508    8B80 4C030000      mov eax,dword ptr ds:[eax+34C]  

0058450E    B2 05              mov dl,5  

00584510    E8 B791ECFF        call dumped_.0044D6CC  

00584515    8B45 FC            mov eax,dword ptr ss:[ebp-4]  

00584518    8B80 28060000      mov eax,dword ptr ds:[eax+628]  

0058451E    33D2                xor edx,edx  

00584520    E8 DB9BECFF        call dumped_.0044E100  

00584525    8B45 FC            mov eax,dword ptr ss:[ebp-4]  

00584528    8B80 5C060000      mov eax,dword ptr ds:[eax+65C]  

0058452E    33D2                xor edx,edx  

00584530    E8 CB9BECFF        call dumped_.0044E100  

00584535    8B45 FC            mov eax,dword ptr ss:[ebp-4]  

00584538    8B80 60030000      mov eax,dword ptr ds:[eax+360]  

0058453E    33D2                xor edx,edx  

00584540    E8 BB9BECFF        call dumped_.0044E100  

00584545    8B45 FC            mov eax,dword ptr ss:[ebp-4]  

00584548    8B80 68030000      mov eax,dword ptr ds:[eax+368]  

0058454E    33D2                xor edx,edx  

00584550    E8 AB9BECFF        call dumped_.0044E100  

00584555    8B45 FC            mov eax,dword ptr ss:[ebp-4]  

00584558    8B80 6C030000      mov eax,dword ptr ds:[eax+36C]  

0058455E    33D2                xor edx,edx  

00584560    E8 9B9BECFF        call dumped_.0044E100  

00584565    8B45 FC            mov eax,dword ptr ss:[ebp-4]  

00584568    8B80 70030000      mov eax,dword ptr ds:[eax+370]  

0058456E    33D2                xor edx,edx  

00584570    E8 8B9BECFF        call dumped_.0044E100  

00584575    8B45 FC            mov eax,dword ptr ss:[ebp-4]  

00584578    8B80 0C060000      mov eax,dword ptr ds:[eax+60C]  

0058457E    33D2                xor edx,edx  

00584580    E8 7B9BECFF        call dumped_.0044E100  

00584585    8B45 FC            mov eax,dword ptr ss:[ebp-4]  

00584588    8B80 4C030000      mov eax,dword ptr ds:[eax+34C]  

0058458E    33D2                xor edx,edx  

00584590    E8 6B9BECFF        call dumped_.0044E100  

00584595    8B45 F8            mov eax,dword ptr ss:[ebp-8]  

00584598    E8 AFF2E7FF        call dumped_.0040384C  

0058459D    C3                  retn  

0058459E  ^\\E9 3DFAE7FF        jmp dumped_.00403FE0  

005845A3  ^ E9 4DFFFFFF         jmp dumped_.005844F5  

005845A8    33C0                xor eax,eax                            ; 异或，eax=00000000  

005845AA    5A                  pop edx  

005845AB    59                  pop ecx                                ; ecx=00000000  

005845AC    59                  pop ecx                                ; ecx=00584660  

005845AD    64:8910             mov dword ptr fs:[eax],edx  

005845B0    68 6A465800         push dumped_.0058466A  

005845B5    8D85 50FFFFFF       lea eax,dword ptr ss:[ebp-B0]  

005845BB    BA 05000000         mov edx,5  

005845C0    E8 9B00E8FF         call dumped_.00404660  

005845C5    8D85 70FFFFFF       lea eax,dword ptr ss:[ebp-90]  

005845CB    BA 03000000         mov edx,3  

005845D0    E8 8B00E8FF         call dumped_.00404660  

005845D5    8D85 7CFFFFFF       lea eax,dword ptr ss:[ebp-84]  

005845DB    E8 5C00E8FF         call dumped_.0040463C  

005845E0    8D45 80             lea eax,dword ptr ss:[ebp-80]  

005845E3    E8 8007E8FF         call dumped_.00404D68  

005845E8    8D45 84             lea eax,dword ptr ss:[ebp-7C]  

005845EB    E8 4C00E8FF         call dumped_.0040463C  

005845F0    8D45 88             lea eax,dword ptr ss:[ebp-78]  

005845F3    E8 7007E8FF         call dumped_.00404D68  

005845F8    8D45 8C             lea eax,dword ptr ss:[ebp-74]  

005845FB    E8 3C00E8FF         call dumped_.0040463C  

00584600    8D45 90             lea eax,dword ptr ss:[ebp-70]  

00584603    E8 6007E8FF         call dumped_.00404D68  

00584608    8D45 94             lea eax,dword ptr ss:[ebp-6C]  

0058460B    E8 2C00E8FF         call dumped_.0040463C  

00584610    8D45 98             lea eax,dword ptr ss:[ebp-68]  

00584613    E8 5007E8FF         call dumped_.00404D68  

00584618    8D45 9C             lea eax,dword ptr ss:[ebp-64]  

0058461B    E8 1C00E8FF         call dumped_.0040463C  

00584620    8D45 A0             lea eax,dword ptr ss:[ebp-60]  

00584623    E8 4007E8FF         call dumped_.00404D68  

00584628    8D45 A4             lea eax,dword ptr ss:[ebp-5C]  

0058462B    BA 06000000         mov edx,6  

00584630    E8 2B00E8FF         call dumped_.00404660  

00584635    8D45 BC             lea eax,dword ptr ss:[ebp-44]  

00584638    BA 07000000         mov edx,7  

0058463D    E8 1E00E8FF         call dumped_.00404660  

00584642    8D45 D8             lea eax,dword ptr ss:[ebp-28]  

00584645    E8 F2FFE7FF         call dumped_.0040463C  

0058464A    8D45 E0             lea eax,dword ptr ss:[ebp-20]  

0058464D    BA 03000000         mov edx,3  

00584652    E8 0900E8FF         call dumped_.00404660  

00584657    8D45 F4             lea eax,dword ptr ss:[ebp-C]  

0058465A    E8 0907E8FF         call dumped_.00404D68  

0058465F    C3                  retn  

00584660  ^\\E9 7BF9E7FF        jmp dumped_.00403FE0  

00584665  ^ E9 4BFFFFFF         jmp dumped_.005845B5  

0058466A    5F                  pop edi                                ; edi=00470850  

0058466B    5E                  pop esi                                ; esi=0043E118  
```
