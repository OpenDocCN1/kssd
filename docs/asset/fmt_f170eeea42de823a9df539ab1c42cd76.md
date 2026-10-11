
### Assembly Code Snippet

```asm
003B44F2  68 C8000000     push    0C8  
003B44F7  50             push    eax    ; buffer  
003B44F8  FF95 BBCE4100  call    dword ptr ss:[ebp+41CEBB]             ; kernel32.GetWindowsDirectoryA  
                                0012FF94   003B44FE  /CALL to GetWindowsDirectoryA from 003B44F8  
                                0012FF98   003B47F4  |Buffer = 003B47F4  
                                0012FF9C   000000C8  \BufSize = C8 (200.)  
003B4503  33C0           xor     eax, eax  
003B451C  B9 C8000000    mov     ecx, 0C8  
003B4526  8DBD C1A04100  lea     edi, dword ptr ss:[ebp+41A0C1]  
003B4531  F2:AE          repne   scas byte ptr es:[edi]   ; 复制目录  
003B4538  C647 FF 5C    mov     byte ptr ds:[edi-1], 5C   ; \  
003B4541  C707 75736572 mov     dword ptr ds:[edi], 72657375   ; user  
003B455E  C747 04 2E64617>mov  dword ptr ds:[edi+4], 7461642E   ; .dat  
003B4592  C647 08 00    mov     byte ptr ds:[edi+8], 0  
003B45C4  8D85 C1A04100 lea     eax, dword ptr ss:[ebp+41A0C1]  
003B45CF  8D9D C1A14100 lea     ebx, dword ptr ss:[ebp+41A1C1]  
003B45DA  53             push    ebx  
003B45DB  50             push    eax  
003B45DC  FF95 B7CE4100 call    dword ptr ss:[ebp+41CEB7]             ; kernel32.FindFirstFileA  
003B45F9  8D9D C1A14100 lea     ebx, dword ptr ss:[ebp+41A1C1]  
003B462C  8D43 14        lea     eax, dword ptr ds:[ebx+14]  
003B4646  8D9D FCA24100 lea     ebx, dword ptr ss:[ebp+41A2FC]  
003B4679  53             push    ebx  
003B467A  50             push    eax  
003B467B  FF95 B3CE4100 call    dword ptr ss:[ebp+41CEB3]             ; kernel32.FileTimeToSystemTime  
                                0012FF94   003B4681  /CALL to `FileTimeToSystemTime` from 003B467B  
                                0012FF98   003B4908  |pFileTime = 003B4908  
                                0012FF9C   003B4A2F  \pSystemTime = 003B4A2F  
003B469D  8D9D FCA24100 lea     ebx, dword ptr ss:[ebp+41A2FC]  
; 计算一个 sum  
003B46A8  33C0           xor     eax, eax  
003B46AF  33D2           xor     edx, edx  
003B46CD  66:8B03        mov     ax, word ptr ds:[ebx]  
003B46E7  B9 6D010000    mov     ecx, 16D  
003B46F1  F7E1           mul     ecx  
003B470A  8985 80D24100 mov     dword ptr ss:[ebp+41D280], eax  
003B4742  33C0           xor     eax, eax  
003B475B  66:8B43 02     mov     ax, word ptr ds:[ebx+2]  
003B4764  B9 1E000000    mov     ecx, 1E  
003B476E  F7E1           mul     ecx  
003B4775  0185 80D24100 add     dword ptr ss:[ebp+41D280], eax  
003B47AD  66:8B43 06     mov     ax, word ptr ds:[ebx+6]  
003B47B6  0185 80D24100 add     dword ptr ss:[ebp+41D280], eax  
003B47EE  C3             ret      ; 班师回朝  
003B380E  C785 88D24100 0>mov  dword ptr ss:[ebp+41D288], 0  
003B381D  C785 84D24100 0>mov  dword ptr ss:[ebp+41D284], 0  
003B3836  83BD F01D4200 0>cmp  dword ptr ss:[ebp+421DF0], 1  
003B383D  /0F85 10010000 jnz     003B3953   ; nj  
003B3848  FFB5 FECF4100 push    dword ptr ss:[ebp+41CFFE]  
003B384E  8F85 42344200 pop     dword ptr ss:[ebp+423442]  
003B3859  FFB5 E41D4200 push    dword ptr ss:[ebp+421DE4]  
003B385F  8F85 46344200 pop     dword ptr ss:[ebp+423446]  
003B387C  8B9D E41D4200 mov     ebx, dword ptr ss:[ebp+421DE4]  
003B3899  399D FECF4100 cmp     dword ptr ss:[ebp+41CFFE], ebx  
003B389F  /0F83 95000000 jnb     003B393A   ; j  
003B393F  C785 84D24100 0>mov  dword ptr ss:[ebp+41D284], 1  
003B3974  83BD F01D4200 0>cmp  dword ptr ss:[ebp+421DF0], 2  
003B397B  /0F85 E0000000 jnz     003B3A61   ; j  
003B3A98  83BD F01D4200 0>cmp  dword ptr ss:[ebp+421DF0], 3  
003B3A9F  0F85 C3000000 jnz     003B3B68  
003B3B9F  33C0           xor     eax, eax  
003B3BA6  83BD 88D24100 0>cmp  dword ptr ss:[ebp+41D288], 1  
003B3BAD  74 0D          je      short 003B3BBC  
003B3BAF  83BD 84D24100 0>cmp  dword ptr ss:[ebp+41D284], 1  
003B3BB6  0F85 F5000000 jnz     003B3CB1  
003B3BD3  8D85 081E4200 lea     eax, dword ptr ss:[ebp+421E08] ; caption  
003B3BDE  8D9D 261E4200 lea     ebx, dword ptr ss:[ebp+421E26] ; http...  
003B3BE9  8D8D 441E4200 lea     ecx, dword ptr ss:[ebp+421E44] ; message  
; 此时发现这两个牛插手了……  
----------------------------------------------------------------------------  
003BC777              20 45 6E 63 72 79 70 72   Encrypt  
003BC787  65 64 20 62 79 20 7A 65 72 30 21 00 00 00 00  ed by zer0!.....  
003BC797  00 00 00 00 00 00 63 6F 64 65 5F 69 6E 6A 65  ......code_injec  
003BC7A7  74 00 00 00 00 00 00 00 00                   t........  
----------------------------------------------------------------------------  
003B3C1C  89A5 74894100 mov     dword ptr ss:[ebp+418974], esp  
003B3C27  FFB5 81C84100 push    dword ptr ss:[ebp+41C881]  
003B3C2D  8F85 F9344200 pop     dword ptr ss:[ebp+4234F9]  
003B3C38  51             push    ecx  
003B3C39  53             push    ebx  
003B3C3A  50             push    eax  
003B3C3B  FFB5 46344200 push    dword ptr ss:[ebp+423446]  
003B3C41  FFB5 42344200 push    dword ptr ss:[ebp+423442]  
003B3C47  E8 39810000    call    003BBD85   ; int3 调用，都 nop  
003B3C7E  8BA5 74894100 mov     esp, dword ptr ss:[ebp+418974]  
003B3D22  83BD 84D24100 0>cmp  dword ptr ss:[ebp+41D284], 1  
003B3D29  74 09          je      short 003B3D34  
003B3D2B  83F8 02        cmp     eax, 2  
003B3D2E  0F85 C0040000 jnz     003B41F4  
003B3D66  8D85 FD344200 lea     eax, dword ptr ss:[ebp+4234FD]  
003B3D99  8D9D 13354200 lea     ebx, dword ptr ss:[ebp+423513]  
003B3DA4  8D8D 27354200 lea     ecx, dword ptr ss:[ebp+423527]  
; 003BDC5A  30 36 37 38 31 46 38 42                      06781F8B  
003B3DAF  8D95 38204200 lea     edx, dword ptr ss:[ebp+422038]  
003B3DF9  89A5 74894100 mov     dword ptr ss:[ebp+418974], esp  
003B3E2C  FFB5 85C84100 push    dword ptr ss:[ebp+41C885]  
003B3E32  8F85 F9344200 pop     dword ptr ss:[ebp+4234F9]  
003B3E4F  52             push    edx  
003B3E50  51             push    ecx  
003B3E51  53             push    ebx  
003B3E52  50             push    eax  
003B3E53  E8 2D7F0000    call    003BBD85; 还是他，nop  
003B3E74  8BA5 74894100 mov     esp, dword ptr ss:[ebp+418974]  
003B3E91  83F8 02        cmp     eax, 2  
```
