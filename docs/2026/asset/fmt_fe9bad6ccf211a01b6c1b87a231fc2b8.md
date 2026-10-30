
# [原创]贴一个可以对付DOS下所有磁盘加密的仿真软件，现在还有用...

```
CSEG   SEGMENT

       ASSUME CS:CSEG,DS:CSEG,ES:CSEG

       ORG 100H

START:         JMP   INSTALL\_TSR

OLD\_INT15  DD ?

OLD\_INT40  DD ?

DISK\_BIOS  DW 0EC59H,0F000H

F\_NAME DB '000000.DAT',0

OFF\_IP DW 0

OFF\_CS DW 0

OFF\_AX DW 0

OFF\_CX DW 0

OFF\_DX DW 0

HANDLE DW ?

IN\_DOS DD ?

OLD\_AX DW ?

OLD\_CX DW ?

OLD\_DH DB ?

OLD\_F DW ?

NEW\_CS DW ?

NEW\_IP DW ?

OLD\_CS DW ?

OLD\_IP DW ?

SEC\_NUM DB 0

NEW\_INT15      PROC FAR

               CLI

               PUSHF

               PUSH DI

               CMP AX,9001H

               JNZ DOS\_BUY

               MOV DI,CS:[OFF\_AX]

               CMP BYTE PTR [BP+DI+1],02H             ; ah=02 is read disk

               JNZ DOS\_BUY

               CALL CHK\_DOS

               JZ READ\_ROM

DOS\_BUY:       POP DI

               POPF

               STI

               JMP DWORD PTR CS:[OLD\_INT15]

READ\_ROM:      POP DI

               POPF

               PUSH ES

               PUSH AX

               PUSH BX

               PUSH DI

               MOV DI,CS:[OFF\_AX]

               MOV AL,BYTE PTR [BP+DI]           ; al=? is read sec num

               MOV CS:[SEC\_NUM],AL

               MOV DI,CS:[OFF\_CX]

               MOV AX,WORD PTR SS:[BP+DI]

               MOV WORD PTR CS:[OLD\_CX],AX       ; SAVE CX

               MOV DI,CS:[OFF\_DX]

               MOV AL,BYTE PTR SS:[BP+DI+1]

               MOV BYTE PTR CS:[OLD\_DH],AL       ; SAVE DH

               MOV DI,CS:[OFF\_IP]

               MOV AX,WORD PTR SS:[BP+DI]        ; BP+DI=IP

               MOV BX,WORD PTR SS:[BP+DI+2]         ; BP+DI+2=CS

               CMP AX,CS:[NEW\_IP]

               JNZ SET\_IP

               CMP BX,CS:[NEW\_CS]

               JNZ SET\_IP

               JMP NO\_SET

SET\_IP:        MOV CS:[OLD\_IP],AX

               MOV CS:[OLD\_CS],BX               ; SAVE OLD CS:IP

               MOV AX,CS:[NEW\_IP]

               MOV BX,CS:[NEW\_CS]

               MOV WORD PTR SS:[BP+DI],AX

               MOV WORD PTR SS:[BP+DI+2],BX        ; SET NEW CS:IP

NO\_SET:        POP DI

               POP BX

               POP AX

               POP ES

               STI

               JMP DWORD PTR CS:[OLD\_INT15]

NEW\_INT15      ENDP

NEW\_CODE       PROC FAR

               CLI

               PUSH AX

               PUSH BX

               PUSH CX

               PUSH DX

               PUSH SI

               PUSH DI

               PUSH DS

               PUSH ES

               PUSHF

               PUSH CS

               POP DS

               PUSHF

               CMP AH,80H                     ; NO FLOPPY DISK ?

               JNZ HAVE\_DISK                  ; NO, HAVE\_DISK

               POPF

               JMP RETURN                     ; IS, JMP RETURN

HAVE\_DISK:     POPF

               PUSH BX

               PUSH ES                        ; ES:BX = KeyDisk Data Buffer

               MOV WORD PTR CS:[OLD\_AX],AX

               PUSHF

               POP AX

               MOV WORD PTR CS:[OLD\_F],AX

               call get\_filename

               MOV AH,3CH

               MOV CX,00

               MOV DX,OFFSET F\_NAME

               INT 21H

               MOV WORD PTR CS:[HANDLE],AX

               PUSH CS

               POP DS

               MOV DX,OFFSET OLD\_F

               MOV CX,2

               MOV BX,WORD PTR CS:[HANDLE]

               MOV AH,40H

               INT 21H                      ; FLAGS write to file (1,2Byte)

               MOV DX,OFFSET OLD\_AX

               MOV CX,2

               MOV BX,WORD PTR CS:[HANDLE]

               MOV AH,40H

               INT 21H                      ; AX write to file (3,4Byte)

               POP ES

               POP BX

               MOV DX,BX

               MOV AX,ES

               MOV DS,AX

               CALL SET\_SIZE

               MOV BX,WORD PTR CS:[HANDLE]

               MOV AH,40H

               INT 21H                      ; SEC\_DATA write to file (5-?Byte)

               MOV AH,3EH

               MOV BX,WORD PTR CS:[HANDLE]

               INT 21H

RETURN:        POPF

               POP ES

               POP DS

               POP DI

               POP SI

               POP DX

               POP CX

               POP BX

               POP AX

               PUSH CS:[OLD\_F]

               SUB SP,04

               MOV AX,CS:[OLD\_CS]

               ADD SP,04

               PUSH AX

               MOV AX,CS:[OLD\_IP]

               PUSH AX

               MOV AX,WORD PTR CS:[OLD\_AX]

               IRET

NEW\_CODE       ENDP

NEW\_INT40      PROC FAR

               CLI

               PUSH AX

               PUSH BX

               PUSH CX

               PUSH DX

               PUSH SI

               PUSH DI

               PUSH DS

               PUSH ES

               PUSHF

               MOV CS:[OLD\_CX],CX

               MOV CS:[OLD\_DH],DH

               MOV CS:[SEC\_NUM],AL

               CALL CHK\_DOS

               JNZ RETU40

               CMP AH,02H

               JNZ RETU40

               push cs

               pop ds

               PUSHF

               CALL DWORD PTR CS:[DISK\_BIOS]

               PUSH BX

               PUSH ES                        ; ES:BX = KeyDisk Data Buffer

               MOV WORD PTR CS:[OLD\_AX],AX

               PUSHF

               POP AX

               MOV WORD PTR CS:[OLD\_F],AX

               CALL GET\_FILENAME

               MOV AH,3CH

               MOV CX,00

               MOV DX,OFFSET F\_NAME

               INT 21H

               MOV WORD PTR CS:[HANDLE],AX

               PUSH CS

               POP DS

               MOV DX,OFFSET OLD\_F

               MOV CX,2

               MOV BX,WORD PTR CS:[HANDLE]

               MOV AH,40H

               INT 21H                      ; FLAGS write to file (1,2Byte)

               MOV DX,OFFSET OLD\_AX

               MOV CX,2

               MOV BX,WORD PTR CS:[HANDLE]

               MOV AH,40H

               INT 21H                      ; AX write to file (3,4Byte)

               POP ES

               POP BX

               MOV DX,BX

               MOV AX,ES

               MOV DS,AX

               CALL SET\_SIZE

               MOV BX,WORD PTR CS:[HANDLE]

               MOV AH,40H

               INT 21H                      ; SEC\_DATA write to file (5-?Byte)

               MOV AH,3EH

               MOV BX,WORD PTR CS:[HANDLE]

               INT 21H

               POPF

               POP ES

               POP DS

               POP DI

               POP SI

               POP DX

               POP CX

               POP BX

               POP AX

               ADD SP,06

               PUSH CS:[OLD\_F]

               SUB SP,04

               MOV AX,WORD PTR CS:[OLD\_AX]

               IRET

RETU40:        POPF

               POP ES

               POP DS

               POP DI

               POP SI

               POP DX

               POP CX

               POP BX

               POP AX

               CLI

               JMP DWORD PTR CS:[OLD\_INT40]

NEW\_INT40      ENDP

SET\_SIZE    PROC NEAR

            PUSH AX

            PUSH BX

            PUSH DX

            PUSH ES

            PUSH DI

            XOR AX,AX

            MOV ES,AX

            MOV DI,WORD PTR ES:[0078H]      ; 0000:0078H = CS:IP is disk para table

            MOV AX,WORD PTR ES:[0078H+2]

            MOV ES,AX

            MOV CL,BYTE PTR ES:[DI+3]       ; ES:DI+3 = disk sec data size

            MOV AX,128

            MOV BX,2

GO\_SIZE:    MUL BX

            DEC CL

            OR CL,CL

            JNE GO\_SIZE

            MOV BL,CS:[SEC\_NUM]

            MUL BX

            MOV CX,AX

            POP DI

            POP ES

            POP DX

            POP BX

            POP AX

            RET

SET\_SIZE    ENDP

CHK\_DOS     PROC NEAR

            PUSH BX

            PUSH ES
```
