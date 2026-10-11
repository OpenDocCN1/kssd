
# OBSIDIUM 1.25 Code Injection

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-544.htm`](https://www.pediy.com/kssd/pediy07/pediy7-544.htm)

OBSIDIUM V1.25 Code Injection

【目     标】: 超级自动注册申请王 V1.9

【工     具】:Olydbg1.1(diy 版)

【任     务】:不脱壳代码注入破解

【操作平台】:Win2003

【作     者】: LOVEBOOM\[DFCG\]\[FCG\]\[US\]

【相关链接】: `http://www2.skycn.com/soft/21992.html`

【简要说明】： 上午看到 FLY 的那篇文章，GOOD，没什么话好说，pf，刚好这几天正在研究壳的 Injection 技术，也搞定了几个对象，正好 OBS 的 Code Injection 没有看过，今天就看一下,不小心捡了个便宜。

【详细过程】:

怎么脱壳我就不多说了，想脱的朋友直接看 FLY 的文章就可以了。载入目标程序去掉那几个 Anti-debug，然后一路跑到 OEP 处,停下先，忘记了改系统时间，把系统时间改后一年二年等，反正就是让软件过期了。改好后，follow rtcMsgBox,然后在 ret 处设置断点，运行中断后，返回程序代码，再找一下就会发现以下几个地方改成 jmp 就可以跳过日期检测。

```
004D862C   . /0F84 30010000 JE 004D8762

……

004D7F81   /0F84 BE010000   JE 004D8145

……

004D8818   /0F84 49020000   JE 004D8A67
```

找到这几个破解点后，重新加载目标程序，去掉 Anti-debug,在 `4d862c` 处下内存写入断点，运行后中断后，跟踪会发现到下面的地方:

```
0066EF49    8B47 F8         MOV EAX,DWORD PTR DS:[EDI-8]

0066EF4C    0347 EC         ADD EAX,DWORD PTR DS:[EDI-14]
```

到这里后原程序的代码就全部解压完了。

用 `lordpe` 打开后发现这里的代码也同样加密过的，我们又重来一次找出解码处，内存访问断点，断下后，发现解码，当然这里你可直接用他的解码方法来 patch 上面的地址，我是用解压后修改代码的方法。有解压后修改代码的方法的话发现解出正确的代码后，执行上面的代码之前会经过这里：

```
0066FD0F    33C0            XOR EAX,EAX

0066FD11    40              INC EAX

0066FD12    5B              POP EBX

0066FD13    5F              POP EDI
```

用 `Lordpe` 打开再看一下，这里的代码还是加密过的，再次做重复工作，发现这里有机可乘：

```
006734F6    61              POPAD

006734F7    C3              RETN

006734F8    8105 00000000 0>ADD DWORD PTR DS:[0],340000
```

这里的代码用 `lordpe` 看一下，没错了，这里的代码没有加密。用 hex 编辑器随便写点东西运行一下，发现没有问题，这样就说明程序没有 CRC 检测。分析完毕，到这里我们要的东西全部有了，同样开始动手，写代码。代码是非常之简单的，和这篇文章一样的简单:-)，因此，我不写注释了哦。

Patch 代码如下：

```
006784EF    B8 0FFD6600     MOV EAX,0066FD0F                     ; 0

006784F4    BB 18856700     MOV EBX,00678518

006784F9    2BD8            SUB EBX,EAX

006784FB    83EB 05         SUB EBX,5

006784FE    C600 E9         MOV BYTE PTR DS:[EAX],0E9

00678501    8958 01         MOV DWORD PTR DS:[EAX+1],EBX

00678504    832C24 05       SUB DWORD PTR SS:[ESP],5

00678508    8B0424          MOV EAX,DWORD PTR SS:[ESP]

0067850B    C600 61         MOV BYTE PTR DS:[EAX],61

0067850E    C740 01 C381050>MOV DWORD PTR DS:[EAX+1],581C3

00678515    C3              RETN

00678516    90              NOP

00678517    90              NOP

00678518    50              PUSH EAX

00678519    B8 49EF6600     MOV EAX,0066EF49

0067851E    8138 8B47F803   CMP DWORD PTR DS:[EAX],3F8478B

00678524    75 21           JNZ SHORT 00678547

00678526    53              PUSH EBX

00678527    BB 60856700     MOV EBX,00678560

0067852C    2BD8            SUB EBX,EAX

0067852E    83EB 05         SUB EBX,5

00678531    C600 E8         MOV BYTE PTR DS:[EAX],0E8

00678534    8958 01         MOV DWORD PTR DS:[EAX+1],EBX

00678537    5B              POP EBX

00678538    B8 0FFD6600     MOV EAX,0066FD0F

0067853D    C600 33         MOV BYTE PTR DS:[EAX],33

00678540    C740 01 C0405B5>MOV DWORD PTR DS:[EAX+1],5F5B40C0

00678547    58              POP EAX

00678548    33C0            XOR EAX,EAX

0067854A    40              INC EAX

0067854B    5B              POP EBX

0067854C    5F              POP EDI

0067854D  ^ E9 C277FFFF     JMP 0066FD14

00678552    0000            ADD BYTE PTR DS:[EAX],AL

00678554    0000            ADD BYTE PTR DS:[EAX],AL

00678556    0000            ADD BYTE PTR DS:[EAX],AL

00678558    0000            ADD BYTE PTR DS:[EAX],AL

0067855A    0000            ADD BYTE PTR DS:[EAX],AL

0067855C    0000            ADD BYTE PTR DS:[EAX],AL

0067855E    0000            ADD BYTE PTR DS:[EAX],AL

00678560    50              PUSH EAX

00678561    B8 2C864D00     MOV EAX,004D862C

00678566    66:C700 90E9    MOV WORD PTR DS:[EAX],0E990

0067856B    B8 817F4D00     MOV EAX,004D7F81

00678570    66:C700 90E9    MOV WORD PTR DS:[EAX],0E990

00678575    B8 18884D00     MOV EAX,004D8818

0067857A    66:C700 90E9    MOV WORD PTR DS:[EAX],0E990

0067857F    8D4424 04       LEA EAX,DWORD PTR SS:[ESP+4]

00678583    8328 05         SUB DWORD PTR DS:[EAX],5

00678586    8B00            MOV EAX,DWORD PTR DS:[EAX]

00678588    C600 8B         MOV BYTE PTR DS:[EAX],8B

0067858B    C740 01 47F8034>MOV DWORD PTR DS:[EAX+1],4703F847

00678592    58              POP EAX

00678593    C3              RETN
```

这篇文章和 OBSIDIUM 1.25 脱壳一样，非常之简单，脱壳破解半个钟可以搞定，直接 Code injection 也同样半个钟就可以搞的说。祝你成功!!不要问我关于这个软件的注册算法/完全破解之类的哦。

BTW: Vcasm’s portect also can Code Injectioned.try it

Greetz:

Fly.Jingulong,yock,tDasm.David.hexer,hmimys,ahao.UFO(brother).alan(sister).all of my friends and you!

By loveboom\[DFCG\]\[FCG\]\[US\]

Email:loveboom#163.com

Date:4/30/2005 3:09:00 PM

`4dd466d0e1e0cc123afbe504da2462ca`
