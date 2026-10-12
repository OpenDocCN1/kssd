
# EXEscope 5.0

> 原文：[`www.pediy.com/kssd/pediy02/forum125.htm`](https://www.pediy.com/kssd/pediy02/forum125.htm)

`EXEscope 5.0`

设断点`bpx hmemcpy`

程序被中断后，按`F12`键 12 次，再按`F10`键，便可找到下面的程序

```
:0047CD77 8D55FC               lea edx, dword ptr [ebp-04]

:0047CD7A 8B83DC010000         mov eax, dword ptr [ebx+000001DC]

:0047CD80 E82363FAFF           call 004230A8

:0047CD85 8B55FC               mov edx, dword ptr [ebp-04]

:0047CD88 A1E8704800           mov eax, dword ptr [004870E8]

:0047CD8D E8C26BF8FF           call 00403954

:0047CD92 8D55FC               lea edx, dword ptr [ebp-04]

:0047CD95 8B83E0010000         mov eax, dword ptr [ebx+000001E0]

:0047CD9B E80863FAFF           call 004230A8

:0047CDA0 8B55FC               mov edx, dword ptr [ebp-04]

:0047CDA3 A198704800           mov eax, dword ptr [00487098]

:0047CDA8 E8A76BF8FF           call 00403954

:0047CDAD 8B1598704800         mov edx, dword ptr [00487098]

:0047CDB3 8B12                 mov edx, dword ptr [edx]

:0047CDB5 A1906F4800           mov eax, dword ptr [00486F90]

:0047CDBA 8B00                 mov eax, dword ptr [eax]

:0047CDBC E84B770000           call 0048450C

:0047CDC1 84C0                 test al, al  <----此处测试标志

:0047CDC3 0F848D000000         je 0047CE56
```

下面分析产生标志的过程:

* Referenced by a CALL at Addresses:

| `:0047CDBC`, `:0048433C`

|

```
:0048450C 55                   push ebp

:0048450D 8BEC                 mov ebp, esp

:0048450F 51                   push ecx

:00484510 53                   push ebx

:00484511 8955FC               mov dword ptr [ebp-04], edx

:00484514 8B45FC               mov eax, dword ptr [ebp-04]

:00484517 E814F8F7FF           call 00403D30

:0048451C 33C0                 xor eax, eax

:0048451E 55                   push ebp

:0048451F 6876454800           push 00484576

:00484524 64FF30               push dword ptr fs:[eax]

:00484527 648920               mov dword ptr fs:[eax], esp

:0048452A 33DB                 xor ebx, ebx

:0048452C 8B45FC               mov eax, dword ptr [ebp-04]

:0048452F E848F6F7FF           call 00403B7C

:00484534 83F80A               cmp eax, 0000000A <----比较注册码是否为 10 位

:00484537 7527                 jne 00484560  <----不是,继续试用期

:00484539 8B45FC               mov eax, dword ptr [ebp-04]

:0048453C 803841               cmp byte ptr [eax], 41 <---比较第一位注册码是否为"A"

:0048453F 751F                 jne 00484560  <----不是,继续试用期

:00484541 8B45FC               mov eax, dword ptr [ebp-04] <----读取第八个字符

:00484544 0FB64008             movzx eax, byte ptr [eax+08]

:00484548 8B55FC               mov edx, dword ptr [ebp-04] <----读取第九个字符

:0048454B 0FB65209             movzx edx, byte ptr [edx+09]

:0048454F 03C2                 add eax, edx  <----相加

:00484551 B90A000000           mov ecx, 0000000A

:00484556 99                   cdq

:00484557 F7F9                 idiv ecx   <----作除法

:00484559 83FA04               cmp edx, 00000004 <----余数是否为 4

:0048455C 7502                 jne 00484560  <----不是,继续试用期

:0048455E B301                 mov bl, 01  <----标志
```

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

| `:00484537(C)`, `:0048453F(C)`, `:0048455C(C)`

|

```
:00484560 33C0                 xor eax, eax

:00484562 5A                   pop edx

:00484563 59                   pop ecx

:00484564 59                   pop ecx

:00484565 648910               mov dword ptr fs:[eax], edx

:00484568 687D454800           push 0048457D
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `:0048457B(U)`

|

```
:0048456D 8D45FC               lea eax, dword ptr [ebp-04]

:00484570 E88BF3F7FF           call 00403900

:00484575 C3                   ret

:00484576 E90DEEF7FF           jmp 00403388

:0048457B EBF0                 jmp 0048456D

:0048457D 8BC3                 mov eax, ebx  <----标志

:0048457F 5B                   pop ebx

:00484580 59                   pop ecx

:00484581 5D                   pop ebp

:00484582 C3                   ret
```

用户名：（好像没用）

注册码：`A123456799`

请问 6.0 版在哪儿下? `4dd466d0e1e0cc123afbe504da2462ca`
