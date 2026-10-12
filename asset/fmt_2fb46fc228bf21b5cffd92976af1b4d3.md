
## 汇编代码片段

```
00413524    A9 00000080           TEST EAX,80000000  
00413529    74 0A               JE SHORT 00413535  
0041352B    25 FFFFFF7F         AND EAX,7FFFFFFF  
00413530    2BFF                SUB EDI,EDI  
00413532    EB 09               JMP SHORT 0041353D  
00413534    90                  NOP  
00413535    40                  INC EAX  
00413536    0385 C34B4000       ADD EAX,DWORD PTR SS:[EBP+404BC3]  
0041353C    97                  XCHG EAX,EDI  
0041353D    68 AFFAD0F9         PUSH F9D0FAAF  
00413542    012C24              ADD DWORD PTR SS:[ESP],EBP  
00413545    810424 B4466F06     ADD DWORD PTR SS:[ESP],66F46B4  
0041354C    68 4D7B630F         PUSH 0F637B4D  
00413551    812C24 9643230F     SUB DWORD PTR SS:[ESP],0F234396  
00413558    012C24              ADD DWORD PTR SS:[ESP],EBP  
0041355B    C3                  RETN                      ; 这里返回 API 处理部分
```

跟进看看：

……

```
00412C70    8B00                MOV EAX,DWORD PTR DS:[EAX]  
00412C72    0385 AA374000       ADD EAX,DWORD PTR SS:[EBP+4037AA]          ; 获取到的 API 放到 eax 中  
00412C78    EB 10               JMP SHORT 00412C8A  
00412C7A    83C3 04             ADD EBX,4  
00412C7D    41                  INC ECX  
00412C7E    81F9 B5030000       CMP ECX,3B5  
00412C84  ^ 75 97               JNZ SHORT 00412C1D  
00412C86    33C0                XOR EAX,EAX  
00412C88    EB 3F               JMP SHORT 00412CC9  
00412C8A    8BBD 9E374000       MOV EDI,DWORD PTR SS:[EBP+40379E]  
00412C90    3BC7                CMP EAX,EDI                                 ; 判断是否要加密  
00412C92    76 35               JBE SHORT 00412CC9                          ; 如果小于或等于 7c80262c 则不加密直接填充  
00412C94    03BD A2374000       ADD EDI,DWORD PTR SS:[EBP+4037A2]  
00412C9A    3BF8                CMP EDI,EAX  
00412C9C    76 2B               JBE SHORT 00412CC9  
00412C9E    8DBD 052C4000       LEA EDI,DWORD PTR SS:[EBP+402C05]  
00412CA4    96                  XCHG EAX,ESI  
00412CA5    33C9                XOR ECX,ECX  
00412CA7    8A0431              MOV AL,BYTE PTR DS:[ECX+ESI]  
00412CAA    3C 2E               CMP AL,2E  
00412CAC    74 04               JE SHORT 00412CB2  
00412CAE    41                  INC ECX  
00412CAF    AA                  STOS BYTE PTR ES:[EDI]  
00412CB0  ^ EB F5               JMP SHORT 00412CA7  
00412CB2    41                  INC ECX  
00412CB3    03F1                ADD ESI,ECX  
00412CB5    56                  PUSH ESI  
00412CB6    2C 2E               SUB AL,2E  
00412CB8    AA                  STOS BYTE PTR ES:[EDI]  
00412CB9    2BF9                SUB EDI,ECX  
00412CBB    57                  PUSH EDI  
00412CBC    FF95 F54B4000       CALL DWORD PTR SS:[EBP+404BF5]  
00412CC2    50                  PUSH EAX  
00412CC3    FF95 FF4B4000       CALL DWORD PTR SS:[EBP+404BFF]  
00412CC9    EB 01               JMP SHORT 00412CCC  
00412CCB    90                  NOP  
00412CCC    894424 1C           MOV DWORD PTR SS:[ESP+1C],EAX             ; 填充 API  
00412CD0    61                  POPAD  
00412CD1    FF0424              INC DWORD PTR SS:[ESP]
```

……

```
0041355F   /0F84 36080000       JE 00413D9B                                   ; 如果获取 API 失败则 over
```

……

```
004135A2    0FBA67 FF 07    BT DWORD PTR DS:[EDI-1],7        ; 获取[EDI-1]的第 7 位位传送给 CF,如果 cf 为 1 刚加密 api
```

所以这里可以直接 patch 成 `clc`

