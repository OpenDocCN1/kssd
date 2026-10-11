
# 汇编代码分析

## 代码片段

```
0041A4D2   .  FF15 98104000 CALL DWORD PTR DS:[<&msvbvm60.__vbaExitP>;  msvbvm60.__vbaExitProc  

0041A4D8   .  68 C2A64100   PUSH 1.0041A6C2  

0041A4DD   .  E9 C6010000   JMP 1.0041A6A8  

0041A4E2   >  C745 8C 0C9A4>MOV DWORD PTR SS:[EBP-74],1.00409A0C  

0041A4E9   .  C745 84 08000>MOV DWORD PTR SS:[EBP-7C],8  

0041A4F0   .  8D55 84       LEA EDX,DWORD PTR SS:[EBP-7C]  

0041A4F3   .  8D4D A4       LEA ECX,DWORD PTR SS:[EBP-5C]  

0041A4F6   .  FF15 20124000 CALL DWORD PTR DS:[<&msvbvm60.__vbaVarDu>;  msvbvm60.__vbaVarDup  

0041A4FC   .  6A 00         PUSH 0  

0041A4FE   .  6A FF         PUSH -1  

0041A500   .  8D55 A4       LEA EDX,DWORD PTR SS:[EBP-5C]  

0041A503   .  52            PUSH EDX  

0041A504   .  8B45 BC       MOV EAX,DWORD PTR SS:[EBP-44]  

0041A507   .  50            PUSH EAX  

0041A508   .  8D4D 94       LEA ECX,DWORD PTR SS:[EBP-6C]  

0041A50B   .  51            PUSH ECX  

0041A50C   .  FF15 5C114000 CALL DWORD PTR DS:[<&msvbvm60.rtcSplit>] ;  msvbvm60.rtcSplit  

0041A512   .  8D55 94       LEA EDX,DWORD PTR SS:[EBP-6C]            ;  这一部分是看有几个“-”  

0041A515   .  8D4D CC       LEA ECX,DWORD PTR SS:[EBP-34]  

0041A518   .  FF15 1C104000 CALL DWORD PTR DS:[<&msvbvm60.__vbaVarMo>;  msvbvm60.__vbaVarMove  

0041A51E   .  8D4D A4       LEA ECX,DWORD PTR SS:[EBP-5C]  

0041A521   .  FFD6          CALL ESI  

0041A523   .  8D55 CC       LEA EDX,DWORD PTR SS:[EBP-34]  

0041A526   .  52            PUSH EDX  

0041A527   .  8B1D D8104000 MOV EBX,DWORD PTR DS:[<&msvbvm60.__vbaRe>;  msvbvm60.__vbaRefVarAry  

0041A52D   .  FFD3          CALL EBX                            ;  <&msvbvm60.__vbaRefVarAry>  

0041A52F   .  8B00          MOV EAX,DWORD PTR DS:[EAX]  

0041A531   .  50            PUSH EAX  

0041A532   .  6A 01         PUSH 1  

0041A534   .  FF15 30114000 CALL DWORD PTR DS:[<&msvbvm60.__vbaLboun>;  msvbvm60.__vbaLbound  

0041A53A   .  8BF8          MOV EDI,EAX  

0041A53C   .  8D4D CC       LEA ECX,DWORD PTR SS:[EBP-34]  

0041A53F   .  51            PUSH ECX  

0041A540   .  FFD3          CALL EBX  

0041A542   .  8B10          MOV EDX,DWORD PTR DS:[EAX]  

0041A544   .  52            PUSH EDX  

0041A545   .  6A 01         PUSH 1  

0041A547   .  FF15 90114000 CALL DWORD PTR DS:[<&msvbvm60.__vbaUboun>;  msvbvm60.__vbaUbound  

0041A54D   .  2BC7          SUB EAX,EDI  

0041A54F   .  0F80 82010000 JO 1.0041A6D7  

0041A555   .  83C0 01       ADD EAX,1  

0041A558   .  0F80 79010000 JO 1.0041A6D7  

0041A55E   .  83F8 03       CMP EAX,3  

0041A561   .  74 17         JE SHORT 1.0041A57A                ;  注册码是否由三部分组成，不是就 over.  

0041A563   .  C745 C8 00000>MOV DWORD PTR SS:[EBP-38],0  

0041A56A   .  FF15 98104000 CALL DWORD PTR DS:[<&msvbvm60.__vbaExitP>;  msvbvm60.__vbaExitProc  

0041A570   .  68 C2A64100   PUSH 1.0041A6C2  

0041A575     E9 2E010000   JMP 1.0041A6A8  

0041A57A   >  8D45 CC       LEA EAX,DWORD PTR SS:[EBP-34]  

0041A57D   .  50            PUSH EAX  

0041A57E   .  FFD3          CALL EBX  

0041A580   .  8B08          MOV ECX,DWORD PTR DS:[EAX]  

0041A582   .  51            PUSH ECX  

0041A583   .  6A 01         PUSH 1  

0041A585   .  FF15 30114000 CALL DWORD PTR DS:[<&msvbvm60.__vbaLboun>;  msvbvm60.__vbaLbound  

0041A58B   .  8945 8C       MOV DWORD PTR SS:[EBP-74],EAX  

0041A58E   .  B9 03000000   MOV ECX,3  

0041A593   .  894D 84       MOV DWORD PTR SS:[EBP-7C],ECX  

0041A596   .  83EC 10       SUB ESP,10  

0041A599   .  8BD4          MOV EDX,ESP  

0041A59B   .  890A          MOV DWORD PTR DS:[EDX],ECX  

0041A59D   .  8B4D 88       MOV ECX,DWORD PTR SS:[EBP-78]  

0041A5A0   .  894A 04       MOV DWORD PTR DS:[EDX+4],ECX  

0041A5A3   .  8942 08       MOV DWORD PTR DS:[EDX+8],EAX  

0041A5A6   .  8B45 90       MOV EAX,DWORD PTR SS:[EBP-70]  

0041A5A9   .  8942 0C       MOV DWORD PTR DS:[EDX+C],EAX  

0041A5AC   .  6A 01         PUSH 1  

0041A5AE   .  8D4D CC       LEA ECX,DWORD PTR SS:[EBP-34]  

0041A5B1   .  51            PUSH ECX  

0041A5B2   .  8D55 A4       LEA EDX,DWORD PTR SS:[EBP-5C]  

0041A5B5   .  52            PUSH EDX  

0041A5B6   .  8B3D C0104000 MOV EDI,DWORD PTR DS:[<&msvbvm60.__vbaVa>;  msvbvm60.__vbaVarIndexLoad  

0041A5BC   .  FFD7          CALL EDI                            ;  <&msvbvm60.__vbaVarIndexLoad>  

0041A5BE   .  83C4 1C       ADD ESP,1C  

0041A5C1   .  50            PUSH EAX  

0041A5C2   .  FF15 7C124000 CALL DWORD PTR DS:[<&msvbvm60.__vbaI4Err>;  msvbvm60.__vbaI4ErrVar  

0041A5C8   .  8945 C4       MOV DWORD PTR SS:[EBP-3C],EAX        ;  得假码的第一部分十六进制值，我这是 4d2  

0041A5CB   .  8D4D A4       LEA ECX,DWORD PTR SS:[EBP-5C]  

0041A5CE   .  FFD6          CALL ESI  

0041A5D0   .  8D45 CC       LEA EAX,DWORD PTR SS:[EBP-34]  

0041A5D3   .  50            PUSH EAX  

0041A5D4   .  FFD3          CALL EBX  

0041A5D6   .  8B08          MOV ECX,DWORD PTR DS:[EAX]  

0041A5D8   .  51            PUSH ECX  

0041A5D9   .  6A 01         PUSH 1  

0041A5DB   .  FF15 90114000 CALL DWORD PTR DS:[<&msvbvm60.__vbaUboun>;  msvbvm60.__vbaUbound  

0041A5E1   .  8945 8C       MOV DWORD PTR SS:[EBP-74],EAX  

0041A5E4   .  B9 03000000   MOV ECX,3  

0041A5E9   .  894D 84       MOV DWORD PTR SS:[EBP-7C],ECX  

0041A5EC   .  83EC 10       SUB ESP,10  

0041A5EF   .  8BD4          MOV EDX,ESP  

0041A5F1   .  890A          MOV DWORD PTR DS:[EDX],ECX  

0041A5F3   .  8B4D 88       MOV ECX,DWORD PTR SS:[EBP-78]  

0041A5F6   .  894A 04       MOV DWORD PTR DS:[EDX+4],ECX  

0041A5F9   .  8942 08       MOV DWORD PTR DS:[EDX+8],EAX  

0041A5FC   .  8B45 90       MOV EAX,DWORD PTR SS:[EBP-70]  

0041A5FF   .  8942 0C       MOV DWORD PTR DS:[EDX+C],EAX  

0041A602   .  6A 01         PUSH 1  

0041A604   .  8D4D CC       LEA ECX,DWORD PTR SS:[EBP-34]  

0041A607   .  51            PUSH ECX  

0041A608   .  8D55 A4       LEA EDX,DWORD PTR SS:[EBP-5C]  

0041A60B   .  52            PUSH EDX  

0041A60C   .  FFD7          CALL EDI  

0041A60E   .  83C4 1C       ADD ESP,1C  

0041A611   .  50            PUSH EAX  

0041A612   .  FF15 7C124000 CALL DWORD PTR DS:[<&msvbvm60.__vbaI4Err>;  msvbvm60.__vbaI4ErrVar  

0041A618   .  8BF8          MOV EDI,EAX                      ;  得假码的第三部分十六进制值，我这是 457  

0041A61A   .  8D4D A4       LEA ECX,DWORD PTR SS:[EBP-5C]  

0041A61D   .  FFD6          CALL ESI  

0041A61F   .  8B45 BC       MOV EAX,DWORD PTR SS:[EBP-44]  

0041A622   .  50            PUSH EAX  

0041A623   .  57            PUSH EDI  

0041A624   .  8B4D C4       MOV ECX,DWORD PTR SS:[EBP-3C]  

0041A627   .  51            PUSH ECX  

0041A628   .  8B55 DC       MOV EDX,DWORD PTR SS:[EBP-24]  

0041A62B   .  52            PUSH EDX  

0041A62C   .  E8 BFFAFFFF   CALL 1.0041A0F0                   ;  关键 CALL,跟进  

0041A631 >  .  8BD0          MOV EDX,EAX                      ;  可在这里做内存注册机。  

0041A633   .  8D4D B4       LEA ECX,DWORD PTR SS:[EBP-4C]  

0041A636   .  FF15 50124000 CALL DWORD PTR DS:[<&msvbvm60.__vbaStrMo>;  msvbvm60.__vbaStrMove  

0041A63C   .  50            PUSH EAX  

0041A63D   .  FF15 0C114000 CALL DWORD PTR DS:[<&msvbvm60.__vbaStrCm>;  msvbvm60.__vbaStrCmp  

0041A643   .  8BF0          MOV ESI,EAX  

0041A645   .  F7DE          NEG ESI  

0041A647   .  1BF6          SBB ESI,ESI  

0041A649   .  46            INC ESI  

0041A64A   .  F7DE          NEG ESI  

0041A64C   .  8D4D B4       LEA ECX,DWORD PTR SS:[EBP-4C]  

0041A64F   .  FF15 80124000 CALL DWORD PTR DS:[<&msvbvm60.__vbaFreeS>;  msvbvm60.__vbaFreeStr  

0041A655   .  66:85F6       TEST SI,SI  

0041A658   .  74 19         JE SHORT 1.0041A673                ;  

0041A65A   .  83C8 FF       OR EAX,FFFFFFFF  

0041A65D   .  8945 C8       MOV DWORD PTR SS:[EBP-38],EAX  

0041A660   .  66:A3 44C0410>MOV WORD PTR DS:[41C044],AX  
```
