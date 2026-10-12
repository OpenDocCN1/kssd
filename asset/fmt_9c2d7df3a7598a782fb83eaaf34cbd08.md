
```
0040135F   |.  3BF0                cmp esi,eax  

00401361   |.  8945 F4            mov dword ptr ss:[ebp-C],eax  

00401364   |.  885D A4            mov byte ptr ss:[ebp-5C],bl  

00401367   |.  0F83 B4000000      jnb 3_.00401421  

0040136D   |>  8BFE               /mov edi,esi  

0040136F   |.  56                 |push esi  

00401370   |.  897D 08            |mov dword ptr ss:[ebp+8],edi  

00401373   |.  E8 38200000        |call 3_.004033B0  

00401378   |.  C70424 4C714000    |mov dword ptr ss:[esp],3_.0040714C              ;  ASCII "krnln.fnr"  

0040137F   |.  57                 |push edi  

00401380   |.  8D7406 01          |lea esi,dword ptr ds:[esi+eax+1]  

00401384   |.  E8 47480000        |call 3_.00405BD0  

00401389   |.  59                 |pop ecx  

0040138A   |.  85C0              |test eax,eax  

0040138C   |.  59                 |pop ecx  

0040138D   |.  74 11             |je short 3_.004013A0  

0040138F   |.  68 40714000       |push 3_.00407140                                ;  ASCII "krnln.fne"  

00401394   |.  57                 |push edi  

00401395   |.  E8 36480000        |call 3_.00405BD0  

0040139A   |.  59                 |pop ecx  

0040139B   |.  85C0              |test eax,eax  

0040139D   |.  59                 |pop ecx  

0040139E   |.  75 0C             |jnz short 3_.004013AC  

004013A0   |>  8D45 A4            |lea eax,dword ptr ss:[ebp-5C]  

004013A3   |.  57                 |push edi  

004013A4   |.  50                 |push eax  

004013A5   |.  E8 161F0000        |call 3_.004032C0  

004013AA   |.  59                 |pop ecx  

004013AB   |.  59                 |pop ecx  

004013AC   |>  8B3E               |mov edi,dword ptr ds:[esi]  

004013AE   |.  8D85 6CFEFFFF      |lea eax,dword ptr ss:[ebp-194]  

004013B4   |.  50                 |push eax  

004013B5   |.  8D85 68FDFFFF      |lea eax,dword ptr ss:[ebp-298]  

004013BB   |.  50                 |push eax  

004013BC   |.  83C6 04            |add esi,4  

004013BF   |.  E8 FC1E0000        |call 3_.004032C0  

004013C4   |.  FF75 08            |push dword ptr ss:[ebp+8]  

004013C7   |.  8D85 68FDFFFF      |lea eax,dword ptr ss:[ebp-298]  

004013CD   |.  50                 |push eax  

004013CE   |.  E8 FD1E0000        |call 3_.004032D0  

004013D3   |.  83C4 10            |add esp,10  

004013D6   |.  8D85 68FDFFFF      |lea eax,dword ptr ss:[ebp-298]  

004013DC   |.  53                 |push ebx                                    ; /hTemplateFile  

004013DD   |.  68 80000000        |push 80                                     ; |Attributes = NORMAL  

004013E2   |.  6A 02             |push 2                                      ; |Mode = CREATE_ALWAYS  

004013E4   |.  53                 |push ebx                                    ; |pSecurity  

004013E5   |.  53                 |push ebx                                    ; |ShareMode  

004013E6   |.  68 00000040        |push 40000000                               ; |Access = GENERIC_WRITE  

004013EB   |.  50                 |push eax                                    ; |FileName  

004013EC   |.  FF15 20604000      |call dword ptr ds:[<&kernel32.CreateFileA>]    ; \CreateFileA  

004013F2   |.  83F8 FF            |cmp eax,-1  

004013F5   |.  8945 08            |mov dword ptr ss:[ebp+8],eax  

004013F8   |.  74 17             |je short 3_.00401411  

004013FA   |.  8D4D D8            |lea ecx,dword ptr ss:[ebp-28]  

004013FD   |.  53                 |push ebx                                    ; /pOverlapped  

004013FE   |.  51                 |push ecx                                    ; |pBytesWritten  

004013FF   |.  57                 |push edi                                    ; |nBytesToWrite  

00401400   |.  56                 |push esi                                    ; |Buffer  

00401401   |.  50                 |push eax                                    ; |hFile  

00401402   |.  FF15 0C604000      |call dword ptr ds:[<&kernel32.WriteFile>]      ; \WriteFile  

00401408   |.  FF75 08            |push dword ptr ss:[ebp+8]                     ; /hObject  

0040140B   |.  FF15 08604000      |call dword ptr ds:[<&kernel32.CloseHandle>]    ; \CloseHandle  

00401411   |>  03F7               |add esi,edi  

00401413   |.  3B75 F4            |cmp esi,dword ptr ss:[ebp-C]  

00401416   |.^ 0F82 51FFFFFF      \jb 3_.0040136D  

0040141C   |.  385D A4            cmp byte ptr ss:[ebp-5C],bl  

0040141F   |.  75 0C             jnz short 3_.0040142D  

00401421   |>  C745 FC 20714000  mov dword ptr ss:[ebp-4],3_.00407120          ;  ASCII "Not found the kernel library!"  

00401428   |.  E9 92000000        jmp 3_.004014BF  

0040142D   |>  8D85 6CFEFFFF      lea eax,dword ptr ss:[ebp-194]  

00401433   |.  50                 push eax  

00401434   |.  8D85 68FDFFFF      lea eax,dword ptr ss:[ebp-298]  

0040143A   |.  50                 push eax  

0040143B   |.  E8 801E0000        call 3_.004032C0  

00401440   |.  8D45 A4            lea eax,dword ptr ss:[ebp-5C]  

00401443   |.  50                 push eax  

00401444   |.  8D85 68FDFFFF      lea eax,dword ptr ss:[ebp-298]  

0040144A   |.  50                 push eax  

0040144B   |.  E8 801E0000        call 3_.004032D0  

00401450   |.  83C4 10            add esp,10  

00401453   |.  8D85 68FDFFFF      lea eax,dword ptr ss:[ebp-298]  

00401459   |.  50                 push eax                                    ; /FileName  

0040145A   |.  FF15 04604000      call dword ptr ds:[<&kernel32.LoadLibraryA>]  ; \LoadLibraryA  

00401460   |.  3BC3               cmp eax,ebx  

00401462   |.  75 09             jnz short 3_.0040146D  

00401464   |.  C745 FC 00714000  mov dword ptr ss:[ebp-4],3_.00407100          ;  ASCII "Failed to load kernel library!"  

0040146B   |.  EB 52             jmp short 3_.004014BF  

0040146D   |>  68 F4704000       push 3_.004070F4                              ; /ProcNameOrOrdinal = "GetNewSock"  

00401472   |.  50                 push eax                                    ; |hModule  

00401473   |.  FF15 00604000      call dword ptr ds:[<&kernel32.GetProcAddress]]; \GetProcAddress  

00401479   |.  3BC3               cmp eax,ebx  

0040147B   |.  75 09             jnz short 3_.00401486  

0040147D   |.  C745 FC D4704000  mov dword ptr ss:[ebp-4],3_.004070D4          ;  ASCII "The kernel library is invalid!"  

00401484   |.  EB 39             jmp short 3_.004014BF  

00401486   |>  68 E8030000       push 3E8  

0040148B   |.  FFD0              call eax  

0040148D   |.  3BC3               cmp eax,ebx  

0040148F   |.  8945 F0            mov dword ptr ss:[ebp-10],eax  

00401492   |.  75 2B             jnz short 3_.004014BF  

00401494   |.  C745 FC A8704000  mov dword ptr ss:[ebp-4],3_.004070A8          ;  ASCII "The interface of kernel library is invalid!"  

0040149B   |.  EB 22             jmp short 3_.004014BF  

0040149D   |>  C745 FC 8C704000  mov dword ptr ss:[ebp-4],3_.0040708C          ;  ASCII "Invalid data in the file!"  

004014A4   |.  EB 19             jmp short 3_.004014BF  

004014A6   |>  C745 FC 5C704000  mov dword ptr ss:[ebp-4],3_.0040705C          ;  ASCII "Failed to read file or invalid data in file!"  

004014AD   |.  EB 10             jmp short 3_.004014BF  

004014AF   |>  C745 FC 8C704000  mov dword ptr ss:[ebp-4],3_.0040708C          ;  ASCII "Invalid data in the file!"  

004014B6   |.  EB 15             jmp short 3_.004014CD  

004014B8   |>  C745 FC 38704000  mov dword ptr ss:[ebp-4],3_.00407038          ;  ASCII "Failed to read data from the file!"  

004014BF   |>  395D F8            cmp dword ptr ss:[ebp-8],ebx  

004014C2   |.  74 09             je short 3_.004014CD  

004014C4   |.  FF75 F8            push dword ptr ss:[ebp-8]  

004014C7   |.  E8 5F1F0000        call 3_.0040342B  

004014CC   |.  59                 pop ecx  
```
