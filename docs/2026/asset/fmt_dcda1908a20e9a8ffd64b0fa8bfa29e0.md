
```asm
:00413417 8806

  mov byte ptr [esi], al ----------------->al给[esi]

:00413419 46

  inc esi -------------------------------->esi加一

:0041341A 4B

  dec ebx -------------------------------->ebx减一

:0041341B 75CB

  jne 004133E8 --------------------------->循环；即得新注册码

:0041341D 8A5C2424                mov

bl, byte ptr [esp+24]

:00413421 C6470C00                mov

[edi+0C], 00

:00413425 881F

  mov byte ptr [edi], bl ----------------->将注册码的第一位补到新

    注册码的前面

:00413427 33C0

  xor eax, eax ---------------------------

:00413429 B901000000              mov ecx,

00000001              |

    | 将新注册码累乘

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

  | 值给eax

|:0041343B(C)

    |

|

    |

:0041342E 0FBE1439                movsx

edx, byte ptr [ecx+edi]          |

:00413432 0FAFC1

imul eax, ecx

        |

:00413435 03C2

  add eax, edx

          |

:00413437 41

  inc ecx

              |

:00413438 83F90C

cmp ecx, 0000000C

    |

:0041343B 7CF1

  jl 0041342E ----------------------------

:0041343D 5F

  pop edi

:0041343E 5E

  pop esi

:0041343F 85C0

  test eax, eax -------------------------->检查eax

:00413441 5D

  pop ebp

:00413442 7D02

  jge 00413446 --------------------------->eax大于等于43446就跳

:00413444 F7D8

  neg eax -------------------------------->？？？

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00413442(C)

|

:00413446 99

  cdq ------------------------------------>双字扩展

:00413447 B91A000000              mov ecx,

0000001A ---------------------->ecx=1A

:0041344C F7F9

  idiv ecx ------------------------------->eax=eax/ecx

:0041344E 0FBEC3

movsx eax, bl -------------------------->先符号扩展,再把bl给al

:00413451 5B

  pop ebx

:00413452 83C241

add edx, 00000041 ---------------------->edx=edx+41

:00413455 3BC2

  cmp eax, edx --------------------------->比较eax和edx

:00413457 7512

  jne 0041346B --------------------------->不相等就完蛋

:00413459 8B4C2430                mov

ecx, dword ptr [esp+30] ------------>将新注册码的第一位地址

            给ecx

:0041345D 803932

cmp byte ptr [ecx], 32 ----------------->比较ecx和32

:00413460 7509

  jne 0041346B --------------------------->不相等就完蛋

:00413462 B801000000              mov eax,

00000001

:00413467 83C428

add esp, 00000028

:0041346A C3

  ret

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00413457(C), :00413460(C)

|

:0041346B 33C0

  xor eax, eax

:0041346D 83C428

add esp, 00000028

:00413470 C3

  ret

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
```

