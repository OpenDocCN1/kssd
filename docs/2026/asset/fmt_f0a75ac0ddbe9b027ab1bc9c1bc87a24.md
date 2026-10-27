
```asm
0167:0049E2E7 EB0A             JMP      SHORT 0049E2F3 ;☆▲
0167:0049E2E9 C7854CFFFFFF0000+MOV      DWORD [EBP+FFFFFF4C],00 ;☆★
;**
;☆▲
0167:0049E2F3 8B4DCC           MOV      ECX,[EBP-34] ;tempString取出的两个字符
0167:0049E2F6 51               PUSH     ECX
0167:0049E2F7 FF1528124000     CALL     MSVBVM60!__vbaR8Str
0167:0049E2FD DC1DF8434000     FCOMP    QWORD [004043F8]
0167:0049E303 DFE0             FNSTSW   AX
0167:0049E305 F6C401           TEST     AH,01
0167:0049E308 750C             JNZ      0049E316 ;☆■
0167:0049E30A C78548FFFFFF0100+MOV      DWORD [EBP+FFFFFF48],01 ;**
0167:0049E314 EB0A             JMP      SHORT 0049E320 ;☆◆
0167:0049E316 C78548FFFFFF0000+MOV      DWORD [EBP+FFFFFF48],00 ;☆■
;**
;☆◆
0167:0049E320 8B55CC           MOV      EDX,[EBP-34] ;tempString取出的两个字符
0167:0049E323 52               PUSH     EDX
0167:0049E324 FF1528124000     CALL     MSVBVM60!__vbaR8Str
0167:0049E32A DC1DF0434000     FCOMP    QWORD [004043F0]
0167:0049E330 DFE0             FNSTSW   AX
0167:0049E332 F6C441           TEST     AH,41
0167:0049E335 740C             JZ       0049E343 ;☆●
0167:0049E337 C78544FFFFFF0100+MOV      DWORD [EBP+FFFFFF44],01 ;**
0167:0049E341 EB0A             JMP      SHORT 0049E34D ;☆★
0167:0049E343 C78544FFFFFF0000+MOV      DWORD [EBP+FFFFFF44],00 ;☆●
;**
;☆★
0167:0049E34D 8B45CC           MOV      EAX,[EBP-34] ;tempString取出的两个字符
0167:0049E350 50               PUSH     EAX
0167:0049E351 FF1528124000     CALL     MSVBVM60!__vbaR8Str
0167:0049E357 DC1DE8434000     FCOMP    QWORD [004043E8]
0167:0049E35D DFE0             FNSTSW   AX
0167:0049E35F F6C401           TEST     AH,01
0167:0049E362 750C             JNZ      0049E370 ;☆※
0167:0049E364 C78540FFFFFF0100+MOV      DWORD [EBP+FFFFFF40],01 ;**
0167:0049E36E EB0A             JMP      SHORT 0049E37A ;☆◎
0167:0049E370 C78540FFFFFF0000+MOV      DWORD [EBP+FFFFFF40],00 ;☆※
;**E
;☆◎
0167:0049E37A 8B8D4CFFFFFF     MOV      ECX,[EBP+FFFFFF4C] ;??
0167:0049E380 0B8D48FFFFFF     OR       ECX,[EBP+FFFFFF48] ;??
0167:0049E386 F7D9             NEG      ECX ;将操作数按位求反后末位加1
0167:0049E388 1BC9             SBB      ECX,ECX ;带借位减
0167:0049E38A F7D9             NEG      ECX
0167:0049E38C 8B9544FFFFFF     MOV      EDX,[EBP+FFFFFF44]
0167:0049E392 0B9540FFFFFF     OR       EDX,[EBP+FFFFFF40]
0167:0049E398 F7DA             NEG      EDX
0167:0049E39A 1BD2             SBB      EDX,EDX
0167:0049E39C F7DA             NEG      EDX
0167:0049E39E 23CA             AND      ECX,EDX
0167:0049E3A0 85C9             TEST     ECX,ECX
0167:0049E3A2 0F859B000000     JNZ      NEAR 0049E443
;********************************************************************************
;不跳，就把这个字符串转为整数，再取其对应的字符，最后化大写～终于知道了
0167:0049E3A8 C745FC12000000   MOV      DWORD [EBP-04],12
0167:0049E3AF FF1558124000     CALL     MSVBVM60!rtcErrObj
0167:0049E3B5 8945BC           MOV      [EBP-44],EAX
0167:0049E3B8 C745B409000000   MOV      DWORD [EBP-4C],09
0167:0049E3BF 8D45B4           LEA      EAX,[EBP-4C]
0167:0049E3C2 50               PUSH     EAX
0167:0049E3C3 FF1508114000     CALL     MSVBVM60!__vbaBoolVarNull
0167:0049E3C9 66894580         MOV      [EBP-80],AX
0167:0049E3CD 8D4DB4           LEA      ECX,[EBP-4C]
0167:0049E3D0 FF1530104000     CALL     MSVBVM60!__vbaFreeVar
0167:0049E3D6 0FBF4D80         MOVSX    ECX,WORD [EBP-80]
0167:0049E3DA 85C9             TEST     ECX,ECX
0167:0049E3DC 7413             JZ       0049E3F1
0167:0049E3DE C745FC13000000   MOV      DWORD [EBP-04],13
0167:0049E3E5 8B55CC           MOV      EDX,[EBP-34]
0167:0049E3E8 8D4DCC           LEA      ECX,[EBP-34]
0167:0049E3EB FF1540124000     CALL     MSVBVM60!__vbaStrCopy
0167:0049E3F1 C745FC15000000   MOV      DWORD [EBP-04],15
0167:0049E3F8 8B55CC           MOV      EDX,[EBP-34] ;操作对象
0167:0049E3FB 52               PUSH     EDX
0167:0049E3FC FF1544124000     CALL     MSVBVM60!__vbaI4Str ;整数取对应字符
0167:0049E402 50               PUSH     EAX
0167:0049E403 8D45B4           LEA      EAX,[EBP-4C]
0167:0049E406 50               PUSH     EAX
0167:0049E407 FF15DC114000     CALL     MSVBVM60!rtcVarBstrFromAnsi
0167:0049E40D 8D4DB4           LEA      ECX,[EBP-4C]
0167:0049E410 51               PUSH     ECX
0167:0049E411 8D55A4           LEA      EDX,[EBP-5C]
0167:0049E414 52               PUSH     EDX
0167:0049E415 FF1538114000     CALL     MSVBVM60!rtcUpperCaseVar ;大写
0167:0049E41B 8D45A4           LEA      EAX,[EBP-5C]
0167:0049E41E 50               PUSH     EAX
0167:0049E41F FF1538104000     CALL     MSVBVM60!__vbaStrVarMove
0167:0049E425 8BD0             MOV      EDX,EAX
0167:0049E427 8D4DCC           LEA      ECX,[EBP-34]
0167:0049E42A FF15B4124000     CALL     MSVBVM60!__vbaStrMove
0167:0049E430 8D4DA4           LEA      ECX,[EBP-5C]
0167:0049E433 51               PUSH     ECX
0167:0049E434 8D55B4           LEA      EDX,[EBP-4C]
0167:0049E437 52               PUSH     EDX
0167:0049E438 6A02             PUSH     BYTE +02
0167:0049E43A FF1548104000     CALL     MSVBVM60!__vbaFreeVarList
0167:0049E440 83C40C           ADD      ESP,BYTE +0C
;********************************************************************************
0167:0049E443 C745FC17000000   MOV      DWORD [EBP-04],17 ;**
0167:0049E44A 8B45D0           MOV      EAX,[EBP-30] ;00
0167:0049E44D 50               PUSH     EAX
0167:0049E44E 8B4DCC           MOV      ECX,[EBP-34] ;从tempString取出的或经过计算了的
0167:0049E451 51               PUSH     ECX
0167:0049E452 FF157C104000     CALL     MSVBVM60!__vbaStrCat ;这里连接的就是注册码
0167:0049E458 8BD0             MOV      EDX,EAX
0167:0049E45A 8D4DD0           LEA      ECX,[EBP-30]
0167:0049E45D FF15B4124000     CALL     MSVBVM60!__vbaStrMove
0167:0049E463 C745FC18000000   MOV      DWORD [EBP-04],18
0167:0049E46A E933FDFFFF       JMP 0049E1A2
;第二个循环完
;///////////////////////////////////////////////////////////////////////////
```

