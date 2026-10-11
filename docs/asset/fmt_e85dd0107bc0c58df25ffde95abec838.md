
```
00591059    33C2            XOR EAX,EDX  
0059105B    81C2 10B43C8D   ADD EDX,8D3CB410  
00591061    89048E          MOV DWORD PTR DS:[ESI+ECX*4],EAX  
00591064    49              DEC ECX  
00591065  ^ EB E1           JMP SHORT 00591048  
00591067    61              POPAD  
00591068    61              POPAD  
00591069    C3              RETN  
```

`sub_SDK_Disposal`:

```
0059106A >  60              PUSHAD                               ; sub_SDK_Disposal  
0059106B    E8 90B0FFFF     CALL <sub_GetEBP >  
00591070    33D2            XOR EDX,EDX  
00591072 >  8BB495 3D1B4000 MOV ESI,DWORD PTR SS:[EBP+EDX*4+401B3D]  ; 如果用了 SDK 这里面的值不为空.  
00591079    0BF6            OR ESI,ESI                            ; 如果没有用 SDK 或 SDK 处理部分已经操作完成则跳  
0059107B    74 31           JE SHORT <finished>  
0059107D    90              NOP  
0059107E    90              NOP  
0059107F    90              NOP  
00591080    90              NOP  
00591081    03B5 46F84000   ADD ESI,DWORD PTR SS:[EBP+40F846]       ; 把 RVA 转为 VA  
00591087    8B8C95 CD1C4000 MOV ECX,DWORD PTR SS:[EBP+EDX*4+401CCD]  ; 加密的代码大小  
0059108E    60              PUSHAD  
0059108F    52              PUSH EDX  
00591090    51              PUSH ECX                              ; /MemSize  
00591091    6A 40           PUSH 40                               ; |Flags = GPTR  
00591093    FF95 FFFC4000   CALL DWORD PTR SS:[EBP+40FCFF]         ; \GlobalAlloc  
00591099    5A              POP EDX  
0059109A    898495 5D1E4000 MOV DWORD PTR SS:[EBP+EDX*4+401E5D],EAX  ; 保存动态申请的地址  
005910A1    61              POPAD  
005910A2    8BBC95 5D1E4000 MOV EDI,DWORD PTR SS:[EBP+EDX*4+401E5D]  
005910A9    F3:A4           REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[>; 把代码复制到动态申请的地址空间  
005910AB    42              INC EDX  
005910AC  ^ EB C4           JMP SHORT <Loop MEMCopy>  
005910AE >  61              POPAD  
005910AF    C3              RETN  
```

也就是说壳加密时已经处理好了。这个壳只是加载一而已，看看数据窗口，总结一下就这么回事：

```
0057CB3D  D1 AB 02 00 A1 DE 02 00 D6 CC 01 00 32 C9 06 00  勋.∞.痔.2?.             ;这里就是用了 SDK 的各地址的 RVA  
0057CB4D  DF 3A 02 00 38 FC 05 00 C8 2E 06 00 0F 2C 05 00  ?.8?.?.,.  
0057CB5D  0F 93 03 00 71 FE 00 00 8B 61 06 00 7D C7 03 00  ?.q?.媋.}?.  
0057CB6D  3D FA 03 00 BD 2D 04 00 7D 60 04 00 3D 93 04 00  =?.?.}\/=?.  
0057CB7D  DD F8 04 00 F0 5E 05 00 5D C9 05 00 96 43 08 00  蔌.館.\]?.朇.  
0057CB8D  89 93 05 00 00 C6 04 00 D9 08 07 00 22 74 02 00  墦..?.?."t.  
0057CB9D  02 DE 07 00 D7 32 01 00 D1 66 01 00 A2 17 03 00  ?.?.裦.?.  
0057CBAD  0F A8 07 00 9B 9A 01 00 9A 08 02 00 05 CC 00 00  ?.洑.?.?.  
0057CBBD  FF 96 06 00 26 11 08 00 67 60 03 00 C3 75 07 00  ?.&.g\/胾.  
0057CBCD  89 43 07 00 00 00 00 00 00 00 00 00 00 00 00 00  塁.............  
......  
0057CCCD  55 32 00 00 63 32 00 00 69 32 00 00 6B 32 00 00  U2..c2..i2..k2..             ;用了 SDK 的大小  
0057CCDD  6C 32 00 00 6D 32 00 00 6D 32 00 00 6E 32 00 00  l2..m2..m2..n2..  
0057CCED  7B 32 00 00 88 32 00 00 89 32 00 00 9B 32 00 00  {2..?..?..?..  
0057CCFD  9B 32 00 00 9B 32 00 00 9B 32 00 00 9B 32 00 00  ?..?..?..?..  
0057CD0D  9B 32 00 00 9B 32 00 00 9B 32 00 00 AF 32 00 00  ?..?..?..?..  
0057CD1D  B2 32 00 00 B5 32 00 00 D5 32 00 00 DA 32 00 00  ?..?..?..?..  
0057CD2D  DA 32 00 00 E6 32 00 00 F8 32 00 00 3B 33 00 00  ?..?..?..;3..  
0057CD3D  CB 33 00 00 DC 64 00 00 F2 64 00 00 24 65 00 00  ?..躣..騞..$e..  
0057CD4D  27 65 00 00 8C 65 00 00 E2 65 00 00 1E 67 00 00  'e..宔..鈋..g..  
0057CD5D  AE 99 00 00 00 00 00 00 00 00 00 00 00 00 00 00  畽..............  
```

