
```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004719E3(U)

|

:004719CD 8D85ECFCFFFF            lea eax, dword

ptr [ebp+FFFFFCEC]

:004719D3 BA04000000              mov edx,

00000004

:004719D8 E81322F9FF              call 00403BF0

:004719DD C3

  ret

--------------------------------------------------------------------------------
```
由 `call 00402A20` 跟入：此Call判断注册码是否为`xxxx-yyyy`的形式.
```
:00402A20 53
  push ebx
:00402A21 56
  push esi
:00402A22 57
  push edi
:00402A23 89C6
  mov esi, eax
:00402A25 89D7
  mov edi, edx
:00402A27 31C9
  xor ecx, ecx
:00402A29 8A0F
  mov cl, byte ptr [edi]
:00402A2B 47
  inc edi
:00402A2C 57
  push edi
:00402A2D 31D2
  xor edx, edx
:00402A2F 8A16
  mov dl, byte ptr [esi]
:00402A31 46
  inc esi
:00402A32 4A
  dec edx
:00402A33 781B
  js 00402A50
:00402A35 8A06
  mov al, byte ptr [esi] <---将AL赋值'2D',也就是符号'-';
:00402A37 46
  inc esi
:00402A38 29D1
  sub ecx, edx
:00402A3A 7E14
  jle 00402A50

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00402A4E(U)
|
:00402A3C F2
  repnz
:00402A3D AE
  scasb <---循环依次取输入的注册码与AL中的'-'比较
:00402A3E 7510
  jne 00402A50 <---注册码中没有'-'符就跳走；
:00402A40 89CB
  mov ebx, ecx
:00402A42 56
  push esi
:00402A43 57
  push edi
:00402A44 89D1
  mov ecx, edx
:00402A46 F3
  repz
:00402A47 A6
  cmpsb
:00402A48 5F
  pop edi
:00402A49 5E
  pop esi
:00402A4A 7409
  je 00402A55
:00402A4C 89D9
  mov ecx, ebx
:00402A4E EBEC
  jmp 00402A3C

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00402A33(C), :00402A3A(C), :00402A3E(C)
|
:00402A50 5A
  pop edx
:00402A51 31C0
  xor eax, eax
:00402A53 EB05
  jmp 00402A5A
```
整理一下我的注册码为：liangs-5944406
