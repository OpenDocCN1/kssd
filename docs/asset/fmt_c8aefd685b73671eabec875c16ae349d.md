
```
00457F57  |. 8D4D D0         LEA ECX,DWORD PTR SS:[EBP-30]  

00457F5A  |. E8 69900000     CALL <JMP.&MFC42.#859_??4CString@@QAEABV0@D@Z>  

00457F5F  |. 8D4D D0         LEA ECX,DWORD PTR SS:[EBP-30]                   ;   

00457F62  |. E8 09B2FAFF     CALL CDRWIN5.00403170  

00457F67  |. 50              PUSH EAX                                       ;   

00457F68  |. FF15 948A4600   CALL DWORD PTR DS:[<&MSVCRT.atoi>]            ; 转成数值  ->必须是阿拉伯数字  

00457F6E  |. 83C4 04         ADD ESP,4                                      ;   

00457F71  |. 8945 94         MOV DWORD PTR SS:[EBP-6C],EAX  

00457F74  |. 837D 94 00      CMP DWORD PTR SS:[EBP-6C],0  

00457F78  |. 0F85 C0000000   JNZ CDRWIN5.0045803E  

00457F7E  |. 6A 03           PUSH 3                                         ;   

00457F80  |. 8D4D 08         LEA ECX,DWORD PTR SS:[EBP+8]                  ;   

00457F83  |. E8 E87FFCFF     CALL CDRWIN5.0041FF70                          ;   

00457F88  |. 0FBEC8          MOVSX ECX,AL  

00457F8B  |. 83F9 30         CMP ECX,30  

00457F8E  |. 0F84 AA000000   JE CDRWIN5.0045803E  

.........................  

0045803E  |> 837D 94 00      CMP DWORD PTR SS:[EBP-6C],0  

00458042  |. 75 1B           JNZ SHORT CDRWIN5.0045805F  

.........................  

0045805F  |> 837D 94 00      CMP DWORD PTR SS:[EBP-6C],0  

00458063  |. 75 13          JNZ SHORT CDRWIN5.00458078  

.........................  

00458078  |> 8D4D C0         LEA ECX,DWORD PTR SS:[EBP-40]  

0045807B  |. E8 26840000     CALL <JMP.&MFC42.#540_??0CString@@QAE@XZ>  

00458080  |. C645 FC 0E      MOV BYTE PTR SS:[EBP-4],0E  

00458084  |. 6A 01           PUSH 1                                         ;   

00458086  |. 8D4D 08         LEA ECX,DWORD PTR SS:[EBP+8]                  ;   

00458089  |. E8 E27EFCFF     CALL CDRWIN5.0041FF70                          ; 取 code[1]  

0045808E  |. 50              PUSH EAX  

0045808F  |. 8D4D C0         LEA ECX,DWORD PTR SS:[EBP-40]  

00458092  |. E8 318F0000     CALL <JMP.&MFC42.#859_??4CString@@QAEABV0@D@Z> ;   

00458097  |. 8D4D C0         LEA ECX,DWORD PTR SS:[EBP-40]  

0045809A  |. E8 D1B0FAFF     CALL CDRWIN5.00403170  

0045809F  |. 50              PUSH EAX                                       ;   

004580A0  |. FF15 948A4600   CALL DWORD PTR DS:[<&MSVCRT.atoi>]            ; 也必须是阿拉伯数字  

004580A6  |. 83C4 04         ADD ESP,4                                      ;   

004580A9  |. 8945 C4         MOV DWORD PTR SS:[EBP-3C],EAX  

004580AC  |. 837D C4 00      CMP DWORD PTR SS:[EBP-3C],0  

004580B0  |. 0F85 CC000000   JNZ CDRWIN5.00458182  

004580B6  |. 6A 01           PUSH 1                                         ; /Arg1 = 00000001  

004580B8  |. 8D4D 08         LEA ECX,DWORD PTR SS:[EBP+8]                  ; |  

004580BB  |. E8 B07EFCFF     CALL CDRWIN5.0041FF70                          ; \CDRWIN5.0041FF70  

004580C0  |. 0FBEC8          MOVSX ECX,AL  

004580C3  |. 83F9 30         CMP ECX,30  

004580C6  |. 0F84 B6000000   JE CDRWIN5.00458182  

.........................  

00458182  |> 837D C4 00      CMP DWORD PTR SS:[EBP-3C],0  

00458186  |. 75 1B           JNZ SHORT CDRWIN5.004581A3  

........................  

004581A3  |> 837D 94 00      CMP DWORD PTR SS:[EBP-6C],0  

004581A7  |. 75 13          JNZ SHORT CDRWIN5.004581BC  

004581A9  |. 6A 01           PUSH 1                                         ; /Arg1 = 00000001  

004581AB  |. 8D4D 08         LEA ECX,DWORD PTR SS:[EBP+8]                  ; |  

004581AE  |. E8 BD7DFCFF     CALL CDRWIN5.0041FF70                          ; \CDRWIN5.0041FF70  

004581B3  |. 0FBEC0          MOVSX EAX,AL  

004581B6  |. 83E8 37         SUB EAX,37  

004581B9  |. 8945 C4         MOV DWORD PTR SS:[EBP-3C],EAX                 ;   

004581BC  |> 8B4D C4         MOV ECX,DWORD PTR SS:[EBP-3C]                 ；code [1]  

004581BF  |. C1E1 04         SHL ECX,4                                      ; 乘以 16  

004581C2  |. 894D C4         MOV DWORD PTR SS:[EBP-3C],ECX  

004581C5  |. 8B55 C4         MOV EDX,DWORD PTR SS:[EBP-3C]                 ；code[3]  

004581C8  |. 0355 94         ADD EDX,DWORD PTR SS:[EBP-6C]                 ;加和  

004581CB  |. 8955 C4         MOV DWORD PTR SS:[EBP-3C],EDX  

004581CE  |. 8B45 C4         MOV EAX,DWORD PTR SS:[EBP-3C]  

004581D1  |. 25 07000080     AND EAX,80000007                              ;  测试结果的后 3 位是否都为 0  

004581D6  |. 79 05           JNS SHORT CDRWIN5.004581DD                   ；由于 code[1]对应的数字已经乘以 16，它不影响这个测试，  

004581D8  |. 48              DEC EAX                                      ；主要是看 code[3]，因此实际上 code[3]只能为 0 或 8   

004581D9  |. 83C8 F8         OR EAX,FFFFFFF8  

004581DC  |. 40              INC EAX  

004581DD  |> 85C0            TEST EAX,EAX                                  ;   

004581DF  |. 0F84 8E000000   JE CDRWIN5.00458273                          ；若后 3 位不全为 0，  

004581E5  |. C785 2CFFFFFF   >MOV DWORD PTR SS:[EBP-D4],0A                ；返回 0A 失败  

............................  

00458268  |. 8B85 2CFFFFFF   MOV EAX,DWORD PTR SS:[EBP-D4]  

0045826E  |. E9 74030000     JMP CDRWIN5.004585E7  

00458273  |> C745 D8 000000>MOV DWORD PTR SS:[EBP-28],0  

0045827A  |. 6A 12           PUSH 12                                       ; 取 code[0x12]  

0045827C  |. 8D4D 08         LEA ECX,DWORD PTR SS:[EBP+8]                  ;   

0045827F  |. E8 EC7CFCFF     CALL CDRWIN5.0041FF70                         ;   

00458284  |. 0FBEF0          MOVSX ESI,AL  

00458287  |. 6A 13           PUSH 13                                       ; code[0x13]  

00458289  |. 8D4D 08         LEA ECX,DWORD PTR SS:[EBP+8]                  ;   

0045828C  |. E8 DF7CFCFF     CALL CDRWIN5.0041FF70                         ;   

00458291  |. 0FBED0          MOVSX EDX,AL  

00458294  |. 03F2            ADD ESI,EDX                                    ;  加和  

00458296  |. 6A 14           PUSH 14                                       ; code[0x14]  

00458298  |. 8D4D 08         LEA ECX,DWORD PTR SS:[EBP+8]                  ;   

0045829B  |. E8 D07CFCFF     CALL CDRWIN5.0041FF70                         ;   

004582A0  |. 0FBEC0          MOVSX EAX,AL  

004582A3  |. 03F0            ADD ESI,EAX                                    ;  加和  

004582A5  |. 6A 15           PUSH 15                                       ; code[0x15]  

004582A7  |. 8D4D 08         LEA ECX,DWORD PTR SS:[EBP+8]                  ;   

004582AA  |. E8 C17CFCFF     CALL CDRWIN5.0041FF70                         ;   

004582AF  |. 0FBEC8          MOVSX ECX,AL  

004582B2  |. 034D D8         ADD ECX,DWORD PTR SS:[EBP-28]                 ;  [ebp-28] 存放结果  

004582B5  |. 03CE            ADD ECX,ESI                                    ；全部加和  

004582B7  |. 894D D8         MOV DWORD PTR SS:[EBP-28],ECX                 ；  

004582BA  |. 8B55 D8         MOV EDX,DWORD PTR SS:[EBP-28]  

004582BD  |. 6BD2 0D         IMUL EDX,EDX,0D                               ；结果乘以 0D  

004582C0  |. 8955 D8         MOV DWORD PTR SS:[EBP-28],EDX  

004582C3  |. 8B45 D8         MOV EAX,DWORD PTR SS:[EBP-28]  

004582C6  |. 99              CDQ                                           ;符号扩展为 4 字在 edx:eax 中  

004582C7  |. B9 0A000000     MOV ECX,0A  

004582CC  |. F7F9            IDIV ECX  

004582CE  |. 8955 C8         MOV DWORD PTR SS:[EBP-38],EDX                ;  除以 0A  

004582D1  |. 8B55 C8         MOV EDX,DWORD PTR SS:[EBP-38]                ;   

004582D4  |. 83C2 30         ADD EDX,30                                    ;  余数加 0x30 变回字符  

004582D7  |. 8955 C8         MOV DWORD PTR SS:[EBP-38],EDX  
```
