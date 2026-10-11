
# Crunch v1.1

> 原文：[`www.pediy.com/kssd/pediy03/forum349.htm`](https://www.pediy.com/kssd/pediy03/forum349.htm)

## 对 `Crunch` `v1.1` 加壳程序的手动脱壳及反跟踪代码的一点分析

【声明】

我写文章以交流为主，希望大家在转载时能保持文章的完整性。

【前言】

由于我现在还没有破解版的 `Crunch` `v1.1`，所以就拿未注册版的 `Crunch` 来测试。好象未注册版的 `Crunch` 加壳的程序的名字有限制，其他的我就不清楚了。由于我 `DOWN` 的这个文件是根据看雪论坛的一个贴子提供的地址下载的。连帮助文件也没有。:-(

而且目前我手头也没有注册版的 `Crunch`，所以只好以未注册版的 `Crunch` 加壳 `Notepad.EXE` 程序来进行说明。

样例文件： `bitarts_evaluation.exe` （把 `Notepad.EXE` 加壳后的文件）

（未注册版 `Crunch` 加壳的程序文件名都是这个，不过可以在跟踪中发现文件名比较的地方）

加壳方式： 未注册版 `Crunch` `v1.1` 加壳

目标： 手动脱壳

作者： `ljttt`

写作日期： 2000-10-02

## 1、这次我们用 `ProcDump` 提供的 `Bhrama Server` 的功能来手动脱壳，这样可以简化一点手动脱壳的麻烦。

所以首先我们启动 `ProcDump`，选择 `Option/Rebuild new import table`。点击 `Bhrama Server`，这时就启动了服务器，将等待客户命令。

## 2、`Ctrl-D` 进入 `SoftICE`，设断点

```
bpint 3
```

## 3、启动 `PEditor`，打开 `bitarts_evaluation.exe`，点击 `break'n'enter`。

单击 `Run`。这时将中断进入 `SoftICE`。可以看到如下提示。

`?You have to enter "eb eip 55" before you continue ! ?`

为了继续跟踪，我们根据提示下指令

```
eb eip 55
```

现在将显示如下：

```
015F:0040D000  55                 PUSH       EBP       <--这就是我们修改后的结果
015F:0040D001  E800000000         CALL       0040D006
015F:0040D006  5D                 POP        EBP
```

## 4、继续跟踪到如下处：

```
015F:0040D0AC  FF95B1320000       CALL       [EBP+000032B1]
015F:0040D0B2  83C408             ADD        ESP,08
015F:0040D0B5  8BB5AD320000       MOV        ESI,[EBP+000032AD]
015F:0040D0BB  BFC6000000         MOV        EDI,000000C6
015F:0040D0C0  03FD               ADD        EDI,EBP
015F:0040D0C2  8BC8               MOV        ECX,EAX
015F:0040D0C4  F3A4               REPZ MOVSB              <--在此处停下，此指令执行后，以下的指令将改变，这就是 SMC 的技巧，以后还有多处，就不一一列举了。
015F:0040D0C6  8B07               MOV        EAX,[EDI]     <--现在我们看到的是加密的代码，只有上一指令执行后才会显示正确的指令代码
015F:0040D0C8  D581               AAD
015F:0040D0CA  C23813             RET        1338
```

## 5、在 `REPZ MOVSB` 指令执行后我们将看到如下的正确代码：

```
015F:0040D0C6  8BD5               MOV        EDX,EBP
015F:0040D0C8  81C238130000       ADD        EDX,00001338
015F:0040D0CE  52                 PUSH       EDX
015F:0040D0CF  33C0               XOR        EAX,EAX
015F:0040D0D1  8CD8               MOV        AX,DS
015F:0040D0D3  A804               TEST       AL,04
015F:0040D0D5  7408               JZ         0040D0DF
015F:0040D0D7  B402               MOV        AH,02
015F:0040D0D9  CD1A               INT        1A
015F:0040D0DB  8BC2               MOV        EAX,EDX
015F:0040D0DD  EB02               JMP        0040D0E1
015F:0040D0DF  0F31               RDTSC
015F:0040D0E1  33D2               XOR        EDX,EDX
015F:0040D0E3  69C00D661900       IMUL       EAX,EAX,0019660D
015F:0040D0E9  05CD0D0100         ADD        EAX,00010DCD
015F:0040D0EE  8985383A0000       MOV        [EBP+00003A38],EAX
015F:0040D0F4  BB56340200         MOV        EBX,00023456
015F:0040D0F9  43                 INC        EBX
015F:0040D0FA  F7F3               DIV        EBX
015F:0040D0FC  8BC2               MOV        EAX,EDX
015F:0040D0FE  5A                 POP        EDX
015F:0040D0FF  90                 NOP
015F:0040D100  FFD2               CALL       EDX       <--在此处我们要停下，如果你想了解反跟踪代码，可以进入看看
015F:0040D102  CC                 INT        3
015F:0040D103  CC                 INT        3
015F:0040D104  E80E130000         CALL       0040E417
015F:0040D109  E838100000         CALL       0040E146      <--反跟踪
```

## 6、在执行到 `CALL EDX` 指令处时，我们下指令

```
r eip eip+E
r esi 10
```

这样就可以跳过这一段反跟踪的代码。

## 7、这里我们来了解一下 `CALL EDX` 中的一段反跟踪代码

```
015F:0040E369  0F010B             SIDT       FWORD PTR [EBX]    <--取 IDTR 的内容
015F:0040E36C  8B7302             MOV        ESI,[EBX+02]       <--取 IDT 表的基地址
015F:0040E36F  83C608             ADD        ESI,08
015F:0040E372  8B1E               MOV        EBX,[ESI]           <--取 int 1 的低位偏移
015F:0040E374  53                 PUSH       EBX
015F:0040E375  56                 PUSH       ESI
015F:0040E376  83C610             ADD        ESI,10
015F:0040E379  8B36               MOV        ESI,[ESI]           <--取 int 3 的低位偏移
015F:0040E37B  81E6FFFF0000       AND        ESI,0000FFFF
015F:0040E381  81E3FFFF0000       AND        EBX,0000FFFF
015F:0040E387  2BF3               SUB        ESI,EBX
015F:0040E389  83FE1E             CMP        ESI,1E              <--比较低位偏移
015F:0040E38C  B090               MOV        AL,90
015F:0040E38E  7427               JZ         0040E3B7            <--关键了！
015F:0040E390  8BDD               MOV        EBX,EBP
015F:0040E392  81C311140000       ADD        EBX,00001411
015F:0040E398  0F010B             SIDT       FWORD PTR [EBX]     <--取 IDTR 的内容
015F:0040E39B  8B7B02             MOV        EDI,[EBX+02]        <--取 IDT 表的基地址
015F:0040E39E  83C708             ADD        EDI,08
015F:0040E3A1  8B4704             MOV        EAX,[EDI+04]        <--取 int 1 的高位偏移
015F:0040E3A4  668B07             MOV        AX,[EDI]            <--取 int 1 的低位偏移
015F:0040E3A7  8038E9             CMP        BYTE PTR [EAX],E9   <--判断 int 1 中断处理程序第一个字节是否为 JMP 指令
015F:0040E3AA  B090               MOV        AL,90
015F:0040E3AC  7409               JZ         0040E3B7            <--关键了！
015F:0040E3AE  83FE1E             CMP        ESI,1E
015F:0040E3B1  B090               MOV        AL,90
015F:0040E3B3  7402               JZ         0040E3B7            <--关键了！
015F:0040E3B5  045B               ADD        AL,5B
```

## 8、这里我们来了解一下 `CALL 0040E146` 中的反跟踪代码

```
015F:0040E146  8B8587390000       MOV        EAX,[EBP+00003987]
015F:0040E14C  8B8DC5300000       MOV        ECX,[EBP+000030C5]
015F:0040E152  C1E908             SHR        ECX,08
015F:0040E155  33D2               XOR        EDX,EDX
015F:0040E157  B902000000         MOV        ECX,00000002
015F:0040E15C  F7F1               DIV        ECX
015F:0040E15E  0BD2               OR         EDX,EDX
015F:0040E160  7502               JNZ        0040E164            <--关键了！
015F:0040E162  EB27               JMP        0040E18B

==> 0040E164  8BDD                MOV        EBX,EBP
015F:0040E166  81C38B390000       ADD        EBX,0000398B
015F:0040E16C  0F010B             SIDT       FWORD PTR [EBX]     <--取 IDTR 的内容
015F:0040E16F  8B7302             MOV        ESI,[EBX+02]        <--取 IDT 表的基地址
015F:0040E172  83C608             ADD        ESI,08
015F:0040E175  8B1E               MOV        EBX,[ESI]           <--取 int 1 的低位偏移
015F:0040E177  83C610             ADD        ESI,10
015F:0040E17A  8B36               MOV        ESI,[ESI]           <--取 int 3 的低位偏移
015F:0040E17C  81E6FFFF0000       AND        ESI,0000FFFF
015F:0040E182  81E3FFFF0000       AND        EBX,0000FFFF
015F:0040E188  2BF3               SUB        ESI,EBX             <--ESI 返回低位偏移之差
015F:0040E18A  C3                 RET
015F:0040E18B  BE10000000         MOV        ESI,00000010        <--ESI 返回 0x10
015F:0040E190  C3                 RET
```

## 9、我们继续跟踪来到如下，（这也是由 `SMC` 得来的）

```
015F:0040D1F7  49                 DEC        ECX
015F:0040D1F8  83F900             CMP        ECX,00
015F:0040D1FB  75EB               JNZ        0040D1E8
015F:0040D1FD  59                 POP        ECX
015F:0040D1FE  81E942000000       SUB        ECX,00000042
015F:0040D204  51                 PUSH       ECX
015F:0040D205  E873100000         CALL       0040E27D            <--内有反跟踪代码
015F:0040D20A  33C0               XOR        EAX,EAX
015F:0040D20C  59                 POP        ECX
015F:0040D20D  8AC1               MOV        AL,CL
015F:0040D20F  8BDD               MOV        EBX,EBP
015F:0040D211  81C3A5320000       ADD        EBX,000032A5
015F:0040D217  8BB57D390000       MOV        ESI,[EBP+0000397D]
015F:0040D21D  50                 PUSH       EAX
015F:0040D21E  51                 PUSH       ECX
```
