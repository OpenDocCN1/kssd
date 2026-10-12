
# 我自己写的个虚拟机

> 原文：[`www.pediy.com/kssd/pediy12/97440.html`](https://www.pediy.com/kssd/pediy12/97440.html)

想申个精，这是我昨天写了一晚上写的虚拟机，模拟了一个调用 `MessageBox` 的过程。实现的指令不算多。支持立即数和 `[]` 这种取地址的方式。大家直接看程序吧。

对不起，10 分钟前发布的版本有点问题，这个已经解决了 `masm32` 直接编译 `ml /c /coff` 链接的时候 `link /subsystem:windows /section:.text,RWE` 完全可以运行: :)

```asm
.586
.model flat, stdcall
option casemap: none
include windows.inc
include user32.inc
includelib user32.lib
include kernel32.inc
includelib kernel32.lib

.data
chCaption db 'Message', 0
chText db 'oh yeah!', 0
chUser32 db 'user32.dll', 0

chSrc db 0h, 08h ;这条相当于 x86 的 push 0
dd 0h
db 7h, 0C0h ;相当于 x86 的 mov eax, chCaption
dd chCaption
db 0h, 00h ;相当于 x86 的 push chText
db 0h, 08h
dd chText
db 0h, 08h ;相当于 x86 的 push 0
dd 0h
db 2h, 08h ;call MessageBox
dd 77d507eah
db 3h, 0h ;ret 虚拟机退出

.code
opcode dd ?

start:
call here

here:
pop ebx
mov eax, here
sub ebx, eax
lea eax, [chSrc + ebx]
mov opcode, eax
;指定 opcode
invoke LoadLibrary, offset chUser32
push eax
push esp
push ebp
push esi
push edi
push ebx
push ecx
push edx
pushfd
mov esi, opcode
lea edi, [esp + 20h]
lea ebp, [ASM + ebx]
jmp Dispatch
db 0CCh

VNOT:
pop eax
not dword ptr [eax]
pushfd
pop [edi - 20h]
jmp Dispatch

VNEG:
pop eax
neg dword ptr [eax]
pushfd
pop [edi - 20h]
jmp Dispatch

VJMP:
pop eax
add esi, [eax]
jmp Dispatch

VCALL:
pop ecx
mov ecx, [ecx]
call ecx
mov [edi], eax
jmp Dispatch
db 0CCh
;以上是初始化工作，从这开始进入分配器

Dispatch:
xor eax, eax ;reset eax
mov al, [esi]
inc esi
mov edx, [ebp + 4*eax] ;reset edx
mov al, [esi]
inc esi
test al, 80h ;;;比较单双操作数
jnz Double

test al, 40h ;这里开始是对单操作数指令的处理
jnz SingalAddr
cmp al, 08h
jz ImmiValue
lea ecx, [edi + 4*eax] ;push eax
push ecx ;操作寄存器
jmp edx

ImmiValue:
push esi ;push immi
add esi, 4
jmp edx

SingalAddr:
cmp al, 08h
jz ImmiAddr
mov ecx, [edi + 4*eax] ;push [eax]
push ecx ;操作寄存器中的地址
jmp edx

ImmiAddr:
mov eax, [esi] ;push [immi]
push eax
add esi, 4
jmp edx

Double: ;这里开始是对双操作数指令的处理
xor ecx, ecx
and al, 7Fh
test al, 40h
jnz HaveImmi

and al, 40h
mov cl, [esi]
inc esi
test cl, 80h
jnz FValue

lea eax, [edi + 4*eax] ;mov eax, ecx
push eax
lea ecx, [edi + 4*ecx]
push ecx
jmp edx

FValue:
test cl, 40h ;mov eax, [ecx]
jnz BValue
lea eax, [edi + 4*eax]
push eax
mov ecx, [edi + 4*ecx]
push ecx
jmp edx

BValue:
mov eax, [edi + 4*eax] ;mov [eax], ecx
push eax
mov ecx, [edi + 4*ecx]
push ecx
jmp edx

HaveImmi:
and al, 0BFh
test al, 20h
jnz HaveAddr
lea ecx, [edi + 4*eax]
push ecx ;mov eax, immi
push esi
add esi, 4
jmp edx

HaveAddr:
test al, 10h
jnz ImmiBValue
and al, 0Fh ;mov eax, [immi]
lea ecx, [edi + 4*eax]
push ecx
push [esi]
add esi, 4
jmp edx

ImmiBValue:
and al, 0EFh ;mov [eax], immi
mov ecx, [edi + 4*eax]
push ecx
push esi
add esi, 4
jmp edx

jmp edx
db 0CCh ;以上都是分配器程序

VMOV:
pop eax
pop ecx
mov eax, [eax]
mov [ecx], eax
jmp Dispatch

VLEA:
pop eax
pop ecx
mov [ecx], eax
jmp Dispatch

VRETN:
mov al, [esi]
lea esp, [edi + 8h]
mov edx, [esp - 4]
lea esp, [esp + eax]
jmp edx

VPUSH:
pop eax
push [eax]
jmp Dispatch

VPOP:
pop eax
pop [eax]
jmp Dispatch

VADD:
pop eax
pop ecx
add [ecx], eax
jmp Dispatch

VSUB:
pop eax
pop ecx
sub [ecx], eax
jmp Dispatch

ASM dd VPUSH
dd VPOP
dd VCALL
dd VRETN
dd VJMP
dd VNOT
dd VNEG
dd VMOV
dd VLEA
dd VADD
dd VSUB

end start
```
