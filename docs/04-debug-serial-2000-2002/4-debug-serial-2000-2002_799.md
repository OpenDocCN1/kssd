
# FASTNET

> 原文：[`www.pediy.com/kssd/pediy04/kanxue324.htm`](https://www.pediy.com/kssd/pediy04/kanxue324.htm)

How to make a keygen for: 

FASTNET

**Written by tnHuAn**

## Introduction

Well, long time don't write anything. Now I back in this tutorial....You know, I live in VietNam, this is a beautiful country, but in my country, Internet is very expensive, so I must make something to speech my connection. And `FastNet` is one of software I use .... And in this tutorial, I will show you : HOW TO MAKE A KEYGEN FOR `FASTNET`, this is a shareware, a production of micro-team...

## Tools required

- `SoftIce` 4.05 ( I used `SoftIce` 4.05 for `WinNT`)
- `MASM` 6.0 to code a keygen
- Some music, some happy in cracking he he he....

## Target's URL

`http://www.micro-team.com`

## Essay

Now first of all, you must run this software... Then you will see the icon in your system tray, right click on this icon, the choose Registration... You will see your Primary Key, and all you must is : send you primary key to micro-team, and get your Equal Key, then type it into text box, then you will have a registered software. To make a Equal Key, you must pay for micro-team $29.

OK, now input your fake Equal Key, I use `123456789`,

Then I set breakpoint in `SoftIce` : `bpx getdlgitemtextA`, and it break when I click on Register button, now Using `F10`, and trace ....trace... Until you see the second Call ( or the 3rd Call , I don't remember) then trace into this call, and using `F10` to trace... trace... Until you see this code :

```
mov dword ptr [ebp-08],1Fh
mov dword ptr [ebp-04],00h
jmp .....
mov edx, dword ptr [ebp-04]
add edx, 01
mov dword ptr [ebp-04], edx
cmp dword ptr [ebp-04],20h
jge ......
mov eax,[ebp+08]
add eax, dword ptr [ebp-04]
movsx ecx, byte ptr [eax]
mov edx, dword ptr [ebp-08]
sub edx, dword ptr [ebp-04]
mov eax, [ebp+08]
movsx edx,byte ptr [edx+eax]
mov eax, dword ptr [ebp-04]
add eax,ecx
lea eax,[eax+edx+05]
mov ecx, dword ptr [ebp-08]
mov edx, dword ptr [ebp-04]
lea ecx, [ecx+edx+01]
imul eax, ecx
cdq
mov ecx,1Ah
idiv ecx
add edx,41h
mov eax, dword ptr [ebp-04]
.........bla ..bla... ( I can't Remember all)
```

This is all of calculation your Equal Key, to make a keygen, all of you need is code a keygen follow this asm code. Here is source code of my keygen :

```
.386
.model flat, stdcall
option casemap :none ; case sensitive

;------------------------------------------------
include \masm32\include\windows.inc
include \masm32\include\user32.inc
include \masm32\include\kernel32.inc

includelib \masm32\lib\user32.lib
includelib \masm32\lib\kernel32.lib

;------------------------------------------------
WndProc PROTO :DWORD,:DWORD,:DWORD,:DWORD
;------------------------------------------------

.data
include about.inc
Caption db "Keygen for FastNet. Made by tnHuAn/Tres2K",0
moremsg db "Primary key must be 32 character",0
dlgtitle db "Another keygen of tnHuAn/ Tres2K",0
hInstance dd 0
hIcon dd 0

.data?
buffer db 512 dup(?)
buffer2 db 512 dup(?)
ebp04 db 512 dup(?)
ebp08 db 512 dup(?)

.const
DEMO equ 102
IDC_STATIC equ -1
IDC_INPUTBOX equ 1000
IDC_OUTPUTBOX equ 1001
IDC_REGBUTTON equ 1002
IDC_ABOUTBUTTON equ 1003
IDC_CANCELBUTTON equ 1004

.code
start:
; #########################################################################
invoke GetModuleHandle, NULL
mov hInstance, eax
invoke DialogBoxParam,hInstance,DEMO,0,ADDR WndProc,0
invoke ExitProcess,eax

; #########################################################################
WndProc proc hWin :DWORD,
uMsg :DWORD,
wParam :DWORD,
lParam :DWORD

.if uMsg == WM_INITDIALOG
invoke SendMessage,hWin,WM_SETTEXT,0,ADDR dlgtitle
xor eax, eax

.elseif uMsg == WM_COMMAND
.if wParam == IDC_REGBUTTON
push 100h
push offset buffer
push IDC_INPUTBOX
push hWin
call GetDlgItemTextA
cmp eax, 20h
jne @moreletters
call MakeHi
push offset [buffer2]
push IDC_OUTPUTBOX
push hWin
call SetDlgItemTextA

.elseif wParam == IDC_ABOUTBUTTON
push MB_OK
push offset Caption
push offset AboutMsg
push 0
call MessageBoxA

.elseif wParam == IDC_CANCELBUTTON
invoke EndDialog,hWin,0

.endif
.elseif uMsg == WM_CLOSE
invoke EndDialog,hWin,0

.endif
xor eax, eax
ret

@moreletters:
push offset [moremsg]
push IDC_OUTPUTBOX
push hWin
call SetDlgItemTextA
ret

WndProc endp

MakeHi PROC
push ebx
mov ebx,00
mov dword ptr [ebp08],1Fh
mov dword ptr [ebp04],00h
jmp @hehe
@batdau:
mov edx, dword ptr [ebp04]
add edx, 01
mov dword ptr [ebp-04], edx
@hehe:
cmp dword ptr [ebp-04],20h
jge @Ketthuc
mov eax, offset buffer
add eax, dword ptr [ebp04]
movsx ecx, byte ptr [eax]
mov edx, dword ptr [ebp08]
sub edx, dword ptr [ebp04]
mov eax, offset buffer
movsx edx,byte ptr [edx+eax]
mov eax, dword ptr [ebp04]
add eax,ecx
lea eax,[eax+edx+05]
mov ecx, dword ptr [ebp08]
mov edx, dword ptr [ebp04]
lea ecx, [ecx+edx+01]
imul eax, ecx
cdq
mov ecx,1Ah
idiv ecx
add edx,41h
mov eax, dword ptr [ebp04]
mov [buffer2+ebx],dl
inc ebx
jmp @batdau
@Ketthuc:
pop ebx
ret
MakeHi ENDP

end start
```

Well you can code keygen by yourself

## Final Notes

Ok, if you want to comment me something , please email : `tnhuan19@hotmail.com`

My Greetz Goes to:

- All member of `Tres2000`: `MCcodEMaN`, `Sphinx`, ....
- All member of `HAH`, and `Eddile` in `Digital Insight`
- `Acid_Cool_178` ( Crack my crackme#6), `BiSHoP`, `a_evil`, `tKc`,....
