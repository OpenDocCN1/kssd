
# MT unlocker

> 原文：[`www.pediy.com/kssd/pediy04/kanxue127.htm`](https://www.pediy.com/kssd/pediy04/kanxue127.htm)

`MT unlocker` 是一个手机解锁软件，它的加密方式是用 `hardlock` 加密狗加密，这种加密狗是一种外壳型加密狗，和 `HASP` 很相象。

小弟没有接触过外壳的狗，所以破解时用最原始的方法：跟踪。

工具： `TRW2000`

我们先安装好程序和狗驱动程序，然后运行 `TRW`，选择 `TRW` 菜单中的“`TRnewTCB`”功能，然后运行程序，中断后就会显示程序的入口：

```
016F：008E240  PUSH    EBP    <=====中断在这里

016F:008E0241  MOV      EBP,ESP

016F:008E0243  PUSH    EBX

016F:008E0244  PUSH    ESI

016F:008E0245  PUSH    EDI

016F:008E0246  PUSHA

016F:008E0247  MOV      EAX,ESP

016F:008E0249  MOV      [008E04C4],EAX

016F:008E024E  MOV      EAX,008E04E0

016F:008E0253  SUB      EAX,[008E0500]

016F:008E0259  MOV      [008E0500],EAX

016F:008E025E  CMP      DWORD [008E04C0],BYTE +00

016F:008E0265  JZ      NEAR 008E027F

016F:008E026B  MOV      EAX,[008E04C4]

016F:008E0270  PUSH    EAX
```

然后一直按 `F10`，直到下面：

```
016F:008E0301  MOV      EDI,[008E04E8]

016F:008E0307  MOV      ECX,06

016F:008E030C  REP MOVSD

016F:008E030E  MOV      EAX,[008E04C4]

016F:008E0313  PUSH    EAX

016F:008E0314  CALL    NEAR [008E04C0]  <=====F8 进入,以后凡是遇到象这样的 CALL 都要按 F8 进入

016F:008E031A  ADD      ESP,BYTE +04
```

进入后继续按 `F10`,在中间会进入 `hardlook` 的狗驱动程序内,它的进入方式很特殊,至今我都不明白它是怎么进入的,就象是我们运行程序出错时会被 `TRW` 中断一样,在 `TRW` 的命令栏中会出现 "Break Due On Fault 03 FaultCode=A000" 的字样,我们不管它,只管按 `F10` 和 `F12` 返回到主程序中,如下:

```
016F:008F611E  MOV      EAX,[EBP-50]

016F:008F6121  PUSH    EAX

016F:008F6122  CALL    NEAR [EBP-54]

016F:008F6125  TEST    EAX,EAX      <=====返回到这里

016F:008F6127  JNZ      NEAR 008F6140

016F:008F612D  MOV      EAX,[EBP-50]
```

然后就是一直 `F10` 了,这个过程是解压缩判断狗的代码,为了加快追踪,当遇到象下面的代码时,将光标直接移到 `JNA` 指令所指向的地址,然后按 `F7` 键,在以后的地方有很多类似这样的指令,你一样可以按这种方法跳过它:

```
016F:008F61F1  ADD      DWORD [EBP-5C],BYTE +02

016F:008F61F5  ADD      DWORD [EBP-64],BYTE +02

016F:008F61F9  MOV      EAX,[EBP-68]

016F:008F61FC  MOV      ECX,[EBP-5C]

016F:008F61FF  CMP      [EAX+04],ECX    <======

016F:008F6202  JNA      NEAR 008F626B  <======注意这条指令

016F:008F6208  MOV      EAX,[EBP-64]

016F:008F620B  XOR      ECX,ECX

016F:008F620D  MOV      CX,[EAX]

016F:008F6210  AND      ECX,F000

016F:008F6216  MOV      [EBP+FFFFFF70],ECX
```

`GO,GO,GO....GO TO:`

```
016F:008F64B3  MOV      EAX,`DOSMGR_BackFill_Allowed`

016F:008F64B9  MOV      EAX,[EAX]

016F:008F64BB  MOV      `DOSMGR_BackFill_Allowed`,EAX

016F:008F64C1  MOV      EAX,[EBP+08]

016F:008F64C4  PUSH    EAX

016F:008F64C5  CALL    NEAR [008F67CC]    <=====F8 进入

016F:008F64CB  ADD      ESP,BYTE +04

016F:008F64CE  JMP      008F64D3

016F:008F64D3  MOV      ECX,[EBP-10]

016F:008F64D6  MOV      `DOSMGR_BackFill_Allowed`,ECX
```

进入 `CALL` 后会第二次进入到狗驱动程序,按上面的方法返回到主程序:

```
016F:008F8411  PUSH    EAX

016F:008F8412  CALL    NEAR [EBP-54]

016F:008F8415  TEST    EAX,EAX      <=====返回到这里

016F:008F8417  JNZ      NEAR 008F8430

016F:008F841D  MOV      EAX,[EBP-50]

016F:008F8420  PUSH    EAX

016F:008F8421  CALL    NEAR [EBP-58]

016F:008F8424  MOV      DWORD [EBP-28],08
```

一直按 `F10`,到下面的地方:

```
016F:008F87A3  MOV      EAX,`DOSMGR_BackFill_Allowed`

016F:008F87A9  MOV      EAX,[EAX]

016F:008F87AB  MOV      `DOSMGR_BackFill_Allowed`,EAX

016F:008F87B1  MOV      EAX,[EBP+08]

016F:008F87B4  PUSH    EAX

016F:008F87B5  CALL    NEAR [008F8ABC]    <=====F8 进入

016F:008F87BB  ADD      ESP,BYTE +04
```

进入后按一阵 `F10`,就到了判断加密狗的地方了:

```
016F:008E5A67  MOV      EDI,[008EFA68]

016F:008E5A6D  MOV      ECX,06

016F:008E5A72  REP MOVSD

016F:008E5A74  MOV      EAX,[EBP+08]

016F:008E5A77  PUSH    EAX

016F:008E5A78  CALL    NEAR [008EFA10]

016F:008E5A7E  ADD      ESP,BYTE +04

016F:008E5A81  JMP      008E5B7F

016F:008E5A86  CALL    008E5EC8

<=====注意这个 CALL,这个就是检测加密狗的 CALL,当没有找到狗时,EAX 就会返回 6,怎样判断它是检测狗的 CALL 呢?我的经验是,如果一个 CALL 很明显的比其它的 CALL 花费的时间要多,那么这个 CALL 九成是读狗的 CALL,在我破解的狗中,几乎是所有读狗的 CALL 都会花比较多的时间,大概是 1-3 秒吧,所以这个可以做为判断读狗的一个依据

016F:008E5A8B  MOV      [008EFA28],EAX  <=====运行到这里后,用 R 指令修改 EAX 的数值,将 EAX 改为 0,改完后你按 F5 后,你发现程序已经能够正常运行了,但我们的目标是要脱掉狗保护,所以就继续 F10 了

016F:008E5A90  CMP      DWORD [008EFA28],BYTE +00

016F:008E5A97  JZ      NEAR 008E5AD6

016F:008E5A9D  MOV      EAX,[008EFA14]

016F:008E5AA2  PUSH    EAX

016F:008E5AA3  MOV      EAX,[008EFA28]
```

继续按 `F10`,在这个过程中就是主程序的解压缩了,中间有三个 `CALL` 要 `F8` 进去的,同样可以用上来所说的加快跟踪的方法来跟踪下去,当出现下面的指令时,解压缩就基本上完成了:

```
016F:008FE97D  MOV      [009006B0],EAX

016F:008FE982  PUSH    DWORD 009005A4

016F:008FE987  MOV      EAX,[00900498]

016F:008FE98C  PUSH    EAX

016F:008FE98D  CALL    `KERNEL32!GetProcAddress`  <=====当出现这些指令时,就要放慢跟踪的速度了

016F:008FE993  MOV      [009006C0],EAX

016F:008FE998  PUSH    DWORD 009005B8

016F:008FE99D  MOV      EAX,[00900498]

016F:008FE9A2  PUSH    EAX

016F:008FE9A3  CALL    `KERNEL32!GetProcAddress`    <=====

016F:008FE9A9  MOV      [009006BC],EAX

016F:008FE9AE  PUSH    DWORD 009005CC

016F:008FE9B3  MOV      EAX,[00900498]

016F:008FE9B8  PUSH    EAX

016F:008FE9B9  CALL    `KERNEL32!GetProcAddress`    <=====
```

注意下面的代码,这些代码我估计是反跟踪的,因为我跟踪下去的时候,程序就会出现一个警告对话框,说程序是不能运行在调试状态下的,我没有办法去掉它,因为我的跟踪技术还不是很好,哪位高手来帮我补充怎么样去掉这些反跟踪代码,在这里我先谢谢了.

代码如下:

```
016F:008FEBDF  JMP      008FEC88

016F:008FEBE4  CMP      DWORD [009004BC],BYTE +00

016F:008FEBEB  JZ      NEAR 008FEC64

016F:008FEBF1  CMP      DWORD [009004C8],BYTE +00

016F:008FEBF8  JNZ      NEAR 008FEC64

016F:008FEBFE  PUSH    DWORD 008FF9C2

016F:008FEC03  MOV      EAX,[009004BC]

016F:008FEC08  PUSH    EAX

016F:008FEC09  PUSH    BYTE +00

016F:008FEC0B  PUSH    BYTE +00

016F:008FEC0D  CALL    `USER32!SetTimer`  <====设置系统定时器,如果程序在跟踪状态,程序就会退出

016F:008FEC13  MOV      [00900520],EAX

016F:008FEC18  CMP      DWORD [00900520],BYTE +00

016F:008FEC1F  JNZ      NEAR 008FEC64

016F:008FEC25  MOV      DWORD [009004A8],03
```

我们不管它了,继续 `F10`,中间有两个 `CALL` 要 `F8` 进入,进入后按 `F10` 直到出现下面的代码:

```
016F:008E1FF3  JMP      008E2003

016F:008E1FF8  POP      EDI      <=====光标停在这里

016F:008E1FF9  POP      ESI

016F:008E1FFA  POP      EBX

016F:008E1FFB  POP      EBP

016F:008E1FFC  PUSH    DWORD [008E2050]

016F:008E2002  RET

016F:008E2003  POP      EDI

016F:008E2004  POP      ESI

016F:008E2005  POP      EBX

016F:008E2006  LEAVE

016F:008E2007  RET      0C

016F:008E200A  ADD      [EAX],AL

016F:008E200C  ADD      [EAX],AL

016F:008E200E  ADD      [EAX],AL
```

在地址 `016F:008E1FF8` 处停下来,我们已经可以 `DUMP` 出文件了,下指令 `PEDUMP d:\temp\crack.exe`,至此,我们已经破掉 `hardlook` 的外壳狗了,但是由于我没有学过脱壳,所以脱出来的文件竟然有 `4M`(脱壳前只有 `1M` 多一点),而且我不知道怎么样修正 `Import` 表,让程序能在 `win2000` 下运行,不知哪位大虾对脱壳很有经验的话,能否指点小弟一两招呢,小弟感激涕零.

通过我们对这个程序的破解,我们应该清楚,在破解外壳型的加密狗时,关键在于你是否有耐心,还要有一双明亮的眼睛,外壳狗并不可怕.同时也再次证明,无论是怎么好的狗,如果开发商不好好利用的话,结果是整个加密方案以失败告终. `4dd466d0e1e0cc123afbe504da2462ca`
