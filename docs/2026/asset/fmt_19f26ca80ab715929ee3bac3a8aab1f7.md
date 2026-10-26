
```
053BEFBA    50              PUSH EAX

053BEF74    A1 84A03B05     MOV EAX,DWORD PTR DS:[53BA084]

053BEFC7    F7D0            NOT EAX

053BEF8D    50              PUSH EAX                                                      ; KERNEL32.LoadLibraryA

053BEF67    A1 60A03B05     MOV EAX,DWORD PTR DS:[53BA060]

053BEF7E    F7D0            NOT EAX

053BEF53    50              PUSH EAX                                                      ; KERNEL32.7C570000

053BEE9B    FF15 DED33B05   CALL DWORD PTR DS:[53BD3DE]                                   ; EXERe.053A102C,  ret = 155098

053BEEB8    F7D0            NOT EAX

053BEECF    A3 84A03B05     MOV DWORD PTR DS:[53BA084],EAX                                ; 替换原来的入口,  SKIP \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

053BEEBF    8D45 E0         LEA EAX,DWORD PTR SS:[EBP-20]

053BEEEB    6A 01           PUSH 1

053BEF01    50              PUSH EAX

053BEF1D    A1 A8A03B05     MOV EAX,DWORD PTR DS:[53BA0A8]

053BEF2D    F7D0            NOT EAX

053BEDB3    50              PUSH EAX                                                      ; KERNEL32.TerminateProcess

053BEF0D    A1 60A03B05     MOV EAX,DWORD PTR DS:[53BA060]

053BEEAF    F7D0            NOT EAX

053BEE4B    50              PUSH EAX                                                      ; KERNEL32.7C570000

053BEE60    FF15 8EEE3B05   CALL DWORD PTR DS:[53BEE8E]                                   ; EXERe.053A102C, ret = 1558d0

053BEE75    F7D0            NOT EAX

053BEE82    A3 A8A03B05     MOV DWORD PTR DS:[53BA0A8],EAX                                ; 替换原来的入口,  SKIP \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

053BEDE1    8D45 E0         LEA EAX,DWORD PTR SS:[EBP-20]

053BEDD2    6A 01           PUSH 1

053BEE2B    50              PUSH EAX

053BEDE9    A1 9CA03B05     MOV EAX,DWORD PTR DS:[53BA09C]

053BEE02    F7D0            NOT EAX

053BEE0F    50              PUSH EAX                                                      ; KERNEL32.ResumeThread

053BEC6F    A1 60A03B05     MOV EAX,DWORD PTR DS:[53BA060]

053BEC4C    F7D0            NOT EAX

053BEC59    50              PUSH EAX                                                      ; KERNEL32.7C570000

053BECA4    FF15 46EC3B05   CALL DWORD PTR DS:[53BEC46]                                   ; EXERe.053A102C, ret = 52EE58

053BEC92    8B4D DC         MOV ECX,DWORD PTR SS:[EBP-24]                                 ; EXERe.053C4100

053BED0D    83C4 30         ADD ESP,30

053BEC80    F7D0            NOT EAX

053BECC8    A3 9CA03B05     MOV DWORD PTR DS:[53BA09C],EAX                                ; 替换原来的入口,  SKIP \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
```

等我出差回来, 继续

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
