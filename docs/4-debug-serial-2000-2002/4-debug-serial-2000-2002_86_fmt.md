
# Offline Explorer 1.3 230

> 原文：[`www.pediy.com/kssd/pediy02/forum124.htm`](https://www.pediy.com/kssd/pediy02/forum124.htm)

软件名称：`Offline Explorer 1.3 230`

简介：功能强大的离线浏览器

解密日期：2000 年 7 月 7 日

解密工具：`Trw2000 1.22`

## 说明

网上的朋友，大家好！

这是我写的第二篇破解实战，如若有纰漏，请各位大虾多指教！

特别感谢看雪大虾。是他编辑的破解教材《CRACK 教学》，让我在学习破解的过程中受益匪浅。

## 操作步骤

首先用 `Trw2000` 载入 `Offline Explorer`

输入注册信息：

- `name`: `xiA Qin` （任意输入）
- `register key`: `1949101` （任意输入）

下指令 `bpx hmemcpy` //下中断点

按 `F5` 回到程序，按确定，这时会被 `Trw2000` 拦截到。

下指令 `bd *` //屏障中断点

下指令 `pmodule` //直接跳到程序的领空

按 `F10` 来到下面指令：

```assembly
015F:004B8B1F  MOV      EAX,[EBP-04]
015F:004B8B22  PUSH     EAX
015F:004B8B23  LEA      EDX,[EBP-08]
015F:004B8B26  MOV      EAX,[EBX+02E0]
015F:004B8B2C  CALL     00432F2C
015F:004B8B31  MOV      EDX,[EBP-08]
015F:004B8B34  MOV      EAX,[004D8C4C]
015F:004B8B39  MOV      EAX,[EAX]
015F:004B8B3B  POP      ECX
015F:004B8B3C  CALL     004C65E0           ->这个 CALL 有问题，按 F8 进入。
015F:004B8B41  TEST     AL,AL              ->检验注册标记
015F:004B8B43  JZ       004B8BAA           ->注册码不相等，就跳 004B8BAA
015F:004B8B45  MOV      EAX,[004D8C4C]
015F:004B8B4A  MOV      EAX,[EAX]
015F:004B8B4C  MOV      BYTE [EAX+060C],01
015F:004B8B53  LEA      EDX,[EBP-04]
...............................
015F:004B8BAA  PUSH     BYTE +10
015F:004B8BAC  LEA      ECX,[EBP-10]
015F:004B8BAF  MOV      EAX,[004D8C4C]
015F:004B8BB4  MOV      EAX,[EAX]
015F:004B8BB6  MOV      EAX,[EAX+04F8]
015F:004B8BBC  MOV      EDX,07
015F:004B8BC1  CALL     004A4CB0
015F:004B8BC6  PUSH     DWORD [EBP-10]
015F:004B8BC9  PUSH     DWORD 004B8CA8
...................
015F:004B8C52  CALL     00404034
015F:004B8C57  MOV      ECX,EAX
015F:004B8C59  MOV      EDX,004B8CB8
015F:004B8C5E  MOV      EAX,[004D8E4C]
015F:004B8C63  MOV      EAX,[EAX]
015F:004B8C65  CALL     0044F6D8           ->到这里 GAME OVER 了,
015F:004B8C6A  XOR      EAX,EAX            向上看什么地方可以
015F:004B8C6C  POP      EDX                跳过这个 CALL.
015F:004B8C6D  POP      ECX
\=========================================
```

按 `F8` 进入 `004B8B3C CALL` 后

按 `F10` 来到下面的指令：

```assembly
015F:004C6930 FF75FC        PUSH     DWORD [EBP-04]
015F:004C6933 681C6D4C00    PUSH     DWORD 004C6D1C
015F:004C6938 8D45F4        LEA      EAX,[EBP-0C]
015F:004C693B BA03000000    MOV      EDX,03
015F:004C6940 E8EBD5F3FF    CALL     00403F30
015F:004C6945 BAC163D306    MOV      EDX,06D363C1
015F:004C694A 8B45F4        MOV      EAX,[EBP-0C]
015F:004C694D E8FAF9FFFF    CALL     004C634C
015F:004C6952 8D55F0        LEA      EDX,[EBP-10]
015F:004C6955 E876FAFFFF    CALL     004C63D0
015F:004C695A 8B55F0        MOV      EDX,[EBP-10]    ->真正的注册码 下指令 D EDX
015F:004C695D 8B45F8        MOV      EAX,[EBP-08]    ->这里是你输入的注册码
015F:004C6960 E81BD6F3FF    CALL     00403F80
015F:004C6965 0F94C3        SETZ     BL
015F:004C6968 33C0         XOR      EAX,EAX
015F:004C696A 5A           POP      EDX
015F:004C696B 59           POP      ECX
015F:004C696C 59           POP      ECX
015F:004C696D 648910       MOV      [FS:EAX],EDX
........................
```

抄下来，填入试一试。 `4dd466d0e1e0cc123afbe504da2462ca`
