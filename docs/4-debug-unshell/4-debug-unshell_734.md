

# Krypton v0.4

> 原文：[`www.pediy.com/kssd/pediy05/pediy50336.htm`](https://www.pediy.com/kssd/pediy05/pediy50336.htm)

首先载入目标，F9   
  
003D042F  8918        MOV DWORD PTR DS:\[EAX\],EBX  ---第一处异常   
003D0431  EB 3C        JMP SHORT 003D046F   
003D0433  DF69 4E      FILD QWORD PTR DS:\[ECX+4E\]   
003D0436  58          POP EAX   
003D0437  DF59 71      FISTP WORD PTR DS:\[ECX+71\]   
003D043A  F4          HLT                    
003D043B  EB 01        JMP SHORT 003D043E   
003D043D  DF70 EF      FBSTP TBYTE PTR DS:\[EAX-11\]   
003D0440  DF51 EB      FIST WORD PTR DS:\[ECX-15\]   
003D0443  F4          HLT                    
003D0444  EB EB        JMP SHORT 003D0431   
\-----------------------------------------------------   
按 SHIFT+F9 到这里   
003D4C83  D975 F8      FSTENV (28-BYTE) PTR SS:\[EBP-8\] ---第二处异常   
003D4C86  2BC0        SUB EAX,EAX   
003D4C88  0FB74C16 06  MOVZX ECX,WORD PTR DS:\[ESI+EDX+6\]   
003D4C8D  C1E0 05      SHL EAX,5   
003D4C90  C1E1 03      SHL ECX,3   
003D4C93  8908        MOV DWORD PTR DS:\[EAX\],ECX   
003D4C95  0FB74C16 14  MOVZX ECX,WORD PTR DS:\[ESI+EDX+14\]   
003D4C9A  03C1        ADD EAX,ECX   
003D4C9C  83E8 10      SUB EAX,10   
003D4C9F  03D0        ADD EDX,EAX   
003D4CA1  8B4C16 0C    MOV ECX,DWORD PTR DS:\[ESI+EDX+C\]   
\---------------------------------------------------------   
看一下 ESP+4,并设断点，按 SHIFT+F9   
003D4D38  64:67:A1 0000    MOV EAX,DWORD PTR FS:\[0\] ---在这里设断   
003D4D3D  8B20              MOV ESP,DWORD PTR DS:\[EAX\]   
003D4D3F  64:67:8F06 0000  POP DWORD PTR FS:\[0\]   
003D4D45  E8 00000000      CALL 003D4D4A   
003D4D4A  5D                POP EBP   
003D4D4B  81ED 86AA4000    SUB EBP,40AA86   
003D4D51  80BD 72CF4000 FF  CMP BYTE PTR SS:\[EBP+40CF72\],FF   
003D4D58  74 11            JE SHORT 003D4D6B   
点右键里的 seach for ..command   
输入 jmp edx ,    Find   
到这里   
003D4D60  890C24            MOV DWORD PTR SS:\[ESP\],ECX   
003D4D63  8B95 E1A44200    MOV EDX,DWORD PTR SS:\[EBP+42A4E1\]   
003D4D69  FFE2              JMP EDX    ---不是她   
003D4D6B  E8 00000000      CALL 003D4D70   
003D4D70  5D                POP EBP   
003D4D71  81ED ACAA4000    SUB EBP,40AAAC   
003D4D77  EB 47            JMP SHORT 003D4DC0   
003D4D79  DF69 4E          FILD QWORD PTR DS:\[ECX+4E\]   
点右键里的 seach for ..  Next   
到这里   
003D5CE9  0D FD0A4369      OR EAX,69430AFD   
003D5CEE  EB F4            JMP SHORT 003D5CE4   
003D5CF0  EB EB            JMP SHORT 003D5CDD   
003D5CF2  EB 8C            JMP SHORT 003D5C80   
003D5CF4  40                INC EAX   
003D5CF5  FFE2              JMP EDX  ---aha 设个断点，F9 下车吧。   
003D5CF7  EB 3A            JMP SHORT 003D5D33   
003D5CF9  DF69 4E          FILD QWORD PTR DS:\[ECX+4E\]   
003D5CFC  58                POP EAX   
003D5CFD  DF59 72          FISTP WORD PTR DS:\[ECX+72\]   
003D5D00  F4                HLT    
003D5D01  EB 01            JMP SHORT 003D5D04   
dump 出来，下面就是重建 Import table   
非常简单的^\_^  4dd466d0e1e0cc123afbe504da2462ca