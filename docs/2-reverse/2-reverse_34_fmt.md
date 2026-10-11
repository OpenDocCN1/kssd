
# OD 如何 d fs:0???

> 原文：[www.pediy.com/kssd/pediy08/pediy8-827.htm](https://www.pediy.com/kssd/pediy08/pediy8-827.htm)

提示语法错误

注意，不是要 `d fs:[0]`

看 `od` 的寄存器窗口:

```
EAX 00000000
ECX 00010101
EDX FFFFFFFF
EBX 7FFDF000
ESP 000AFFC4
EBP 000AFFF0
ESI 00000000
EDI 00000000
EIP 00410070 loaddll.<ModuleEntryPoint>
C 0  ES 0023 32bit 0(FFFFFFFF)
P 1  CS 001B 32bit 0(FFFFFFFF)
A 0  SS 0023 32bit 0(FFFFFFFF)
Z 1  DS 0023 32bit 0(FFFFFFFF)
S 0  FS 0038 32bit 7FFDE000(FFF)
T 0  GS 0000 NULL
D 0
O 0  LastErr ERROR_MOD_NOT_FOUND (0000007E)
EFL 00000246 (NO,NB,E,BE,NS,PE,GE,LE)
ST0 empty 0.0
ST1 empty 0.0
ST2 empty 0.0
ST3 empty 0.0
ST4 empty 0.0
ST5 empty 0.0
ST6 empty 0.0
ST7 empty -UNORM D730 00000020 00000000
                   3 2 1 0      E S P U O Z D I
FST 0000  Cond 0 0 0 0  Err 0 0 0 0 0 0 0 0  (GT)
FCW 027F  Prec NEAR,53  Mask    1 1 1 1 1 1
```

`fs` = `0038`, 经 `GDT` (好像这里应该是 `LDT` 吧) 映射到线性地址 `7FFDE000`

因此 `fs:0` = `7FFDE000`

(我是 `win2000` 系统)
