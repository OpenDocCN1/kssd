
### `sub_401FC6` proc near
; CODE XREF: `sub_401458+58p`
; `CompressPE+122p` ...

```asm
push  esi
push  edi
push  edx
push  ecx
push  eax
mov  eax, ds:dword_40378E
mov  ecx, ds:dword_4037BB
loc_401FD6:        ; CODE XREF: sub_401FC6+1Aj
mov  edx, [eax+0Ch]
cmp  edx, ebx
ja  short loc_401FE2
add  eax, 28h
loop  loc_401FD6
loc_401FE2:        ; CODE XREF: sub_401FC6+15j
sub  eax, 28h
sub  ebx, [eax+0Ch]
add  ebx, [eax+14h]
pop  eax
pop  ecx
pop  edx
pop  edi
pop  esi
retn
```
`sub_401FC6` endp

### `sub_401FF1` proc near
; CODE XREF: `sub_4015C4+16p`

```asm
push  esi
push  edi
push  edx
push  ecx
push  eax
mov  eax, ds:dword_40378E
mov  ecx, ds:dword_4037BB
loc_402001:        ; CODE XREF: sub_401FF1+1Aj
mov  edx, [eax+14h]
cmp  edx, ebx
ja  short loc_40200D
add  eax, 28h
loop  loc_402001
loc_40200D:        ; CODE XREF: sub_401FF1+15j
sub  eax, 28h
sub  ebx, [eax+14h]
add  ebx, [eax+0Ch]
pop  eax
pop  ecx
pop  edx
pop  edi
pop  esi
retn
```
`sub_401FF1` endp

### `loc_40201C`
; DATA XREF: `CompressIt+3Fo`

```asm
mov  eax, [esp+4]
xor  edx, edx
mov  ecx, 64h
mul  ecx
xor  edx, edx
mov  ecx, ds:dword_4038B3
or  ecx, ecx
jz  short loc_40204A
div  ecx
push  0
push  eax
push  402h
push  ds:hProgress
call  SendMessageA
loc_40204A:        ; CODE XREF: pcs1:00402033j
mov  eax, 1
retn
```

### `sub_402050` proc near
; CODE XREF: `CompressPE+23Ep`

```asm
pop  edx
pop  eax
pop  edi
push  edx
mov  ecx, ds:dword_4037BB
dec  ecx
xor  ebp, ebp
loc_40205D:        ; CODE XREF: sub_402050+116j
push  ecx
call  sub_401CF4
jnb  short loc_40206D
xor  ebp, ebp
jz  loc_40215E
loc_40206D:        ; CODE XREF: sub_402050+13j
cmp  ebp, 1
jz  short loc_40207C
mov  ebp, 1
jmp  loc_40215E
loc_40207C:        ; CODE XREF: sub_402050+20j
push  eax
mov  ebx, [eax-14h]
add  ebx, [eax-18h]
add  ebx, ds:lpBaseAddress
push  edi
mov  edi, ebx
call  sub_40216C
mov  ecx, edi
sub  ecx, [eax-14h]
sub  ecx, ds:lpBaseAddress
mov  ds:dword_4038AB, ecx
mov  ebx, edi
pop  edi
pop  eax
push  eax
push  ebx
mov  ebx, [eax+14h]
add  ebx, [eax+10h]
add  ebx, ds:lpBaseAddress
push  edi
mov  edi, ebx
call  sub_40216C
mov  ecx, edi
sub  ecx, [eax+14h]
sub  ecx, ds:lpBaseAddress
mov  ds:dword_4038AF, ecx
pop  edi
mov  ebp, edi
pop  edi
mov  esi, [eax+14h]
add  esi, ds:lpBaseAddress
mov  ecx, ds:dword_4038AF
rep movsb
pop  eax
mov  ebx, [eax+34h]
mov  ecx, [eax-1Ch]
sub  ebx, ecx
mov  [eax-20h], ebx
mov  ebx, ds:dword_4038AB
add  ebx, ds:dword_4038AF
mov  [eax-18h], ebx
mov  edi, ebp
mov  ebx, [eax+0Ch]
add  ebx, ds:@ImageBase
mov  [edi+4], ebx
mov  ebx, [eax-1Ch]
add  ebx, ds:dword_4038AB
add  ebx, ds:@ImageBase
mov  [edi], ebx
mov  ebx, ds:dword_4038AF
mov  [edi+8], ebx
mov  ecx, [eax+34h]
sub  ecx, [eax+0Ch]
sub  ecx, ebx
mov  [edi+0Ch], ecx
add  edi, 10h
pop  ecx
push  ecx
push  eax
inc  ecx
mov  eax, ecx
xor  edx, edx
mov  ecx, 28h
mul  ecx
mov  ecx, eax
pop  eax
mov  esi, eax
add  esi, 28h
push  edi
mov  edi, eax
rep movsb
pop  edi
dec  ds:dword_4037BB
mov  ebp, 1
sub  eax, 28h
loc_40215E:        ; CODE XREF: sub_402050+17j
          ; sub_402050+27j
pop  ecx
add  eax, 28h
loop  loc_402166
jmp  short locret_40216B
loc_402166:        ; CODE XREF: sub_402050+112j
jmp  loc_40205D
locret_40216B:        ; CODE XREF: sub_402050+114j
retn
```
`sub_402050` endp ; sp =  8

