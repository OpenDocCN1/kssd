
# 英语听力通 v2.1

> 原文：[`www.pediy.com/kssd/pediy04/kanxue357.htm`](https://www.pediy.com/kssd/pediy04/kanxue357.htm)

**破解来由:**

昨晚在论坛闲逛,居然发现了这么一条消息.

你好 eryl，能否帮忙破解”听力通（Audio-Aid)"---我见过的功能最完善，界面最友好的一款复读软件。

**软件名称:**英语听力通

**最新版本:**Ver 2.1

**文件大小:**3,923KB

**升级时间:**2001/12/21

**操作平台:**Win9x/NT/2000/Me

**软件授权:**共享

注册用户免费升级

Download Link:http://www.englishwriters.com/aa/aadown.asp?website=7

用过 3 种复读机各有优缺点：1、先博电脑仿真可视化英文复读机只能使用 wav 音频；2、winamp 英语复读机不便于操作；3、SitMan PC 复读机功能不够强大.

英语听力通是我见过的功能最完善，界面最友好的一款复读软件。

我也纳闷怎么会要我破解的.不过想一想自己也是学语言的,说不定那东西还真有用,就载下来看了看.

倒!用 `fi` 一看,是 `vb6` 的东西!

俺顿时没了情绪.不过既然都花了好半天下载了,还是看看吧!

首先想到用 `w32dasm` 反汇编,郁闷,没有找到任何有价值的提示.然后我就会想到要用 `winhex`,结果也是无效.只好找 `ollydbg` 啦!

俺高高兴兴的拿 `ollydbg` 载入程序,走了一步,心里就暗喜找对了对象.可到了第二步就怎么也下不去了!............晕!`vb` 的东西!

呜呼!没的选啦!被逼无奈啦!俺终于不得不祭起 `trw2000` 的大旗.硬着头皮,"排除万难,去夺取胜利",嘿嘿.

先用 `trw` 装载程序,然后启动程序,输入注册信息.

返回 `trw`,下 `bpx hmemcpy`,`f5` 返回程序,确认.立刻被拦.

`f12`,先走一边吧!你会发现程序在两处不同的地方呼出出错框.不管啦,俺的汇编又不好,只要能得结果就满足了.就在其中选其一吧!再想象这是 `vb` 的东西,自然选择在 `vb` 的程序段初下断.

于是你再走一边!这次知道 `f12 16` 此就可以了.

你会来到这里:

`6605f5ba ret 08.......`光标在这里!`f10` 往下走喽!

