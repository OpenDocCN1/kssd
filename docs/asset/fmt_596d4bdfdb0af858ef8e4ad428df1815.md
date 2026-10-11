
# Advanced Direct Remailer 2.12

> 原文：[`www.pediy.com/kssd/pediy03/bc004.htm`](https://www.pediy.com/kssd/pediy03/bc004.htm)

## Asprotect 保护的程序脱壳后的修正--DialogBoxIndirectParamA

Liotta\[BCG\]

本文主要介绍 `Asprotect1.3` 保护的程序脱壳后关于加密函数 `DialogBoxIndirectParamA` 的修正问题。

仅供新手参考。

### 基本信息

*   **名称**: `Advanced Direct Remailer`
*   **版本**: `2.12`
*   **发布日期**: June 1, 2001
*   **描述**: Powerful remailer with SMTP server, support for mailing lists, plugins.
*   **操作系统**: `Windows 98`
*   **目标文件**: `Adr.exe`

### 相关工具

*   `FI`
*   `Peditor`
*   `Softice`
*   `icedump` + `iceload`
*   `ImpREC`

### 参考文档

*   `fs0`: “脱 `Advanced Email Extractor PRO` 的壳”
*   `Liotta[BCG]`: “UNPack `CommView` v.3.0”

---

`Asprotect` 保护的程序，当我们手动脱壳后发现程序总是无法运行，要么自动退出、要么出现加密函数未找到等出错信息。总是不能顺顺利利的，对于我们新手来说能够完成手动脱壳已是不易，再出现这种问题对我们来说，实在是一大打击！

如何修正这个问题呢？

`fs0` 在“脱 `Advanced Email Extractor PRO` 的壳”一文中提到：

用 `Asprotect` 加壳的程序，脱壳后不能运行，我知道的有:

1.  原程序调用 `Asprotect` 的"Export Function", 最简单的就是:
    ```
    push ..
    push -1
    call GetProcAddress
    ```
    看其返回值是否为 0，为 0 就出错或退出(`AeePro` 就是这样的)
2.  原程序自检加壳后的程序(已经加壳了, 真不知道他是怎么检的)。
3.  `Asprotect` 调用原程序的“初始化”函数, 当到达 `OEP` 前, 某些全局变量已经初始化了, 脱壳后运行就很可能出错, 解决方法是, 修改脱壳后的程序, 先运行该“初始化”函数, 再跳到 `OEP` 运行就可以了。
4.  再有就是 `DialogBoxIndirectParamA` 等。

现在结合 `ADR` 脱壳后的修正来说说这个 `DialogBoxIndirectParamA` 问题，

首先，要找到程序的 `OEP` 并 DUMP，然后重构输入表并修改 DUMP 档。如果你还不大清楚 `Asprotect` 保护脱壳请细阅上述参考文档。

当运行我们脱壳后的程序时，却出现“Crypt API not found.……”错误信息框，然后就退出。

让我们 `BPX MessageBoxA`，按 F12 返回到程序空间，向上看看：

```
017F:00433590 6A04           PUSH   BYTE +04
017F:00433592 6AFF           PUSH   BYTE -01
017F:00433594 FF154C014400   CALL   `KERNEL32!GetProcAddress`
017F:0043359A 85C0           TEST   EAX,EAX          <--EAX=0 则出错
017F:0043359C A36C564600     MOV    [0046566C],EAX
017F:004335A1 7516           JNZ    004335B9         <--75 改成 EB 即可修正
017F:004335A3 6A10           PUSH   BYTE +10
017F:004335A5 68E0424400     PUSH   DWORD 004442E0
017F:004335AA 68C4774400     PUSH   DWORD 004477C4
017F:004335AF 50             PUSH   EAX
017F:004335B0 FF15A4034400   CALL   `USER32!MessageBoxA`   <--出错信息框
017F:004335B6 33C0           XOR    EAX,EAX
017F:004335B8 C3             RET                 <--退出
```

看到了吧！用 HEX 编辑器修改 `75` 成 `EB` 即可修正最简单的错误。

修正后，程序能运行。但是当我们点击 `HELP` 中的 `Registration` 时却出现严重错误！！

我们用 `W32DASM` 来反汇编该程序，发现 `HELP` 中的 `Registration` 调用 `USER32.dll!DialogBoxIndirectParamA`

可以肯定是函数 `DialogBoxIndirectParamA` 在搞鬼！并发现有几处调用该函数。

#### 其中之（一）

```
* Reference To: USER32.DialogBoxIndirectParamA, Ord:0091h
                         |
:00434713 FF1568034400    Call dword ptr [00440368]
```

地址 `00440368` 存放的就是 `USER32.dll!DialogBoxIndirectParamA` 函数调用地址。

让我们先看看，脱壳是找到的 `USER32.dll!DialogBoxIndirectParamA` 函数原型：

```
017F:00F5C898 55           PUSH   EBP
017F:00F5C899 8BEC         MOV    EBP,ESP
017F:00F5C89B 53           PUSH   EBX
017F:00F5C89C 8B5D08       MOV    EBX,[EBP+08]
017F:00F5C89F 8B4518       MOV    EAX,[EBP+18]
017F:00F5C8A2 50           PUSH   EAX
017F:00F5C8A3 8B4514       MOV    EAX,[EBP+14]
017F:00F5C8A6 50           PUSH   EAX
017F:00F5C8A7 8B4510       MOV    EAX,[EBP+10]
017F:00F5C8AA 50           PUSH   EAX
017F:00F5C8AB 6A05         PUSH   BYTE +05
017F:00F5C8AD 8B450C       MOV    EAX,[EBP+0C]
017F:00F5C8B0 50           PUSH   EAX
017F:00F5C8B1 53           PUSH   EBX
017F:00F5C8B2 E8157BFFFF   CALL   `KERNEL32!FindResourceA`   <--注意机器码(5 字节)
017F:00F5C8B7 50           PUSH   EAX
017F:00F5C8B8 53           PUSH   EBX
017F:00F5C8B9 E87E7BFFFF   CALL   `KERNEL32!LoadResource`     <--注意机器码(5 字节)
017F:00F5C8BE 50           PUSH   EAX
017F:00F5C8BF E8807BFFFF   CALL   `KERNEL32!LockResource`     <--注意机器码(5 字节)
017F:00F5C8C4 50           PUSH   EAX
017F:00F5C8C5 53           PUSH   EBX
017F:00F5C8C6 E8917BFFFF   CALL   `USER32!DialogBoxIndirectParamA`<--注意机器码(5 字节)
017F:00F5C8CB 5B           POP    EBX
017F:00F5C8CC 5D           POP    EBP
017F:00F5C8CD C21400       RET    14
```

但由于我们已重构输入表，故上述函数要稍作修改才能引用。

在用 `ImpREC` 重构输入表时我们看到相关的几项

```
Flag   RVA       ModuleName    Ordinal   Name
1      00040138  KERNEL32.dll  0123      FindResourceA
1      0004012C  KERNEL32.dll  023E      LockResource
1      00040130  KERNEL32.dll  022E      LoadResource
1      00040368  USER32.dll    0091      DialogBoxIndirectParamA
```

就可知：

*   `KERNEL32!FindResourceA` 函数调用地址存放在 `00040138+400000=00440138`
*   `KERNEL32!LoadResource` 函数调用地址存放在 `00440130`
*   `KERNEL32!LockResource` 函数调用地址存放在 `0044012C`
*   `USER32!DialogBoxIndirectParamA` 函数调用地址存放在 `00440368`

所以我们作以下修改

```
017F:004D0F98 55           PUSH   EBP
017F:004D0F99 8BEC         MOV    EBP,ESP
017F:004D0F9B 53           PUSH   EBX
017F:004D0F9C 8B5D08       MOV    EBX,[EBP+08]
017F:004D0F9F 8B4518       MOV    EAX,[EBP+18]
017F:004D0FA2 50           PUSH   EAX
017F:004D0FA3 8B4514       MOV    EAX,[EBP+14]
017F:004D0FA6 50           PUSH   EAX
017F:004D0FA7 8B4510       MOV    EAX,[EBP+10]
017F:004D0FAA 50           PUSH   EAX
017F:004D0FAB 6A05         PUSH   BYTE +05
017F:004D0FAD 8B450C       MOV    EAX,[EBP+0C]
017F:004D0FB0 50           PUSH   EAX
017F:004D0FB1 53           PUSH   EBX
017F:004D0FB2 FF1538014400 CALL   `KERNEL32!FindResourceA`    <--注意机器码(6 字节)
017F:004D0FB8 50           PUSH   EAX
017F:004D0FB9 53           PUSH   EBX
017F:004D0FBA FF1530014400 CALL   `KERNEL32!LoadResource`     <--注意机器码(6 字节)
017F:004D0FC0 50           PUSH   EAX
017F:004D0FC1 FF152C014400 CALL   `KERNEL32!LockResource`     <--注意机器码(6 字节)
017F:004D0FC7 50           PUSH   EAX
017F:004D0FC8 53           PUSH   EBX
017F:004D0FC9 FF1568034400 CALL   `USER32!DialogBoxIndirectParamA`<--注意机器码(6 字节)
017F:004D0FCF 5B           POP    EBX
017F:004D0FD0 5D           POP    EBP
017F:004D0FD1 C21400       RET    14
```

然后我们来打补丁！找一块大小合适没有使用空间，我在用 `ImpREC` 重构输入表时所加的段中选了一块

```
Offset     0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F
000D0F90   98 0F 4D 00 00 00 00 00  55 8B EC 53 8B 5D 08 8B  ?M.....U 嬱 S 媇.?
000D0FA0   45 18 50 8B 45 14 50 8B  45 10 50 6A 05 8B 45 0C  E.P 婨.P 婨.Pj.婨.
000D0FB0   50 53 FF 15 38 01 44 00  50 53 FF 15 30 01 44 00  PS�.8.D.PS�.0.D.
000D0FC0   50 FF 15 2C 01 44 00 50  53 FF 15 68 03 44 00 5B  P�.,.D.PS�.h.D.[
000D0FD0   5D C2 14 00 00 00 00 00  00 00 00 00 00 00 00 00  ]?.............
```

`Offset000D0F90` 存放的是函数调用地址 `004D0F98`，并在 `Offset000D0F98` 中写入修正后函数的机器码。

最后把原先找到的几处调用中 `Call dword ptr [00440368]` 改为 `[004D0f90]`

#### 其中之（一）

```
* Reference To: USER32.DialogBoxIndirectParamA, Ord:0091h
                         |
:00434713 FF1568034400    Call dword ptr [004D0f90]
```

终于大功告成！点击 `HELP` 中的 `Registration` 弹出 `Registration` 窗口。`Asprotect` 保护的程序脱壳后的修正就讲到这里。希望对大家有帮助！

`4dd466d0e1e0cc123afbe504da2462ca`
