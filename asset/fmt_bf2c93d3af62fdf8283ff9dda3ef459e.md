
```
004579F4 |> 837D CC 01    CMP DWORD PTR SS:[EBP-34],1                          ;  
004579F8 |. 75 1B         JNZ SHORT CDRWIN5.00457A15  
004579FA |. 68 78E04700   PUSH CDRWIN5.0047E078                                ;  'R'  
004579FF |. 8D4D AC       LEA ECX,DWORD PTR SS:[EBP-54]  
00457A02 |. E8 478E0000   CALL <JMP.&MFC42.#2764_?Find@CString@@QBEHPBD@Z>  
00457A07 |. 83F8 FF      CMP EAX,-1                                             ;  也必须有'R'  
00457A0A |. 74 09        JE SHORT CDRWIN5.00457A15  
00457A0C |. C745 CC 010000>MOV DWORD PTR SS:[EBP-34],1  
00457A13 |. EB 07        JMP SHORT CDRWIN5.00457A1C  
00457A15 |> C745 CC 000000>MOV DWORD PTR SS:[EBP-34],0  
00457A1C |> 68 7CE04700  PUSH CDRWIN5.0047E07C                                  ;  'P'  
00457A21 |. 8D4D AC      LEA ECX,DWORD PTR SS:[EBP-54]  
00457A24 |. E8 258E0000  CALL <JMP.&MFC42.#2764_?Find@CString@@QBEHPBD@Z>  
00457A29 |. 83F8 FF     CMP EAX,-1                                             ;  却不能有 'P'  
00457A2C |. 74 52       JE SHORT CDRWIN5.00457A80                              ；否则返回 0D 失败  
00457A2E |. C785 58FFFFFF >MOV DWORD PTR SS:[EBP-A8],0D  
.............................  
00457A7B |. E9 670B0000 JMP CDRWIN5.004585E7  
00457A80 |> 68 80E04700  PUSH CDRWIN5.0047E080                                  ;  'O'  
00457A85 |. 8D4D 08      LEA ECX,DWORD PTR SS:[EBP+8]                           ;  试炼码  
00457A88 |. E8 C18D0000  CALL <JMP.&MFC42.#2764_?Find@CString@@QBEHPBD@Z>  
00457A8D |. 83F8 FF     CMP EAX,-1                                             ；整个注册码不能有'O'  
00457A90 |. 74 52       JE SHORT CDRWIN5.00457AE4                              ；否则返回 0E 失败  
00457A92 |. C785 54FFFFFF >MOV DWORD PTR SS:[EBP-AC],0E  
................................  
00457ADF |. E9 030B0000  JMP CDRWIN5.004585E7  
00457AE4 |> 68 84E04700  PUSH CDRWIN5.0047E084                                  ;  'I'  
00457AE9 |. 8D4D 08      LEA ECX,DWORD PTR SS:[EBP+8]                           ;  试炼码  
00457AEC |. E8 5D8D0000  CALL <JMP.&MFC42.#2764_?Find@CString@@QBEHPBD@Z>       ;  
00457AF1 |. 83F8 FF      CMP EAX,-1  
00457AF4 |. 74 52        JE SHORT CDRWIN5.00457B48                              ;  也不能有 'I'  
00457AF6 |. C785 50FFFFFF >MOV DWORD PTR SS:[EBP-B0],0F                         ；否则返回 0F 失败  
..............................  
00457B43 |. E9 9F0A0000  JMP CDRWIN5.004585E7  
00457B48 |> C745 B0 000000>MOV DWORD PTR SS:[EBP-50],0  
00457B4F |. C745 A8 060000>MOV DWORD PTR SS:[EBP-58],6                          ;  字符索引，从试炼码的第二部分开始。  
00457B56 |. EB 09        JMP SHORT CDRWIN5.00457B61                              ;   
00457B58 |> 8B45 A8      /MOV EAX,DWORD PTR SS:[EBP-58]  
00457B5B |. 83C0 01      |ADD EAX,1  
00457B5E |. 8945 A8      |MOV DWORD PTR SS:[EBP-58],EAX  
00457B61 |> 837D A8 0B   CMP DWORD PTR SS:[EBP-58],0B                           ;  终止位置  
00457B65 |. 7D 33        |JGE SHORT CDRWIN5.00457B9A  
00457B67 |. 8B4D A8      |MOV ECX,DWORD PTR SS:[EBP-58]  
00457B6A |. 51           |PUSH ECX                                               ; 6  
00457B6B |. 8D4D 08      |LEA ECX,DWORD PTR SS:[EBP+8]                          ; 试炼码  
00457B6E |. E8 FD83FCFF  |CALL CDRWIN5.0041FF70                                 ; 返回第六个字符  
00457B73 |. 0FBED0       |MOVSX EDX,AL  
00457B76 |. 8B45 B0      |MOV EAX,DWORD PTR SS:[EBP-50]                         ;  [ebp-50] 存放加和结果  
00457B79 |. 03C2         |ADD EAX,EDX  
00457B7B |. 8945 B0      |MOV DWORD PTR SS:[EBP-50],EAX  
00457B7E |. 8B4D A8      |MOV ECX,DWORD PTR SS:[EBP-58]  
00457B81 |. 83C1 06      |ADD ECX,6                                              ;  字符索引+6  
00457B84 |. 51           |PUSH ECX                                               ;  即取试炼码第三部分的对应字符  
00457B85 |. 8D4D 08      |LEA ECX,DWORD PTR SS:[EBP+8]                          ;   
00457B88 |. E8 E383FCFF  |CALL CDRWIN5.0041FF70                                 ;   
00457B8D |. 0FBED0       |MOVSX EDX,AL  
00457B90 |. 8B45 B0      |MOV EAX,DWORD PTR SS:[EBP-50]  
00457B93 |. 03C2         |ADD EAX,EDX  
00457B95 |. 8945 B0      |MOV DWORD PTR SS:[EBP-50],EAX                         ；加上  
00457B98 |.^EB BE        \\JMP SHORT CDRWIN5.00457B58                           ;  以上循环计算第二、第三部分共十个字符的 ascii 码的加和。  
00457B9A |> 8B4D B0      MOV ECX,DWORD PTR SS:[EBP-50]                          ;   
00457B9D |. 81E1 01000080 AND ECX,80000001                                      ；检查是否为偶数  
00457BA3 |. 79 05        JNS SHORT CDRWIN5.00457BAA                             ; 没有符号是肯定的。   
00457BA5 |. 49           DEC ECX                                                ;   
00457BA6 |. 83C9 FE      OR ECX,FFFFFFFE  
00457BA9 |. 41           INC ECX  
00457BAA |> 85C9         TEST ECX,ECX  
00457BAC |. 74 52       JE SHORT CDRWIN5.00457C00                              ；必须是偶数  
00457BAE |. C785 4CFFFFFF >MOV DWORD PTR SS:[EBP-B4],3                          ；否则返回 3 失败  
.............................  
00457BFB |. E9 E7090000 JMP CDRWIN5.004585E7  
00457C00 |> 8D4D D4      LEA ECX,DWORD PTR SS:[EBP-2C]                          ;   
00457C03 |. E8 9E880000  CALL <JMP.&MFC42.#540_??0CString@@QAE@XZ>  
00457C08 |. C645 FC 09   MOV BYTE PTR SS:[EBP-4],9  
00457C0C |. 6A 10        PUSH 10                                                ; 取试炼码第 0x10 个字符  
00457C0E |. 8D4D 08      LEA ECX,DWORD PTR SS:[EBP+8]                           ;   
00457C11 |. E8 5A83FCFF  CALL CDRWIN5.0041FF70                                 ;   
00457C16 |. 50           PUSH EAX  
00457C17 |. 8D4D D4      LEA ECX,DWORD PTR SS:[EBP-2C]  
00457C1A |. E8 A9930000  CALL <JMP.&MFC42.#859_??4CString@@QAEABV0@D@Z>        ;  存入数据段 [3A8360]处  
00457C1F |. 8D4D D4      LEA ECX,DWORD PTR SS:[EBP-2C]  
00457C22 |. E8 49B5FAFF  CALL CDRWIN5.00403170  
00457C27 |. 50           PUSH EAX                                              ;   
00457C28 |. FF15 948A4600 CALL DWORD PTR DS:[<&MSVCRT.atoi>]                  ; 转换成对应的值  
00457C2E |. 83C4 04      ADD ESP,4  
00457C31 |. 8945 F0      MOV DWORD PTR SS:[EBP-10],EAX                         ;  必须是一个阿拉伯数字  
00457C34 |. 837D F0 00   CMP DWORD PTR SS:[EBP-10],0                           ；返回零通常表示不是数字  
00457C38 |. 75 70        JNZ SHORT CDRWIN5.00457CAA  
00457C3A |. 6A 10        PUSH 10                                              ;   
00457C3C |. 8D4D 08      LEA ECX,DWORD PTR SS:[EBP+8]                          ; 但必须检查他是不是就是'0'  
00457C3F |. E8 2C83FCFF  CALL CDRWIN5.0041FF70                                 ;   
00457C44 |. 0FBEC0       MOVSX EAX,AL  
00457C47 |. 83F8 30      CMP EAX,30                                            ;   
00457C4A |. 74 5E       JE SHORT CDRWIN5.00457CAA  
00457C4C |. C785 48FFFFFF >MOV DWORD PTR SS:[EBP-B8],4                         ;若不是阿拉伯数字，则返回 4 失败  
.............................  
00457CA5 |. E9 3D090000  JMP CDRWIN5.004585E7  
00457CAA |> 8B55 F0      MOV EDX,DWORD PTR SS:[EBP-10]                        ;  code[0x10]对应的值加上 5  
00457CAD |. 83C2 05      ADD EDX,5  
00457CB0 |. 8955 F0      MOV DWORD PTR SS:[EBP-10],EDX  
00457CB3 |. 837D F0 09   CMP DWORD PTR SS:[EBP-10],9                          ;  超过 9?  
00457CB7 |. 0F8E 84000000 JLE CDRWIN5.00457D41  
00457CBD |. 8B45 F0      MOV EAX,DWORD PTR SS:[EBP-10]  
00457CC0 |. 83E8 0A      SUB EAX,0A                                            ;  若超过 9 则对 10 取余  
00457CC3 |. 8945 F0      MOV DWORD PTR SS:[EBP-10],EAX  
00457CC6 |. 8B4D F0      MOV ECX,DWORD PTR SS:[EBP-10]                        ；于数加上 0x41 变成大写字母  
00457CC9 |. 83C1 41      ADD ECX,41                                            ;   
```
