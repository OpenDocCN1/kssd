
```
004A5C7C    328429 C2BC4000  XOR AL,BYTE PTR DS:[ECX+EBP+40BCC2]      ; 查表后的值和上面计算自己进程名的值 xor  
004A5C83    0AC0            OR AL,AL                                ; 其中有一位不同  
004A5C85    74 05           JE SHORT 004A5C8C                       ; 如果相等就跳  
004A5C87    BA 01000000     MOV EDX,1                               ; 不同则 edx 设置为 1  
004A5C8C    41              INC ECX  
004A5C8D  ^ EB D8           JMP SHORT 004A5C67  
004A5C8F    83FA 01         CMP EDX,1                               ; 比较如果不相同的话就不跳  
004A5C92    75 6C           JNZ SHORT 004A5D00                      ; 如果用 OD 来调试的子进程话这里就一定要跳,否则后面不会跳去正确的 OEP 处  
……  
004A5CAA    BE 9A7C4000     MOV ESI,00407C9A  
004A5CC9    81EE FF000000   SUB ESI,0FF  
004A5CCF    03F5            ADD ESI,EBP  
……  
004A5CE7    C706 E77D4000   MOV DWORD PTR DS:[ESI],00407DE7          ; 这几句不知道是什么意思:-(  
004A5CED    83C6 1F         ADD ESI,1F  
004A5CF0    8906            MOV DWORD PTR DS:[ESI],EAX  
004A5CF2    FFB5 FFC24000   PUSH DWORD PTR SS:[EBP+40C2FF]           ; /hObject = 0000001C  
004A5CF8    FF95 4DBC4000   CALL DWORD PTR SS:[EBP+40BC4D]           ; \CloseHandle  
004A5CFE    EB 15           JMP SHORT 004A5D15  
004A5D00    57              PUSH EDI                              ; /pProcessentry = Explorer.004AA303  
004A5D01    FFB5 FBC24000   PUSH DWORD PTR SS:[EBP+40C2FB]           ; |hSnapshot = 00000010  
004A5D07    FF95 15BC4000   CALL DWORD PTR SS:[EBP+40BC15]           ; \Process32Next  
004A5D0D    0BC0            OR EAX,EAX  
004A5D0F  ^ 0F85 3CFFFFFF   JNZ 004A5C51                            ; 如果没有枚举完则继续  
……  
004A5D2F    FFB5 FBC24000   PUSH DWORD PTR SS:[EBP+40C2FB]           ; /hObject = 00000010  
004A5D5E    FF95 4DBC4000   CALL DWORD PTR SS:[EBP+40BC4D]           ; \CloseHandle 关闭对象  
```

到这里为些壳的 `ANTI-Debug` 就结束了。

```
……  
004A5D7A    FF95 99BB4000   CALL DWORD PTR SS:[EBP+40BB99]           ; 获取命令行  
004A5D80    50              PUSH EAX  
004A5D81    90              NOP  
004A5D82    90              NOP  
……  
004A5DAD    5A              POP EDX                             ; 00141EE0  
004A5DAE    8BF2            MOV ESI,EDX  
004A5DB0    90              NOP  
……  
004A5DC6    803E 58         CMP BYTE PTR DS:[ESI],58              ; 比较命令行第一个字符是否为"X",调试标志  
004A5DC9    75 2E           JNZ SHORT 004A5DF9                      ; 不是则跳  
……  
004A5DF7   /EB 6B           JMP SHORT 004A5E64                      ; 如果是子进程则跳去解壳部分  
……  
004A5E0F    90              NOP  
004A5E10    90              NOP                                   ; 准备获取程序的完整路径名  
004A5E11    90              NOP  
004A5E12    90              NOP  
004A5E13    68 00010000     PUSH 100                              ; /BufSize = 100 (256.)  
004A5E18    8D85 C2BC4000   LEA EAX,DWORD PTR SS:[EBP+40BCC2]        ; |  
004A5E1E    50              PUSH EAX                              ; |PathBuffer = Explorer.004A9CC2  
004A5E1F    6A 00           PUSH 0                                ; |hModule = NULL  
004A5E21    FF95 37BB4000   CALL DWORD PTR SS:[EBP+40BB37]           ; \GetModuleFileNameA  
004A5E27    90              NOP  
……  
004A5E3A    8D85 C2BC4000   LEA EAX,DWORD PTR SS:[EBP+40BCC2]  
004A5E40    50              PUSH EAX                              ; 获取到的程序完整路径名入栈  
004A5E41    E8 1196FFFF     CALL 0049F457                           ; 这里跟进就是 Shell Application(debug process)  
```

跟进去看看：

