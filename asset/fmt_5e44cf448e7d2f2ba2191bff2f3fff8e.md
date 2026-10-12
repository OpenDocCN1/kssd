
```
00407A10       .  8B3D 88014200       mov edi,dword ptr ds:[<&KERNEL32.GetModuleHandleA>]      ; |kernel32.GetModuleHandleA  
00407A16       .  FFD7               call near edi                                           ; \GetModuleHandleA  
00407A18       .  66:8138 4D5A       cmp word ptr ds:[eax],5A4D  
00407A1D       .  75 1F              jnz short eMuleUpd.00407A3E  
00407A1F       .  8B48 3C            mov ecx,dword ptr ds:[eax+3C]  
00407A22       .  03C8               add ecx,eax  
00407A24       .  8139 50450000      cmp dword ptr ds:[ecx],4550  
00407A2A       .  75 12              jnz short eMuleUpd.00407A3E  
00407A2C       .  0FB741 18          movzx eax,word ptr ds:[ecx+18]  
00407A30       .  3D 0B010000        cmp eax,10B  
00407A35       .  74 1F              je short eMuleUpd.00407A56  
00407A37       .  3D 0B020000        cmp eax,20B  
00407A3C       .  74 05              je short eMuleUpd.00407A43  
00407A3E       >  8975 E4            mov dword ptr ss:[ebp-1C],esi  
00407A41       .  EB 27              jmp short eMuleUpd.00407A6A  
00407A43       >  83B9 84000000 0E   cmp dword ptr ds:[ecx+84],0E  
00407A4A       .^ 76 F2             jbe short eMuleUpd.00407A3E  
00407A4C       .  33C0              xor eax,eax  
00407A4E       .  39B1 F8000000     cmp dword ptr ds:[ecx+F8],esi  
00407A54       .  EB 0E             jmp short eMuleUpd.00407A64  
00407A56       >  8379 74 0E        cmp dword ptr ds:[ecx+74],0E  
00407A5A       .^ 76 E2             jbe short eMuleUpd.00407A3E  
00407A5C       .  33C0              xor eax,eax  
00407A5E       .  39B1 E8000000     cmp dword ptr ds:[ecx+E8],esi  
00407A64       >  0F95C0            setne al  
00407A67       .  8945 E4           mov dword ptr ss:[ebp-1C],eax  
00407A6A       >  6A 01             push 1  
00407A6C       .  E8 A3550000       call eMuleUpd.0040D014  
00407A71       .  59                pop ecx                                           ;  kernel32.7C816FD7  
00407A72       .  85C0              test eax,eax  
00407A74       .  75 08             jnz short eMuleUpd.00407A7E  
00407A76       .  6A 1C             push 1C  
00407A78       .  E8 02FFFFFF       call eMuleUpd.0040797F  
00407A7D       .  59                pop ecx                                           ;  kernel32.7C816FD7  
00407A7E       >  E8 572A0000       call eMuleUpd.0040A4DA  
00407A83       .  85C0              test eax,eax  
00407A85       .  75 08             jnz short eMuleUpd.00407A8F  
00407A87       .  6A 10             push 10  
00407A89       .  E8 F1FEFFFF       call eMuleUpd.0040797F  
00407A8E       .  59                pop ecx                                           ;  kernel32.7C816FD7  
00407A8F       >  E8 756A0000       call eMuleUpd.0040E509  
00407A94       .  8975 FC           mov dword ptr ss:[ebp-4],esi  
00407A97       .  E8 6F680000       call eMuleUpd.0040E30B  
00407A9C       .  85C0              test eax,eax  
00407A9E       .  7D 08             jge short eMuleUpd.00407AA8  
00407AA0       .  6A 1B             push 1B  
00407AA2       .  E8 B3FEFFFF       call eMuleUpd.0040795A  
00407AA7       .  59                pop ecx                                           ;  kernel32.7C816FD7  
00407AA8       >  FF15 0C024200     call near dword ptr ds:[<&KERNEL32.GetCommandLineA>]     ; [GetCommandLineA  
00407AAE       .  A3 10D44200       mov dword ptr ds:[42D410],eax  
00407AB3       .  E8 31670000       call eMuleUpd.0040E1E9  
00407AB8       .  A3 14BB4200       mov dword ptr ds:[42BB14],eax  
00407ABD       .  E8 85660000       call eMuleUpd.0040E147  
00407AC2       .  85C0              test eax,eax  
00407AC4       .  7D 08             jge short eMuleUpd.00407ACE  
00407AC6       .  6A 08             push 8  
00407AC8       .  E8 8DFEFFFF       call eMuleUpd.0040795A  
00407ACD       .  59                pop ecx                                           ;  kernel32.7C816FD7  
00407ACE       >  E8 41640000       call eMuleUpd.0040DF14  
00407AD3       .  85C0              test eax,eax  
00407AD5       .  7D 08             jge short eMuleUpd.00407ADF  
00407AD7       .  6A 09             push 9  
00407AD9       .  E8 7CFEFFFF       call eMuleUpd.0040795A  
00407ADE       .  59                pop ecx                                           ;  kernel32.7C816FD7  
00407ADF       >  6A 01             push 1  
00407AE1       .  E8 72030000       call eMuleUpd.00407E58  
00407AE6       .  59                pop ecx                                           ;  kernel32.7C816FD7  
00407AE7       .  8945 D8           mov dword ptr ss:[ebp-28],eax  
00407AEA       .  3BC6              cmp eax,esi  
00407AEC       .  74 07             je short eMuleUpd.00407AF5  
00407AEE       .  50                push eax  
00407AEF       .  E8 66FEFFFF       call eMuleUpd.0040795A  
00407AF4       .  59                pop ecx                                           ;  kernel32.7C816FD7  
00407AF5       >  8975 BC           mov dword ptr ss:[ebp-44],esi  
00407AF8       .  8D45 90          lea eax,dword ptr ss:[ebp-70]  
00407AFB       .  50                push eax                                          ; /pStartupinfo = NULL  
00407AFC       .  FF15 B4004200     call near dword ptr ds:[<&KERNEL32.GetStartupInfoA>]     ; \GetStartupInfoA  
00407B02       .  E8 B0630000       call eMuleUpd.0040DEB7  
00407B07       .  8945 E0          mov dword ptr ss:[ebp-20],eax  
00407B0A       .  F645 BC 01       test byte ptr ss:[ebp-44],1  
00407B0E       .  74 06             je short eMuleUpd.00407B16  
00407B10       .  0FB745 C0        movzx eax,word ptr ss:[ebp-40]  
00407B14       .  EB 03             jmp short eMuleUpd.00407B19  
00407B16       >  6A 0A             push 0A  
00407B18       .  58                pop eax                                           ;  kernel32.7C816FD7  
00407B19       >  50                push eax  
00407B1A       .  FF75 E0          push dword ptr ss:[ebp-20]  
00407B1D       .  56                push esi  
00407B1E       .  56                push esi  
00407B1F       .  FFD7              call near edi                                     ;  ntdll.7C930738  
00407B21       .  50                push eax  
00407B22       .  E8 33B50000       call eMuleUpd.0041305A  
00407B27       .  8BF8              mov edi,eax  
00407B29       .  897D D4          mov dword ptr ss:[ebp-2C],edi                       ;  ntdll.7C930738  
00407B2C       .  3975 E4          cmp dword ptr ss:[ebp-1C],esi  
00407B2F       .  75 06             jnz short eMuleUpd.00407B37  
00407B31       .  57                push edi                                          ;  ntdll.7C930738  
00407B32       .  E8 4E040000       call eMuleUpd.00407F85  
00407B37       >  E8 6B040000       call eMuleUpd.00407FA7  
00407B3C       .  EB 2B             jmp short eMuleUpd.00407B69  
00407B3E       .  8B45 EC          mov eax,dword ptr ss:[ebp-14]  
00407B41       .  8B08              mov ecx,dword ptr ds:[eax]  
00407B43       .  8B09              mov ecx,dword ptr ds:[ecx]                          ;  ntdll.7C92E64E  
00407B45       .  894D DC          mov dword ptr ss:[ebp-24],ecx  
00407B48       .  50                push eax  
00407B49       .  51                push ecx  
00407B4A       .  E8 04620000       call eMuleUpd.0040DD53
```
