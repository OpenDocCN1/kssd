
### 汇编代码段 1

```
00D30E86    A1 D439D300     mov eax, dword ptr ds:[D339D4]

00D30E8B    8945 DC         mov dword ptr ss:[ebp-24], eax

00D30E8E    8D95 ACD7FFFF   lea edx, dword ptr ss:[ebp-2854]

00D30E94    8B07            mov eax, dword ptr ds:[edi]

00D30E96    E8 D9F8FFFF     call 00D30774

00D30E9B    8B85 ACD7FFFF   mov eax, dword ptr ss:[ebp-2854]

00D30EA1    8D95 B0D7FFFF   lea edx, dword ptr ss:[ebp-2850]

00D30EA7    E8 4093FFFF     call 00D2A1EC

00D30EAC    8B95 B0D7FFFF   mov edx, dword ptr ss:[ebp-2850]

00D30EB2    8BC7            mov eax, edi

00D30EB4    E8 5323FFFF     call 00D2320C

00D30EB9    E8 3A000000     call 00D30EF8

00D30EBE    68 C70ED300     push 0D30EC7

00D30EC3    FF0424          inc dword ptr ss:[esp]

00D30EC6    C3              retn
```

#### 堆栈内容

```
0012D724   0012FFE0  指针到下一个 SEH 记录

0012D728   00D312CF  SE 句柄

0012D72C   0012FF90

0012D730   00D20000

0012D734   00D00000

0012D738   00D306D4

0012D73C   00000000

0012D740   00D40414  ASCII "SOFTWAREASPack"
```

---

### 汇编代码段 2

```
00D30F07    3100            xor dword ptr ds:[eax], eax

00D30F09    EB 01           jmp short 00D30F0C

00D30F0B    68 648F0500     push 58F64

00D30F10    0000            add byte ptr ds:[eax], al

00D30F12    00EB            add bl, ch

00D30F14    02E8            add ch, al

00D30F16    0158 8D         add dword ptr ds:[eax-73], ebx

00D30F19    05 6C36D300     add eax, 0D3366C

00D30F1E    50              push eax

00D30F1F    E8 04F6FFFF     call 00D30528

00D30F24    E8 3A000000     call 00D30F63

00D30F29    68 320FD300     push 0D30F32

00D30F2E    FF0424          inc dword ptr ss:[esp]

00D30F31    C3              retn
```

#### 堆栈内容

```
0012D724   0012FFE0  指针到下一个 SEH 记录

0012D728   00D312CF  SE 句柄

0012D72C   0012FF90

0012D730   00D20000

0012D734   00D00000

0012D738   00D306D4

0012D73C   00D40430  ASCII

"0vKiRNl66uKx8sot6BFZOoQUZGrqmy5ObFIeSG8OZ0JJh0vizDoYi8KDpenBkQOczSPgwYqx9XcacySkO3xpCZPVv/GvABIkLwh4+x7emwMecMKZOavEkpJrmNMt

Mq0gOS4Ad06Qr1Tzfd6D3zrsLchc/7zMOFljkOAroxn79Ilc="

0012D740   00D404EC  ASCII

"0vKiRNl66uKx8sot6BFZOoQUZGrqmy5ObFIeSG8OZ0JJh0vizDoYi8KDpenBkQOczSPgwYqx9XcacySkO3xpCZPVv/GvABIkLwh4+x7emwMecMKZOavEkpJrmNMt

Mq0gOS4Ad06Qr1Tzfd6D3zrsLchc/7zMOFljkOAroxn79Ilc="
```

---

### 汇编代码段 3

```
00D305CC    8B43 02         mov eax, dword ptr ds:[ebx+2]

00D305CF    50              push eax

00D305D0    8D43 06         lea eax, dword ptr ds:[ebx+6]

00D305D3    50              push eax

00D305D4    E8 47BAFFFF     call 00D2C020

00D305D9    85C0            test eax, eax

00D305DB    74 15           je short 00D305F2

00D305DD    33C9            xor ecx, ecx

00D305DF    B2 01           mov dl, 1

00D305E1    B8 F8ECD200     mov eax, 0D2ECF8                         ; ASCII 06,"EFound"

00D305E6    E8 15ADFFFF     call 00D2B300

00D305EB    E8 5C28FFFF     call 00D22E4C

00D305F0    EB 13           jmp short 00D30605

00D305F2    33C9            xor ecx, ecx

00D305F4    B2 01           mov dl, 1

00D305F6    B8 4C9CD200     mov eax, 0D29C4C

00D305FB    E8 00ADFFFF     call 00D2B300

00D30600    E8 4728FFFF     call 00D22E4C
```

#### 堆栈内容

```
0012D6D8   00D33670

0012D6DC   00000010

0012D6E0   0012D6EC  指针到下一个 SEH 记录

0012D6E4   00D30612  SE 句柄

0012D6E8   0012D718

0012D6EC   0012D724  指针到下一个 SEH 记录

0012D6F0   00D306C3  SE 句柄
```

