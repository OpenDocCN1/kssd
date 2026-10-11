
# 有声有色 3.10

> 原文：[`www.pediy.com/kssd/pediy02/forum252.htm`](https://www.pediy.com/kssd/pediy02/forum252.htm)

软件下载地址：
`http://www.newhua.com.cn/down/you310.zip`

iis 兄，你说的那个程序我看了一下，由于时间关系，没有深究，应该不算太难，首先需要脱壳，这一步比较容易，然后再用 `trw` 载入脱壳后的程序，此程序用过了 30 次之后，就会出现注册提示，拒绝执行。

这时可下：
```
:pmodule
:X (返回)
```
按下回车，被拦于下：

```
015F:0047437A 50               PUSH    EAX
015F:0047437B E8242AF9FF       CALL    `USER32!MessageBoxA` //注册提示框
015F:00474380 8BF0             MOV     ESI,EAX
015F:00474382 66837B4A00       CMP     WORD [EBX+4A],BYTE +00
015F:00474387 7408             JZ      00474391    (NO JUMP) //此处不跳则死
015F:00474389 8BD3             MOV     EDX,EBX
015F:0047438B 8B434C           MOV     EAX,[EBX+4C]
015F:0047438E FF5348           CALL    NEAR [EBX+48]
015F:00474391 33C0             XOR     EAX,EAX
015F:00474393 5A               POP     EDX
015F:00474394 59               POP     ECX
015F:00474395 59               POP     ECX
015F:00474396 648910           MOV     [FS:EAX],EDX
015F:00474399 68AE434700       PUSH    DWORD 004743AE
015F:0047439E 8D45FC           LEA     EAX,[EBP-04]
015F:004743A1 E882F6F8FF       CALL    00403A28
015F:004743A6 C3               RET
```

程序中只要将 `015F:00474387 JZ 00474391` 改为 `JMP 00474391` 即可继续执行程序！(找 `74088BD38B` 改为 `E905000000`)

iis 兄，那个软件我看了一下，用它来做网页特效真的不错，这个东西的注册程序不知道如何下手，我试了一下，修改程序可以让其继续执行。

方法如下：

```
015F:0047437A 50               PUSH    EAX
015F:0047437B E8242AF9FF       CALL    `USER32!MessageBoxA` //注册提示框
015F:00474380 8BF0             MOV     ESI,EAX
015F:00474382 66837B4A00       CMP     WORD [EBX+4A],BYTE +00
015F:00474387 7408             JZ      00474391    (NO JUMP) //此处不跳则死
015F:00474389 8BD3             MOV     EDX,EBX
015F:0047438B 8B434C           MOV     EAX,[EBX+4C]
015F:0047438E FF5348           CALL    NEAR [EBX+48]
015F:00474391 33C0             XOR     EAX,EAX
015F:00474393 5A               POP     EDX
015F:00474394 59               POP     ECX
015F:00474395 59               POP     ECX
015F:00474396 648910           MOV     [FS:EAX],EDX
015F:00474399 68AE434700       PUSH    DWORD 004743AE
015F:0047439E 8D45FC           LEA     EAX,[EBP-04]
015F:004743A1 E882F6F8FF       CALL    00403A28
015F:004743A6 C3               RET
```

程序中只要将 `015F:00474387 JZ 00474391` 改为 `JMP 00474391` 即可继续执行！(找 `74088BD38B` 改为 `E905000000`)

我们在 `015F:0047437B E8242AF9FF CALL USER32!MessageBoxA` 用 `F8` 追入，可以到达如下程序：

```
015F:BFF52E29 E8B0540000       CALL    `KERNEL32!SMapLS_IP_EBP_12` //R eip BFF52E4D
015F:BFF52E2E 50               PUSH    EAX
015F:BFF52E2F E8B6540000       CALL    `KERNEL32!SMapLS_IP_EBP_16`
015F:BFF52E34 50               PUSH    EAX
015F:BFF52E35 FF7514           PUSH    DWORD [EBP+14]
015F:BFF52E38 66FF7518         PUSH    WORD [EBP+18]
015F:BFF52E3C FF15B327F5BF     CALL    NEAR [BFF527B3]
015F:BFF52E42 98               CWDE
015F:BFF52E43 E8FC540000       CALL    `KERNEL32!SUnMapLS_IP_EBP_12`
015F:BFF52E48 E803550000       CALL    `KERNEL32!SUnMapLS_IP_EBP_16`
015F:BFF52E4D C9               LEAVE
015F:BFF52E4E C21400           RET     14
```

在消除注册提示框我有一个问题：

在程序处 `015F:BFF52E29 E8B0540000 CALL KERNEL32!SMapLS_IP_EBP_12` 修改 `eip` 的值 `r eip BFF52E4D`，可以跳过注册提示框！但命令修改为 `JMP BFF52E4D`，可是我用 `hiew` 修改程序时，找不到 `E8B0540000` 机器码？

请问高手，这是为什么？？？

my mail: `scyb@371.net`

***

`4dd466d0e1e0cc123afbe504da2462ca`
