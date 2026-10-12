
6601B7CC    C9                  LEAVE  
6601B7CD    C2 0800             RETN 8

—————————————

下面是调用 `MSVBVM60.__vbaI4Str` 取 `14229037` 的 16 进制值

66103597    E8 BB1FFEFF         CALL MSVBVM60.__vbaI4Str  
                                   ====>取 `14229037` 的 16 进制值 `00D91E2D`

6610359C    50                  PUSH EAX  
                                   ====>EAX=00D91E2D

6610359D    33C0                XOR EAX,EAX  
6610359F    8A06                MOV AL,BYTE PTR DS:[ESI]  
661035A1    46                  INC ESI  
661035A2    FF2485 58EA0F66     JMP DWORD PTR DS:[EAX*4+660FEA58]

—————————————

下面是对 `00D91E2D` 进行简单的 除 2 运算，取得商：

6610533B    59                  POP ECX  
6610533C    58                  POP EAX  
6610533D    99                  CDQ  
6610533E    F7F9                IDIV ECX  
                                   ====>EAX=00D91E2D / 2=006C8F16（H）=7114518（D）

66105340    50                  PUSH EAX  
66105341    33C0                XOR EAX,EAX  
66105343    8A06                MOV AL,BYTE PTR DS:[ESI]  
66105345    46                  INC ESI  
66105346    FF2485 58EA0F66     JMP DWORD PTR DS:[EAX*4+660FEA58]

—————————————

下面是取商 `006C8F16` 的 10 进制值：

661034CF    E8 FA15FEFF         CALL MSVBVM60.__vbaStrI4  
661034D4    50                  PUSH EAX  
                                   ====>EAX=7114518

661034D5    33C0                XOR EAX,EAX  
661034D7    8A06                MOV AL,BYTE PTR DS:[ESI]  
661034D9    46                  INC ESI  
661034DA    FF2485 58EA0F66     JMP DWORD PTR DS:[EAX*4+660FEA58]

—————————————

下面是调用 `rtcStrReverse` 命令将 `7114518` 倒序排列：^O^^O^

660F9A00    8BF8                MOV EDI,EAX  
                                   ====>EDI=EAX=7114518

660F9A02    85FF                TEST EDI,EDI  
660F9A04    75 07               JNZ SHORT MSVBVM60.660F9A0D  
660F9A06    6A 0E               PUSH 0E  
660F9A08    E8 85BBF2FF         CALL MSVBVM60.66025592  
660F9A0D    66:832477 00        AND WORD PTR DS:[EDI+ESI*2],0  
660F9A12    8D0477              LEA EAX,DWORD PTR DS:[EDI+ESI*2]  
660F9A15    8BD6                MOV EDX,ESI  
660F9A17    4E                  DEC ESI  
660F9A18    85D2                TEST EDX,EDX  
660F9A1A    8BCB                MOV ECX,EBX  
660F9A1C    76 10               JBE SHORT MSVBVM60.660F9A2E  
660F9A1E    8D56 01             LEA EDX,DWORD PTR DS:[ESI+1]  
660F9A21    66:8B31             MOV SI,WORD PTR DS:[ECX]  
660F9A24    48                  DEC EAX  
660F9A25    48                  DEC EAX  
660F9A26    41                  INC ECX  
660F9A27    41                  INC ECX  
660F9A28    4A                  DEC EDX  
660F9A29    66:8930             MOV WORD PTR DS:[EAX],SI  
660F9A2C    ^ 75 F3             JNZ SHORT MSVBVM60.660F9A21  
660F9A2E    8BC7                MOV EAX,EDI  
                                   ====>EAX=8154117       倒序排列的结果！注意！！

660F9A30    5E                  POP ESI  
660F9A31    5F                  POP EDI  
660F9A32    ^ EB B8             JMP SHORT MSVBVM60.660F99EC

—————————————

下面是调用 `MSVBVM60.__vbaLenBstr` 命令取 `8154117` 的长度：

661071D4    E8 F2D6F1FF         CALL MSVBVM60.__vbaLenBstr  
661071D9    50                  PUSH EAX  
                                   ====>EAX=7      以此长度取下面的试炼码进行比较！

661071DA    33C0                XOR EAX,EAX  
661071DC    8A06                MOV AL,BYTE PTR DS:[ESI]  
661071DE    46                  INC ESI

—————————————

下面是调用 `MSVBVM60.rtcLeftVar` 命令取试炼码的前 7 位字符：