---

### 汇编代码段 4

```
00D30F72    3100            xor dword ptr ds:[eax], eax

00D30F74    EB 01           jmp short 00D30F77

00D30F76    68 648F0500     push 58F64

00D30F7B    0000            add byte ptr ds:[eax], al

00D30F7D    00EB            add bl, ch

00D30F7F    02E8            add ch, al

00D30F81    0158 8B         add dword ptr ds:[eax-75], ebx

00D30F84    07              pop es

00D30F85    E8 8223FFFF     call 00D2330C

00D30F8A    48              dec eax

00D30F8B    50              push eax

00D30F8C    8BC7            mov eax, edi

00D30F8E    E8 A524FFFF     call 00D23438

00D30F93    40              inc eax

00D30F94    8D8D B0D7FFFF   lea ecx, dword ptr ss:[ebp-2850]

00D30F9A    5A              pop edx

00D30F9B    E8 80D7FFFF     call 00D2E720

00D30FA0    8B85 B0D7FFFF   mov eax, dword ptr ss:[ebp-2850]

00D30FA6    50              push eax

00D30FA7    8D85 ACD7FFFF   lea eax, dword ptr ss:[ebp-2854]

00D30FAD    8B17            mov edx, dword ptr ds:[edi]

00D30FAF    8A12            mov dl, byte ptr ds:[edx]

00D30FB1    E8 1623FFFF     call 00D232CC
```

#### 堆栈内容

```
0012D71C   0012D724  指针到下一个 SEH 记录

0012D720   00D30F29  SE 句柄

0012D724   0012FFE0  指针到下一个 SEH 记录

0012D728   00D312CF  SE 句柄

0012D72C   0012FF90

0012D730   00D20000

0012D734   00D00000

0012D738   00D306D4

0012D73C   00D40430  ASCII

"0vKiRNl66uKx8sot6BFZOoQUZGrqmy5ObFIeSG8OZ0JJh0vizDoYi8KDpenBkQOczSPgwYqx9XcacySkO3xpCZPVv/GvABIkLwh4+x7emwMecMKZOavEkpJrmNMt

Mq0gOS4Ad06Qr1Tzfd6D3zrsLchc/7zMOFljkOAroxn79Ilc="

0012D740   00D404EC  ASCII

"0vKiRNl66uKx8sot6BFZOoQUZGrqmy5ObFIeSG8OZ0JJh0vizDoYi8KDpenBkQOczSPgwYqx9XcacySkO3xpCZPVv/GvABIkLwh4+x7emwMecMKZOavEkpJrmNMt

Mq0gOS4Ad06Qr1Tzfd6D3zrsLchc/7zMOFljkOAroxn79Ilc="
```

---

### 汇编代码段 5

```
00D31013    3100            xor dword ptr ds:[eax], eax

00D31015    EB 01           jmp short 00D31018

00D31017    68 648F0500     push 58F64

00D3101C    0000            add byte ptr ds:[eax], al

00D3101E    00EB            add bl, ch

00D31020    02E8            add ch, al

00D31022    0158 8B         add dword ptr ds:[eax-75], ebx

00D31025    45              inc ebp

00D31026    C4E8            les ebp, eax                       ; 非法使用寄存器

00D31028    E0 22           loopdne short 00D3104C

00D3102A    FFFF            ???                                ; 未知命令

00D3102C    50              push eax

00D3102D    8B45 C4         mov eax, dword ptr ss:[ebp-3C]

00D31030    E8 F723FFFF     call 00D2342C

00D31035    8D4D D0         lea ecx, dword ptr ss:[ebp-30]

00D31038    5A              pop edx

00D31039    E8 FED9FFFF     call 00D2EA3C

00D3103E    8BD8            mov ebx, eax

00D31040    E8 3A000000     call 00D3107F

00D31045    68 4E10D300     push 0D3104E

00D3104A    FF0424          inc dword ptr ss:[esp]

00D3104D    C3              retn
```

#### 堆栈内容

```
0012D71C   0012D724  指针到下一个 SEH 记录

0012D720   00D30FCA  SE 句柄

0012D724   0012FFE0  指针到下一个 SEH 记录

0012D728   00D312CF  SE 句柄
```

---

### 汇编代码段 6

```
00D3108E    3100            xor dword ptr ds:[eax], eax

00D31090    EB 01           jmp short 00D31093

00D31092    68 648F0500     push 58F64

00D31097    0000            add byte ptr ds:[eax], al

00D31099    00EB            add bl, ch

00D3109B    02E8            add ch, al

00D3109D    0158 83         add dword ptr ds:[eax-7D], ebx

00D310A0    FB              sti

00D310A1    0175 64         add dword ptr ss:[ebp+64], esi

00D310A4    837D F8 00      cmp dword ptr ss:[ebp-8], 0

00D310A8    74 5E           je short 00D31108
```
