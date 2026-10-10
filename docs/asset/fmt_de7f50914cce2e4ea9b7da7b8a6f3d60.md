
# 贴一个可以对付 DOS 下所有磁盘加密的仿真软件，现在还有用...

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-308.htm`](https://www.pediy.com/kssd/pediy07/pediy7-308.htm)

```
CSEG    SEGMENT  
        ASSUME CS:CSEG,DS:CSEG,ES:CSEG  
        ORG 100H  
START:          JMP   INSTALL_TSR  
OLD_INT15  DD  ?  
OLD_INT40  DD  ?  
DISK_BIOS  DW 0EC59H,0F000H  
F_NAME DB '000000.DAT',0  
OFF_IP DW 0  
OFF_CS DW 0  
OFF_AX DW 0  
OFF_CX DW 0  
OFF_DX DW 0  
HANDLE DW  ?  
IN_DOS DD  ?  
OLD_AX DW  ?  
OLD_CX DW  ?  
OLD_DH DB  ?  
OLD_F DW  ?  
NEW_CS DW  ?  
NEW_IP DW  ?  
OLD_CS DW  ?  
OLD_IP DW  ?  
SEC_NUM DB 0  
NEW_INT15      PROC FAR  
               CLI  
               PUSHF  
               PUSH DI  
               CMP AX,9001H  
               JNZ DOS_BUY  
               MOV DI,CS:[OFF_AX]  
               CMP BYTE PTR [BP+DI+1],02H             ; ah=02 is read disk  
               JNZ DOS_BUY  
               CALL CHK_DOS  
               JZ READ_ROM  
DOS_BUY:       POP DI  
               POPF  
               STI  
               JMP DWORD PTR CS:[OLD_INT15]  
READ_ROM:      POP DI  
               POPF  
               PUSH ES  
               PUSH AX  
               PUSH BX  
               PUSH DI  
               MOV DI,CS:[OFF_AX]  
               MOV AL,BYTE PTR [BP+DI]           ; al=? is read sec num  
               MOV CS:[SEC_NUM],AL  
               MOV DI,CS:[OFF_CX]                  
               MOV AX,WORD PTR SS:[BP+DI]  
               MOV WORD PTR CS:[OLD_CX],AX       ; SAVE CX          
               MOV DI,CS:[OFF_DX]  
               MOV AL,BYTE PTR SS:[BP+DI+1]  
               MOV BYTE PTR CS:[OLD_DH],AL       ; SAVE DH  
               MOV DI,CS:[OFF_IP]  
               MOV AX,WORD PTR SS:[BP+DI]        ; BP+DI=IP  
               MOV BX,WORD PTR SS:[BP+DI+2]         ; BP+DI+2=CS  
               CMP AX,CS:[NEW_IP]  
               JNZ SET_IP  
               CMP BX,CS:[NEW_CS]  
               JNZ SET_IP  
               JMP NO_SET  
SET_IP:        MOV CS:[OLD_IP],AX  
               MOV CS:[OLD_CS],BX               ; SAVE OLD CS:IP  
               MOV AX,CS:[NEW_IP]  
               MOV BX,CS:[NEW_CS]  
               MOV WORD PTR SS:[BP+DI],AX  
               MOV WORD PTR SS:[BP+DI+2],BX        ; SET NEW CS:IP  
NO_SET:        POP DI  
               POP BX  
               POP AX  
               POP ES  
               STI  
               JMP DWORD PTR CS:[OLD_INT15]  
NEW_INT15      ENDP  
NEW_CODE       PROC FAR  
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
               JNZ HAVE_DISK                  ; NO, HAVE_DISK  
               POPF  
               JMP RETURN                     ; IS, JMP RETURN  
HAVE_DISK:     POPF  
               PUSH BX  
               PUSH ES                        ; ES:BX = KeyDisk Data Buffer  
               MOV WORD PTR CS:[OLD_AX],AX  
               PUSHF  
               POP AX  
               MOV WORD PTR CS:[OLD_F],AX  
               call get_filename  
               MOV AH,3CH  
               MOV CX,00  
               MOV DX,OFFSET F_NAME  
               INT 21H  
               MOV WORD PTR CS:[HANDLE],AX  
               PUSH CS  
               POP DS  
               MOV DX,OFFSET OLD_F  
               MOV CX,2  
               MOV BX,WORD PTR CS:[HANDLE]  
               MOV AH,40H  
               INT 21H                      ; FLAGS write to file (1,2Byte)  
               MOV DX,OFFSET OLD_AX  
               MOV CX,2  
               MOV BX,WORD PTR CS:[HANDLE]  
               MOV AH,40H  
               INT 21H                      ; AX write to file (3,4Byte)  
               POP ES  
               POP BX  
               MOV DX,BX  
               MOV AX,ES  
               MOV DS,AX  
               CALL SET_SIZE  
               MOV BX,WORD PTR CS:[HANDLE]  
               MOV AH,40H  
               INT 21H                      ; SEC_DATA write to file (5-?Byte)  
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
               PUSH CS:[OLD_F]  
               SUB SP,04  
               MOV AX,CS:[OLD_CS]  
               ADD SP,04  
               PUSH AX  
               MOV AX,CS:[OLD_IP]  
               PUSH AX  
               MOV AX,WORD PTR CS:[OLD_AX]  
               IRET  
NEW_CODE       ENDP  
NEW_INT40      PROC FAR  
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
               MOV CS:[OLD_CX],CX  
               MOV CS:[OLD_DH],DH  
               MOV CS:[SEC_NUM],AL  
               CALL CHK_DOS  
               JNZ RETU40  
               CMP AH,02H  
               JNZ RETU40  
               push cs  
               pop ds  
               PUSHF  
               CALL DWORD PTR CS:[DISK_BIOS]  
               PUSH BX  
               PUSH ES                        ; ES:BX = KeyDisk Data Buffer  
               MOV WORD PTR CS:[OLD_AX],AX  
               PUSHF  
               POP AX  
               MOV WORD PTR CS:[OLD_F],AX  
               CALL GET_FILENAME  
               MOV AH,3CH  
               MOV CX,00  
               MOV DX,OFFSET F_NAME  
               INT 21H  
               MOV WORD PTR CS:[HANDLE],AX  
               PUSH CS  
               POP DS  
               MOV DX,OFFSET OLD_F  
               MOV CX,2  
               MOV BX,WORD PTR CS:[HANDLE]  
               MOV AH,40H  
               INT 21H                      ; FLAGS write to file (1,2Byte)  
               MOV DX,OFFSET OLD_AX  
               MOV CX,2  
               MOV BX,WORD PTR CS:[HANDLE]  
               MOV AH,40H  
               INT 21H                      ; AX write to file (3,4Byte)  
               POP ES  
               POP BX  
               MOV DX,BX  
               MOV AX,ES  
               MOV DS,AX  
               CALL SET_SIZE  
               MOV BX,WORD PTR CS:[HANDLE]  
               MOV AH,40H  
               INT 21H                      ; SEC_DATA write to file (5-?Byte)  
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
               PUSH CS:[OLD_F]  
               SUB SP,04  
               MOV AX,WORD PTR CS:[OLD_AX]  
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
               JMP DWORD PTR CS:[OLD_INT40]  
NEW_INT40      ENDP  
SET_SIZE    PROC NEAR  
            PUSH AX  
            PUSH BX  
            PUSH DX  
            PUSH ES  
```