```asm
\* Referenced by a CALL at Addresses:

|:0040808A  , :004133CA

|

:004132F0 56

  push esi

:004132F1 8B74240C                mov

esi, dword ptr [esp+0C]

:004132F5 57

  push edi

:004132F6 8BFE

  mov edi, esi

:004132F8 83C9FF

or ecx, FFFFFFFF

:004132FB 33C0

  xor eax, eax

:004132FD F2

  repnz

:004132FE AE

  scasb

:004132FF F7D1

  not ecx

:00413301 49

  dec ecx

:00413302 F6C103

test cl, 03

:00413305 7403

  je 0041330A

:00413307 5F

  pop edi

:00413308 5E

  pop esi

:00413309 C3

  ret

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00413305(C)

|

:0041330A 8A06

  mov al, byte ptr [esi] ------------->依次取注册码到al

:0041330C 84C0

  test al, al ------------------------>检查是否是零

:0041330E 7467

  je 00413377 ------------------------>是就完蛋！

:00413310 8B7C240C                mov

edi, dword ptr [esp+0C]

:00413314 53

  push ebx

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041336A(C)

|

:00413315 50

  push eax

:00413316 46

  inc esi

:00413317 E894FFFFFF              call 004132B0

----------------------->也需要追入！（是换算）

:0041331C 8AD8

  mov bl, al -------------------------->al给bl

:0041331E 8A06

  mov al, byte ptr [esi] -------------->下一个注册码给al

:00413320 50

  push eax

:00413321 46

  inc esi ----------------------------->esi加一

:00413322 E889FFFFFF              call 004132B0

----------------------->换算

:00413327 8AC8

  mov cl, al -------------------------->al给cl

:00413329 240F

  and al, 0F -------------------------->al and 0f

:0041332B C0F904

sar cl, 04 -------------------------->cl算术右移4

:0041332E C0E302

shl bl, 02 -------------------------->bl逻辑左移2

:00413331 02CB

  add cl, bl -------------------------->cl=cl+bl

:00413333 8AD8

  mov bl, al -------------------------->al给bl

:00413335 880F

  mov byte ptr [edi], cl -------------->cl给[edi]

:00413337 8A06

  mov al, byte ptr [esi] -------------->取下一个注册码给al

:00413339 47

  inc edi ----------------------------->edi加一

:0041333A 50

  push eax

:0041333B 46

  inc esi ----------------------------->esi加一

:0041333C E86FFFFFFF              call 004132B0

----------------------->换算

:00413341 8AD0

  mov dl, al -------------------------->al给dl

:00413343 2403

  and al, 03 -------------------------->al and 3

:00413345 C0FA02

sar dl, 02 -------------------------->dl算术右移2

:00413348 C0E304

shl bl, 04 -------------------------->bl逻辑左移4

:0041334B 02D3

  add dl, bl -------------------------->dl=dl+bl

:0041334D 8AD8

  mov bl, al -------------------------->bl给al

:0041334F 8817

  mov byte ptr [edi], dl -------------->dl给[edi]

:00413351 8A06

  mov al, byte ptr [esi] -------------->取下一个注册码给al

:00413353 47

  inc edi ----------------------------->edi加一

:00413354 50

  push eax

:00413355 46

  inc esi ----------------------------->esi加一

:00413356 E855FFFFFF              call 004132B0

----------------------->换算

:0041335B C0E306

shl bl, 06 -------------------------->bl逻辑左移6

:0041335E 02C3

  add al, bl -------------------------->al=al+bl

:00413360 83C410

add esp, 00000010

:00413363 8807

  mov byte ptr [edi], al -------------->al给edi

:00413365 8A06

  mov al, byte ptr [esi] -------------->取下一个注册码给al

:00413367 47

  inc edi ----------------------------->edi加一

:00413368 84C0

  test al, al ------------------------->检查是否取完

:0041336A 75A9

  jne 00413315 ------------------------>没有取完，循环！

:0041336C 5B

  pop ebx

:0041336D 8807

  mov byte ptr [edi], al -------------->将最后一个给[edi]

:0041336F 5F

  pop edi

:00413370 B801000000              mov eax,

00000001

:00413375 5E

  pop esi

:00413376 C3

  ret

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041330E(C)

|

:00413377 8B7C240C                mov

edi, dword ptr [esp+0C]

:0041337B B801000000              mov eax,

00000001

:00413380 C60700

mov byte ptr [edi], 00

:00413383 5F

  pop edi

:00413384 5E

  pop esi

:00413385 C3

  ret

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
```

这个`CALL`作用是：如果注册码在41和5A之间，则注册码减41；如果注册码在61和71之间，则注册码减47；如果注册码在30和39之间，则注册码加4。

```asm
\* Referenced by a CALL at Addresses:

|:00413317  , :00413322  , :0041333C  , :00413356

|

:004132B0 8A442404                mov

al, byte ptr [esp+04]

:004132B4 3C41

  cmp al, 41

:004132B6 7C08

  jl 004132C0

:004132B8 3C5A

  cmp al, 5A

:004132BA 7F04

  jg 004132C0

:004132BC 83E841

sub eax, 00000041

:004132BF C3

  ret

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:004132B6(C), :004132BA(C)

|

:004132C0 3C61

  cmp al, 61

:004132C2 7C08

  jl 004132CC

:004132C4 3C7A

  cmp al, 7A

:004132C6 7F04

  jg 004132CC

:004132C8 83E847

sub eax, 00000047

:004132CB C3

  ret

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:004132C2(C), :004132C6(C)

|

:004132CC 3C30

  cmp al, 30

:004132CE 7C08

  jl 004132D8

:004132D0 3C39

  cmp al, 39

:004132D2 7F04

  jg 004132D8

:004132D4 83C004

add eax, 00000004

:004132D7 C3

  ret

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:004132CE(C), :004132D2(C)

|

:004132D8 3C2B

  cmp al, 2B

:004132DA 0F95C0

setne al

:004132DD 83C03E

add eax, 0000003E

:004132E0 C3

  ret
```

由于本人的能力有限，做不出它的注册机，还请高人指点！！！！！！
