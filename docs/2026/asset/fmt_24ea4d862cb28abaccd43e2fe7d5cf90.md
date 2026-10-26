
```
0058FC5A    90              NOP

0058FC5B    90              NOP

0058FC5C    90              NOP

0058FC5D    90              NOP

0058FC5E    90              NOP

0058FC5F    90              NOP

0058FC60    90              NOP

0058FC61    90              NOP

0058FC62    90              NOP

0058FC63    0000            ADD BYTE PTR DS:[EAX],AL

0058FC65    43              INC EBX

0058FC66    3A5C43 30       CMP BL,BYTE PTR DS:[EBX+EAX*2+30]

0058FC6A    6E              OUTS DX,BYTE PTR ES:[EDI]                ;         I/O command

0058FC6B    66:6967 2E 7361 IMUL SP,WORD PTR DS:[EDI+2E],6173

0058FC71    76 00           JBE SHORT 0058FC73

0058FC73    0000            ADD BYTE PTR DS:[EAX],AL

0058FC75    0000            ADD BYTE PTR DS:[EAX],AL

0058FC77    0000            ADD BYTE PTR DS:[EAX],AL

0058FC79    0000            ADD BYTE PTR DS:[EAX],AL

0058FC7B    0000            ADD BYTE PTR DS:[EAX],AL

0058FC7D    0000            ADD BYTE PTR DS:[EAX],AL

0058FC7F    0000            ADD BYTE PTR DS:[EAX],AL

0058FC81    0000            ADD BYTE PTR DS:[EAX],AL

0058FC83    0000            ADD BYTE PTR DS:[EAX],AL

0058FC85    0000            ADD BYTE PTR DS:[EAX],AL

0058FC87    0000            ADD BYTE PTR DS:[EAX],AL

0058FC89 >  6A 00           PUSH 0                                   ; /hTemplateFile = NULL

0058FC8B    6A 02           PUSH 2                                   ; |Attributes = HIDDEN

0058FC8D    6A 04           PUSH 4                                   ; |Mode = OPEN_ALWAYS

0058FC8F    6A 00           PUSH 0                                   ; |pSecurity = NULL

0058FC91    6A 03           PUSH 3                                   ; |ShareMode = FILE_SHARE_READ|FILE_SHARE_WRITE

0058FC93    68 000000C0     PUSH C0000000                            ; |Access = GENERIC_READ|GENERIC_WRITE

0058FC98    8DB5 654C4100   LEA ESI,DWORD PTR SS:[EBP+414C65]        ; |

0058FC9E    56              PUSH ESI                                 ; |Try information FileName

0058FC9F    50              PUSH EAX                                 ; |

0058FCA0    8B85 2BFD4000   MOV EAX,DWORD PTR     SS:[EBP+40FD2B]      ; |

0058FCA6    0FB600          MOVZX EAX,BYTE PTR DS:[EAX]              ; |

0058FCA9    83E8 33         SUB EAX,33                               ; |

0058FCAC    3D 99000000     CMP EAX,99                               ; |

0058FCB1    74 10           JE SHORT 0058FCC3                        ; |

0058FCB3    90              NOP                                      ; |

0058FCB4    90              NOP                                      ; |

0058FCB5    90              NOP                                      ; |

0058FCB6    90              NOP                                      ; |

0058FCB7    58              POP EAX                                  ; |

0058FCB8    FF95 2BFD4000   CALL DWORD PTR SS:[EBP+40FD2B]           ; \CreateFileA

0058FCBE    EB 17           JMP SHORT 0058FCD7

0058FCC0    90              NOP

0058FCC1    90              NOP

0058FCC2    90              NOP

0058FCC3    B8 E8030000     MOV EAX,3E8

0058FCC8    E8 01BCFFFF     CALL <Get_RND_VALUE>

0058FCCD    8DBD 615D4000   LEA EDI,DWORD PTR SS:[EBP+405D61]

0058FCD3    03F8            ADD EDI,EAX

0058FCD5    AB              STOS DWORD PTR ES:[EDI]

0058FCD6    58              POP EAX

0058FCD7    0BC0            OR EAX,EAX

0058FCD9    75 32           JNZ SHORT 0058FD0D

0058FCDB    90              NOP

0058FCDC    90              NOP

0058FC DD    90              NOP

0058FCDE    90              NOP

0058FCDF    8DB5 F64C4100   LEA ESI,DWORD PTR SS:[EBP+414CF6]

0058FCE5    6A 00           PUSH 0

0058FCE7    56              PUSH ESI

0058FCE8    56              PUSH ESI

0058FCE9    6A 00           PUSH 0

0058FCEB    FF95 2C854100   CALL DWORD PTR SS:[EBP+41852C]

0058FCF1    E9 DD020000     JMP <Use>

0058FCF6    74 72           JE SHORT 0058FD6A

0058FCF8    6961 6C 2063726>IMUL ESP,DWORD PTR DS:[ECX+6C],65726320

0058FCFF    61              POPAD

0058FD00    74 65           JE SHORT 0058FD67

0058FD02    66:696C65 20 65>IMUL BP,WORD PTR SS:[EBP+20],7265

0058FD09    72 6F           JB SHORT 0058FD7A

0058FD0B    72 00           JB SHORT 0058FD0D

0058FD0D    8985 774C4100   MOV DWORD PTR SS:[EBP+414C77],EAX        ;         这里开始读取试用信息之类的

0058FD13    6A 00           PUSH 0

0058FD15    8DB5 854C4100   LEA ESI,DWORD PTR SS:[EBP+414C85]

......

005901B9    E8 42BFFFFF     CALL <sub_GetEBP >

005901BE    80BD 744A4100 0>CMP BYTE PTR SS:[EBP+414A74],0

005901C5    0F84 97000000   JE <Try way>                             ; 如果为0则表示没有注册提示

005901CB    80BD 6DF74000 0>CMP BYTE PTR SS:[EBP+40F76D],0           ; 如果为0则表示过期了

005901D2    0F85 8A000000   JNZ <Try way>

005901D8 >  6A 00           PUSH 0                                   ; Game Over

005901DA    E8 08000000     CALL <sub_Show_Expired_MSG>              ; 过期提示信息

005901DF    45              INC EBP

005901E0    78 70           JS SHORT 00590252

005901E2    6972 65 64008DB>IMUL ESI,DWORD PTR DS:[EDX+65],B58D0064

005901E9    91              XCHG EAX,ECX

005901EA    D240 00         ROL BYTE PTR DS:[EAX],CL

005901ED    56              PUSH ESI

.........

00590262 >  60              PUSHAD                                   ; 加密代码

00590263    E8 00000000     CALL 00590268

00590268    5E              POP ESI

00590269    83EE 06         SUB ESI,6

0059026C    B9 A9000000     MOV ECX,0A9

00590271    29CE            SUB ESI,ECX

00590273    BA D244E9AE     MOV EDX,AEE944D2

00590278    C1E9 02         SHR ECX,2

0059027B    83E9 02         SUB ECX,2

0059027E    83F9 00         CMP ECX,0

00590281    7C 1A           JL SHORT 0059029D

00590283    8B048E          MOV EAX,DWORD PTR DS:[ESI+ECX*4]

00590286    8B5C8E 04       MOV EBX,DWORD PTR DS:[ESI+ECX*4+4]

0059028A    33C3            XOR EAX,EBX

0059028C    C1C0 05         ROL EAX,5

0059028F    33C2            XOR EAX,EDX

00590291    81F2 5D121221   XOR EDX,2112125D

00590297    89048E          MOV DWORD PTR DS:[ESI+ECX*4],EAX

0059029A    49              DEC ECX

0059029B  ^ EB E1           JMP SHORT 0059027E

0059029D    61              POPAD

0059029E    61              POPAD

0059029F    C3              RETN

sub_AntiDebug:

0058D9EC >  60              PUSHAD                                   ; Anti_debug

0058D9ED    E8 01000000     CALL 0058D9F3

0058D9F2    9A 83042406 C37>CALL FAR 71C3:06240483                   ; Far call

0058D9F9    05 B8EB51F3     ADD EAX,F351EBB8

0058D9FE    46              INC ESI

0058D9FF    74 03           JE SHORT 0058DA04

......

0058DB97    E8 64E5FFFF     CALL <sub_GetEBP >

0058DB9C    E8 2B050000     CALL <EAX=0 Do something>

0058DBA1    0BC0            OR EAX,EAX

0058DBA3    75 50           JNZ SHORT <safe way>

0058DBA5    90              NOP

0058DBA6    90              NOP

0058DBA7    90              NOP

0058DBA8    90              NOP

0058DBA9    C685 EC294100 C>MOV BYTE PTR SS:[EBP+4129EC],0C3

0058DBB0    8CC8            MOV AX,CS

0058DBB2    A8 04           TEST AL,4

0058DBB4    75 1A           JNZ SHORT <OS is Win9x>                  ; 如        果不是nt系统则跳

0058DBB6    90              NOP

0058DBB7    90              NOP

0058DBB8    90              NOP

0058DBB9    90              NOP

0058DBBA    64:67:A1 3000   MOV EAX,DWORD PTR FS:[30]                ;         IsDebuggerPresent 检测ring3级调试器
```