```
004135A7    EB 01            JMP SHORT 004135AA  
004135A9    90               NOP  
004135AA    9C               PUSHFD  
004135AB    F71424           NOT DWORD PTR SS:[ESP]  
004135AE    832424 01        AND DWORD PTR SS:[ESP],1  
004135B2    50               PUSH EAX  
004135B3    52               PUSH EDX  
004135B4    B8 2E306BF9      MOV EAX,F96B302E  
004135B9    05 31D09406      ADD EAX,694D031  
004135BE    F76424 08        MUL DWORD PTR SS:[ESP+8]  
004135C2    8D8428 E9414000 LEA EAX,DWORD PTR DS:[EAX+EBP+4041E9]  
004135C9    894424 08        MOV DWORD PTR SS:[ESP+8],EAX  
004135CD    5A               POP EDX  
004135CE    58               POP EAX  
004135CF    90               NOP  
004135D0    90               NOP  
004135D1    90               NOP  
004135D2    90               NOP  
004135D3    90               NOP  
004135D4    90               NOP  
004135D5    90               NOP  
004135D6    90               NOP  
004135D7    90               NOP  
004135D8    90               NOP  
004135D9    90               NOP  
004135DA    90               NOP  
004135DB    8D6424 04        LEA ESP,DWORD PTR SS:[ESP+4]  
004135DF    FF6424 FC        JMP DWORD PTR SS:[ESP-4]      ; CF 为 1 则加密 API，加密就跳去 eip+5 处
```

……

```
00413614    E8 03000000      CALL 0041361C  
00413619    A0 9AFF5B81      MOV AL,BYTE PTR DS:[815BFF9A]  
0041361E    C3               RETN  
0041361F    1900             SBB DWORD PTR DS:[EAX],EAX  
00413621    0000             ADD BYTE PTR DS:[EAX],AL  
00413623    53               PUSH EBX  
00413624    8D9D C050288E    LEA EBX,DWORD PTR SS:[EBP+8E2850C0]  
0041362A    81EB BC1AE88D    SUB EBX,8DE81ABC  
00413630    FFE3             JMP EBX                        ; 这里跳去加密 api
```

……

当 `CF` 为 0 时跳到这里:

```
00413643    E8 C4F6FFFF      CALL 00412D0C                   ; 不用加密则处理 jmp 表
```

进来看看：

```
00412D18    57               PUSH EDI                         ; 这段代码和 1.0 没有什么变化  
00412D19    EB 01            JMP SHORT 00412D1C  
00412D1B    90               NOP  
00412D1C    51               PUSH ECX  
00412D1D    90               NOP  
00412D1E    90               NOP  
00412D1F    90               NOP  
00412D20    90               NOP  
00412D21    90               NOP  
00412D22    90               NOP  
00412D23    90               NOP  
00412D24    90               NOP  
00412D25    90               NOP  
00412D26    BF DA9A4000      MOV EDI,00409ADA  
00412D2B    EB 01            JMP SHORT 00412D2E  
00412D2D    90               NOP  
00412D2E    B9 8C010000      MOV ECX,18C  
00412D33    90               NOP  
00412D34    90               NOP  
00412D35    90               NOP  
00412D36    90               NOP  
00412D37    90               NOP  
00412D38    90               NOP  
00412D39    90               NOP  
00412D3A    90               NOP  
00412D3B    90               NOP  
00412D3C    90               NOP  
00412D3D    90               NOP  
00412D3E    90               NOP  
00412D3F    90               NOP  
00412D40    90               NOP  
00412D41    90               NOP  
00412D42    90               NOP  
00412D43    90               NOP  
00412D44    3917             CMP DWORD PTR DS:[EDI],EDX      ; 判断是否找到了该地址  
00412D46    90               NOP  
00412D47    90               NOP  
00412D48    90               NOP  
00412D49    90               NOP  
00412D4A    90               NOP  
00412D4B    90               NOP  
00412D4C    90               NOP  
00412D4D    90               NOP  
00412D4E    90               NOP  
00412D4F    90               NOP  
00412D50    90               NOP  
00412D51    90               NOP  
00412D52    0F84 90000000    JE 00412DE8                     ; 如果找到则跳  
00412D58    47               INC EDI  
00412D59    EB 01            JMP SHORT 00412D5C  
00412D5B    90               NOP  
00412D5C    49               DEC ECX  
00412D5D    9C               PUSHFD  
00412D5E    C12C24 06        SHR DWORD PTR SS:[ESP],6  
00412D62    F71424           NOT DWORD PTR SS:[ESP]  
00412D65    832424 01        AND DWORD PTR SS:[ESP],1  
00412D69    50               PUSH EAX  
00412D6A    52               PUSH EDX  
00412D6B    B8 6592DC52      MOV EAX,52DC9265  
00412D70    05 446D23AD      ADD EAX,AD236D44  
00412D75    F76424 08        MUL DWORD PTR SS:[ESP+8]  
00412D79    90               NOP
```
