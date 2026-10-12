
# netsowell 新壳之脱壳 & 脱壳机制作

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-595.htm`](https://www.pediy.com/kssd/pediy08/pediy8-595.htm)

## 1.0 前言

想想也好久没写过什么东西了，没有花指令和反调试，所以比较简单就写一个。

## 1.1 寻找 IAT

首先用 `OllyDbg` 载入：

```
01001000   60              PUSHA
01001001   6A 00           PUSH    0
01001003   E8 00000000     CALL    01001008
01001008   55              PUSH    EBP
01001009   8BEC            MOV     EBP, ESP
0100100B   81EC 20020000   SUB     ESP, 220
01001011   53              PUSH    EBX
01001012   56              PUSH    ESI
01001013   57              PUSH    EDI                       ; ntdll.7C930738
01001014   8DBD E0FDFFFF   LEA     EDI, [EBP-220]
0100101A   B9 88000000     MOV     ECX, 88
0100101F   B8 CCCCCCCCC    MOV     EAX, CCCCCCCCC
01001024   F3:AB           REP     STOS DWORD PTR ES:[EDI]
01001026   C745 F8 0000000>MOV     DWORD PTR [EBP-8], 0
0100102D   8B45 08         MOV     EAX, [EBP+8]                  ; notepad.<ModuleEntryPoint>
01001030   8985 E0FDFFFF   MOV     [EBP-220], EAX
```

在 `Cmdbar` 里面输入 `hr esp-4`

按 `F9` 运行后输入密码 `123456` 中断在 `OEP` 附近：

```
00A01F41   E8 00000000     CALL    00A01F46
00A01F46   58              POP     EAX                       ; kernel32.7C816D4F
00A01F47   2D 461FA000     SUB     EAX, 0A01F46
00A01F4C   8B9D 74FCFFFF   MOV     EBX, [EBP-38C]
00A01F52   035D 08         ADD     EBX, [EBP+8]                  ; notepad.<ModuleEntryPoint>
00A01F55   8998 621FA000   MOV     [EAX+A01F62], EBX
00A01F5B   C9              LEAVE
00A01F5C   C9              LEAVE
00A01F5D   83C4 10         ADD     ESP, 10
00A01F60   61              POPA
00A01F61   68 9D730001     PUSH    100739D                     ; 中断在这里
00A01F66   C3              RETN
```

可以看出 `OEP` = `100739D`。

## 1.2 修复 IAT

走到 `OEP`，选择一个调用：

```
0100739D   6A 70           PUSH    70
0100739F   68 98180001     PUSH    01001898
010073A4   E8 BF010000     CALL    01007568
010073A9   33DB            XOR     EBX, EBX
010073AB   53              PUSH    EBX
010073AC   8B3D CC100001   MOV     EDI, [10010CC]                ; 就这个吧
010073B2   FFD7            CALL    EDI                       ; ntdll.7C930738
```

进入 `[10010CC]` 看看：

```
01770000   68 9F6E2719     PUSH    19276E9F
01770005   68 816B2819     PUSH    19286B81
0177000A   68 D61EC686     PUSH    86C61ED6
0177000F   68 01028819     PUSH    19880201
01770014   E8 E7FFF9FF     CALL    01710000
01770019   0000            ADD     [EAX], AL
0177001B   0000            ADD     [EAX], AL
0177001D   0000            ADD     [EAX], AL
0177001F   0000            ADD     [EAX], AL
01770021   0000            ADD     [EAX], AL
01770023   0000            ADD     [EAX], AL
01770025   0000            ADD     [EAX], AL
01770027   0000            ADD     [EAX], AL
01770029   0000            ADD     [EAX], AL
```

似乎参数不少，进入 `CALL` 看看：

```
01710000   55              PUSH    EBP
01710001   8BEC            MOV     EBP, ESP
01710003   60              PUSHA
01710004   9C              PUSHF
01710005   8B85 08000000   MOV     EAX, [EBP+8]                  ; ntdll.7C930738
0171000B   81F0 7277B93B   XOR     EAX, 3BB97772
01710011   81F8 76753122   CMP     EAX, 22317576
01710017   0F85 B5000000   JNZ     017100D2
0171001D   E8 11000000     CALL    01710033
01710022   58              POP     EAX                       ; 01770019
01710023   9D              POPF
01710024   61              POPA
01710025   C9              LEAVE
01710026   81C4 14000000   ADD     ESP, 14
0171002C    FFA424 C0FFFFFF JMP     [ESP-40]
01710033   5E              POP     ESI                       ; 01770019
01710034   81EE 05000000   SUB     ESI, 5
0171003A   68 44656C65     PUSH    656C6544
0171003F   68 00008F00     PUSH    8F0000
01710044   68 2E646C6C     PUSH    6C6C642E
01710049   68 454C3332     PUSH    32334C45
0171004E   68 4B45524E     PUSH    4E52454B
01710053   54              PUSH    ESP
01710054   8B85 10000000   MOV     EAX, [EBP+10]
0171005A   81F0 19068819   XOR     EAX, 19880619
01710060   FF10            CALL    [EAX]
01710062   81F8 00000000   CMP     EAX, 0
01710068   0F85 0F000000   JNZ     0171007D
0171006E   54              PUSH    ESP
0171006F   8B85 14000000   MOV     EAX, [EBP+14]
01710075   81F0 03038719   XOR     EAX, 19870303
0171007B   FF10            CALL    [EAX]
0171007D   68 46035465     PUSH    65540346
01710082   68 696D6500     PUSH    656D69
01710087   68 696C6554     PUSH    54656C69
0171008C   68 65417346     PUSH    46734165
01710091   68 6D54696D     PUSH    6D69546D
01710096   68 79737465     PUSH    65747379
0171009B   68 47657453     PUSH    53746547
017100A0   54              PUSH    ESP
017100A1   50              PUSH    EAX
017100A2   8B85 0C000000   MOV     EAX, [EBP+C]
017100A8   81F0 42736686   XOR     EAX, 86667342
017100AE   FF10            CALL    [EAX]
017100B0   C606 68         MOV     BYTE PTR [ESI], 68
017100B3   8986 01000000   MOV     [ESI+1], EAX
017100B9   C9              LEAVE
017100BA   81EC 28000000   SUB     ESP, 28
017100C0   50              PUSH    EAX
017100C1   58              POP     EAX                       ; 01770019
017100C2   9D              POPF
017100C3   61              POPA
017100C4   C9              LEAVE
017100C5   81C4 14000000   ADD     ESP, 14
017100CB    FFA424 C0FFFFFF JMP     [ESP-40]
```

不需要仔细分析，可以看出进入真正 API 的地方指令都是：

```
JMP [ESP-40h]
```

所以总结 IAT 修复方案是，搜索出一个 IAT，然后执行代码，直到 `JMP [ESP-40h]`，`[ESP-40h]` 即 API 地址，将它写回 IAT 即可。

## 1.3 脱壳机制作

随便找个调试代码模板，编写以下功能代码即可完成：

1. `bp VirtualAlloc`，记录外壳解压缩的地址，为了搜索代码。
2. `bp GetModuleHandleA`，为了确认外壳开始执行，已经解压缩。
3. 搜索整个段中的 `add esp, 10h` / `popa` / `push const` / `retn`
   `bp retn`，`[esp]` 为 `OEP`。
4. `bp OEP`，执行到之后开始修复 IAT
5. 修复 IAT 代码可以参考上面一节，我是注入了一些代码，方便操作内存：




### 1.4 结论

\========

花指令非常重要。

壳不可出现规律性代码。

脱壳不难，但是写教程真是自虐。

`forgot/iPB`

![img](img/77a5f7cfd3ec840c98be3aecc80df838.jpg) 在虚拟机的 `win98` 下，不能再用 `jmp [esp-40h]`了，还有一位大牛的修复方案是 `Hook GetProcAddress`。对于壳中出现重复的代码这点，以后肯定是不会一直这样规律的。

现在的 `IAT`，这样就能兼容虚拟机中的 `win98`

```
;01150000    68 9F6E2619     PUSH    19266E9F
;01150005    68 816B2919     PUSH    19296B81
;0115000A    68 D61EC786     PUSH    86C71ED6
;0115000F    68 01028819     PUSH    19880201
;01150014    E8 E7FFF9FF     CALL    010F0000

