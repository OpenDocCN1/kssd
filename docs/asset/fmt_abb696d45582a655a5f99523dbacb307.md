
# CleanReg 3.2.6

> 原文：[`www.pediy.com/kssd/pediy02/forum241.htm`](https://www.pediy.com/kssd/pediy02/forum241.htm)

菜鸟破解录之 `CleanReg` `3.2.6`

- 软件名称：`CleanReg`
- 软件版本：`3.2.6`
- 软件大小：`1574KB`
- 软件授权：共享软件
- 使用平台：`Win95/98/NT`
- 发布公司：[`http://www.cleanreg.com/`](http://www.cleanreg.com/)
- 软件简介：

`CleanReg3` 帮您将系统中无用的文件全部一扫而空。`CleanReg3` 与一般系统登录文件修改工具软件不同的是，它并非真正立即将这些登录设定给移除掉，而是先将您的系统中目前有那些文件连结有错误的先显示出来，然后您可以再选择是要透过手动的方式选择是要忽略掉这些无用的文件或者是要移除掉。

- 作 者：`xiA Qin`
- 级 别：很菜....很菜.....
- 解密日前：`2000` 年 `8` 月 `12` 日
- 解密工具：`Trw2000` `1.22`
- 破解目的：学习注册码的破解。
- 说 明：

本文是在我的软件破解记录上整理出来的。如若有纰漏，请各位大侠多指教！

首先运行 `CleanReg`

输入注册信息

`Registered: xiA Qin` & 任意输入
`License: 1234567890` & 输入的十进制数字不能大于 `2147483647`。
也就是十六进制不能大于 `7FFFFFFF`。

下指令 `bpx hmemcpy` // 下中断点

按 `F5` 回到程序，按确定，这时会被 `Trw2000` 拦截到。

下指令 `bd *` // 屏障中断点
下指令 `pmodule` // 直接跳到程序的领空

按 `F10` 来到下面指令

```
.................
015F:004062CA  MOV     ECX,ESI
015F:004062CC  PUSH    EAX
015F:004062CD  PUSH    00000409
015F:004062D2  MOV     BYTE PTR [ESP+24],01
015F:004062D7  CALL    00411CC2
015F:004062DC  PUSH    01
015F:004062DE  PUSH    00
015F:004062E0  PUSH    000003ED
015F:004062E5  MOV     ECX,ESI     <-|拌脚石一块，检察输入的注册码
015F:004062E7  CALL    00411CBC       |是否大于十六进数字制的 7FFFFFFF
015F:004062EC  MOV     EDI,EAX       |
015F:004062EE  TEST    EDI,EDI     <-|
015F:004062F0  JZ      0040634E      //大于 7FFFFFFF,就跳，玩完了。
015F:004062F2  MOV     ECX,[ESP+08]
015F:004062F6  MOV     EAX,[ECX-08]
015F:004062F9  TEST    EAX,EAX
015F:004062FB  JLE     0040634E
015F:004062FD  LEA     EDX,[ESP+10]
015F:00406301  LEA     EAX,[ESP+0C]
015F:00406305  PUSH    EDX
015F:00406306  LEA     ECX,[ESP+0C]
015F:0040630A  PUSH    EAX
015F:0040630B  PUSH    ECX
015F:0040630C  CALL    00411550      //注册码运算子程序。

015F:00406311  ADD     ESP,0C
015F:00406314  TEST    EAX,EAX
015F:00406316  JZ      0040634E
015F:00406318  CMP     [ESP+10],EDI  //比较输入的注册码。
015F:0040631C  JNZ     0040634E      //不相等，就跳。玩完了
015F:0040631E  PUSH    EDI
015F:0040631F  MOV     ECX,0041BBA8
015F:00406324  CALL    00405A10
015F:00406329  MOV     EDX,[ESP+08]
015F:0040632D  MOV     EAX,[0041BCFC]
015F:00406332  MOV     ECX,[0041BCF8]
015F:00406338  PUSH    EDX
015F:00406339  PUSH    EAX
015F:0040633A  PUSH    ECX
015F:0040633B  MOV     ECX,0041BBA8
015F:00406340  CALL    00411BD8
015F:00406345  MOV     ECX,ESI
015F:00406347  CALL    004116A4
015F:0040634C  JMP     00406368
015F:0040634E  PUSH    00
015F:00406350  PUSH    05
015F:00406352  PUSH    0041B3AC
015F:00406357  CALL    004119EC      //
015F:0040635C  SUB     EAX,02
015F:0040635F  JNZ     00406368
015F:00406361  MOV     ECX,ESI
.......................
```

当光标在 `00406318 CMP [ESP+10],EDI` 时

看一下 `EDI` 和 `SS:0066F344=` 的值

`EDI=499602D2` <<-- 眼熟吧...嘻嘻...就是 `1234567890` 的 `16` 进值...
`SS:0066F344=66A29C38` <<-- 当然就是注册码罗...换算成 `10` 进值...

答案就是：`1721932856`

按 `F5` 回到程序。重新输入。

整理一下注册信息。

`Registered: xiA Qin`
`License: 1721932856`

注册信息在注册表中的

`[HKEY_CURRENT_USER\Software\ArmstrongSystems\CleanReg3\Registered]`

```
"CodeB"=hex:14,00,00,00,03,02,05,00,45,9c,af,66,40,ae,1f,40,04,05,c0,01
"User"="xiA Qin"
```
`4dd466d0e1e0cc123afbe504da2462ca`
