
```
00413300    90                  NOP  

00413301    90                  NOP  

00413302    90                  NOP  

00413303    90                  NOP  

00413304    90                  NOP  

00413305    90                  NOP  

00413306    90                  NOP  

00413307    90                  NOP  

00413308    90                  NOP  

00413309    90                  NOP  

0041330A    90                  NOP  

0041330B    90                  NOP  

0041330C    90                  NOP  

0041330D    90                  NOP  

0041330E    90                  NOP  

0041330F    90                  NOP  

00413310    BB 3C080000         MOV EBX,83C                             ; 重定位 api 大小  

00413315    0BDB                OR EBX,EBX  

00413317    74 37               JE SHORT 00413350                       ; 如果重定位 API 大小为 0 就跳  

00413319    2BC0                SUB EAX,EAX  

0041331B    2185 D14B4000       AND DWORD PTR SS:[EBP+404BD1],EAX  

00413321    E8 01000000         CALL 00413327  

00413326    90                  NOP  

00413327    59                  POP ECX  

00413328    6A 40               PUSH 40                                 ; /Protect = PAGE\_EXECUTE\_READWRITE  

0041332A    68 00300000         PUSH 3000                               ; |AllocationType = MEM\_COMMIT|MEM\_RESERVE  

0041332F    53                  PUSH EBX                                  ; |Size = 83C (2108.)  

00413330    50                  PUSH EAX                                  ; |Address = NULL  

00413331    8D6424 FC           LEA ESP,DWORD PTR SS:[ESP-4]                ; |  

00413335    81C1 23000000       ADD ECX,23                                ; |  

0041333B    890C24              MOV DWORD PTR SS:[ESP],ECX                ; |Return Address  

0041333E    FFA5 0E4C4000       JMP DWORD PTR SS:[EBP+404C0E]             ; \\VirtualAlloc  

00413344    90                  NOP  

00413345    85C0                TEST EAX,EAX  

00413347    74 21               JE SHORT 0041336A  

00413349    50                  PUSH EAX  

0041334A    8F85 C94B4000       POP DWORD PTR SS:[EBP+404BC9]             ; [EBP+404BC9]保存 hmem(00D20000)  

00413350    8D85 4A0D3400       LEA EAX,DWORD PTR SS:[EBP+340D4A]  

00413356    8D80 5F320C00       LEA EAX,DWORD PTR DS:[EAX+C325F]  

0041335C    48                  DEC EAX  

0041335D    FFD0                CALL EAX                                  ; 004133A3  

……  

00414F25    6A 04               PUSH 4                                  ; /Protect = PAGE\_READWRITE  

00414F27    68 00300000         PUSH 3000                               ; |AllocationType = MEM\_COMMIT|MEM\_RESERVE  

00414F2C    51                  PUSH ECX                                  ; |Size = 5C (92.)  

00414F2D    6A 00               PUSH 0                                    ; |Address = NULL  

00414F2F    53                  PUSH EBX                                  ; |Return address  

00414F30    FFA5 0E4C4000       JMP DWORD PTR SS:[EBP+404C0E]             ; \\VirtualAlloc  

00414F36    0F01FE              INVLPG DH                                 ; Privileged command  

00414F39    8DB5 AA5A4000       LEA ESI,DWORD PTR SS:[EBP+405AAA]  

00414F3F    97                  XCHG EAX,EDI  

00414F40    8BDF                MOV EBX,EDI  

00414F42    B9 22000000         MOV ECX,22  

00414F47    F3:A4               REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[ESI]   ; From:414ea5 to:D30000 szie:22  

……  

0041340C    3BB5 C34B4000       CMP ESI,DWORD PTR SS:[EBP+404BC3]           ; ESI 保存输入表的起始地址 0040C160  

……  

00413468    8B5E 0C             MOV EBX,DWORD PTR DS:[ESI+C]  

0041346B    039D C34B4000       ADD EBX,DWORD PTR SS:[EBP+404BC3]  

00413471    8BFB                MOV EDI,EBX                               ; 第一个 API 的 Name 地址  

……  

00413473    E8 4C120000         CALL 004146C4                             ; 进去就是还原 DLL 名  
```

