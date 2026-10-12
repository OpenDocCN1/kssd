
```
    005904C3    90                NOP  

    005904C4    90                NOP  

    005904C5    90                NOP  

    005904C6 >   8B95 46F84000    MOV EDX,DWORD PTR SS:\[EBP+40F846\]        ; 载入失败的路  

    005904CC    0195 351B4000    ADD DWORD PTR SS:\[EBP+401B35\],EDX  

    005904D2    0195 391B4000    ADD DWORD PTR SS:\[EBP+401B39\],EDX  

    005904D8    6A 00            PUSH 0                          ; /Style  

    005904DA    FFB5 351B4000    PUSH DWORD PTR SS:\[EBP+401B35\]            ; |Caption  

    005904E0    FFB5 391B4000    PUSH DWORD PTR SS:\[EBP+401B39\]            ; |TEXT  

    005904E6    6A 00            PUSH 0                          ; |hOwner = NULL  

    005904E8    FF95 2C854100    CALL DWORD PTR SS:\[EBP+41852C\]            ; \\MessageBoxA  

    005904EE    6A 00            PUSH 0                          ; /ExitCode==0  

    005904F0    FF95 28854100    CALL DWORD PTR SS:\[EBP+418528\]            ; \\ExitProcess  

    005904F6 >   60                PUSHAD                        ; 载入 DLL 后壳把 DLL 名给清除掉  

    005904F7    2BC0             SUB EAX,EAX  

    005904F9 >   8803             MOV BYTE PTR DS:\[EBX\],AL             ; loop clea dllname  

    005904FB    43                INC EBX  

    005904FC    3803             CMP BYTE PTR DS:\[EBX\],AL  

    005904FE   ^ 75 F9            JNZ SHORT <loop clea dllname>  

    00590500    61                POPAD  

    00590501    8985 3EF84000    MOV DWORD PTR SS:\[EBP+40F83E\],EAX        ; 保存 hModule  

    00590507    C785 42F84000 0>MOV DWORD PTR SS:\[EBP+40F842\],0  

    00590511    8B95 46F84000    MOV EDX,DWORD PTR SS:\[EBP+40F846\]  

    00590517    8B06             MOV EAX,DWORD PTR DS:\[ESI\]  

    00590519    0BC0             OR EAX,EAX  

    0059051B    75 07            JNZ SHORT 00590524  

    0059051D    90                NOP  

    0059051E    90                NOP  

    0059051F    90                NOP  

    00590520    90                NOP  

    00590521    8B46 10          MOV EAX,DWORD PTR DS:\[ESI+10\]  

    00590524    03C2             ADD EAX,EDX  

    00590526    0385 42F84000    ADD EAX,DWORD PTR SS:\[EBP+40F842\]  

    0059052C    8B18             MOV EBX,DWORD PTR DS:\[EAX\]  

    0059052E    8B7E 10          MOV EDI,DWORD PTR DS:\[ESI+10\]  

    00590531    03FA             ADD EDI,EDX  

    00590533    03BD 42F84000    ADD EDI,DWORD PTR SS:\[EBP+40F842\]  

    00590539    85DB             TEST EBX,EBX                      ; 判断当前 DLL 的 API 有没有处理完  

    0059053B    0F84 62010000    JE <Disposal Next DLL>  

    00590541    F7C3 00000080    TEST EBX,80000000                     ; 判断为 API 名字还是为序号  

    00590547    75 1D            JNZ SHORT 00590566  

    00590549    90                NOP  

    0059054A    90                NOP  

    0059054B    90                NOP  

    0059054C    90                NOP  

    0059054D    03DA             ADD EBX,EDX                      ; 如果是字符串，还得先还原出正确的 API 名字  

    0059054F    83C3 02          ADD EBX,2  

    00590552    56                PUSH ESI  

    00590553    57                PUSH EDI  

    00590554    50                PUSH EAX  

    00590555    8BF3             MOV ESI,EBX  

    00590557    8BFB             MOV EDI,EBX  

    00590559 >   AC                LODS BYTE PTR DS:\[ESI\]              ; Restor\_API\_NAME  

    0059055A    C0C0 03          ROL AL,3  

    0059055D    AA                STOS BYTE PTR ES:\[EDI\]  

    0059055E    803F 00          CMP BYTE PTR DS:\[EDI\],0  

    00590561   ^ 75 F6            JNZ SHORT <Restor\_API\_NAME>  

    00590563    58                POP EAX  

    00590564    5F                POP EDI  

    00590565    5E                POP ESI  

    00590566    3B9D 46F84000    CMP EBX,DWORD PTR SS:\[EBP+40F846\]  

    0059056C    7C 11            JL SHORT 0059057F                     ; 如果小于 IMAGEBASE 说明是序号  

    0059056E    90                NOP  

    0059056F    90                NOP  

    00590570    90                NOP  

    00590571    90                NOP  

    00590572    83BD 1A204000 0>CMP DWORD PTR SS:\[EBP+40201A\],0  

    00590579    75 0A            JNZ SHORT 00590585  

    0059057B    90                NOP  

    0059057C    90                NOP  

    0059057D    90                NOP  

    0059057E    90                NOP  

    0059057F    81E3 FFFFFF0F    AND EBX,0FFFFFFF  

    00590585    53                PUSH EBX                          ; /API Name  

    00590586    FFB5 3EF84000    PUSH DWORD PTR SS:\[EBP+40F83E\]            ; |hModule  

    0059058C    FF95 1C854100    CALL DWORD PTR SS:\[EBP+41851C\]            ; \\GetProcAddress  

    00590592    3B9D 46F84000    CMP EBX,DWORD PTR SS:\[EBP+40F846\]  

    00590598    7C 0F            JL SHORT 005905A9                     ; 如果是序号则跳  

    0059059A    90                NOP  

    0059059B    90                NOP  

    0059059C    90                NOP  

    0059059D    90                NOP  

    0059059E    60                PUSHAD                        ; 如果是字符串名则清除 API 名  

    0059059F    2BC0             SUB EAX,EAX  

    005905A1 >   8803             MOV BYTE PTR DS:\[EBX\],AL  

    005905A3    43                INC EBX  

    005905A4    3803             CMP BYTE PTR DS:\[EBX\],AL  

    005905A6   ^ 75 F9            JNZ SHORT <Clea API NAME>  

    005905A8    61                POPAD  

    005905A9    0BC0             OR EAX,EAX                      ; 如果获取 API 地址失败则 OVER  

    005905AB   ^ 0F84 15FFFFFF    JE <Fail Message>  

    005905B1    3B85 2C854100    CMP EAX,DWORD PTR SS:\[EBP+41852C\]        ; 判断是否为特殊函数 MessageBoxA  

    005905B7    74 20            JE SHORT 005905D9  

    005905B9    90                NOP  

    005905BA    90                NOP  

    005905BB    90                NOP  

    005905BC    90                NOP  

    005905BD    3B85 C4FD4000    CMP EAX,DWORD PTR SS:\[EBP+40FDC4\]        ; 判断是否为特殊函数 RegisterHotKey  

    005905C3    74 09            JE SHORT 005905CE  

    005905C5    90                NOP  

    005905C6    90                NOP  

    005905C7    90                NOP  

    005905C8    90                NOP  

    005905C9    EB 14            JMP SHORT 005905DF  

    005905CB    90                NOP  

    005905CC    90                NOP  

    005905CD    90                NOP  

    005905CE    8D85 31FE4000    LEA EAX,DWORD PTR SS:\[EBP+40FE31\]  

    005905D4    EB 09            JMP SHORT 005905DF  

    005905D6    90                NOP  

    005905D7    90                NOP  

    005905D8    90                NOP  

    005905D9    8D85 4BFE4000    LEA EAX,DWORD PTR SS:\[EBP+40FE4B\]  

    005905DF    56                PUSH ESI  

    005905E0    FFB5 3EF84000    PUSH DWORD PTR SS:\[EBP+40F83E\]  

    005905E6    5E                POP ESI  

    005905E7    39B5 12204000    CMP DWORD PTR SS:\[EBP+402012\],ESI        ; 判断 hmodule 是否为 kernel32.dll 的句柄  

    005905ED    74 15            JE SHORT 00590604  

    005905EF    90                NOP  

    005905F0    90                NOP  

    005905F1    90                NOP  

    005905F2    90                NOP  

    005905F3    39B5 16204000    CMP DWORD PTR SS:\[EBP+402016\],ESI        ; 判断 hmodule 是否为 user32.dll 的句柄  

    005905F9    74 09            JE SHORT 00590604  

    005905FB    90                NOP  

    005905FC    90                NOP  

    005905FD    90                NOP  

    005905FE    90                NOP  

    005905FF    EB 63            JMP SHORT 00590664  

    00590601    90                NOP  

    00590602    90                NOP  

    00590603    90                NOP  

    00590604    80BD 16564100 0>CMP BYTE PTR SS:\[EBP+415616\],0            ; 如果是 kernel32.dll 或 user32.dll 的话，再判断是否要加密特殊函数  

    0059060B    74 57            JE SHORT 00590664  

    0059060D    90                NOP  

    0059060E    90                NOP  

    0059060F    90                NOP  

    00590610    90                NOP  

    00590611    EB 07            JMP SHORT 0059061A  
```
