
# 手工修复 Armadillo 壳中的 CC 方法之一

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-540.htm`](https://www.pediy.com/kssd/pediy08/pediy8-540.htm)

准备--必须有完成了代码和 iat 表修复后的 dump 文件

工具--修改后的 OD 等

试验对象：Easy CD-DA Extractor

下载地址：`http://www.hanzify.org/?Go=Show::List&ID=7377`

试验中使用 `machenglin` 提供的 dump 文件

让我们开始：

### 1. 获得修复 CC 环境

已知主程序 `ezcddax.exe` 是 Armadillo 壳，且使用了 CC 保护方式。OD 加载主程序 `ezcddax.exe` 在 Command 窗口中：`bp GetThreadContext` 运行。

第一次中断后，`F9` 继续，被中断在函数 `GetThreadContext` 中：

```
7C838EEB >  8BFF            MOV     EDI, EDI
7C838EED    55              PUSH    EBP
7C838EEE    8BEC            MOV     EBP, ESP
7C838EF0    FF75 0C         PUSH    DWORD PTR SS:[EBP+C]
7C838EF3    FF75 08         PUSH    DWORD PTR SS:[EBP+8]
7C838EF6    FF15 F814807C   CALL    DWORD PTR DS:[<&ntdll.NtGetConte>; ntdll.ZwGetContextThread
7C838EFC    85C0            TEST    EAX, EAX
7C838EFE    0F8C 57B60000   JL      kernel32.7C84455B
7C838F04    33C0            XOR     EAX, EAX
7C838F06    40              INC     EAX
7C838F07    5D              POP     EBP
7C838F08    C2 0800         RETN    8
```

`Alt+F9` 到：

```
00805E47   .  50            PUSH    EAX
00805E48   .  F7D0          NOT     EAX
00805E4A   .  0FC8          BSWAP   EAX
00805E4C   .  58            POP     EAX
00805E4D   .  73 00         JNB     SHORT ezcddax.00805E4F
00805E4F   >  9C            PUSHFD
00805E50   .  60            PUSHAD
00805E51   .  EB 2B         JMP     SHORT ezcddax.00805E7E
```

在 OD 中删除分析，还原壳的代码，分析这段处理 CC 的代码。

请参考：`http://bbs.pediy.com//showthread ... 0&threadid=6991`

这里就不再分析了：

```
00805E26    83C4 0C         ADD     ESP, 0C
00805E29    C785 7CEBFFFF 0>MOV    DWORD PTR SS:[EBP-1484], 10001
00805E33    8D85 7CEBFFFF   LEA     EAX, DWORD PTR SS:[EBP-1484]
00805E39    50              PUSH    EAX
00805E3A    8B8D 50EEFFFF   MOV     ECX, DWORD PTR SS:[EBP-11B0]
00805E40    51              PUSH    ECX
00805E41    FF15 E0808300   CALL    DWORD PTR DS:[<&KERNEL32.GetThre>; kernel32.GetThreadContext
00805E47    50              PUSH    EAX
00805E48    F7D0            NOT     EAX
00805E4A    0FC8            BSWAP   EAX

{处理代码}

00806201    66:92           XCHG    AX, DX
00806203    8BC0            MOV     EAX, EAX
00806205    8D95 7CEBFFFF   LEA     EDX, DWORD PTR SS:[EBP-1484]
0080620B    52              PUSH    EDX
0080620C    8B85 50EEFFFF   MOV     EAX, DWORD PTR SS:[EBP-11B0]
00806212    50              PUSH    EAX
00806213    FF15 DC808300   CALL    DWORD PTR DS:[<&KERNEL32.SetThre>; kernel32.SetThreadContext
00806219    60              PUSHAD
0080621A    33C0            XOR     EAX, EAX
0080621C    75 02           JNZ     SHORT ezcddax.00806220
0080621E    EB 15           JMP     SHORT ezcddax.00806235
00806220    EB 33           JMP     SHORT ezcddax.00806255
```

### 2. 通过这段代码，大概查找 CC 的范围

通过分析知道：

```
00805EC3    8B95 34ECFFFF   MOV     EDX, DWORD PTR SS:[EBP-13CC]
00805EC9    52              PUSH    EDX
00805ECA    8B85 48EEFFFF   MOV     EAX, DWORD PTR SS:[EBP-11B8]
```

这段代码中的：

```
00805EC3    8B95 34ECFFFF   MOV     EDX, DWORD PTR SS:[EBP-13CC]
```

是把 Context 域的 CC 发生时的下一字节地址传送的 EDX 中，在

```
00805E9F    52              PUSH    EDX
```

设置 `[条件记录断点]`

表达式：`[EDX]`

暂停程序：从不

记录表达式数据：永远

设置 `[记录]` 到文件，输入文件名保存，取消断点 `GetThreadContext`。

设置好后运行程序，最好运行程序的所有功能，关闭 OD 得到一个 CC 地址表：

```
00805E9F  COND: 00439891
7C838EEB  断点位于 kernel32.GetThreadContext
00805E9F  COND: 0043989E
00805E9F  COND: 00439962
00805E9F  COND: 00439989
00805E9F  COND: 004399A3
00805E9F  COND: 004399AD
00805E9F  COND: 00439B5A
00805E9F  COND: 00439B84
00805E9F  COND: 00439B92
00805E9F  COND: 00439C2D
00805E9F  COND: 00439B9C
00805E9F  COND: 00439BA4
00805E9F  COND: 00439BE2
00805E9F  COND: 00439C2D
00805E9F  COND: 00439B9C
00805E9F  COND: 00439BA4
00805E9F  COND: 00439BE2
00805E9F  COND: 00439C2D
00805E9F  COND: 00439C52
00805E9F  COND: 0046C5D4
00805E9F  COND: 0046C604
00805E9F  COND: 0046C786
00805E9F  COND: 0046C82B
00805E9F  COND: 004E3251
00805E9F  COND: 004E3262
00805E9F  COND: 004E3280
00805E9F  COND: 004E32B1
00805E9F  COND: 004E33D1
00805E9F  COND: 004E34F1
00805E9F  COND: 004E35A7
00805E9F  COND: 0046DB82
00805E9F  COND: 0046DBC7
00805E9F  COND: 0046DC19
00400000  卸载 C:\Program Files\Easy CD-DA Extractor 9\ezcddax.exe
操作完成
```

