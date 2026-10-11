
```
005E7D00      52                push edx  

005E7D01      50                push eax  

005E7D02      6A 02             push 2  

005E7D04      FF15 58104000     call dword ptr ds:[<&MSVBVM60.__vbaFreeVarList>]  ; MSVBVM60.__vbaFreeVarList  

005E7D0A      8B0F              mov ecx,dword ptr ds:[edi]  

005E7D0C      83C4 0C           add esp,0C  

005E7D0F      51                push ecx  

005E7D10      FFD3              call ebx  

005E7D12      8BD0              mov edx,eax  

005E7D14      8B45 E0           mov eax,dword ptr ss:[ebp-20]  

005E7D17      83C2 10           add edx,10                              ; 机器码长度+10  

005E7D1A      0F80 04020000     jo Syncroni.005E7F24  

005E7D20      0FAFD0            imul edx,eax                            ; edx*eax  

005E7D23      0F80 FB010000     jo Syncroni.005E7F24  

005E7D29      03D0              add edx,eax                             ; edx+eax  

005E7D2B      8B07              mov eax,dword ptr ds:[edi]  

005E7D2D      0F80 F1010000     jo Syncroni.005E7F24  

005E7D33      50                push eax  

005E7D34      8955 E0           mov dword ptr ss:[ebp-20],edx           ; edx 为上面计算结果  

005E7D37      FFD3              call ebx  

005E7D39      8B55 E0           mov edx,dword ptr ss:[ebp-20]  

005E7D3C      8B4D E4           mov ecx,dword ptr ss:[ebp-1C]  

005E7D3F      03C2              add eax,edx                             ; eax+edx  

005E7D41      898D 7CFFFFFF     mov dword ptr ss:[ebp-84],ecx  

005E7D47      0F80 D7010000     jo Syncroni.005E7F24  

005E7D4D      8945 E0           mov dword ptr ss:[ebp-20],eax           ; eax 为最后计算结果  

005E7D50      8D45 84           lea eax,dword ptr ss:[ebp-7C]  

005E7D53      8D4D C4           lea ecx,dword ptr ss:[ebp-3C]  

005E7D56      8D55 E0           lea edx,dword ptr ss:[ebp-20]  

005E7D59      50                push eax  

005E7D5A      51                push ecx  

005E7D5B      C785 74FFFFFF 080>mov dword ptr ss:[ebp-8C],8  

005E7D65      8955 8C           mov dword ptr ss:[ebp-74],edx  

005E7D68      C745 84 03400000  mov dword ptr ss:[ebp-7C],4003  

005E7D6F      FF15 E8134000     call dword ptr ds:[<&MSVBVM60.#613>]        ; MSVBVM60.rtcVarStrFromVar  

005E7D75      8D55 C4           lea edx,dword ptr ss:[ebp-3C]  

005E7D78      8D45 D4           lea eax,dword ptr ss:[ebp-2C]  

005E7D7B      52                push edx  

005E7D7C      50                push eax  

005E7D7D      FF15 04134000     call dword ptr ds:[<&MSVBVM60.__vbaStrVarVal>]    ; MSVBVM60.__vbaStrVarVal  

005E7D83      50                push eax                          ; 最后结果转化为 10 进制后压栈  

005E7D84      FF15 A8124000     call dword ptr ds:[<&MSVBVM60.#713>]        ; MSVBVM60.rtcStrReverse  

005E7D8A      8D4D B4           lea ecx,dword ptr ss:[ebp-4C]     ; 逆向  

005E7D8D      8D55 A4           lea edx,dword ptr ss:[ebp-5C]  

005E7D90      51                push ecx  

005E7D91      52                push edx  

005E7D92      8945 BC           mov dword ptr ss:[ebp-44],eax     ; 逆向后结果  

005E7D95      C745 B4 08000000  mov dword ptr ss:[ebp-4C],8  

005E7D9C      FF15 60114000     call dword ptr ds:[<&MSVBVM60.#520>]        ; MSVBVM60.rtcTrimVar  

005E7DA2      8D85 74FFFFFF     lea eax,dword ptr ss:[ebp-8C]  

005E7DA8      8D4D A4           lea ecx,dword ptr ss:[ebp-5C]  

005E7DAB      50                push eax  

005E7DAC      51                push ecx  

005E7DAD      8D55 94           lea edx,dword ptr ss:[ebp-6C]  

005E7DB0      52                push edx  

005E7DB1      FF15 08134000     call dword ptr ds:[<&MSVBVM60.__vbaVarCat>]       ; MSVBVM60.__vbaVarCat  

005E7DB7      50                push eax  

005E7DB8      FF15 48104000     call dword ptr ds:[<&MSVBVM60.__vbaStrVarMove>]   ; MSVBVM60.__vbaStrVarMove  

005E7DBE      8BD0              mov edx,eax  

005E7DC0      8D4D E4           lea ecx,dword ptr ss:[ebp-1C]  

005E7DC3      FF15 28144000     call dword ptr ds:[<&MSVBVM60.__vbaStrMove>]      ; MSVBVM60.__vbaStrMove  

005E7DC9      8D4D D4           lea ecx,dword ptr ss:[ebp-2C]  

005E7DCC      FF15 8C144000     call dword ptr ds:[<&MSVBVM60.__vbaFreeStr>]      ; MSVBVM60.__vbaFreeStr  

005E7DD2      8D45 94           lea eax,dword ptr ss:[ebp-6C]  

005E7DD5      8D4D A4           lea ecx,dword ptr ss:[ebp-5C]  

005E7DD8      50                push eax  

005E7DD9      8D55 B4           lea edx,dword ptr ss:[ebp-4C]  

005E7DDC      51                push ecx  

005E7DDD      8D45 C4           lea eax,dword ptr ss:[ebp-3C]  

005E7DE0      52                push edx  

005E7DE1      50                push eax  

005E7DE2      6A 04             push 4  

005E7DE4      FF15 58104000     call dword ptr ds:[<&MSVBVM60.__vbaFreeVarList>]  ; MSVBVM60.__vbaFreeVarList  

005E7DEA      B8 01000000       mov eax,1  

005E7DEF      83C4 14           add esp,14  

005E7DF2      66:03C6           add ax,si  

005E7DF5      0F80 29010000     jo Syncroni.005E7F24  

005E7DFB      8BF0              mov esi,eax  

005E7DFD    ^ E9 8FFEFFFF       jmp Syncroni.005E7C91                   ; 循环计算  

005E7E02      8B55 E4           mov edx,dword ptr ss:[ebp-1C]  

005E7E05      8D4D DC           lea ecx,dword ptr ss:[ebp-24]  

005E7E08      FF15 74134000     call dword ptr ds:[<&MSVBVM60.__vbaStrCopy>]      ; MSVBVM60.__vbaStrCopy  

005E7E0E      8B4D DC           mov ecx,dword ptr ss:[ebp-24]  

005E7E11      51                push ecx  

005E7E12      FFD3              call ebx  

005E7E14      83F8 08           cmp eax,8                         ; 长度与 8 比较  

005E7E17      7C 39             jl short Syncroni.005E7E52        ; 小于就跳  

005E7E19      8B35 7C144000     mov esi,dword ptr ds:[<&MSVBVM60.__vbaMidStmtBstr>; MSVBVM60.__vbaMidStmtBstr  

005E7E1F      8D55 DC           lea edx,dword ptr ss:[ebp-24]  

005E7E22      52                push edx  

005E7E23      6A 05             push 5  

005E7E25      6A 01             push 1  

005E7E27      68 74724200       push Syncroni.00427274  

005E7E2C      6A 00             push 0  

005E7E2E      FFD6              call esi                          ; 第五，十，十五位使用“-”代替  

005E7E30      8D45 DC           lea eax,dword ptr ss:[ebp-24]  

005E7E33      50                push eax  

005E7E34      6A 0A             push 0A  

005E7E36      6A 01             push 1  

005E7E38      68 74724200       push Syncroni.00427274  

005E7E3D      6A 00             push 0  

005E7E3F      FFD6              call esi                          ;  

005E7E41      8D4D DC           lea ecx,dword ptr ss:[ebp-24]  

005E7E44      51                push ecx  

005E7E45      6A 0F             push 0F  

005E7E47      6A 01             push 1  

005E7E49      68 74724200       push Syncroni.00427274  

005E7E4E      6A 00             push 0  

005E7E50      FFD6              call esi  

005E7E52      8B35 10144000     mov esi,dword ptr ds:[<&MSVBVM60.#617>]         ; MSVBVM60.rtcLeftCharVar  

005E7E58      8D45 84           lea eax,dword ptr ss:[ebp-7C]  

005E7E5B      6A 10             push 10  

005E7E5D      8D4D C4           lea ecx,dword ptr ss:[ebp-3C]  

005E7E60      8D55 DC           lea edx,dword ptr ss:[ebp-24]  

005E7E63      50                push eax  

005E7E64      51                push ecx  

005E7E65      8955 8C           mov dword ptr ss:[ebp-74],edx  

005E7E68      C745 84 08400000  mov dword ptr ss:[ebp-7C],4008  

005E7E6F      FFD6              call esi  

005E7E71      8D55 C4           lea edx,dword ptr ss:[ebp-3C]  

005E7E74      52                push edx  
```
