
# Virtual Driver 2000 (tm) Version 5.0

> 原文：[`www.pediy.com/kssd/pediy03/forum582.htm`](https://www.pediy.com/kssd/pediy03/forum582.htm)

## 一、制作注册版过程

当出现提示框"Virtual Drive Evaluation Version .Please click "OK" to contiue "时，在 `Ctrl+n` 进入 `trw` 中，执行 `:pmodule` (返回)。

按下“OK”鍵，程序中断如下: (分析可见旁边注释)

```
0177:0040C632  LEA     ECX,[EBP+FFFFFF10]
0177:0040C638  MOV     BYTE [EBP-04],11
0177:0040C63C  CALL    `ZEN!?GetDWORD@ZRegBase@@QAEKH@Z`
0177:0040C642  CMP     DWORD [0042BA08],BYTE +00    //与 42BA08 中比较
0177:0040C649  MOV     ESI,EAX
0177:0040C64B  JZ      0040C65B                     //相等则可跳过那个提示框:)
0177:0040C64D  PUSH    BYTE -01
0177:0040C64F  PUSH    BYTE +00
0177:0040C651  PUSH    DWORD 01C8
0177:0040C656  CALL    `MFC42!ord_000004AF`  //提示框"Virtual  Drive Evaluation Version...."
0177:0040C65B  TEST    ESI,ESI                 //你会在这儿中断，请向上看
0177:0040C65D  JZ      NEAR 0040C838
0177:0040C663  MOV     EAX,[EBP-14]
0177:0040C666  CMP     DWORD [EAX+0178],BYTE +00
```

在 `0040c642` 处可下 `d 42BA08` 可见这个内存中的值为 `15`，我将其改为 `0`。再回到程序中，此时的 `mgr` 已经是注册版了。我想知道程序是在哪儿给 `42BA08` 赋值的,于是下 `bpm 42BA08`。

## 二、设好断点

设置断点 `:bpm 42ba08`，重新运行 `mgr.exe`，程序果然中断。

```
0177:0040BAB1  PUSH    DWORD 1393
0177:0040BAB6  LEA     ECX,[EBP-20]
0177:0040BAB9  CALL    `ZEN!?GetDWORD@ZRegBase@@QAEKH@Z`
0177:0040BABF  MOV     ECX,EAX
0177:0040BAC1  AND     ECX,FF00
0177:0040BAC7  CMP     ECX,9900
0177:0040BACD  JNZ     0040BADB
0177:0040BACF  MOV     ECX,1113
0177:0040BAD4  AND     EAX,FF
0177:0040BAD9  JMP     SHORT 0040BAF3      //如果跳到 0040BAF1，哈哈。。
0177:0040BADB  CMP     ECX,8800
0177:0040BAE1  JNZ     0040BAEF
0177:0040BAE3  MOV     ECX,4811
0177:0040BAE8  AND     EAX,FF
0177:0040BAED  JMP     SHORT 0040BAF3
0177:0040BAEF  MOV     ECX,EAX
0177:0040BAF1  XOR     EAX,EAX             //EAX 清零^\_^
0177:0040BAF3  MOV     [ESI+D0],EAX      //给 42BA08 赋值:),再向上看
0177:0040BAF9  CMP     ECX,1113          //你会在这儿中断
0177:0040BAFF  JZ      0040BB54
0177:0040BB01  CMP     ECX,4811
0177:0040BB07  JZ      0040BB60
0177:0040BB09  PUSH    DWORD 0042B3B4
0177:0040BB0E  LEA     EAX,[EBP-10]
0177:0040BB11  PUSH    DWORD 1B5B
0177:0040BB16  PUSH    EAX
0177:0040BB17  CALL    `MFC42!ord_00000B01`
```

## 三、修补程序

用 `hiew` 将 `0040BA09` 改为 `JMP SHORT 0040BAF1`，此时 `mgr` 就被 patch 为注册版的了。只需改动一个字节。^\_^

\*\*\*\*\* 程序早就破过了，应 `peterchen` 要求,写了这个简要的过程，希望大家能够喜欢!

十三少
China Cracking Group
2000.2.2 `4dd466d0e1e0cc123afbe504da2462ca`
