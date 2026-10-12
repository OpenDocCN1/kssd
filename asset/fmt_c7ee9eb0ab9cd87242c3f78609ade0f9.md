
```
009A7259    90                      NOP
009A725A    90                      NOP
009A725B    90                      NOP
```

下面的几行，把程序中的 `CALL 009A702E`，改成 `CALL ( 00134938 + 0x0a * index )`

可能主要是为了提高速度，因为 `CALL` 到 `009A702E`，要执行这么多的指令，而实际上的指令只有 `0x0a` byte

```
009A725C    8B7424 24               MOV ESI,DWORD PTR SS:[ESP+24]
009A7260    83EE 04                 SUB ESI,4
009A7263    AD                      LODS DWORD PTR DS:[ESI]
009A7264    81EF 2E204000           SUB EDI,RemindBo.0040202E
009A726A    2BFD                    SUB EDI,EBP
009A726C    03C7                    ADD EAX,EDI
009A726E    8946 FC                 MOV DWORD PTR DS:[ESI-4],EAX
009A7271    5F                      POP EDI
009A7272    57                      PUSH EDI
009A7273    33C9                    XOR ECX,ECX
009A7275    83F9 08                 CMP ECX,8                     ; 循环 8 次, 将目的地址挪到 PUSHAD 进
栈的 8 个 DWORD 下面
009A7278    74 0E                   JE SHORT RemindBo.009A7288
009A727A    90                      NOP
009A727B    90                      NOP
009A727C    90                      NOP
009A727D    90                      NOP
009A727E    8B448C 04               MOV EAX,DWORD PTR SS:[ESP+ECX*4+4]
009A7282    89048C                  MOV DWORD PTR SS:[ESP+ECX*4],EAX
009A7285    41                      INC ECX
009A7286  ^ EB ED                  JMP SHORT RemindBo.009A7275
009A7288    893C8C                  MOV DWORD PTR SS:[ESP+ECX*4],EDI
009A728B    60                      PUSHAD                      ; 重新加密 009A71D9-009A7284
009A728C    E8 00000000             CALL RemindBo.009A7291
009A7291    5E                      POP ESI
009A7292    83EE 06                 SUB ESI,6
009A7295    B9 B2000000             MOV ECX,0B2
009A729A    29CE                    SUB ESI,ECX
009A729C    BA B71A20FB             MOV EDX,FB201AB7
009A72A1    C1E9 02                 SHR ECX,2
009A72A4    83E9 02                 SUB ECX,2
009A72A7    83F9 00                 CMP ECX,0
009A72AA    7C 1A                   JL SHORT RemindBo.009A72C6
009A72AC    8B048E                  MOV EAX,DWORD PTR DS:[ESI+ECX*4]
009A72AF    8B5C8E 04               MOV EBX,DWORD PTR DS:[ESI+ECX*4+4]
009A72B3    2BC3                    SUB EAX,EBX
009A72B5    C1C8 10                 ROR EAX,10
009A72B8    2BC2                    SUB EAX,EDX
009A72BA    81F2 3D309808           XOR EDX,898303D
009A72C0    89048E                  MOV DWORD PTR DS:[ESI+ECX*4],EAX
009A72C3    49                      DEC ECX
009A72C4  ^ EB E1                  JMP SHORT RemindBo.009A72A7
009A72C6    61                      POPAD                       ; 加密结束
009A72C7    61                      POPAD
009A72C8    C3                      RETN                        ; 返回 目的地址
```

目的地址，`0x0a` bytes，这就是程序 `CALL 009A702E` 真正要执行的 CODE

```
009A6807    33FD                    XOR EDI,EBP                 ; 干扰指令
009A6809    33DB                    XOR EBX,EBX                 ; 从原程序中抽掉的指令
009A680B    8B40 04                 MOV EAX,DWORD PTR DS:[EAX+4]    ; 从原程序中抽掉的指令
009A680E    33FD                    XOR EDI,EBP                 ; 干扰指令
009A6810    C3                      RETN                        ; 返回 00404FB0
```

其中有两条干扰指令，真正起作用的指令只有两条，为 `5` BYTES

壳实际上是把真正的原程序中的
```
33DB                    XOR EBX,EBX
8B40 04                 MOV EAX,DWORD PTR DS:[EAX+4]
```
抽掉，再加上两条干扰指令，然后加密保存起来，再把原程序中被抽掉的地方改成

`CALL 009A702E`

从 `009A702E` 开始的 CODE，就是根据返回地址，找到保存的加密 code，解密，然后执行.

看看从原程序中都抽掉了多少 CODE

`009A72C9 - 009A7E80`，每块为 `1` BYTE， 保存次数
`009A7E81 - 009AAD60`，每块为 `1` DWORD， 保存 Offset
`009AAD61 - 009B2290`，每块为 `10` BYTES，保存加密的 CODE

总共是 `0BB8` 块，用 `10` 进制是 `3000`，够变态的吧?!

而且每块被抽的 `5` BYTES，不仅加密，还加上两条干扰指令，搞的不能很简单的自动还原，脱了壳，
还得带着壳的一些 CODE 运行，够狠的!

先说脱壳后的问题:

脱壳后的程序，如果运行到 `009A722A`，从`[009B484A]`中取出的地址 `00134938`，
已经不可以使用了，这时候如果把解密的 CODE 放到其中，不出问题才怪!

`[009B484A]`的地址 `00134938`，应该是壳在解码程序，运行到 OEP 之前分配的内存，一脱壳就把这脱没了.

再说一说如何修复吧:

用 `OLLYDBG` 载入 `dumped_.exe`，`F2` 在 `009A702E` 下断，`F9` 运行，停在 `009A702E`

再 `F2` 在 `009A728B` 下断，`F9` 运行，停在 `009A728B`，

这时壳 `009A71D9-009A7284` 的 CODE 已解密，

然后将下面的三个地方改掉，

```
009A702E    60                      PUSHAD
009A702F    E9 A5010000             JMP RemindBo.009A71D9       ; 直接 跳过垃圾指令与壳解密的指令
009A7222    3C 20                   CMP AL,20
009A7224   /EB 13                  JMP SHORT RemindBo.009A7239 ; 总是使用共用的目的地址
009A728B   /EB 3A                  JMP SHORT RemindBo.009A72C7 ; 直接 跳过壳加密指令
```

最后将 `009A702E - 009A6810` 的区域 DUMP 出来，再用 `16` 进制编辑器，覆盖 `dumped_.exe` 的相应地方，

然后运行，一切正常.

`4dd466d0e1e0cc123afbe504da2462ca`
