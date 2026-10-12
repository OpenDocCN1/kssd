
# 环宇通汉英翻译系统 3.0 脱壳

> 原文：[`www.pediy.com/kssd/pediy04/kanxue132.htm`](https://www.pediy.com/kssd/pediy04/kanxue132.htm)

环宇通汉英翻译系统 3.0 用 `LOCK98` 加壳，这个壳相对来说还是很温和的。没有 `ANTI-DEBUG`，没有破坏输入表。

简要说一下。

一开始有很多 `SMC` 来还原程序的代码，很简单，但比较长就不多说了。遇到 `LOOP` 就 `G` 到它的下面一句，只要有耐心通过他并不困难。然后走到读密匙盘的地方如下：

```
0167:004E7C37  MOV       BYTE [EBP+00433DE9],00
0167:004E7C3E  MOV       AL,01
0167:004E7C40  MOV       DL,[EBP+00433DE9]
0167:004E7C46  CALL      004E78B5
0167:004E7C4B  MOV       AL,02
0167:004E7C4D  MOV       DL,[EBP+00433DE9]
0167:004E7C53  CALL      004E78B5
0167:004E7C58  MOV       AX,0401
0167:004E7C5C  MOV       CX,4F01
0167:004E7C60  MOV       DH,01
0167:004E7C62  MOV       DL,[EBP+00433DE9]
0167:004E7C68  CALL      004E7925
0167:004E7C6D  MOV       AX,0201
0167:004E7C71  MOV       CX,4F32
0167:004E7C75  MOV       DH,01
0167:004E7C77  MOV       DL,[EBP+00433DE9]
0167:004E7C7D  CALL      004E7925
0167:004E7C82  MOV       DL,[EBP+00433DE9]
0167:004E7C88  CALL      004E78FD
0167:004E7C8D  MOV       DL,[EBP+00433DE9]
0167:004E7C93  CALL      004E78FD
0167:004E7C98  MOV       EBX,[EBP+00433DD7]
0167:004E7C9E  CMP       EBX,[EBP+00433DBF]
0167:004E7CA4  JZ        004E7CB7    <----注意这个，强迫跳
0167:004E7CA6  CMP       BYTE [EBP+00433DE9],01
0167:004E7CAD  JNC       004E7CCE
0167:004E7CAF  INC       BYTE [EBP+00433DE9]
0167:004E7CB5  JMP       SHORT 004E7C3E
0167:004E7CB7  PUSH      EAX
0167:004E7CB8  LEA       EDI,[EBP+00433BD7]
0167:004E7CBE  LEA       ECX,[EBP+00433CCA]
0167:004E7CC4  SUB       ECX,EDI
0167:004E7CC6  DEC       ECX
0167:004E7CC7  CLD
0167:004E7CC8  XOR       AL,AL
0167:004E7CCA  REP STOSB
0167:004E7CCC  POP       EAX
0167:004E7CCD  RET
```

上面强迫跳转后就不会要求密匙盘了。再往下还有一些还原代码，最后到如下：

```
0167:004E7118  MOV       EAX,004300A6
0167:004E711D  ADD       EAX,EBP
0167:004E711F  SUB       EAX,[EBP+00433DBB]
0167:004E7125  PUSH      EAX
0167:004E7126  LEA       EDI,[EBP+00430000]
0167:004E712C  LEA       ECX,[EBP+00433138]
0167:004E7132  SUB       ECX,EDI
0167:004E7134  DEC       ECX
0167:004E7135  CLD
0167:004E7136  XOR       AL,AL
0167:004E7138  REP STOSB
0167:004E713A  POP       EAX
0167:004E713B  PUSH      EAX
0167:004E713C  LEA       EDI,[EBP+004332D3]
0167:004E7142  LEA       ECX,[EBP+00434000]
0167:004E7148  SUB       ECX,EDI
0167:004E714A  DEC       ECX
0167:004E714B  CLD
0167:004E714C  XOR       AL,AL
0167:004E714E  REP STOSB
0167:004E7150  POP       EAX
0167:004E7151  MOV       BYTE [EBP+00433DE8],00
0167:004E7158  POP       EBP
0167:004E7159  JMP       EAX     <----到原入口
```

原入口可能需要密盘数据来还原，没有密盘所以得到的入口是不正确的，再运行会非法操作。可以这样，在最后那句把程序 `DUMP` 下来，反汇编。看里面的函数调用，比如：`GetCommandLineA`、`GetVersion` 这些函数一般都出现在程序入口处，找一下，找到如下：

```
:0042DED7 55             push ebp
:0042DED8 8BEC           mov ebp, esp
:0042DEDA 6AFF           push FFFFFFFF
:0042DEDC 68D0754600     push 004675D0
:0042DEE1 6810224300     push 00432210
:0042DEE6 64A100000000   mov eax, dword ptr fs:[00000000]
:0042DEEC 50             push eax
:0042DEED 64892500000000 mov dword ptr fs:[00000000], esp
:0042DEF4 83EC58         sub esp, 00000058
:0042DEF7 53             push ebx
:0042DEF8 56             push esi
:0042DEF9 57             push edi
:0042DEFA 8965E8         mov dword ptr [ebp-18], esp
* Reference To: KERNEL32.GetVersion, Ord:0174h
                           |
:0042DEFD FF1570034600   Call dword ptr [00460370]
:0042DF03 33D2           xor edx, edx
:0042DF05 8AD4           mov dl, ah
:0042DF07 8915945A4800   mov dword ptr [00485A94], edx
:0042DF0D 8BC8           mov ecx, eax
:0042DF0F 81E1FF000000   and ecx, 000000FF
:0042DF15 890D905A4800   mov dword ptr [00485A90], ecx
:0042DF1B C1E108         shl ecx, 08
:0042DF1E 03CA           add ecx, edx
:0042DF20 890D8C5A4800   mov dword ptr [00485A8C], ecx
:0042DF26 C1E810         shr eax, 10
:0042DF29 A3885A4800     mov dword ptr [00485A88], eax
:0042DF2E 6A01           push 00000001
:0042DF30 E862560000     call 00433597
:0042DF35 59             pop ecx
:0042DF36 85C0           test eax, eax
:0042DF38 7508           jne 0042DF42
:0042DF3A 6A1C           push 0000001C
:0042DF3C E8C3000000     call 0042E004
:0042DF41 59             pop ecx
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0042DF38(C)
|
:0042DF42 E8A5330000     call 004312EC
:0042DF47 85C0           test eax, eax
:0042DF49 7508           jne 0042DF53
:0042DF4B 6A10           push 00000010
:0042DF4D E8B2000000     call 0042E004
:0042DF52 59             pop ecx
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0042DF49(C)
|
:0042DF53 33F6           xor esi, esi
:0042DF55 8975FC         mov dword ptr [ebp-04], esi
:0042DF58 E8C13E0000     call 00431E1E
* Reference To: KERNEL32.GetCommandLineA, Ord:00CAh
                           |
:0042DF5D FF15F8014600   Call dword ptr [004601F8]
:0042DF63 A3C8714800     mov dword ptr [004871C8], eax
:0042DF68 E8F8540000     call 00433465
:0042DF6D A3D05A4800     mov dword ptr [00485AD0], eax
:0042DF72 E8A1520000     call 00433218
:0042DF77 E8E3510000     call 0043315F
:0042DF7C E8E5EEFFFF     call 0042CE66
:0042DF81 8975D0         mov dword ptr [ebp-30], esi
:0042DF84 8D45A4         lea eax, dword ptr [ebp-5C]
:0042DF87 50             push eax
```

所以估计 `42DED7` 就是真正的入口，试一下果然正确。 `4dd466d0e1e0cc123afbe504da2462ca`
