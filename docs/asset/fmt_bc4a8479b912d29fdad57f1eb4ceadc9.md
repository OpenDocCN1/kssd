
005A5EB4   .  E8 4B01E6FF   CALL UN\_THINK.00406004                 ; F7 跟入看看  

\>>>>>>>>>>>>>>>>>>>>>>>>>>>>  

     00406004  /$  52            PUSH EDX                            ; F7 跟入后,我们来到这里  
     00406005  |.  50            PUSH EAX  
     00406006  |.  8B4424 10     MOV EAX,DWORD PTR SS:\[ESP+10\]     ; 干扰代码  
     0040600A  |.  F72424        MUL DWORD PTR SS:\[ESP\]            ; 干扰……  
     0040600D  |.  89C1          MOV ECX,EAX                         ; 干扰……  
     0040600F  |.  8B4424 04     MOV EAX,DWORD PTR SS:\[ESP+4\]      ; ……  
     00406013  |.  F76424 0C     MUL DWORD PTR SS:\[ESP+C\]          ; ……  
     00406017  |.  01C1          ADD ECX,EAX                         ; ……  
     00406019  |.  8B0424        MOV EAX,DWORD PTR SS:\[ESP\]        ; 将除以 2 后的序列号放到 EAX  
     0040601C  |.  F76424 0C     MUL DWORD PTR SS:\[ESP+C\]          ; EAX\=EAX\*2  
     00406020  |.  01CA          ADD EDX,ECX                         ; ……  
     00406022  |.  59            POP ECX                              ; 设 EAX 为 temp1  
     00406023  |.  59            POP ECX  
     00406024  .  C2 0800        RETN 8  
                                                                     
     <<<<<<<<<<<<<<<<<<<<<<<<  

005A5EB9   .  8945 F8       MOV DWORD PTR SS:\[EBP\-8\],EAX  

005A5EBC   .  8955 FC       MOV DWORD PTR SS:\[EBP\-4\],EDX  

005A5EBF   .  DF6D F8       FILD QWORD PTR SS:\[EBP\-8\]              

005A5EC2   .  DB7D EC       FSTP TBYTE PTR SS:\[EBP\-14\]             ; 将 temp1 转移到 EBP-14  

005A5EC5   .  9B            WAIT  

005A5EC6   .  6A 00         PUSH 0  

005A5EC8   .  6A 00         PUSH 0  

005A5ECA   .  B8 085F5A00   MOV EAX,UN\_THINK.005A5F08                ; ASCII "Softkey"  

005A5ECF   .  E8 14E6FFFF   CALL UN\_THINK.005A44E8                   ; 读取软件序列号到 ST(0)  

005A5ED4   .  DB6D EC       FLD TBYTE PTR SS:\[EBP\-14\]             ; ST(0)=EBP\-14, ST(1)=软件序列号  

005A5ED7   .  DED9          FCOMPP                                   ; 比较!  

005A5ED9   .  DFE0          FSTSW AX      

005A5EDB   .  9E            SAHF  

005A5EDC   .  0F94C3        SETE BL                                  ; 相等 BL 就置 1!  

005A5EDF   .  33C0          XOR EAX,EAX  

005A5EE1   .  5A            POP EDX  

005A5EE2   .  59            POP ECX  

005A5EE3   .  59            POP ECX  

005A5EE4   .  64:8910       MOV DWORD PTR FS:\[EAX\],EDX  

005A5EE7   .  EB 0C         JMP SHORT UN\_THINK.005A5EF5  

005A5EE9   .^ E9 1AE5E5FF    JMP UN\_THINK.00404408  

005A5EEE   .  33DB          XOR EBX,EBX  

005A5EF0   .  E8 3FE9E5FF   CALL UN\_THINK.00404834  

005A5EF5   >  8BC3          MOV EAX,EBX                              ; EAX\=EBX  

005A5EF7   .  5F            POP EDI  

005A5EF8   .  5E            POP ESI  

005A5EF9   .  5B            POP EBX  

005A5EFA   .  8BE5          MOV ESP,EBP  

005A5EFC   .  5D            POP EBP  

005A5EFD   .  C3            RETN  

▲小结:这段代码就是为了检测,(软件序列号÷2)×2 是否等于软件序列号.想想,什么时候才会成立?对,当序列号是偶数的时候!  

