
# 反汇编分析

```asm
mov eax, dword ptr [eax+00000314]

:00606414 8B10

             mov edx, dword ptr [eax]

:00606416

FF9248020000            call dword ptr [edx+00000248]

:0060641C

A1481B6100              mov eax, dword ptr

[00611B48]

:00606421 8B00

     mov eax, dword ptr [eax]

:00606423 8B8014030000

         mov eax, dword ptr [eax+00000314]

:00606429

E85EDEEBFF              call 004C428C
```
\*
Possible StringData Ref from Code Obj ->"注册成功！请重启本软件！"
       ====>呵呵，胜利女神！

```asm
:0060642E

B8BC646000              mov eax, 006064BC

:00606433

E88460E4FF              call 0044C4BC

:00606438

A1D4216100              mov eax, dword ptr

[006121D4]

:0060643D 8B00

     mov eax, dword ptr [eax]

:0060643F E8A8AEE6FF

         call 004712EC

:00606444 EB0A

               jmp 00606450
```
\*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0060633F(C)
|
\*
Possible StringData Ref from Code Obj ->"无效注册码！"
     ====>BAD BOY！

```asm
:00606446

B8E0646000              mov eax, 006064E0

:0060644B

E86C60E4FF              call 0044C4BC
```
\*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00606444(U)
|
```asm
:00606450

33C0                    xor
eax, eax

:00606452 5A

     pop edx

:00606453 59

           pop ecx

:00606454 59

                 pop ecx

:00606455

648910                  mov dword
ptr fs:[eax], edx

:00606458 6875646000
 push 00606475
```
\*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00606473(U)
|
```asm
:0060645D

8D45F0                  lea eax,
dword ptr [ebp-10]

:00606460 E827E8DFFF
   call 00404C8C

:00606465 8D45FC
       lea eax, dword ptr [ebp-04]

:00606468 E81FE8DFFF
             call 00404C8C

:0060646D C3
                     ret
```
―――――――――――――――――――――――――――――――――

## 算法调用

进入算法`CALL:006062E4   call 0057FD98`

