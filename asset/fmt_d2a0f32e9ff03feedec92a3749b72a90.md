
```
00461DB9   |.  C683 79420000 00         MOV BYTE PTR DS:\[EBX+4279\],0  

00461DC0   |.  80BB 7A420000 00         CMP BYTE PTR DS:\[EBX+427A\],0  

00461DC7   |.  74 13                    JE SHORT QUOD.00461DDC  

00461DC9   |.  BE 4C1E4600              MOV ESI,QUOD.00461E4C  

00461DCE   |.  8DBB 84420000            LEA EDI,DWORD PTR DS:\[EBX+4284\]  

00461DD4   |.  A5                       MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ES>  

00461DD5   |.  A5                       MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ES>  

00461DD6   |.  A5                       MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ES>  

00461DD7   |.  66:A5                    MOVS WORD PTR ES:\[EDI\],WORD PTR DS:\[ESI\]  

00461DD9   |.  A4                       MOVS BYTE PTR ES:\[EDI\],BYTE PTR DS:\[ESI\]  

00461DDA   |.  EB 10                    JMP SHORT QUOD.00461DEC  

00461DDC   |>  BE 5C1E4600              MOV ESI,QUOD.00461E5C              ;  \n(双人游戏)  

00461DE1   |.  8DBB 84420000            LEA EDI,DWORD PTR DS:\[EBX+4284\]  

00461DE7   |.  A5                       MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ES>  

00461DE8   |.  A5                       MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ES>  

00461DE9   |.  66:A5                    MOVS WORD PTR ES:\[EDI\],WORD PTR DS:\[ESI\]  

00461DEB   |.  A4                       MOVS BYTE PTR ES:\[EDI\],BYTE PTR DS:\[ESI\]  

00461DEC   |>  BA 681E4600              MOV EDX,QUOD.00461E68  

00461DF1   |.  8D45 DC                  LEA EAX,DWORD PTR SS:\[EBP-24\]  

00461DF4   |.  E8 A710FAFF              CALL QUOD.00402EA0  

00461DF9   |.  8D93 84420000            LEA EDX,DWORD PTR DS:\[EBX+4284\]  

00461DFF   |.  8D45 DC                  LEA EAX,DWORD PTR SS:\[EBP-24\]  

00461E02   |.  B1 1F                    MOV CL,1F  

00461E04   |.  E8 6710FAFF              CALL QUOD.00402E70  

00461E09   |.  8D55 DC                  LEA EDX,DWORD PTR SS:\[EBP-24\]  

00461E0C   |.  8D45 FC                  LEA EAX,DWORD PTR SS:\[EBP-4\]  

00461E0F   |.  E8 8828FAFF              CALL QUOD.0040469C  

00461E14   |.  8B55 FC                  MOV EDX,DWORD PTR SS:\[EBP-4\]  

00461E17   |.  8BC3                     MOV EAX,EBX  

00461E19   |.  E8 C673FDFF              CALL QUOD.004391E4  

00461E1E   |.  8BC3                     MOV EAX,EBX  

00461E20   |.  E8 23FDFFFF              CALL QUOD.00461B48  

00461E25   |.  33C0                     XOR EAX,EAX  

00461E27   |.  5A                       POP EDX  

00461E28   |.  59                       POP ECX  

00461E29   |.  59                       POP ECX  

00461E2A   |.  64:8910                  MOV DWORD PTR FS:\[EAX\],EDX  

00461E2D   |.  68 421E4600              PUSH QUOD.00461E42  

00461E32   |>  8D45 FC                  LEA EAX,DWORD PTR SS:\[EBP-4\]  

00461E35   |.  E8 0626FAFF              CALL QUOD.00404440  

00461E3A   \\.  C3                      RETN  
```

棋盘是如下形状的，`11X11` 的盘，最角落的四个是没有的，所以用上面两个 `cmp 0A` 控制掉了  

```
X000000000X
00000000000
00000000000
00000000000
………
X000000000X
--------------------------------
```

代码窗口往上卷，哈哈，看到有趣的东西  

