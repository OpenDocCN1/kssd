
```
00453194  |. 8B15 E41A5A00  MOV EDX,DWORD PTR DS:[5A1AE4]            ;  010Edito.\_Form010Ed

0045319A  |. 8B02           MOV EAX,DWORD PTR DS:[EDX]

0045319C  |. E8 8B830A00    CALL 010Edito.004FB52C

004531A1  |. 8B45 A0        MOV EAX,DWORD PTR SS:[EBP-60]

004531A4  |. E8 7B830A00    CALL 010Edito.004FB524

004531A9  |. EB 0D          JMP SHORT 010Edito.004531B8

004531AB  |> 8B55 A0        MOV EDX,DWORD PTR SS:[EBP-60]

004531AE  |. C782 2C020000 >MOV DWORD PTR DS:[EDX+22C],1

004531B8  |> 8B4D A4        MOV ECX,DWORD PTR SS:[EBP-5C]

004531BB  |. 64:890D 000000>MOV DWORD PTR FS:[0],ECX

004531C2  |> 8BE5           MOV ESP,EBP

004531C4  |. 5D             POP EBP

004531C5  \. C3             RETN

/////////////////////////////////////////////////////////////////////////////////////////////

00450E58  /$ 55             PUSH EBP

00450E59  |. 8BEC           MOV EBP,ESP

00450E5B  |. FF75 0C        PUSH DWORD PTR SS:[EBP+C]

00450E5E  |. FF75 08        PUSH DWORD PTR SS:[EBP+8]

00450E61  |. E8 66000000    CALL 010Edito.00450ECC         //see below

00450E66  |. 83C4 08        ADD ESP,8

00450E69  |. 83E8 2D        SUB EAX,2D                               ;  Switch (cases 2D..E7)

00450E6C  |. 74 0E          JE SHORT 010Edito.00450E7C

00450E6E  |. 83E8 21        SUB EAX,21

00450E71  |. 74 17          JE SHORT 010Edito.00450E8A

00450E73  |. 2D 99000000    SUB EAX,99

00450E78  |. 74 09          JE SHORT 010Edito.00450E83

00450E7A  |. EB 15          JMP SHORT 010Edito.00450E91

00450E7C  |> B8 DB000000    MOV EAX,0DB                              ;  Case 2D of switch 00450E69

00450E81  |. 5D             POP EBP

00450E82  |. C3             RETN

00450E83  |> B8 77010000    MOV EAX,177                              ;  Case E7 of switch 00450E69  //here

00450E88  |. 5D             POP EBP

00450E89  |. C3             RETN

00450E8A  |> B8 10000000    MOV EAX,10                               ;  Case 4E of switch 00450E69

00450E8F  |. 5D             POP EBP

00450E90  |. C3             RETN

00450E91  |> FF75 08        PUSH DWORD PTR SS:[EBP+8]                ; /Arg1; Default case of switch 00450E69

00450E94  |. E8 03030000    CALL 010Edito.0045119C                   ; \010Edito.0045119C

00450E99  |. 59             POP ECX

00450E9A  |. 83E8 17        SUB EAX,17                               ;  Switch (cases 17..138)

00450E9D  |. 74 0E          JE SHORT 010Edito.00450EAD

00450E9F  |. 83E8 13        SUB EAX,13

00450EA2  |. 74 17          JE SHORT 010Edito.00450EBB

00450EA4  |. 2D 0E010000    SUB EAX,10E

00450EA9  |. 74 09          JE SHORT 010Edito.00450EB4

00450EAB  |. EB 15          JMP SHORT 010Edito.00450EC2

00450EAD  |> B8 71000000    MOV EAX,71                               ;  Case 17 of switch 00450E9A

00450EB2  |. 5D             POP EBP

00450EB3  |. C3             RETN

00450EB4  |> B8 2F000000    MOV EAX,2F                               ;  Case 138 of switch 00450E9A

00450EB9  |. 5D             POP EBP

00450EBA  |. C3             RETN

00450EBB  |> B8 77010000    MOV EAX,177                              ;  Case 2A of switch 00450E9A

00450EC0  |. 5D             POP EBP

00450EC1  |. C3             RETN

00450EC2  |> B8 77010000    MOV EAX,177                              ;  Default case of switch 00450E9A

00450EC7  |. 5D             POP EBP

00450EC8  \. C3             RETN

///////////////////////////////////////////////////////////////////////////////////////////

00450ECC   $ 55             PUSH EBP

00450ECD   . 8BEC           MOV EBP,ESP

00450ECF   . 83C4 C0        ADD ESP,-40

00450ED2   . B8 10435800    MOV EAX,010Edito.00584310    //global variable

00450ED7   . E8 20551000    CALL 010Edito.005563FC    //初始化

00450EDC   . 8B45 08        MOV EAX,DWORD PTR SS:[EBP+8]

00450EDF   . 83C0 04        ADD EAX,4

00450EE2   . E8 A9431100    CALL 010Edito.00565290  //检测EAX是否为空

00450EE7   . 84C0           TEST AL,AL

00450EE9   . 75 0F          JNZ SHORT 010Edito.00450EFA //jump(go die!)

00450EEB   . 8B45 08        MOV EAX,DWORD PTR SS:[EBP+8]//go go go

00450EEE   . 83C0 08        ADD EAX,8

00450EF1   . E8 9A431100    CALL 010Edito.00565290  //test eax + 8 (user name )

00450EF6   . 84C0           TEST AL,AL

00450EF8   . 74 14          JE SHORT 010Edito.00450F0E //go further

00450EFA   > B8 93000000    MOV EAX,93

00450EFF   . 8B55 D8        MOV EDX,DWORD PTR SS:[EBP-28]

00450F02   . 64:8915 000000>MOV DWORD PTR FS:[0],EDX

00450F09   . E9 89020000    JMP 010Edito.00451197 //go to return

00450F0E   > 8D4D C4        LEA ECX,DWORD PTR SS:[EBP-3C]

00450F11   . 51             PUSH ECX                                 ; /Arg2

00450F12   . FF75 08        PUSH DWORD PTR SS:[EBP+8]                ; |Arg1

00450F15   . E8 6A0C0000    CALL 010Edito.00451B84                   ; \010Edito.00451B84

00450F1A   . 83C4 08        ADD ESP,8                                ;  将序列号ascii转为二进制

00450F1D   . 33C0           XOR EAX,EAX

00450F1F   . 8945 D4        MOV DWORD PTR SS:[EBP-2C],EAX

00450F22   > 66:C745 E8 080>MOV WORD PTR SS:[EBP-18],8

00450F28   . 8B55 D4        MOV EDX,DWORD PTR SS:[EBP-2C]

00450F2B   . 8B1495 6841580>MOV EDX,DWORD PTR DS:[EDX\*4+584168]      ;  =="999"

00450F32   . 8D45 FC        LEA EAX,DWORD PTR SS:[EBP-4]

00450F35   . E8 BA3F1100    CALL 010Edito.00564EF4

00450F3A   . FF45 F4        INC DWORD PTR SS:[EBP-C]

00450F3D   . 8D55 FC        LEA EDX,DWORD PTR SS:[EBP-4]

00450F40   . 8B45 08        MOV EAX,DWORD PTR SS:[EBP+8]

00450F43   . 83C0 04        ADD EAX,4

00450F46   . E8 1D421100    CALL 010Edito.00565168

00450F4B   . 50             PUSH EAX                                 ; /Arg1

00450F4C   . FF4D F4        DEC DWORD PTR SS:[EBP-C]                 ; |

00450F4F   . 8D45 FC        LEA EAX,DWORD PTR SS:[EBP-4]             ; |

00450F52   . BA 02000000    MOV EDX,2                                ; |

00450F57   . E8 28411100    CALL 010Edito.00565084                   ; \010Edito.00565084

00450F5C   . 59             POP ECX

00450F5D   . 84C9           TEST CL,CL

00450F5F   . 74 14          JE SHORT 010Edito.00450F75    //go further

00450F61   . B8 E7000000    MOV EAX,0E7

00450F66   . 8B55 D8        MOV EDX,DWORD PTR SS:[EBP-28]

00450F69   . 64:8915 000000>MOV DWORD PTR FS:[0],EDX

00450F70   . E9 22020000    JMP 010Edito.00451197

00450F75   > FF45 D4        INC DWORD PTR SS:[EBP-2C]

00450F78   . 837D D4 01     CMP DWORD PTR SS:[EBP-2C],1              ;  HERE TO GO

00450F7C   .^7C A4          JL SHORT 010Edito.00450F22               ;  NO JUMP

00450F7E   . 807D C7 9C     CMP BYTE PTR SS:[EBP-39],9C              ;  if 9C

00450F82   . 0F85 83000000  JNZ 010Edito.0045100B                    ;  NO JUMP

00450F88   . 8A4D C4        MOV CL,BYTE PTR SS:[EBP-3C]

00450F8B   . 324D CA        XOR CL,BYTE PTR SS:[EBP-36]              ;  cl =1 xor 7

00450F8E   . 884D C3        MOV BYTE PTR SS:[EBP-3D],CL              ;  save dl to [ebp-3d]

00450F91   . 8A45 C5        MOV AL,BYTE PTR SS:[EBP-3B]

00450F94   . 3245 CB        XOR AL,BYTE PTR SS:[EBP-35]              ;  al = 2 xor 8

00450F97   . 33D2           XOR EDX,EDX

00450F99   . 8AD0           MOV DL,AL                                ;  dl = al

00450F9B   . C1E2 08        SHL EDX,8                                ;  ??8bit  dh = al

00450F9E   . 8A4D C6        MOV CL,BYTE PTR SS:[EBP-3A]

00450FA1   . 324D C9        XOR CL,BYTE PTR SS:[EBP-37]              ;  cl = ?3? xor ?5?

00450FA4   . 33C0           XOR EAX,EAX

00450FA6   . 8AC1           MOV AL,CL

00450FA8   . 66:03D0        ADD DX,AX                                ;  ?????????????,

00450FAB   . 66:8955 C0     MOV WORD PTR SS:[EBP-40],DX              ;  A VALUE USE LATER

00450FAF   . 8A55 C3        MOV DL,BYTE PTR SS:[EBP-3D]              ;  ???1? xor ?7??????

00450FB2   . 52             PUSH EDX                                 ; /Arg1

00450FB3   . E8 FA170000    CALL 010Edito.004527B2                   ; \010Edito.004527B2

00450FB8   . 59             POP ECX
```
