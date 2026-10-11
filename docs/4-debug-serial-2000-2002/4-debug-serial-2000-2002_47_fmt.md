
# EditPlus v2.01

> 原文：[`www.pediy.com/kssd/pediy02/forum196.htm`](https://www.pediy.com/kssd/pediy02/forum196.htm)

菜鸟破解实录（五）之 EditPlus v2.01

软件名称：EditPlus &nbsp;&nbsp;&nbsp;&nbsp;-->(30 天试用)

版&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;本：v2.01

简&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;介： 一套功能强大，可取代记事本的文字编辑器，拥有无限制的 `Undo/Redo`、英文拼字检查、自动换行、列数标记、搜寻取代、同时编辑多文件、全屏幕浏览功能。而它还有一个好用的功能，就是它有监视剪贴簿的功能，能够同步于剪贴簿自动将文字贴进 EditPlus 的编辑窗口中，让你省去做贴上的步骤。另外它也是一个好用的 HTML 编辑器，除了可以颜色标记 `HTMLTag`(同时支持 C/C++、Perl、Java)外，还内建完整的 HTML 和 CSS1 指令功能，对于习惯用记事本编辑网页的朋友，它可帮你节省一半以上的网页制作时间，若你有安装 `IE3.0` 以上版本，它还会结合 IE 浏览器于 EditPlus 窗口中，让你可以直接预览编辑好的网页(若没安装 IE，也可指定浏览器路径)。

作&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;者：xiA Qin

级&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;别：刚学不久

解密日前：2000 年 7 月 30 日

解密工具：`Trw2000 1.22`

破解目的：学习注册码的破解。（简单）

说&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;明：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;本文是在我的软件破解记录上整理出来的。如若有纰漏，请各位大侠多指教！

首先运行 `EditPlus`

输入注册信息

`name:` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`xiA Qin` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`&任意输入`

`register key:` &nbsp;&nbsp;`9876543210` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`&任意输入`

下指令 `bpx hmemcpy` &nbsp;&nbsp;`//下中断点`

按 `F5` 回到程序，按确定，这时会被 `Trw2000` 拦截到。

下指令 `bd \*` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`//屏障中断点`

下指令 `pmodule` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`//直接跳到程序的领空`

按 `F10` 来到下面指令

...............

```
015F:00462070  LEA       ECX,[EDI+5C]
015F:00462073  PUSH      EDX
015F:00462074  CALL      0049FE4D
015F:00462079  MOV       AL,[ESI]
015F:0046207B  CMP       AL,20
015F:0046207D  JZ        00462083
015F:0046207F  CMP       AL,09
015F:00462081  JNZ       00462086
015F:00462083  INC       ESI
015F:00462084  JMP       00462079
015F:00462086  PUSH      ESI
015F:00462087  CALL      [004CD278]
015F:0046208D  TEST      EAX,EAX
015F:0046208F  JLE       004620A4
015F:00462091  MOV       CL,[EAX+ESI-01]
015F:00462095  CMP       CL,20
015F:00462098  JZ        0046209F
015F:0046209A  CMP       CL,09
015F:0046209D  JNZ       004620A4
015F:0046209F  DEC       EAX
015F:004620A0  TEST      EAX,EAX
015F:004620A2  JG        00462091
015F:004620A4  TEST      EAX,EAX
015F:004620A6  MOV       BYTE PTR [ESI+EAX],00
015F:004620AA  JZ        0046219E
015F:004620B0  MOV       EAX,[ESP+08]
015F:004620B4  MOV       ECX,[EAX-08]
015F:004620B7  TEST      ECX,ECX
015F:004620B9  JZ        0046219E
015F:004620BF  LEA       ECX,[ESP+08]
015F:004620C3  CALL      004A3D99
015F:004620C8  MOV       ECX,[ESP+08]
015F:004620CC  PUSH      ECX
015F:004620CD  PUSH      ESI
015F:004620CE  CALL      00462280        //有问题，按 F8 进入子程序。
015F:004620D3  ADD       ESP,08
015F:004620D6  TEST      EAX,EAX
015F:004620D8  JNZ       004620ED        //这里可以跳过失败对话框。
................................

015F:00462280  83EC0C                 SUB       ESP,0C
015F:00462283  53                      PUSH      EBX
015F:00462284  8B1D78D24C00            MOV       EBX,[004CD278]
015F:0046228A  56                      PUSH      ESI
015F:0046228B  57                      PUSH      EDI
015F:0046228C  8B7C241C                MOV       EDI,[ESP+1C]
015F:00462290  57                      PUSH      EDI
015F:00462291  FFD3                    CALL      EBX
015F:00462293  8BF0                    MOV       ESI,EAX
015F:00462295  85F6                    TEST      ESI,ESI
015F:00462297  7507                    JNZ       004622A0        //修改这里，变成任意注册版
015F:00462299  5F                      POP       EDI
015F:0046229A  5E                      POP       ESI
015F:0046229B  5B                      POP       EBX
015F:0046229C  83C40C                 ADD       ESP,0C
015F:0046229F  C3                      RET
015F:004622A0  E83BFFFFFF              CALL      004621E0
015F:004622A5  56                      PUSH      ESI
015F:004622A6  57                      PUSH      EDI
015F:004622A7  6A00                    PUSH      00
015F:004622A9  E882FFFFFF              CALL      00462230
015F:004622AE  25FFFF0000              AND       EAX,0000FFFF
015F:004622B3  50                      PUSH      EAX
015F:004622B4  8D44241C                LEA       EAX,[ESP+1C]
015F:004622B8  6814624F00              PUSH      004F6214
015F:004622BD  50                      PUSH      EAX
015F:004622BE  E8CDD60200              CALL      0048F990
015F:004622C3  8B742438                MOV       ESI,[ESP+38]
015F:004622C7  8A442424                MOV       AL,[ESP+24]
015F:004622CB  83C418                 ADD       ESP,18
015F:004622CE  8A4E02                 MOV       CL,[ESI+02]
015F:004622D1  8D7E02                 LEA       EDI,[ESI+02]
015F:004622D4  3AC8                    CMP       CL,AL
015F:004622D6  7409                    JZ        004622E1
015F:004622D8  5F                      POP       EDI
015F:004622D9  5E                      POP       ESI
015F:004622DA  33C0                    XOR       EAX,EAX
015F:004622DC  5B                      POP       EBX
015F:004622DD  83C40C                 ADD       ESP,0C
015F:004622E0  C3                      RET
015F:004622E1  8A5603                 MOV       DL,[ESI+03]
015F:004622E4  8A44240D                MOV       AL,[ESP+0D]
015F:004622E8  3AD0                    CMP       DL,AL
015F:004622EA  7409                    JZ        004622F5
015F:004622EC  5F                      POP       EDI
015F:004622ED  5E                      POP       ESI
015F:004622EE  33C0                    XOR       EAX,EAX
015F:004622F0  5B                      POP       EBX
015F:004622F1  83C40C                 ADD       ESP,0C
015F:004622F4  C3                      RET
015F:004622F5  56                      PUSH      ESI
015F:004622F6  FFD3                    CALL      EBX
015F:004622F8  83E802                 SUB       EAX,02
015F:004622FB  50                      PUSH      EAX
015F:004622FC  57                      PUSH      EDI
015F:004622FD  6A00                    PUSH      00
015F:004622FF  E82CFFFFFF              CALL      00462230
015F:00462304  25FFFF0000              AND       EAX,0000FFFF
015F:00462309  50                      PUSH      EAX
015F:0046230A  8D44241C                LEA       EAX,[ESP+1C]
015F:0046230E  6814624F00              PUSH      004F6214
015F:00462313  50                      PUSH      EAX
015F:00462314  E877D60200              CALL      0048F990
015F:00462319  8A0E                    MOV       CL,[ESI]
015F:0046231B  8A442424                MOV       AL,[ESP+24]
015F:0046231F  83C418                 ADD       ESP,18
015F:00462322  3AC8                    CMP       CL,AL
015F:00462324  7409                    JZ        0046232F
015F:00462326  5F                      POP       EDI
015F:00462327  5E                      POP       ESI
015F:00462328  33C0                    XOR       EAX,EAX
015F:0046232A  5B                      POP       EBX
015F:0046232B  83C40C                 ADD       ESP,0C
015F:0046232E  C3                      RET

............................
015F:004620DA  PUSH      FF
015F:004620DC  PUSH      10
015F:004620DE  PUSH      00005F81
015F:004620E3  CALL      004AB493        //注册失败对话框
015F:004620E8  JMP       004621A6
015F:004620ED  MOV       EAX,[EDI+000000D4]
015F:004620F3  TEST      EAX,EAX
015F:004620F5  JNZ       0046214C
015F:004620F7  MOV       EDX,[004F7750]
015F:004620FD  MOV       [ESP+0C],EDX
015F:00462101  MOV       EAX,[ESP+08]
015F:00462105  LEA       ECX,[ESP+0C]
015F:00462109  PUSH      EAX
015F:0046210A  PUSH      ESI
015F:0046210B  PUSH      000053D7
015F:00462110  PUSH      ECX
015F:00462111  MOV       BYTE PTR [ESP+0000021C],01
015F:00462119  CALL      0049CB8A
015F:0046211E  MOV       EDX,[ESP+1C]
015F:00462122  ADD       ESP,10
015F:00462125  PUSH      00
015F:00462127  PUSH      24
```

从上面可以看出。

将 `00462297 7507 JNZ 004622A0`

改 `00462297 7407 JZ 004622A0`

就可以注册。

整里一下，用 `Ultraedt` 打开 `EditPlus`

找到 `75 07 5f 5e 5b 83 c4 0c`

改成 `74 07 5f 5e 5b 83 c4 0c`

保存修改文件，重新运行

输入注册信息 ，现在是想输入什么都可以。

`name:` &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`China` &nbsp;&nbsp;&nbsp;`&任意输入`

`register key:` &nbsp;&nbsp;`1949101` &nbsp;&nbsp;`&任意输入`

保存修改程序，重新运行试一试。



好象从`1.21`版本就开始了。它判断注册码不是集中在一个地方判断，程序刚启动的时候判断几位，退出的时候再判断另外几位，你使用它的`preferences`菜单的时候再判断一位。

以上是旧版本的情况。`2.01a`版本与此有所不同。由于不知道它会在哪里判断注册码，所以不能清除`BPR`断点，即要一直保留针对输入的假注册码所设的`BPR`断点；或者用`bpx RegDeleteValueA`设断点，试用一下其各种功能，看它是否会删除注册表中的假注册码，就可以找到判断注册码的地方。另外，这个软件的注册码的前`5`位是由后面的计算出来的，而后面有多少位并无明显的限制，所以各人作出来的注册码长短不一。另，据说注册码似乎有随机性。

1.22 版本在用户使用`preferences`菜单时会判一位注册码，此时注册码会失效(空)`(dr0`

`4dd466d0e1e0cc123afbe504da2462ca`