### `sub_40216C` proc near
; CODE XREF: `sub_402050+3Cp`
; `sub_402050+67p`

```asm
push  eax
push  edi
std
mov  ecx, 0FFFFFFFh
xor  eax, eax
repe scasb
add  edi, 4
cld
pop  edx
cmp  edx, edi
jnb  short loc_402183
mov  edi, edx
loc_402183:        ; CODE XREF: sub_40216C+13j
pop  eax
retn
```
`sub_40216C` endp

### `sub_402185` proc near
; CODE XREF: `CompressPE+50Ep`

```asm
push  esi
push  edi
push  ecx
push  edx
mov  ecx, 28Ch
push  ecx
push  ecx    ; dwBytes
push  8    ; dwFlags
push  ds:hHeap  ; hHeap
call  HeapAlloc
mov  ds:lpMem, eax
push  0FA000h    ; dwBytes
push  8    ; dwFlags
push  ds:hHeap  ; hHeap
call  HeapAlloc
mov  ds:dword_4038A7, eax
pop  ecx
push  ecx
push  0
push  ds:dword_4038A7
push  ecx
push  ds:lpMem
push  offset loc_403ADD
call  _aP_pack
push  eax
mov  ecx, eax
mov  esi, ds:lpMem
mov  edi, offset loc_403ADD
rep movsb
push  ds:dword_4038A7  ; lpMem
push  0    ; dwFlags
push  ds:hHeap  ; hHeap
call  HeapFree
push  ds:lpMem  ; lpMem
push  0    ; dwFlags
push  ds:hHeap  ; hHeap
call  HeapFree
pop  ecx
pop  eax
sub  eax, ecx
push  eax
mov  edi, offset loc_403ADD
add  edi, ecx
mov  ecx, eax
xor  eax, eax
rep stosb
pop  eax
pop  edx
pop  ecx
pop  edi
pop  esi
retn
```
`sub_402185` endp ; sp = -14h

### `sub_402222` proc near
; CODE XREF: `CompressPE+248p`

```asm
pop  eax
pop  ebx
push  eax
pusha
xchg  ebx, edi
add  edi, 0B0h
loc_40222E:        ; CODE XREF: sub_402222+12j
sub  edi, 10h
cmp  dword ptr [edi], 0
jz  short loc_40222E
xchg  edi, esi
mov  edi, offset dword_40383F
loc_40223D:        ; CODE XREF: sub_402222+2Bj
mov  ecx, 4
rep movsd
sub  esi, 20h
cmp  esi, offset byte_403CC9
jnb  short loc_40223D
mov  esi, offset dword_40383F
mov  edi, offset byte_403CC9
mov  ecx, 0A0h
rep movsb
popa
retn
```
`sub_402222` endp ; sp =  4

### `MakeBackup` proc near
; CODE XREF: `start+3Cp`

```asm
pop  ebx
pop  edi
push  ebx
push  edi
push  edi
push  offset lpNewFileName
call  lstrcpy
pop  edi
push  edi
push  offset szDotBak  ; ".BAK"
push  offset lpNewFileName
call  lstrcat    ; 加入后缀 .Bak
pop  edi
push  0    ; bFailIfExists
push  offset lpNewFileName ; lpNewFileName
push  edi    ; lpExistingFileName
call  CopyFileA
retn
```
`MakeBackup` endp ; sp =  4

### `sub_402291` proc near
; CODE XREF: `sub_4013D7+2Dp`
; `_CheckDlgButton+2p` ...

```asm
pop  edx
pop  ebx
push  edx
mov  esi, offset dword_4038DC
loc_402299:        ; CODE XREF: sub_402291+Fj
lodsd
or  eax, eax
jz  short loc_4022A4
cmp  eax, ebx
jnz  short loc_402299
clc
retn
loc_4022A4:        ; CODE XREF: sub_402291+Bj
stc
retn
```
`sub_402291` endp ; sp =  4

### `sub_4022A6` proc near
; CODE XREF: `ProcessCheckBoxes+24p`
