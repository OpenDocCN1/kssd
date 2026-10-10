

# 一些常见语言所编写的程序的 OEP 特征

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-464.htm`](https://www.pediy.com/kssd/pediy09/pediy09-464.htm)

> 闲得无聊搜集的一些 OEP 和 Section，希望能对研究脱壳的朋友们帮上一点小忙。  
> 末尾有打包下载。  
>   
> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
> Borland C++ 1999  
>   
> .text  
> .data  
> .tls  
> .rdata  
> .idata  
> .edata  
> .rsrc  
> .reloc  
>   
> 00401000 Find> $ /EB 10               jmp short Finder33.00401012  
> 00401002         |66                  db 66                                                    ;  CHAR 'f'  
> 00401003         |62                  db 62                                                    ;  CHAR 'b'  
> 00401004         |3A                  db 3A                                                    ;  CHAR ':'  
> 00401005         |43                  db 43                                                    ;  CHAR 'C'  
> 00401006         |2B                  db 2B                                                    ;  CHAR '+'  
> 00401007         |2B                  db 2B                                                    ;  CHAR '+'  
> 00401008         |48                  db 48                                                    ;  CHAR 'H'  
> 00401009         |4F                  db 4F                                                    ;  CHAR 'O'  
> 0040100A         |4F                  db 4F                                                    ;  CHAR 'O'  
> 0040100B         |4B                  db 4B                                                    ;  CHAR 'K'  
> 0040100C         |90                  nop  
> 0040100D         |E9                  db E9  
> 0040100E       . |AC                  lods byte ptr ds:\[esi\]  
> 0040100F       . |2348 00             and ecx,dword ptr ds:\[eax\]  
> 00401012       > \\A1 9F234800         mov eax,dword ptr ds:\[48239F\]  
> 00401017       .  C1E0 02             shl eax,2  
> 0040101A       .  A3 A3234800         mov dword ptr ds:\[4823A3\],eax  
> 0040101F       .  52                  push edx                                                 ;  ntdll.KiFastSystemCallRet  
> 00401020       .  6A 00               push 0                                                   ; /pModule = NULL  
> 00401022       .  E8 79010800         call <jmp.&KERNEL32.GetModuleHandleA>                    ; \\GetModuleHandleA  
> 00401027       .  8BD0                mov edx,eax  
> 00401029       .  E8 F64F0700         call Finder33.00476024  
> 0040102E       .  5A                  pop edx                                                  ;  kernel32.7C816FD7  
> 0040102F       .  E8 544F0700         call Finder33.00475F88  
> 00401034       .  E8 2B500700         call Finder33.00476064  
> 00401039       .  6A 00               push 0                                                   ; /Arg1 = 00000000  
> 0040103B       .  E8 48620700         call Finder33.00477288                                   ; \\Finder33.00477288  
> 00401040       .  59                  pop ecx                                                  ;  kernel32.7C816FD7  
> 00401041       .  68 48234800         push Finder33.00482348  
> 00401046       .  6A 00               push 0                                                   ; /pModule = NULL  
> 00401048       .  E8 53010800         call <jmp.&KERNEL32.GetModuleHandleA>                    ; \\GetModuleHandleA  
> 0040104D       .  A3 A7234800         mov dword ptr ds:\[4823A7\],eax  
> 00401052       .  6A 00               push 0  
> 00401054       .  E9 0BB40700         jmp Finder33.0047C464  
> 00401059 Find> $  E9 76620700         jmp Finder33.004772D4  
> 0040105E       .  33C0                xor eax,eax  
> 00401060       .  A0 91234800         mov al,byte ptr ds:\[482391\]  
> 00401065       .  C3                  retn  
>   
> EB 10 66 62 3A 43 2B 2B 48 4F 4F 4B 90 E9 AC 23 48 00 A1 9F 23 48 00 C1 E0 02 A3 A3 23 48 00 52  
> 6A 00 E8 79 01 08 00 8B D0 E8 F6 4F 07 00 5A E8 54 4F 07 00 E8 2B 50 07 00 6A 00 E8 48 62 07 00  
> 59 68 48 23 48 00 6A 00 E8 53 01 08 00 A3 A7 23 48 00 6A 00 E9 0B B4 07 00 E9 76 62 07 00 33 C0  
> A0 91 23 48 00 C3  
>   
> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
> Borland C++  
>   
> CODE  
> DATA  
> .INIT  
> .idata  
> .edata  
> .reloc  
> .rsrc  
>   
> 00401000 BCW.> $  A1 59B05000         mov eax,dword ptr ds:\[50B059\]  
> 00401005       .  C1E0 02             shl eax,2  
> 00401008       .  A3 5DB05000         mov dword ptr ds:\[50B05D\],eax  
> 0040100D       .  57                  push edi                                                 ;  ntdll.7C930738  
> 0040100E       .  51                  push ecx  
> 0040100F       .  33C0                xor eax,eax  
> 00401011       .  BF 84665400         mov edi,BCW.00546684  
> 00401016       .  B9 8C345500         mov ecx,BCW.0055348C  
> 0040101B       .  3BCF                cmp ecx,edi                                              ;  ntdll.7C930738  
> 0040101D       .  76 05               jbe short BCW.00401024  
> 0040101F       .  2BCF                sub ecx,edi                                              ;  ntdll.7C930738  
> 00401021       .  FC                  cld  
> 00401022       .  F3:AA               rep stos byte ptr es:\[edi\]  
> 00401024       >  59                  pop ecx                                                  ;  kernel32.7C816FD7  
> 00401025       .  5F                  pop edi                                                  ;  kernel32.7C816FD7  
> 00401026       .  64:67:8B16 0400     mov edx,dword ptr fs:\[4\]  
> 0040102C       .  8B42 F8             mov eax,dword ptr ds:\[edx-8\]  
> 0040102F       .  A3 61B05000         mov dword ptr ds:\[50B061\],eax  
> 00401034       .  8B42 FC             mov eax,dword ptr ds:\[edx-4\]  
> 00401037       .  A3 65B05000         mov dword ptr ds:\[50B065\],eax  
> 0040103C       .  83EA 04             sub edx,4  
> 0040103F       .  8915 80345500       mov dword ptr ds:\[553480\],edx                            ;  ntdll.KiFastSystemCallRet  
> 00401045       .  83EA 04             sub edx,4  
> 00401048       .  3BD4                cmp edx,esp  
> 0040104A       .  73 02               jnb short BCW.0040104E  
> 0040104C       .  8BE2                mov esp,edx                                              ;  ntdll.KiFastSystemCallRet  
> 0040104E       >  6A 00               push 0                                                   ; /Arg1 = 00000000  
> 00401050       .  E8 45100000         call BCW.0040209A                                        ; \\BCW.0040209A  
> 00401055       .  59                  pop ecx                                                  ;  kernel32.7C816FD7  
> 00401056       .  68 2CB05000         push BCW.0050B02C  
> 0040105B       .  6A 00               push 0                                                   ; /pModule = NULL  
> 0040105D       .  E8 73821000         call <jmp.&KERNEL32.GetModuleHandleA>                    ; \\GetModuleHandleA  
> 00401062       .  A3 6AB05000         mov dword ptr ds:\[50B06A\],eax  
> 00401067       .  6A 00               push 0  
> 00401069       .  E9 93801000         jmp <jmp.&cw3220mt.\_\_startup>  
> 0040106E BCW.> $  E9 09110000         jmp BCW.0040217C  
> 00401073          00                  db 00  
> 00401074          00                  db 00  
> 00401075          00                  db 00  
> 00401076          00                  db 00  
> 00401077          00                  db 00  
> 00401078      /$  55                  push ebp  
> 00401079      |.  8BEC                mov ebp,esp  
> 0040107B      |.  53                  push ebx  
> 0040107C      |.  56                  push esi  
> 0040107D      |.  57                  push edi                                                 ;  ntdll.7C930738  
> 0040107E      |.  8B75 0C             mov esi,dword ptr ss:\[ebp+C\]  
> 00401081      |.  6A 00               push 0                                                   ; /pDefaultCharUsed = NULL  
> 00401083      |.  6A 00               push 0                                                   ; |pDefaultChar = NULL  
> 00401085      |.  6A 00               push 0                                                   ; |MultiByteCount = 0  
> 00401087      |.  6A 00               push 0                                                   ; |MultiByteStr = NULL  
> 00401089      |.  56                  push esi                                                 ; |WideCharCount = FFFFFFFF (-1.)  
> 0040108A      |.  FF75 08             push dword ptr ss:\[ebp+8\]                                ; |WideCharStr = "?",82,"???",82,"?,87,"??暄 T 偎",86,"",85,"?盏????,A1,"",96,"v?",04,"???P???P?漾",B8,"?,86,"",85,"?,85,"????j?",10,"",89,"?",82,"?,98,"旦Ｑ?,10,"",99,"{",82,"?,98,"?肓",10,"?",11,""  
> 0040108D      |.  6A 00               push 0                                                   ; |Options = 0  
> 0040108F      |.  6A 00               push 0                                                   ; |CodePage = CP\_ACP  
> 00401091      |.  E8 99821000         call <jmp.&KERNEL32.WideCharToMultiByte>                 ; \\WideCharToMultiByte  
> 00401096      |.  8BD8                mov ebx,eax  
> 00401098      |.  83FE FF             cmp esi,-1  
> 0040109B      |.  0F95C0              setne al  
> 0040109E      |.  83E0 01             and eax,1  
> 004010A1      |.  03C3                add eax,ebx  
> 004010A3      |.  50                  push eax  
> 004010A4      |.  E8 A47F1000         call <jmp.&cw3220mt.@$bnwa$qui>  
> 004010A9      |.  59                  pop ecx                                                  ;  kernel32.7C816FD7  
> 004010AA      |.  8BF8                mov edi,eax  
> 004010AC      |.  6A 00               push 0                                                   ; /pDefaultCharUsed = NULL  
> 004010AE      |.  6A 00               push 0                                                   ; |pDefaultChar = NULL  
> 004010B0      |.  53                  push ebx                                                 ; |MultiByteCount = 7FFDF000 (2147348480.)  
> 004010B1      |.  57                  push edi                                                 ; |MultiByteStr = ntdll.7C930738  
> 004010B2      |.  56                  push esi                                                 ; |WideCharCount = FFFFFFFF (-1.)  
> 004010B3      |.  FF75 08             push dword ptr ss:\[ebp+8\]                                ; |WideCharStr = "?",82,"???",82,"?,87,"??暄 T 偎",86,"",85,"?盏????,A1,"",96,"v?",04,"???P???P?漾",B8,"?,86,"",85,"?,85,"????j?",10,"",89,"?",82,"?,98,"旦Ｑ?,10,"",99,"{",82,"?,98,"?肓",10,"?",11,""  
> 004010B6      |.  6A 00               push 0                                                   ; |Options = 0  
> 004010B8      |.  6A 00               push 0                                                   ; |CodePage = CP\_ACP  
> 004010BA      |.  E8 70821000         call <jmp.&KERNEL32.WideCharToMultiByte>                 ; \\WideCharToMultiByte  
> 004010BF      |.  8BD8                mov ebx,eax  
> 004010C1      |.  83FE FF             cmp esi,-1  
> 004010C4      |.  74 04               je short BCW.004010CA  
> 004010C6      |.  C6041F 00           mov byte ptr ds:\[edi+ebx\],0  
> 004010CA      |>  8BC7                mov eax,edi                                              ;  ntdll.7C930738  
> 004010CC      |.  5F                  pop edi                                                  ;  kernel32.7C816FD7  
> 004010CD      |.  5E                  pop esi                                                  ;  kernel32.7C816FD7  
> 004010CE      |.  5B                  pop ebx                                                  ;  kernel32.7C816FD7  
> 004010CF      |.  5D                  pop ebp                                                  ;  kernel32.7C816FD7  
> 004010D0      \\.  C3                  retn  
>   
> A1 59 B0 50 00 C1 E0 02 A3 5D B0 50 00 57 51 33 C0 BF 84 66 54 00 B9 8C 34 55 00 3B CF 76 05 2B  
> CF FC F3 AA 59 5F 64 67 8B 16 04 00 8B 42 F8 A3 61 B0 50 00 8B 42 FC A3 65 B0 50 00 83 EA 04 89  
> 15 80 34 55 00 83 EA 04 3B D4 73 02 8B E2 6A 00 E8 45 10 00 00 59 68 2C B0 50 00 6A 00 E8 73 82  
> 10 00 A3 6A B0 50 00 6A 00 E9 93 80 10 00 E9 09 11 00 00 00 00 00 00 00 55 8B EC 53 56 57 8B 75  
> 0C 6A 00 6A 00 6A 00 6A 00 56 FF 75 08 6A 00 6A 00 E8 99 82 10 00 8B D8 83 FE FF 0F 95 C0 83 E0  
> 01 03 C3 50 E8 A4 7F 10 00 59 8B F8 6A 00 6A 00 53 57 56 FF 75 08 6A 00 6A 00 E8 70 82 10 00 8B  
> D8 83 FE FF 74 04 C6 04 1F 00 8B C7 5F 5E 5B 5D C3  
>   
> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
> Borland Delphi 2.0  
>   
> CODE  
> DATA  
> BSS  
> .idata  
> .tls  
> .rdata  
> .reloc  
> .rsrc  
>   
> 00433D9C htes>  55                    push ebp  
> 00433D9D        8BEC                  mov ebp,esp  
> 00433D9F        83C4 F4               add esp,-0C  
> 00433DA2        E8 F1F4FCFF           call htest.00403298  
> 00433DA7        E8 940AFDFF           call htest.00404840  
> 00433DAC        E8 CF3AFDFF           call htest.00407880  
> 00433DB1        E8 92A4FDFF           call htest.0040E248  
> 00433DB6        E8 D1A5FDFF           call htest.0040E38C  
> 00433DBB        E8 28C5FDFF           call htest.004102E8  
> 00433DC0        E8 633CFEFF           call htest.00417A28  
> 00433DC5        E8 CE0BFFFF           call htest.00424998  
> 00433DCA        E8 2956FFFF           call htest.004293F8  
> 00433DCF        E8 1474FFFF           call htest.0042B1E8  
> 00433DD4        E8 C3A2FFFF           call htest.0042E09C  
> 00433DD9        E8 4ED3FFFF           call htest.0043112C  
> 00433DDE        E8 A5E2FFFF           call htest.00432088  
> 00433DE3        A1 28664300           mov eax,dword ptr ds:\[436628\]  
> 00433DE8        E8 1302FFFF           call htest.00424000  
> 00433DED        BA 2C3E4300           mov edx,htest.00433E2C                                   ; ASCII "MP3-2-EXE Player"  
> 00433DF2        A1 28664300           mov eax,dword ptr ds:\[436628\]  
> 00433DF7        E8 20FFFEFF           call htest.00423D1C  
> 00433DFC        B9 A4664300           mov ecx,htest.004366A4  
> 00433E01        BA 04254300           mov edx,htest.00432504  
> 00433E06        A1 28664300           mov eax,dword ptr ds:\[436628\]  
> 00433E0B        E8 0002FFFF           call htest.00424010  
> 00433E10        A1 28664300           mov eax,dword ptr ds:\[436628\]  
> 00433E15        E8 8602FFFF           call htest.004240A0  
> 00433E1A        E8 2904FDFF           call htest.00404248  
> 00433E1F        8BE5                  mov esp,ebp  
> 00433E21        5D                    pop ebp                                                  ; kernel32.7C816FD7  
> 00433E22        C3                    retn  
>   
> 55 8B EC 83 C4 F4 E8 F1 F4 FC FF E8 94 0A FD FF E8 CF 3A FD FF E8 92 A4 FD FF E8 D1 A5 FD FF E8  
> 28 C5 FD FF E8 63 3C FE FF E8 CE 0B FF FF E8 29 56 FF FF E8 14 74 FF FF E8 C3 A2 FF FF E8 4E D3  
> FF FF E8 A5 E2 FF FF A1 28 66 43 00 E8 13 02 FF FF BA 2C 3E 43 00 A1 28 66 43 00 E8 20 FF FE FF  
> B9 A4 66 43 00 BA 04 25 43 00 A1 28 66 43 00 E8 00 02 FF FF A1 28 66 43 00 E8 86 02 FF FF E8 29  
> 04 FD FF 8B E5 5D C3  
>   
> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
> Borland Delphi 3.0  
>   
> CODE  
> DATA  
> BSS  
> .idata  
> .tls  
> .rdata  
> .reloc  
> .rsrc  
>   
> 004ABA30 ResH>  55                    push ebp  
> 004ABA31        8BEC                  mov ebp,esp  
> 004ABA33        83C4 F0               add esp,-10  
> 004ABA36        33C0                  xor eax,eax  
> 004ABA38        8945 F0               mov dword ptr ss:\[ebp-10\],eax  
> 004ABA3B        B8 60B84A00           mov eax,ResHacke.004AB860  
> 004ABA40        E8 B79CF5FF           call ResHacke.004056FC  
> 004ABA45        33C0                  xor eax,eax  
> 004ABA47        55                    push ebp  
> 004ABA48        68 40BB4A00           push ResHacke.004ABB40  
> 004ABA4D        64:FF30               push dword ptr fs:\[eax\]  
> 004ABA50        64:8920               mov dword ptr fs:\[eax\],esp  
> 004ABA53        A1 60DC4A00           mov eax,dword ptr ds:\[4ADC60\]  
> 004ABA58        8B00                  mov eax,dword ptr ds:\[eax\]  
> 004ABA5A        E8 856EF8FF           call ResHacke.004328E4  
> 004ABA5F        A1 60DC4A00           mov eax,dword ptr ds:\[4ADC60\]  
> 004ABA64        8B00                  mov eax,dword ptr ds:\[eax\]  
> 004ABA66        BA 54BB4A00           mov edx,ResHacke.004ABB54                                ; ASCII "Resource Hacker"  
> 004ABA6B        E8 8C6BF8FF           call ResHacke.004325FC  
> 004ABA70        A1 60DC4A00           mov eax,dword ptr ds:\[4ADC60\]  
> 004ABA75        8B00                  mov eax,dword ptr ds:\[eax\]  
> 004ABA77        C640 3F 00            mov byte ptr ds:\[eax+3F\],0  
> 004ABA7B        8B0D D8DA4A00         mov ecx,dword ptr ds:\[4ADAD8\]                            ; ResHacke.004AE81C  
> 004ABA81        A1 60DC4A00           mov eax,dword ptr ds:\[4ADC60\]  
> 004ABA86        8B00                  mov eax,dword ptr ds:\[eax\]  
> 004ABA88        8B15 DCE44900         mov edx,dword ptr ds:\[49E4DC\]                            ; ResHacke.0049E51C  
> 004ABA8E        E8 696EF8FF           call ResHacke.004328FC  
> 004ABA93        E8 786DF5FF           call ResHacke.00402810  
> 004ABA98        48                    dec eax  
> 004ABA99        7E 78                 jle short ResHacke.004ABB13  
> 004ABA9B        8D55 F0               lea edx,dword ptr ss:\[ebp-10\]  
> 004ABA9E        B8 01000000           mov eax,1  
> 004ABAA3        E8 C86DF5FF           call ResHacke.00402870  
> 004ABAA8        8B45 F0               mov eax,dword ptr ss:\[ebp-10\]  
> 004ABAAB        8038 2D               cmp byte ptr ds:\[eax\],2D  
> 004ABAAE        75 63                 jnz short ResHacke.004ABB13  
> 004ABAB0        B2 01                 mov dl,1  
> 004ABAB2        A1 88AE4800           mov eax,dword ptr ds:\[48AE88\]  
> 004ABAB7        E8 B872F5FF           call ResHacke.00402D74  
> 004ABABC        A3 84E84A00           mov dword ptr ds:\[4AE884\],eax  
> 004ABAC1        33C0                  xor eax,eax  
> 004ABAC3        55                    push ebp  
> 004ABAC4        68 F9BA4A00           push ResHacke.004ABAF9  
> 004ABAC9        64:FF30               push dword ptr fs:\[eax\]  
> 004ABACC        64:8920               mov dword ptr fs:\[eax\],esp  
> 004ABACF        8B15 D8DA4A00         mov edx,dword ptr ds:\[4ADAD8\]                            ; ResHacke.004AE81C  
> 004ABAD5        8B12                  mov edx,dword ptr ds:\[edx\]  
> 004ABAD7        A1 84E84A00           mov eax,dword ptr ds:\[4AE884\]  
> 004ABADC        E8 8BF5FDFF           call ResHacke.0048B06C  
> 004ABAE1        33C0                  xor eax,eax  
> 004ABAE3        5A                    pop edx                                                  ; kernel32.7C816FD7  
> 004ABAE4        59                    pop ecx                                                  ; kernel32.7C816FD7  
> 004ABAE5        59                    pop ecx                                                  ; kernel32.7C816FD7  
> 004ABAE6        64:8910               mov dword ptr fs:\[eax\],edx                               ; ntdll.KiFastSystemCallRet  
> 004ABAE9        68 00BB4A00           push ResHacke.004ABB00  
> 004ABAEE        A1 84E84A00           mov eax,dword ptr ds:\[4AE884\]  
> 004ABAF3        E8 A472F5FF           call ResHacke.00402D9C  
> 004ABAF8        C3                    retn  
> 004ABAF9      ^ E9 6A78F5FF           jmp ResHacke.00403368  
> 004ABAFE      ^ EB EE                 jmp short ResHacke.004ABAEE  
> 004ABB00        A1 D8DA4A00           mov eax,dword ptr ds:\[4ADAD8\]  
> 004ABB05        8B00                  mov eax,dword ptr ds:\[eax\]  
> 004ABB07        E8 304DF8FF           call ResHacke.0043083C  
> 004ABB0C        E8 837CF5FF           call ResHacke.00403794  
> 004ABB11        EB 0B                 jmp short ResHacke.004ABB1E  
> 004ABB13        A1 60DC4A00           mov eax,dword ptr ds:\[4ADC60\]  
> 004ABB18        8B00                  mov eax,dword ptr ds:\[eax\]  
> 004ABB1A        C640 3F 01            mov byte ptr ds:\[eax+3F\],1  
> 004ABB1E        A1 60DC4A00           mov eax,dword ptr ds:\[4ADC60\]  
> 004ABB23        8B00                  mov eax,dword ptr ds:\[eax\]  
> 004ABB25        E8 5E6EF8FF           call ResHacke.00432988  
> 004ABB2A        33C0                  xor eax,eax  
> 004ABB2C        5A                    pop edx                                                  ; kernel32.7C816FD7  
> 004ABB2D        59                    pop ecx                                                  ; kernel32.7C816FD7  
> 004ABB2E        59                    pop ecx                                                  ; kernel32.7C816FD7  
> 004ABB2F        64:8910               mov dword ptr fs:\[eax\],edx                               ; ntdll.KiFastSystemCallRet  
> 004ABB32        68 47BB4A00           push ResHacke.004ABB47  
> 004ABB37        8D45 F0               lea eax,dword ptr ss:\[ebp-10\]  
> 004ABB3A        E8 097EF5FF           call ResHacke.00403948  
> 004ABB3F        C3                    retn  
>   
> 55 8B EC 83 C4 F0 33 C0 89 45 F0 B8 60 B8 4A 00 E8 B7 9C F5 FF 33 C0 55 68 40 BB 4A 00 64 FF 30  
> 64 89 20 A1 60 DC 4A 00 8B 00 E8 85 6E F8 FF A1 60 DC 4A 00 8B 00 BA 54 BB 4A 00 E8 8C 6B F8 FF  
> A1 60 DC 4A 00 8B 00 C6 40 3F 00 8B 0D D8 DA 4A 00 A1 60 DC 4A 00 8B 00 8B 15 DC E4 49 00 E8 69  
> 6E F8 FF E8 78 6D F5 FF 48 7E 78 8D 55 F0 B8 01 00 00 00 E8 C8 6D F5 FF 8B 45 F0 80 38 2D 75 63  
> B2 01 A1 88 AE 48 00 E8 B8 72 F5 FF A3 84 E8 4A 00 33 C0 55 68 F9 BA 4A 00 64 FF 30 64 89 20 8B  
> 15 D8 DA 4A 00 8B 12 A1 84 E8 4A 00 E8 8B F5 FD FF 33 C0 5A 59 59 64 89 10 68 00 BB 4A 00 A1 84  
> E8 4A 00 E8 A4 72 F5 FF C3 E9 6A 78 F5 FF EB EE A1 D8 DA 4A 00 8B 00 E8 30 4D F8 FF E8 83 7C F5  
> FF EB 0B A1 60 DC 4A 00 8B 00 C6 40 3F 01 A1 60 DC 4A 00 8B 00 E8 5E 6E F8 FF 33 C0 5A 59 59 64  
> 89 10 68 47 BB 4A 00 8D 45 F0 E8 09 7E F5 FF C3  
>   
> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
> Borland Delphi 4.0 - 5.0  
>   
> CODE  
> DATA  
> BSS  
> .idata  
> .tls  
> .rdata  
> .reloc  
> .rsrc  
>   
> 00457E00 YUCE>  55                    push ebp  
> 00457E01        8BEC                  mov ebp,esp  
> 00457E03        83C4 F4               add esp,-0C  
> 00457E06        B8 C87C4500           mov eax,YUCE.00457CC8  
> 00457E0B        E8 44E4FAFF           call YUCE.00406254  
> 00457E10        A1 048F4500           mov eax,dword ptr ds:\[458F04\]  
> 00457E15        8B00                  mov eax,dword ptr ds:\[eax\]  
> 00457E17        BA 547E4500           mov edx,YUCE.00457E54  
> 00457E1C        E8 83DEFEFF           call YUCE.00445CA4  
> 00457E21        8B0D 688D4500         mov ecx,dword ptr ds:\[458D68\]                            ; YUCE.0045A878  
> 00457E27        A1 048F4500           mov eax,dword ptr ds:\[458F04\]  
> 00457E2C        8B00                  mov eax,dword ptr ds:\[eax\]  
> 00457E2E        8B15 C85B4500         mov edx,dword ptr ds:\[455BC8\]                            ; YUCE.00455C14  
> 00457E34        E8 2FE2FEFF           call YUCE.00446068  
> 00457E39        A1 048F4500           mov eax,dword ptr ds:\[458F04\]  
> 00457E3E        8B00                  mov eax,dword ptr ds:\[eax\]  
> 00457E40        E8 A3E2FEFF           call YUCE.004460E8  
> 00457E45        E8 F6B8FAFF           call YUCE.00403740  
> 00457E4A        0000                  add byte ptr ds:\[eax\],al  
> 00457E4C        FFFF                  ???                                                      ; Unknown command  
> 00457E4E        FFFF                  ???                                                      ; Unknown command  
> 00457E50        0C 00                 or al,0  
> 00457E52        0000                  add byte ptr ds:\[eax\],al  
> 00457E54        C9                    leave  
> 00457E55        FA                    cli  
> 00457E56        C3                    retn  
>   
> 55 8B EC 83 C4 F4 B8 C8 7C 45 00 E8 44 E4 FA FF A1 04 8F 45 00 8B 00 BA 54 7E 45 00 E8 83 DE FE  
> FF 8B 0D 68 8D 45 00 A1 04 8F 45 00 8B 00 8B 15 C8 5B 45 00 E8 2F E2 FE FF A1 04 8F 45 00 8B 00  
> E8 A3 E2 FE FF E8 F6 B8 FA FF 00 00 FF FF FF FF 0C 00 00 00 C9 FA C3  
>   
> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
> Borland Delphi 6.0 - 7.0  
>   
> CODE  
> DATA  
> BSS  
> .idata  
> .tls  
> .rdata  
> .reloc  
> .rsrc  
>   
> 0047845C pymf>  55                    push ebp  
> 0047845D        8BEC                  mov ebp,esp  
> 0047845F        83C4 E8               add esp,-18  
> 00478462        33C0                  xor eax,eax  
> 00478464        8945 EC               mov dword ptr ss:\[ebp-14\],eax  
> 00478467        8945 E8               mov dword ptr ss:\[ebp-18\],eax  
> 0047846A        B8 6C824700           mov eax,pymf.0047826C  
> 0047846F        E8 94E3F8FF           call pymf.00406808  
> 00478474        33C0                  xor eax,eax  
> 00478476        55                    push ebp  
> 00478477        68 21854700           push pymf.00478521  
> 0047847C        64:FF30               push dword ptr fs:\[eax\]  
> 0047847F        64:8920               mov dword ptr fs:\[eax\],esp  
> 00478482        E8 0DD5FFFF           call pymf.00475994  
> 00478487        84C0                  test al,al  
> 00478489        75 18                 jnz short pymf.004784A3  
> 0047848B        6A 24                 push 24  
> 0047848D        68 30854700           push pymf.00478530                                       ; ASCII "Error"  
> 00478492        68 38854700           push pymf.00478538                                       ; ASCII "YMF7x4 driver is not found. Start the program anyway?"  
> 00478497        6A 00                 push 0  
> 00478499        E8 C2ECF8FF           call <jmp.&user32.MessageBoxA>  
> 0047849E        83F8 06               cmp eax,6  
> 004784A1        75 63                 jnz short pymf.00478506  
> 004784A3        8D55 E8               lea edx,dword ptr ss:\[ebp-18\]  
> 004784A6        B8 01000000           mov eax,1  
> 004784AB        E8 BCA5F8FF           call pymf.00402A6C  
> 004784B0        8B45 E8               mov eax,dword ptr ss:\[ebp-18\]  
> 004784B3        8D55 EC               lea edx,dword ptr ss:\[ebp-14\]  
> 004784B6        E8 3900F9FF           call pymf.004084F4  
> 004784BB        8B45 EC               mov eax,dword ptr ss:\[ebp-14\]  
> 004784BE        BA 78854700           mov edx,pymf.00478578                                    ; ASCII "-clean"  
> 004784C3        E8 5CC3F8FF           call pymf.00404824  
> 004784C8        75 07                 jnz short pymf.004784D1  
> 004784CA        E8 B1C7FFFF           call pymf.00474C80  
> 004784CF        EB 35                 jmp short pymf.00478506  
> 004784D1        A1 E8B24700           mov eax,dword ptr ds:\[47B2E8\]  
> 004784D6        8B00                  mov eax,dword ptr ds:\[eax\]  
> 004784D8        E8 A75BFEFF           call pymf.0045E084  
> 004784DD        8B0D 00B14700         mov ecx,dword ptr ds:\[47B100\]                            ; pymf.0047CDC0  
> 004784E3        A1 E8B24700           mov eax,dword ptr ds:\[47B2E8\]  
> 004784E8        8B00                  mov eax,dword ptr ds:\[eax\]  
> 004784EA        8B15 F85B4700         mov edx,dword ptr ds:\[475BF8\]                            ; pymf.00475C44  
> 004784F0        E8 A75BFEFF           call pymf.0045E09C  
> 004784F5        A1 E8B24700           mov eax,dword ptr ds:\[47B2E8\]  
> 004784FA        8B00                  mov eax,dword ptr ds:\[eax\]  
> 004784FC        E8 1B5CFEFF           call pymf.0045E11C  
> 00478501        E8 5ED6FFFF           call pymf.00475B64  
> 00478506        33C0                  xor eax,eax  
> 00478508        5A                    pop edx                                                  ; kernel32.7C816FD7  
> 00478509        59                    pop ecx                                                  ; kernel32.7C816FD7  
> 0047850A        59                    pop ecx                                                  ; kernel32.7C816FD7  
> 0047850B        64:8910               mov dword ptr fs:\[eax\],edx                               ; ntdll.KiFastSystemCallRet  
> 0047850E        68 28854700           push pymf.00478528  
> 00478513        8D45 E8               lea eax,dword ptr ss:\[ebp-18\]  
> 00478516        BA 02000000           mov edx,2  
> 0047851B        E8 2CBFF8FF           call pymf.0040444C  
> 00478520        C3                    retn  
>   
> 55 8B EC 83 C4 E8 33 C0 89 45 EC 89 45 E8 B8 6C 82 47 00 E8 94 E3 F8 FF 33 C0 55 68 21 85 47 00  
> 64 FF 30 64 89 20 E8 0D D5 FF FF 84 C0 75 18 6A 24 68 30 85 47 00 68 38 85 47 00 6A 00 E8 C2 EC  
> F8 FF 83 F8 06 75 63 8D 55 E8 B8 01 00 00 00 E8 BC A5 F8 FF 8B 45 E8 8D 55 EC E8 39 00 F9 FF 8B  
> 45 EC BA 78 85 47 00 E8 5C C3 F8 FF 75 07 E8 B1 C7 FF FF EB 35 A1 E8 B2 47 00 8B 00 E8 A7 5B FE  
> FF 8B 0D 00 B1 47 00 A1 E8 B2 47 00 8B 00 8B 15 F8 5B 47 00 E8 A7 5B FE FF A1 E8 B2 47 00 8B 00  
> E8 1B 5C FE FF E8 5E D6 FF FF 33 C0 5A 59 59 64 89 10 68 28 85 47 00 8D 45 E8 BA 02 00 00 00 E8  
> 2C BF F8 FF C3  
>   
> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
> MASM32 / TASM32  
>   
> .text  
> .rdata  
> .data  
> .rsrc  
>   
> 00401000 RVA.>/$  6A 00               push 0                                                   ; /pModule = NULL  
> 00401002      |.  E8 830A0000         call <jmp.&KERNEL32.GetModuleHandleA>                    ; \\GetModuleHandleA  
> 00401007      |.  A3 07304000         mov dword ptr ds:\[403007\],eax  
> 0040100C      |.  6A 00               push 0                                                   ; /lParam = NULL  
> 0040100E      |.  68 6C104000         push RVA.0040106C                                        ; |DlgProc = RVA.0040106C  
> 00401013      |.  6A 00               push 0                                                   ; |hOwner = NULL  
> 00401015      |.  68 00304000         push RVA.00403000                                        ; |pTemplate = "DIALOG"  
> 0040101A      |.  FF35 07304000       push dword ptr ds:\[403007\]                               ; |hInst = NULL  
> 00401020      |.  E8 F9090000         call <jmp.&USER32.DialogBoxParamA>                       ; \\DialogBoxParamA  
> 00401025      |.  50                  push eax                                                 ; /ExitCode = 0  
> 00401026      \\.  E8 4D0A0000         call <jmp.&KERNEL32.ExitProcess>                         ; \\ExitProcess  
> 0040102B      /.  55                  push ebp  
> 0040102C      |.  8BEC                mov ebp,esp  
> 0040102E      |.  837D 0C 10          cmp dword ptr ss:\[ebp+C\],10  
> 00401032      |.  75 0C               jnz short RVA.00401040  
> 00401034      |.  6A 00               push 0                                                   ; /Result = 0  
> 00401036      |.  FF75 08             push dword ptr ss:\[ebp+8\]                                ; |hWnd = 00401000  
> 00401039      |.  E8 EC090000         call <jmp.&USER32.EndDialog>                             ; \\EndDialog  
> 0040103E      |.  EB 26               jmp short RVA.00401066  
> 00401040      |>  817D 0C 11010000    cmp dword ptr ss:\[ebp+C\],111  
> 00401047      |.  75 1D               jnz short RVA.00401066  
> 00401049      |.  8B45 10             mov eax,dword ptr ss:\[ebp+10\]  
> 0040104C      |.  66:3D B80B          cmp ax,0BB8  
> 00401050      |.  75 14               jnz short RVA.00401066  
> 00401052      |.  C1E8 10             shr eax,10  
> 00401055      |.  66:0BC0             or ax,ax  
> 00401058      |.  75 0A               jnz short RVA.00401064  
> 0040105A      |.  6A 00               push 0                                                   ; /Result = 0  
> 0040105C      |.  FF75 08             push dword ptr ss:\[ebp+8\]                                ; |hWnd = 00401000  
> 0040105F      |.  E8 C6090000         call <jmp.&USER32.EndDialog>                             ; \\EndDialog  
> 00401064      |>  EB 00               jmp short RVA.00401066  
> 00401066      |>  33C0                xor eax,eax  
> 00401068      |.  C9                  leave  
> 00401069      \\.  C2 1000             retn 10  
>   
> 6A 00 E8 83 0A 00 00 A3 07 30 40 00 6A 00 68 6C 10 40 00 6A 00 68 00 30 40 00 FF 35 07 30 40 00  
> E8 F9 09 00 00 50 E8 4D 0A 00 00 55 8B EC 83 7D 0C 10 75 0C 6A 00 FF 75 08 E8 EC 09 00 00 EB 26  
> 81 7D 0C 11 01 00 00 75 1D 8B 45 10 66 3D B8 0B 75 14 C1 E8 10 66 0B C0 75 0A 6A 00 FF 75 08 E8  
> C6 09 00 00 EB 00 33 C0 C9 C2 10 00  
>   
> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
> Microsoft Visual Basic 5.0 / 6.0  
>   
> .text  
> .data  
> .rsrc  
>   
> 00402360 Kill>/$  68 2C4D4000         push KillBox.00404D2C                                    ;  ASCII "VB5!6&\*"  
> 00402365      |.  E8 EEFFFFFF         call <jmp.&MSVBVM60.#100>  
> 0040236A      |.  0000                add byte ptr ds:\[eax\],al  
> 0040236C      |.  0000                add byte ptr ds:\[eax\],al  
> 0040236E      |.  0000                add byte ptr ds:\[eax\],al  
> 00402370      |.  3000                xor byte ptr ds:\[eax\],al  
> 00402372      |.  0000                add byte ptr ds:\[eax\],al  
> 00402374      |.  3800                cmp byte ptr ds:\[eax\],al  
> 00402376      |.  0000                add byte ptr ds:\[eax\],al  
> 00402378      |.  0000                add byte ptr ds:\[eax\],al  
> 0040237A      |.  0000                add byte ptr ds:\[eax\],al  
> 0040237C      |.  4F                  dec edi                                                  ;  ntdll.7C930738  
> 0040237D      \\.  C2 F150             retn 50F1  
>   
> 68 2C 4D 40 00 E8 EE FF FF FF 00 00 00 00 00 00 30 00 00 00 38 00 00 00 00 00 00 00 4F C2 F1 50  
>   
> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
> Microsoft Visual C++ 4.x  
>   
> .text  
> .rdata  
> .data  
> .idata  
> .reloc  
>   
> 00401CC0 memt> $  64:A1 00000000      mov eax,dword ptr fs:\[0\]  
> 00401CC6       .  55                  push ebp  
> 00401CC7       .  8BEC                mov ebp,esp  
> 00401CC9       .  6A FF               push -1  
> 00401CCB       .  68 28804000         push memtest.00408028  
> 00401CD0       .  68 001B4000         push memtest.00401B00  
> 00401CD5       .  50                  push eax  
> 00401CD6       .  64:8925 00000000    mov dword ptr fs:\[0\],esp  
> 00401CDD       .  83EC 60             sub esp,60  
> 00401CE0       .  53                  push ebx  
> 00401CE1       .  56                  push esi  
> 00401CE2       .  57                  push edi                                                 ;  ntdll.7C930738  
> 00401CE3       .  8965 E8             mov dword ptr ss:\[ebp-18\],esp  
> 00401CE6       .  FF15 30D14000       call near dword ptr ds:\[<&KERNEL32.GetVersion>\]          ;  kernel32.GetVersion  
> 00401CEC       .  A3 B89A4000         mov dword ptr ds:\[409AB8\],eax  
> 00401CF1       .  33C0                xor eax,eax  
> 00401CF3       .  A0 B99A4000         mov al,byte ptr ds:\[409AB9\]  
> 00401CF8       .  A3 C49A4000         mov dword ptr ds:\[409AC4\],eax  
> 00401CFD       .  A1 B89A4000         mov eax,dword ptr ds:\[409AB8\]  
> 00401D02       .  C12D B89A4000 10    shr dword ptr ds:\[409AB8\],10  
> 00401D09       .  25 FF000000         and eax,0FF  
> 00401D0E       .  A3 C09A4000         mov dword ptr ds:\[409AC0\],eax  
> 00401D13       .  C1E0 08             shl eax,8  
> 00401D16       .  0305 C49A4000       add eax,dword ptr ds:\[409AC4\]  
> 00401D1C       .  A3 BC9A4000         mov dword ptr ds:\[409ABC\],eax  
> 00401D21       .  E8 6A010000         call memtest.00401E90  
> 00401D26       .  85C0                test eax,eax  
> 00401D28       .  75 0A               jnz short memtest.00401D34  
> 00401D2A       .  6A 1C               push 1C  
> 00401D2C       .  E8 2F010000         call memtest.00401E60  
> 00401D31       .  83C4 04             add esp,4  
> 00401D34       >  C745 FC 00000000    mov dword ptr ss:\[ebp-4\],0  
> 00401D3B       .  E8 00260000         call memtest.00404340  
> 00401D40       .  E8 EB250000         call memtest.00404330  
> 00401D45       .  FF15 2CD14000       call near dword ptr ds:\[<&KERNEL32.GetCommandLineA>\]     ; \[GetCommandLineA  
> 00401D4B       .  A3 80CD4000         mov dword ptr ds:\[40CD80\],eax  
> 00401D50       .  E8 8B210000         call memtest.00403EE0  
> 00401D55       .  A3 3C924000         mov dword ptr ds:\[40923C\],eax  
> 00401D5A       .  85C0                test eax,eax  
> 00401D5C       .  74 09               je short memtest.00401D67  
> 00401D5E       .  833D 80CD4000 00    cmp dword ptr ds:\[40CD80\],0  
> 00401D65       .  75 0A               jnz short memtest.00401D71  
> 00401D67       >  6A FF               push -1  
> 00401D69       .  E8 021B0000         call memtest.00403870  
> 00401D6E       .  83C4 04             add esp,4  
> 00401D71       >  E8 EA1E0000         call memtest.00403C60  
> 00401D76       .  E8 F51D0000         call memtest.00403B70  
> 00401D7B       .  E8 C01A0000         call memtest.00403840  
> 00401D80       .  8B35 80CD4000       mov esi,dword ptr ds:\[40CD80\]  
> 00401D86       .  8A06                mov al,byte ptr ds:\[esi\]  
> 00401D88       .  3C 22               cmp al,22  
> 00401D8A       .  74 0C               je short memtest.00401D98  
> 00401D8C       .  3C 20               cmp al,20  
> 00401D8E       .  76 35               jbe short memtest.00401DC5  
> 00401D90       >  46                  inc esi  
> 00401D91       .  803E 20             cmp byte ptr ds:\[esi\],20  
> 00401D94       .^ 77 FA               ja short memtest.00401D90  
> 00401D96       .  EB 2D               jmp short memtest.00401DC5  
> 00401D98       >  46                  inc esi  
> 00401D99       .  803E 22             cmp byte ptr ds:\[esi\],22  
> 00401D9C       .  74 26               je short memtest.00401DC4  
> 00401D9E       .  8A5D D8             mov bl,byte ptr ss:\[ebp-28\]  
> 00401DA1       >  8A1E                mov bl,byte ptr ds:\[esi\]  
> 00401DA3       .  84DB                test bl,bl  
> 00401DA5       .  74 18               je short memtest.00401DBF  
> 00401DA7       .  33C0                xor eax,eax  
> 00401DA9       .  8AC3                mov al,bl  
> 00401DAB       .  50                  push eax  
> 00401DAC       .  E8 5F1D0000         call memtest.00403B10  
> 00401DB1       .  83C4 04             add esp,4  
> 00401DB4       .  85C0                test eax,eax  
> 00401DB6       .  74 01               je short memtest.00401DB9  
> 00401DB8       .  46                  inc esi  
> 00401DB9       >  46                  inc esi  
> 00401DBA       .  803E 22             cmp byte ptr ds:\[esi\],22  
> 00401DBD       .^ 75 E2               jnz short memtest.00401DA1  
> 00401DBF       >  803E 22             cmp byte ptr ds:\[esi\],22  
> 00401DC2       .  75 01               jnz short memtest.00401DC5  
> 00401DC4       >  46                  inc esi  
> 00401DC5       >  803E 00             cmp byte ptr ds:\[esi\],0  
> 00401DC8       .  74 0B               je short memtest.00401DD5  
> 00401DCA       >  803E 20             cmp byte ptr ds:\[esi\],20  
> 00401DCD       .  77 06               ja short memtest.00401DD5  
> 00401DCF       .  46                  inc esi  
> 00401DD0       .  803E 00             cmp byte ptr ds:\[esi\],0  
> 00401DD3       .^ 75 F5               jnz short memtest.00401DCA  
> 00401DD5       >  C745 BC 00000000    mov dword ptr ss:\[ebp-44\],0  
> 00401DDC       .  8D45 90             lea eax,dword ptr ss:\[ebp-70\]  
> 00401DDF       .  50                  push eax                                                 ; /pStartupinfo = NULL  
> 00401DE0       .  FF15 28D14000       call near dword ptr ds:\[<&KERNEL32.GetStartupInfoA>\]     ; \\GetStartupInfoA  
> 00401DE6       .  F645 BC 01          test byte ptr ss:\[ebp-44\],1  
> 00401DEA       .  B8 0A000000         mov eax,0A  
> 00401DEF       .  74 08               je short memtest.00401DF9  
> 00401DF1       .  8B45 C0             mov eax,dword ptr ss:\[ebp-40\]                            ;  ntdll.7C92E64E  
> 00401DF4       .  25 FFFF0000         and eax,0FFFF  
> 00401DF9       >  50                  push eax                                                 ; /Arg4 = 00000000  
> 00401DFA       .  56                  push esi                                                 ; |Arg3 = FFFFFFFF  
> 00401DFB       .  6A 00               push 0                                                   ; |Arg2 = 00000000  
> 00401DFD       .  6A 00               push 0                                                   ; |/pModule = NULL  
> 00401DFF       .  FF15 24D14000       call near dword ptr ds:\[<&KERNEL32.GetModuleHandleA>\]    ; |\\GetModuleHandleA  
> 00401E05       .  50                  push eax                                                 ; |Arg1 = 00000000  
> 00401E06       .  E8 F5F1FFFF         call memtest.00401000                                    ; \\memtest.00401000  
> 00401E0B       .  50                  push eax  
> 00401E0C       .  E8 5F1A0000         call memtest.00403870  
> 00401E11       .  EB 27               jmp short memtest.00401E3A  
> 00401E13       .  8B45 EC             mov eax,dword ptr ss:\[ebp-14\]  
> 00401E16       .  8B00                mov eax,dword ptr ds:\[eax\]  
> 00401E18       .  8B00                mov eax,dword ptr ds:\[eax\]  
> 00401E1A       .  8945 E0             mov dword ptr ss:\[ebp-20\],eax  
> 00401E1D       .  8B45 EC             mov eax,dword ptr ss:\[ebp-14\]  
> 00401E20       .  50                  push eax  
> 00401E21       .  8B45 E0             mov eax,dword ptr ss:\[ebp-20\]  
> 00401E24       .  50                  push eax  
> 00401E25       .  E8 561B0000         call memtest.00403980  
> 00401E2A       .  83C4 08             add esp,8  
> 00401E2D       .  C3                  retn  
>   
> 64 A1 00 00 00 00 55 8B EC 6A FF 68 28 80 40 00 68 00 1B 40 00 50 64 89 25 00 00 00 00 83 EC 60  
> 53 56 57 89 65 E8 FF 15 30 D1 40 00 A3 B8 9A 40 00 33 C0 A0 B9 9A 40 00 A3 C4 9A 40 00 A1 B8 9A  
> 40 00 C1 2D B8 9A 40 00 10 25 FF 00 00 00 A3 C0 9A 40 00 C1 E0 08 03 05 C4 9A 40 00 A3 BC 9A 40  
> 00 E8 6A 01 00 00 85 C0 75 0A 6A 1C E8 2F 01 00 00 83 C4 04 C7 45 FC 00 00 00 00 E8 00 26 00 00  
> E8 EB 25 00 00 FF 15 2C D1 40 00 A3 80 CD 40 00 E8 8B 21 00 00 A3 3C 92 40 00 85 C0 74 09 83 3D  
> 80 CD 40 00 00 75 0A 6A FF E8 02 1B 00 00 83 C4 04 E8 EA 1E 00 00 E8 F5 1D 00 00 E8 C0 1A 00 00  
> 8B 35 80 CD 40 00 8A 06 3C 22 74 0C 3C 20 76 35 46 80 3E 20 77 FA EB 2D 46 80 3E 22 74 26 8A 5D  
> D8 8A 1E 84 DB 74 18 33 C0 8A C3 50 E8 5F 1D 00 00 83 C4 04 85 C0 74 01 46 46 80 3E 22 75 E2 80  
> 3E 22 75 01 46 80 3E 00 74 0B 80 3E 20 77 06 46 80 3E 00 75 F5 C7 45 BC 00 00 00 00 8D 45 90 50  
> FF 15 28 D1 40 00 F6 45 BC 01 B8 0A 00 00 00 74 08 8B 45 C0 25 FF FF 00 00 50 56 6A 00 6A 00 FF  
> 15 24 D1 40 00 50 E8 F5 F1 FF FF 50 E8 5F 1A 00 00 EB 27 8B 45 EC 8B 00 8B 00 89 45 E0 8B 45 EC  
> 50 8B 45 E0 50 E8 56 1B 00 00 83 C4 08 C3  
>   
> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
> Microsoft Visual C++ 5.0  
>   
> .text  
> .rdata  
> .data  
> .rsrc  
>   
> 0040B060 HEdi> $  55                  push ebp  
> 0040B061       .  8BEC                mov ebp,esp  
> 0040B063       .  6A FF               push -1  
> 0040B065       .  68 C8264400         push HEdit.004426C8  
> 0040B06A       .  68 38E24000         push HEdit.0040E238                                      ;  SE handler installation  
> 0040B06F       .  64:A1 00000000      mov eax,dword ptr fs:\[0\]  
> 0040B075       .  50                  push eax  
> 0040B076       .  64:8925 00000000    mov dword ptr fs:\[0\],esp  
> 0040B07D       .  83C4 A8             add esp,-58  
> 0040B080       .  53                  push ebx  
> 0040B081       .  56                  push esi  
> 0040B082       .  57                  push edi                                                 ;  ntdll.7C930738  
> 0040B083       .  8965 E8             mov dword ptr ss:\[ebp-18\],esp  
> 0040B086       .  FF15 F0C14300       call near dword ptr ds:\[<&KERNEL32.GetVersion>\]          ;  kernel32.GetVersion  
> 0040B08C       .  33D2                xor edx,edx                                              ;  ntdll.KiFastSystemCallRet  
> 0040B08E       .  8AD4                mov dl,ah  
> 0040B090       .  8915 A0174500       mov dword ptr ds:\[4517A0\],edx                            ;  ntdll.KiFastSystemCallRet  
> 0040B096       .  8BC8                mov ecx,eax  
> 0040B098       .  81E1 FF000000       and ecx,0FF  
> 0040B09E       .  890D 9C174500       mov dword ptr ds:\[45179C\],ecx  
> 0040B0A4       .  C1E1 08             shl ecx,8  
> 0040B0A7       .  03CA                add ecx,edx                                              ;  ntdll.KiFastSystemCallRet  
> 0040B0A9       .  890D 98174500       mov dword ptr ds:\[451798\],ecx  
> 0040B0AF       .  C1E8 10             shr eax,10  
> 0040B0B2       .  A3 94174500         mov dword ptr ds:\[451794\],eax  
> 0040B0B7       .  E8 F45C0000         call HEdit.00410DB0  
> 0040B0BC       .  85C0                test eax,eax  
> 0040B0BE       .  75 0A               jnz short HEdit.0040B0CA  
> 0040B0C0       .  6A 1C               push 1C  
> 0040B0C2       .  E8 79010000         call HEdit.0040B240  
> 0040B0C7       .  83C4 04             add esp,4  
> 0040B0CA       >  E8 11290000         call HEdit.0040D9E0  
> 0040B0CF       .  85C0                test eax,eax  
> 0040B0D1       .  75 0A               jnz short HEdit.0040B0DD  
> 0040B0D3       .  6A 10               push 10  
> 0040B0D5       .  E8 66010000         call HEdit.0040B240  
> 0040B0DA       .  83C4 04             add esp,4  
> 0040B0DD       >  C745 FC 00000000    mov dword ptr ss:\[ebp-4\],0  
> 0040B0E4       .  E8 B75A0000         call HEdit.00410BA0  
> 0040B0E9       .  E8 821B0000         call HEdit.0040CC70  
> 0040B0EE       .  FF15 70C24300       call near dword ptr ds:\[<&KERNEL32.GetCommandLineA>\]     ; \[GetCommandLineA  
> 0040B0F4       .  A3 D02E4500         mov dword ptr ds:\[452ED0\],eax  
> 0040B0F9       .  E8 42590000         call HEdit.00410A40  
> 0040B0FE       .  A3 DC174500         mov dword ptr ds:\[4517DC\],eax  
> 0040B103       .  85C0                test eax,eax  
> 0040B105       .  74 09               je short HEdit.0040B110  
> 0040B107       .  A1 D02E4500         mov eax,dword ptr ds:\[452ED0\]  
> 0040B10C       .  85C0                test eax,eax  
> 0040B10E       .  75 0A               jnz short HEdit.0040B11A  
> 0040B110       >  6A FF               push -1  
> 0040B112       .  E8 C9F7FFFF         call HEdit.0040A8E0  
> 0040B117       .  83C4 04             add esp,4  
> 0040B11A       >  E8 71560000         call HEdit.00410790  
> 0040B11F       .  E8 7C550000         call HEdit.004106A0  
> 0040B124       .  E8 87F7FFFF         call HEdit.0040A8B0  
> 0040B129       .  8B35 D02E4500       mov esi,dword ptr ds:\[452ED0\]  
> 0040B12F       .  8975 9C             mov dword ptr ss:\[ebp-64\],esi  
> 0040B132       .  803E 22             cmp byte ptr ds:\[esi\],22  
> 0040B135       .  0F85 BE000000       jnz HEdit.0040B1F9  
> 0040B13B       >  46                  inc esi  
> 0040B13C       .  8975 9C             mov dword ptr ss:\[ebp-64\],esi  
> 0040B13F       .  8A06                mov al,byte ptr ds:\[esi\]  
> 0040B141       .  3C 22               cmp al,22  
> 0040B143       .  74 1C               je short HEdit.0040B161  
> 0040B145       .  84C0                test al,al  
> 0040B147       .  74 18               je short HEdit.0040B161  
> 0040B149       .  25 FF000000         and eax,0FF  
> 0040B14E       .  50                  push eax  
> 0040B14F       .  E8 EC540000         call HEdit.00410640  
> 0040B154       .  83C4 04             add esp,4  
> 0040B157       .  85C0                test eax,eax  
> 0040B159       .^ 74 E0               je short HEdit.0040B13B  
> 0040B15B       .  46                  inc esi  
> 0040B15C       .  8975 9C             mov dword ptr ss:\[ebp-64\],esi  
> 0040B15F       .^ EB DA               jmp short HEdit.0040B13B  
> 0040B161       >  803E 22             cmp byte ptr ds:\[esi\],22  
> 0040B164       .  75 04               jnz short HEdit.0040B16A  
> 0040B166       .  46                  inc esi  
> 0040B167       .  8975 9C             mov dword ptr ss:\[ebp-64\],esi  
> 0040B16A       >  8A06                mov al,byte ptr ds:\[esi\]  
> 0040B16C       .  84C0                test al,al  
> 0040B16E       .  74 0A               je short HEdit.0040B17A  
> 0040B170       .  3C 20               cmp al,20  
> 0040B172       .  77 06               ja short HEdit.0040B17A  
> 0040B174       .  46                  inc esi  
> 0040B175       .  8975 9C             mov dword ptr ss:\[ebp-64\],esi  
> 0040B178       .^ EB F0               jmp short HEdit.0040B16A  
> 0040B17A       >  C745 D0 00000000    mov dword ptr ss:\[ebp-30\],0  
> 0040B181       .  8D45 A4             lea eax,dword ptr ss:\[ebp-5C\]  
> 0040B184       .  50                  push eax                                                 ; /pStartupinfo = NULL  
> 0040B185       .  FF15 68C24300       call near dword ptr ds:\[<&KERNEL32.GetStartupInfoA>\]     ; \\GetStartupInfoA  
> 0040B18B       .  F645 D0 01          test byte ptr ss:\[ebp-30\],1  
> 0040B18F       .  74 0A               je short HEdit.0040B19B  
> 0040B191       .  8B45 D4             mov eax,dword ptr ss:\[ebp-2C\]                            ;  kernel32.7C816FD7  
> 0040B194       .  25 FFFF0000         and eax,0FFFF  
> 0040B199       .  EB 05               jmp short HEdit.0040B1A0  
> 0040B19B       >  B8 0A000000         mov eax,0A  
> 0040B1A0       >  50                  push eax  
> 0040B1A1       .  56                  push esi  
> 0040B1A2       .  6A 00               push 0  
> 0040B1A4       .  6A 00               push 0                                                   ; /pModule = NULL  
> 0040B1A6       .  FF15 78C24300       call near dword ptr ds:\[<&KERNEL32.GetModuleHandleA>\]    ; \\GetModuleHandleA  
> 0040B1AC       .  50                  push eax  
> 0040B1AD       .  E8 B5FC0000         call HEdit.0041AE67  
> 0040B1B2       .  8945 A0             mov dword ptr ss:\[ebp-60\],eax  
> 0040B1B5       .  50                  push eax  
> 0040B1B6       .  E8 25F7FFFF         call HEdit.0040A8E0  
> 0040B1BB       .  EB 21               jmp short HEdit.0040B1DE  
> 0040B1BD       .  8B45 EC             mov eax,dword ptr ss:\[ebp-14\]  
> 0040B1C0       .  8B08                mov ecx,dword ptr ds:\[eax\]  
> 0040B1C2       .  8B09                mov ecx,dword ptr ds:\[ecx\]                               ;  ntdll.7C92E64E  
> 0040B1C4       .  894D 98             mov dword ptr ss:\[ebp-68\],ecx  
> 0040B1C7       .  50                  push eax  
> 0040B1C8       .  51                  push ecx  
> 0040B1C9       .  E8 22520000         call HEdit.004103F0  
> 0040B1CE       .  83C4 08             add esp,8  
> 0040B1D1       .  C3                  retn  
>   
> 55 8B EC 6A FF 68 C8 26 44 00 68 38 E2 40 00 64 A1 00 00 00 00 50 64 89 25 00 00 00 00 83 C4 A8  
> 53 56 57 89 65 E8 FF 15 F0 C1 43 00 33 D2 8A D4 89 15 A0 17 45 00 8B C8 81 E1 FF 00 00 00 89 0D  
> 9C 17 45 00 C1 E1 08 03 CA 89 0D 98 17 45 00 C1 E8 10 A3 94 17 45 00 E8 F4 5C 00 00 85 C0 75 0A  
> 6A 1C E8 79 01 00 00 83 C4 04 E8 11 29 00 00 85 C0 75 0A 6A 10 E8 66 01 00 00 83 C4 04 C7 45 FC  
> 00 00 00 00 E8 B7 5A 00 00 E8 82 1B 00 00 FF 15 70 C2 43 00 A3 D0 2E 45 00 E8 42 59 00 00 A3 DC  
> 17 45 00 85 C0 74 09 A1 D0 2E 45 00 85 C0 75 0A 6A FF E8 C9 F7 FF FF 83 C4 04 E8 71 56 00 00 E8  
> 7C 55 00 00 E8 87 F7 FF FF 8B 35 D0 2E 45 00 89 75 9C 80 3E 22 0F 85 BE 00 00 00 46 89 75 9C 8A  
> 06 3C 22 74 1C 84 C0 74 18 25 FF 00 00 00 50 E8 EC 54 00 00 83 C4 04 85 C0 74 E0 46 89 75 9C EB  
> DA 80 3E 22 75 04 46 89 75 9C 8A 06 84 C0 74 0A 3C 20 77 06 46 89 75 9C EB F0 C7 45 D0 00 00 00  
> 00 8D 45 A4 50 FF 15 68 C2 43 00 F6 45 D0 01 74 0A 8B 45 D4 25 FF FF 00 00 EB 05 B8 0A 00 00 00  
> 50 56 6A 00 6A 00 FF 15 78 C2 43 00 50 E8 B5 FC 00 00 89 45 A0 50 E8 25 F7 FF FF EB 21 8B 45 EC  
> 8B 08 8B 09 89 4D 98 50 51 E8 22 52 00 00 83 C4 08 C3  
>   
> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
> Microsoft Visual C++ 6.0 \[Debug\]  
>   
> .text  
> .rdata  
> .data  
> .rsrc  
>   
> 005522F3 Baby>/$  55                  push ebp  
> 005522F4      |.  8BEC                mov ebp,esp  
> 005522F6      |.  6A FF               push -1  
> 005522F8      |.  68 58235800         push Babylon.00582358  
> 005522FD      |.  68 6C5B5500         push Babylon.00555B6C                                    ;  SE handler installation  
> 00552302      |.  64:A1 00000000      mov eax,dword ptr fs:\[0\]  
> 00552308      |.  50                  push eax  
> 00552309      |.  64:8925 00000000    mov dword ptr fs:\[0\],esp  
> 00552310      |.  83EC 58             sub esp,58  
> 00552313      |.  53                  push ebx  
> 00552314      |.  56                  push esi  
> 00552315      |.  57                  push edi                                                 ;  ntdll.7C930738  
> 00552316      |.  8965 E8             mov dword ptr ss:\[ebp-18\],esp  
> 00552319      |.  FF15 00635700       call near dword ptr ds:\[<&KERNEL32.GetVersion>\]          ;  kernel32.GetVersion  
> 0055231F      |.  33D2                xor edx,edx                                              ;  ntdll.KiFastSystemCallRet  
> 00552321      |.  8AD4                mov dl,ah  
> 00552323      |.  8915 00BF5D00       mov dword ptr ds:\[5DBF00\],edx                            ;  ntdll.KiFastSystemCallRet  
> 00552329      |.  8BC8                mov ecx,eax  
> 0055232B      |.  81E1 FF000000       and ecx,0FF  
> 00552331      |.  890D FCBE5D00       mov dword ptr ds:\[5DBEFC\],ecx  
> 00552337      |.  C1E1 08             shl ecx,8  
> 0055233A      |.  03CA                add ecx,edx                                              ;  ntdll.KiFastSystemCallRet  
> 0055233C      |.  890D F8BE5D00       mov dword ptr ds:\[5DBEF8\],ecx  
> 00552342      |.  C1E8 10             shr eax,10  
> 00552345      |.  A3 F4BE5D00         mov dword ptr ds:\[5DBEF4\],eax  
> 0055234A      |.  6A 01               push 1  
> 0055234C      |.  E8 391D0000         call Babylon.0055408A  
> 00552351      |.  59                  pop ecx                                                  ;  kernel32.7C816FD7  
> 00552352      |.  85C0                test eax,eax  
> 00552354      |.  75 08               jnz short Babylon.0055235E  
> 00552356      |.  6A 1C               push 1C  
> 00552358      |.  E8 C3000000         call Babylon.00552420  
> 0055235D      |.  59                  pop ecx                                                  ;  kernel32.7C816FD7  
> 0055235E      |>  E8 99350000         call Babylon.005558FC  
> 00552363      |.  85C0                test eax,eax  
> 00552365      |.  75 08               jnz short Babylon.0055236F  
> 00552367      |.  6A 10               push 10  
> 00552369      |.  E8 B2000000         call Babylon.00552420  
> 0055236E      |.  59                  pop ecx                                                  ;  kernel32.7C816FD7  
> 0055236F      |>  33F6                xor esi,esi  
> 00552371      |.  8975 FC             mov dword ptr ss:\[ebp-4\],esi  
> 00552374      |.  E8 1B840000         call Babylon.0055A794  
> 00552379      |.  FF15 0C625700       call near dword ptr ds:\[<&KERNEL32.GetCommandLineA>\]     ; \[GetCommandLineA  
> 0055237F      |.  A3 08D85D00         mov dword ptr ds:\[5DD808\],eax  
> 00552384      |.  E8 1B8C0000         call Babylon.0055AFA4  
> 00552389      |.  A3 7CBE5D00         mov dword ptr ds:\[5DBE7C\],eax  
> 0055238E      |.  E8 C4890000         call Babylon.0055AD57  
> 00552393      |.  E8 06890000         call Babylon.0055AC9E  
> 00552398      |.  E8 A7380000         call Babylon.00555C44  
> 0055239D      |.  8975 D0             mov dword ptr ss:\[ebp-30\],esi  
> 005523A0      |.  8D45 A4             lea eax,dword ptr ss:\[ebp-5C\]  
> 005523A3      |.  50                  push eax                                                 ; /pStartupinfo = NULL  
> 005523A4      |.  FF15 18625700       call near dword ptr ds:\[<&KERNEL32.GetStartupInfoA>\]     ; \\GetStartupInfoA  
> 005523AA      |.  E8 97880000         call Babylon.0055AC46  
> 005523AF      |.  8945 9C             mov dword ptr ss:\[ebp-64\],eax  
> 005523B2      |.  F645 D0 01          test byte ptr ss:\[ebp-30\],1  
> 005523B6      |.  74 06               je short Babylon.005523BE  
> 005523B8      |.  0FB745 D4           movzx eax,word ptr ss:\[ebp-2C\]  
> 005523BC      |.  EB 03               jmp short Babylon.005523C1  
> 005523BE      |>  6A 0A               push 0A  
> 005523C0      |.  58                  pop eax                                                  ;  kernel32.7C816FD7  
> 005523C1      |>  50                  push eax                                                 ; /Arg4 = 00000000  
> 005523C2      |.  FF75 9C             push dword ptr ss:\[ebp-64\]                               ; |Arg3 = 00000001  
> 005523C5      |.  56                  push esi                                                 ; |Arg2 = FFFFFFFF  
> 005523C6      |.  56                  push esi                                                 ; |/pModule = FFFFFFFF ???  
> 005523C7      |.  FF15 68645700       call near dword ptr ds:\[<&KERNEL32.GetModuleHandleA>\]    ; |\\GetModuleHandleA  
> 005523CD      |.  50                  push eax                                                 ; |Arg1 = 00000000  
> 005523CE      |.  E8 2DC0EBFF         call Babylon.0040E400                                    ; \\Babylon.0040E400  
> 005523D3      |.  8945 A0             mov dword ptr ss:\[ebp-60\],eax  
> 005523D6      |.  50                  push eax  
> 005523D7      |.  E8 95380000         call Babylon.00555C71  
> 005523DC      |.  8B45 EC             mov eax,dword ptr ss:\[ebp-14\]  
> 005523DF      |.  8B08                mov ecx,dword ptr ds:\[eax\]  
> 005523E1      |.  8B09                mov ecx,dword ptr ds:\[ecx\]                               ;  ntdll.7C92E64E  
> 005523E3      |.  894D 98             mov dword ptr ss:\[ebp-68\],ecx  
> 005523E6      |.  50                  push eax  
> 005523E7      |.  51                  push ecx  
> 005523E8      |.  E8 68550000         call Babylon.00557955  
> 005523ED      |.  59                  pop ecx                                                  ;  kernel32.7C816FD7  
> 005523EE      |.  59                  pop ecx                                                  ;  kernel32.7C816FD7  
> 005523EF      \\.  C3                  retn  
>   
> 55 8B EC 6A FF 68 58 23 58 00 68 6C 5B 55 00 64 A1 00 00 00 00 50 64 89 25 00 00 00 00 83 EC 58  
> 53 56 57 89 65 E8 FF 15 00 63 57 00 33 D2 8A D4 89 15 00 BF 5D 00 8B C8 81 E1 FF 00 00 00 89 0D  
> FC BE 5D 00 C1 E1 08 03 CA 89 0D F8 BE 5D 00 C1 E8 10 A3 F4 BE 5D 00 6A 01 E8 39 1D 00 00 59 85  
> C0 75 08 6A 1C E8 C3 00 00 00 59 E8 99 35 00 00 85 C0 75 08 6A 10 E8 B2 00 00 00 59 33 F6 89 75  
> FC E8 1B 84 00 00 FF 15 0C 62 57 00 A3 08 D8 5D 00 E8 1B 8C 00 00 A3 7C BE 5D 00 E8 C4 89 00 00  
> E8 06 89 00 00 E8 A7 38 00 00 89 75 D0 8D 45 A4 50 FF 15 18 62 57 00 E8 97 88 00 00 89 45 9C F6  
> 45 D0 01 74 06 0F B7 45 D4 EB 03 6A 0A 58 50 FF 75 9C 56 56 FF 15 68 64 57 00 50 E8 2D C0 EB FF  
> 89 45 A0 50 E8 95 38 00 00 8B 45 EC 8B 08 8B 09 89 4D 98 50 51 E8 68 55 00 00 59 59 C3  
>   
> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
> Microsoft Visual C++ 6.0  
>   
> .text  
> .rdata  
> .data  
> .tls  
> .rsrc  
>   
> 004AD06E ACDS>/$  55                  push ebp  
> 004AD06F      |.  8BEC                mov ebp,esp  
> 004AD071      |.  6A FF               push -1  
> 004AD073      |.  68 28014E00         push ACDSee.004E0128  
> 004AD078      |.  68 9C0C4B00         push ACDSee.004B0C9C                                     ;  SE handler installation  
> 004AD07D      |.  64:A1 00000000      mov eax,dword ptr fs:\[0\]  
> 004AD083      |.  50                  push eax  
> 004AD084      |.  64:8925 00000000    mov dword ptr fs:\[0\],esp  
> 004AD08B      |.  83EC 58             sub esp,58  
> 004AD08E      |.  53                  push ebx  
> 004AD08F      |.  56                  push esi  
> 004AD090      |.  57                  push edi                                                 ;  ntdll.7C930738  
> 004AD091      |.  8965 E8             mov dword ptr ss:\[ebp-18\],esp  
> 004AD094      |.  FF15 A8744D00       call near dword ptr ds:\[<&KERNEL32.GetVersion>\]          ;  kernel32.GetVersion  
> 004AD09A      |.  33D2                xor edx,edx                                              ;  ntdll.KiFastSystemCallRet  
> 004AD09C      |.  8AD4                mov dl,ah  
> 004AD09E      |.  8915 F4584F00       mov dword ptr ds:\[4F58F4\],edx                            ;  ntdll.KiFastSystemCallRet  
> 004AD0A4      |.  8BC8                mov ecx,eax  
> 004AD0A6      |.  81E1 FF000000       and ecx,0FF  
> 004AD0AC      |.  890D F0584F00       mov dword ptr ds:\[4F58F0\],ecx  
> 004AD0B2      |.  C1E1 08             shl ecx,8  
> 004AD0B5      |.  03CA                add ecx,edx                                              ;  ntdll.KiFastSystemCallRet  
> 004AD0B7      |.  890D EC584F00       mov dword ptr ds:\[4F58EC\],ecx  
> 004AD0BD      |.  C1E8 10             shr eax,10  
> 004AD0C0      |.  A3 E8584F00         mov dword ptr ds:\[4F58E8\],eax  
> 004AD0C5      |.  6A 01               push 1  
> 004AD0C7      |.  E8 C12D0000         call ACDSee.004AFE8D  
> 004AD0CC      |.  59                  pop ecx                                                  ;  kernel32.7C816FD7  
> 004AD0CD      |.  85C0                test eax,eax  
> 004AD0CF      |.  75 08               jnz short ACDSee.004AD0D9  
> 004AD0D1      |.  6A 1C               push 1C  
> 004AD0D3      |.  E8 C3000000         call ACDSee.004AD19B  
> 004AD0D8      |.  59                  pop ecx                                                  ;  kernel32.7C816FD7  
> 004AD0D9      |>  E8 4E230000         call ACDSee.004AF42C  
> 004AD0DE      |.  85C0                test eax,eax  
> 004AD0E0      |.  75 08               jnz short ACDSee.004AD0EA  
> 004AD0E2      |.  6A 10               push 10  
> 004AD0E4      |.  E8 B2000000         call ACDSee.004AD19B  
> 004AD0E9      |.  59                  pop ecx                                                  ;  kernel32.7C816FD7  
> 004AD0EA      |>  33F6                xor esi,esi  
> 004AD0EC      |.  8975 FC             mov dword ptr ss:\[ebp-4\],esi  
> 004AD0EF      |.  E8 42510000         call ACDSee.004B2236  
> 004AD0F4      |.  FF15 7C724D00       call near dword ptr ds:\[<&KERNEL32.GetCommandLineA>\]     ; \[GetCommandLineA  
> 004AD0FA      |.  A3 58704F00         mov dword ptr ds:\[4F7058\],eax  
> 004AD0FF      |.  E8 00500000         call ACDSee.004B2104  
> 004AD104      |.  A3 D8584F00         mov dword ptr ds:\[4F58D8\],eax  
> 004AD109      |.  E8 A94D0000         call ACDSee.004B1EB7  
> 004AD10E      |.  E8 EB4C0000         call ACDSee.004B1DFE  
> 004AD113      |.  E8 EF080000         call ACDSee.004ADA07  
> 004AD118      |.  8975 D0             mov dword ptr ss:\[ebp-30\],esi  
> 004AD11B      |.  8D45 A4             lea eax,dword ptr ss:\[ebp-5C\]  
> 004AD11E      |.  50                  push eax                                                 ; /pStartupinfo = NULL  
> 004AD11F      |.  FF15 74724D00       call near dword ptr ds:\[<&KERNEL32.GetStartupInfoA>\]     ; \\GetStartupInfoA  
> 004AD125      |.  E8 7C4C0000         call ACDSee.004B1DA6  
> 004AD12A      |.  8945 9C             mov dword ptr ss:\[ebp-64\],eax  
> 004AD12D      |.  F645 D0 01          test byte ptr ss:\[ebp-30\],1  
> 004AD131      |.  74 06               je short ACDSee.004AD139  
> 004AD133      |.  0FB745 D4           movzx eax,word ptr ss:\[ebp-2C\]  
> 004AD137      |.  EB 03               jmp short ACDSee.004AD13C  
> 004AD139      |>  6A 0A               push 0A  
> 004AD13B      |.  58                  pop eax                                                  ;  kernel32.7C816FD7  
> 004AD13C      |>  50                  push eax  
> 004AD13D      |.  FF75 9C             push dword ptr ss:\[ebp-64\]  
> 004AD140      |.  56                  push esi  
> 004AD141      |.  56                  push esi                                                 ; /pModule = FFFFFFFF ???  
> 004AD142      |.  FF15 90744D00       call near dword ptr ds:\[<&KERNEL32.GetModuleHandleA>\]    ; \\GetModuleHandleA  
> 004AD148      |.  50                  push eax  
> 004AD149      |.  E8 F9E50000         call ACDSee.004BB747  
> 004AD14E      |.  8945 A0             mov dword ptr ss:\[ebp-60\],eax  
> 004AD151      |.  50                  push eax  
> 004AD152      |.  E8 DD080000         call ACDSee.004ADA34  
> 004AD157      |.  8B45 EC             mov eax,dword ptr ss:\[ebp-14\]  
> 004AD15A      |.  8B08                mov ecx,dword ptr ds:\[eax\]  
> 004AD15C      |.  8B09                mov ecx,dword ptr ds:\[ecx\]                               ;  ntdll.7C92E64E  
> 004AD15E      |.  894D 98             mov dword ptr ss:\[ebp-68\],ecx  
> 004AD161      |.  50                  push eax  
> 004AD162      |.  51                  push ecx  
> 004AD163      |.  E8 B3390000         call ACDSee.004B0B1B  
> 004AD168      |.  59                  pop ecx                                                  ;  kernel32.7C816FD7  
> 004AD169      |.  59                  pop ecx                                                  ;  kernel32.7C816FD7  
> 004AD16A      \\.  C3                  retn  
>   
> 55 8B EC 6A FF 68 28 01 4E 00 68 9C 0C 4B 00 64 A1 00 00 00 00 50 64 89 25 00 00 00 00 83 EC 58  
> 53 56 57 89 65 E8 FF 15 A8 74 4D 00 33 D2 8A D4 89 15 F4 58 4F 00 8B C8 81 E1 FF 00 00 00 89 0D  
> F0 58 4F 00 C1 E1 08 03 CA 89 0D EC 58 4F 00 C1 E8 10 A3 E8 58 4F 00 6A 01 E8 C1 2D 00 00 59 85  
> C0 75 08 6A 1C E8 C3 00 00 00 59 E8 4E 23 00 00 85 C0 75 08 6A 10 E8 B2 00 00 00 59 33 F6 89 75  
> FC E8 42 51 00 00 FF 15 7C 72 4D 00 A3 58 70 4F 00 E8 00 50 00 00 A3 D8 58 4F 00 E8 A9 4D 00 00  
> E8 EB 4C 00 00 E8 EF 08 00 00 89 75 D0 8D 45 A4 50 FF 15 74 72 4D 00 E8 7C 4C 00 00 89 45 9C F6  
> 45 D0 01 74 06 0F B7 45 D4 EB 03 6A 0A 58 50 FF 75 9C 56 56 FF 15 90 74 4D 00 50 E8 F9 E5 00 00  
> 89 45 A0 50 E8 DD 08 00 00 8B 45 EC 8B 08 8B 09 89 4D 98 50 51 E8 B3 39 00 00 59 59 C3  
>   
> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
> Microsoft Visual C++ 7.0 \[Debug\]  
>   
> .text  
> .rdata  
> .data  
> .rsrc  
>   
> 004079A3 eMul> $  6A 60               push 60  
> 004079A5       .  68 B0244200         push eMuleUpd.004224B0  
> 004079AA       .  E8 F5E1FFFF         call eMuleUpd.00405BA4  
> 004079AF       .  BF 94000000         mov edi,94  
> 004079B4       .  8BC7                mov eax,edi                                              ;  ntdll.7C930738  
> 004079B6       .  E8 65FCFFFF         call eMuleUpd.00407620  
> 004079BB       .  8965 E8             mov dword ptr ss:\[ebp-18\],esp  
> 004079BE       .  8BF4                mov esi,esp  
> 004079C0       .  893E                mov dword ptr ds:\[esi\],edi                               ;  ntdll.7C930738  
> 004079C2       .  56                  push esi                                                 ; /pVersionInformation = FFFFFFFF  
> 004079C3       .  FF15 70024200       call near dword ptr ds:\[<&KERNEL32.GetVersionExA>\]       ; \\GetVersionExA  
> 004079C9       .  8B4E 10             mov ecx,dword ptr ds:\[esi+10\]  
> 004079CC       .  890D 28BB4200       mov dword ptr ds:\[42BB28\],ecx  
> 004079D2       .  8B46 04             mov eax,dword ptr ds:\[esi+4\]  
> 004079D5       .  A3 34BB4200         mov dword ptr ds:\[42BB34\],eax  
> 004079DA       .  8B56 08             mov edx,dword ptr ds:\[esi+8\]  
> 004079DD       .  8915 38BB4200       mov dword ptr ds:\[42BB38\],edx                            ;  ntdll.KiFastSystemCallRet  
> 004079E3       .  8B76 0C             mov esi,dword ptr ds:\[esi+C\]  
> 004079E6       .  81E6 FF7F0000       and esi,7FFF  
> 004079EC       .  8935 2CBB4200       mov dword ptr ds:\[42BB2C\],esi  
> 004079F2       .  83F9 02             cmp ecx,2  
> 004079F5       .  74 0C               je short eMuleUpd.00407A03  
> 004079F7       .  81CE 00800000       or esi,8000  
> 004079FD       .  8935 2CBB4200       mov dword ptr ds:\[42BB2C\],esi  
> 00407A03       >  C1E0 08             shl eax,8  
> 00407A06       .  03C2                add eax,edx                                              ;  ntdll.KiFastSystemCallRet  
> 00407A08       .  A3 30BB4200         mov dword ptr ds:\[42BB30\],eax  
> 00407A0D       .  33F6                xor esi,esi  
> 00407A0F       .  56                  push esi                                                 ; /pModule = FFFFFFFF ???  
> 00407A10       .  8B3D 88014200       mov edi,dword ptr ds:\[<&KERNEL32.GetModuleHandleA>\]      ; |kernel32.GetModuleHandleA  
> 00407A16       .  FFD7                call near edi                                            ; \\GetModuleHandleA  
> 00407A18       .  66:8138 4D5A        cmp word ptr ds:\[eax\],5A4D  
> 00407A1D       .  75 1F               jnz short eMuleUpd.00407A3E  
> 00407A1F       .  8B48 3C             mov ecx,dword ptr ds:\[eax+3C\]  
> 00407A22       .  03C8                add ecx,eax  
> 00407A24       .  8139 50450000       cmp dword ptr ds:\[ecx\],4550  
> 00407A2A       .  75 12               jnz short eMuleUpd.00407A3E  
> 00407A2C       .  0FB741 18           movzx eax,word ptr ds:\[ecx+18\]  
> 00407A30       .  3D 0B010000         cmp eax,10B  
> 00407A35       .  74 1F               je short eMuleUpd.00407A56  
> 00407A37       .  3D 0B020000         cmp eax,20B  
> 00407A3C       .  74 05               je short eMuleUpd.00407A43  
> 00407A3E       >  8975 E4             mov dword ptr ss:\[ebp-1C\],esi  
> 00407A41       .  EB 27               jmp short eMuleUpd.00407A6A  
> 00407A43       >  83B9 84000000 0E    cmp dword ptr ds:\[ecx+84\],0E  
> 00407A4A       .^ 76 F2               jbe short eMuleUpd.00407A3E  
> 00407A4C       .  33C0                xor eax,eax  
> 00407A4E       .  39B1 F8000000       cmp dword ptr ds:\[ecx+F8\],esi  
> 00407A54       .  EB 0E               jmp short eMuleUpd.00407A64  
> 00407A56       >  8379 74 0E          cmp dword ptr ds:\[ecx+74\],0E  
> 00407A5A       .^ 76 E2               jbe short eMuleUpd.00407A3E  
> 00407A5C       .  33C0                xor eax,eax  
> 00407A5E       .  39B1 E8000000       cmp dword ptr ds:\[ecx+E8\],esi  
> 00407A64       >  0F95C0              setne al  
> 00407A67       .  8945 E4             mov dword ptr ss:\[ebp-1C\],eax  
> 00407A6A       >  6A 01               push 1  
> 00407A6C       .  E8 A3550000         call eMuleUpd.0040D014  
> 00407A71       .  59                  pop ecx                                                  ;  kernel32.7C816FD7  
> 00407A72       .  85C0                test eax,eax  
> 00407A74       .  75 08               jnz short eMuleUpd.00407A7E  
> 00407A76       .  6A 1C               push 1C  
> 00407A78       .  E8 02FFFFFF         call eMuleUpd.0040797F  
> 00407A7D       .  59                  pop ecx                                                  ;  kernel32.7C816FD7  
> 00407A7E       >  E8 572A0000         call eMuleUpd.0040A4DA  
> 00407A83       .  85C0                test eax,eax  
> 00407A85       .  75 08               jnz short eMuleUpd.00407A8F  
> 00407A87       .  6A 10               push 10  
> 00407A89       .  E8 F1FEFFFF         call eMuleUpd.0040797F  
> 00407A8E       .  59                  pop ecx                                                  ;  kernel32.7C816FD7  
> 00407A8F       >  E8 756A0000         call eMuleUpd.0040E509  
> 00407A94       .  8975 FC             mov dword ptr ss:\[ebp-4\],esi  
> 00407A97       .  E8 6F680000         call eMuleUpd.0040E30B  
> 00407A9C       .  85C0                test eax,eax  
> 00407A9E       .  7D 08               jge short eMuleUpd.00407AA8  
> 00407AA0       .  6A 1B               push 1B  
> 00407AA2       .  E8 B3FEFFFF         call eMuleUpd.0040795A  
> 00407AA7       .  59                  pop ecx                                                  ;  kernel32.7C816FD7  
> 00407AA8       >  FF15 0C024200       call near dword ptr ds:\[<&KERNEL32.GetCommandLineA>\]     ; \[GetCommandLineA  
> 00407AAE       .  A3 10D44200         mov dword ptr ds:\[42D410\],eax  
> 00407AB3       .  E8 31670000         call eMuleUpd.0040E1E9  
> 00407AB8       .  A3 14BB4200         mov dword ptr ds:\[42BB14\],eax  
> 00407ABD       .  E8 85660000         call eMuleUpd.0040E147  
> 00407AC2       .  85C0                test eax,eax  
> 00407AC4       .  7D 08               jge short eMuleUpd.00407ACE  
> 00407AC6       .  6A 08               push 8  
> 00407AC8       .  E8 8DFEFFFF         call eMuleUpd.0040795A  
> 00407ACD       .  59                  pop ecx                                                  ;  kernel32.7C816FD7  
> 00407ACE       >  E8 41640000         call eMuleUpd.0040DF14  
> 00407AD3       .  85C0                test eax,eax  
> 00407AD5       .  7D 08               jge short eMuleUpd.00407ADF  
> 00407AD7       .  6A 09               push 9  
> 00407AD9       .  E8 7CFEFFFF         call eMuleUpd.0040795A  
> 00407ADE       .  59                  pop ecx                                                  ;  kernel32.7C816FD7  
> 00407ADF       >  6A 01               push 1  
> 00407AE1       .  E8 72030000         call eMuleUpd.00407E58  
> 00407AE6       .  59                  pop ecx                                                  ;  kernel32.7C816FD7  
> 00407AE7       .  8945 D8             mov dword ptr ss:\[ebp-28\],eax  
> 00407AEA       .  3BC6                cmp eax,esi  
> 00407AEC       .  74 07               je short eMuleUpd.00407AF5  
> 00407AEE       .  50                  push eax  
> 00407AEF       .  E8 66FEFFFF         call eMuleUpd.0040795A  
> 00407AF4       .  59                  pop ecx                                                  ;  kernel32.7C816FD7  
> 00407AF5       >  8975 BC             mov dword ptr ss:\[ebp-44\],esi  
> 00407AF8       .  8D45 90             lea eax,dword ptr ss:\[ebp-70\]  
> 00407AFB       .  50                  push eax                                                 ; /pStartupinfo = NULL  
> 00407AFC       .  FF15 B4004200       call near dword ptr ds:\[<&KERNEL32.GetStartupInfoA>\]     ; \\GetStartupInfoA  
> 00407B02       .  E8 B0630000         call eMuleUpd.0040DEB7  
> 00407B07       .  8945 E0             mov dword ptr ss:\[ebp-20\],eax  
> 00407B0A       .  F645 BC 01          test byte ptr ss:\[ebp-44\],1  
> 00407B0E       .  74 06               je short eMuleUpd.00407B16  
> 00407B10       .  0FB745 C0           movzx eax,word ptr ss:\[ebp-40\]  
> 00407B14       .  EB 03               jmp short eMuleUpd.00407B19  
> 00407B16       >  6A 0A               push 0A  
> 00407B18       .  58                  pop eax                                                  ;  kernel32.7C816FD7  
> 00407B19       >  50                  push eax  
> 00407B1A       .  FF75 E0             push dword ptr ss:\[ebp-20\]  
> 00407B1D       .  56                  push esi  
> 00407B1E       .  56                  push esi  
> 00407B1F       .  FFD7                call near edi                                            ;  ntdll.7C930738  
> 00407B21       .  50                  push eax  
> 00407B22       .  E8 33B50000         call eMuleUpd.0041305A  
> 00407B27       .  8BF8                mov edi,eax  
> 00407B29       .  897D D4             mov dword ptr ss:\[ebp-2C\],edi                            ;  ntdll.7C930738  
> 00407B2C       .  3975 E4             cmp dword ptr ss:\[ebp-1C\],esi  
> 00407B2F       .  75 06               jnz short eMuleUpd.00407B37  
> 00407B31       .  57                  push edi                                                 ;  ntdll.7C930738  
> 00407B32       .  E8 4E040000         call eMuleUpd.00407F85  
> 00407B37       >  E8 6B040000         call eMuleUpd.00407FA7  
> 00407B3C       .  EB 2B               jmp short eMuleUpd.00407B69  
> 00407B3E       .  8B45 EC             mov eax,dword ptr ss:\[ebp-14\]  
> 00407B41       .  8B08                mov ecx,dword ptr ds:\[eax\]  
> 00407B43       .  8B09                mov ecx,dword ptr ds:\[ecx\]                               ;  ntdll.7C92E64E  
> 00407B45       .  894D DC             mov dword ptr ss:\[ebp-24\],ecx  
> 00407B48       .  50                  push eax  
> 00407B49       .  51                  push ecx  
> 00407B4A       .  E8 04620000         call eMuleUpd.0040DD53  
> 00407B4F       .  59                  pop ecx                                                  ;  kernel32.7C816FD7  
> 00407B50       .  59                  pop ecx                                                  ;  kernel32.7C816FD7  
> 00407B51       .  C3                  retn  
>   
> 6A 60 68 B0 24 42 00 E8 F5 E1 FF FF BF 94 00 00 00 8B C7 E8 65 FC FF FF 89 65 E8 8B F4 89 3E 56  
> FF 15 70 02 42 00 8B 4E 10 89 0D 28 BB 42 00 8B 46 04 A3 34 BB 42 00 8B 56 08 89 15 38 BB 42 00  
> 8B 76 0C 81 E6 FF 7F 00 00 89 35 2C BB 42 00 83 F9 02 74 0C 81 CE 00 80 00 00 89 35 2C BB 42 00  
> C1 E0 08 03 C2 A3 30 BB 42 00 33 F6 56 8B 3D 88 01 42 00 FF D7 66 81 38 4D 5A 75 1F 8B 48 3C 03  
> C8 81 39 50 45 00 00 75 12 0F B7 41 18 3D 0B 01 00 00 74 1F 3D 0B 02 00 00 74 05 89 75 E4 EB 27  
> 83 B9 84 00 00 00 0E 76 F2 33 C0 39 B1 F8 00 00 00 EB 0E 83 79 74 0E 76 E2 33 C0 39 B1 E8 00 00  
> 00 0F 95 C0 89 45 E4 6A 01 E8 A3 55 00 00 59 85 C0 75 08 6A 1C E8 02 FF FF FF 59 E8 57 2A 00 00  
> 85 C0 75 08 6A 10 E8 F1 FE FF FF 59 E8 75 6A 00 00 89 75 FC E8 6F 68 00 00 85 C0 7D 08 6A 1B E8  
> B3 FE FF FF 59 FF 15 0C 02 42 00 A3 10 D4 42 00 E8 31 67 00 00 A3 14 BB 42 00 E8 85 66 00 00 85  
> C0 7D 08 6A 08 E8 8D FE FF FF 59 E8 41 64 00 00 85 C0 7D 08 6A 09 E8 7C FE FF FF 59 6A 01 E8 72  
> 03 00 00 59 89 45 D8 3B C6 74 07 50 E8 66 FE FF FF 59 89 75 BC 8D 45 90 50 FF 15 B4 00 42 00 E8  
> B0 63 00 00 89 45 E0 F6 45 BC 01 74 06 0F B7 45 C0 EB 03 6A 0A 58 50 FF 75 E0 56 56 FF D7 50 E8  
> 33 B5 00 00 8B F8 89 7D D4 39 75 E4 75 06 57 E8 4E 04 00 00 E8 6B 04 00 00 EB 2B 8B 45 EC 8B 08  
> 8B 09 89 4D DC 50 51 E8 04 62 00 00 59 59 C3  
>   
> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
> Microsoft Visual C++ 7.0 Method2 \[Debug\]  
>   
> .text  
> .data  
> .rsrc  
>   
> 0100739D note>  6A 70                 push 70  
> 0100739F        68 98180001           push notepad.01001898  
> 010073A4        E8 BF010000           call notepad.01007568  
> 010073A9        33DB                  xor ebx,ebx  
> 010073AB        53                    push ebx  
> 010073AC        8B3D CC100001         mov edi,dword ptr ds:\[<&KERNEL32.GetModuleHandleA>\]      ; kernel32.GetModuleHandleA  
> 010073B2        FFD7                  call near edi                                            ; ntdll.7C930738  
> 010073B4        66:8138 4D5A          cmp word ptr ds:\[eax\],5A4D  
> 010073B9        75 1F                 jnz short notepad.010073DA  
> 010073BB        8B48 3C               mov ecx,dword ptr ds:\[eax+3C\]  
> 010073BE        03C8                  add ecx,eax  
> 010073C0        8139 50450000         cmp dword ptr ds:\[ecx\],4550  
> 010073C6        75 12                 jnz short notepad.010073DA  
> 010073C8        0FB741 18             movzx eax,word ptr ds:\[ecx+18\]  
> 010073CC        3D 0B010000           cmp eax,10B  
> 010073D1        74 1F                 je short notepad.010073F2  
> 010073D3        3D 0B020000           cmp eax,20B  
> 010073D8        74 05                 je short notepad.010073DF  
> 010073DA        895D E4               mov dword ptr ss:\[ebp-1C\],ebx  
> 010073DD        EB 27                 jmp short notepad.01007406  
> 010073DF        83B9 84000000 0E      cmp dword ptr ds:\[ecx+84\],0E  
> 010073E6      ^ 76 F2                 jbe short notepad.010073DA  
> 010073E8        33C0                  xor eax,eax  
> 010073EA        3999 F8000000         cmp dword ptr ds:\[ecx+F8\],ebx  
> 010073F0        EB 0E                 jmp short notepad.01007400  
> 010073F2        8379 74 0E            cmp dword ptr ds:\[ecx+74\],0E  
> 010073F6      ^ 76 E2                 jbe short notepad.010073DA  
> 010073F8        33C0                  xor eax,eax  
> 010073FA        3999 E8000000         cmp dword ptr ds:\[ecx+E8\],ebx  
> 01007400        0F95C0                setne al  
> 01007403        8945 E4               mov dword ptr ss:\[ebp-1C\],eax  
> 01007406        895D FC               mov dword ptr ss:\[ebp-4\],ebx  
> 01007409        6A 02                 push 2  
> 0100740B        FF15 38130001         call near dword ptr ds:\[<&msvcrt.\_\_set\_app\_type>\]        ; msvcrt.\_\_set\_app\_type  
> 01007411        59                    pop ecx                                                  ; kernel32.7C816FD7  
> 01007412        830D 9CAB0001 FF      or dword ptr ds:\[100AB9C\],FFFFFFFF  
> 01007419        830D A0AB0001 FF      or dword ptr ds:\[100ABA0\],FFFFFFFF  
> 01007420        FF15 34130001         call near dword ptr ds:\[<&msvcrt.\_\_p\_\_fmode>\]            ; msvcrt.\_\_p\_\_fmode  
> 01007426        8B0D B89A0001         mov ecx,dword ptr ds:\[1009AB8\]  
> 0100742C        8908                  mov dword ptr ds:\[eax\],ecx  
> 0100742E        FF15 30130001         call near dword ptr ds:\[<&msvcrt.\_\_p\_\_commode>\]          ; msvcrt.\_\_p\_\_commode  
> 01007434        8B0D B49A0001         mov ecx,dword ptr ds:\[1009AB4\]  
> 0100743A        8908                  mov dword ptr ds:\[eax\],ecx  
> 0100743C        A1 2C130001           mov eax,dword ptr ds:\[<&msvcrt.\_adjust\_fdiv>\]  
> 01007441        8B00                  mov eax,dword ptr ds:\[eax\]  
> 01007443        A3 A4AB0001           mov dword ptr ds:\[100ABA4\],eax  
> 01007448        E8 A7010000           call notepad.010075F4  
> 0100744D        391D 08960001         cmp dword ptr ds:\[1009608\],ebx  
> 01007453        75 0C                 jnz short notepad.01007461  
> 01007455        68 F4750001           push notepad.010075F4                                    ; Entry address  
> 0100745A        FF15 28130001         call near dword ptr ds:\[<&msvcrt.\_\_setusermatherr>\]      ; msvcrt.\_\_setusermatherr  
> 01007460        59                    pop ecx                                                  ; kernel32.7C816FD7  
> 01007461        E8 77010000           call notepad.010075DD  
> 01007466        68 10900001           push notepad.01009010  
> 0100746B        68 0C900001           push notepad.0100900C  
> 01007470        E8 5D010000           call <jmp.&msvcrt.\_initterm>  
> 01007475        A1 B09A0001           mov eax,dword ptr ds:\[1009AB0\]  
> 0100747A        8945 DC               mov dword ptr ss:\[ebp-24\],eax  
> 0100747D        8D45 DC               lea eax,dword ptr ss:\[ebp-24\]  
> 01007480        50                    push eax  
> 01007481        FF35 AC9A0001         push dword ptr ds:\[1009AAC\]  
> 01007487        8D45 D4               lea eax,dword ptr ss:\[ebp-2C\]  
> 0100748A        50                    push eax  
> 0100748B        8D45 D0               lea eax,dword ptr ss:\[ebp-30\]  
> 0100748E        50                    push eax  
> 0100748F        8D45 CC               lea eax,dword ptr ss:\[ebp-34\]  
> 01007492        50                    push eax  
> 01007493        FF15 20130001         call near dword ptr ds:\[<&msvcrt.\_\_getmainargs>\]         ; msvcrt.\_\_getmainargs  
> 01007499        8945 C8               mov dword ptr ss:\[ebp-38\],eax  
> 0100749C        68 08900001           push notepad.01009008  
> 010074A1        68 00900001           push notepad.01009000  
> 010074A6        E8 27010000           call <jmp.&msvcrt.\_initterm>  
> 010074AB        83C4 24               add esp,24  
> 010074AE        A1 1C130001           mov eax,dword ptr ds:\[<&msvcrt.\_acmdln>\]  
> 010074B3        8B30                  mov esi,dword ptr ds:\[eax\]  
> 010074B5        8975 E0               mov dword ptr ss:\[ebp-20\],esi  
> 010074B8        803E 22               cmp byte ptr ds:\[esi\],22  
> 010074BB        75 3A                 jnz short notepad.010074F7  
> 010074BD        46                    inc esi  
> 010074BE        8975 E0               mov dword ptr ss:\[ebp-20\],esi  
> 010074C1        8A06                  mov al,byte ptr ds:\[esi\]  
> 010074C3        3AC3                  cmp al,bl  
> 010074C5        74 04                 je short notepad.010074CB  
> 010074C7        3C 22                 cmp al,22  
> 010074C9      ^ 75 F2                 jnz short notepad.010074BD  
> 010074CB        803E 22               cmp byte ptr ds:\[esi\],22  
> 010074CE        75 04                 jnz short notepad.010074D4  
> 010074D0        46                    inc esi  
> 010074D1        8975 E0               mov dword ptr ss:\[ebp-20\],esi  
> 010074D4        8A06                  mov al,byte ptr ds:\[esi\]  
> 010074D6        3AC3                  cmp al,bl  
> 010074D8        74 04                 je short notepad.010074DE  
> 010074DA        3C 20                 cmp al,20  
> 010074DC      ^ 76 F2                 jbe short notepad.010074D0  
> 010074DE        895D AC               mov dword ptr ss:\[ebp-54\],ebx  
> 010074E1        8D45 80               lea eax,dword ptr ss:\[ebp-80\]  
> 010074E4        50                    push eax  
> 010074E5        FF15 D0100001         call near dword ptr ds:\[<&KERNEL32.GetStartupInfoA>\]     ; kernel32.GetStartupInfoA  
> 010074EB        F645 AC 01            test byte ptr ss:\[ebp-54\],1  
> 010074EF        74 11                 je short notepad.01007502  
> 010074F1        0FB745 B0             movzx eax,word ptr ss:\[ebp-50\]  
> 010074F5        EB 0E                 jmp short notepad.01007505  
> 010074F7        803E 20               cmp byte ptr ds:\[esi\],20  
> 010074FA      ^ 76 D8                 jbe short notepad.010074D4  
> 010074FC        46                    inc esi  
> 010074FD        8975 E0               mov dword ptr ss:\[ebp-20\],esi  
> 01007500      ^ EB F5                 jmp short notepad.010074F7  
> 01007502        6A 0A                 push 0A  
> 01007504        58                    pop eax                                                  ; kernel32.7C816FD7  
> 01007505        50                    push eax  
> 01007506        56                    push esi  
> 01007507        53                    push ebx  
> 01007508        53                    push ebx  
> 01007509        FFD7                  call near edi                                            ; ntdll.7C930738  
> 0100750B        50                    push eax  
> 0100750C        E8 25B4FFFF           call notepad.01002936  
> 01007511        8BF0                  mov esi,eax  
> 01007513        8975 C4               mov dword ptr ss:\[ebp-3C\],esi  
> 01007516        395D E4               cmp dword ptr ss:\[ebp-1C\],ebx  
> 01007519        75 07                 jnz short notepad.01007522  
> 0100751B        56                    push esi  
> 0100751C        FF15 18130001         call near dword ptr ds:\[<&msvcrt.exit>\]                  ; msvcrt.exit  
> 01007522        FF15 00130001         call near dword ptr ds:\[<&msvcrt.\_cexit>\]                ; msvcrt.\_cexit  
> 01007528        EB 2D                 jmp short notepad.01007557  
> 0100752A        8B45 EC               mov eax,dword ptr ss:\[ebp-14\]  
> 0100752D        8B08                  mov ecx,dword ptr ds:\[eax\]  
> 0100752F        8B09                  mov ecx,dword ptr ds:\[ecx\]                               ; ntdll.7C92E64E  
> 01007531        894D D8               mov dword ptr ss:\[ebp-28\],ecx  
> 01007534        50                    push eax  
> 01007535        51                    push ecx  
> 01007536        E8 8B000000           call <jmp.&msvcrt.\_XcptFilter>  
> 0100753B        59                    pop ecx                                                  ; kernel32.7C816FD7  
> 0100753C        59                    pop ecx                                                  ; kernel32.7C816FD7  
> 0100753D        C3                    retn  
>   
> 6A 70 68 98 18 00 01 E8 BF 01 00 00 33 DB 53 8B 3D CC 10 00 01 FF D7 66 81 38 4D 5A 75 1F 8B 48  
> 3C 03 C8 81 39 50 45 00 00 75 12 0F B7 41 18 3D 0B 01 00 00 74 1F 3D 0B 02 00 00 74 05 89 5D E4  
> EB 27 83 B9 84 00 00 00 0E 76 F2 33 C0 39 99 F8 00 00 00 EB 0E 83 79 74 0E 76 E2 33 C0 39 99 E8  
> 00 00 00 0F 95 C0 89 45 E4 89 5D FC 6A 02 FF 15 38 13 00 01 59 83 0D 9C AB 00 01 FF 83 0D A0 AB  
> 00 01 FF FF 15 34 13 00 01 8B 0D B8 9A 00 01 89 08 FF 15 30 13 00 01 8B 0D B4 9A 00 01 89 08 A1  
> 2C 13 00 01 8B 00 A3 A4 AB 00 01 E8 A7 01 00 00 39 1D 08 96 00 01 75 0C 68 F4 75 00 01 FF 15 28  
> 13 00 01 59 E8 77 01 00 00 68 10 90 00 01 68 0C 90 00 01 E8 5D 01 00 00 A1 B0 9A 00 01 89 45 DC  
> 8D 45 DC 50 FF 35 AC 9A 00 01 8D 45 D4 50 8D 45 D0 50 8D 45 CC 50 FF 15 20 13 00 01 89 45 C8 68  
> 08 90 00 01 68 00 90 00 01 E8 27 01 00 00 83 C4 24 A1 1C 13 00 01 8B 30 89 75 E0 80 3E 22 75 3A  
> 46 89 75 E0 8A 06 3A C3 74 04 3C 22 75 F2 80 3E 22 75 04 46 89 75 E0 8A 06 3A C3 74 04 3C 20 76  
> F2 89 5D AC 8D 45 80 50 FF 15 D0 10 00 01 F6 45 AC 01 74 11 0F B7 45 B0 EB 0E 80 3E 20 76 D8 46  
> 89 75 E0 EB F5 6A 0A 58 50 56 53 53 FF D7 50 E8 25 B4 FF FF 8B F0 89 75 C4 39 5D E4 75 07 56 FF  
> 15 18 13 00 01 FF 15 00 13 00 01 EB 2D 8B 45 EC 8B 08 8B 09 89 4D D8 50 51 E8 8B 00 00 00 59 59  
> C3  
>   
> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
> Microsoft Visual C++ 7.0  
>   
> .text  
> .rdata  
> .data  
> .rsrc  
>   
> 004A5D0A BitB> $  6A 60               push 60  
> 004A5D0C       .  68 D07A4C00         push BitBuddy.004C7AD0  
> 004A5D11       .  E8 7A060000         call BitBuddy.004A6390  
> 004A5D16       .  BF 94000000         mov edi,94  
> 004A5D1B       .  8BC7                mov eax,edi                                              ;  ntdll.7C930738  
> 004A5D1D       .  E8 CEE6FFFF         call BitBuddy.004A43F0  
> 004A5D22       .  8965 E8             mov dword ptr ss:\[ebp-18\],esp  
> 004A5D25       .  8BF4                mov esi,esp  
> 004A5D27       .  893E                mov dword ptr ds:\[esi\],edi                               ;  ntdll.7C930738  
> 004A5D29       .  56                  push esi                                                 ; /pVersionInformation = FFFFFFFF  
> 004A5D2A       .  FF15 8CF24B00       call near dword ptr ds:\[<&KERNEL32.GetVersionExA>\]       ; \\GetVersionExA  
> 004A5D30       .  8B4E 10             mov ecx,dword ptr ds:\[esi+10\]  
> 004A5D33       .  890D 78614E00       mov dword ptr ds:\[4E6178\],ecx  
> 004A5D39       .  8B46 04             mov eax,dword ptr ds:\[esi+4\]  
> 004A5D3C       .  A3 84614E00         mov dword ptr ds:\[4E6184\],eax  
> 004A5D41       .  8B56 08             mov edx,dword ptr ds:\[esi+8\]  
> 004A5D44       .  8915 88614E00       mov dword ptr ds:\[4E6188\],edx                            ;  ntdll.KiFastSystemCallRet  
> 004A5D4A       .  8B76 0C             mov esi,dword ptr ds:\[esi+C\]  
> 004A5D4D       .  81E6 FF7F0000       and esi,7FFF  
> 004A5D53       .  8935 7C614E00       mov dword ptr ds:\[4E617C\],esi  
> 004A5D59       .  83F9 02             cmp ecx,2  
> 004A5D5C       .  74 0C               je short BitBuddy.004A5D6A  
> 004A5D5E       .  81CE 00800000       or esi,8000  
> 004A5D64       .  8935 7C614E00       mov dword ptr ds:\[4E617C\],esi  
> 004A5D6A       >  C1E0 08             shl eax,8  
> 004A5D6D       .  03C2                add eax,edx                                              ;  ntdll.KiFastSystemCallRet  
> 004A5D6F       .  A3 80614E00         mov dword ptr ds:\[4E6180\],eax  
> 004A5D74       .  33F6                xor esi,esi  
> 004A5D76       .  56                  push esi                                                 ; /pModule = FFFFFFFF ???  
> 004A5D77       .  8B3D 08F24B00       mov edi,dword ptr ds:\[<&KERNEL32.GetModuleHandleA>\]      ; |kernel32.GetModuleHandleA  
> 004A5D7D       .  FFD7                call near edi                                            ; \\GetModuleHandleA  
> 004A5D7F       .  66:8138 4D5A        cmp word ptr ds:\[eax\],5A4D  
> 004A5D84       .  75 1F               jnz short BitBuddy.004A5DA5  
> 004A5D86       .  8B48 3C             mov ecx,dword ptr ds:\[eax+3C\]  
> 004A5D89       .  03C8                add ecx,eax  
> 004A5D8B       .  8139 50450000       cmp dword ptr ds:\[ecx\],4550  
> 004A5D91       .  75 12               jnz short BitBuddy.004A5DA5  
> 004A5D93       .  0FB741 18           movzx eax,word ptr ds:\[ecx+18\]  
> 004A5D97       .  3D 0B010000         cmp eax,10B  
> 004A5D9C       .  74 1F               je short BitBuddy.004A5DBD  
> 004A5D9E       .  3D 0B020000         cmp eax,20B  
> 004A5DA3       .  74 05               je short BitBuddy.004A5DAA  
> 004A5DA5       >  8975 E4             mov dword ptr ss:\[ebp-1C\],esi  
> 004A5DA8       .  EB 27               jmp short BitBuddy.004A5DD1  
> 004A5DAA       >  83B9 84000000 0E    cmp dword ptr ds:\[ecx+84\],0E  
> 004A5DB1       .^ 76 F2               jbe short BitBuddy.004A5DA5  
> 004A5DB3       .  33C0                xor eax,eax  
> 004A5DB5       .  39B1 F8000000       cmp dword ptr ds:\[ecx+F8\],esi  
> 004A5DBB       .  EB 0E               jmp short BitBuddy.004A5DCB  
> 004A5DBD       >  8379 74 0E          cmp dword ptr ds:\[ecx+74\],0E  
> 004A5DC1       .^ 76 E2               jbe short BitBuddy.004A5DA5  
> 004A5DC3       .  33C0                xor eax,eax  
> 004A5DC5       .  39B1 E8000000       cmp dword ptr ds:\[ecx+E8\],esi  
> 004A5DCB       >  0F95C0              setne al  
> 004A5DCE       .  8945 E4             mov dword ptr ss:\[ebp-1C\],eax  
> 004A5DD1       >  6A 01               push 1  
> 004A5DD3       .  E8 6C340000         call BitBuddy.004A9244  
> 004A5DD8       .  59                  pop ecx                                                  ;  kernel32.7C816FD7  
> 004A5DD9       .  85C0                test eax,eax  
> 004A5DDB       .  75 08               jnz short BitBuddy.004A5DE5  
> 004A5DDD       .  6A 1C               push 1C  
> 004A5DDF       .  E8 02FFFFFF         call BitBuddy.004A5CE6  
> 004A5DE4       .  59                  pop ecx                                                  ;  kernel32.7C816FD7  
> 004A5DE5       >  E8 EE300000         call BitBuddy.004A8ED8  
> 004A5DEA       .  85C0                test eax,eax  
> 004A5DEC       .  75 08               jnz short BitBuddy.004A5DF6  
> 004A5DEE       .  6A 10               push 10  
> 004A5DF0       .  E8 F1FEFFFF         call BitBuddy.004A5CE6  
> 004A5DF5       .  59                  pop ecx                                                  ;  kernel32.7C816FD7  
> 004A5DF6       >  E8 EA6F0000         call BitBuddy.004ACDE5  
> 004A5DFB       .  8975 FC             mov dword ptr ss:\[ebp-4\],esi  
> 004A5DFE       .  E8 BC630000         call BitBuddy.004AC1BF  
> 004A5E03       .  85C0                test eax,eax  
> 004A5E05       .  7D 08               jge short BitBuddy.004A5E0F  
> 004A5E07       .  6A 1B               push 1B  
> 004A5E09       .  E8 B3FEFFFF         call BitBuddy.004A5CC1  
> 004A5E0E       .  59                  pop ecx                                                  ;  kernel32.7C816FD7  
> 004A5E0F       >  FF15 54F14B00       call near dword ptr ds:\[<&KERNEL32.GetCommandLineA>\]     ; \[GetCommandLineA  
> 004A5E15       .  A3 F4784E00         mov dword ptr ds:\[4E78F4\],eax  
> 004A5E1A       .  E8 A46E0000         call BitBuddy.004ACCC3  
> 004A5E1F       .  A3 D85F4E00         mov dword ptr ds:\[4E5FD8\],eax  
> 004A5E24       .  E8 F86D0000         call BitBuddy.004ACC21  
> 004A5E29       .  85C0                test eax,eax  
> 004A5E2B       .  7D 08               jge short BitBuddy.004A5E35  
> 004A5E2D       .  6A 08               push 8  
> 004A5E2F       .  E8 8DFEFFFF         call BitBuddy.004A5CC1  
> 004A5E34       .  59                  pop ecx                                                  ;  kernel32.7C816FD7  
> 004A5E35       >  E8 B46B0000         call BitBuddy.004AC9EE  
> 004A5E3A       .  85C0                test eax,eax  
> 004A5E3C       .  7D 08               jge short BitBuddy.004A5E46  
> 004A5E3E       .  6A 09               push 9  
> 004A5E40       .  E8 7CFEFFFF         call BitBuddy.004A5CC1  
> 004A5E45       .  59                  pop ecx                                                  ;  kernel32.7C816FD7  
> 004A5E46       >  6A 01               push 1  
> 004A5E48       .  E8 42290000         call BitBuddy.004A878F  
> 004A5E4D       .  59                  pop ecx                                                  ;  kernel32.7C816FD7  
> 004A5E4E       .  8945 D8             mov dword ptr ss:\[ebp-28\],eax  
> 004A5E51       .  3BC6                cmp eax,esi  
> 004A5E53       .  74 07               je short BitBuddy.004A5E5C  
> 004A5E55       .  50                  push eax  
> 004A5E56       .  E8 66FEFFFF         call BitBuddy.004A5CC1  
> 004A5E5B       .  59                  pop ecx                                                  ;  kernel32.7C816FD7  
> 004A5E5C       >  8975 BC             mov dword ptr ss:\[ebp-44\],esi  
> 004A5E5F       .  8D45 90             lea eax,dword ptr ss:\[ebp-70\]  
> 004A5E62       .  50                  push eax                                                 ; /pStartupinfo = NULL  
> 004A5E63       .  FF15 58F14B00       call near dword ptr ds:\[<&KERNEL32.GetStartupInfoA>\]     ; \\GetStartupInfoA  
> 004A5E69       .  E8 236B0000         call BitBuddy.004AC991  
> 004A5E6E       .  8945 E0             mov dword ptr ss:\[ebp-20\],eax  
> 004A5E71       .  F645 BC 01          test byte ptr ss:\[ebp-44\],1  
> 004A5E75       .  74 06               je short BitBuddy.004A5E7D  
> 004A5E77       .  0FB745 C0           movzx eax,word ptr ss:\[ebp-40\]  
> 004A5E7B       .  EB 03               jmp short BitBuddy.004A5E80  
> 004A5E7D       >  6A 0A               push 0A  
> 004A5E7F       .  58                  pop eax                                                  ;  kernel32.7C816FD7  
> 004A5E80       >  50                  push eax  
> 004A5E81       .  FF75 E0             push dword ptr ss:\[ebp-20\]  
> 004A5E84       .  56                  push esi  
> 004A5E85       .  56                  push esi  
> 004A5E86       .  FFD7                call near edi                                            ;  ntdll.7C930738  
> 004A5E88       .  50                  push eax                                                 ; |Arg1 = 00000000  
> 004A5E89       .  E8 620BF8FF         call BitBuddy.004269F0                                   ; \\BitBuddy.004269F0  
> 004A5E8E       .  8BF8                mov edi,eax  
> 004A5E90       .  897D D4             mov dword ptr ss:\[ebp-2C\],edi                            ;  ntdll.7C930738  
> 004A5E93       .  3975 E4             cmp dword ptr ss:\[ebp-1C\],esi  
> 004A5E96       .  75 06               jnz short BitBuddy.004A5E9E  
> 004A5E98       .  57                  push edi                                                 ;  ntdll.7C930738  
> 004A5E99       .  E8 1E2A0000         call BitBuddy.004A88BC  
> 004A5E9E       >  E8 3B2A0000         call BitBuddy.004A88DE  
> 004A5EA3       .  EB 2B               jmp short BitBuddy.004A5ED0  
> 004A5EA5       .  8B45 EC             mov eax,dword ptr ss:\[ebp-14\]  
> 004A5EA8       .  8B08                mov ecx,dword ptr ds:\[eax\]  
> 004A5EAA       .  8B09                mov ecx,dword ptr ds:\[ecx\]                               ;  ntdll.7C92E64E  
> 004A5EAC       .  894D DC             mov dword ptr ss:\[ebp-24\],ecx  
> 004A5EAF       .  50                  push eax  
> 004A5EB0       .  51                  push ecx  
> 004A5EB1       .  E8 77690000         call BitBuddy.004AC82D  
> 004A5EB6       .  59                  pop ecx                                                  ;  kernel32.7C816FD7  
> 004A5EB7       .  59                  pop ecx                                                  ;  kernel32.7C816FD7  
> 004A5EB8       .  C3                  retn  
>   
> 6A 60 68 D0 7A 4C 00 E8 7A 06 00 00 BF 94 00 00 00 8B C7 E8 CE E6 FF FF 89 65 E8 8B F4 89 3E 56  
> FF 15 8C F2 4B 00 8B 4E 10 89 0D 78 61 4E 00 8B 46 04 A3 84 61 4E 00 8B 56 08 89 15 88 61 4E 00  
> 8B 76 0C 81 E6 FF 7F 00 00 89 35 7C 61 4E 00 83 F9 02 74 0C 81 CE 00 80 00 00 89 35 7C 61 4E 00  
> C1 E0 08 03 C2 A3 80 61 4E 00 33 F6 56 8B 3D 08 F2 4B 00 FF D7 66 81 38 4D 5A 75 1F 8B 48 3C 03  
> C8 81 39 50 45 00 00 75 12 0F B7 41 18 3D 0B 01 00 00 74 1F 3D 0B 02 00 00 74 05 89 75 E4 EB 27  
> 83 B9 84 00 00 00 0E 76 F2 33 C0 39 B1 F8 00 00 00 EB 0E 83 79 74 0E 76 E2 33 C0 39 B1 E8 00 00  
> 00 0F 95 C0 89 45 E4 6A 01 E8 6C 34 00 00 59 85 C0 75 08 6A 1C E8 02 FF FF FF 59 E8 EE 30 00 00  
> 85 C0 75 08 6A 10 E8 F1 FE FF FF 59 E8 EA 6F 00 00 89 75 FC E8 BC 63 00 00 85 C0 7D 08 6A 1B E8  
> B3 FE FF FF 59 FF 15 54 F1 4B 00 A3 F4 78 4E 00 E8 A4 6E 00 00 A3 D8 5F 4E 00 E8 F8 6D 00 00 85  
> C0 7D 08 6A 08 E8 8D FE FF FF 59 E8 B4 6B 00 00 85 C0 7D 08 6A 09 E8 7C FE FF FF 59 6A 01 E8 42  
> 29 00 00 59 89 45 D8 3B C6 74 07 50 E8 66 FE FF FF 59 89 75 BC 8D 45 90 50 FF 15 58 F1 4B 00 E8  
> 23 6B 00 00 89 45 E0 F6 45 BC 01 74 06 0F B7 45 C0 EB 03 6A 0A 58 50 FF 75 E0 56 56 FF D7 50 E8  
> 62 0B F8 FF 8B F8 89 7D D4 39 75 E4 75 06 57 E8 1E 2A 00 00 E8 3B 2A 00 00 EB 2B 8B 45 EC 8B 08  
> 8B 09 89 4D DC 50 51 E8 77 69 00 00 59 59 C3  
>   
> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
> VC8 -> Microsoft Corporation \*  
>   
> .text  
> .rdata  
> .data  
> .rsrc  
>   
> 00495FCE       > /6A 60               push 60  
> 00495FD0       . |68 387B4C00         push foobar20.004C7B38  
> 00495FD5       . |E8 A60C0000         call foobar20.00496C80  
> 00495FDA       . |8365 FC 00          and dword ptr ss:\[ebp-4\],0  
> 00495FDE       . |8D45 90             lea eax,dword ptr ss:\[ebp-70\]  
> 00495FE1       . |50                  push eax                                                 ; /pStartupinfo = 535C08DE  
> 00495FE2       . |FF15 14414B00       call near dword ptr ds:\[<&KERNEL32.GetStartupInfoW>\]     ; \\GetStartupInfoW  
> 00495FE8       . |C745 FC FEFFFFFF    mov dword ptr ss:\[ebp-4\],-2  
> 00495FEF       . |BF 94000000         mov edi,94  
> 00495FF4       . |57                  push edi                                                 ; /HeapSize = 7C930738 (2090010424.)  
> 00495FF5       . |6A 00               push 0                                                   ; |Flags = 0  
> 00495FF7       . |8B1D 10414B00       mov ebx,dword ptr ds:\[<&KERNEL32.GetProcessHeap>\]        ; |kernel32.GetProcessHeap  
> 00495FFD       . |FFD3                call near ebx                                            ; |\[GetProcessHeap  
> 00495FFF       . |50                  push eax                                                 ; |hHeap = 535C08DE  
> 00496000       . |FF15 08414B00       call near dword ptr ds:\[<&KERNEL32.HeapAlloc>\]           ; \\HeapAlloc  
> 00496006       . |8BF0                mov esi,eax  
> 00496008       . |85F6                test esi,esi  
> 0049600A       . |75 0D               jnz short foobar20.00496019  
> 0049600C       . |6A 12               push 12  
> 0049600E       . |E8 56FFFFFF         call foobar20.00495F69  
> 00496013       . |59                  pop ecx                                                  ;  kernel32.7C816FD7  
> 00496014       . |E9 89010000         jmp foobar20.004961A2  
> 00496019       > |893E                mov dword ptr ds:\[esi\],edi                               ;  ntdll.7C930738  
> 0049601B       . |56                  push esi                                                 ; /pVersionInformation = FFFFFFFF  
> 0049601C       . |FF15 0C414B00       call near dword ptr ds:\[<&KERNEL32.GetVersionExA>\]       ; \\GetVersionExA  
> 00496022       . |56                  push esi  
> 00496023       . |6A 00               push 0  
> 00496025       . |85C0                test eax,eax  
> 00496027       . |75 0E               jnz short foobar20.00496037  
> 00496029       . |FFD3                call near ebx  
> 0049602B       . |50                  push eax                                                 ; |hHeap = 535C08DE  
> 0049602C       . |FF15 C8414B00       call near dword ptr ds:\[<&KERNEL32.HeapFree>\]            ; \\HeapFree  
> 00496032       . |E9 6B010000         jmp foobar20.004961A2  
> 00496037       > |8B46 10             mov eax,dword ptr ds:\[esi+10\]  
> 0049603A       . |8945 E0             mov dword ptr ss:\[ebp-20\],eax  
> 0049603D       . |8B46 04             mov eax,dword ptr ds:\[esi+4\]  
> 00496040       . |8945 DC             mov dword ptr ss:\[ebp-24\],eax  
> 00496043       . |8B46 08             mov eax,dword ptr ds:\[esi+8\]  
> 00496046       . |8945 D8             mov dword ptr ss:\[ebp-28\],eax  
> 00496049       . |8B7E 0C             mov edi,dword ptr ds:\[esi+C\]  
> 0049604C       . |81E7 FF7F0000       and edi,7FFF  
> 00496052       . |FFD3                call near ebx  
> 00496054       . |50                  push eax                                                 ; |hHeap = 535C08DE  
> 00496055       . |FF15 C8414B00       call near dword ptr ds:\[<&KERNEL32.HeapFree>\]            ; \\HeapFree  
> 0049605B       . |8B75 E0             mov esi,dword ptr ss:\[ebp-20\]  
> 0049605E       . |83FE 02             cmp esi,2  
> 00496061       . |74 06               je short foobar20.00496069  
> 00496063       . |81CF 00800000       or edi,8000  
> 00496069       > |8B4D DC             mov ecx,dword ptr ss:\[ebp-24\]  
> 0049606C       . |8BC1                mov eax,ecx  
> 0049606E       . |C1E0 08             shl eax,8  
> 00496071       . |8B55 D8             mov edx,dword ptr ss:\[ebp-28\]                            ;  ntdll.7C930738  
> 00496074       . |03C2                add eax,edx                                              ;  ntdll.KiFastSystemCallRet  
> 00496076       . |8935 CC084E00       mov dword ptr ds:\[4E08CC\],esi  
> 0049607C       . |A3 D4084E00         mov dword ptr ds:\[4E08D4\],eax  
> 00496081       . |890D D8084E00       mov dword ptr ds:\[4E08D8\],ecx  
> 00496087       . |8915 DC084E00       mov dword ptr ds:\[4E08DC\],edx                            ;  ntdll.KiFastSystemCallRet  
> 0049608D       . |893D D0084E00       mov dword ptr ds:\[4E08D0\],edi                            ;  ntdll.7C930738  
> 00496093       . |E8 F5FEFFFF         call foobar20.00495F8D  
> 00496098       . |8945 E0             mov dword ptr ss:\[ebp-20\],eax  
> 0049609B       . |33DB                xor ebx,ebx  
> 0049609D       . |43                  inc ebx  
> 0049609E       . |53                  push ebx  
> 0049609F       . |E8 8C100000         call foobar20.00497130  
> 004960A4       . |59                  pop ecx                                                  ;  kernel32.7C816FD7  
> 004960A5       . |85C0                test eax,eax  
> 004960A7       . |75 08               jnz short foobar20.004960B1  
> 004960A9       . |6A 1C               push 1C  
> 004960AB       . |E8 B9FEFFFF         call foobar20.00495F69  
> 004960B0       . |59                  pop ecx                                                  ;  kernel32.7C816FD7  
> 004960B1       > |E8 C5050000         call foobar20.0049667B  
> 004960B6       . |85C0                test eax,eax  
> 004960B8       . |75 08               jnz short foobar20.004960C2  
> 004960BA       . |6A 10               push 10  
> 004960BC       . |E8 A8FEFFFF         call foobar20.00495F69  
> 004960C1       . |59                  pop ecx                                                  ;  kernel32.7C816FD7  
> 004960C2       > |E8 80790000         call foobar20.0049DA47  
> 004960C7       . |895D FC             mov dword ptr ss:\[ebp-4\],ebx  
> 004960CA       . |E8 38770000         call foobar20.0049D807  
> 004960CF       . |85C0                test eax,eax  
> 004960D1       . |7D 08               jge short foobar20.004960DB  
> 004960D3       . |6A 1B               push 1B  
> 004960D5       . |E8 9B080000         call foobar20.00496975  
> 004960DA       . |59                  pop ecx                                                  ;  kernel32.7C816FD7  
> 004960DB       > |E8 88760000         call foobar20.0049D768  
> 004960E0       . |A3 9C544E00         mov dword ptr ds:\[4E549C\],eax  
> 004960E5       . |E8 1D750000         call foobar20.0049D607  
> 004960EA       . |A3 AC084E00         mov dword ptr ds:\[4E08AC\],eax  
> 004960EF       . |E8 68740000         call foobar20.0049D55C  
> 004960F4       . |85C0                test eax,eax  
> 004960F6       . |7D 08               jge short foobar20.00496100  
> 004960F8       . |6A 08               push 8  
> 004960FA       . |E8 76080000         call foobar20.00496975  
> 004960FF       . |59                  pop ecx                                                  ;  kernel32.7C816FD7  
> 00496100       > |E8 31720000         call foobar20.0049D336  
> 00496105       . |85C0                test eax,eax  
> 00496107       . |7D 08               jge short foobar20.00496111  
> 00496109       . |6A 09               push 9  
> 0049610B       . |E8 65080000         call foobar20.00496975  
> 00496110       . |59                  pop ecx                                                  ;  kernel32.7C816FD7  
> 00496111       > |53                  push ebx  
> 00496112       . |E8 7A090000         call foobar20.00496A91  
> 00496117       . |59                  pop ecx                                                  ;  kernel32.7C816FD7  
> 00496118       . |85C0                test eax,eax  
> 0049611A       . |74 07               je short foobar20.00496123  
> 0049611C       . |50                  push eax  
> 0049611D       . |E8 53080000         call foobar20.00496975  
> 00496122       . |59                  pop ecx                                                  ;  kernel32.7C816FD7  
> 00496123       > |E8 C8710000         call foobar20.0049D2F0  
> 00496128       . |845D BC             test byte ptr ss:\[ebp-44\],bl  
> 0049612B       . |74 06               je short foobar20.00496133  
> 0049612D       . |0FB74D C0           movzx ecx,word ptr ss:\[ebp-40\]  
> 00496131       . |EB 03               jmp short foobar20.00496136  
> 00496133       > |6A 0A               push 0A  
> 00496135       . |59                  pop ecx                                                  ;  kernel32.7C816FD7  
> 00496136       > |51                  push ecx  
> 00496137       . |50                  push eax  
> 00496138       . |6A 00               push 0  
> 0049613A       . |68 00004000         push foobar20.00400000  
> 0049613F       . |E8 8EAAF9FF         call foobar20.00430BD2  
> 00496144       . |8945 E4             mov dword ptr ss:\[ebp-1C\],eax  
> 00496147       . |837D E0 00          cmp dword ptr ss:\[ebp-20\],0  
> 0049614B       . |75 06               jnz short foobar20.00496153  
> 0049614D       . |50                  push eax  
> 0049614E       . |E8 9E0A0000         call foobar20.00496BF1  
> 00496153       > |E8 BB0A0000         call foobar20.00496C13  
> 00496158       . |EB 2E               jmp short foobar20.00496188  
> 0049615A       . |8B45 EC             mov eax,dword ptr ss:\[ebp-14\]  
> 0049615D       . |8B08                mov ecx,dword ptr ds:\[eax\]  
> 0049615F       . |8B09                mov ecx,dword ptr ds:\[ecx\]                               ;  ntdll.7C92E10E  
> 00496161       . |894D D4             mov dword ptr ss:\[ebp-2C\],ecx  
> 00496164       . |50                  push eax  
> 00496165       . |51                  push ecx  
> 00496166       . |E8 16700000         call foobar20.0049D181  
> 0049616B       . |59                  pop ecx                                                  ;  kernel32.7C816FD7  
> 0049616C       . |59                  pop ecx                                                  ;  kernel32.7C816FD7  
> 0049616D       . |C3                  retn  
> 0049616E       . |8B65 E8             mov esp,dword ptr ss:\[ebp-18\]  
> 00496171       . |8B45 D4             mov eax,dword ptr ss:\[ebp-2C\]                            ;  kernel32.7C816FD7  
> 00496174       . |8945 E4             mov dword ptr ss:\[ebp-1C\],eax  
> 00496177       . |837D E0 00          cmp dword ptr ss:\[ebp-20\],0  
> 0049617B       . |75 06               jnz short foobar20.00496183  
> 0049617D       . |50                  push eax  
> 0049617E       . |E8 7F0A0000         call foobar20.00496C02  
> 00496183       > |E8 9A0A0000         call foobar20.00496C22  
> 00496188       > |C745 FC FEFFFFFF    mov dword ptr ss:\[ebp-4\],-2  
> 0049618F       . |8B45 E4             mov eax,dword ptr ss:\[ebp-1C\]  
> 00496192       . |EB 13               jmp short foobar20.004961A7  
> 00496194       . |33C0                xor eax,eax  
> 00496196       . |40                  inc eax  
> 00496197       . |C3                  retn  
> 00496198       . |8B65 E8             mov esp,dword ptr ss:\[ebp-18\]  
> 0049619B       . |C745 FC FEFFFFFF    mov dword ptr ss:\[ebp-4\],-2  
> 004961A2       > |B8 FF000000         mov eax,0FF  
> 004961A7       > |E8 190B0000         call foobar20.00496CC5  
> 004961AC       . |C3                  retn  
> 004961AD foob> $ |E8 DD780000         call foobar20.0049DA8F  
> 004961B2       .^\\E9 17FEFFFF         jmp foobar20.00495FCE  
> 004961B7       $  3B0D B0CC4D00       cmp ecx,dword ptr ds:\[4DCCB0\]  
> 004961BD       .  75 02               jnz short foobar20.004961C1  
> 004961BF       .  F3:                 prefix rep:  
> 004961C0       .  C3                  retn  
>   
> 6A 60 68 38 7B 4C 00 E8 A6 0C 00 00 83 65 FC 00 8D 45 90 50 FF 15 14 41 4B 00 C7 45 FC FE FF FF  
> FF BF 94 00 00 00 57 6A 00 8B 1D 10 41 4B 00 FF D3 50 FF 15 08 41 4B 00 8B F0 85 F6 75 0D 6A 12  
> E8 56 FF FF FF 59 E9 89 01 00 00 89 3E 56 FF 15 0C 41 4B 00 56 6A 00 85 C0 75 0E FF D3 50 FF 15  
> C8 41 4B 00 E9 6B 01 00 00 8B 46 10 89 45 E0 8B 46 04 89 45 DC 8B 46 08 89 45 D8 8B 7E 0C 81 E7  
> FF 7F 00 00 FF D3 50 FF 15 C8 41 4B 00 8B 75 E0 83 FE 02 74 06 81 CF 00 80 00 00 8B 4D DC 8B C1  
> C1 E0 08 8B 55 D8 03 C2 89 35 CC 08 4E 00 A3 D4 08 4E 00 89 0D D8 08 4E 00 89 15 DC 08 4E 00 89  
> 3D D0 08 4E 00 E8 F5 FE FF FF 89 45 E0 33 DB 43 53 E8 8C 10 00 00 59 85 C0 75 08 6A 1C E8 B9 FE  
> FF FF 59 E8 C5 05 00 00 85 C0 75 08 6A 10 E8 A8 FE FF FF 59 E8 80 79 00 00 89 5D FC E8 38 77 00  
> 00 85 C0 7D 08 6A 1B E8 9B 08 00 00 59 E8 88 76 00 00 A3 9C 54 4E 00 E8 1D 75 00 00 A3 AC 08 4E  
> 00 E8 68 74 00 00 85 C0 7D 08 6A 08 E8 76 08 00 00 59 E8 31 72 00 00 85 C0 7D 08 6A 09 E8 65 08  
> 00 00 59 53 E8 7A 09 00 00 59 85 C0 74 07 50 E8 53 08 00 00 59 E8 C8 71 00 00 84 5D BC 74 06 0F  
> B7 4D C0 EB 03 6A 0A 59 51 50 6A 00 68 00 00 40 00 E8 8E AA F9 FF 89 45 E4 83 7D E0 00 75 06 50  
> E8 9E 0A 00 00 E8 BB 0A 00 00 EB 2E 8B 45 EC 8B 08 8B 09 89 4D D4 50 51 E8 16 70 00 00 59 59 C3  
> 8B 65 E8 8B 45 D4 89 45 E4 83 7D E0 00 75 06 50 E8 7F 0A 00 00 E8 9A 0A 00 00 C7 45 FC FE FF FF  
> FF 8B 45 E4 EB 13 33 C0 40 C3 8B 65 E8 C7 45 FC FE FF FF FF B8 FF 00 00 00 E8 19 0B 00 00 C3 E8  
> DD 78 00 00 E9 17 FE FF FF 3B 0D B0 CC 4D 00 75 02 F3 C3  
>   
> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* 

 4dd466d0e1e0cc123afbe504da2462ca