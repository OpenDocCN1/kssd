
# HAPPYEO 电子琴 2.4

> 原文：[`www.pediy.com/kssd/pediy03/forum1246.htm`](https://www.pediy.com/kssd/pediy03/forum1246.htm)

用 `TRW 2000` 破解 `HAPPYEO 电子琴 2.4` - 音频工具

作者: `X man`

等级：`easy`

授权方式： 共享软件

运行平台： `Win9x/Me/NT/20000`

官方主页： [http://www.happyeo.com/](http://www.happyeo.com/)

编辑评定：

功能简介： 一个用电脑的声卡和键盘来模拟电子琴的程序。主要功能特点：有两个独立的键盘，用左右手分别演奏。可分别设置不同的音色、音区和效果。128 种音色，复音数仅受声卡限制。12 种效果！(包括自动双音、自动和弦、双音色混合，甚至还有模拟弹拨乐器的音色）。有自动伴奏功能！12 种自动伴奏风格，支持节奏填充、节奏同步启动。可以录音，放音，暂停，快放，循环放音。放音的同时仍然可以正常演奏！同时支持声卡和喇叭。即使使用喇叭，也有四种不同的“音色”。界面逼真，操作方便，同时支持鼠标和键盘。帮助详尽。绿色软件，无需安装。

### 1. 文件检测
把《HAPPYEO 电子琴》主程序 `happyeo.exe` 拷到文件内型检测 `fi` 目录下进行文件是否加壳检测，还好没有任何的壳。

### 2. 运行程序
运行《HAPPYEO 电子琴》，点右键选“注册”。

马上弹出对话框，填入以下信息：

用户名：`lbcoolboy`
地址：`lbcoolboylbcoolboy`
注册码：`123456`

（顺便说一下，《HAPPYEO 电子琴》好想很为我们 `CRACKER` 着想，还提示用户名、地址、注册码的限定条件哟！^\_^）

### 3. 加载调试器
加载 `TRW2K`，按下热键 `CTRL+N`。

```
BPX HMEMCPY
G
```

### 4. 设置断点
回到《HAPPYEO 电子琴》点“确定”，马上被 `TRW2K` 截下。

```
BD *
PMODULE
```

按下 `F12` 七次（按八下就会出错对话框，说明按七下就到了接近计算注册码的地址），来到：

```
016F:510986A6  ADD     [EAX],AL
016F:510986A8  POP     ES
016F:510986A9  ADD     [EAX],AL
016F:510986AB  ADD     [EBP-75],DL
016F:510986AE  IN      AL,DX
016F:510986AF  MOV     ECX,06
016F:510986B4  PUSH    BYTE +00
016F:510986B6  PUSH    BYTE +00
016F:510986B8  DEC     ECX
016F:510986B9  JNZ     510986B4
016F:510986BB  PUSH    EBX
016F:510986BC  MOV     EBX,EAX
016F:510986BE  XOR     EAX,EAX
016F:510986C0  PUSH    EBP
016F:510986C1  PUSH    DWORD 51098876
016F:510986C6  PUSH    DWORD [FS:EAX]
016F:510986C9  MOV     [FS:EAX],ESP
016F:510986CC  LEA     EDX,[EBP-04]
016F:510986CF  MOV     EAX,[EBX+02D0]
016F:510986D5  CALL    510312E0
016F:510986DA  MOV     EAX,[EBP-04]
016F:510986DD  CALL    51003F28
016F:510986E2  CMP     EAX,BYTE +04 // 用户名不小于4个字符，否则出错
016F:510986E5  JL      NEAR 5109883C
016F:510986EB  LEA     EDX,[EBP-08]
016F:510986EE  MOV     EAX,[EBX+02EC]
016F:510986F4  CALL    510312E0
016F:510986F9  MOV     EAX,[EBP-08]
016F:510986FC  CALL    51003F28
016F:51098701  CMP     EAX,BYTE +0C // 地址不得小于12个字符，否则出错
016F:51098704  JL      NEAR 5109883C
016F:5109870A  LEA     EDX,[EBP-0C]
016F:5109870D  MOV     EAX,[EBX+02D4]
016F:51098713  CALL    510312E0
016F:51098718  MOV     EAX,[EBP-0C]
016F:5109871B  CALL    51003F28
016F:51098720  CMP     EAX,BYTE +06 // 注册码必须是6个字符，否则出错
016F:51098723  JNZ     NEAR 5109883C
016F:51098729  LEA     EDX,[EBP-10]
016F:5109872C  MOV     EAX,[EBX+02D4]
016F:51098732  CALL    510312E0
016F:51098737  MOV     EAX,[EBP-10]
016F:5109873A  PUSH    EAX
016F:5109873B  LEA     EDX,[EBP-18]
016F:5109873E  MOV     EAX,[EBX+02EC]
016F:51098744  CALL    510312E0
016F:51098749  MOV     EAX,[EBP-18]
016F:5109874C  PUSH    EAX
016F:5109874D  LEA     EDX,[EBP-1C]
016F:51098750  MOV     EAX,[EBX+02D0]
016F:51098756  CALL    510312E0
016F:5109875B  MOV     EAX,[EBP-1C]
016F:5109875E  LEA     ECX,[EBP-14]
016F:51098761  POP     EDX
016F:51098762  CALL    51097B40
016F:51098767  MOV     EDX,[EBP-14]
016F:5109876A  POP     EAX // d eax 得到 OWXEEH
                         // 6个字符？难到是注册码？
                         // 没错，别怀疑！
016F:5109876B  CALL    51004038 // 很可疑，进去看看（注册码的比较）
016F:51098770  JNZ     NEAR 51098826
```

进入 `CALL 51004038`

```
016F:51004038  PUSH    EBX
016F:51004039  PUSH    ESI
016F:5100403A  PUSH    EDI
016F:5100403B  MOV     ESI,EAX
016F:5100403D  MOV     EDI,EDX
016F:5100403F  CMP     EAX,EDX // D EAX
                                // D EDX分别得到真假注册码
                                // 比较真注册码和假注册码
016F:51004041  JZ      NEAR 510040D6
..........
```

OK，现在就完成了《HAPPYEO 电子琴》的破解

好吧，收工！对了我的注册码是：

`NAME`: `lbcoolboy`
地址：`lbcoolboylbcoolboy`
注册码：`OWXEEH`

------------------------------------------------------------

### 后记
软件注册后在 `HKEY_CURRENT_USER\Software\HappyEO2.4` 下建立注册信息 `4dd466d0e1e0cc123afbe504da2462ca`
