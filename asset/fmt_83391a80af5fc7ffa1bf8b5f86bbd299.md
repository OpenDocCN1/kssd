
```
004137B0   > \55                 PUSH EBP                              ;  一大堆垃圾代码影响大家的眼球
004137B1   . 8BEC                MOV EBP,ESP
004137B3   . 83EC 0C             SUB ESP,0C
004137B6   . 68 E6194000         PUSH <JMP.&MSVBVM60.\_\_vbaExceptHandler>  ;  SE 处理程序安装
004137BB   . 64:A1 0000000>MOV EAX,DWORD PTR FS:\[0\]
004137C1   . 50                  PUSH EAX
004137C2   . 64:8925 00000>MOV DWORD PTR FS:\[0\],ESP
004137C9   . 81EC B0000000       SUB ESP,0B0
004137CF   . 53                  PUSH EBX
004137D0   . 56                  PUSH ESI
004137D1   . 57                  PUSH EDI
004137D2   . 8965 F4             MOV DWORD PTR SS:\[EBP-C\],ESP
004137D5   . C745 F8 38124>MOV DWORD PTR SS:\[EBP-8\],ExcelWor.004012>
004137DC   . 8B75 08             MOV ESI,DWORD PTR SS:\[EBP+8\]
004137DF   . 8BC6                MOV EAX,ESI
004137E1   . 83E0 01             AND EAX,1
004137E4   . 8945 FC             MOV DWORD PTR SS:\[EBP-4\],EAX
004137E7   . 83E6 FE             AND ESI,FFFFFFFE
004137EA   . 56                  PUSH ESI
004137EB   . 8975 08             MOV DWORD PTR SS:\[EBP+8\],ESI
004137EE   . 8B0E                MOV ECX,DWORD PTR DS:\[ESI\]
004137F0   . FF51 04             CALL DWORD PTR DS:\[ECX+4\]
004137F3   . 8B16                MOV EDX,DWORD PTR DS:\[ESI\]
004137F5   . 33DB                XOR EBX,EBX
004137F7   . 56                  PUSH ESI
004137F8   . 895D E8             MOV DWORD PTR SS:\[EBP-18\],EBX
004137FB   . 895D E4             MOV DWORD PTR SS:\[EBP-1C\],EBX
004137FE   . 895D E0             MOV DWORD PTR SS:\[EBP-20\],EBX
00413801   . 895D DC             MOV DWORD PTR SS:\[EBP-24\],EBX
00413804   . 895D CC             MOV DWORD PTR SS:\[EBP-34\],EBX
00413807   . 895D BC             MOV DWORD PTR SS:\[EBP-44\],EBX
0041380A   . 895D AC             MOV DWORD PTR SS:\[EBP-54\],EBX
0041380D   . 895D 9C             MOV DWORD PTR SS:\[EBP-64\],EBX
00413810   . 895D 8C             MOV DWORD PTR SS:\[EBP-74\],EBX
00413813   . 899D 7CFFFFFF       MOV DWORD PTR SS:\[EBP-84\],EBX
00413819   . 899D 58FFFFFF       MOV DWORD PTR SS:\[EBP-A8\],EBX
0041381F   . FF92 0C030000       CALL DWORD PTR DS:\[EDX+30C\]
00413825   . 50                  PUSH EAX
00413826   . 8D45 DC             LEA EAX,DWORD PTR SS:\[EBP-24\]
00413829   . 50                  PUSH EAX
0041382A   . FF15 6C104000       CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaObjSe>;  MSVBVM60.\_\_vbaObjSet
00413830   . 8BF8                MOV EDI,EAX
00413832   . 8D55 E8             LEA EDX,DWORD PTR SS:\[EBP-18\]
00413835   . 52                  PUSH EDX
00413836   . 57                  PUSH EDI
00413837   . 8B0F                MOV ECX,DWORD PTR DS:\[EDI\]
00413839   . FF91 A0000000       CALL DWORD PTR DS:\[ECX+A0\]
0041383F   . 3BC3                CMP EAX,EBX
00413841   . DBE2                FCLEX
00413843   . 7D 12               JGE SHORT ExcelWor.00413857
00413845   . 68 A0000000         PUSH 0A0
0041384A   . 68 88564000         PUSH ExcelWor.00405688
0041384F   . 57                  PUSH EDI
00413850   . 50                  PUSH EAX
00413851   . FF15 44104000       CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaHresu>;  MSVBVM60.\_\_vbaHresultCheckObj
00413857   > 8B45 E8             MOV EAX,DWORD PTR SS:\[EBP-18\]                ;  用户名
0041385A   . 8D4D BC             LEA ECX,DWORD PTR SS:\[EBP-44\]
0041385D   . 8945 D4             MOV DWORD PTR SS:\[EBP-2C\],EAX
00413860   . 8D45 CC             LEA EAX,DWORD PTR SS:\[EBP-34\]
00413863   . 50                  PUSH EAX
00413864   . 51                  PUSH ECX
00413865   . 895D E8             MOV DWORD PTR SS:\[EBP-18\],EBX
00413868   . C745 CC 08000>MOV DWORD PTR SS:\[EBP-34\],8
0041386F   . FF15 84104000       CALL DWORD PTR DS:\[<&MSVBVM60.#520>\]       ;  MSVBVM60.rtcTrimVar
00413875   . 8B15 20604200       MOV EDX,DWORD PTR DS:\[426020\]
0041387B   . 8D85 58FFFFFF       LEA EAX,DWORD PTR SS:\[EBP-A8\]
00413881   . 8D4D BC             LEA ECX,DWORD PTR SS:\[EBP-44\]
00413884   . 50                  PUSH EAX
00413885   . 8B3A                MOV EDI,DWORD PTR DS:\[EDX\]
00413887   . 8D55 E4             LEA EDX,DWORD PTR SS:\[EBP-1C\]
0041388A   . 51                  PUSH ECX
0041388B   . 52                  PUSH EDX
0041388C   . FF15 20114000       CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaStrVa>;  MSVBVM60.\_\_vbaStrVarVal
00413892   . 50                  PUSH EAX
00413893   . A1 20604200         MOV EAX,DWORD PTR DS:\[426020\]
00413898   . 68 4C524000         PUSH ExcelWor.0040524C                       ;  UNICODE "LicenseName"
0041389D   . 50                  PUSH EAX
0041389E   . FF57 20             CALL DWORD PTR DS:\[EDI+20\]
004138A1   . 3BC3                CMP EAX,EBX
004138A3   . DBE2                FCLEX
004138A5   . 7D 15               JGE SHORT ExcelWor.004138BC
004138A7   . 8B0D 20604200       MOV ECX,DWORD PTR DS:\[426020\]
004138AD   . 6A 20               PUSH 20
004138AF   . 68 7C524000         PUSH ExcelWor.0040527C
004138B4   . 51                  PUSH ECX
004138B5   . 50                  PUSH EAX
004138B6   . FF15 44104000       CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaHresu>;  MSVBVM60.\_\_vbaHresultCheckObj
004138BC   > 8D4D E4             LEA ECX,DWORD PTR SS:\[EBP-1C\]
004138BF   . FF15 D8114000       CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaFreeS>;  MSVBVM60.\_\_vbaFreeStr
004138C5   . 8D4D DC             LEA ECX,DWORD PTR SS:\[EBP-24\]
004138C8   . FF15 DC114000       CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaFreeO>;  MSVBVM60.\_\_vbaFreeObj
004138CE   . 8B1D 24104000       MOV EBX,DWORD PTR DS:\[<&MSVBVM60.\_\_vbaFr>;  MSVBVM60.\_\_vbaFreeVarList
004138D4   . 8D55 BC             LEA EDX,DWORD PTR SS:\[EBP-44\]
004138D7   . 8D45 CC             LEA EAX,DWORD PTR SS:\[EBP-34\]
004138DA   . 52                  PUSH EDX
004138DB   . 50                  PUSH EAX
004138DC   . 6A 02               PUSH 2
004138DE   . FFD3                CALL EBX                                    ;  <&MSVBVM60.\_\_vbaFreeVarList>
004138E0   . 8B0E                MOV ECX,DWORD PTR DS:\[ESI\]
004138E2   . 83C4 0C             ADD ESP,0C
004138E5   . 56                  PUSH ESI
004138E6   . FF91 08030000       CALL DWORD PTR DS:\[ECX+308\]
004138EC   . 8D55 DC             LEA EDX,DWORD PTR SS:\[EBP-24\]
004138EF   . 50                  PUSH EAX
004138F0   . 52                  PUSH EDX
004138F1   . FF15 6C104000       CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaObjSe>;  MSVBVM60.\_\_vbaObjSet
004138F7   . 8BF8                MOV EDI,EAX
004138F9   . 8D4D E8             LEA ECX,DWORD PTR SS:\[EBP-18\]
004138FC   . 51                  PUSH ECX
004138FD   . 57                  PUSH EDI
004138FE   . 8B07                MOV EAX,DWORD PTR DS:\[EDI\]
00413900   . FF90 A0000000       CALL DWORD PTR DS:\[EAX+A0\]
00413906   . 85C0                TEST EAX,EAX
00413908   . DBE2                FCLEX
0041390A   . 7D 12               JGE SHORT ExcelWor.0041391E
0041390C   . 68 A0000000         PUSH 0A0
00413911   . 68 88564000         PUSH ExcelWor.00405688
00413916   . 57                  PUSH EDI
00413917   . 50                  PUSH EAX
00413918   . FF15 44104000       CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaHresu>;  MSVBVM60.\_\_vbaHresultCheckObj
0041391E   > 8B45 E8             MOV EAX,DWORD PTR SS:\[EBP-18\]                ;  假码
00413921   . 8D55 CC             LEA EDX,DWORD PTR SS:\[EBP-34\]
00413924   . 8945 D4             MOV DWORD PTR SS:\[EBP-2C\],EAX
00413927   . 8D45 BC             LEA EAX,DWORD PTR SS:\[EBP-44\]
0041392A   . 52                  PUSH EDX
0041392B   . 50                  PUSH EAX
0041392C   . C745 E8 00000>MOV DWORD PTR SS:\[EBP-18\],0
00413933   . C745 CC 08000>MOV DWORD PTR SS:\[EBP-34\],8
0041393A   . FF15 84104000       CALL DWORD PTR DS:\[<&MSVBVM60.#520>\]       ;  MSVBVM60.rtcTrimVar
00413940   . 8B0D 20604200       MOV ECX,DWORD PTR DS:\[426020\]
00413946   . 8D95 58FFFFFF       LEA EDX,DWORD PTR SS:\[EBP-A8\]
0041394C   . 8D45 BC             LEA EAX,DWORD PTR SS:\[EBP-44\]
0041394F   . 52                  PUSH EDX
00413950   . 8B39                MOV EDI,DWORD PTR DS:\[ECX\]
00413952   . 8D4D E4             LEA ECX,DWORD PTR SS:\[EBP-1C\]
00413955   . 50                  PUSH EAX
00413956   . 51                  PUSH ECX
00413957   . FF15 20114000       CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaStrVa>;  MSVBVM60.\_\_vbaStrVarVal
0041395D   . 8B15 20604200       MOV EDX,DWORD PTR DS:\[426020\]
00413963   . 50                  PUSH EAX
00413964   . 68 90524000         PUSH ExcelWor.00405290                       ;  UNICODE "LicenseCode"
00413969   . 52                  PUSH EDX
0041396A   . FF57 20             CALL DWORD PTR DS:\[EDI+20\]
0041396D   . 85C0                TEST EAX,EAX
0041396F   . DBE2                FCLEX
00413971   . 7D 15               JGE SHORT ExcelWor.00413988
00413973   . 8B0D 20604200       MOV ECX,DWORD PTR DS:\[426020\]
00413979   . 6A 20               PUSH 20
0041397B   . 68 7C524000         PUSH ExcelWor.0040527C
00413980   . 51                  PUSH ECX
00413981   . 50                  PUSH EAX
00413982   . FF15 44104000       CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaHresu>;  MSVBVM60.\_\_vbaHresultCheckObj
00413988   > 8D4D E4             LEA ECX,DWORD PTR SS:\[EBP-1C\]
0041398B   . FF15 D8114000       CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaFreeS>;  MSVBVM60.\_\_vbaFreeStr
00413991   . 8D4D DC             LEA ECX,DWORD PTR SS:\[EBP-24\]
00413994   . FF15 DC114000       CALL DWORD PTR DS:\[<&MSVBVM60.\_\_vbaFreeO>;  MSVBVM60.\_\_vbaFreeObj
0041399A   . 8D55 BC             LEA EDX,DWORD PTR SS:\[EBP-44\]
0041399D   . 8D45 CC             LEA EAX,DWORD PTR SS:\[EBP-34\]
004139A0   . 52                  PUSH EDX
004139A1   . 50                  PUSH EAX
004139A2   . 6A 02               PUSH 2
004139A4   . FFD3                CALL EBX
004139A6   . 83C4 0C             ADD ESP,0C
004139A9   . E8 52F5FFFF         CALL ExcelWor.00412F00                       ;  关键 CALL，F7 进
004139AE   . 66:3D FFFFF          CMP AX,0FFFF                                ;  比较
004139B2   . B9 04000280         MOV ECX,80020004
004139B7   . B8 0A000000         MOV EAX,0A
004139BC   . 894D A4             MOV DWORD PTR SS:\[EBP-5C\],ECX
004139BF   . 8945 9C             MOV DWORD PTR SS:\[EBP-64\],EAX
004139C2   . 894D B4             MOV DWORD PTR SS:\[EBP-4C\],ECX
004139C5   . 8945 AC             MOV DWORD PTR SS:\[EBP-54\],EAX
004139C8   . 0F85 25010000       JNZ ExcelWor.00413AF3                         ;  关键跳，跳向失败
004139CE   . 894D C4             MOV DWORD PTR SS:\[EBP-3C\],ECX              ;  以下无关代码省略......
```