; 求最后的 CALL, 好像很多
lea     eax, [edi+4*5]
add     eax, [eax+1]
add     eax, 5
```

```
;01710000    55              PUSH    EBP
;01710001    8BEC            MOV     EBP, ESP
;01710003    60              PUSHA
;01710004    9C              PUSHF
;01710005    8B85 08000000   MOV     EAX, [EBP+8]
;0171000B    81F0 7277B93B   XOR     EAX, 3BB97772
;01710011    81F8 76753122   CMP     EAX, 22317576
;01710017    0F85 B5000000   JNZ     017100D2
;0171001D    68 E517807C     PUSH    kernel32.GetSystemTimeAsFileTime
;01710022    58              POP     EAX
;01710023    9D              POPF
;01710024    61              POPA
;01710025    C9              LEAVE
;01710026    81C4 14000000   ADD     ESP, 14
;0171002C  - FFA424 C0FFFFFF JMP     [ESP-40]

;01001000 >  54              PUSH    ESP
;01001001    5D              POP     EBP                       ; kernel32.7C816D4F
;
```

为了加快速度，把第 2 行修改为 `push esp / pop ebp` 作为标记
不影响运行

`IAT` 之后回到这里

`magicret`

为了找到自己的 stack

`call`

模拟 `iat` 之后回到这里

`mov`

`.while`

`popa`

`;` `修复 IAT`
`;`
`pusha`

`mov` `esi, c_start[ebp]`
`mov` `edi, c_end[ebp]`

`@msg` `"* emulating IAT decryptions"`

`.WHILE` `esi<edi`

`push` `esi;防止漏掉 api`

`.IF` `w [esi] == 15FFh ||` `; JMP D[]`
`w [esi] == 25FFh ||` `; CALL D[]`
`w [esi] == 058Bh ||` `; MOV EAX, []`
`w [esi] == 158Bh ||` `; MOV EDX, []`
`w [esi] == 0D8Bh ||` `; MOV ECX, []`
`w [esi] == 1D8Bh ||` `; MOV EBX, []`
`w [esi] == 358Bh ||` `; MOV ESI, []`
`w [esi] == 3D8Bh ||` `; MOV EDI, []`
`w [esi] == 2D8Bh` `; MOV EBP, []`

`lodsw` `; 跳过 2 个字节`
`jmp` `fixiat`

`.ELSEIF` `b [esi] == 0A1h`

`lodsb` `; 跳过 1 个字节`

`fixiat:` `lodsd`

`pusha`
`xchg` `esi, eax` `; ESI = IAT 地址`

`push` `4`
`push` `esi`
`callX` `IsBadReadPtr`
`.IF` `eax == 0`

`mov` `edi, [esi]`

`push` `5*5`
`push` `edi`
`callX` `IsBadReadPtr`
`.IF` `eax == 0`

`.IF` `b [edi] == 68h &&`
`b [edi+5] == 68h &&`
`b [edi+5+5] == 68h &&`
`b [edi+5+5+5] == 68h &&`
`b [edi+5+5+5+5] == 0E8h`

`.IF` `w [eax+1] == 0EC8Bh`
`mov` `w [eax+1], 5d54h`

`;@showdw` `'new director', eax`

`;还要搜索所有的 jmp [esp-xx]`
`;修改成 jmp magicret`
`lea` `edx, magicret[ebp]`

`;开始搜索直到一片 000`
`.while` `1`
`;d [eax] != 0 && d [eax-2] !=0`
`.IF` `d [eax]==0 &&`
`d [eax+4]==0`
`.BREAK`
`.ENDIF`

`; add esp, 14`
`; jmp [esp-xx]`

`.if` `w [eax] == 0c481h &&`
`w [eax+6] == 0a4ffh`

`add` `eax, 6;停在 jmp`

`call` `makejmp`
`.endif`

`inc` `eax`
`.endw`
`.ENDIF`

`;@bpx`

`;已经定向到自己的 fixer，调用修复`

`pusha`
`push` `'FUCK'` `;为了找到自己的 stack`

`call` `d [esi]`

`; 模拟 iat 之后回到这里`
`magicret:` `@delta eax`
`mov` `ecx, [esp-40h]` `;???`
`mov` `d buf[eax], ecx`

`.while` `d [esp-4] !='FUCK'`
`add` `esp, 4`
`.endw`
`popa;恢复我们的 context`

`mov` `eax, d buf[ebp]`

`mov` `[esi], eax`

`.ENDIF`
`.ENDIF`
`.ENDIF`
`popa`

`.ENDIF`
`pop` `esi`
`inc` `esi`
`.ENDW`

`popa`

\* \* \*

```
/*950000*/  push    ebp
/*950001*/  mov     ebp, esp
/*950003*/  pushad
/*950004*/  pushfd
/*950005*/  mov     eax, [ebp+8]
/*95000B*/  xor     eax, 1B923724
/*950011*/  cmp     eax, 21A3520
/*950017*/  jnz     009500D5
/*95001D*/  call    00950038
/*950022*/  call    0054F005
/*950027*/  popfd
/*950028*/  popad
/*950029*/  mov     esp, ebp
/*95002B*/  pop     ebp
/*95002C*/  add     esp, 14
/*950032*/  jmp     [5BFDB7FA]

