
|:0040EBEE(C)

|:0040EA4C 8B4D84
mov ecx, dword ptr [ebp-7C]    <====取V5

|:0040EA4F 41
inc ecx
<====V5+1

|:0040EA50 8B8570FFFFFF
mov eax, dword ptr [ebp+FFFFFF70]    取NEWBIT

|:0040EA56 33D2
xor edx, edx
|:0040EA58 F7F1
div ecx    <====NEWBIT MOD (V5+1)

|:0040EA5A 8BCA
mov ecx, edx

|:0040EA5C
8B4580                  mov eax,
dword ptr [ebp-80]    <====取V6

|:0040EA5F 40
inc eax    <====V6+1

|:0040EA60 50
push eax

|:0040EA61 8B8570FFFFFF
mov eax, dword ptr [ebp+FFFFFF70]    <====取NEWBIT
|:0040EA67 5A
pop edx

|:0040EA68 8BDA
mov ebx, edx

|:0040EA6A 33D2
xor edx, edx

|:0040EA6C F7F3
div ebx
<====NEWBIT MOD (V6+1)

|:0040EA6E 03CA
add ecx, edx    <====(NEWBIT MOD (V5+1))+(NEWBIT MOD (V6+1))

|:0040EA70 8B857CFFFFFF
mov eax, dword ptr [ebp+FFFFFF7C]    <====取V7
|:0040EA76 40
inc eax    <====V7+1

|:0040EA77 50
push eax

|:0040EA78
8B8570FFFFFF            mov eax, dword ptr [ebp+FFFFFF70]
<====取NEWBIT

|:0040EA7E 5A
pop edx

|:0040EA7F 8BDA
mov ebx, edx

|:0040EA81 33D2
xor edx, edx
|:0040EA83 F7F3
div ebx    <====NEWBIT MOD (V7+1)

|:0040EA85 03CA
add ecx, edx
<====(NEWBIT MOD (V5+1))+(NEWBIT MOD (V6+1))+(NEWBIT MOD (V7+1))

|:0040EA87
8BC1                    mov
eax, ecx

|:0040EA89 8B9578FFFFFF
mov edx, dword ptr [ebp+FFFFFF78]    <====取V8

|:0040EA8F 42
inc edx
<====V8+1

|:0040EA90 8BCA
mov ecx, edx

|:0040EA92 33D2
xor edx, edx

|:0040EA94 F7F1
div ecx
<====((NEWBIT MOD (V5+1))+(NEWBIT MOD (V6+1))+(NEWBIT MOD (V7+1)))  MOD (V8+1)

|:0040EA96 895588
mov dword ptr [ebp-78], edx    <====保存结果(即计算出来的注册码)

|:0040EA99
83458402                add dword ptr [ebp-7C], 00000002    <====V5:=V5+2

|:0040EA9D 836D8007
sub dword ptr [ebp-80], 00000007
<====V6:=V6-7

|:0040EAA1 83857CFFFFFF05
add dword ptr [ebp+FFFFFF7C], 00000005    <====V7:=V7+5
|:0040EAA8 83AD78FFFFFF03          sub dword ptr [ebp+FFFFFF78], 00000003    <====V8:=V8-3

|:0040EAAF 8B4588
mov eax, dword ptr [ebp-78]

|:0040EAB2
83F830                  cmp eax, 00000030

|:0040EAB5 7C08
jl 0040EABF

|:0040EAB7 8B4588
mov eax, dword ptr [ebp-78]

|:0040EABA 83F839
cmp eax, 00000039
|:0040EABD 7E20
jle 0040EADF    <====注册码是不是数字?

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040EAB5(C)

|:0040EABF 8B5588
mov edx, dword ptr [ebp-78]

|:0040EAC2 83FA41
cmp edx, 00000041

|:0040EAC5 7C08
jl 0040EACF

|:0040EAC7 8B4D88
mov ecx, dword ptr [ebp-78]

|:0040EACA
83F95A                  cmp ecx, 0000005A

|:0040EACD 7E10
jle 0040EADF    <====注册码是不是大写字母?

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040EAC5(C)

|:0040EACF 8B4588
mov eax, dword ptr [ebp-78]

|:0040EAD2 83F861
cmp eax, 00000061

|:0040EAD5 7C63
jl 0040EB3A
|:0040EAD7 8B5588
mov edx, dword ptr [ebp-78]

|:0040EADA 83FA7A
cmp edx, 0000007A

|:0040EADD 7F5B
jg 0040EB3A
<====注册码是不是小写

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0040EABD(C), :0040EACD(C)

|:0040EADF 66C745B86800
mov [ebp-48], 0068

|:0040EAE5 8D45D8
lea eax, dword ptr [ebp-28]

|:0040EAE8 E8AB35FFFF              call 00402098

|:0040EAED 50
push eax

|:0040EAEE FF45C4
inc [ebp-3C]

|:0040EAF1 8A5588
mov dl, byte ptr [ebp-78]
|:0040EAF4 8D45DC
lea eax, dword ptr [ebp-24]

|:0040EAF7 E8800C2B00
call 006BF77C

|:0040EAFC 8BD0
mov edx, eax

|:0040EAFE FF45C4
inc [ebp-3C]

