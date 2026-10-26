
## 分析过程

仔细观察这段代码，发现地址`004C2B59`处`PUSH`的数据是用于显示“无效的ID或名字”对话框的起始参数，上面（地址`004C2B57`）一句`JMP`指令则跳过了这一段，与此形成了一个分水岭；故而再往上看，看看有哪条指令会跳转到地址`004C2B59`的`push`指令处（也就是注册失败的处理逻辑处），发现是地址`004C2AC6`处的`JE`指令，这就说明前面地址`004C2ABF`处的`CALL`函数调用，如果返回值为0的话，则表明注册操作失败，因而此函数值得重点对待（对其下断点）；然后再在本函数起始地址`004C2A55`处下断点，随便在注册框中输入数据（如`NAME:111`，`ID:222`）；随后单击确定按钮，不出意料断在了函数起始地址`004C2A55`处，接下来我们再`F8`单步跟踪至地址`004C2ABF`处，单步的过程中观察寄存器的变化，发现前面的代码主要是用于获取用户在注册对话框中输入的内容；最后`F7`进入我们需要重点关注的函数内部；

### 地址`004C2ABF`处`CALL`的函数的具体实现如下：

```
004CBF7C /$  55                     PUSH    EBP
004CBF7D |.  8BEC                   MOV     EBP, ESP
004CBF7F |.  51                     PUSH    ECX
004CBF80 |.  53                     PUSH    EBX
004CBF81 |.  8955 FC                MOV     [LOCAL.1], EDX
004CBF84 |.  8B45 FC                MOV     EAX, [LOCAL.1]
004CBF87 |.  E8 B48EF3FF            CALL    eXeScope.00404E40
004CBF8C |.  33C0                   XOR     EAX, EAX
004CBF8E |.  55                     PUSH    EBP
004CBF8F |.  68 1BC04C00            PUSH    eXeScope.004CC01B
004CBF94 |.  64:FF30                PUSH    DWORD PTR FS:[EAX]
004CBF97 |.  64:8920                MOV     DWORD PTR FS:[EAX], ESP
004CBF9A |.  33DB                   XOR     EBX, EBX
004CBF9C |.  8B45 FC                MOV     EAX, [LOCAL.1]
004CBF9F |.  E8 AC8CF3FF            CALL    eXeScope.00404C50
004CBFA4 |.  83F8 0A                CMP     EAX, 0A
004CBFA7 |.  75 5C                  JNZ     SHORT eXeScope.004CC005
004CBFA9 |.  8B55 FC                MOV     EDX, [LOCAL.1]
004CBFAC |.  B8 30C04C00            MOV     EAX, eXeScope.004CC030        ;  a1910
004CBFB1 |.  E8 DE8FF3FF            CALL    eXeScope.00404F94
004CBFB6 |.  48                     DEC     EAX
004CBFB7 |.  74 10                  JE      SHORT eXeScope.004CBFC9
004CBFB9 |.  8B55 FC                MOV     EDX, [LOCAL.1]
004CBFBC |.  B8 40C04C00            MOV     EAX, eXeScope.004CC040        ;  a1423
004CBFC1 |.  E8 CE8FF3FF            CALL    eXeScope.00404F94
004CBFC6 |.  48                     DEC     EAX
004CBFC7 |.  75 3C                  JNZ     SHORT eXeScope.004CC005
004CBFC9 |>  B8 02000000            MOV     EAX, 2
004CBFCE |>  8B55 FC                /MOV     EDX, [LOCAL.1]
004CBFD1 |.  8A5402 FF              |MOV     DL, BYTE PTR DS:[EDX+EAX-1]
004CBFD5 |.  80FA 30                |CMP     DL, 30
004CBFD8 |.  72 2B                  |JB      SHORT eXeScope.004CC005
004CBFDA |.  80FA 39                |CMP     DL, 39
004CBFDD |.  77 26                  |JA      SHORT eXeScope.004CC005
004CBFDF |.  40                     |INC     EAX
004CBFE0 |.  83F8 0B                |CMP     EAX, 0B
004CBFE3 |.^ 75 E9                  \JNZ     SHORT eXeScope.004CBFCE
004CBFE5 |.  8B45 FC                MOV     EAX, [LOCAL.1]
004CBFE8 |.  0FB640 08              MOVZX   EAX, BYTE PTR DS:[EAX+8]
004CBFEC |.  8B55 FC                MOV     EDX, [LOCAL.1]
004CBFEF |.  0FB652 09              MOVZX   EDX, BYTE PTR DS:[EDX+9]
004CBFF3 |.  03C2                   ADD     EAX, EDX
004CBFF5 |.  B9 0A000000            MOV     ECX, 0A
004CBFFA |.  33D2                   XOR     EDX, EDX
004CBFFC |.  F7F1                   DIV     ECX
004CBFFE |.  83FA 04                CMP     EDX, 4
004CC001 |.  75 02                  JNZ     SHORT eXeScope.004CC005
004CC003 |.  B3 01                  MOV     BL, 1
004CC005 |>  33C0                   XOR     EAX, EAX
004CC007 |.  5A                     POP     EDX                           ;  eXeScope.004C2AC4
004CC008 |.  59                     POP     ECX                           ;  eXeScope.004C2AC4
004CC009 |.  59                     POP     ECX                           ;  eXeScope.004C2AC4
004CC00A |.  64:8910                MOV     DWORD PTR FS:[EAX], EDX
004CC00D |.  68 22C04C00            PUSH    eXeScope.004CC022
004CC012 |>  8D45 FC                LEA     EAX, [LOCAL.1]
004CC015 |.  E8 7689F3FF            CALL    eXeScope.00404990
004CC01A \.  C3                     RETN
004CC01B .^ E9 5483F3FF             JMP     eXeScope.00404374
004CC020 .^ EB F0                   JMP     SHORT eXeScope.004CC012
004CC022 .  8BC3                    MOV     EAX, EBX
004CC024 .  5B                      POP     EBX                           ;  eXeScope.004C2AC4
004CC025 .  59                      POP     ECX                           ;  eXeScope.004C2AC4
004CC026 .  5D                      POP     EBP                           ;  eXeScope.004C2AC4
004CC027 .  C3                      RETN
```