/*950038*/  push    72616843
/*95003D*/  push    250000
/*950042*/  push    6C6C642E
/*950047*/  push    32334C45
/*95004C*/  push    4E52454B
/*950051*/  push    esp
/*950052*/  mov     eax, [ebp+10]
/*950058*/  xor     eax, 19880620
/*95005E*/  call    eax
/*950060*/  cmp     eax, 0
/*950066*/  jnz     0095007B
/*95006C*/  push    esp
/*95006D*/  mov     eax, [ebp+14]
/*950073*/  xor     eax, 19870303
/*950079*/  call    eax
/*95007B*/  add     esp, 14
/*950081*/  push    7274736C
/*950086*/  push    2E40079
/*95008B*/  push    726F6D65
/*950090*/  push    4D65766F
/*950095*/  push    4D6C7452
/*95009A*/  push    esp
/*95009B*/  push    eax
/*95009C*/  mov     eax, [ebp+C]
/*9500A2*/  xor     eax, 86667342
/*9500A8*/  call    [eax]
/*9500AA*/  add     esp, 14
/*9500B0*/  pop     esi
```



```
/\*9500B1\*/  mov     [esi], eax
/\*9500B3\*/  call    009500B8
/\*9500B8\*/  pop     eax
/\*9500B9\*/  sub     eax, 0B3
/\*9500BF\*/  mov     [eax+2F], esi
/\*9500C5\*/  popfd
/\*9500C6\*/  popad
/\*9500C7\*/  mov     esp, ebp
/\*9500C9\*/  pop     ebp
/\*9500CA\*/  add     esp, 14
/\*9500D0\*/  jmp     00950032
```
