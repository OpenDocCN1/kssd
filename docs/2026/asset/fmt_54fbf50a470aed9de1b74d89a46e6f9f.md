
`pop ebx`

```
:004035EE 83C420
add esp, 00000020
:004035F1 C20800
ret 0008
```

―――――――――――――――――――――――――――――――――

进入 `004035A` `call 00403620`

*
Referenced by a CALL at Address:

|:0040345A
|

```
:00403620 81EC00020000
sub esp, 00000200
:00403626 33C0
xor eax, eax
:00403628
B980000000              mov ecx, 00000080
:0040362D
53                      push
ebx
:0040362E 56
push esi
:0040362F 33DB
xor ebx, ebx
:00403631 57
push edi
:00403632 55
push ebp
:00403633
8D7C2410                lea edi, dword
ptr [esp+10]
:00403637 F3
repz
:00403638 AB
stosd
:00403639 8BB42414020000
mov esi, dword ptr [esp+00000214]
:00403640 56
push esi
*
Reference To: KERNEL32.lstrlenA, Ord:0308h
|
:00403641
FF15F4704000            Call dword ptr [004070F4]
:00403647
3D00020000              cmp eax, 00000200
:0040364C
8BE8                    mov
ebp, eax
:0040364E 7612
jbe 00403662
:00403650 BD00020000
mov ebp, 00000200
:00403655 8D7C2410
lea edi, dword ptr [esp+10]
:00403659 B980000000
mov ecx, 00000080
:0040365E
F3                      repz
:0040365F
A5                      movsd
:00403660
EB0C                    jmp
0040366E
* Referenced
by a (U)nconditional or (C)onditional Jump at Address:
|:0040364E(C)
|
:00403662
8D442410                lea eax, dword
ptr [esp+10]
:00403666 56
push esi
:00403667 50
push eax
*
Reference To: KERNEL32.lstrcpyA, Ord:0302h
|
:00403668
FF15E4704000            Call dword ptr [004070E4]
*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00403660(U)
|
:0040366E
8BC5                    mov
eax, ebp
:00403670 99
cdq
:00403671 83E203
and edx, 00000003
:00403674 03C2
add eax, edx
:00403676 C1F802
sar eax, 02
*
Possible Reference to String Resource ID=00003: "User ID is need,Please Input"
|
:00403679 F7C503000000
test ebp, 00000003
:0040367F 8BF0
mov esi, eax
:00403681 7401
je 00403684
:00403683
46                      inc
esi
* Referenced by
a (U)nconditional or (C)onditional Jump at Address:
|:00403681(C)
|
:00403684
33FF                    xor
edi, edi
:00403686 85ED
test ebp, ebp
:00403688 7E17
jle 004036A1
*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040369F(C)
|
:0040368A
8B443C10                mov eax, dword
ptr [esp+edi+10]
```
====>依次取fly字符的HEX值
1、====>EAX=66
2、====>EAX=6C
3、====>EAX=79
```
:0040368E
50                      push
eax
:0040368F 57
push edi
:00403690 47
inc edi
:00403691 E86AFFFFFF
call 00403600
```
====>对上面的EAX值依次运算得出新值！进入！
```
:00403696
88443C17                mov byte ptr [esp+edi+17],
al
```
====>保存结果
1、
====>AL=33
2、====>AL=B6
3、
====>AL=1E
```
:0040369A
83C408                  add esp,
00000008
:0040369D 3BFD
cmp edi, ebp
:0040369F 7CE9
jl 0040368A
* Referenced by a (U)nconditional
or (C)onditional Jump at Address:
|:00403688(C)
|
:004036A1 33FF
xor edi, edi
:004036A3
85F6                    test
esi, esi
:004036A5 7E1A
jle 004036C1
*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004036BF(C)
|
:004036A7
035CBC10                add ebx, dword
ptr [esp+4*edi+10]
```
====>EBX=00000000
+ 001EB633=001EB633
```
:004036AB
8BC7                    mov
eax, edi
:004036AD 83E01F
and eax, 0000001F
:004036B0 47
inc edi
:004036B1 50
push eax
:004036B2
53                      push
ebx
:004036B3 E818000000              call
004036D0
:004036B8 83C408
add esp, 00000008
:004036BB 8BD8
mov ebx, eax
:004036BD 3BFE
cmp edi, esi
:004036BF
7CE6                    jl 004036A7
*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004036A5(C)
|
:004036C1
8BC3                    mov
eax, ebx
:004036C3 5D
pop ebp
:004036C4 5F
pop edi
:004036C5 5E
pop esi
:004036C6
5B                      pop
ebx
:004036C7 81C400020000            add esp,
00000200
:004036CD C3
ret
```

