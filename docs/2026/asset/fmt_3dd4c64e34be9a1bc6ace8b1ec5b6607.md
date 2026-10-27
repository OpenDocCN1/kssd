
```
0048707F   |.  8D45 F0        lea eax,dword ptr ss:[ebp-10]        ; |
00487082   |.  8B0E           mov ecx,dword ptr ds:[esi]           ; |
00487084   |.  8B55 EC        mov edx,dword ptr ss:[ebp-14]        ; |
00487087   |.  E8 A8FBFFFF    call INS.00486C34                    ; \INS.00486C34
0048708C   |.  8B06           mov eax,dword ptr ds:[esi]
0048708E   |.  50             push eax                             ; /Arg3
0048708F   |.  8B43 38        mov eax,dword ptr ds:[ebx+38]        ; |
00487092   |.  05 0C38E5FD    add eax,FDE5380C                     ; |
00487097   |.  50             push eax                             ; |Arg2
00487098   |.  6A 17          push 17                              ; |Arg1 = 00000017
0048709A   |.  8BC7           mov eax,edi                          ; |
0048709C   |.  8B4D EC        mov ecx,dword ptr ss:[ebp-14]        ; |
0048709F   |.  8B55 F0        mov edx,dword ptr ss:[ebp-10]        ; |
004870A2   |.  E8 8DFBFFFF    call INS.00486C34                    ; \INS.00486C34
004870A7   |.  8B45 EC        mov eax,dword ptr ss:[ebp-14]
004870AA   |.  50             push eax                             ; /Arg3
004870AB   |.  8B43 04        mov eax,dword ptr ds:[ebx+4]         ; |
004870AE   |.  05 44EABEA4    add eax,A4BEEA44                     ; |
004870B3   |.  50             push eax                             ; |Arg2
004870B4   |.  6A 04          push 4                               ; |Arg1 = 00000004
004870B6   |.  8BC6           mov eax,esi                          ; |
004870B8   |.  8B4D F0        mov ecx,dword ptr ss:[ebp-10]        ; |
004870BB   |.  8B17           mov edx,dword ptr ds:[edi]           ; |
004870BD   |.  E8 72FBFFFF    call INS.00486C34                    ; \INS.00486C34
004870C2   |.  8B45 F0        mov eax,dword ptr ss:[ebp-10]
004870C5   |.  50             push eax                             ; /Arg3
004870C6   |.  8B43 10        mov eax,dword ptr ds:[ebx+10]        ; |
004870C9   |.  05 A9CFDE4B    add eax,4BDECFA9                     ; |
004870CE   |.  50             push eax                             ; |Arg2
004870CF   |.  6A 0B          push 0B                              ; |Arg1 = 0000000B
004870D1   |.  8D45 EC        lea eax,dword ptr ss:[ebp-14]        ; |
004870D4   |.  8B0F           mov ecx,dword ptr ds:[edi]           ; |
004870D6   |.  8B16           mov edx,dword ptr ds:[esi]           ; |
004870D8   |.  E8 57FBFFFF    call INS.00486C34                    ; \INS.00486C34
004870DD   |.  8B07           mov eax,dword ptr ds:[edi]
004870DF   |.  50             push eax                             ; /Arg3
004870E0   |.  8B43 1C        mov eax,dword ptr ds:[ebx+1C]        ; |
004870E3   |.  05 604BBBF6    add eax,F6BB4B60                     ; |
004870E8   |.  50             push eax                             ; |Arg2
004870E9   |.  6A 10          push 10                              ; |Arg1 = 00000010
004870EB   |.  8D45 F0        lea eax,dword ptr ss:[ebp-10]        ; |
004870EE   |.  8B0E           mov ecx,dword ptr ds:[esi]           ; |
004870F0   |.  8B55 EC        mov edx,dword ptr ss:[ebp-14]        ; |
004870F3   |.  E8 3CFBFFFF    call INS.00486C34                    ; \INS.00486C34
004870F8   |.  8B06           mov eax,dword ptr ds:[esi]
004870FA   |.  50             push eax                             ; /Arg3
004870FB   |.  8B43 28        mov eax,dword ptr ds:[ebx+28]        ; |
004870FE   |.  05 70BCBFBE    add eax,BEBFBC70                     ; |
00487103   |.  50             push eax                             ; |Arg2
00487104   |.  6A 17          push 17                              ; |Arg1 = 00000017
00487106   |.  8BC7           mov eax,edi                          ; |
00487108   |.  8B4D EC        mov ecx,dword ptr ss:[ebp-14]        ; |
0048710B   |.  8B55 F0        mov edx,dword ptr ss:[ebp-10]        ; |
0048710E   |.  E8 21FBFFFF    call INS.00486C34                    ; \INS.00486C34
00487113   |.  8B45 EC        mov eax,dword ptr ss:[ebp-14]
00487116   |.  50             push eax                             ; /Arg3
00487117   |.  8B43 34        mov eax,dword ptr ds:[ebx+34]        ; |
0048711A   |.  05 C67E9B28    add eax,289B7EC6                     ; |
0048711F   |.  50             push eax                             ; |Arg2
00487120   |.  6A 04          push 4                               ; |Arg1 = 00000004
00487122   |.  8BC6           mov eax,esi                          ; |
00487124   |.  8B4D F0        mov ecx,dword ptr ss:[ebp-10]        ; |
00487127   |.  8B17           mov edx,dword ptr ds:[edi]           ; |
00487129   |.  E8 06FBFFFF    call INS.00486C34                    ; \INS.00486C34
0048712E   |.  8B45 F0        mov eax,dword ptr ss:[ebp-10]
00487131   |.  50             push eax                             ; /Arg3
00487132   |.  8B03           mov eax,dword ptr ds:[ebx]           ; |
00487134   |.  05 FA27A1EA    add eax,EAA127FA                     ; |
00487139   |.  50             push eax                             ; |Arg2
0048713A   |.  6A 0B          push 0B                              ; |Arg1 = 0000000B
0048713C   |.  8D45 EC        lea eax,dword ptr ss:[ebp-14]        ; |
0048713F   |.  8B0F           mov ecx,dword ptr ds:[edi]           ; |
00487141   |.  8B16           mov edx,dword ptr ds:[esi]           ; |
00487143   |.  E8 ECFAFFFF    call INS.00486C34                    ; \INS.00486C34
00487148   |.  8B07           mov eax,dword ptr ds:[edi]
0048714A   |.  50             push eax                             ; /Arg3
0048714B   |.  8B43 0C        mov eax,dword ptr ds:[ebx+C]         ; |
0048714E   |.  05 8530EFD4    add eax,D4EF3085                     ; |
00487153   |.  50             push eax                             ; |Arg2
00487154   |.  6A 10          push 10                              ; |Arg1 = 00000010
00487156   |.  8D45 F0        lea eax,dword ptr ss:[ebp-10]        ; |
00487159   |.  8B0E           mov ecx,dword ptr ds:[esi]           ; |
0048715B   |.  8B55 EC        mov edx,dword ptr ss:[ebp-14]        ; |
0048715E   |.  E8 D1FAFFFF    call INS.00486C34                    ; \INS.00486C34
00487163   |.  8B06           mov eax,dword ptr ds:[esi]
00487165   |.  50             push eax                             ; /Arg3
00487166   |.  8B43 18        mov eax,dword ptr ds:[ebx+18]        ; |
00487169   |.  05 051D8804    add eax,4881D05                      ; |
0048716E   |.  50             push eax                             ; |Arg2
0048716F   |.  6A 17          push 17                              ; |Arg1 = 00000017
00487171   |.  8BC7           mov eax,edi                          ; |
00487173   |.  8B4D EC        mov ecx,dword ptr ss:[ebp-14]        ; |
00487176   |.  8B55 F0        mov edx,dword ptr ss:[ebp-10]        ; |
00487179   |.  E8 B6FAFFFF    call INS.00486C34                    ; \INS.00486C34
0048717E   |.  8B45 EC        mov eax,dword ptr ss:[ebp-14]
00487181   |.  50             push eax                             ; /Arg3
00487182   |.  8B43 24        mov eax,dword ptr ds:[ebx+24]        ; |
00487185   |.  05 39D0D4D9    add eax,D9D4D039                     ; |
0048718A   |.  50             push eax                             ; |Arg2
0048718B   |.  6A 04          push 4                               ; |Arg1 = 00000004
0048718D   |.  8BC6           mov eax,esi                          ; |
0048718F   |.  8B4D F0        mov ecx,dword ptr ss:[ebp-10]        ; |
00487192   |.  8B17           mov edx,dword ptr ds:[edi]           ; |
00487194   |.  E8 9BFAFFFF    call INS.00486C34                    ; \INS.00486C34
00487199   |.  8B45 F0        mov eax,dword ptr ss:[ebp-10]
0048719C   |.  50             push eax                             ; /Arg3
0048719D   |.  8B43 30        mov eax,dword ptr ds:[ebx+30]        ; |
```