|:0040EB01
8D45F8                  lea eax, dword ptr [ebp-08]

|:0040EB04 59
pop ecx

|:0040EB05 E8960E2B00
call 006BF9A0

|:0040EB0A 8D55D8
lea edx, dword ptr [ebp-28]
|:0040EB0D 8D45F8
lea eax, dword ptr [ebp-08]

|:0040EB10 E8630E2B00
call 006BF978

|:0040EB15 FF4DC4
dec [ebp-3C]

|:0040EB18 8D45D8
lea eax, dword ptr [ebp-28]
|:0040EB1B BA02000000              mov edx, 00000002

|:0040EB20 E8230E2B00
call 006BF948

|:0040EB25 FF4DC4
dec [ebp-3C]

|:0040EB28 8D45DC
lea eax, dword ptr [ebp-24]

|:0040EB2B BA02000000
mov edx, 00000002

|:0040EB30 E8130E2B00
call 006BF948

|:0040EB35 E9BA000000
jmp 0040EBF4

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  <====6次循环后不符合条件则跳到这里
|:0040EAD5(C), :0040EADD(C)

|:0040EB3A 8B4DA0
mov ecx, dword ptr [ebp-60]

|:0040EB3D
83F905                  cmp ecx, 00000005

|:0040EB40 0F859F000000
jne 0040EBE5

|:0040EB46 8B4588
mov eax, dword ptr [ebp-78]

|:0040EB49 83F83D
cmp eax, 0000003D    <====比较结果是不是大于等于3D

|:0040EB4C 7D13
jge 0040EB61    <====是则跳

|:0040EB4E 8B4588
mov eax, dword ptr [ebp-78]
<====取运算结果REGCODEBIT

|:0040EB51 B90A000000
mov ecx, 0000000A

|:0040EB56 99
cdq

|:0040EB57 F7F9
idiv ecx
<====REGCODEBIT MOD 0A

|:0040EB59 83C230
add edx, 00000030    <====(REGCODEBIT MOD 0A)+30

|:0040EB5C 895588
mov dword ptr [ebp-78], edx    <====保存注册码

|:0040EB5F
EB2C                    jmp 0040EB8D

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040EB4C(C)

|:0040EB61 8B4588
mov eax, dword ptr [ebp-78]

|:0040EB64
83F85D                  cmp eax, 0000005D    <====比较结果是不是小于等于5D

|:0040EB67 7E13
jle 0040EB7C    <====是则跳

|:0040EB69 8B4588
mov eax, dword ptr [ebp-78]    <====取运算结果REGCODEBIT

|:0040EB6C
B91A000000              mov ecx, 0000001A
|:0040EB71 99
cdq

|:0040EB72 F7F9
idiv ecx    <====REGCODEBIT MOD 1A

|:0040EB74
83C261                  add edx, 00000061    <====(REGCODEBIT MOD 1A)+61

|:0040EB77 895588
mov dword ptr [ebp-78], edx    <====保存注册码

|:0040EB7A EB11
jmp 0040EB8D

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040EB67(C)

|:0040EB7C 8B4588
mov eax, dword ptr [ebp-78]    <====取运算结果REGCODEBIT

|:0040EB7F
B91A000000              mov ecx, 0000001A
|:0040EB84 99
cdq

|:0040EB85 F7F9
idiv ecx    <====REGCODEBIT MOD 1A

|:0040EB87
83C241                  add edx, 00000041    <====(REGCODEBIT MOD 1A)+41

|:0040EB8A 895588
mov dword ptr [ebp-78], edx    <====保存注册码

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0040EB5F(U), :0040EB7A(U)

|:0040EB8D 66C745B87400            mov [ebp-48], 0074

|:0040EB93 8D45D0
lea eax, dword ptr [ebp-30]

|:0040EB96 E8FD34FFFF
call 00402098

|:0040EB9B 50
push eax

|:0040EB9C
FF45C4                  inc [ebp-3C]

|:0040EB9F 8A5588
mov dl, byte ptr [ebp-78]

|:0040EBA2 8D45D4
lea eax, dword ptr [ebp-2C]

|:0040EBA5 E8D20B2B00
call 006BF77C

|:0040EBAA 8BD0
mov edx, eax
|:0040EBAC FF45C4
inc [ebp-3C]

|:0040EBAF 8D45F8
lea eax, dword ptr [ebp-08]

|:0040EBB2 59
pop ecx

|:0040EBB3
E8E80D2B00              call 006BF9A0
|:0040EBB8 8D55D0
lea edx, dword ptr [ebp-30]

|:0040EBBB 8D45F8
lea eax, dword ptr [ebp-08]

|:0040EBBE E8B50D2B00
call 006BF978

|:0040EBC3 FF4DC4
dec [ebp-3C]

|:0040EBC6
8D45D0                  lea eax, dword ptr [ebp-30]

|:0040EBC9 BA02000000
mov edx, 00000002

|:0040EBCE E8750D2B00
call 006BF948

|:0040EBD3 FF4DC4
dec [ebp-3C]

|:0040EBD6 8D45D4
lea eax, dword ptr [ebp-2C]

|:0040EBD9 BA02000000              mov edx, 00000002

|:0040EBDE E8650D2B00
call 006BF948