―――――――――――――――――――――――――――――――――

进入 `00403691`
`call 00403600`

*
Referenced by a CALL at Addresses:
|:004034CE , :00403691
|

```
:00403600
53                      push
ebx
:00403601 8A442408                mov
al, byte ptr [esp+08]
:00403605 8A5C240C
mov bl, byte ptr [esp+0C]
```
====>BL=依次取fly字符的HEX值
*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00403614(C)
|
:00403609
F6C3C3                  test bl,
C3
:0040360C 7A01
jpe 0040360F
:0040360E F9
stc
*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040360C(C)
|
:0040360F
C0DB01                  rcr bl, 01
1、====>BL=66 RCR 1=带进位循环右移256次=33
2、
====>BL=6C RCR 1=带进位循环右移 1 次=B6
3、
====>BL=3C RCR 1=带进位循环右移 2 次=1E
:00403612
FEC8                    dec
al
:00403614 75F3
jne 00403609
====>继续循环？！
:00403616
885C240C                mov byte ptr [esp+0C],
bl
:0040361A 8A44240C                mov
al, byte ptr [esp+0C]
:0040361E 5B
pop ebx
:0040361F C3
ret
```

―――――――――――――――――――――――――――――――――

进入 `0040347C`
`call 004036F0`

对EAX的值进行比较，按条件进行运算，很简单，所以不记录了。只要自己亲自跟踪一下就会明白的！

*
Referenced by a CALL at Addresses:
|:0040347C , :004034DB , :00403534
, :00403588
|

```
:004036F0 8B442404
mov eax, dword ptr [esp+04]
*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00403731(U)
|
:004036F4
83E07F                  and eax,
0000007F
:004036F7 3C41
cmp al, 41
:004036F9 7208
jb 00403703
:004036FB 83F85A
cmp eax, 0000005A
:004036FE 7F03
jg 00403703
:00403700
83C820                  or eax, 00000020
*
Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:004036F9(C),
:004036FE(C)
|
:00403703 83F86F
cmp eax, 0000006F
:00403706 7505
jne 0040370D
:00403708 35FF000000
xor eax, 000000FF
*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00403706(C)
|
:0040370D
83F830                  cmp eax,
00000030
:00403710 7505
jne 00403717
:00403712 35FF000000
xor eax, 000000FF
*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00403710(C)
|
:00403717
83F861                  cmp eax,
00000061
:0040371A 7C05
jl 00403721
:0040371C 83F87A
cmp eax, 0000007A
:0040371F 7E12
jle 00403733
*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040371A(C)
|
:00403721
83F831                  cmp eax,
00000031
:00403724 7C05
jl 0040372B
:00403726 83F839
cmp eax, 00000039
:00403729 7E08
jle 00403733
*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00403724(C)
|
:0040372B
83F03F                  xor eax,
0000003F
:0040372E 83C831
or eax, 00000031
:00403731 EBC1
jmp 004036F4
*
Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0040371F(C),
:00403729(C)
|
:00403733 C3
ret
```

―――――――――――――――――――――――――――――――――

【注册信息保存】：

`Authreg1.exe`启动时找几个DLL、INI文件，或许这个东东只是某个程序的注册部分?

每次启动时都需要重新注册！

―――――――――――――――――――――――――――――――――

【整 理】：

User
ID：fly

注册码 ：3611-2e37-k11s-p141

―――――――――――――――――――――――――――――――――

Cracked By 巢水工作坊――fly [OCN][FCG]

2003-04-17 18:23:07
