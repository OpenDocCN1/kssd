
# Archpr1.01

> 原文：[`www.pediy.com/kssd/pediy03/forum1299.htm`](https://www.pediy.com/kssd/pediy03/forum1299.htm)

作个自我介绍先，我叫 Liotta，CRACK 初学者，请多关照。

`Advanced Archive Password Recovery`
Version 1.01
`archpr.exe`

`Entry Point    :00001000    Image Base    :00400000`
`Base of Data    :00033000    Size of Image    :00377000`

Tools used :
`FI`
`Peditor`
`Softice`
`icedump + iceload`
`ImpREC`

## 一，脱壳

用 `FI` 查一查 `Archpr1.01` 的保护方式，显示是用 `Asprotect.V1.3` 加壳的，现在让我们来找到 `Archpr` 的 OEP。找 OEP 方法请参考 ljttt“对 Asprotect 脱壳的一点总结”一文。我是这样做的：

先 `BPX GetVersion` 中断进入加壳程序，用 `icedump` 中的 `/tracex 400000 eip-8` 来接近并找到 OEP，
`Archpr` 的 OEP 是 `00001000`。然后用 `/dump 400000 377000 c:\dump.dmp`，再用 `Peditor` 的 `dumpfixer`
来修改。

## 二，用 ImpREC 重构输入表

在 `ImpREC` 中 "Attach to an Active Process" 下拉框中选择目标 `Archpr`，按 "IAT AutoSearch"
"Get Import"、"Auto Trace" 并 "Show Invalid" 是发现：

```
……
    00033530    USER32.dll    0090    DestroyWindow
(1)    00033534    KERNEL32.dll    0123    FindResourceA    (注 PRT 0092C898)
    00033538    USER32.dll    0095    DispatchMessageA
……
    0003369C    KERNEL32.dll    0133    FreeLibrary
(2)    000336A0    ?    0000    0092C87C
(3)    000336A4    ?    0000    0092C86C
    000336A8    KERNEL32.dll    0158    GetCurrentDirectoryA
(4)    000336AC    ?    0000    0092C85C
    000336B0    KERNEL32.dll    015C    GetCurrentThread
……
    00033734    KERNEL32.dll    0238    LocalReAlloc
(5)    00033738    ?    0000    0092C874
    0003373C    KERNEL32.dll    0251    MulDiv
……
```

怎么 `USER32.dll` 中出现 `KERNEL32.dll` 的函数，确定有误！右键选中并 invalidate 掉它。

记下这些未知函数的 `RVA` 和 `PTR`，然后设断 `BPX GetVersion` 并再次装入目标程序。

`:u 92c7d8 l 92c92e-92c7d8`

```
017F:0092C7D8  6A00                PUSH      00
017F:0092C7DA  E82D7CFFFF          CALL      KERNEL32!GetModuleHandleA
017F:0092C7DF  A3D4359300          MOV       [009335D4],EAX
017F:0092C7E4  E83B7CFFFF          CALL      KERNEL32!GetVersion    <--中断
017F:0092C7E9  A3D8359300          MOV       [009335D8],EAX
017F:0092C7EE  68E4359300          PUSH      009335E4
017F:0092C7F3  E8347CFFFF          CALL      KERNEL32!GetVersionExA
017F:0092C7F8  E8DF7BFFFF          CALL      KERNEL32!GetCurrentProcess
017F:0092C7FD  A3DC359300          MOV       [009335DC],EAX    <(4)--009335DC
017F:0092C802  E8DD7BFFFF          CALL      KERNEL32!GetCurrentProcessId
017F:0092C807  A3E0359300          MOV       [009335E0],EAX
017F:0092C80C  E8C37BFFFF          CALL      KERNEL32!GetCommandLineA
017F:0092C811  A378369300          MOV       [00933678],EAX    <(3)--00933678
017F:0092C816  C3                  RET
```

对于每项未知函数都先 `u PTR` 看看。

如(4)：`u 0092C85C`

```
017F:0092C85C  A1DC359300          MOV       EAX,[009335DC]    <(4)--009335DC
017F:0092C861  C3                  RET
```

如(3): `u 0092C86C`

```
017F:0092C86C  A178369300          MOV       EAX,[00933678]    <(3)
017F:0092C871  C3                  RET
```

看到的吧！
(3)号未知函数是 `KERNEL32!GetCommandLineA`
(4)号未知函数是 `KERNEL32!GetCurrentProcess`

如果我们跟进以下任一函数，就可发现一些有意思的 `JMP` 表：

`:u 92437c l 9244a3-92437c`

```
017F:0092437C  FF2534419300        JMP       [ADVAPI32!RegCloseKey]    <--反过来 0093413425
                          ^^^^^^^^^^
017F:00924382  8BC0                MOV       EAX,EAX
017F:00924384  FF2530419300        JMP       [ADVAPI32!RegCreateKeyExA]
017F:0092438A  8BC0                MOV       EAX,EAX
017F:0092438C  FF252C419300        JMP       [ADVAPI32!RegFlushKey]
017F:00924392  8BC0                MOV       EAX,EAX
```

比较困难一点是另外一种情况。

如(1): `u 0092C898`

```
017F:0092C874  55                  PUSH      EBP    <--(5)
017F:0092C875  8BEC                MOV       EBP,ESP
017F:0092C877  5D                  POP       EBP
017F:0092C878  C20400              RET       0004
017F:0092C87B  90                  NOP
017F:0092C87C  55                  PUSH      EBP    <--(2)
017F:0092C87D  8BEC                MOV       EBP,ESP
017F:0092C87F  5D                  POP       EBP
017F:0092C880  C20400              RET       0004
017F:0092C883  90                  NOP
017F:0092C884  55                  PUSH      EBP
017F:0092C885  8BEC                MOV       EBP,ESP
017F:0092C887  8B450C              MOV       EAX,[EBP+0C]
017F:0092C88A  83C004              ADD       EAX,04
017F:0092C88D  8B00                MOV       EAX,[EAX]
017F:0092C88F  034508              ADD       EAX,[EBP+08]
017F:0092C892  5D                  POP       EBP
017F:0092C893  C20800              RET       0008
017F:0092C896  8BC0                MOV       EAX,EAX
017F:0092C898  55                  PUSH      EBP    <--(1)
017F:0092C899  8BEC                MOV       EBP,ESP
017F:0092C89B  53                  PUSH      EBX
017F:0092C89C  8B5D08              MOV       EBX,[EBP+08]
017F:0092C89F  8B4518              MOV       EAX,[EBP+18]
017F:0092C8A2  50                  PUSH      EAX
017F:0092C8A3  8B4514              MOV       EAX,[EBP+14]
017F:0092C8A6  50                  PUSH      EAX
017F:0092C8A7  8B4510              MOV       EAX,[EBP+10]
017F:0092C8AA  50                  PUSH      EAX
017F:0092C8AB  6A05                PUSH      05
017F:0092C8AD  8B450C              MOV       EAX,[EBP+0C]
017F:0092C8B0  50                  PUSH      EAX
017F:0092C8B1  53                  PUSH      EBX
017F:0092C8B2  E8157BFFFF          CALL      KERNEL32!FindResourceA
017F:0092C8B7  50                  PUSH      EAX
017F:0092C8B8  53                  PUSH      EBX
017F:0092C8B9  E87E7BFFFF          CALL      KERNEL32!LoadResource
017F:0092C8BE  50                  PUSH      EAX
017F:0092C8BF  E8807BFFFF          CALL      KERNEL32!LockResource
017F:0092C8C4  50                  PUSH      EAX
017F:0092C8C5  53                  PUSH      EBX
017F:0092C8C6  E8917BFFFF          CALL      USER32!DialogBoxIndirectParamA    <--(1)
017F:0092C8CB  5B                  POP       EBX
017F:0092C8CC  5D                  POP       EBP
017F:0092C8CD  C21400              RET       0014
```

(1)号未知函数是 `USER32!DialogBoxIndirectParamA`
(2)号未知函数是 `KERNEL32.dll!FreeResource` ？？ 请指点！
(5)号未知函数是 `KERNEL32.dll!LockResource` ？？ 请指点！

其实我自已并不明白(2)(5)号未知函数是如何获得的！只是参考了 NchantA[PGC][EVC]写
“Unpacking asprotect”一文。请知情者多多指点！

修复完输入表后，就可 "Fix Dump"，为了避免不必要的麻烦我们还是 "add new section" 吧。

如此脱壳后的程序运行是能运行，不过在出现‘信息框’时总是出错！唉，失败。

不过，写这篇文章的目的就是要抛砖引玉，请各位多多指点，Thanks。

`4dd466d0e1e0cc123afbe504da2462ca`
