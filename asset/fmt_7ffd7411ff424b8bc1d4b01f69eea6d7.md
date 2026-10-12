
# PE-Armor

> 原文：[`www.pediy.com/kssd/pediy06/pediy6507.htm`](https://www.pediy.com/kssd/pediy06/pediy6507.htm)

## 第二部分 脱壳

#################################################################################################

脱壳过程，已经有了前面的分析，这里尽量简化操作：

用 `OD` 载入，来到入口点：

```
00611F19 > EB 20             JMP SHORT DreamRo.00611F3B

... ...

00611F3B  9C                 PUSHFD

... ...

00611F54  9D                 POPFD
00611F55  ^E9 73A1FFFF       JMP DreamRo.0060C0CD
```

光标定在 `611F55` 这行上按 `F4`，再按 `F7`，来到这里：

```
0060C0CD  60                 PUSHAD
0060C0CE  E8 00000000        CALL DreamRo.0060C0D3
0060C0D3  5D                 POP EBP
0060C0D4  81ED D3000000      SUB EBP,0D3
0060C0DA  8DB5 EA000000      LEA ESI,DWORD PTR SS:[EBP+EA]
0060C0E0  55                 PUSH EBP
0060C0E1  56                 PUSH ESI
0060C0E2  81C5 24100000      ADD EBP,1024
0060C0E8  55                 PUSH EBP
0060C0E9  C3                 RETN
0060C0EA  C6                 ???
0060C0EB  B3 D7              MOV BL,0D7
```

光标定在 `60C0EB` 这行上按 `F4`，来到这里：

```
0060C0EB  5D                 POP EBP
0060C0EC  8B45 00            MOV EAX,DWORD PTR SS:[EBP]

... ...

0060C141  C640 01 10         MOV BYTE PTR DS:[EAX+1],10
0060C145  FFE2               JMP EDX
```

光标定在 `60C145` 这行上按 `F4`，再按 `F7`，来到这里：

```
00340000  E8 24000000        CALL 00340029
00340005  8B4424 04          MOV EAX,DWORD PTR SS:[ESP+4]
00340009  8B00               MOV EAX,DWORD PTR DS:[EAX]
0034000B  3D 04000080        CMP EAX,80000004
00340010  75 08              JNZ SHORT 0034001A
00340012  8B6424 08          MOV ESP,DWORD PTR SS:[ESP+8]
00340016  EB 04              JMP SHORT 0034001C
00340018  58                 POP EAX
00340019  EB 0C              JMP SHORT 00340027

... ...
```

一直向下翻到这里：

```
00343533  3D 940000C0        CMP EAX,C0000094
00343538  75 2A              JNZ SHORT 00343564
0034353A  C702 00000000      MOV DWORD PTR DS:[EDX],0
00343540  FF81 B8000000      INC DWORD PTR DS:[ECX+B8]
00343546  33C0               XOR EAX,EAX
00343548  2141 04            AND DWORD PTR DS:[ECX+4],EAX
0034354B  2141 08            AND DWORD PTR DS:[ECX+8],EAX
0034354E  2141 0C            AND DWORD PTR DS:[ECX+C],EAX
00343551  2141 10            AND DWORD PTR DS:[ECX+10],EAX
00343554  8161 14 F00FFFFF   AND DWORD PTR DS:[ECX+14],FFFF0FF0
0034355B  8161 18 00DC0000   AND DWORD PTR DS:[ECX+18],0DC00
00343562  EB 6E              JMP SHORT 003435D2
```

在 `34353A` 这行按 `F2` 下一个断点，再按 `F9` 运行，断下来后清掉此断点，向下翻到这里：

```
003436A6  8BFC               MOV EDI,ESP
003436A8  8DA5 FC314000      LEA ESP,DWORD PTR SS:[EBP+4031FC]
003436AE  B9 FB180000        MOV ECX,18FB
003436B3  B8 A4ABA45B        MOV EAX,5BA4ABA4
003436B8  BB BDD89800        MOV EBX,98D8BD
003436BD  BE D5260000        MOV ESI,26D5
003436C2  33D2               XOR EDX,EDX
003436C4  F7E6               MUL ESI
003436C6  05 78563412        ADD EAX,12345678
003436CB  83D2 00            ADC EDX,0
003436CE  F7F3               DIV EBX
003436D0  58                 POP EAX
003436D1  32C2               XOR AL,DL
003436D3  50                 PUSH EAX
003436D4  4C                 DEC ESP
003436D5  8BC2               MOV EAX,EDX
003436D7  ^E2 E9             LOOPD SHORT 003436C2
003436D9  C9                 LEAVE
003436DA  6BDB CD            IMUL EBX,EBX,-33
```

在 `3436A6` 这行按 `F2` 下一个断点，再按 `F9` 运行，断下来后清掉此断点。

光标定在 `3436D9` 按 `F4` 来到这里：

```
003436D9  8BE7               MOV ESP,EDI
003436DB  8DB5 FD314000      LEA ESI,DWORD PTR SS:[EBP+4031FD]
003436E1  B9 03000000        MOV ECX,3
003436E6  EB 07              JMP SHORT 003436EF
```

光标定在 `34397E` 按 `F4` 来到这里：

```
0034397E  87E6               XCHG ESI,ESP
00343980  6A 04              PUSH 4
00343982  68 00100000        PUSH 1000
00343987  68 00200000        PUSH 2000
0034398C  6A 00              PUSH 0
0034398E  FF95 09324000      CALL DWORD PTR SS:[EBP+403209]
```

下面我们要开始修改代码了，目的是去掉 API 重定位，便于以后的 `IAT` 重建。

首先注意下面这段代码，我打算先从这里入手：

```
00343B6A  8B0A               MOV ECX,DWORD PTR DS:[EDX]       ;重定位处理开始
00343B6C  81E1 FFFFFF7F      AND ECX,7FFFFFFF
00343B72  51                 PUSH ECX                          ;函数数目
00343B73  52                 PUSH EDX
00343B74  C1E1 05            SHL ECX,5                         ;每个函数占用 32 字节重定位空间
00343B77  6A 04              PUSH 4
00343B79  68 00100000        PUSH 1000
00343B7E  51                 PUSH ECX
00343B7F  6A 00              PUSH 0
00343B81  8D85 BD1D4000      LEA EAX,DWORD PTR SS:[EBP+401DBD] ;=343B94
00343B87  50                 PUSH EAX
00343B88  8B85 09324000      MOV EAX,DWORD PTR SS:[EBP+403209] ;VirtualAlloc
00343B8E  E9 98080000        JMP 0034442B
00343B93  E8                 DB E8
00343B94  8985 4D324000      MOV DWORD PTR SS:[EBP+40324D],EAX ;为第一层重定位分配的空间
00343B9A  5A                 POP EDX
00343B9B  59                 POP ECX
00343B9C  50                 PUSH EAX
00343B9D  51                 PUSH ECX
00343B9E  2BBD 0D324000      SUB EDI,DWORD PTR SS:[EBP+40320D] ;BASE ADDRESS
00343BA4  83FF FF            CMP EDI,-1
00343BA7  74 15              JE SHORT 00343BBE
00343BA9  03BD 0D324000      ADD EDI,DWORD PTR SS:[EBP+40320D]
00343BAF  EB 09              JMP SHORT 00343BBA
00343BB1  8907               MOV DWORD PTR DS:[EDI],EAX        ;往 IAT 中填充第一层重定位的函数地址
00343BB3  83C0 20            ADD EAX,20
00343BB6  83C7 04            ADD EDI,4
00343BB9  49                 DEC ECX
00343BBA  0BC9               OR ECX,ECX
00343BBC  75 F3              JNZ SHORT 00343BB1
00343BBE  59                 POP ECX
00343BBF  58                 POP EAX
00343BC0  8BF8               MOV EDI,EAX
00343BC2  57                 PUSH EDI
00343BC3  51                 PUSH ECX
00343BC4  EB 2D              JMP SHORT 00343BF3
00343BC6  8D47 1C            LEA EAX,DWORD PTR DS:[EDI+1C]     ;REAL_PROC_ADDR
00343BC9  66:C707 FF35      MOV WORD PTR DS:[EDI],35FF        ;PUSH DWORD PTR [REAL_PROC_ADDR]
00343BCE  C747 06 81342400   MOV DWORD PTR DS:[EDI+6],243481   ;XOR DWORD PTR [ESP],XORKEY
00343BD5  8947 02            MOV DWORD PTR DS:[EDI+2],EAX      ;RET
00343BD8  C647 0D C3         MOV BYTE PTR DS:[EDI+D],0C3       ;
00343BDC  52                 PUSH EDX
00343BDD  0F31               RDTSC
00343BDF  32E0               XOR AH,AL
00343BE1  C1C8 08            ROR EAX,8
00343BE4  02E0               ADD AH,AL
00343BE6  C1C8 08            ROR EAX,8
00343BE9  32E0               XOR AH,AL
00343BEB  8947 09            MOV DWORD PTR DS:[EDI+9],EAX      ;XORKEY
00343BEE  5A                 POP EDX
00343BEF  83C7 20            ADD EDI,20
00343BF2  49                 DEC ECX
00343BF3  0BC9               OR ECX,ECX
00343BF5  75 CF              JNZ SHORT 00343BC6                ;上面一段是 FILL REDIR CODE
00343BF7  59                 POP ECX
00343BF8  5F                 POP EDI
00343BF9  83C2 04            ADD EDX,4
00343BFC  51                 PUSH ECX
00343BFD  0FB602             MOVZX EAX,BYTE PTR DS:[EDX]
00343C00  0BC0               OR EAX,EAX
00343C02  75 2D              JNZ SHORT 00343C31
```

用 `F4` 执行到 `343B9C` 这行，开始修改代码，目的就是跳过第一层重定位，修改后的效果是这样的：

```
00343B9C  EB 5B              JMP SHORT 00343BF9
```

在代码窗口内按 `Ctrl+G`，输入 `343C22`，看到这两行代码：

```
00343C22  3347 06            XOR EAX,DWORD PTR DS:[EDI+6]
00343C25  8947 1C            MOV DWORD PTR DS:[EDI+1C],EAX
```

把它们修改为：

```
00343C22  8907               MOV DWORD PTR DS:[EDI],EAX         ;把 API 函数地址添入原始 IAT
00343C24  90                 NOP                                 ;这个是 IMPORT BY ORD 的
00343C25  90                 NOP
00343C26  90                 NOP
00343C27  90                 NOP
```

在代码窗口内按 `Ctrl+G`，输入 `343C5F`，看到这两行代码：

```
00343C5E  52                 PUSH EDX
00343C5F  52                 PUSH EDX
00343C60  8D85 EF344000      LEA EAX,DWORD PTR SS:[EBP+4034EF]
```

把它们修改为：

```
00343C5E  E9 3A020000        JMP 00343E9D                        ;跳过 SDK 的函数处理
```