```
00461B48  /$  55                       PUSH EBP  

00461B49  |.  8BEC                     MOV EBP,ESP  

00461B4B  |.  81C4 74FFFFFF            ADD ESP,-8C  

00461B51  |.  53                       PUSH EBX  

00461B52  |.  33D2                     XOR EDX,EDX  

00461B54  |.  8995 74FFFFFF            MOV DWORD PTR SS:\[EBP-8C\],EDX  

00461B5A  |.  8955 AC                  MOV DWORD PTR SS:\[EBP-54\],EDX  

00461B5D  |.  8955 FC                  MOV DWORD PTR SS:\[EBP-4\],EDX  

00461B60  |.  8BD8                     MOV EBX,EAX  

00461B62  |.  33C0                     XOR EAX,EAX  

00461B64  |.  55                       PUSH EBP  

00461B65  |.  68 A51C4600              PUSH QUOD.00461CA5  

00461B6A  |.  64:FF30                  PUSH DWORD PTR FS:\[EAX\]  

00461B6D  |.  64:8920                  MOV DWORD PTR FS:\[EAX\],ESP  

00461B70  |.  80BB 78420000 00         CMP BYTE PTR DS:\[EBX+4278\],0             ;  判断是谁下的，为 0 不跳，显示红方下，为 1 跳，显示蓝方下（或者计算机正在思考），试着把它改了  

00461B77      75 51                    JNZ SHORT QUOD.00461BCA  

00461B79  |.  BA B41C4600              MOV EDX,QUOD.00461CB4  

00461B7E  |.  8D45 DC                  LEA EAX,DWORD PTR SS:\[EBP-24\]  

00461B81  |.  E8 1A13FAFF              CALL QUOD.00402EA0  

00461B86  |.  8D93 84420000            LEA EDX,DWORD PTR DS:\[EBX+4284\]  

00461B8C  |.  8D45 DC                  LEA EAX,DWORD PTR SS:\[EBP-24\]  

00461B8F  |.  B1 1F                    MOV CL,1F  

00461B91  |.  E8 DA12FAFF              CALL QUOD.00402E70  

00461B96  |.  8D55 DC                  LEA EDX,DWORD PTR SS:\[EBP-24\]  

00461B99  |.  8D45 B0                  LEA EAX,DWORD PTR SS:\[EBP-50\]  

00461B9C  |.  E8 FF12FAFF              CALL QUOD.00402EA0  

00461BA1  |.  BA C01C4600              MOV EDX,QUOD.00461CC0                  ;  \t 红方下！  

00461BA6  |.  8D45 B0                  LEA EAX,DWORD PTR SS:\[EBP-50\]  

00461BA9  |.  B1 28                    MOV CL,28  

00461BAB  |.  E8 C012FAFF              CALL QUOD.00402E70  

00461BB0  |.  8D55 B0                  LEA EDX,DWORD PTR SS:\[EBP-50\]  

00461BB3  |.  8D45 FC                  LEA EAX,DWORD PTR SS:\[EBP-4\]  

00461BB6  |.  E8 E12AFAFF              CALL QUOD.0040469C  

00461BBB  |.  8B55 FC                  MOV EDX,DWORD PTR SS:\[EBP-4\]  

00461BBE  |.  8BC3                     MOV EAX,EBX  

00461BC0  |.  E8 1F76FDFF              CALL QUOD.004391E4  

00461BC5  |.  E9 B2000000              JMP QUOD.00461C7C  

00461BCA  |>  80BB 7A420000 00         CMP BYTE PTR DS:\[EBX+427A\],0             ;  是不是与计算机下，是的话就往下走喽，不是的话跳走  

00461BD1  |.  74 57                    JE SHORT QUOD.00461C2A  

00461BD3  |.  BA B41C4600              MOV EDX,QUOD.00461CB4                  ;  00461CB4 存放的是标题前辍“监狱(Quod)”  

00461BD8  |.  8D45 DC                  LEA EAX,DWORD PTR SS:\[EBP-24\]  

00461BDB  |.  E8 C012FAFF              CALL QUOD.00402EA0  

00461BE0  |.  8D93 84420000            LEA EDX,DWORD PTR DS:\[EBX+4284\]  

00461BE6  |.  8D45 DC                  LEA EAX,DWORD PTR SS:\[EBP-24\]  

00461BE9  |.  B1 1F                    MOV CL,1F  

00461BEB  |.  E8 8012FAFF              CALL QUOD.00402E70  

00461BF0  |.  8D55 DC                  LEA EDX,DWORD PTR SS:\[EBP-24\]  

00461BF3  |.  8D85 78FFFFFF            LEA EAX,DWORD PTR SS:\[EBP-88\]  

00461BF9  |.  E8 A212FAFF              CALL QUOD.00402EA0  

00461BFE  |.  BA CC1C4600              MOV EDX,QUOD.00461CCC                  ;  00461CCC 存放的是“计算机正在思考”  

00461C03  |.  8D85 78FFFFFF            LEA EAX,DWORD PTR SS:\[EBP-88\]  

00461C09  |.  B1 31                    MOV CL,31  

00461C0B  |.  E8 6012FAFF              CALL QUOD.00402E70  

00461C10  |.  8D95 78FFFFFF            LEA EDX,DWORD PTR SS:\[EBP-88\]  

00461C16  |.  8D45 AC                  LEA EAX,DWORD PTR SS:\[EBP-54\]  

00461C19  |.  E8 7E2AFAFF              CALL QUOD.0040469C  

00461C1E  |.  8B55 AC                  MOV EDX,DWORD PTR SS:\[EBP-54\]  

00461C21  |.  8BC3                     MOV EAX,EBX  

00461C23  |.  E8 BC75FDFF              CALL QUOD.004391E4  

00461C28  |.  EB 52                    JMP SHORT QUOD.00461C7C  

00461C2A  |>  BA B41C4600              MOV EDX,QUOD.00461CB4                  ;  00461CB4 存放的是标题前辍“监狱(Quod)”  

00461C2F  |.  8D45 DC                  LEA EAX,DWORD PTR SS:\[EBP-24\]  

00461C32  |.  E8 6912FAFF              CALL QUOD.00402EA0  

00461C37  |.  8D93 84420000            LEA EDX,DWORD PTR DS:\[EBX+4284\]  

00461C3D  |.  8D45 DC                  LEA EAX,DWORD PTR SS:\[EBP-24\]  

00461C40  |.  B1 1F                    MOV CL,1F  

00461C42  |.  E8 2912FAFF              CALL QUOD.00402E70  

00461C47  |.  8D55 DC                  LEA EDX,DWORD PTR SS:\[EBP-24\]  

00461C4A  |.  8D45 B0                  LEA EAX,DWORD PTR SS:\[EBP-50\]  

00461C4D  |.  E8 4E12FAFF              CALL QUOD.00402EA0  

00461C52  |.  BA E01C4600              MOV EDX,QUOD.00461CE0                  ;  \t 蓝方下！  
```
