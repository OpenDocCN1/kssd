
### 016F:00E5F984 - 016F:00E5F9C4 代码段

```
016F:00E5F984  LODSB   
016F:00E5F985  CMP      AL,00   
016F:00E5F988  JZ       00E5F967       ;该 DLL 处理完了吗?   
016F:00E5F98A  CMP      AL,06          ;特殊 Import function ?  如 GetVersion, 下面有列出   
016F:00E5F98D  JNZ      00E5F995       ;不是就跳(本想把这里改为 jmp, 就以为可以得到完整的 IAT, 可惜再细看后才发现该特殊 Import function name 根本就没有存起来!   
016F:00E5F98F  ADD      DWORD [EBP-08],BYTE +04  ;是特殊 Import function 就跳过不处理, 后面有说这些是怎么处理的   
016F:00E5F993  JMP      SHORT 00E5F984   
016F:00E5F995  PUSH     EBX            ;<--DLL name   
016F:00E5F996  PUSH     ESI            ;<--Encrpyted Function Name   
016F:00E5F997  PUSH     EBX   
016F:00E5F998  LEA      EBX,[EBP-08]  ;<--IAT   
016F:00E5F99B  PUSH     EBX   
016F:00E5F99C  CMP      AL,02          ;import function by name or ord?   
016F:00E5F99F  JZ       00E5F9A7   
016F:00E5F9A1  MOVZX    ECX,BYTE [ESI]   
016F:00E5F9A4  INC      ECX   
016F:00E5F9A5  JMP      SHORT 00E5F9AC   
016F:00E5F9A7  MOV      ECX,04   
016F:00E5F9AC  ADD      ESI,ECX  ;esi --> end of Encrpyted Function by name or import function by ord?   
016F:00E5F9AE  CALL     00E5F674  ;<--看下面的 00E5F674   
016F:00E5F9B3  POP      EBX   
016F:00E5F9B4  JMP      SHORT 00E5F984   
016F:00E5F9B6  POPA     
016F:00E5F9B7  CALL     00E5F9F6   
016F:00E5F9BC  PUSH     DWORD 00E5F9C5   
016F:00E5F9C1  INC      DWORD [ESP]   
016F:00E5F9C4  RET      
```

调用 `GetProcAddress`, 生成重定向代码的函数:

### 016F:00E5F674 - 016F:00E5F784 代码段

