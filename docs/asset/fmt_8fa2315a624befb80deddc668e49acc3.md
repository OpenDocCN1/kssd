
# 安全卫士注册机（masm6.0）

> 原文：[`www.pediy.com/kssd/pediy04/kanxue290.htm`](https://www.pediy.com/kssd/pediy04/kanxue290.htm)

```asm
.386

.model flat,stdcall

option casemap:none

include \MASM32\INCLUDE\windows.inc
include \MASM32\INCLUDE\user32.inc
include \MASM32\INCLUDE\kernel32.inc
include \MASM32\INCLUDE\comctl32.inc
include \MASM32\INCLUDE\comdlg32.inc
include \MASM32\INCLUDE\masm32.inc

includelib \MASM32\LIB\masm32.lib
includelib \MASM32\LIB\user32.lib
includelib \MASM32\LIB\kernel32.lib
includelib \MASM32\LIB\comctl32.lib
includelib \MASM32\LIB\comdlg32.lib

DLG_MAIN equ 100
IDGEN equ 10
Edit1 equ 11
Edit2 equ 12

_ProcDlgMain PROTO :DWORD,:DWORD,:DWORD,:DWORD
_Math PROTO :DWORD,:DWORD,:DWORD
BlowFish_En PROTO :DWORD,:DWORD
BlowFish_Fun PROTO :DWORD
BlowFish_Init PROTO :DWORD,:DWORD

.data?

hInstance dd ?

.data

szhex dd 0
jqhlen dd 0
szID db 20 dup(0)
szText db 9 dup(0)
fudian1 dd 3fdca138h,406d9168h,415c6944h,42c60000h
zhong dd 0
sn dd 0
tem dd 0
ftem dd 0
ftem1 dd 1
ftem2 dd 0
fa dd 0
fb dd 0
fc dd 0
fd dd 0
sztxt db 20 dup (0)
sztxt1 db 20 dup (0)

.code

gao proc

mov edx,tem
imul edx, 08088405h
inc edx
mov tem,edx
mul edx
mov eax, edx
ret

gao endp

jsaqws proc uses eax ebx ecx edx esi edi

mov eax,szhex
mov ecx,013dh
cdq
idiv ecx
mov tem,edx
mov eax,0104ah
call gao
mov esi,eax
mov eax,016c6h
call gao
add esi,eax
mov zhong,esi
fild dword ptr zhong
lea eax,fudian1
fmul dword ptr [eax]
fstp dword ptr ftem
fild ftem1
fld ftem
fprem
fstp dword ptr ftem2
fld ftem
fld ftem2
fsub
lea eax,fudian1
fmul dword ptr [eax]
fstp dword ptr ftem
fild ftem1
fld ftem
fprem
fstp dword ptr ftem2
fld ftem
fld ftem2
fsub
fist dword ptr fa
finit
mov eax,szhex
mov ecx,011h
cdq
idiv ecx
xor edx,013h
mov zhong,edx
fild dword ptr zhong
lea eax,fudian1
add eax,4
fmul dword ptr [eax]
fstp dword ptr ftem
fild ftem1
fld ftem
fprem
fstp dword ptr ftem2
fld ftem
fld ftem2
fsub
fist dword ptr tem
finit
mov eax,0f77h
call gao
mov esi,eax
mov eax,021e1h
call gao
add esi,eax
mov eax,017h
call gao
xor esi,eax
mov fb,esi
fld dword ptr fudian1
lea eax,fudian1
add eax,4
fadd dword ptr [eax]
add eax,4
fadd dword ptr [eax]
mov eax,szhex
mov ecx,0fh
cdq
idiv ecx
xor edx,0dh
mov ftem,edx
fild dword ptr ftem
faddp st(1), st
fstp dword ptr ftem
fild ftem1
fld ftem
fprem
fstp dword ptr ftem2
fld ftem
fld ftem2
fsub
fist dword ptr tem
finit
mov eax,0f8bh
call gao
mov esi,edx
mov eax,01c69h
call gao
add esi,edx
mov eax,0e59h
call gao
add esi,edx
mov eax,0defh
call gao
add esi,edx
mov eax,02172h
call gao
add esi,edx
mov fc,esi
lea eax,fudian1
add eax,12
fld dword ptr [eax]
lea eax,fudian1
fmul dword ptr [eax]
fstp dword ptr ftem
fild ftem1
fld ftem
fprem
fstp dword ptr ftem2
fld ftem
fld ftem2
fsub
fist dword ptr ftem
mov eax,szhex
mov ecx,ftem
cdq
idiv ecx
mov edi,edx
mov eax,0b4h
call gao
mov zhong,eax
fild zhong
fsin
mov zhong,100
fild zhong
fmul
fstp dword ptr ftem
fild ftem1
fld ftem
fprem
fstp dword ptr ftem2
fld ftem
fld ftem2
fsub
fist dword ptr ftem
mov esi,ftem
finit
mov ftem,edi
fild ftem
lea eax,fudian1
add eax,8
fmul dword ptr [eax]
fstp dword ptr ftem
fild ftem1
fld ftem
fprem
fstp dword ptr ftem2
fld ftem
fld ftem2
fsub
fist dword ptr ftem
add esi,ftem
mov tem,esi
mov eax,0f4fh
call gao
mov esi,eax
mov eax,1dd1h
call gao
add esi,eax
add esi,fa
```