由于只是演示方法，所以这个表可能不完整。

从上面的记录中可以看到 CC 的大概范围，用 OD 打开 dump 下的文件，到内存窗口中把 text 段的二进制代码保存到一个文件 `code.txt` 中待用。

### 3. 查找 CC 地址

OD 重新加载主程序，`bp GetThreadContext` 运行，第二次依然来得：

```
00805E26    83C4 0C         ADD     ESP, 0C
00805E29    C785 7CEBFFFF 0>MOV    DWORD PTR SS:[EBP-1484], 10001
00805E33    8D85 7CEBFFFF   LEA     EAX, DWORD PTR SS:[EBP-1484]
00805E39    50              PUSH    EAX
00805E3A    8B8D 50EEFFFF   MOV     ECX, DWORD PTR SS:[EBP-11B0]
00805E40    51              PUSH    ECX
00805E41    FF15 E0808300   CALL    DWORD PTR DS:[<&KERNEL32.GetThre>; kernel32.GetThreadContext
00805E47    50              PUSH    EAX        //停在这里
```

到 OD 的内存窗口中，选择 text 段 双击打开 text 段:

```
00401000  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
00401010  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
```

打开保存的 `code.txt` 文件把二进制代码复制到 text 段的内存中，在壳的段中找一段暂时不用的地址，存放获得的 CC 地址，如：

```
00828000  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
```

记下现在的寄存器：

```
EAX 00000001
ECX 0012DC78
EDX 7C92EB94 ntdll.KiFastSystemCallRet
EBX 7FFDE000
ESP 0012DC98
EBP 0012F79C
ESI 00000017
EDI 0012E2EC
EIP 00805E47 ezcddax.00805E47
C 0  ES 0023 32 位 0(FFFFFFFF)
P 0  CS 001B 32 位 0(FFFFFFFF)
A 0  SS 0023 32 位 0(FFFFFFFF)
Z 0  DS 0023 32 位 0(FFFFFFFF)
S 0  FS 003B 32 位 7FFDD000(FFF)
T 0  GS 0000 NULL
D 0
O 0  LastErr ERROR_SEM_TIMEOUT (00000079)
EFL 00000202 (NO,NB,NE,A,NS,PO,GE,G)
ST0 empty -UNORM D1D8 01050104 00000000
ST1 empty 0.0
ST2 empty 0.0
ST3 empty 0.0
ST4 empty 0.0
ST5 empty 0.0
ST6 empty 0.0
ST7 empty 0.0
             3 2 1 0      E S P U O Z D I
FST 0000  Cond 0 0 0 0  Err 0 0 0 0 0 0 0 0  (GT)
FCW 027F  Prec NEAR,53  掩码    1 1 1 1 1 1
```

写段代码，查找 CC 地址：

```
007D8000    9C              PUSHFD
007D8001    60              PUSHAD
007D8002    B8 00984300     MOV     EAX, ezcddax.00439800
007D8007    BB 00808200     MOV     EBX, ezcddax.00828000
007D800C    8038 CC         CMP     BYTE PTR DS:[EAX], 0CC
007D800F    74 0C           JE      SHORT ezcddax.007D801D
007D8011    83C0 01         ADD     EAX, 1
007D8014    3D 00004700     CMP     EAX, ezcddax.00470000
007D8019    74 11           JE      SHORT ezcddax.007D802C
007D801B  ^ EB EF          JMP     SHORT ezcddax.007D800C
007D801D    8D40 01         LEA     EAX, DWORD PTR DS:[EAX+1]
007D8020    8903            MOV     DWORD PTR DS:[EBX], EAX
007D8022    83C0 01         ADD     EAX, 1
007D8025    83C3 04         ADD     EBX, 4
007D8028  ^ EB E5          JMP     SHORT ezcddax.007D800F
007D802A    90              NOP
007D802B    90              NOP
007D802C    61              POPAD
007D802D    9D              POPFD
007D802E    90              NOP
```

二进制代码：

```
9C 60 B8 00 98 43 00 BB 00 80 82 00 80 38 CC 74 0C 83 C0 01 3D 00 00 47 00 74 11 EB EF 8D 40 01
89 03 83 C0 01 83 C3 04 EB E5 90 90 61 9D 90
```

在 `007D8000` 新建 EIP，`007D802E` 下中断，运行这段代码，在地址 `00828000` 处得到一张 CC 地址表：

```
00828000  91 98 43 00 9E 98 43 00 62 99 43 00 73 99 43 00   憳 C.灅 C.b 機.s 機.
00828010  89 99 43 00 A3 99 43 00 A7 99 43 00 AD 99 43 00   墮 C.C. C.瓩 C.
00828020  5A 9B 43 00 84 9B 43 00 92 9B 43 00 9C 9B 43 00   Z 汣.剾 C.挍 C.湜 C.
00828030  A4 9B 43 00 E2 9B 43 00 F7 9B 43 00 FB 9B 43 00    C.鉀 C.鳑 C.麤 C.
00828040  2D 9C 43 00 52 9C 43 00 BC 9D 43 00 87 9F 43 00   -淐.R 淐.紳 C.嚐 C.
```
