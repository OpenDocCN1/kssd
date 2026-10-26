
# 我学软盘加解密时也用这本书，这是我的学有所用KV3000密钥盘源程序

```
.MODEL LARGE
STACK SEGMENT PARA STACK 'STACK'
DW 128 DUP(?)
STACK ENDS
DATA SEGMENT PARA PUBLIC 'DATA'
PA70_0  DB      46h,0,50,08;70道0面
        DB      46h,0,245,08
        DB      46h,0,110,08
        DB      46h,0,64,08
        DB      46h,0,87,08
PA70_1  DB      46h,1,13,08;70道1面
        DB      46h,1,37,08
        DB      46h,1,206,08
        DB      46h,1,169,08
        DB      46h,1,162,08
PA71_0  DB      47h,0,231,6
        DB      47h,0,44,6;71道0面
PA71_1  DB      47h,1,198,6
        DB      47h,1,192,6;71道1面
PA72_0  DB      48h,0,51,4;72道0面
        DB      48h,0,118,6
        DB      48h,0,147,6
        DB      48h,0,223,06h
        DB      48h,0,102,6
PA72_1  DB      48h,1,117,6
        DB      48h,1,111,6;72道1面
PA73_0  DB      49h,0,44,4
        DB      49h,0,172,6
        DB      49h,0,86,6
        DB      49h,0,103,6
        DB      49h,0,190,6
PA73_1  DB      49h,1,77,6
        DB      49h,1,179,6
PA74_0  DB      4ah,0,67,6
        DB      4ah,0,107,6
PA74_1  DB      4ah,1,39,6
        DB      4ah,1,61,6
PA75_0  DB      4bh,0,22,6
        DB      4bh,0,18,6
PA75_1  DB      4bh,1,115,6
        DB      4bh,1,86,6
PA76_0  DB      4ch,0,102,6
        DB      4ch,0,52,6
PA76_1  DB      4ch,1,99,6
        DB      4ch,1,244,6
PA77_0  DB      4dh,0,25,6
        DB      4dh,0,101,5
        DB      4dh,0,138,6
        DB      4dh,0,83,6
        DB      4dh,0,53,6
PA77_1  DB      4dh,1,3,6
        DB      4dh,1,117,6
PA78_0  DB      4eh,0,133,7
        DB      4eh,0,146,6
        DB      4eh,0,254,4
        DB      4eh,0,252,6
        DB      4eh,0,156,6
PA78_1  DB      4eh,1,56,06
        DB      4eh,1,199,6
PA79_0  DB      4Fh,0,126,6
        DB      4Fh,0,100,6
PA79_1  DB      4fh,1,31,04
        DB      4fh,1,145,06
        DB      4fh,1,166,08
        DB      4fh,1,104,06
        DB      4fh,1,201,6
PA80_0    DB      50h,0,19,08
          DB      50h,0,125,06
          DB      50h,0,133,06
          DB      50h,0,56,05
          DB      50h,0,247,06h
PA80_1  DB      50h,1,101,6
        DB      50h,1,248,6
f710 db 'k71044.fin',0
f711 db 'k711192.fin',0
f721 db 'k721111.fin',0
f731 db 'k731179.fin',0
f740 db 'k740107.fin',0
f741 db 'k74161.fin',0
f750 db 'k75018.fin',0
f751 db 'k75186.fin',0
f760 db 'k76052.fin',0
f761 db 'k761244.fin',0
f771 db 'k771117.fin',0
f781 db 'k781199.fin',0
f790 db 'k790100.fin',0
f801 db 'k801248.fin',0
DISP1 DB 'Failure!',0ah,0dh,24h
DISP2 DB 'Success!',0ah,0dh,24h
DISP3 DB 'Making KV3000 Key Disk!',0AH,0DH,24H
DISP4 DB 'CRACKED by ZHUXIAOHUA 2000.11.18',0AH,0DH,24H
DISP5 DB 'TRY A BUG',0AH,0DH,24H
DISP6 DB 'Insert a disk in drive A: press any key to continue!',0ah,0dh,24h
DISP7 DB 'E-mail:zhuxiaohua@126.com', 0ah,0dh,24h
READ_ERR DB 'Can not open fin file or file not exist!',0AH,0DH,24H
DISP700 DB 'Track 70 side 0 made OK',0AH,0DH,24H
DISP701 DB 'Track 70 side 1 made OK',0ah,0dh,24h
DISP710 DB 'Track 71 side 0 made OK',0AH,0DH,24H
DISP711 DB 'Track 71 side 1 made OK',0ah,0dh,24h
DISP720 DB 'Track 72 side 0 made OK',0ah,0dh,24h
DISP721 DB 'Track 72 side 1 made OK',0AH,0DH,24H
DISP730 DB 'Track 73 side 0 made OK',0AH,0DH,24H
DISP731 DB 'Track 73 side 1 made OK',0ah,0dh,24h
DISP740 DB 'Track 74 side 0 made OK',0ah,0dh,24h
DISP741 DB 'Track 74 side 1 made OK',0ah,0dh,24h
DISP750 DB 'Track 75 side 0 made OK',0ah,0dh,24h
DISP751 DB 'Track 75 side 1 made OK',0AH,0DH,24H
DISP760 DB 'Track 76 side 0 made OK',0ah,0dh,24h
DISP761 DB 'Track 76 side 1 made OK',0AH,0DH,24H
DISP770 DB 'Track 77 side 0 made OK',0AH,0DH,24H
DISP771 DB 'Track 77 side 1 made OK',0ah,0dh,24h
DISP780 DB 'Track 78 side 0 made OK',0AH,0DH,24H
DISP781 DB 'Track 78 side 1 made OK',0ah,0dh,24h
DISP790 DB 'Track 79 side 0 made OK',0ah,0dh,24h
DISP791 DB 'Track 79 side 1 made OK',0ah,0dh,24h
DISP800 DB 'Track 80 side 0 made OK',0ah,0dh,24h
DISP801 DB 'Track 80 side 1 made OK',0ah,0dh,24h
DISP710W DB 'Track 71 side 0 Write OK',0AH,0DH,24H
DISP711W DB 'Track 71 side 1 Write OK',0ah,0dh,24h
DISP721W DB 'Track 72 side 1 Write OK',0AH,0DH,24H
DISP731W DB 'Track 73 side 1 Write OK',0ah,0dh,24h
DISP740W DB 'Track 74 side 0 Write OK',0ah,0dh,24h
DISP741W DB 'Track 74 side 1 Write OK',0ah,0dh,24h
DISP750W DB 'Track 75 side 0 Write OK',0ah,0dh,24h
DISP751W DB 'Track 75 side 1 Write OK',0AH,0DH,24H
DISP760W DB 'Track 76 side 0 Write OK',0ah,0dh,24h
DISP761W DB 'Track 76 side 1 Write OK',0AH,0DH,24H
DISP770W DB 'Track 77 side 0 Write OK',0AH,0DH,24H
DISP771W DB 'Track 77 side 1 Write OK',0ah,0dh,24h
DISP780W DB 'Track 78 side 0 Write OK',0AH,0DH,24H
DISP781W DB 'Track 78 side 1 Write OK',0ah,0dh,24h
DISP790W DB 'Track 79 side 0 Write OK',0ah,0dh,24h
DISP791W DB 'Track 79 side 1 Write OK',0ah,0dh,24h
DISP800W DB 'Track 80 side 0 Write OK',0ah,0dh,24h
DISP801W DB 'Track 80 side 1 Write OK',0ah,0dh,24h
ERR DB 00;格式化或写盘成功标志01成功00失败
DATA_CX DW 0000H;格式化磁盘磁道号临时存区CX
DATA_BX DW 0000H;
DATA_FILE DB 8192 DUP(0)
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
    LEA DX,DISP6
    INT 21H
    MOV AH,0
    INT 16H
    PUSH ES
    MOV AH,35H
    MOV AL,1EH
    INT 21H
    MOV AL,04
    MOV ES:[BX+3],AL;扇区大小
    MOV AL,05;每道格式化扇区数
    MOV ES:[BX+4],AL
    MOV AL,54H;扇区间隙正常为54h
    MOV ES:[BX+7],AL
    MOV AL,06fH;充填字节，正常为F6H
    MOV ES:[BX+8],AL
    POP ES
    MOV AX,4600H;开始格式化70道0面
    LEA BX,DATA_CX
    MOV ES:[BX],AX
    MOV DX,0;0面
    LEA AX,PA70_0
    LEA BX,DATA_BX
    MOV ES:[BX],AX
    CALL FORMATS
    LEA BX,ERR;判断是否出错
    MOV AX,[BX]
    CMP AX,00
    JZ  R700
    JMP FAIL
R700: LEA DX,DISP700
    MOV AH,09H
    INT 21H
    PUSH ES
    MOV AH,35H
    MOV AL,1EH
    INT 21H
    MOV AL,04
    MOV ES:[BX+3],AL;扇区大小
    MOV AL,05;每道格式化扇区数
    MOV ES:[BX+4],AL
    MOV AL,54H;扇区间隙正常为54h
    MOV ES:[BX+7],AL
    MOV AL,06fH;充填字节，正常为F6H
    MOV ES:[BX+8],AL
    POP ES
    MOV AX,4600H;开始格式化70道1面
    LEA BX,DATA_CX
    MOV [BX],AX
    MOV DX,0100h;1面
    LEA AX,PA70_1
    LEA BX,DATA_BX
    MOV [BX],AX
    CALL FORMATS
    LEA BX,ERR;判断是否出错
    MOV AX,[BX]
    CMP AX,00
    JZ  R701
    JMP FAIL
R701:LEA DX,DISP701
    MOV AH,09H
    INT 21H
    PUSH ES
    MOV AH,35H
    MOV AL,1EH
    INT 21H
    MOV AL,04
    MOV ES:[BX+3],AL;扇区大小
    MOV AL,02;每道格式化扇区数
    MOV ES:[BX+4],AL
    MOV AL,54H;扇区间隙正常为54h
    MOV ES:[BX+7],AL
    MOV AL,06fH;充填字节，正常为F6H
    MOV ES:[BX+8],AL
    POP ES
    MOV AX,4700H;开始格式化71道0面
    LEA BX,DATA_CX
    MOV [BX],AX
    MOV DX,0;0面
    LEA AX,PA71_0
    LEA BX,DATA_BX
    MOV [BX],AX
    CALL FORMATS
    LEA BX,ERR;判断是否出错
    MOV AX,[BX]
    CMP AX,00
    JZ  R710
    JMP FAIL
R710:LEA DX,DISP710
    MOV AH,09H
    INT 21H
    PUSH ES
    MOV AH,35H
    MOV AL,1EH
    INT 21H
    MOV AL,04
    MOV ES:[BX+3],AL;扇区大小
    MOV AL,02;每道格式化扇区数
    MOV ES:[BX+4],AL
    MOV AL,54H;扇区间隙正常为54h
    MOV ES:[BX+7],AL
    MOV AL,06fH;充填字节，正常为F6H
    MOV ES:[BX+8],AL
    POP ES
    MOV AX,4700H;开始格式化71道1面
    LEA BX,DATA_CX
    MOV [BX],AX
    MOV DX,0100h;1面
    LEA AX,PA71_1
    LEA BX,DATA_BX
    MOV [BX],AX
    CALL FORMATS
    LEA BX,ERR;判断是否出错
    MOV AX,[BX]
    CMP AX,00
    JZ R711
    JMP FAIL
R711:LEA DX,DISP711
    MOV AH,09H
    INT 21H
    PUSH ES
    MOV AH,35H
    MOV AL,1EH
    INT 21H
    MOV AL,04
    MOV ES:[BX+3],AL;扇区大小
    MOV AL,05;每道格式化扇区数
    MOV ES:[BX+4],AL
    MOV AL,54H;扇区间隙正常为54h
    MOV ES:[BX+7],AL
    MOV AL,06fH;充填字节，正常为F6H
    MOV ES:[BX+8],AL
    POP ES
    MOV AX,4800H;开始格式化72道0面
    LEA BX,DATA_CX
    MOV [BX],AX
    MOV DX,0;0面
    LEA AX,PA72_0
    LEA BX,DATA_BX
```
