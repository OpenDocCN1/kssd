

# PECompact1.48

> 原文：[`www.pediy.com/kssd/pediy03/forum1097.htm`](https://www.pediy.com/kssd/pediy03/forum1097.htm)

pecompact1.48 破解过程   
用 trw2000 载入下 bpx 41a000 后用 suspend 打开 peditor---tasks---dump(full)   
再把入口点改为 41a000,脱壳完成   
用 W32Dasm 打开 pecompact.exe   
\* Possible StringData Ref from Code Obj ->"About PECompact"   
                                  |   
:00404DA7 688ACA4000              push 0040CA8A   
:00404DAC FF7508                  push \[ebp+08\]   
:00404DAF E852500000              call 00409E06   
:00404DB4 A1F0E14000              mov eax, dword ptr \[0040E1F0\]   
:00404DB9 83F800                  cmp eax, 00000000   
:00404DBC 7F23                    jg 00404DE1------>>改为 jmp 00404DE1   
  
\* Possible StringData Ref from Code Obj ->"YOUR TRIAL PERIOD HAS ENDED!"   
                                  |   
:00404DBE 68E4EF4000              push 0040EFE4   
:00404DC3 6870040000              push 00000470   
:00404DC8 FF7508                  push \[ebp+08\]   
:00404DCB E848500000              call 00409E18   
  
\* Possible StringData Ref from Code Obj ->"Exit"   
                                  |   
:00404DD0 6801F04000              push 0040F001   
:00404DD5 6A01                    push 00000001   
:00404DD7 FF7508                  push \[ebp+08\]   
:00404DDA E839500000              call 00409E18   
:00404DDF EB25                    jmp 00404E06   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00404DBC(C)   
|   
:00404DE1 50                      push eax   
  
\* Possible StringData Ref from Code Obj ->"You have %d days remaining of "   
                                        ->"your trial."   
                                  |   
:00404DE2 68BAEF4000              push 0040EFBA   
:00404DE7 6856EF4000              push 0040EF56   
:00404DEC E863500000              call 00409E54   
:00404DF1 83C40C                  add esp, 0000000C   
:00404DF4 6856EF4000              push 0040EF56   
:00404DF9 6870040000              push 00000470   
:00404DFE FF7508                  push \[ebp+08\]   
:00404E01 E812500000              call 00409E18   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00404DDF(U)   
|   
  
\* Possible StringData Ref from Code Obj ->"   
Licensed for 14-day evaluation. "   
                                        ->"Registration is required for continued "   
                                        ->"use."   
                                  |   
:00404E06 6839DA4000              push 0040DA39   
:00404E0B 680C040000              push 0000040C   
:00404E10 FF7508                  push \[ebp+08\]   
:00404E13 E800500000              call 00409E18   
  
\* Possible StringData Ref from Code Obj ->"PECompact v1.48, ?999-2001 by "   
                                        ->"Jeremy Collake"   
                                  |   
:00404E18 680BDA4000              push 0040DA0B   
:00404E1D 6853040000              push 00000453   
:00404E22 FF7508                  push \[ebp+08\]   
:00404E25 E8EE4F0000              call 00409E18   
  
\* Possible StringData Ref from Code Obj ->"Unregistered!"   
                                  |   
:00404E2A 689ACA4000              push 0040CA9A   
:00404E2F 6834040000              push 00000434   
:00404E34 FF7508                  push \[ebp+08\]   
:00404E37 E8DC4F0000              call 00409E18   
:00404E3C 33C0                    xor eax, eax   
:00404E3E 5E                      pop esi   
:00404E3F 5F                      pop edi   
:00404E40 5B                      pop ebx   
:00404E41 C9                      leave   
:00404E42 C21000                  ret 0010   
  
  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00404D6F(C)   
|   
:00404E45 837D0C10                cmp dword ptr \[ebp+0C\], 00000010   
:00404E49 7443                    je 00404E8E   
:00404E4B 817D0C11010000          cmp dword ptr \[ebp+0C\], 00000111   
:00404E52 7531                    jne 00404E85   
:00404E54 837D1001                cmp dword ptr \[ebp+10\], 00000001   
:00404E58 7434                    je 00404E8E   
:00404E5A 817D1010040000          cmp dword ptr \[ebp+10\], 00000410   
:00404E61 7522                    jne 00404E85   
:00404E63 6A00                    push 00000000   
:00404E65 6A00                    push 00000000   
:00404E67 6A00                    push 00000000   
  
\* Possible StringData Ref from Code Obj ->"http://www.collakesoftware.com"   
                                  |   
:00404E69 68A7D84000              push 0040D8A7   
:00404E6E 6A00                    push 00000000   
:00404E70 FF3563E14000            push dword ptr \[0040E163\]   
:00404E76 E863500000              call 00409EDE   
:00404E7B 33C0                    xor eax, eax   
:00404E7D 40                      inc eax   
:00404E7E 5E                      pop esi   
:00404E7F 5F                      pop edi   
:00404E80 5B                      pop ebx   
:00404E81 C9                      leave   
:00404E82 C21000                  ret 0010   
  
  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:00404E52(C), :00404E61(C)   
|   
:00404E85 33C0                    xor eax, eax   
:00404E87 5E                      pop esi   
:00404E88 5F                      pop edi   
:00404E89 5B                      pop ebx   
:00404E8A C9                      leave   
:00404E8B C21000                  ret 0010   
  
  
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:00404E49(C), :00404E58(C)   
|   
:00404E8E 833DF0E1400000          cmp dword ptr \[0040E1F0\], 00000000   
:00404E95 7F07                    jg 00404E9E------>>改为 jmp 00404E9E   
:00404E97 6A00                    push 00000000   
:00404E99 E8544E0000              call 00409CF2   
  
时间破解完成,但在压缩时会跳出 About 窗口,干掉他!!!!   
  
\* Possible Ref to Menu: MenuID\_0066, Item: "About"   
                                  |   
:00401093 68429C0000              push 00009C42   
:00401098 FF3563E14000            push dword ptr \[0040E163\]   
:0040109E E8E2520000              call 00406385------>>改为 5 个 nop   
:004010A3 E87D160000              call 00402725   
:004010A8 6A42                    push 00000042   
:004010AA E8B7160000              call 00402766   
:004010AF 6838DC4000              push 0040DC38   
:004010B4 E82A1E0000              call 00402EE3   
:004010B9 C7055CF54000A6F04000    mov dword ptr \[0040F55C\], 0040F0A6   
:004010C3 6896DA4000              push 0040DA96   
:004010C8 FF355CF54000            push dword ptr \[0040F55C\]   
:004010CE E8F8530000              call 004064CB   
:004010D3 A3F9F54000              mov dword ptr \[0040F5F9\], eax   
:004010D8 6A00                    push 00000000   
:004010DA 684F040000              push 0000044F   
:004010DF FF3563E14000            push dword ptr \[0040E163\]   
:004010E5 E82E8D0000              call 00409E18   
  
破解到此完成!!!!   
交作业了!!!!!   
conanxu\[BCG\]   
conanxu@eastday.com  4dd466d0e1e0cc123afbe504da2462ca