```
016F:004FE3A1  PUSH    EAX
016F:004FE3A2  CALL    `MSVBVM60!__vbaHresultCheckObj`
016F:004FE3A8  MOV     EDX,[EBP-48]
016F:004FE3AB  PUSH    EDX......when you get here,and give the order:d edx, you can get your false code!that is to say you should be careful for the near later processes.
016F:004FE3AC  CALL    `MSVBVM60!rtcR8ValFromBstr`
016F:004FE3B2  FILD    DWORD [0051B198]
016F:004FE3B8  FSTP    QWORD [EBP+FFFFFF08]
016F:004FE3BE  FADD    QWORD [EBP+FFFFFF08]
016F:004FE3C4  FADD    QWORD [004048B8]
016F:004FE3CA  FNSTSW  AX
016F:004FE3CC  TEST    AL,0D
016F:004FE3CE  JNZ     NEAR 004FE7F3
016F:004FE3D4  CALL    `MSVBVM60!__vbaFpR8`
016F:004FE3DA  FSTP    QWORD [EBP+FFFFFF00]
016F:004FE3E0  PUSH    DWORD 004214EC....a dubious reg code! but i am not sure.
016F:004FE3E5  CALL    `MSVBVM60!__vbaR8Str`
016F:004FE3EB  FCOMP   QWORD [EBP+FFFFFF00]
016F:004FE3F1  FNSTSW  AX
016F:004FE3F3  TEST    AH,40
016F:004FE3F6  JNZ     004FE3FF
016F:004FE3F8  MOV     EAX,01
016F:004FE3FD  JMP     SHORT 004FE401
016F:004FE3FF  XOR     EAX,EAX
016F:004FE401  NEG     EAX
016F:004FE403  LEA     ECX,[EBP-48]
016F:004FE406  MOV     EDI,EAX
016F:004FE408  CALL    `MSVBVM60!__vbaFreeStr`
016F:004FE40E  LEA     ECX,[EBP-5C]
016F:004FE411  CALL    `MSVBVM60!__vbaFreeObj`
016F:004FE417  MOV     ECX,80020004
016F:004FE41C  MOV     EAX,0A
016F:004FE421  CMP     DI,SI
016F:004FE424  MOV     [EBP+FFFFFF6C],ECX
016F:004FE42A  MOV     [EBP+FFFFFF64],EAX
016F:004FE430  MOV     [EBP+FFFFFF7C],EC
016F:004FE436  MOV     [EBP+FFFFFF74],EAX
016F:004FE43C  JZ      NEAR 004FE4C7..here is the critical jump!actually,if you make a jump here,the software itself will automatically write the reg file into the reg table.:>
016F:004FE442  MOV     EDI,[004012B4]
016F:004FE448  MOV     EBX,08
016F:004FE44D  LEA     EDX,[EBP+FFFFFF44]
016F:004FE453  LEA     ECX,[EBP-7C]
016F:004FE456  MOV     DWORD [EBP+FFFFFF4C],00421534
016F:004FE460  MOV     [EBP+FFFFFF44],EBX
016F:004FE466  CALL    EDI
016F:004FE468  LEA     EDX,[EBP+FFFFFF54]
016F:004FE46E  LEA     ECX,[EBP-6C]
016F:004FE471  MOV     DWORD [EBP+FFFFFF5C],00421508
016F:004FE47B  MOV     [EBP+FFFFFF54],EBX
016F:004FE481  CALL    EDI
016F:004FE483  LEA     EAX,[EBP+FFFFFF64]
016F:004FE489  LEA     ECX,[EBP+FFFFFF74]
016F:004FE48F  PUSH    EAX
016F:004FE490  LEA     EDX,[EBP-7C]
016F:004FE493  PUSH    ECX
016F:004FE494  PUSH    EDX
016F:004FE495  LEA     EAX,[EBP-6C]
016F:004FE498  PUSH    BYTE +30
016F:004FE49A  PUSH    EAX
016F:004FE49B  CALL    `MSVBVM60!rtcMsgBox`.....the crashing nag!
016F:004FE4A1  LEA     ECX,[EBP+FFFFFF64]
016F:004FE4A7  LEA     EDX,[EBP+FFFFFF74]
016F:004FE4AD  PUSH    ECX
016F:004FE4AE  LEA     EAX,[EBP-7C]
016F:004FE4B1  PUSH    EDX
016F:004FE4B2  LEA     ECX,[EBP-6C]
016F:004FE4B5  PUSH    EAX
016F:004FE4B6  PUSH    ECX
016F:004FE4B7  PUSH    BYTE +04
016F:004FE4B9  CALL    `MSVBVM60!__vbaFreeVarList`
016F:004FE4BF  ADD     ESP,BYTE +14
016F:004FE4C2  JMP     004FE755
016F:004FE4C7  MOV     EDI,[004012B4]....jump here from 4fe43c!
016F:004FE4CD  MOV     EBX,08
016F:004FE4D2  LEA     EDX,[EBP+FFFFFF44]
016F:004FE4D8  LEA     ECX,[EBP-7C]
016F:004FE4DB  MOV     DWORD [EBP+FFFFFF4C],00421594
016F:004FE4E5  MOV     [EBP+FFFFFF44],EBX
016F:004FE4EB  CALL    EDI
016F:004FE4ED  LEA     EDX,[EBP+FFFFFF54]
016F:004FE4F3  LEA     ECX,[EBP-6C]
016F:004FE4F6  MOV     DWORD [EBP+FFFFFF5C],00421544
016F:004FE500  MOV     [EBP+FFFFFF54],EBX
016F:004FE506  CALL    EDI
016F:004FE508  LEA     EDX,[EBP+FFFFFF64]
016F:004FE50E  LEA     EAX,[EBP+FFFFFF74]
016F:004FE514  PUSH    EDX
016F:004FE515  LEA     ECX,[EBP-7C]
016F:004FE518  PUSH    EAX
016F:004FE519  PUSH    ECX
016F:004FE51A  LEA     EDX,[EBP-6C]
016F:004FE51D  PUSH    BYTE +30
016F:004FE51F  PUSH    EDX
016F:004FE520  CALL    `MSVBVM60!rtcMsgBox`.....succeed!
016F:004FE526  MOV     EBX,[00401040]
016F:004FE52C  LEA     EAX,[EBP+FFFFFF64]
016F:004FE532  LEA     ECX,[EBP+FFFFFF74]
016F:004FE538  PUSH    EAX
016F:004FE539  LEA     EDX,[EBP-7C]
016F:004FE53C  PUSH    ECX
016F:004FE53D  LEA     EAX,[EBP-6C]
016F:004FE540  PUSH    EDX
016F:004FE541  PUSH    EAX
016F:004FE542  PUSH    BYTE +04
016F:004FE544  CALL    EBX
016F:004FE546  MOV     EAX,[0051B098]
016F:004FE54B  ADD     ESP,BYTE +14
016F:004FE54E  CMP     EAX,ESI
016F:004FE550  JNZ     004FE567
016F:004FE552  PUSH    DWORD 0051B098
```

`ok!`让我们回到程序界面,发现已经注册.重起及删除重装都没有问题.

但俺还有一个小问题,不知哪位仁兄能给解答!

俺在 `016F:004FE3E0  PUSH    DWORD 004214EC` 处是可以得到一个注册码的,但这个注册码和注册后软件显示的注册码并不一样.而且还只有用软件显示的注册码注册才显示注册正确!不知何故?那个新码是哪里来的呢?怎么跟出来呢?

有兴趣的兄弟可以看一看!  `4dd466d0e1e0cc123afbe504da2462ca`
