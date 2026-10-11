
```assembly
011F62E2    B8 58140000     MOV EAX,1458   
011F62E7    8D8D DB004B00   LEA ECX,DWORD PTR SS:\[EBP+4B00DB\]   
011F62ED    8901            MOV DWORD PTR DS:\[ECX\],EAX   
011F62EF    8D8D 69014B00   LEA ECX,DWORD PTR SS:\[EBP+4B0169\]   
011F62F5    8D85 69134B00   LEA EAX,DWORD PTR SS:\[EBP+4B1369\]   
011F62FB    51              PUSH ECX   
011F62FC    50              PUSH EAX   
011F62FD    E8 7BFFFFFF     CALL 011F627D   
011F6302    64:EB 02        JMP SHORT 011F6307          //F4 直接下来。    
011F6307   /2E:EB 01        JMP SHORT 011F630B                          ; 多余的前缀   
011F630B    83EC 1A         SUB ESP,1A   
011F630E    83C4 16         ADD ESP,16   
011F6311    66:8135 1A631F0>XOR WORD PTR DS:\[11F631A\],0C6CB   
011F7E0A    F2:             PREFIX REPNE:                       ; 多余的前缀   
011F7E0E    50              PUSH EAX   
011F7E0F    2E:EB 02        JMP SHORT 011F7E14                          ; 多余的前缀   
011F7E14    8F4424 00       POP DWORD PTR SS:\[ESP\]             ; 011F7DF2   
011F7E18    65:EB 02        JMP SHORT 011F7E1D                          ; 多余的前缀   
011F7E1D    F2:             PREFIX REPNE:                       ; 多余的前缀   
011F7E21    66:0FBCC7       BSF AX,DI   
011F7E25    66:812D 2E7E1F0>SUB WORD PTR DS:\[11F7E2E\],6B2A   
011F7E2E    F2:             PREFIX REPNE:                       ; 多余的前缀   
011F7E33    83E0 67         AND EAX,67   
011F7E36    66:812D 3F7E1F0>SUB WORD PTR DS:\[11F7E3F\],0B47A   
011F7E3F   /EB 01           JMP SHORT 011F7E42   
011F7E42   /EB 02           JMP SHORT 011F7E46   
011F7E46    8D83 3A184A79   LEA EAX,DWORD PTR DS:\[EBX+794A183A\]   
011F7E4C    F2:             PREFIX REPNE:                       ; 多余的前缀   
011F7E51    66:812D 5B7E1F0>SUB WORD PTR DS:\[11F7E5B\],98C5   
011F7E5A   /3E:EB 02        JMP SHORT 011F7E5F                          ; 多余的前缀   
011F7E5F    06              PUSH ES   
011F7E60    EB 02           JMP SHORT 011F7E64   
011F7E64    C74424 00 477E1>MOV DWORD PTR SS:\[ESP\],11F7E47   
011F7E6C    58              POP EAX   
011F7E6D    FF50 2B         CALL DWORD PTR DS:\[EAX+2B\]   
011F7E7B    58              POP EAX                       ; 011F7E70   
011F7E7C    EB 01           JMP SHORT 011F7E7F   
011F7E7F   /EB 02           JMP SHORT 011F7E83   
011F7E83    8D4458 6F       LEA EAX,DWORD PTR DS:\[EAX+EBX\*2+6F\]   
011F7E87    F2:             PREFIX REPNE:                       ; 多余的前缀   
011F7E8B    F2:             PREFIX REPNE:                       ; 多余的前缀   
011F7E90    8D81 4CDFC21F   LEA EAX,DWORD PTR DS:\[ECX+1FC2DF4C\]   
011F7E96    58              POP EAX   
011F7E97    64:EB 02        JMP SHORT 011F7E9C                          ; 多余的前缀   
011F7E9C    55              PUSH EBP      \*     //这里看到 ESP 的值是 0012FFA4，难怪 12ffc0 的硬件写入断点没断下来，EBP 的值也不是 12fff0 模拟跟踪也失败，所以这个程序用 0012FFA0 的硬件写入断点一步就可以断下来，不过手动跟踪也非常快。    
011F7E9D    8BEC            MOV EBP,ESP   \*    //执行的结果是 esp=ebp=12ffa0    
011F7E9F    6A FF           PUSH -1       \*        带\*的内容是 Stolen Code   
011F7EA1    68 30284700     PUSH 472830   \*   
011F7EA6    68 E4234400     PUSH 4423E4   \*   
011F7EAB    64:A1 00000000  MOV EAX,DWORD PTR FS:\[0\]   \*   
011F7EB1    64:EB 02        JMP SHORT 011F7EB6                          ; 多余的前缀   
011F7EB6    50              PUSH EAX      \*   
011F7EB7    64:8925 0000000>MOV DWORD PTR FS:\[0\],ESP   \*   
011F7EBE    83EC 58         SUB ESP,58           \*   
011F7EC1    64:EB 02        JMP SHORT 011F7EC6                          ; 多余的前缀   
011F7EC6    53              PUSH EBX    \*   
011F7EC7    64:EB 02        JMP SHORT 011F7ECC                          ; 多余的前缀   
011F7ECC    56              PUSH ESI    \*   
011F7ECD    64:EB 02        JMP SHORT 011F7ED2                          ; 多余的前缀   
011F7ED2    57              PUSH EDI    \*   
011F7ED3    8965 E8         MOV DWORD PTR SS:\[EBP\-18\],ESP  \*   
011F7ED6    64:EB 02        JMP SHORT 011F7EDB                          ; 多余的前缀   
011F7EDB   /64:EB 01        JMP SHORT 011F7EDF                          ; 多余的前缀   
011F7EDF    68 EA3E4400     PUSH 443EEA   
011F7EE4    68 5B7D1F01     PUSH 11F7D5B   
011F7EE9    C3              RETN   
011F7D5B    60              PUSHAD   
011F7D5C    9C              PUSHFD   
011F7D5D    FC              CLD   
011F7D5E    BF 9A7D1F01     MOV EDI,11F7D9A   
011F7D63    B9 58140000     MOV ECX,1458   
011F7D68    F3:AA           REP STOS BYTE PTR ES:\[EDI\]   //F7 可以看着它循环执行 Stolen Code     
011F7D6A    9D              POPFD   
011F7D6B    61              POPAD   
011F7D6C    C3              RETN   
```

