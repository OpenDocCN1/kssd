
# NOX

> 原文：[`www.pediy.com/kssd/pediy03/forum507.htm`](https://www.pediy.com/kssd/pediy03/forum507.htm)

## NOX 免光盘版制作手记

**作者**：风飘雪  
**主页**：`http://duba.126.com`

NOX 乃 WESTWOOD 公司大作，暗黑类游戏，运行需读光盘。不插盘显示 `"NOX CD Missing"`。用 `w32dasm` 反汇编 `NOX.exe`，用串式参考 `string data ref` 找不到该语句。一筹莫展时突然看见：

`"CDFailText"`
`"CDFailTitle"`
`"CDFS"`

双击 `"CDFailText"`，得到：

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004134A4(C)  (由此跳来)
|
:004134AB 68A0364100             push 004136A0
:004134B0 68C0364100             push 004136C0
:004134B5 6A23                   push 00000023
:004134B7 68C8000000             push 000000C8
* Possible StringData Ref from Data Obj -> "C:\Nox\src\common\System\gamedisk.c"
                                  |
:004134BC 68C86A5700             push 00576AC8
:004134C1 6A00                   push 00000000
* Possible StringData Ref from Data Obj -> "CDFailText"  (关键)
                                  |
:004134C3 68EC6A5700             push 00576AEC
:004134C8 E833BDFFFF             call 0040F200
:004134CD 83C410                 add esp, 00000010
:004134D0 50                     push eax
:004134D1 68C7000000             push 000000C7
* Possible StringData Ref from Data Obj -> "C:\Nox\src\common\System\gamedisk.c"
                                  |
:004134D6 68F86A5700             push 00576AF8
:004134DB 6A00                   push 00000000
* Possible StringData Ref from Data Obj -> "CDFailTitle"  (关键)
                                  |
:004134DD 681C6B5700             push 00576B1C
:004134E2 EB60                   jmp 00413544
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00413489(C)
|
:004134E4 83F801                 cmp eax, 00000001  (此处为关键，到此可跳过检测 CD)
:004134E7 0F8581000000           jne 0041356E
```

向上看，找跳转语句绕过 CD 检测段：

```
:0041347D E80EEA0000             call 00421E90
:00413482 85C0                   test eax, eax
:00413484 A3CC105F00             mov dword ptr [005F10CC], eax
:00413489 7559                   jne 004134E4  (改此 jne => jmp)
:0041348B E870020000             call 00413700
:00413490 E85B6C0300             call 0044A0F0
:00413495 6800080000             push 00000800
:0041349A E80174FFFF             call 0040A8A0
:0041349F 83C404                 add esp, 00000004
:004134A2 85C0                   test eax, eax
:004134A4 7405                   je 004134AB   (关键)
:004134A6 E835020000             call 004136E0
```

用 `UltraEdit` 搜寻 `A3CC105F007559` => `A3CC105F00EB59`。

收工，真的想来罐红牛！

有空来主页逛逛，提点儿意见，并向 `toye`、`sunfeng`、`fwd`、`wocy` 问好。

2001.1.4 下午 4dd466d0e1e0cc123afbe504da2462ca
