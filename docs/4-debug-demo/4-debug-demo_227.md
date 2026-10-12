
# ACDSEE4.0

> 原文：[`www.pediy.com/kssd/pediy04/kanxue352.htm`](https://www.pediy.com/kssd/pediy04/kanxue352.htm)

ACDSEE 这个看图软件相信大伙都知道了。本人前些时候安装了它的 4.0 版，今天出现过期提示不能用了。只好拿出 `SOFTICE`、`HIEW` 来，目的——暴力破解 ACDSEE V4.0。以下是破解笔记，我也懒得整理了，大家凑合着看吧，不足之处请指出，谢谢。

用 `BPX HMEMCPY` 中断后，走到以下：

```
017F:0040B955 8D442418       LEA      EAX,[ESP+18]
017F:0040B959 52             PUSH     EDX
017F:0040B95A 8D4C2474       LEA      ECX,[ESP+74]
017F:0040B95E 50             PUSH     EAX
017F:0040B95F 51             PUSH     ECX
017F:0040B960 8B0D74535400   MOV      ECX,[00545374]
017F:0040B966 FF1590535400   CALL     `LICENSE!?LicenseKey_GetInfo@CLicense@@QAEHPBDPAH111@Z`
017F:0040B96C 8B0D74535400   MOV      ECX,[00545374]
017F:0040B972 FF1578535400   CALL     `LICENSE!?GetLicenseDLLType@CLicense@@QAEHXZ`----进入
017F:0040B978 85C0           TEST     EAX,EAX
017F:0040B97A 7416           JZ       0040B992------ 改为 NOP
017F:0040B97C 39742408       CMP      [ESP+08],ESI
017F:0040B980 7510           JNZ      0040B992-------改为 NOP
017F:0040B982 5F             POP      EDI
017F:0040B983 B801000000     MOV      EAX,01
017F:0040B988 5E             POP      ESI
017F:0040B989 81C4B0000000   ADD      ESP,B0
017F:0040B98F C20800         RET      08
```

进入 `40B966` 的 `CALL` 发现如下：

```
017F:10001CDB 90             NOP
017F:10001CDC 90             NOP
017F:10001CDD 90             NOP
017F:10001CDE 90             NOP
017F:10001CDF 90             NOP
017F:10001CE0 33C0           XOR      EAX,EAX -改为 MOV AX，1
017F:10001CE2 C3             RET      -----改为 RET
```

经过以上改动后，程序出现 NAP 提示，发现把 `40B010` 处的

```
E872C41200
85C0
7431 ----------------改为 7531
8B0D38685700
6A00
```

要注意的是：用二进制编辑软件编辑时，此处的 `7431` 显示出来的是 `CC31`，（我也不知是何故），不理会它，照改成 `7531` 即可。

至此，程序已完全破解。唯一不足的是在 `About Acdsee` 菜单下的：`This software is licensed to :`及 `License number:`后面是空白的。 `4dd466d0e1e0cc123afbe504da2462ca`