整理一下 Stolen Code 

```assembly
PUSH EBP   
MOV EBP,ESP   
PUSH -1   
PUSH 00472830   
PUSH 004423E4               
MOV EAX,DWORD PTR FS:\[0\]   
PUSH EAX   
MOV DWORD PTR FS:\[0\],ESP   
SUB ESP,58   
PUSH EBX   
PUSH ESI   
PUSH EDI   
MOV DWORD PTR SS:\[EBP\-18\],ESP   
```

```assembly
00443EC4    0000            ADD BYTE PTR DS:\[EAX\],AL        //真 Oep,OD 直接补上整理的 Stolen Code     
00443EC6    0000            ADD BYTE PTR DS:\[EAX\],AL   
00443EC8    0000            ADD BYTE PTR DS:\[EAX\],AL   
00443ECA    0000            ADD BYTE PTR DS:\[EAX\],AL   
00443ECC    0000            ADD BYTE PTR DS:\[EAX\],AL   
00443ECE    0000            ADD BYTE PTR DS:\[EAX\],AL   
00443ED0    0000            ADD BYTE PTR DS:\[EAX\],AL   
00443ED2    0000            ADD BYTE PTR DS:\[EAX\],AL   
00443ED4    0000            ADD BYTE PTR DS:\[EAX\],AL   
00443ED6    0000            ADD BYTE PTR DS:\[EAX\],AL   
00443ED8    0000            ADD BYTE PTR DS:\[EAX\],AL   
00443EDA    0000            ADD BYTE PTR DS:\[EAX\],AL   
00443EDC    0000            ADD BYTE PTR DS:\[EAX\],AL   
00443EDE    0000            ADD BYTE PTR DS:\[EAX\],AL   
00443EE0    0000            ADD BYTE PTR DS:\[EAX\],AL   
00443EE2    0000            ADD BYTE PTR DS:\[EAX\],AL   
00443EE4    0000            ADD BYTE PTR DS:\[EAX\],AL   
00443EE6    0000            ADD BYTE PTR DS:\[EAX\],AL   
00443EE8    0000            ADD BYTE PTR DS:\[EAX\],AL   
00443EEA    FF15 C4834600   CALL DWORD PTR DS:\[4683C4\]      伪 OEP,滚动条往上面看。  
00443EF0    33D2            XOR EDX,EDX   
00443EF2    8AD4            MOV DL,AH   
00443EF4    8915 C0F24800   MOV DWORD PTR DS:\[48F2C0\],EDX   
00443EFA    8BC8            MOV ECX,EAX   
00443EFC    81E1 FF000000   AND ECX,0FF   
00443F02    890D BCF24800   MOV DWORD PTR DS:\[48F2BC\],ECX   
00443F08    C1E1 08         SHL ECX,8   
00443F0B    03CA            ADD ECX,EDX   
00443F0D    890D B8F24800   MOV DWORD PTR DS:\[48F2B8\],ECX   
00443F13    C1E8 10         SHR EAX,10   
00443F16    A3 B4F24800     MOV DWORD PTR DS:\[48F2B4\],EAX   
00443F1B    6A 01           PUSH 1   
00443F1D    E8 84330000     CALL RepairVi.004472A6   
```

```assembly
00443EC4    55              PUSH EBP        //OD 用插件脱壳，修正入口为 43Ec4 重建输入表选方式 2    
00443EC5    8BEC            MOV EBP,ESP   
00443EC7    6A FF           PUSH -1   
00443EC9    68 30284700     PUSH RepairVi.00472830   
00443ECE    68 E4234400     PUSH RepairVi.004423E4   
00443ED3    64:A1 00000000  MOV EAX,DWORD PTR FS:\[0\]   
00443ED9    50              PUSH EAX   
00443EDA    64:8925 0000000>MOV DWORD PTR FS:\[0\],ESP   
00443EE1    83EC 58         SUB ESP,58   
00443EE4    53              PUSH EBX   
00443EE5    56              PUSH ESI   
00443EE6    57              PUSH EDI   
00443EE7    8965 E8         MOV DWORD PTR SS:\[EBP\-18\],ESP   
00443EEA    FF15 C4834600   CALL DWORD PTR DS:\[4683C4\]   
00443EF0    33D2            XOR EDX,EDX   
00443EF2    8AD4            MOV DL,AH   
00443EF4    8915 C0F24800   MOV DWORD PTR DS:\[48F2C0\],EDX   
00443EFA    8BC8            MOV ECX,EAX   
00443EFC    81E1 FF000000   AND ECX,0FF   
```