☆☆☆☆☆☆☆☆☆Call\_2☆☆☆☆☆☆☆☆☆  

     005A5F14   $  55            PUSH EBP                             ; F7 跟入后我们来到了这里  
     005A5F15   .  8BEC          MOV EBP,ESP  
     005A5F17   .  33C9          XOR ECX,ECX  
     005A5F19   .  51            PUSH ECX  
     ………………  
     005A5F3E   .  8D45 FC       LEA EAX,DWORD PTR SS:\[EBP\-4\]  
     005A5F41   .  50            PUSH EAX  
     005A5F42   .  8D4D F0       LEA ECX,DWORD PTR SS:\[EBP\-10\]  
     005A5F45   .  33D2          XOR EDX,EDX  
     005A5F47   .  B8 1C605A00   MOV EAX,UN\_THINK.005A601C                ; ASCII "Softkey"  
     005A5F4C   .  E8 8FE3FFFF   CALL UN\_THINK.005A42E0  
     005A5F51   .  8B45 F0       MOV EAX,DWORD PTR SS:\[EBP\-10\]  
     005A5F54   .  B9 04000000   MOV ECX,4  
     005A5F59   .  BA 01000000   MOV EDX,1  
     005A5F5E   .  E8 E9F3E5FF   CALL UN\_THINK.0040534C                   ; 取软件序列号前四位  
     005A5F63   .  8D45 F8       LEA EAX,DWORD PTR SS:\[EBP\-8\]  
     005A5F66   .  50            PUSH EAX  
     005A5F67   .  8D4D EC       LEA ECX,DWORD PTR SS:\[EBP\-14\]  
     005A5F6A   .  33D2          XOR EDX,EDX  
     005A5F6C   .  B8 1C605A00   MOV EAX,UN\_THINK.005A601C                ; ASCII "Softkey"  
     005A5F71   .  E8 6AE3FFFF   CALL UN\_THINK.005A42E0  
     005A5F76   .  8B45 EC       MOV EAX,DWORD PTR SS:\[EBP\-14\]  
     005A5F79   .  B9 02000000   MOV ECX,2  
     005A5F7E   .  BA 05000000   MOV EDX,5  
     005A5F83   .  E8 C4F3E5FF   CALL UN\_THINK.0040534C                   ; 取序列号第 5、6 位  
     005A5F88   .  8D45 F4       LEA EAX,DWORD PTR SS:\[EBP\-C\]  
     005A5F8B   .  50            PUSH EAX  
     005A5F8C   .  8D4D E8       LEA ECX,DWORD PTR SS:\[EBP\-18\]  
     005A5F8F   .  33D2          XOR EDX,EDX  
     005A5F91   .  B8 1C605A00   MOV EAX,UN\_THINK.005A601C                ; ASCII "Softkey"  
     005A5F96   .  E8 45E3FFFF   CALL UN\_THINK.005A42E0  
     005A5F9B   .  8B45 E8       MOV EAX,DWORD PTR SS:\[EBP\-18\]  
     005A5F9E   .  B9 02000000   MOV ECX,2  
     005A5FA3   .  BA 07000000   MOV EDX,7  
     005A5FA8   .  E8 9FF3E5FF   CALL UN\_THINK.0040534C                   ; 取序列号第 7、8 位  
     005A5FAD   .  8B45 F4       MOV EAX,DWORD PTR SS:\[EBP\-C\]  
     005A5FB0   .  E8 FB42E6FF   CALL UN\_THINK.0040A2B0                   ; 序列号第 7、8 位转换为数值型放入 EAX  
     005A5FB5   .  50            PUSH EAX  
     005A5FB6   .  8B45 F8       MOV EAX,DWORD PTR SS:\[EBP\-8\]  
     005A5FB9   .  E8 F242E6FF   CALL UN\_THINK.0040A2B0                   ; 序列号第 5、6 位转换为数值型放入 EAX  
     005A5FBE   .  50            PUSH EAX  
     005A5FBF   .  8B45 FC       MOV EAX,DWORD PTR SS:\[EBP\-4\]  
     005A5FC2   .  E8 E942E6FF   CALL UN\_THINK.0040A2B0                   ; 序列号前四位转换为数值型放入 EAX  
     005A5FC7   .  5A            POP EDX  
     005A5FC8   .  59            POP ECX  
     005A5FC9   .  E8 B663E6FF   CALL UN\_THINK.0040C384                   ; 这里好像用了类似 SEH 的东西?进行检测如何不符合某种条件就出错跑飞掉,要 Shift+F9 才能继续走下去.咱们 F7 跟入看看...  

\>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  

     0040C384  /$  53            PUSH EBX  
     0040C385  |.  56            PUSH ESI  
     0040C386  |.  57            PUSH EDI  
     0040C387  |.  83C4 F8       ADD ESP,-8  
     0040C38A  |.  8BF9          MOV EDI,ECX  
     0040C38C  |.  8BF2          MOV ESI,EDX  
     0040C38E  |.  8BD8          MOV EBX,EAX  
     0040C390  |.  54            PUSH ESP                         ; /Arg1 = 009FFB78  
     0040C391  |.  8BCF          MOV ECX,EDI                      ; |  
     0040C393  |.  8BD6          MOV EDX,ESI                      ; |  
     0040C395  |.  8BC3          MOV EAX,EBX                      ; |  
     0040C397  |.  E8 20FFFFFF   CALL UN\_THINK.0040C2BC           ; 关键,F7 跟入!  
     0040C39C  |.  84C0          TEST AL,AL  
     0040C39E  |.  75 0A         JNZ SHORT UN\_THINK.0040C3AA      ; 如果这里跳走就没事了  
     0040C3A0  |.  A1 D47A6100   MOV EAX,DWORD PTR DS:\[617AD4\]  
     0040C3A5  |.  E8 1ED5FFFF   CALL UN\_THINK.004098C8           ; 就是这个 Call 引起了错误  
     0040C3AA  |>  DD0424        FLD QWORD PTR SS:\[ESP\]  
     0040C3AD  |.  59            POP ECX  
     0040C3AE  |.  5A            POP EDX  
     0040C3AF  |.  5F            POP EDI  
     0040C3B0  |.  5E            POP ESI  
     0040C3B1  |.  5B            POP EBX  
     0040C3B2  .  C3            RETN  
     0040C2BC  /$  55            PUSH EBP                           ;跟入 40C397    CALL UN\_THINK.0040C2BC   我们来到这里  
     0040C2BD  |.  8BEC          MOV EBP,ESP  
     0040C2BF  |.  83C4 F8       ADD ESP,-8  
     0040C2C2  |.  53            PUSH EBX  
     0040C2C3  |.  56            PUSH ESI  
     0040C2C4  |.  57            PUSH EDI  
     0040C2C5  |.  8BD9          MOV EBX,ECX  
     0040C2C7  |.  8BFA          MOV EDI,EDX