```
0049F457    55              PUSH EBP                              ; 进来 CreateProcess 部分  
0049F458    8BEC            MOV EBP,ESP  
0049F45A    60              PUSHAD  
0049F45B    8B7D 08         MOV EDI,DWORD PTR SS:[EBP+8]             ; 程序完整路径存放地址入 edi  
0049F45E    E8 00000000     CALL 0049F463  
0049F463    5B              POP EBX  
0049F464    81EB 63144000   SUB EBX,00401463                        ; 获取重定位地址  
0049F46A    B8 44000000     MOV EAX,44                              ; 准备申请内存空间  
0049F46F    50              PUSH EAX                              ; /Length = 44 (68.)  
0049F470    8D83 42C24000   LEA EAX,DWORD PTR DS:[EBX+40C242]        ; |  
0049F476    50              PUSH EAX                              ; |Destination = Explorer.004AA242  
0049F477    FF93 D3BB4000   CALL DWORD PTR DS:[EBX+40BBD3]           ; \RtlZeroMemory  
0049F47D    B8 10000000     MOV EAX,10  
0049F482    50              PUSH EAX                              ; /Length = 10 (16.)  
0049F483    8D83 86C24000   LEA EAX,DWORD PTR DS:[EBX+40C286]        ; |  
0049F489    50              PUSH EAX                              ; |Destination = Explorer.004AA286  
0049F48A    FF93 D3BB4000   CALL DWORD PTR DS:[EBX+40BBD3]           ; \RtlZeroMemory  
0049F490    B8 44000000     MOV EAX,44                              ; 准备 CreateProcessA 调试子进程  
0049F495    8983 42C24000   MOV DWORD PTR DS:[EBX+40C242],EAX  
0049F49B    8D83 86C24000   LEA EAX,DWORD PTR DS:[EBX+40C286]  
0049F4A1    50              PUSH EAX                              ; /pProcessInfo = Explorer.004AA286  
0049F4A2    8D83 42C24000   LEA EAX,DWORD PTR DS:[EBX+40C242]        ; |  
0049F4A8    50              PUSH EAX                              ; |pStartupInfo = Explorer.004AA242  
0049F4A9    6A 00           PUSH 0                                ; |CurrentDir = NULL  
0049F4AB    6A 00           PUSH 0                                ; |pEnvironment = NULL  
0049F4AD    B8 01000000     MOV EAX,1  
0049F4B2    83C8 02         OR EAX,2                              ; |  
0049F4B5    50              PUSH EAX                              ; |CreationFlags = DEBUG_PROCESS|DEBUG_ONLY_THIS_PROCESS  
0049F4B6    6A 00           PUSH 0                                ; |InheritHandles = FALSE  
0049F4B8    6A 00           PUSH 0                                ; |pThreadSecurity = NULL  
0049F4BA    6A 00           PUSH 0                                ; |pProcessSecurity = NULL  
0049F4BC    8D83 C2BD4000   LEA EAX,DWORD PTR DS:[EBX+40BDC2]        ; |  
0049F4C2    50              PUSH EAX                              ; |CommandLine = "X"  
0049F4C3    57              PUSH EDI                              ; |ModuleFileName = "D:\\Explorer.exe"  
0049F4C4    FF93 59BB4000   CALL DWORD PTR DS:[EBX+40BB59]           ; \CreateProcessA  
0049F4CA    83F8 01         CMP EAX,1  
0049F4CD    0F85 87010000   JNZ 0049F65A                            ; 如果创建进程失败则退出程序  
0049F4D3    8DBB 64C54000   LEA EDI,DWORD PTR DS:[EBX+40C564]  
0049F4D9    C707 07000100   MOV DWORD PTR DS:[EDI],10007  
0049F4DF    810F 10000100   OR DWORD PTR DS:[EDI],10010  
0049F4E5    8D83 96C24000   LEA EAX,DWORD PTR DS:[EBX+40C296]  
0049F4EB    68 A00F0000     PUSH 0FA0                              ; /Timeout = 4000. ms  
0049F4F0    50              PUSH EAX                              ; |pDebugEvent = Explorer.004AA296  
0049F4F1    FF93 6FBB4000   CALL DWORD PTR DS:[EBX+40BB6F]           ; \WaitForDebugEvent  
0049F4F7    83F8 01         CMP EAX,1  
0049F4FA    0F85 55010000   JNZ 0049F655                            ; 如果 EAX ==FALSE 则跳下一步  
0049F500    8D93 96C24000   LEA EDX,DWORD PTR DS:[EBX+40C296]        ; DebugEvent  
0049F506    8DB3 86C24000   LEA ESI,DWORD PTR DS:[EBX+40C286]        ; pi  
0049F50C    833A 03         CMP DWORD PTR DS:[EDX],3                ; CREATE_PROCESS_DEBUG_EVENT  
0049F50F    75 16           JNZ SHORT 0049F527                      ; 如果调试动作不为 CREATE_PROCESS_DEBUG_EVENT 则跳  
0049F511    68 02000100     PUSH 10002                              ; /ContinueStatus = DBG_CONTINUE  
0049F516    FF72 08         PUSH DWORD PTR DS:[EDX+8]              ; |ThreadId = 214  
0049F519    FF72 04         PUSH DWORD PTR DS:[EDX+4]              ; |ProcessId = 57C  
```