进去看看:

```
004146C4    57                  PUSH EDI  

004146C5    800F 00             OR BYTE PTR DS:[EDI],0                    ; 如果获取完全部的 DLL 就直接返回，否则 not 还原出正确的 DLL 名  

004146C8    74 16               JE SHORT 004146E0  

004146CA    90                  NOP  

004146CB    90                  NOP  

004146CC    90                  NOP  

004146CD    90                  NOP  

004146CE    90                  NOP  

004146CF    90                  NOP  

004146D0    90                  NOP  

004146D1    90                  NOP  

004146D2    90                  NOP  

004146D3    90                  NOP  

004146D4    90                  NOP  

004146D5    90                  NOP  

004146D6    90                  NOP  

004146D7    90                  NOP  

004146D8    90                  NOP  

004146D9    90                  NOP  

004146DA    90                  NOP  

004146DB    F617                NOT BYTE PTR DS:[EDI]  

004146DD    47                  INC EDI  

004146DE    EB E5               JMP SHORT 004146C5  

004146E0    5F                  POP EDI                                   ; PESpin.0040C4C8  

004146E1    C3                  RETN  

……  

0041347F    53                  PUSH EBX                                  ; /FileName = "KERNEL32.DLL  

00413480    50                  PUSH EAX                                  ; |  

00413481    FFB5 F54B4000       PUSH DWORD PTR SS:[EBP+404BF5]             ; \\LoadLibraryA  

00413487    814424 04 14000000  ADD DWORD PTR SS:[ESP+4],14  

……  

00413491    85C0                TEST EAX,EAX  

00413493    0F84 3F090000       JE 00413DD8                               ; 如果载入失败则 OVER  

00413499    E8 01000000         CALL 0041349F  

0041349E    90                  NOP  

0041349F    59                  POP ECX  

004134A0    50                  PUSH EAX  

004134A1    51                  PUSH ECX  

004134A2    55                  PUSH EBP  

004134A3    810424 12374000     ADD DWORD PTR SS:[ESP],00403712  

004134AA    814424 04 22000000  ADD DWORD PTR SS:[ESP+4],22  

004134B2    C3                  RETN                      ;这里进去相当于 GetModuleHandleA 获取 DLL 的句柄  

……  

004134C1    2BD2                SUB EDX,EDX                               ; 获取到句柄后把原有 DLL 的函数名给清 0  

……  

004134F0    800B 00             OR BYTE PTR DS:[EBX],0  

004134F3    74 0D               JE SHORT 00413502                       ; 如果全部清除完毕则跳  

004134F5    8813                MOV BYTE PTR DS:[EBX],DL                ; DLL 名清 0  

004134F7    C1C2 04             ROL EDX,4  

004134FA    90                  NOP  

004134FB    90                  NOP  

004134FC    90                  NOP  

004134FD    43                  INC EBX  

004134FE    FF6424 FC           JMP DWORD PTR SS:[ESP-4]  

00413502    93                  XCHG EAX,EBX  

00413503    8B56 10             MOV EDX,DWORD PTR DS:[ESI+10]  

00413506    0395 C34B4000       ADD EDX,DWORD PTR SS:[EBP+404BC3]         ; 定位 ThunkValue  

0041350C    830A 00             OR DWORD PTR DS:[EDX],0  

0041350F    0F84 59010000       JE 0041366E                               ; 如果该 DLL 的 API 处理完则跳去下一步  

00413515    90                  NOP  

00413516    90                  NOP  

00413517    90                  NOP  

00413518    90                  NOP  

00413519    90                  NOP  

0041351A    90                  NOP  

0041351B    90                  NOP  

0041351C    90                  NOP  

0041351D    90                  NOP  

0041351E    75 02               JNZ SHORT 00413522  

00413520    90                  NOP  

00413521    90                  NOP  

00413522    8B02                MOV EAX,DWORD PTR DS:[EDX]
```
