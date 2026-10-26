
```
003733EE    8B81 B0000000   MOV     EAX, DWORD PTR DS:[ECX+B0]                                  ; 第二次异常发生在00373506处时,ROL regEAX,13h

003733F4    C1C0 13         ROL     EAX, 13

003733F7    8981 B0000000   MOV     DWORD PTR DS:[ECX+B0], EAX                                  ; 写回regEAX

003733FD    EB 39           JMP     SHORT 00373438

003733FF    83F8 03         CMP     EAX, 3                                                      ; Dr2:00373533异常

00373402    75 2B           JNZ     SHORT <Exception Dr3>

00373404    53              PUSH    EBX

00373405    8181 B0000000 2>ADD     DWORD PTR DS:[ECX+B0], 4B23526                              ; 第三次单步异常发生在373533处时,执行add regEAX,4b23526

0037340F    8B81 B0000000   MOV     EAX, DWORD PTR DS:[ECX+B0]                                  ; regEAX

00373415    8B99 A4000000   MOV     EBX, DWORD PTR DS:[ECX+A4]                                  ; regEBX

0037341B    66:93           XCHG    AX, BX                                                      ; 相加后 ax和bx交换再加上bx

0037341D    66:03C3         ADD     AX, BX

00373420    8981 B0000000   MOV     DWORD PTR DS:[ECX+B0], EAX                                  ; 写回eax

00373426    8999 A4000000   MOV     DWORD PTR DS:[ECX+A4], EBX                                  ; 写回regEBX

0037342C    5B              POP     EBX                                                         ; 0012FFE0

0037342D    EB 09           JMP     SHORT 00373438

0037342F >  8B81 A0000000   MOV     EAX, DWORD PTR DS:[ECX+A0]

00373435    8030 55         XOR     BYTE PTR DS:[EAX], 55                                       ; Dr3:00373566异常

00373438    33C0            XOR     EAX, EAX                                                    ; 第四次异常发生时,xor [regESI],55h

0037343A    EB 44           JMP     SHORT <Return_System>

0037343C >  3D 940000C0     CMP     EAX, C0000094                                               ; DIVIDE BY ZERO

00373441    75 3A           JNZ     SHORT 0037347D

00373443    C702 00000000   MOV     DWORD PTR DS:[EDX], 0

00373449    FF81 B8000000   INC     DWORD PTR DS:[ECX+B8]                                       ; inc regEIP

0037344F    33C0            XOR     EAX, EAX                                                    ; 最后一次除0异常后，写入值到Drx中, 后面要参与计算关键值的

00373451    C741 04 2301FF0>MOV     DWORD PTR DS:[ECX+4], 0FFF0123                              ; 设置Dr0

00373458    C741 08 6745FF0>MOV     DWORD PTR DS:[ECX+8], 0FFF4567                              ; 设置Dr1

0037345F    C741 0C AB89FF0>MOV     DWORD PTR DS:[ECX+C], 0FFF89AB                              ; 设置Dr2

00373466    C741 10 EFCDFF0>MOV     DWORD PTR DS:[ECX+10], 0FFFCDEF                             ; 设置Dr3

0037346D    8161 14 F00FFFF>AND     DWORD PTR DS:[ECX+14], FFFF0FF0                             ; 设置Dr6

00373474    C741 18 5501000>MOV     DWORD PTR DS:[ECX+18], 155                                  ; 设置Dr7

0037347B    EB 03           JMP     SHORT <Return_System>

0037347D    33C0            XOR     EAX, EAX

0037347F    40              INC     EAX

00373480 >  C3              RETN                                                                ; Return_System

00373481 >  56              PUSH    ESI                                                         ; Important_SEH

00373482    8DB5 951C4000   LEA     ESI, DWORD PTR SS:[EBP+401C95]                              ; 进入int3异常之前解压出处理int3的代码

00373488    8BFE            MOV     EDI, ESI

0037348A    B9 55000000     MOV     ECX, 55

0037348F    AC              LODS    BYTE PTR DS:[ESI]

00373490    32C1            XOR     AL, CL

00373492    C0C0 04         ROL     AL, 4

00373495    AA              STOS    BYTE PTR ES:[EDI]

00373496  ^ E2 F7           LOOPD   SHORT 0037348F

00373498    5E              POP     ESI                                                         ; 一层一层的解开，真是"浪费"时间:-)

00373499    33C0            XOR     EAX, EAX

0037349B    64:FF35 0000000>PUSH    DWORD PTR FS:[0]

003734A2    64:8925 0000000>MOV     DWORD PTR FS:[0], ESP

003734A9    CC              INT3

003734AA    90              NOP

003734AB    56              PUSH    ESI

003734AC    8DB5 ED1C4000   LEA     ESI, DWORD PTR SS:[EBP+401CED]                              ; 循环解压出处理四个单步异常的代码

003734B2    8BFE            MOV     EDI, ESI

003734B4    B9 6B000000     MOV     ECX, 6B

003734B9    AC              LODS    BYTE PTR DS:[ESI]

003734BA    32C1            XOR     AL, CL

003734BC    04 4D           ADD     AL, 4D

003734BE    C0C0 03         ROL     AL, 3

003734C1    AA              STOS    BYTE PTR ES:[EDI]

003734C2  ^ E2 F5           LOOPD   SHORT 003734B9

003734C4    5E              POP     ESI                                                         ; 0012FFE0

003734C5    8D8D A01A4000   LEA     ECX, DWORD PTR SS:[EBP+<Crc_Start_addr>]                        ; 解压出处理四个单步异常的代码后，开始检测是代码是否被修改过

003734CB    2BCE            SUB     ECX, ESI                                                    ; 这一招有够利害，直接下CC断点，crc检测通不过

003734CD    33DB            XOR     EBX, EBX                                                    ; 下硬件断点的话，原来的硬件断点就破坏了．

003734CF    33C0            XOR     EAX, EAX

003734D1    AC              LODS    BYTE PTR DS:[ESI]

003734D2    03D8            ADD     EBX, EAX                                                    ; 计算校验值

003734D4  ^ E2 FB           LOOPD   SHORT 003734D1

003734D6    8BC3            MOV     EAX, EBX

003734D8    F8              CLC                                                                 ; 第一次异常处,not eax

003734D9    90              NOP

003734DA    8DB5 0D1E4000   LEA     ESI, DWORD PTR SS:[EBP+<De1>]                                ; 异常后开始解压出下面将要执行的代码

003734E0    B9 91250000     MOV     ECX, 2591

003734E5    F7E1            MUL     ECX

003734E7    D3C8            ROR     EAX, CL

003734E9    3006            XOR     BYTE PTR DS:[ESI], AL

003734EB    46              INC     ESI

003734EC    40              INC     EAX

003734ED    D40A            AAM

003734EF  ^ E2 F4           LOOPD   SHORT 003734E5

003734F1    B9 D5010000     MOV     ECX, 1D5                                                    ; De1

003734F6    8DB5 A01A4000   LEA     ESI, DWORD PTR SS:[EBP+<Crc_Start_addr>]                        ; 刚解压出来代码再次检测内存代码，把计算出的值作为解密KEY

003734FC    33C0            XOR     EAX, EAX

003734FE    3206            XOR     AL, BYTE PTR DS:[ESI]

00373500    C1C8 08         ROR     EAX, 8

00373503    46              INC     ESI

00373504  ^ E2 F8           LOOPD   SHORT 003734FE

00373506    FC              CLD                                                                 ; 第二次异常处

00373507    90              NOP

00373508    B9 64250000     MOV     ECX, 2564                                                   ; rol eax,13

0037350D    8DB5 3A1E4000   LEA     ESI, DWORD PTR SS:[EBP+401E3A]

00373513    8D4481 43       LEA     EAX, DWORD PTR DS:[ECX+EAX*4+43]

00373517    3006            XOR     BYTE PTR DS:[ESI], AL

00373519    D40A            AAM

0037351B    46              INC     ESI

0037351C  ^ E2 F5           LOOPD   SHORT 00373513                                              ; 再次把通过异常中断处理来计算eax的值

0037351E    B9 B2000000     MOV     ECX, 0B2                                                    ; 通过eax的值来解密出37351e处的代码

00373523    C1E9 02         SHR     ECX, 2

00373526    8DB5 9D1D4000   LEA     ESI, DWORD PTR SS:[EBP+401D9D]                              ; 这里从开始解密处的代码处开始计算检测值

0037352C    33DB            XOR     EBX, EBX

0037352E    AD              LODS    DWORD PTR DS:[ESI]

0037352F    33D8            XOR     EBX, EAX

00373531  ^ E2 FB           LOOPD   SHORT 0037352E                                              ; 这样在这段范围内的代码又不可以被修改又不可以被下断点

00373533    F9              STC                                                                 ; 第三次异常处

00373534    90              NOP                                                                 ; 执行add regEAX,4b23526
```