```
016F:00E5F674  PUSH     EBP   
016F:00E5F675  MOV      EBP,ESP   
016F:00E5F677  ADD      ESP,FFFFFEFC   
016F:00E5F67D  PUSH     EBX   
016F:00E5F67E  PUSH     ESI   
016F:00E5F67F  PUSH     EDI   
016F:00E5F680  MOV      ESI,[EBP+10]   
016F:00E5F683  MOV      EDI,[EBP+08]   
016F:00E5F686  MOV      EAX,ESI   
016F:00E5F688  DEC      EAX   
016F:00E5F689  XOR      EBX,EBX   
016F:00E5F68B  MOV      BL,[EAX]   
016F:00E5F68D  LEA      EAX,[EBP+FFFFFEFF]   
016F:00E5F693  XOR      ECX,ECX   
016F:00E5F695  MOV      EDX,0100   
016F:00E5F69A  CALL     00E52768       ;<--清空 eax 指向的空间   
016F:00E5F69F  MOV      EAX,EBX   
016F:00E5F6A1  DEC      EAX   
016F:00E5F6A2  SUB      EAX,BYTE +02   
016F:00E5F6A5  JC       00E5F700   
016F:00E5F6A7  JZ       NEAR 00E5F772   
016F:00E5F6AD  DEC      EAX   
016F:00E5F6AE  JZ       00E5F700   
016F:00E5F6B0  DEC      EAX   
016F:00E5F6B1  JNZ      NEAR 00E5F77B   
016F:00E5F6B7  MOV      AL,[ESI]   
016F:00E5F6B9  MOV      [EBP-01],AL   
016F:00E5F6BC  INC      ESI   
016F:00E5F6BD  XOR      EBX,EBX   
016F:00E5F6BF  MOV      BL,[EBP-01]   
016F:00E5F6C2  MOV      ECX,EBX   
016F:00E5F6C4  LEA      EAX,[EBP+FFFFFEFF]   
016F:00E5F6CA  MOV      EDX,ESI   
016F:00E5F6CC  CALL     00E5449C     ;<--Copy encrypted function name to eax 指向的空间   
016F:00E5F6D1  PUSH     BYTE +0A   
016F:00E5F6D3  MOV      ECX,00E639BA   
016F:00E5F6D8  MOV      EDX,EBX   
016F:00E5F6DA  LEA      EAX,[EBP+FFFFFEFF]   
016F:00E5F6E0  CALL     00E5C3E0            ;<--还原 Import Function Name   
016F:00E5F6E5  LEA      ESI,[EBP+FFFFFEFF]  ;<--esi 指向 Import Function Name   
016F:00E5F6EB  PUSH     ESI   
016F:00E5F6EC  MOV      EAX,[EBP+0C]         ;<--eax 指向 dll name   
016F:00E5F6EF  PUSH     EAX   
016F:00E5F6F0  CALL     00E5F30C            ;<--GetProcAddress by name，在 eax 返回地址   
016F:00E5F6F5  CALL     00E5F578            ;<--(下面有说明) 在返回的 eax 指向的空间生成跳到 Import Function 的代码?   
                                          ;可以将这句 call nop 掉, 直接将 EAX 放到 [EDX] 中,   
                                          ;整个输入表就差那些特殊的 Import functions 了要重建了   
                                          ;(当然 Import REConstructor v1.2 已经可以重建改输入表了, 不用这么麻烦)   
                                          ;要是真的改了该代码, 要在处理完所有的 DLL 后, 还原该代码, 因为后面有自检   
016F:00E5F6FA  MOV      EDX,[EDI]   
016F:00E5F6FC  MOV      [EDX],EAX           ;<-- 重定向 IAT?   
016F:00E5F6FE  JMP      SHORT 00E5F77B       ;<--完成   
016F:00E5F700  CMP      EBX,BYTE +01        ;encrypted funciont name?   
016F:00E5F703  JZ       00E5F70A   
016F:00E5F705  CMP      EBX,BYTE +04        ;Import function by ord?   
016F:00E5F708  JNZ      00E5F741   
016F:00E5F70A  MOV      AL,[ESI]   
016F:00E5F70C  MOV      [EBP-01],AL   
016F:00E5F70F  INC      ESI   
016F:00E5F710  XOR      ECX,ECX   
016F:00E5F712  MOV      CL,[EBP-01]   
016F:00E5F715  LEA      EAX,[EBP+FFFFFEFF]   
016F:00E5F71B  MOV      EDX,ESI   
016F:00E5F71D  CALL     00E5449C            ;copy encrypted function name to [eax]   
016F:00E5F722  PUSH     BYTE +0A   
016F:00E5F724  MOV      ECX,00E639B0   
016F:00E5F729  XOR      EDX,EDX   
016F:00E5F72B  MOV      DL,[EBP-01]   
016F:00E5F72E  LEA      EAX,[EBP+FFFFFEFF]   
016F:00E5F734  CALL     00E5C3E0            ;decrypt function name   
016F:00E5F739  LEA      ESI,[EBP+FFFFFEFF]   
016F:00E5F73F  JMP      SHORT 00E5F743   
016F:00E5F741  MOV      ESI,[ESI]   
016F:00E5F743  CMP      EBX,BYTE +04   
016F:00E5F746  JNZ      00E5F762   
016F:00E5F748  PUSH     ESI                 ;<--Import function name is GetProcAddress ?   
016F:00E5F749  MOV      EAX,[EBP+0C]   
016F:00E5F74C  PUSH     EAX   
016F:00E5F74D  CALL     00E5F30C   
016F:00E5F752  MOV      [00E6355C],EAX   
016F:00E5F757  MOV      EAX,00E5C490        ;<--00E5C490 处的代码自己看吧, 应该是 GetProcAddress   
016F:00E5F75C  MOV      EDX,[EDI]   
016F:00E5F75E  MOV      [EDX],EAX   
016F:00E5F760  JMP      SHORT 00E5F77B   
016F:00E5F762  PUSH     ESI         ;<--esi 指向 Import Function Name or esi is import function ord   
016F:00E5F763  MOV      EAX,[EBP+0C]   
016F:00E5F766  PUSH     EAX         ;<--eax 指向 dll name   
016F:00E5F767  CALL     00E5F30C       ;<--GetProcAddress by ord，在 eax 返回地址   
016F:00E5F76C  MOV      EDX,[EDI]   
016F:00E5F76E  MOV      [EDX],EAX   
016F:00E5F770  JMP      SHORT 00E5F77B   
016F:00E5F772  MOV      EAX,00E5C468   
016F:00E5F777  MOV      EDX,[EDI]   
016F:00E5F779  MOV      [EDX],EAX   
016F:00E5F77B  ADD      DWORD [EDI],BYTE +04   
016F:00E5F77E  POP      EDI   
016F:00E5F77F  POP      ESI   
016F:00E5F780  POP      EBX   
016F:00E5F781  MOV      ESP,EBP   
016F:00E5F783  POP      EBP   
016F:00E5F784  RET      0C   
```

