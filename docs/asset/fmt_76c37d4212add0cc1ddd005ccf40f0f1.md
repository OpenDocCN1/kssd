
```
004ABA30 ResH>  55                    push ebp  

004ABA31        8BEC                  mov ebp,esp  

004ABA33        83C4 F0               add esp,-10  

004ABA36        33C0                  xor eax,eax  

004ABA38        8945 F0               mov dword ptr ss:[ebp-10],eax  

004ABA3B        B8 60B84A00           mov eax,ResHacke.004AB860  

004ABA40        E8 B79CF5FF          call ResHacke.004056FC  

004ABA45        33C0                  xor eax,eax  

004ABA47        55                    push ebp  

004ABA48        68 40BB4A00          push ResHacke.004ABB40  

004ABA4D        64:FF30              push dword ptr fs:[eax]  

004ABA50        64:8920              mov dword ptr fs:[eax],esp  

004ABA53        A1 60DC4A00          mov eax,dword ptr ds:[4ADC60]  

004ABA58        8B00                  mov eax,dword ptr ds:[eax]  

004ABA5A        E8 856EF8FF          call ResHacke.004328E4  

004ABA5F        A1 60DC4A00          mov eax,dword ptr ds:[4ADC60]  

004ABA64        8B00                  mov eax,dword ptr ds:[eax]  

004ABA66        BA 54BB4A00          mov edx,ResHacke.004ABB54                                  ; ASCII "Resource Hacker"  

004ABA6B        E8 8C6BF8FF          call ResHacke.004325FC  

004ABA70        A1 60DC4A00          mov eax,dword ptr ds:[4ADC60]  

004ABA75        8B00                  mov eax,dword ptr ds:[eax]  

004ABA77        C640 3F 00            mov byte ptr ds:[eax+3F],0  

004ABA7B        8B0D D8DA4A00         mov ecx,dword ptr ds:[4ADAD8]                              ; ResHacke.004AE81C  

004ABA81        A1 60DC4A00          mov eax,dword ptr ds:[4ADC60]  

004ABA86        8B00                  mov eax,dword ptr ds:[eax]  

004ABA88        8B15 DCE44900         mov edx,dword ptr ds:[49E4DC]                              ; ResHacke.0049E51C  

004ABA8E        E8 696EF8FF          call ResHacke.004328FC  

004ABA93        E8 786DF5FF          call ResHacke.00402810  

004ABA98        48                    dec eax  

004ABA99        7E 78                 jle short ResHacke.004ABB13  

004ABA9B        8D55 F0               lea edx,dword ptr ss:[ebp-10]  

004ABA9E        B8 01000000          mov eax,1  

004ABAA3        E8 C86DF5FF          call ResHacke.00402870  

004ABAA8        8B45 F0               mov eax,dword ptr ss:[ebp-10]  

004ABAAB        8038 2D               cmp byte ptr ds:[eax],2D  

004ABAAE        75 63                 jnz short ResHacke.004ABB13  

004ABAB0        B2 01                 mov dl,1  

004ABAB2        A1 88AE4800          mov eax,dword ptr ds:[48AE88]  

004ABAB7        E8 B872F5FF          call ResHacke.00402D74  

004ABABC        A3 84E84A00          mov dword ptr ds:[4AE884],eax  

004ABAC1        33C0                  xor eax,eax  

004ABAC3        55                    push ebp  

004ABAC4        68 F9BA4A00          push ResHacke.004ABAF9  

004ABAC9        64:FF30              push dword ptr fs:[eax]  

004ABACC        64:8920              mov dword ptr fs:[eax],esp  

004ABACF        8B15 D8DA4A00         mov edx,dword ptr ds:[4ADAD8]                              ; ResHacke.004AE81C  

004ABAD5        8B12                  mov edx,dword ptr ds:[edx]  

004ABAD7        A1 84E84A00          mov eax,dword ptr ds:[4AE884]  

004ABADC        E8 8BF5FDFF          call ResHacke.0048B06C  

004ABAE1        33C0                  xor eax,eax  

004ABAE3        5A                    pop edx                                             ; kernel32.7C816FD7  

004ABAE4        59                    pop ecx                                             ; kernel32.7C816FD7  

004ABAE5        59                    pop ecx                                             ; kernel32.7C816FD7  

004ABAE6        64:8910              mov dword ptr fs:[eax],edx                            ; ntdll.KiFastSystemCallRet  

004ABAE9        68 00BB4A00          push ResHacke.004ABB00  

004ABAEE        A1 84E84A00          mov eax,dword ptr ds:[4AE884]  

004ABAF3        E8 A472F5FF          call ResHacke.00402D9C  

004ABAF8        C3                    retn  

004ABAF9      ^ E9 6A78F5FF          jmp ResHacke.00403368  

004ABAFE      ^ EB EE                jmp short ResHacke.004ABAEE  

004ABB00        A1 D8DA4A00          mov eax,dword ptr ds:[4ADAD8]  

004ABB05        8B00                  mov eax,dword ptr ds:[eax]  

004ABB07        E8 304DF8FF          call ResHacke.0043083C  

004ABB0C        E8 837CF5FF          call ResHacke.00403794  

004ABB11        EB 0B                 jmp short ResHacke.004ABB1E  

004ABB13        A1 60DC4A00          mov eax,dword ptr ds:[4ADC60]  

004ABB18        8B00                  mov eax,dword ptr ds:[eax]  

004ABB1A        C640 3F 01            mov byte ptr ds:[eax+3F],1  

004ABB1E        A1 60DC4A00          mov eax,dword ptr ds:[4ADC60]  

004ABB23        8B00                  mov eax,dword ptr ds:[eax]  

004ABB25        E8 5E6EF8FF          call ResHacke.00432988  

004ABB2A        33C0                  xor eax,eax  

004ABB2C        5A                    pop edx                                             ; kernel32.7C816FD7  

004ABB2D        59                    pop ecx                                             ; kernel32.7C816FD7  

004ABB2E        59                    pop ecx                                             ; kernel32.7C816FD7  

004ABB2F        64:8910              mov dword ptr fs:[eax],edx                            ; ntdll.KiFastSystemCallRet  

004ABB32        68 47BB4A00          push ResHacke.004ABB47  

004ABB37        8D45 F0               lea eax,dword ptr ss:[ebp-10]  

004ABB3A        E8 097EF5FF          call ResHacke.00403948  

004ABB3F        C3                    retn  


55 8B EC 83 C4 F0 33 C0 89 45 F0 B8 60 B8 4A 00 E8 B7 9C F5 FF 33 C0 55 68 40 BB 4A 00 64 FF 30  
64 89 20 A1 60 DC 4A 00 8B 00 E8 85 6E F8 FF A1 60 DC 4A 00 8B 00 BA 54 BB 4A 00 E8 8C 6B F8 FF  
A1 60 DC 4A 00 8B 00 C6 40 3F 00 8B 0D D8 DA 4A 00 A1 60 DC 4A 00 8B 00 8B 15 DC E4 49 00 E8 69  
6E F8 FF E8 78 6D F5 FF 48 7E 78 8D 55 F0 B8 01 00 00 00 E8 C8 6D F5 FF 8B 45 F0 80 38 2D 75 63  
B2 01 A1 88 AE 48 00 E8 B8 72 F5 FF A3 84 E8 4A 00 33 C0 55 68 F9 BA 4A 00 64 FF 30 64 89 20 8B  
15 D8 DA 4A 00 8B 12 A1 84 E8 4A 00 E8 8B F5 FD FF 33 C0 5A 59 59 64 89 10 68 00 BB 4A 00 A1 84  
E8 4A 00 E8 A4 72 F5 FF C3 E9 6A 78 F5 FF EB EE A1 D8 DA 4A 00 8B 00 E8 30 4D F8 FF E8 83 7C F5  
FF EB 0B A1 60 DC 4A 00 8B 00 C6 40 3F 01 A1 60 DC 4A 00 8B 00 E8 5E 6E F8 FF 33 C0 5A 59 59 64  
89 10 68 47 BB 4A 00 8D 45 F0 E8 09 7E F5 FF C3  


********************************************************************************************  

Borland Delphi 4.0 - 5.0  


`CODE`

`DATA`

`BSS`

`.idata`

`.tls`

`.rdata`

`.reloc`

`.rsrc`  


00457E00 YUCE>  55                    push ebp  

00457E01        8BEC                  mov ebp,esp  

00457E03        83C4 F4               add esp,-0C  

00457E06        B8 C87C4500          mov eax,YUCE.00457CC8  

00457E0B        E8 44E4FAFF          call YUCE.00406254  

00457E10        A1 048F4500          mov eax,dword ptr ds:[458F04]  

00457E15        8B00                  mov eax,dword ptr ds:[eax]  

00457E17        BA 547E4500          mov edx,YUCE.00457E54  

00457E1C        E8 83DEFEFF          call YUCE.00445CA4  

00457E21        8B0D 688D4500         mov ecx,dword ptr ds:[458D68]                              ; YUCE.0045A878  

00457E27        A1 048F4500          mov eax,dword ptr ds:[458F04]  

00457E2C        8B00                  mov eax,dword ptr ds:[eax]  
```