大致观察一下，并`F8`单步执行几遍看看流程，发现输入的`ID`长度如果不是10（`0A`），就会通过地址`004CBFA7`处的`JNZ`指令直接跳过一大段代码后直接返回（返回值为0），导致上层认为失败；所以我们接下来需要构造长度为10的`ID`试试，如`ID:0123456789`；此时我们能顺利进入地址`004CBFA9`处，发现此处指令旁边有2个比较明显的字符串参考注释：`a1910`和`a1423`；并且下面指令`CALL eXeScope.00404F94`之后，如果返回值`EAX`为1，都会直接进入地址`004CBFC9`处，否则跟`ID`长度不为10的命运一样；所以我们需要进去看下`eXeScope.00404F94`究竟做什么，才能导致`EAX`为1；

### 函数`eXeScope.00404F94`的具体实现如下：

```
00404F94 /$  85C0                   TEST    EAX, EAX
00404F96 |.  74 40                  JE      SHORT eXeScope.00404FD8
00404F98 |.  85D2                   TEST    EDX, EDX
00404F9A |.  74 31                  JE      SHORT eXeScope.00404FCD
00404F9C |.  53                     PUSH    EBX
00404F9D |.  56                     PUSH    ESI                           ;  eXeScope.0046B5B0
00404F9E |.  57                     PUSH    EDI
00404F9F |.  89C6                   MOV     ESI, EAX
00404FA1 |.  89D7                   MOV     EDI, EDX
00404FA3 |.  8B4F FC                MOV     ECX, DWORD PTR DS:[EDI-4]
00404FA6 |.  57                     PUSH    EDI
00404FA7 |.  8B56 FC                MOV     EDX, DWORD PTR DS:[ESI-4]
00404FAA |.  4A                     DEC     EDX
00404FAB |.  78 1B                  JS      SHORT eXeScope.00404FC8
00404FAD |.  8A06                   MOV     AL, BYTE PTR DS:[ESI]
00404FAF |.  46                     INC     ESI                           ;  eXeScope.0046B5B0
00404FB0 |.  29D1                   SUB     ECX, EDX
00404FB2 |.  7E 14                  JLE     SHORT eXeScope.00404FC8
00404FB4 |>  F2:AE                  /REPNE   SCAS BYTE PTR ES:[EDI]
00404FB6 |.  75 10                  |JNZ     SHORT eXeScope.00404FC8
00404FB8 |.  89CB                   |MOV     EBX, ECX
00404FBA |.  56                     |PUSH    ESI                          ;  eXeScope.0046B5B0
00404FBB |.  57                     |PUSH    EDI
00404FBC |.  89D1                   |MOV     ECX, EDX
00404FBE |.  F3:A6                  |REPE    CMPS BYTE PTR ES:[EDI], BYTE PTR DS:[ESI]
00404FC0 |.  5F                     |POP     EDI                          ;  0012F26C
00404FC1 |.  5E                     |POP     ESI                          ;  0012F26C
00404FC2 |.  74 0C                  |JE      SHORT eXeScope.00404FD0
00404FC4 |.  89D9                   |MOV     ECX, EBX
00404FC6 |.^ EB EC                  \JMP     SHORT eXeScope.00404FB4
00404FC8 |>  5A                     POP     EDX                           ;  0012F26C
00404FC9 |.  31C0                   XOR     EAX, EAX
00404FCB |.  EB 08                  JMP     SHORT eXeScope.00404FD5
00404FCD |>  31C0                   XOR     EAX, EAX
00404FCF |.  C3                     RETN
00404FD0 |>  5A                     POP     EDX                           ;  0012F26C
00404FD1 |.  89F8                   MOV     EAX, EDI
00404FD3 |.  29D0                   SUB     EAX, EDX
00404FD5 |>  5F                     POP     EDI                           ;  0012F26C
00404FD6 |.  5E                     POP     ESI                           ;  0012F26C
00404FD7 |.  5B                     POP     EBX                           ;  0012F26C
00404FD8 \>  C3                     RETN
```
