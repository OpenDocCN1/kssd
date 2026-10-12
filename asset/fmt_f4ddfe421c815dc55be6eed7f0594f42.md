
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
