
```
00487148   |.  8B07             mov eax,dword ptr ds:[edi]  

0048714A   |.  50               push eax                     ; /Arg3  

0048714B   |.  8B43 0C          mov eax,dword ptr ds:[ebx+C]     ; |  

0048714E   |.  05 8530EFD4      add eax,D4EF3085                 ; |  

00487153   |.  50               push eax                     ; |Arg2  

00487154   |.  6A 10            push 10                      ; |Arg1 = 00000010  

00487156   |.  8D45 F0          lea eax,dword ptr ss:[ebp-10]      ; |  

00487159   |.  8B0E             mov ecx,dword ptr ds:[esi]        ; |  

0048715B   |.  8B55 EC          mov edx,dword ptr ss:[ebp-14]      ; |  

0048715E   |.  E8 D1FAFFFF      call INS.00486C34                ; \INS.00486C34  

00487163   |.  8B06             mov eax,dword ptr ds:[esi]  

00487165   |.  50               push eax                     ; /Arg3  

00487166   |.  8B43 18          mov eax,dword ptr ds:[ebx+18]      ; |  

00487169   |.  05 051D8804      add eax,4881D05                  ; |  

0048716E   |.  50               push eax                     ; |Arg2  

0048716F   |.  6A 17            push 17                      ; |Arg1 = 00000017  

00487171   |.  8BC7             mov eax,edi                    ; |  

00487173   |.  8B4D EC          mov ecx,dword ptr ss:[ebp-14]      ; |  

00487176   |.  8B55 F0          mov edx,dword ptr ss:[ebp-10]      ; |  

00487179   |.  E8 B6FAFFFF      call INS.00486C34                ; \INS.00486C34  

0048717E   |.  8B45 EC          mov eax,dword ptr ss:[ebp-14]  

00487181   |.  50               push eax                     ; /Arg3  

00487182   |.  8B43 24          mov eax,dword ptr ds:[ebx+24]      ; |  

00487185   |.  05 39D0D4D9      add eax,D9D4D039                 ; |  

0048718A   |.  50               push eax                     ; |Arg2  

0048718B   |.  6A 04            push 4                       ; |Arg1 = 00000004  

0048718D   |.  8BC6             mov eax,esi                    ; |  

0048718F   |.  8B4D F0          mov ecx,dword ptr ss:[ebp-10]      ; |  

00487192   |.  8B17             mov edx,dword ptr ds:[edi]        ; |  

00487194   |.  E8 9BFAFFFF      call INS.00486C34                ; \INS.00486C34  

00487199   |.  8B45 F0          mov eax,dword ptr ss:[ebp-10]  

0048719C   |.  50               push eax                     ; /Arg3  

0048719D   |.  8B43 30          mov eax,dword ptr ds:[ebx+30]      ; |  

004871A0   |.  05 E599DBE6      add eax,E6DB99E5                 ; |  

004871A5   |.  50               push eax                     ; |Arg2  

004871A6   |.  6A 0B            push 0B                      ; |Arg1 = 0000000B  

004871A8   |.  8D45 EC          lea eax,dword ptr ss:[ebp-14]      ; |  

004871AB   |.  8B0F             mov ecx,dword ptr ds:[edi]        ; |  

004871AD   |.  8B16             mov edx,dword ptr ds:[esi]        ; |  

004871AF   |.  E8 80FAFFFF      call INS.00486C34                ; \INS.00486C34  

004871B4   |.  8B07             mov eax,dword ptr ds:[edi]  

004871B6   |.  50               push eax                     ; /Arg3  

004871B7   |.  8B43 3C          mov eax,dword ptr ds:[ebx+3C]      ; |  

004871BA   |.  05 F87CA21F      add eax,1FA27CF8                 ; |  

004871BF   |.  50               push eax                     ; |Arg2  

004871C0   |.  6A 10            push 10                      ; |Arg1 = 00000010  

004871C2   |.  8D45 F0          lea eax,dword ptr ss:[ebp-10]      ; |  

004871C5   |.  8B0E             mov ecx,dword ptr ds:[esi]        ; |  

004871C7   |.  8B55 EC          mov edx,dword ptr ss:[ebp-14]      ; |  

004871CA   |.  E8 65FAFFFF      call INS.00486C34                ; \INS.00486C34  

004871CF   |.  8B06             mov eax,dword ptr ds:[esi]  

004871D1   |.  50               push eax                     ; /Arg3  

004871D2   |.  8B43 08          mov eax,dword ptr ds:[ebx+8]       ; |  

004871D5   |.  05 6556ACC4      add eax,C4AC5665                 ; |  

004871DA   |.  50               push eax                     ; |Arg2  

004871DB   |.  6A 17            push 17                      ; |Arg1 = 00000017  

004871DD   |.  8BC7             mov eax,edi                    ; |  

004871DF   |.  8B4D EC          mov ecx,dword ptr ss:[ebp-14]      ; |  

004871E2   |.  8B55 F0          mov edx,dword ptr ss:[ebp-10]      ; |  

004871E5   |.  E8 4AFAFFFF      call INS.00486C34                ; \INS.00486C34  

004871EA   |.  8B45 EC          mov eax,dword ptr ss:[ebp-14]  

004871ED   |.  50               push eax                     ; /Arg3  

004871EE   |.  8B03             mov eax,dword ptr ds:[ebx]        ; |  

004871F0   |.  05 442229F4      add eax,F4292244                 ; |  

004871F5   |.  50               push eax                     ; |Arg2  

004871F6   |.  6A 06            push 6                       ; |Arg1 = 00000006  

004871F8   |.  8BC6             mov eax,esi                    ; |  

004871FA   |.  8B4D F0          mov ecx,dword ptr ss:[ebp-10]      ; |  

004871FD   |.  8B17             mov edx,dword ptr ds:[edi]        ; |  

004871FF   |.  E8 68FAFFFF      call INS.00486C6C                ; \INS.00486C6C  

00487204   |.  8B45 F0          mov eax,dword ptr ss:[ebp-10]  

00487207   |.  50               push eax                     ; /Arg3  

00487208   |.  8B43 1C          mov eax,dword ptr ds:[ebx+1C]      ; |  

0048720B   |.  05 97FF2A43      add eax,432AFF97                 ; |  

00487210   |.  50               push eax                     ; |Arg2  

00487211   |.  6A 0A            push 0A                      ; |Arg1 = 0000000A  

00487213   |.  8D45 EC          lea eax,dword ptr ss:[ebp-14]      ; |  

00487216   |.  8B0F             mov ecx,dword ptr ds:[edi]        ; |  

00487218   |.  8B16             mov edx,dword ptr ds:[esi]        ; |  

0048721A   |.  E8 4DFAFFFF      call INS.00486C6C                ; \INS.00486C6C  

0048721F   |.  8B07             mov eax,dword ptr ds:[edi]  

00487221   |.  50               push eax                     ; /Arg3  

00487222   |.  8B43 38          mov eax,dword ptr ds:[ebx+38]      ; |  

00487225   |.  05 A72394AB      add eax,AB9423A7                 ; |  

0048722A   |.  50               push eax                     ; |Arg2  

0048722B   |.  6A 0F            push 0F                      ; |Arg1 = 0000000F  

0048722D   |.  8D45 F0          lea eax,dword ptr ss:[ebp-10]      ; |  

00487230   |.  8B0E             mov ecx,dword ptr ds:[esi]        ; |  

00487232   |.  8B55 EC          mov edx,dword ptr ss:[ebp-14]      ; |  

00487235   |.  E8 32FAFFFF      call INS.00486C6C                ; \INS.00486C6C  

0048723A   |.  8B06             mov eax,dword ptr ds:[esi]  

0048723C   |.  50               push eax                     ; /Arg3  

0048723D   |.  8B43 14          mov eax,dword ptr ds:[ebx+14]      ; |  

00487240   |.  05 39A093FC      add eax,FC93A039                 ; |  

00487245   |.  50               push eax                     ; |Arg2  

00487246   |.  6A 15            push 15                      ; |Arg1 = 00000015  

00487248   |.  8BC7             mov eax,edi                    ; |  

0048724A   |.  8B4D EC          mov ecx,dword ptr ss:[ebp-14]      ; |  

0048724D   |.  8B55 F0          mov edx,dword ptr ss:[ebp-10]      ; |  

00487250   |.  E8 17FAFFFF      call INS.00486C6C                ; \INS.00486C6C  

00487255   |.  8B45 EC          mov eax,dword ptr ss:[ebp-14]  

00487258   |.  50               push eax                     ; /Arg3  

00487259   |.  8B43 30          mov eax,dword ptr ds:[ebx+30]      ; |  
```
