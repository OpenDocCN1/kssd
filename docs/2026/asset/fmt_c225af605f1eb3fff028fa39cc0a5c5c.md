
```
004872C1   |.  8B45 EC        mov eax,dword ptr ss:[ebp-14]
004872C4   |.  50             push eax                             ; /Arg3
004872C5   |.  8B43 20        mov eax,dword ptr ds:[ebx+20]        ; |
004872C8   |.  05 4F7EA86F    add eax,6FA87E4F                     ; |
004872CD   |.  50             push eax                             ; |Arg2
004872CE   |.  6A 06          push 6                               ; |Arg1 = 00000006
004872D0   |.  8BC6           mov eax,esi                          ; |
004872D2   |.  8B4D F0        mov ecx,dword ptr ss:[ebp-10]        ; |
004872D5   |.  8B17           mov edx,dword ptr ds:[edi]           ; |
004872D7   |.  E8 90F9FFFF    call INS.00486C6C                    ; \INS.00486C6C
004872DC   |.  8B45 F0        mov eax,dword ptr ss:[ebp-10]
004872DF   |.  50             push eax                             ; /Arg3
004872E0   |.  8B43 3C        mov eax,dword ptr ds:[ebx+3C]        ; |
004872E3   |.  05 E0E62CFE    add eax,FE2CE6E0                     ; |
004872E8   |.  50             push eax                             ; |Arg2
004872E9   |.  6A 0A          push 0A                              ; |Arg1 = 0000000A
004872EB   |.  8D45 EC        lea eax,dword ptr ss:[ebp-14]        ; |
004872EE   |.  8B0F           mov ecx,dword ptr ds:[edi]           ; |
004872F0   |.  8B16           mov edx,dword ptr ds:[esi]           ; |
004872F2   |.  E8 75F9FFFF    call INS.00486C6C                    ; \INS.00486C6C
004872F7   |.  8B07           mov eax,dword ptr ds:[edi]
004872F9   |.  50             push eax                             ; /Arg3
004872FA   |.  8B43 18        mov eax,dword ptr ds:[ebx+18]        ; |
004872FD   |.  05 144301A3    add eax,A3014314                     ; |
00487302   |.  50             push eax                             ; |Arg2
00487303   |.  6A 0F          push 0F                              ; |Arg1 = 0000000F
00487305   |.  8D45 F0        lea eax,dword ptr ss:[ebp-10]        ; |
00487308   |.  8B0E           mov ecx,dword ptr ds:[esi]           ; |
0048730A   |.  8B55 EC        mov edx,dword ptr ss:[ebp-14]        ; |
0048730D   |.  E8 5AF9FFFF    call INS.00486C6C                    ; \INS.00486C6C
00487312   |.  8B06           mov eax,dword ptr ds:[esi]
00487314   |.  50             push eax                             ; /Arg3
00487315   |.  8B43 34        mov eax,dword ptr ds:[ebx+34]        ; |
00487318   |.  05 A111084E    add eax,4E0811A1                     ; |
0048731D   |.  50             push eax                             ; |Arg2
0048731E   |.  6A 15          push 15                              ; |Arg1 = 00000015
00487320   |.  8BC7           mov eax,edi                          ; |
00487322   |.  8B4D EC        mov ecx,dword ptr ss:[ebp-14]        ; |
00487325   |.  8B55 F0        mov edx,dword ptr ss:[ebp-10]        ; |
00487328   |.  E8 3FF9FFFF    call INS.00486C6C                    ; \INS.00486C6C
0048732D   |.  8B45 EC        mov eax,dword ptr ss:[ebp-14]
00487330   |.  50             push eax                             ; /Arg3
00487331   |.  8B43 10        mov eax,dword ptr ds:[ebx+10]        ; |
00487334   |.  05 827E53F7    add eax,F7537E82                     ; |
00487339   |.  50             push eax                             ; |Arg2
0048733A   |.  6A 06          push 6                               ; |Arg1 = 00000006
0048733C   |.  8BC6           mov eax,esi                          ; |
0048733E   |.  8B4D F0        mov ecx,dword ptr ss:[ebp-10]        ; |
00487341   |.  8B17           mov edx,dword ptr ds:[edi]           ; |
00487343   |.  E8 24F9FFFF    call INS.00486C6C                    ; \INS.00486C6C
00487348   |.  8B45 F0        mov eax,dword ptr ss:[ebp-10]
0048734B   |.  50             push eax                             ; /Arg3
0048734C   |.  8B43 2C        mov eax,dword ptr ds:[ebx+2C]        ; |
0048734F   |.  05 35F23ABD    add eax,BD3AF235                     ; |
00487354   |.  50             push eax                             ; |Arg2
00487355   |.  6A 0A          push 0A                              ; |Arg1 = 0000000A
00487357   |.  8D45 EC        lea eax,dword ptr ss:[ebp-14]        ; |
0048735A   |.  8B0F           mov ecx,dword ptr ds:[edi]           ; |
0048735C   |.  8B16           mov edx,dword ptr ds:[esi]           ; |
0048735E   |.  E8 09F9FFFF    call INS.00486C6C                    ; \INS.00486C6C
00487363   |.  8B07           mov eax,dword ptr ds:[edi]
00487365   |.  50             push eax                             ; /Arg3
00487366   |.  8B43 08        mov eax,dword ptr ds:[ebx+8]         ; |
00487369   |.  05 BBD2D72A    add eax,2AD7D2BB                     ; |
0048736E   |.  50             push eax                             ; |Arg2
0048736F   |.  6A 0F          push 0F                              ; |Arg1 = 0000000F
00487371   |.  8D45 F0        lea eax,dword ptr ss:[ebp-10]        ; |
00487374   |.  8B0E           mov ecx,dword ptr ds:[esi]           ; |
00487376   |.  8B55 EC        mov edx,dword ptr ss:[ebp-14]        ; |
00487379   |.  E8 EEF8FFFF    call INS.00486C6C                    ; \INS.00486C6C
0048737E   |.  8B06           mov eax,dword ptr ds:[esi]
00487380   |.  50             push eax                             ; /Arg3
00487381   |.  8B43 24        mov eax,dword ptr ds:[ebx+24]        ; |
00487384   |.  05 91D386EB    add eax,EB86D391                     ; |
00487389   |.  50             push eax                             ; |Arg2
0048738A   |.  6A 15          push 15                              ; |Arg1 = 00000015
0048738C   |.  8BC7           mov eax,edi                          ; |
0048738E   |.  8B4D EC        mov ecx,dword ptr ss:[ebp-14]        ; |
00487391   |.  8B55 F0        mov edx,dword ptr ss:[ebp-10]        ; |
00487394   |.  E8 D3F8FFFF    call INS.00486C6C                    ; \INS.00486C6C
00487399   |.  8B45 FC        mov eax,dword ptr ss:[ebp-4]
0048739C   |.  8B16           mov edx,dword ptr ds:[esi]
0048739E   |.  0110           add dword ptr ds:[eax],edx
004873A0   |.  8B45 FC        mov eax,dword ptr ss:[ebp-4]
004873A3   |.  8B17           mov edx,dword ptr ds:[edi]
004873A5   |.  0150 04        add dword ptr ds:[eax+4],edx
004873A8   |.  8B45 FC        mov eax,dword ptr ss:[ebp-4]
004873AB   |.  8B55 F0        mov edx,dword ptr ss:[ebp-10]
004873AE   |.  0150 08        add dword ptr ds:[eax+8],edx
004873B1   |.  8B45 FC        mov eax,dword ptr ss:[ebp-4]
004873B4   |.  8B55 EC        mov edx,dword ptr ss:[ebp-14]
004873B7   |.  0150 0C        add dword ptr ds:[eax+C],edx
004873BA   |.  5F             pop edi
004873BB   |.  5E             pop esi
004873BC   |.  5B             pop ebx
004873BD   |.  8BE5           mov esp,ebp
004873BF   |.  5D             pop ebp
004873C0   \.  C2 0400        retn 4
```

