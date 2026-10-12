
# 瑞星世纪版密钥盘

> 原文：[`www.pediy.com/kssd/pediy03/forum303.htm`](https://www.pediy.com/kssd/pediy03/forum303.htm)

```assembly
    STACK SEGMENT PARA STACK 'STACK'
    
    DW 128 DUP(?)
    
    STACK ENDS
    
    DATA SEGMENT PARA PUBLIC 'DATA'
    
    PARA1 DB 4FH,00,01,02
        DB 4FH,00,02,02
        DB 4FH,01,03,02
        DB 4FH,00,0A7H,02
        DB 4eH,01,0D1H,02
        DB 4FH,01,0BCH,02
        DB 4FH,00,0EH,02
        DB 04H,01,08,02
        DB 4FH,01,67h,02
        
    DISP1 DB 'Failure!',0ah,0dh,24h
    DISP2 DB 'Success!',0ah,0dh,24h
    DISP3 DB 'RAV 99 Century Cracked,Only use by yourself!',0AH,0DH,24H
    DISP4 DB 'Cracked by Zhuxiaohua 1999.08.25',0AH,0DH,24H
    
    DATA ENDS
    
    CODE SEGMENT PARA PUBLIC 'CODE'
    
    ASSUME CS:CODE,DS:DATA,SS:STACK
    
        MAIN PROC FAR
        PUSH DS
        XOR AX,AX
        PUSH AX
        MOV AX,DATA
        MOV DS,AX
        MOV ES,AX
        
        LEA DX,DISP3
        MOV AH,09H
        INT 21H
        
        LEA DX,DISP4
        INT 21H
        
        PUSH ES
        MOV AH,35H
        MOV AL,1EH
        INT 21H
        
        MOV AL,00
        MOV ES:[BX+3],AL
        MOV AL,09
        MOV ES:[BX+4],AL
        MOV AL,6CH
        MOV ES:[BX+7],AL
        POP ES
        
        MOV  CX,05
        
    REP1:PUSH CX
        MOV DX,0100h
        MOV CH,4FH
        LEA BX,PARA1
        MOV AH,05
        INT 13H
        JNB EXIT
        
        POP CX
        DEC CX
        JZ FAIL
        JMP REP1
        
    FAIL:LEA DX,DISP1
        MOV AH,09H
        INT 21H
        JMP EXIT1
        
    EXIT:POP CX
        LEA DX,DISP2
        MOV AH,09H
        INT 21H
        JMP EXIT1
        
    EXIT1:PUSH ES
        MOV AH,35H
        MOV AL,1EH
        INT 21H
        
        MOV AL,02H
        MOV ES:[BX+3],AL
        MOV AL,12H
        MOV ES:[BX+4],AL
        MOV AL,54H
        MOV ES:[BX+7],AL
        POP ES
        RET
        
    MAIN ENDP
    
    CODE ENDS
    
    END MAIN   4dd466d0e1e0cc123afbe504da2462ca
```
