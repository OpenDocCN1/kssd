
```asm
004FC87D     E8 1B78FFFF          call 004F409D                       ; 004F409D  
004FC882     8906                 mov dword ptr ds:[esi],eax  
004FC884     E9 37020000         jmp 004FCAC0                        ; 004FCAC0  
004FC889     68 B3E04000         push 40E0B3  
004FC88E     E8 0A78FFFF         call 004F409D                       ; 004F409D  
004FC893     50                  push eax  
004FC894     57                  push edi  
004FC895     E8 D881FFFF         call 004F4A72                       ; 004F4A72  
004FC89A     85C0                test eax,eax  
004FC89C     75 11               jnz short 004FC8AF                  ; 004FC8AF  
004FC89E     68 56CE4000         push 40CE56  
004FC8A3     E8 F577FFFF         call 004F409D                       ; 004F409D  
004FC8A8     8906                mov dword ptr ds:[esi],eax  
004FC8AA     E9 11020000         jmp 004FCAC0                        ; 004FCAC0  
004FC8AF     68 BFE04000         push 40E0BF  
004FC8B4     E8 E477FFFF         call 004F409D                       ; 004F409D  
004FC8B9     50                  push eax  
004FC8BA     57                  push edi  
004FC8BB     E8 B281FFFF         call 004F4A72                       ; 004F4A72  
004FC8C0     85C0                test eax,eax  
004FC8C2     75 11               jnz short 004FC8D5                  ; 004FC8D5  
004FC8C4     68 CDD24000         push 40D2CD  
004FC8C9     E8 CF77FFFF         call 004F409D                       ; 004F409D  
004FC8CE     8906                mov dword ptr ds:[esi],eax  
004FC8D0     E9 EB010000         jmp 004FCAC0                        ; 004FCAC0  
004FC8D5     68 D0E04000         push 40E0D0                          ; ASCII "'H"  
004FC8DA     E8 BE77FFFF         call 004F409D                       ; 004F409D  
004FC8DF     50                  push eax  
004FC8E0     57                  push edi  
004FC8E1     E8 8C81FFFF         call 004F4A72                       ; 004F4A72  
004FC8E6     85C0                test eax,eax  
004FC8E8     75 11               jnz short 004FC8FB                  ; 004FC8FB  
004FC8EA     68 4BD24000         push 40D24B  
004FC8EF     E8 A977FFFF         call 004F409D                       ; 004F409D  
004FC8F4     8906                mov dword ptr ds:[esi],eax  
004FC8F6     E9 C5010000         jmp 004FCAC0                        ; 004FCAC0  
004FC8FB     68 6AE04000         push 40E06A  
004FC900     E8 9877FFFF         call 004F409D                       ; 004F409D  
004FC905     50                  push eax  
004FC906     57                  push edi  
004FC907     E8 6681FFFF         call 004F4A72                       ; 004F4A72  
004FC90C     85C0                test eax,eax  
004FC90E     75 11               jnz short 004FC921                  ; 004FC921  
004FC910     68 B1CE4000         push 40CEB1  
004FC915     E8 8377FFFF         call 004F409D                       ; 004F409D  
004FC91A     8906                mov dword ptr ds:[esi],eax  
004FC91C     E9 9F010000         jmp 004FCAC0                        ; 004FCAC0  
004FC921     68 10CF4000         push 40CF10  
004FC926     E8 7277FFFF         call 004F409D                       ; 004F409D  
004FC92B     50                  push eax  
004FC92C     57                  push edi  
004FC92D     E8 4081FFFF         call 004F4A72                       ; 004F4A72  
004FC932     85C0                test eax,eax  
004FC934     75 11               jnz short 004FC947                  ; 004FC947  
004FC936     68 27CF4000         push 40CF27  
004FC93B     E8 5D77FFFF         call 004F409D                       ; 004F409D  
004FC940     8906                mov dword ptr ds:[esi],eax  
004FC942     E9 79010000         jmp 004FCAC0                        ; 004FCAC0  
004FC947     68 E3E04000         push 40E0E3  
004FC94C     E8 4C77FFFF         call 004F409D                       ; 004F409D  
004FC951     50                  push eax  
004FC952     57                  push edi  
004FC953     E8 1A81FFFF         call 004F4A72                       ; 004F4A72  
004FC958     85C0                test eax,eax  
004FC95A     75 11               jnz short 004FC96D                  ; 004FC96D  
004FC95C     68 80D04000         push 40D080  
004FC961     E8 3777FFFF         call 004F409D                       ; 004F409D  
004FC966     8906                mov dword ptr ds:[esi],eax  
004FC968     E9 53010000         jmp 004FCAC0                        ; 004FCAC0  
004FC96D     68 AFD04000         push 40D0AF  
004FC972     E8 2677FFFF         call 004F409D                       ; 004F409D  
004FC977     50                  push eax  
004FC978     57                  push edi  
004FC979     E8 F480FFFF         call 004F4A72                       ; 004F4A72  
004FC97E     85C0                test eax,eax  
004FC980     75 11               jnz short 004FC993                  ; 004FC993  
004FC982     68 96CF4000         push 40CF96                          ; ASCII "MPM"  
004FC987     E8 1177FFFF         call 004F409D                       ; 004F409D  
004FC98C     8906                mov dword ptr ds:[esi],eax  
004FC98E     E9 2D010000         jmp 004FCAC0                        ; 004FCAC0  
004FC993     68 F3E04000         push 40E0F3  
004FC998     E8 0077FFFF         call 004F409D                       ; 004F409D  
004FC99D     50                  push eax  
004FC99E     57                  push edi  
004FC99F     E8 CE80FFFF         call 004F4A72                       ; 004F4A72  
004FC9A4     85C0                test eax,eax  
004FC9A6     75 11               jnz short 004FC9B9                  ; 004FC9B9  
004FC9A8     68 E7D14000         push 40D1E7  
004FC9AD     E8 EB76FFFF         call 004F409D                       ; 004F409D  
004FC9B2     8906                mov dword ptr ds:[esi],eax  
004FC9B4     E9 07010000         jmp 004FCAC0                        ; 004FCAC0  
004FC9B9     68 08E14000         push 40E108  
004FC9BE     E8 DA76FFFF         call 004F409D                       ; 004F409D  
004FC9C3     50                  push eax  
004FC9C4     57                  push edi  
004FC9C5     E8 A880FFFF         call 004F4A72                       ; 004F4A72  
004FC9CA     85C0                test eax,eax  
004FC9CC     75 11               jnz short 004FC9DF                  ; 004FC9DF  
004FC9CE     68 A0D04000         push 40D0A0  
004FC9D3     E8 C576FFFF         call 004F409D                       ; 004F409D  
004FC9D8     8906                mov dword ptr ds:[esi],eax  
004FC9DA     E9 E1000000         jmp 004FCAC0                        ; 004FCAC0  
004FC9DF     68 1CE14000         push 40E11C  
004FC9E4     E8 B476FFFF         call 004F409D                       ; 004F409D  
004FC9E9     50                  push eax  
004FC9EA     57                  push edi  
004FC9EB     E8 8280FFFF         call 004F4A72                       ; 004F4A72  
004FC9F0     85C0                test eax,eax  
004FC9F2     75 11               jnz short 004FCA05                  ; 004FCA05  
004FC9F4     68 EFD24000         push 40D2EF  
004FC9F9     E8 9F76FFFF         call 004F409D                       ; 004F409D  
004FC9FE     8906                mov dword ptr ds:[esi],eax  
004FCA00     E9 BB000000         jmp 004FCAC0                        ; 004FCAC0  
004FCA05     68 31E14000         push 40E131  
004FCA0A     E8 8E76FFFF         call 004F409D                       ; 004F409D  
004FCA0F     50                  push eax  
```
