
# LeapFTP 2.74

> 原文：[`www.pediy.com/kssd/pediy06/pediy6142.htm`](https://www.pediy.com/kssd/pediy06/pediy6142.htm)

软件大小： `882 KB`

软件语言： `简体中文`

软件类别： `汉化补丁` / `共享版` / `FTP 工具`

应用平台： `Win9x` / `NT` / `2000` / `XP`

界面预览： `无`

加入时间： `2003-06-30 18:31:37`

下载次数： `121021`

推荐等级： `☆☆☆☆☆`

软件下载: [`count.skycn.com/softdown.php?id=711&url=http://on165-http.skycn.net:8080/down/HA_LeapFTP274.exe`](http://count.skycn.com/softdown.php?id=711&url=http://on165-http.skycn.net:8080/down/HA_LeapFTP274.exe)

软件介绍：

功能强大，媲美 `BulletProof FTP` 的 `FTP` 软件。跟 `Netscape` 相仿的书签形式，连线更加方便。下载与上传文件支持续传。可下载或上传整个目录，亦可直接删除整个目录。可让你编列顺序一次下载或上传同一站台中不同目录下的文件。浏览网页时若在文件连结上按鼠标右键选 `[复制捷径]` 便会自动下载该文件。具有不会因闲置过久而被站台踢出的功能。可直接编辑远端 `Server` 上的文件。可设定文件传送完毕自动中断 `Modem` 连接。

## 破解工具

`TRW1.22`

作者声明：初学破解，仅作学习交流之用，失误之处敬请大侠赐教!

这个软件无壳,`Delphi` 编写,但是反反汇编,而且我第 1 次用 `OD` 载入时运行一会儿就死机,不知是不是防 `OD`,只好请出国产的 `TRW` 搞定.

用 `bpx hmemcpy` 下断!

### 简要过程

用户名: `ShenGe[BCG]`

试验码: `ABC1-EF2G-I3JK-4MNO`

.........(略)

```
0048824C  PUSH    EBP
0048824D  MOV     EBP, ESP
00487EA6  MOV     EAX, [LOCAL.2]
00487EA9  LEA     EDX, [LOCAL.1]
00487EAC  CALL    LEAPFTP.00408D74
00487EB1  CMP     BYTE PTR DS:[EBX+2F4], 0
00487EB8  JE      SHORT LEAPFTP.00487EC8
00487EBA  MOV     EDX, [LOCAL.1]
00487EBD  MOV     EAX, EBX
00487EBF  CALL    LEAPFTP.0048824C
                <---这个是关键的 Call,跟进!①
00487EC4  TEST    AL, AL
00487EC6  JNZ     SHORT LEAPFTP.00487EEE
                <---关键跳转!注册码正确则跳!
00487EC8  MOV     EAX, DWORD PTR DS:[EBX+2F0]
00487ECE  PUSH    EAX
00487ECF  LEA     EDX, [LOCAL.3]
00487ED2  MOV     EAX, DWORD PTR DS:[EBX+2D0]
00487ED8  CALL    LEAPFTP.00433E1C
00487EDD  MOV     EDX, [LOCAL.3]
00487EE0  MOV     ECX, [LOCAL.1]
00487EE3  MOV     EAX, EBX
00487EE5  CALL    LEAPFTP.004880A4
00487EEA  TEST    AL, AL
00487EEC  JE      SHORT LEAPFTP.00487F50
00487EEE  LEA     EDX, [LOCAL.4]
00487EF1  MOV     EAX, DWORD PTR DS:[EBX+2E4]
00487EF7  CALL    LEAPFTP.00433E1C
00487EFC  MOV     EAX, [LOCAL.4]
00487EFF  PUSH    EAX
00487F00  LEA     EDX, [LOCAL.5]
00487F03  MOV     EAX, DWORD PTR DS:[EBX+2D0]
00487F09  CALL    LEAPFTP.00433E1C
00487F0E  MOV     ECX, [LOCAL.5]
00487F11  MOV     EDX, DWORD PTR DS:[EBX+2EC]
00487F17  MOV     EAX, EBX
00487F19  CALL    LEAPFTP.004883CC
00487F1E  MOV     EAX, LEAPFTP.00487F9C
00487F23  CALL    LEAPFTP.0045AD88
                <--- Thank You For Registering!
00487F28  MOV     DWORD PTR DS:[EBX+234], 1
00487F32  LEA     EDX, [LOCAL.6]
00487F35  MOV     EAX, DWORD PTR DS:[EBX+2D0]
00487F3B  CALL    LEAPFTP.00433E1C
00487F40  MOV     EDX, [LOCAL.6]
00487F43  LEA     EAX, DWORD PTR DS:[EBX+2E8]
00487F49  CALL    LEAPFTP.00403D48
00487F4E  JMP     SHORT LEAPFTP.00487F65
00487F50  PUSH    0
00487F52  MOV     CX, WORD PTR DS:[487FB8]
00487F59  MOV     DL, 1
00487F5B  MOV     EAX, LEAPFTP.00487FC4
00487F60  CALL    LEAPFTP.0045AC90
                <---注册码错误!
00487F65  XOR     EAX, EAX
```

★★★★★★★★★

①跟进那个关键的 `Call`,来到以下代码:

```
0048824F  ADD     ESP, -0C
00488252  PUSH    EBX
00488253  PUSH    ESI
00488254  PUSH    EDI
00488255  MOV     [LOCAL.1], EDX
00488258  MOV     EAX, [LOCAL.1]
0048825B  CALL    LEAPFTP.00404128
00488260  XOR     EAX, EAX
00488262  PUSH    EBP
00488263  PUSH    LEAPFTP.004883B9
00488268  PUSH    DWORD PTR FS:[EAX]
0048826B  MOV     DWORD PTR FS:[EAX], ESP
0048826E  MOV     BYTE PTR SS:[EBP-5], 0
00488272  MOV     EAX, [LOCAL.1]
                <---EAX="ABC1-EF2G-I3JK-4MNO"
00488275  CALL    LEAPFTP.00403F74
                <---取假码位数
0048827A  CMP     EAX, 13
                <---注册码位数必须为 19 位
0048827D  JNZ     LEAPFTP.004883A3
00488283  MOV     EAX, [LOCAL.1]
00488286  CMP     BYTE PTR DS:[EAX+4], 2D
                <---比较第 5 位是否为"-"
0048828A  JNZ     LEAPFTP.004883A3
00488290  MOV     EAX, [LOCAL.1]
00488293  CMP     BYTE PTR DS:[EAX+9], 2D
                <---比较第 10 位是否为"-"
00488297  JNZ     LEAPFTP.004883A3
0048829D  MOV     EAX, [LOCAL.1]
004882A0  CMP     BYTE PTR DS:[EAX+E], 2D
                <---比较第 15 位是否为"-"
004882A4  JNZ     LEAPFTP.004883A3
004882AA  XOR     ESI, ESI
004882AC  XOR     EDI, EDI
004882AE  XOR     EAX, EAX
                <---EAX=0
004882B0  MOV     [LOCAL.3], EAX
004882B3  MOV     EBX, 1
                <---EBX=1
004882B8  /MOV    EAX, EBX
004882BA  |AND    EAX, 80000003
        |        <---由于 EBX 初始值为 1,所以这儿为判断是否为假码的
                    4 8 12 位,从而转向相关代码段进行不同的处理
004882BF  |JNS     SHORT LEAPFTP.004882C6
004882C1  |DEC     EAX
004882C2  |OR      EAX, FFFFFFFC
004882C5  |INC     EAX
004882C6  |TEST    EAX, EAX
004882C8  |JNZ     SHORT LEAPFTP.004882E0
004882CA  |MOV     EAX, [LOCAL.1]
004882CD  |MOV     AL, BYTE PTR DS:[EAX+EBX-1]
                  <---按位取假码
004882D1  |CALL    LEAPFTP.00488224
                   <---此 Call 对取得的字符进行判断,看是否
                   满足 Hex 值在 2F 与 3A 之间,即必须为数字
004882D6  |TEST    AL, AL
004882D8  |JE      LEAPFTP.004883A3
                  <---不能跳!
004882DE  |JMP     SHORT LEAPFTP.00488302
004882E0  |MOV     EAX, EBX
004882E2  |MOV     ECX, 5
                  <---ECX=5
004882E7  |CDQ
004882E8  |IDIV    ECX
004882EA  |TEST    EDX, EDX
004882EC  |JE      SHORT LEAPFTP.00488302
        |        <---控制 5 个一组地处理注册码
004882EE  |MOV     EAX, [LOCAL.1]
004882F1  |MOV     AL, BYTE PTR DS:[EAX+EBX-1]
        |        <---按位取假码
004882F5  |CALL    LEAPFTP.00488238
                  <---此 Call 对取得的字符进行判断,看是否
                  满足 Hex 值在 40 与 5B 之间,即必须为大写
                  字母所组成!
004882FA  |TEST    AL, AL
004882FC  |JE      LEAPFTP.004883A3
        |        <---若字符不满足这里会跳走!
00488302  |MOV     EAX, [LOCAL.1]
00488305  |MOV     AL, BYTE PTR DS:[EAX+EBX-1]
        |        <---按位取注册码
00488309  |CMP     AL, 2D
        |        <---是否为"-",是"-"则不参与运算
0048830B  |JE      SHORT LEAPFTP.0048833A
0048830D  |CMP     EBX, 5
        |        <---比较是否取完 1 组
00488310  |JGE     SHORT LEAPFTP.0048831E
```

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

```
00488312  |MOV     EDX, [LOCAL.1]
00488315  |AND     EAX, 0FF
        |          <---取低位
0048831A  |ADD     ESI, EAX
        |          <---将其 Hex 值累加到 ESI 中,为
                    第 1 组的结果即注册码第 1 至 4 位,我的结果为 0xF7
0048831C  |JMP     SHORT LEAPFTP.0048833A
0048831E  |CMP     EBX, 0A
                  <---是否处理完第 2 组,即假码第 10 位
00488321  |JGE     SHORT LEAPFTP.0048832F
```

\--------------------------------

```
00488323  |MOV     EDX, [LOCAL.1]
00488326  |AND     EAX, 0FF
0048832B  |ADD     EDI, EAX
0048832D  |JMP     SHORT LEAPFTP.0048833A
```

\---------------------------------

第 2 组的 `Hex` 值累加到 `EDI` 中,即注册码的 6 至 9 位,我
的结果为 `0x104`

\---------------------------------

```
0048832F  |MOV     EDX, [LOCAL.1]
00488332  |AND     EAX, 0FF
00488337  |ADD     [LOCAL.3], EAX
```

\---------------------------------

第 3 组的 `Hex` 值累加到 `[LOCAL.3]` 中,即注册码的 11 至 14 位,我
的结果为 `0x111`

```
0048833A  |INC     EBX
0048833B  |CMP     EBX, 0F
        |        <---比较是否处理完前 15 个字符
0048833E  \JNZ     LEAPFTP.004882B8

00488344  LEA     ECX, DWORD PTR DS:[EDI+ESI]
                <---ECX=EDI+ESI=0xF7+0x104=0x1FB
00488347  ADD     ECX, [LOCAL.3]
                <---ECX=0x1FB+0x111=0x30C,即 3 部分结果的
                的累加值再相加
```

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

★★★★★★★★★

```
0048834A  MOV     EAX, ESI
                <---EAX=0xF7,第一部分累加值
0048834C  MOV     EBX, 1A
                <---EBX=0x1A
00488351  CDQ
00488352  IDIV    EBX
```



00488354  ADD EDX, 41
<---EDX=EDX+0x41=0x4E--->对应字符为"N"

00488357  MOV EAX, [LOCAL.1]

0048835A  CMP DL, BYTE PTR DS:[EAX+F]
<---比较注册码第 16 位

0048835D  JNZ SHORT LEAPFTP.004883A3
----------------------------------------

0048835F  MOV EAX, EDI
<---EAX=0x104,第二部分累加值

00488361  MOV EBX, 1A

00488366  CDQ

00488367  IDIV EBX

00488369  ADD EDX, 41
<---EDX=41,对应字符为"A"

0048836C  MOV EAX, [LOCAL.1]

0048836F  CMP DL, BYTE PTR DS:[EAX+10]
<---比较注册码第 17 位

00488372  JNZ SHORT LEAPFTP.004883A3
------------------------------------------

00488374  MOV EAX, [LOCAL.3]
<---EAX=0x111,第三部分累加值

00488377  MOV EBX, 1A

0048837C  CDQ

0048837D  IDIV EBX

0048837F  ADD EDX, 41
<---EDX=4E,对应字符为"N"

00488382  MOV EAX, [LOCAL.1]

00488385  CMP DL, BYTE PTR DS:[EAX+11]
<---比较注册码第 18 位

00488388  JNZ SHORT LEAPFTP.004883A3
-------------------------------------------

0048838A  MOV EAX, ECX
<---EAX=0x30C,3 部分累加值的总和

0048838C  MOV ECX, 1A

00488391  CDQ

00488392  IDIV ECX

00488394  ADD EDX, 41
<---EDX=41,对应字符为"A"

00488397  MOV EAX, [LOCAL.1]

0048839A  CMP DL, BYTE PTR DS:[EAX+12]
<---比较注册码第 19 位

0048839D  JNZ SHORT LEAPFTP.004883A3

★★★★★★★★★

0048839F  MOV BYTE PTR SS:[EBP-5], 1
<---置标志位 1,程序中应该有校验位

004883A3  XOR EAX, EAX
<---EAX=0!Bad Boy! :(

004883A5  POP EDX

004883A6  POP ECX

004883A7  POP ECX

004883A8  MOV DWORD PTR FS:[EAX], EDX

004883AB  PUSH LEAPFTP.004883C0

004883B0  LEA EAX, [LOCAL.1]

004883B3  CALL LEAPFTP.00403CF4

004883B8  RETN

由此得到我的正确注册码为:`ABC1-EF2G-I3JK-NANA`

### 总结

注册码与用户名无关!注册码必须为 19 位,其中第 5,10,15 位为`"-"`,注册码的第 4,8,12 位必须为数字,其余位为大写字母.假设注册码的第 1 至 4 位的 `Hex` 值累加值为`a`,第 6 至 9 位的 `Hex` 值累加值为`b`,第 11 至 14 位的 `Hex` 值累加值为`c`,则注册码的后 4 位分别为:

第 16 位---->`Asc(a mod 0x1A+41)`

第 17 位---->`Asc(b mod 0x1A+41)`

第 18 位---->`Asc(c mod 0x1A+41)`

第 19 位---->`Asc((a+b+c) mod 0x1A+41)`

软件注册成功后将注册信息保存在注册表的`"HKEY_CURRENT_USER\\Software\\LeapWare\\Registry\\LeapFTP"`下

### 一组可用注册码

用户名:`ShenGe[BCG]`

注册码:`ABC1-EF2G-I3JK-NANA`

Cracked By `ShenGe[BCG]`

```
4dd466d0e1e0cc123afbe504da2462ca
```
