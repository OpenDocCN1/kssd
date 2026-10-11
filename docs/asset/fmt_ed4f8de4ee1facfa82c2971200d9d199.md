
### 异常处理循环

```asm
003449F1    B9 A7040000     MOV ECX,4A7                   ; size = 4A7, 345355  

003449F6    8A0431          MOV AL,BYTE PTR DS:[ECX+ESI]  

003449F9    CC              INT3                          ; Int3, 进入下面的异常处理  

003449FA    90              NOP  

003449FB    880431          MOV BYTE PTR DS:[ECX+ESI],AL  

003449FE  ^ E2 F6           LOOPD SHORT 003449F6          ; 要反复异常 4A7 次, 厉害  

00344A00    64:8F05 0000000> POP DWORD PTR FS:[0]         ; 拆除异常, 进入第四部分  

00344A07    58              POP EAX  

;SEH  

; 第一次进入 SEH 后先设好 Drx  

dr0 0012FCD4   0FFF0123  

dr1 0012FCD8   0FFF4567  

dr2 0012FCDC   0FFF89AB  

dr3 0012FCE0   0FFFCDEF  

003448D7    8B4C24 0C       MOV ECX,DWORD PTR SS:[ESP+C]    ; pContext=12FCD0  

003448DB    8B81 B0000000   MOV EAX,DWORD PTR DS:[ECX+B0]   ; regEAX  

003448E1    8B51 04         MOV EDX,DWORD PTR DS:[ECX+4]    ; Dr0  

00344911    F6D0            NOT AL  

00344913    32C2            XOR AL,DL  

00344915    66:25 FF00      AND AX,0FF  

00344930    66:03D0         ADD DX,AX  

00344933    66:C1CA 03      ROR DX,3  

00344937    66:8951 04      MOV WORD PTR DS:[ECX+4],DX      ; Dr0  

00344952    66:3151 08      XOR WORD PTR DS:[ECX+8],DX      ; Dr1  

00344956    66:8B51 08      MOV DX,WORD PTR DS:[ECX+8]  

0034495A    66:C1CA 02      ROR DX,2  

0034498B    66:0151 0C      ADD WORD PTR DS:[ECX+C],DX      ; Dr2  

0034498F    66:8B51 0C      MOV DX,WORD PTR DS:[ECX+C]      

00344993    66:F7D2         NOT DX  

00344996    66:2B51 10      SUB DX,WORD PTR DS:[ECX+10]     ; Dr3  

0034499A    66:D1CA         ROR DX,1  

003449CA    66:3151 04      XOR WORD PTR DS:[ECX+4],DX      ; Dr1  

003449CE    8981 B0000000   MOV DWORD PTR DS:[ECX+B0],EAX   ; regEAX  

003449D4    FF81 B8000000   INC DWORD PTR DS:[ECX+B8]       ; regEIP+1  

003449DA    33C0            XOR EAX,EAX  

003449DC    C3              RETN                            ; 第一次到这里后如下操做  

在 344A07 设软件断点, 忽略 Int3 异常,  F9, 停下后, 去 Stack 把 最终的 Drx 找回来, 第十一部分还要检查  

0012FCD4   0FFF4A3D  

0012FCD8   0FFF23B9  

0012FCDC   0FFF9002  

0012FCE0   0FFFCDEF  

0012FCE4   0  

0012FCE8   155  

进入第四部分  

;============================================================================================================================================================================  

### 四. 处理壳要用的 API  

00344B57    8D05 A9F24000   LEA EAX,DWORD PTR DS:[40F2A9]  

00344B5D    03C5            ADD EAX,EBP                   ; 3507F1  

00344B5F    8985 22F44000   MOV DWORD PTR SS:[EBP+40F422],EAX  

00344B65    8DB5 3BF54000   LEA ESI,DWORD PTR SS:[EBP+40F53B]    ; 350A83 "kernel32.dll"  

00344B6B    56              PUSH ESI  

00344B6C    8D85 37364000   LEA EAX,DWORD PTR SS:[EBP+403637]    ; 返回地址 344B7F  

00344B72    50              PUSH EAX  

00344B73    8B85 2AF44000   MOV EAX,DWORD PTR SS:[EBP+40F42A]    ; KERNEL32.GetModuleHandleA  

00344B79    E9 3EB70000     JMP 003502BC                  ; 3502BC 相当于 Call EAX, 返回地址前面给定  

00344B8E  

00344B7F    8BF0            MOV ESI,EAX                   ; KERNEL32.7C570000  

00344B81    8985 3BF54000   MOV DWORD PTR SS:[EBP+40F53B],EAX    ; 350A83, 覆盖 字符  

00344B87    8D9D 49F54000   LEA EBX,DWORD PTR SS:[EBP+40F549]    ; 350A91  

00344CDC    B9 21000000     MOV ECX,21                    ; 21 个 API(kernel32.dll)  

00344CE1    8DBD FCF94000   LEA EDI,DWORD PTR SS:[EBP+40F9FC]    ; 350F44  

00344CE7    53              PUSH EBX  

00344CE8    8A03            MOV AL,BYTE PTR DS:[EBX]  

00344CEA   /EB 06           JMP SHORT 00344CF2  

00344CEC   |F6D0            NOT AL                        ; 只是简单的 Not 解密字符  

00344CEE   |AA              STOS BYTE PTR ES:[EDI]  

00344CEF   |43              INC EBX  

00344CF0   |8A03            MOV AL,BYTE PTR DS:[EBX]  

00344CF2   \\0AC0           OR AL,AL  

00344CF4  ^ 75 F6           JNZ SHORT 00344CEC  

00344CF6    AA              STOS BYTE PTR ES:[EDI]  

00344CF7    5B              POP EBX                         ; 00350A91  

00344CF8    8DBD FCF94000   LEA EDI,DWORD PTR SS:[EBP+40F9FC]   ; 350F44 ( API 明文)  

00344CFE    51              PUSH ECX  

00344CFF    57              PUSH EDI  

00344D00    56              PUSH ESI  

00344D01    8D85 CC374000   LEA EAX,DWORD PTR SS:[EBP+4037CC]  ; 返回地址 344D14  

00344D07    50              PUSH EAX  

00344D08    8B85 22F44000   MOV EAX,DWORD PTR SS:[EBP+40F422]  ; 3507F1, 相当于 GetProcAddress  

00344D0E    E9 A9B50000     JMP 003502BC                    

00344D13  

00344D14    0FB64B FF       MOVZX ECX,BYTE PTR DS:[EBX-1]  

00344D18    8903            MOV DWORD PTR DS:[EBX],EAX      ; [350A91] 放 API  

00344D1A    03D9            ADD EBX,ECX  

00344D1C    43              INC EBX  

00344D1D    59              POP ECX  

00344D1E  ^ E2 C1           LOOPD SHORT 00344CE1  

有 `CloseHandle`, `CreateFileA`, `CreateFileMappingA`, `CreateThread`, `DeleteFileA`, `DeviceIoControl`,  

`ExitProcess`, `FindResourceA`, `GetCommandLineA`, `GetFileSize`, `GetCurrentProcess`, `GetCurrentProcessId`,  

`GetCurrentThread`, `GetModuleHandleA`, `GetModuleFileNameA`, `GetTempPathA`, `GetVersion`, `lstrcmp`,  

`LoadResource`, `MapViewOfFile`, `ReadProcessMemory`, `ResetEvent`, `SetEvent`, `SetLastError`, `SetThreadPriority`,  

`TerminateThread`, `UnmapViewOfFile`, `VirtualAllocEx`, `VirtualFree`, `VirtualProtect`, `WaitForSingleObject`,  

`WriteProcessMemory`, `WriteFile`  

00344D20    8DB5 AEF44000   LEA ESI,DWORD PTR SS:[EBP+40F4AE]  ; 3509F6 ("USER32.DLL")  

00344E75    56              PUSH ESI  

00344E76    8D85 41394000   LEA EAX,DWORD PTR SS:[EBP+403941]  ; 返回地址 344E89  

00344E7C    50              PUSH EAX  

00344E7D    8B85 2AF44000   MOV EAX,DWORD PTR SS:[EBP+40F42A]  ; KERNEL32.GetModuleHandleA  

00344E83   /E9 34B40000     JMP 003502BC                     

00344E88  

00344E89    0BC0            OR EAX,EAX                      ; 找不到  

00344E8B    75 15           JNZ SHORT 00344EA2  

00344E8D    56              PUSH ESI  

00344E8E    8D85 5A394000   LEA EAX,DWORD PTR SS:[EBP+40395A]  ; 返回地址 344EA2     

00344E94    50              PUSH EAX  

00344E95    8B85 2EF44000   MOV EAX,DWORD PTR SS:[EBP+40F42E]  ; KERNEL32.LoadLibraryA  

00344E9B   /E9 1CB40000     JMP 003502BC  

00344EA0  

00344EA1  

00344EA2    8BF0            MOV ESI,EAX                   ; USER32.77E10000  

00344EA4    8D9D BAF44000   LEA EBX,DWORD PTR SS:[EBP+40F4BA]  ; 350A02  

00344EAA    B9 08000000     MOV ECX,8  

00344EAF    8DBD FCF94000   LEA EDI,DWORD PTR SS:[EBP+40F9FC]  

00344EB5    53              PUSH EBX  

00344EAF    8DBD FCF94000   LEA EDI,DWORD PTR SS:[EBP+40F9FC]  ; 350F44  

00344EB5    53              PUSH EBX  

00344EE3    8A03            MOV AL,BYTE PTR DS:[EBX]  

00344EE5    EB 06           JMP SHORT 00344EED  

00344EE7    F6D0            NOT AL  

00344EE9    AA              STOS BYTE PTR ES:[EDI]  

00344EEA    43              INC EBX  

00344EEB    8A03            MOV AL,BYTE PTR DS:[EBX]  

00344EED    0AC0            OR AL,AL  

00344EEF  ^ 75 F6           JNZ SHORT 00344EE7  

00344EF1    AA              STOS BYTE PTR ES:[EDI]          ; 解密字符  

00344EF2    5B              POP EBX  

00344EF3    8DBD FCF94000   LEA EDI,DWORD PTR SS:[EBP+40F9FC]  ; 350F44  

00344EF9    51              PUSH ECX  

00344EFA    57              PUSH EDI  

00344EFB    56              PUSH ESI  

00344EFC    8D85 C8394000   LEA EAX,DWORD PTR SS:[EBP+4039C8]  ; 返回地址, 344F10  

00344F02    50              PUSH EAX  

00344F03    8B85 22F44000   MOV EAX,DWORD PTR SS:[EBP+40F422]  ; 3507F1, 相当于 GetProcAddress  

00344F09    E9 AEB30000     JMP 003502BC  
```
