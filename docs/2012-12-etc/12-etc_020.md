
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



# 程序代码

```asm
            PUSH DI
            XOR AX,AX
            MOV ES,AX
            MOV DI,WORD PTR ES:[0078H]      ; 0000:0078H = CS:IP is disk para table
            MOV AX,WORD PTR ES:[0078H+2]
            MOV ES,AX
            MOV CL,BYTE PTR ES:[DI+3]       ; ES:DI+3 = disk sec data size
            MOV AX,128
            MOV BX,2
GO_SIZE:    MUL BX
            DEC CL
            OR CL,CL
            JNE GO_SIZE
            MOV BL,CS:[SEC_NUM]
            MUL BX
            MOV CX,AX
            POP DI
            POP ES
            POP DX
            POP BX
            POP AX
            RET
SET_SIZE    ENDP

CHK_DOS     PROC NEAR
            PUSH BX
            PUSH ES
            MOV BX,WORD PTR CS:[IN_DOS]
            MOV ES,WORD PTR CS:[IN_DOS+2]
            CMP BYTE PTR ES:[BX],0
            POP ES
            POP BX
            RET
CHK_DOS     ENDP

GET_FILENAME   PROC NEAR
               PUSH ES
               PUSH CS
               POP ES
               MOV AX,'00'
               LEA DI,F_NAME
               MOV CX,3
               REP STOSW
               MOV AX,CS:[OLD_CX]
               LEA SI,CS:F_NAME+3
               MOV BX,16
               CALL ASCII
               CMP BYTE PTR CS:[OLD_DH],00
               JZ GET_END
               MOV BYTE PTR CS:[F_NAME+5],'1'
GET_END:       POP ES
               RET
GET_FILENAME   ENDP

ASCII      PROC NEAR
ASC1:      CMP AX,0
           JZ NONUM
           CMP AX,10
           JB AEXIT
           XOR DX,DX
           DIV BX
           OR DL,30H
           CMP DL,'9'+1            ; IF DL < '9'+1
           JB NOCHAR
           ADD DL,07
NOCHAR:    MOV [SI],DL
           DEC SI
           JMP ASC1
AEXIT:     OR AL,30H
           MOV BYTE PTR [SI],AL
NONUM:     RET
ASCII      ENDP

TSR_END_FLAGS EQU $+1
;*********************** MESSAGE   SEGMENT ***********************
name_1 DB 10h,1Fh,10h,10h,17h,10h,10h,1Fh,10h,13h,12h,12h,13h,10h,20h,40h
name_2 DB 02h,0FEh,82h,82h,0FAh,82h,82h,0FEh,02h,0F2h,12h,12h,0F2h,02h,0Eh,04h
ming_1  DB 10h,10h,10h,10h,55h,54h,54h,55h,54h,54h,54h,7Dh,44h,04h,00h,00h
ming_2  DB 20h,20h,48h,84h,0FEh,48h,86h,02h,0FCh,84h,0C4h,28h,10h,28h,44h,82h
MSG1    DB 0dh,0ah,0dh,0ah,0dh,0ah,0dh,0ah,0dh,0ah
        DB '                     赏屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯突',0ah,0dh
        DB '                     ?            Soft  Moth             ?,0ah,0dh
        DB '                     ?-----------------------------------?,0ah,0dh
        DB '                     ?Soft Name: << LockDisk Killer >>   ?,0ah,0dh
        DB '                     ?File Name: READKEY.COM             ?,0ah,0dh
        DB '                     ?Note:      This TSR Program Is For ?,0ah,0dh
        DB '                     ?           KeyDisk Protech System  ?,0ah,0dh
        DB '                     ?           Crack !                  ?,0dh,0ah
        DB '                     ?     Soft Crack Studio 1997.07.18   ?,0ah,0dh
        DB '                     ?     Programmed For Mr.   '          ,0d7h,12h,0d8h,13h
        DB                                                    '     ?,0dh,0ah
        DB '                     韧屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯屯图',0dh,0ah
        DB 0dh,0ah,'                   Press Enter " READKEY.COM " Remove Memory.',0dh,0ah
        DB 0dh,0ah,'                         Press any key to continue...',0dh,0ah,0dh,0ah,'$'
MSG2    DB 0dh,0ah,0dh,0ah,07h
        DB '                         Tsr Program Remove Memory OK!',0dh,0ah,0dh,0ah,'$'
BUFFERS  DB 80*12 DUP(0DH),'$'
MES_BUF1 DB 80 DUP(8EH),'$'
MES_BUF2 DB 80 DUP(0AH),'$'
BUF_IP   DW 0
TIME_HI  DW ?
TIME_LO  DW ?
ERR_MSG  DB 0DH,0AH,0DH,0AH
         DB '                    Open File Error , Program Break !'
         DB 0DH,0AH,0DH,0AH,'$'
BUFFER   DB 768 DUP(?)
RAM_BUF  DB 320 DUP(?)
PIC_FILE  DB 'LOGO.DAT',0
INI_FILE  DB 'C:\\LDK_100.CFG',0
INI_HANDLE DW ?
INI_MSG   DB 0DH,0AH,0DH,0AH,07H
          DB '                    Open File " LDK_100.CFG " Error !'
          DB 0DH,0AH,0DH,0AH,'$'
INI_BUF  DB 36 DUP(0)
INI_ID   DB 'This File For LockDisk Killer v1.00',1Ah
PARM     DB 0
ROM_OFF  EQU 0EC59H
ROM_SEG  EQU 0F000H
```

# 说明

这个程序是读取加密磁盘加密数据部分，读取后文件名字为：`xxxxxxx.dat`（`xxxxxxxx` 是所在的扇区号）

到现在还没发现能避开我这个软件的程序（包括最新版的 `BITLOK`）

PS：因为程序最后部分没任何技术含量了，而且后半部分有自己的个人信息不大方便放出来，作用就是替换 `INT15` 和 `40` 的中断地址，并显示自己的信息而已，没多大用。。。。。。

`4dd466d0e1e0cc123afbe504da2462ca`