660248EE >  55                  PUSH EBP  
660248EF    8BEC                MOV EBP,ESP  
660248F1    83EC 10             SUB ESP,10  
660248F4    8B45 10             MOV EAX,DWORD PTR SS:[EBP+10]  
660248F7    56                  PUSH ESI  
660248F8    03C0                ADD EAX,EAX  
660248FA    57                  PUSH EDI  
660248FB    50                  PUSH EAX  
660248FC    8D45 F0             LEA EAX,DWORD PTR SS:[EBP-10]  
660248FF    FF75 0C             PUSH DWORD PTR SS:[EBP+C]  
66024902    50                  PUSH EAX  
66024903    E8 11000000         CALL MSVBVM60.rtcLeftVar  
66024908    8BF0                MOV ESI,EAX  
6602490A    8B45 08             MOV EAX,DWORD PTR SS:[EBP+8]  
6602490D    8BF8                MOV EDI,EAX  
6602490F    A5                  MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ESI]  
66024910    A5                  MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ESI]  
66024911    A5                  MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ESI]  
66024912    A5                  MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ESI]  
66024913    5F                  POP EDI  
66024914    5E                  POP ESI  
66024915    C9                  LEAVE  
66024916    C2 0C00             RETN 0C

66024919 >  55                  PUSH EBP  
6602491A    8BEC                MOV EBP,ESP  
6602491C    83EC 10             SUB ESP,10  
6602491F    56                  PUSH ESI  
66024920    57                  PUSH EDI  
66024921    FF35 B0FE1066       PUSH DWORD PTR DS:[6610FEB0]  
66024927    FF15 AC100066       CALL DWORD PTR DS:[<&KERNEL32.TlsGetvalue>]  
6602492D    8D70 50             LEA ESI,DWORD PTR DS:[EAX+50]  
66024930    56                  PUSH ESI  
66024931    FF75 0C             PUSH DWORD PTR SS:[EBP+C]  
66024934    E8 37000000         CALL MSVBVM60.66024970  
66024939    83F8 FF             CMP EAX,-1  
6602493C    0F84 52250200       JE MSVBVM60.66046E94  
66024942    FF75 10             PUSH DWORD PTR SS:[EBP+10]  
66024945    50                  PUSH EAX  
                                   ====>EAX=13572468           试炼码

66024946    E8 18FEFFFF         CALL MSVBVM60.rtcLeftBstr  
                                   ====>取试炼码的前 7 位！

6602494B    66:833E 08          CMP WORD PTR DS:[ESI],8  
6602494F    8945 F8             MOV DWORD PTR SS:[EBP-8],EAX  
                                   ====>[EBP-8]=EAX=1357246      试炼码的前 7 位

66024952    66:C745 F0 0800     MOV WORD PTR SS:[EBP-10],8  
66024958    0F84 24250200       JE MSVBVM60.66046E82  
6602495E    8B45 08             MOV EAX,DWORD PTR SS:[EBP+8]  
66024961    8D75 F0             LEA ESI,DWORD PTR SS:[EBP-10]  
66024964    8BF8                MOV EDI,EAX  
66024966    A5                  MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ESI]  
66024967    A5                  MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ESI]  
66024968    A5                  MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ESI]  
66024969    A5                  MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ESI]  
6602496A    5F                  POP EDI  
6602496B    5E                  POP ESI  
6602496C    C9                  LEAVE  
6602496D    C2 0C00             RETN 0C

—————————————

下面是调用 `OLEAUT32.VarCmp` 进行比较了：^O^^O^^O^^O^

66100055    FF15 60FE1066       CALL DWORD PTR DS:[6610FE60] ; OLEAUT32.VarCmp  
6610005B    8945 08             MOV DWORD PTR SS:[EBP+8],EAX  
6610005E    8B75 08             MOV ESI,DWORD PTR SS:[EBP+8]  
66100061    85F6                TEST ESI,ESI  
66100063    0F8C E9780000       JL MSVBVM60.66107952  
                                   ====>不跳则 OVER！     呵呵，可惜无法爆破了。

66100069    8D46 FF             LEA EAX,DWORD PTR DS:[ESI-1]  
6610006C    5F                  POP EDI  
6610006D    5E                  POP ESI  
6610006E    5B                  POP EBX  
6610006F    C9                  LEAVE  
66100070    C2 0C00             RETN 0C

跟进 `OLEAUT32.VarCmp`：

77A329D3  |.  8B75 08           MOV ESI,DWORD PTR SS:[EBP+8]  
                                   ====>ESI=1357246            试炼码的前 7 位！

77A329D6  |.  57                PUSH EDI  
77A329D7  |.  85F6              TEST ESI,ESI  
77A329D9  |.  75 4C             JNZ SHORT OLEAUT32.77A32A27  
77A329DB  |.  2175 FC           AND DWORD PTR SS:[EBP-4],ESI  
77A329DE  |>  8B4D 0C           MOV ECX,DWORD PTR SS:[EBP+C]  
                                   ====>ECX=8154117     呵呵，这就是我的注册码了！     

77A329E1  |.  85C9              TEST ECX,ECX  
77A329E3  |.  75 4A             JNZ SHORT OLEAUT32.77A32A2F  
77A329E5  |.  33DB              XOR EBX,EBX  
77A329E7  |>  395D FC           CMP DWORD PTR SS:[EBP-4],EBX  