生成跳到真正 `Import Function` 地址的代码:

### 016F:00E5F578 - 016F:00E5F605 代码段

```
016F:00E5F578  PUSH     EBX   
016F:00E5F579  PUSH     ESI   
016F:00E5F57A  PUSH     EDI   
016F:00E5F57B  PUSH     EBP   
016F:00E5F57C  ADD      ESP,BYTE -10   
016F:00E5F57F  MOV      [ESP],EAX   
016F:00E5F582  MOV      EAX,C8   
016F:00E5F587  CALL     00E52568   
016F:00E5F58C  MOV      [ESP+04],EAX   
016F:00E5F590  MOV      EAX,[ESP+04]   
016F:00E5F594  MOV      [ESP+08],EAX   
016F:00E5F598  MOV      EBP,[ESP]   
016F:00E5F59B  CALL     00E5F524   
016F:00E5F5A0  MOV      EBX,EAX   
016F:00E5F5A2  MOV      BYTE [ESP+0C],00   
016F:00E5F5A7  MOVZX    ESI,BYTE [EBX]   
016F:00E5F5AA  LEA      EAX,[EBX+01]   
016F:00E5F5AD  MOVZX    EDI,BYTE [EAX]   
016F:00E5F5B0  LEA      EDX,[EBX+02]   
016F:00E5F5B3  MOV      ECX,ESI   
016F:00E5F5B5  MOV      EAX,EBP   
016F:00E5F5B7  CALL     00E5BF44   
016F:00E5F5BC  TEST     AL,AL   
016F:00E5F5BE  JZ       00E5F5D8   
016F:00E5F5C0  MOV      ECX,EDI   
016F:00E5F5C2  MOV      EDX,EBP   
016F:00E5F5C4  MOV      EAX,[ESP+08]   
016F:00E5F5C8  CALL     00E5449C   
016F:00E5F5CD  ADD      [ESP+08],EDI   
016F:00E5F5D1  ADD      EBP,EDI   
016F:00E5F5D3  MOV      BYTE [ESP+0C],01   
016F:00E5F5D8  ADD      ESI,BYTE +02   
016F:00E5F5DB  ADD      EBX,ESI   
016F:00E5F5DD  CMP      BYTE [ESP+0C],00   
016F:00E5F5E2  JNZ      00E5F5E9   
016F:00E5F5E4  CMP      BYTE [EBX],00   
016F:00E5F5E7  JNZ      00E5F5A7   
016F:00E5F5E9  CMP      BYTE [ESP+0C],00   
016F:00E5F5EE  JNZ      00E5F59B   
016F:00E5F5F0  CALL     00E52698   
016F:00E5F5F5  MOV      ESI,[ESP+08]   
016F:00E5F5F9  SUB      ESI,[ESP+04]   
016F:00E5F5FD  LEA      EAX,[ESI+06]   
016F:00E5F600  CALL     00E52568   
016F:00E5F605  MOV      EBX,EAX   
```