`sub_Unknow1`:

```
0058CFBF >  60                PUSHAD                               ;对 DELPHI 的程序，这里会产生异常  
0058CFC0    F8                CLC  
0058CFC1    BE 97B4BED0       MOV ESI,D0BEB497  
0058CFC6    E8 01000000       CALL 0058CFCC  
0058CFCB  ^ 72 83             JB SHORT 0058CF50  
0058CFCD    C40481            LES EAX,FWORD PTR DS:[ECX+EAX*4]     ; Modification of segment register  
0058CFD0    CD D9             INT 0D9  
......  
0058D16A >  E8 91EFFFFF       CALL <sub_GetEBP >  
0058D16F    E8 580F0000       CALL 0058E0CC                         ; EAX 随机产生的值  
0058D174    0BC0              OR EAX,EAX  
0058D176    75 3D             JNZ SHORT 0058D1B5                    ;  没有异常则跳/?,我这个程序里跳  
0058D178    90                NOP  
0058D179    90                NOP  
0058D17A    90                NOP  
0058D17B    90                NOP  
0058D17C    FF95 6FFD4000     CALL DWORD PTR SS:[EBP+40FD6F]         ;  GetCurrentThread  
0058D182    50                PUSH EAX  
0058D183    8DB5 6EF74000     LEA ESI,DWORD PTR SS:[EBP+40F76E]  
0058D189    56                PUSH ESI  
0058D18A    56                PUSH ESI                              ;  /pContext = MainCon.0058A76E  
0058D18B    50                PUSH EAX                              ;  |hThread = FFFFFFFE  
0058D18C    FF95 67FD4000     CALL DWORD PTR SS:[EBP+40FD67]         ;  \GetThreadContext  
0058D192    5F                POP EDI  
0058D193    83C7 04           ADD EDI,4  
0058D196    2BC0              SUB EAX,EAX  
0058D198    B9 04000000       MOV ECX,4  
0058D19D    F3:AB             REP STOS DWORD PTR ES:[EDI]  
0058D19F    8DB5 6EF74000     LEA ESI,DWORD PTR SS:[EBP+40F76E]  
0058D1A5    58                POP EAX  
0058D1A6    56                PUSH ESI                              ;   /pContext  
0058D1A7    50                PUSH EAX                              ;   |hThread  
0058D1A8    FF95 6BFD4000     CALL DWORD PTR      SS:[EBP+40FD6B]     ;   \SetThreadContext  
0058D1AE    C685 BF1F4100 C3 MOV BYTE PTR SS:[EBP+411FBF],0C3  
0058D1B5    60                PUSHAD                               ;   把代码加密回去  
0058D1B6    E8 00000000       CALL 0058D1BB  
0058D1BB    5E                POP ESI  
0058D1BC    83EE 06           SUB ESI,6  
0058D1BF    B9 4B000000       MOV ECX,4B  
0058D1C4    29CE              SUB ESI,ECX  
0058D1C6    BA D0D6B318       MOV EDX,18B3D6D0  
0058D1CB    C1E9 02           SHR ECX,2  
0058D1CE    83E9 02           SUB ECX,2  
0058D1D1    83F9 00           /CMP ECX,0  
0058D1D4    7C 1A             |JL SHORT 0058D1F0  
0058D1D6    8B048E            |MOV EAX,DWORD PTR DS:[ESI+ECX*4]  
0058D1D9    8B5C8E 04         |MOV EBX,DWORD PTR DS:[ESI+ECX*4+4]  
0058D1DD    03C3              |ADD EAX,EBX  
0058D1DF    C1C8 13           |ROR EAX,13  
0058D1E2    03C2              |ADD EAX,EDX  
0058D1E4    81C2 B0B2E98F     |ADD EDX,8FE9B2B0  
0058D1EA    89048E            |MOV DWORD PTR DS:[ESI+ECX*4],EAX  
0058D1ED    49                |DEC ECX  
0058D1EE  ^ EB E1             \\JMP SHORT 0058D1D1  
0058D1F0    61                POPAD  
0058D1F1    61                POPAD  
0058D1F2    C3                RETN     
```

`sub_Kernel`:

```
0058ED39 >  60              PUSHAD                               ; sub_Kernel  
0058ED3A    F8              CLC  
0058ED3B    66:BE DA51      MOV SI,51DA  
0058ED3F    45              INC EBP  
0058ED40    90              NOP  
0058ED41    90              NOP  
0058ED42    90              NOP  
0058ED43    90              NOP  
......  
0058EEE4    E8 17D2FFFF     CALL <sub_GetEBP >  
0058EEE9    80BD A5524100 C>CMP BYTE PTR SS:[EBP+4152A5],0C3  
0058EEF0    74 09           JE SHORT <FinishedIAT>  
0058EEF2    90              NOP  
0058EEF3    90              NOP  
0058EEF4    90              NOP  
0058EEF5    90              NOP  
```