第一轮运算，下断处`00486BC0`:

```
00486BC0   /$  55             push ebp
00486BC1   |.  8BEC           mov ebp,esp
00486BC3   |.  53             push ebx
00486BC4   |.  56             push esi
00486BC5   |.  8B5D 10        mov ebx,dword ptr ss:[ebp+10]
00486BC8   |.  33CB           xor ecx,ebx
00486BCA   |.  23CA           and ecx,edx
00486BCC   |.  33D9           xor ebx,ecx
00486BCE   |.  035D 0C        add ebx,dword ptr ss:[ebp+C]
00486BD1   |.  0118           add dword ptr ds:[eax],ebx
00486BD3   |.  33C9           xor ecx,ecx
00486BD5   |.  8A4D 08        mov cl,byte ptr ss:[ebp+8]
00486BD8   |.  51             push ecx
00486BD9   |.  B9 20000000    mov ecx,20
00486BDE   |.  5B             pop ebx
00486BDF   |.  2BCB           sub ecx,ebx
00486BE1   |.  8B18           mov ebx,dword ptr ds:[eax]
00486BE3   |.  D3EB           shr ebx,cl
00486BE5   |.  8A4D 08        mov cl,byte ptr ss:[ebp+8]
00486BE8   |.  8B30           mov esi,dword ptr ds:[eax]
00486BEA   |.  D3E6           shl esi,cl
00486BEC   |.  0BDE           or ebx,esi
```