## 总结

算法不难，见上。我希望表达清楚了……

## 注册机

```asm
;KeyGen.asm
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
; The KeyGen by lq7972,with MASM32 V8
; E-mail:bruceyu13@sina.com
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

.386
.model flat,stdcall
option casemap:none

;Include文件定义
include windows.inc
include user32.inc
includelib user32.lib
include kernel32.inc
includelib kernel32.lib
include gdi32.inc
includelib gdi32.lib
include comdlg32.inc
includelib comdlg32.lib
include masm32.inc
includelib masm32.lib

;Equ等值定义
ICO_MAIN equ 1000H
DLG_MAIN equ 1
EditName equ 10
EditSN equ 11
;********************************************************************************

.data?
hInstance dd ?
szSN            db 255 dup (?)
szName db 255 dup (?)
regStr db 255 dup (?)
theString db 8   dup (?)
lenString db 8   dup (?)

.data
szErr db '请输入用户名！',0
szCaption db '错误！',0

.const
Num01 dq 64.00000000000000
Num02 dq 90.00000000000000
Num03 dq 97.00000000000000
Num04 dq 123.0000000000000
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

.code
;********************************************************************************

_testStr proc uses ebx edi esi ebp, @tempStr
local @tempStr00
local @temp01, @temp02, @temp03, @temp04
invoke atodw, @tempStr
mov dword ptr @tempStr00, eax
xor eax, eax
fild @tempStr00
fcomp Num01
fnstsw ax
test ah,41h
jz @F
mov dword ptr @temp01, 01
jmp a10
@@:
mov dword ptr @temp01, 00
a10:
xor eax, eax
fild @tempStr00
fcomp Num02
fnstsw ax
test ah, 1
jnz @F
mov dword ptr @temp02, 01
jmp a20
@@:
mov dword ptr @temp02, 00
a20:
xor eax, eax
```