\*
Referenced by a CALL at Address:
|:006062E4
|
```asm
:0057FD98 55
                   push ebp

:0057FD99

8BEC                    mov
ebp, esp

:0057FD9B 33C9
   xor ecx, ecx

:0057FD9D 51
         push ecx

:0057FD9E 51
               push ecx

:0057FD9F 51
                     push
ecx

:0057FDA0 51
   push ecx

:0057FDA1 51
         push ecx

:0057FDA2 51
               push ecx

:0057FDA3 51
                     push
ecx

:0057FDA4 53
   push ebx

:0057FDA5 56
         push esi

:0057FDA6 57
               push edi

:0057FDA7 8BD8
                   mov ebx,
eax

:0057FDA9 33C0
   xor eax, eax

:0057FDAB 55
         push ebp

:0057FDAC 68A9005800
         push 005800A9

:0057FDB1 64FF30
             push dword ptr fs:[eax]

:0057FDB4

648920                  mov dword
ptr fs:[eax], esp

:0057FDB7 8D45EC
     lea eax, dword ptr [ebp-14]

:0057FDBA 8B13
               mov edx, dword ptr [ebx]
         ====>EDX=85KA089DJ012SL    用户号

:0057FDBC

E8634FE8FF              call 00404D24

:0057FDC1

8D45F8                  lea eax,
dword ptr [ebp-08]

:0057FDC4 E8C34EE8FF
   call 00404C8C

:0057FDC9 8B45EC
       mov eax, dword ptr [ebp-14]

:0057FDCC E87351E8FF
             call 00404F44
     ====>取用户号长度

:0057FDD1

8BD8                    mov
ebx, eax
             ====>EBX=EAX=C

:0057FDD3

85DB                    test
ebx, ebx

:0057FDD5 0F8ED2000000            jle
0057FEAD

:0057FDDB BE01000000              mov
esi, 00000001
```
\* Referenced
by a (U)nconditional or (C)onditional Jump at Address:
|:0057FEA7(C)
|
```asm
:0057FDE0

8D45FC                  lea eax,
dword ptr [ebp-04]

:0057FDE3 50
       push eax

:0057FDE4 B901000000
       mov ecx, 00000001

:0057FDE9 8BD6
             mov edx, esi

:0057FDEB 8B45EC
                 mov eax, dword ptr
[ebp-14]

:0057FDEE E8A953E8FF              call
0040519C

:0057FDF3 8B45FC
   mov eax, dword ptr [ebp-04]

:0057FDF6 BAC4005800
         mov edx, 005800C4

:0057FDFB E88852E8FF
           call 00405088

:0057FE00 0F8487000000
           je 0057FE8D

:0057FE06 8B45FC
               mov eax, dword ptr [ebp-04]

:0057FE09

BAD0005800              mov edx, 005800D0

:0057FE0E

E87552E8FF              call 00405088

:0057FE13

7478                    je 0057FE8D

:0057FE15

8B45FC                  mov eax,
dword ptr [ebp-04]

:0057FE18 BADC005800
   mov edx, 005800DC

:0057FE1D E86652E8FF
     call 00405088

:0057FE22 7469
           je 0057FE8D

:0057FE24 8B45FC
               mov eax, dword ptr [ebp-04]

:0057FE27

BAE8005800              mov edx, 005800E8

:0057FE2C

E85752E8FF              call 00405088

:0057FE31

745A                    je 0057FE8D

:0057FE33

8B45FC                  mov eax,
dword ptr [ebp-04]

:0057FE36 BAE8005800
   mov edx, 005800E8

:0057FE3B E84852E8FF
     call 00405088

:0057FE40 744B
           je 0057FE8D

:0057FE42 8B45FC
               mov eax, dword ptr [ebp-04]

:0057FE45

BAF4005800              mov edx, 005800F4

:0057FE4A

E83952E8FF              call 00405088

:0057FE4F

743C                    je 0057FE8D

:0057FE51

8B45FC                  mov eax,
dword ptr [ebp-04]

:0057FE54 BA00015800
   mov edx, 00580100

:0057FE59 E82A52E8FF
     call 00405088

:0057FE5E 742D
           je 0057FE8D

:0057FE60 8B45FC
               mov eax, dword ptr [ebp-04]

:0057FE63

BA0C015800              mov edx, 0058010C

:0057FE68

E81B52E8FF              call 00405088

:0057FE6D

741E                    je 0057FE8D

:0057FE6F

8B45FC                  mov eax,
dword ptr [ebp-04]

:0057FE72 BA18015800
   mov edx, 00580118

:0057FE77 E80C52E8FF
     call 00405088

:0057FE7C 740F
           je 0057FE8D

:0057FE7E 8B45FC
               mov eax, dword ptr [ebp-04]

:0057FE81

BA24015800              mov edx, 00580124

:0057FE86

E8FD51E8FF              call 00405088

:0057FE8B

7518                    jne
0057FEA5
```
\* Referenced
by a (U)nconditional or (C)onditional Jump at Addresses:
|:0057FE00(C), :0057FE13(C),
:0057FE22(C), :0057FE31(C), :0057FE40(C)
|:0057FE4F(C), :0057FE5E(C), :0057FE6D(C),
:0057FE7C(C)
|
```asm
:0057FE8D 8D45F8
     lea eax, dword ptr [ebp-08]

:0057FE90 8B55FC
             mov edx, dword ptr [ebp-04]

:0057FE93

E8B450E8FF              call 00404F4C

:0057FE98

8B45F8                  mov eax,
dword ptr [ebp-08]

:0057FE9B E8A450E8FF
   call 00404F44

:0057FEA0 83F806
       cmp eax, 00000006

:0057FEA3 7408
             je 0057FEAD
```
\*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0057FE8B(C)
|
```asm
:0057FEA5

46                      inc
esi

:0057FEA6 4B
   dec ebx

:0057FEA7 0F8533FFFFFF
 jne 0057FDE0
                 ====>呵呵，这循环其实是正序取用户号中大于0的数字！
```
\*
Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:0057FDD5(C),
:0057FEA3(C)
|
```asm
:0057FEAD 837DF800
   cmp dword ptr [ebp-08], 00000000
 ====>[ebp-08]=858912

:0057FEB1

740C                    je 0057FEBF

:0057FEB3

8B45F8                  mov eax,
dword ptr [ebp-08]
                 ====>EAX=[ebp-08]=858912

:0057FEB6

E8799FE8FF              call 00409E34
       ====>取858912的16进制值

:0057FEBB

8BF8                    mov
edi, eax
             ====>EDI=000D1B20（H）=858912（D）

:0057FEBD

EB05                    jmp
0057FEC4
```
\* Referenced
by a (U)nconditional or (C)onditional Jump at Address:
|:0057FEB1(C)
|
```asm
:0057FEBF

BFB19C0800              mov edi, 00089CB1
```
\*
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0057FEBD(U)
|
```asm
:0057FEC4

8D45FC                  lea eax,
dword ptr [ebp-04]

:0057FEC7 E8C04DE8FF
   call 00404C8C

:0057FECC 8B45F8
       mov eax, dword ptr [ebp-08]

:0057FECF E87050E8FF
             call 00404F44

:0057FED4 8BD8
                   mov ebx,
eax

:0057FED6 4B
   dec ebx

:0057FED7 85DB
       test ebx, ebx

:0057FED9 7C2F
             jl 0057FF0A

:0057FEDB 43
                   inc ebx

:0057FEDC

33F6                    xor
esi, esi
```
\* Referenced
by a (U)nconditional or (C)onditional Jump at Address:
|:0057FF08(C)
|
```asm
:0057FEDE

8D45E8                  lea eax,
dword ptr [ebp-18]

:0057FEE1 50
       push eax

:0057FEE2 8B45F8
```
