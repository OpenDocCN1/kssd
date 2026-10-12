
# AxMan 3.10

> 原文：[`www.pediy.com/kssd/pediy02/forum171.htm`](https://www.pediy.com/kssd/pediy02/forum171.htm)

## 菜鸟破解录(12)之 AxMan 3.10

**软件名称**：`AxMan`

**版本**：`3.10`

**简介**：
一个非常不错的文件分割工具，该工具的主要作用是可以将大文件切成几个小文件便于拷贝到软盘或作为电子邮件附件发送。对于分割的文件，程序提供自动合并功能，同时该工具也提供非常简单的操作界面和操作方式以方便用户使用。

**作者**：`xiA Qin`

**等级**：很菜....很菜....

**破解日期**：2000 年 7 月 24 日

**破解工具**：`Trw2000 1.22`

**破解目的**：学习注册码的破解(简单)

**说明**：
本文是在我的软件破解记录上整理出来的，只作技术交流。如若有纰漏，请各位大侠多指教！

运行程序，输入注册信息。

- `name:` `xiA Qin` & 任意输入
- `Company:` `China` & 任意输入
- `Registration:` `1234567890` & 任意输入

下指令 `bpx hmemcpy` // 下中断点

按 `F5` 回到程序，按 `OK`，这时会被 `Trw2000` 拦截到。

下指令 `bd *` // 屏障中断点

下指令 `pmodule` // 直接跳到程序的领空

按 `F10` 来到下面的代码。

```
015F:004063A9  LEA     EAX,[EBP-18]       // 载入 name:xiA Qin
015F:004063AC  LEA     ECX,[ESI+000000D4]
015F:004063B2  PUSH    EAX
015F:004063B3  CALL    0042B1F4
015F:004063B8  LEA     EAX,[EBP-14]       // 载入 Company：China
015F:004063BB  LEA     ECX,[ESI+5C]
015F:004063BE  PUSH    EAX
015F:004063BF  CALL    0042B1F4
015F:004063C4  LEA     ECX,[EBP-1C]       // 载入 Registration:1234567890
015F:004063C7  CALL    00405F6D
015F:004063CC  PUSH    DWORD PTR [EBP-14]
015F:004063CF  LEA     ECX,[EBP-1C]
015F:004063D2  MOV     BYTE PTR [EBP-04],03
015F:004063D6  PUSH    DWORD PTR [EBP-18]
015F:004063D9  PUSH    DWORD PTR [EBP-10]
015F:004063DC  CALL    00405F99          <-- 有问题，按 F8 进入看一看。
015F:004063E1  TEST    AL,AL
015F:004063E3  JNZ     004063FD          // 注册码不相等，就不跳。
```

### 进入 004063DC CALL 00405F99

按 `F8` 进入 `004063DC CALL 00405F99` 后

按 `F10` 来到下面的指令

```
015F:00405F99  PUSH    EBP
015F:00405F9A  MOV     EBP,ESP
015F:00405F9C  SUB     ESP,18
015F:00405F9F  LEA     EAX,[EBP-18]
015F:00405FA2  PUSH    EAX
015F:00405FA3  PUSH    DWORD PTR [EBP+0C]
015F:00405FA6  PUSH    DWORD PTR [EBP+08]
015F:00405FA9  CALL    00405FC6
015F:00405FAE  PUSH    DWORD PTR [EBP+10]
015F:00405FB1  LEA     EAX,[EBP-18]       // 正确的注册码 D EAX
015F:00405FB4  PUSH    EAX
015F:00405FB5  CALL    00418FC0
015F:00405FBA  NEG     EAX
015F:00405FBC  POP     ECX
015F:00405FBD  SBB     AL,AL
015F:00405FBF  POP     ECX
015F:00405FC0  INC     AL
015F:00405FC2  LEAVE
015F:00405FC3  RET     000C
```

### 返回主流程

```
015F:004063E5  PUSH    30
015F:004063E7  PUSH    0045FAB0
015F:004063EC  PUSH    0045FA8C
015F:004063F1  MOV     ECX,ESI
015F:004063F3  CALL    0042BF41          // 注册失败对话框
015F:004063F8  JMP     0040648B
015F:004063FD  LEA     EAX,[EBP-01AC]
015F:00406403  PUSH    0045FA5C
015F:00406408  PUSH    EAX
015F:00406409  CALL    00419B90
015F:0040640E  LEA     EAX,[EBP-01AC]
015F:00406414  PUSH    0045FA08
015F:00406419  PUSH    EAX
015F:0040641A  CALL    00419BA0
015F:0040641F  ADD     ESP,10
015F:00406422  LEA     EAX,[EBP-01AC]
015F:00406428  MOV     ECX,ESI
```

整理一下，输入注册信息。

- `name:` `xiA Qin`
- `Company:` `China`
- `Registration:` `130-464-421 4dd466d0e1e0cc123afbe504da2462ca`
