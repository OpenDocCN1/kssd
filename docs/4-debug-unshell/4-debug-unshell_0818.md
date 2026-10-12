
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



```
00343C63  90                NOP
00343C64  90                NOP
00343C65  90                NOP
```

在代码窗口内按`Ctrl+G`，输入`343EB2`，看到这两行代码：

```
00343EB2  8B9D E1364000     MOV EBX,DWORD PTR SS:[EBP+4036E1]
00343EB8  039D E5364000     ADD EBX,DWORD PTR SS:[EBP+4036E5]
```

把它们修改为：

```
00343EB2  8907              MOV DWORD PTR DS:[EDI],EAX             ；把 API 函数地址添入原始 IAT
00343EB4  90                NOP                                     ；这个是 IMPORT BY NAME 的
00343EB5  EB 5E             JMP SHORT 00343F15                      ；跳过第二层重定位
00343EB7  90                NOP
```

还要改一下这里：

```
00343F1D  83C7 20           ADD EDI,20
```

改为：

```
00343F1D  83C7 04           ADD EDI,4                                ；调整 IAT 指针
```

接下来要注意下面的陷阱：

```
00344124  64:FF35 30000000  PUSH DWORD PTR FS:[30]                   ；NT
0034412B  58                  POP EAX                                 ；pointer to PEB
0034412C  0FB658 02           MOVZX EBX,BYTE PTR DS:[EAX+2]
00344130  0ADB                OR BL,BL                                ；检测应用程序级 debugger，用 OD 跟踪一定要注意这里
00344132  0F85 9F130000       JNZ 003454D7                            ；这里是本壳中唯一对 OllyDBG 有威胁的地方
00344138  EB 2A               JMP SHORT 00344164
```

用`F4`执行到`344130`这行，把`BL`寄存器清零，

下面对`HOOK_CALL`代码进行修改，直接解密，修复原始代码，关键代码是下面几行：

```
00344255  0385 0D324000       ADD EAX,DWORD PTR SS:[EBP+40320D]
0034425B  2B85 29324000       SUB EAX,DWORD PTR SS:[EBP+403229]
00344261  8BDE                MOV EBX,ESI
00344263  2BD8                SUB EBX,EAX
00344265  8958 FC             MOV DWORD PTR DS:[EAX-4],EBX
00344268  66:C740 FA 90E8     MOV WORD PTR DS:[EAX-6],0E890
```

把它们修改为：

```
00344255  8B5F 04             MOV EBX,DWORD PTR DS:[EDI+4]            ；这几行解密算法来自 3445B5 处的分析
00344258  81C3 CA0D0BF6       ADD EBX,F60B0DCA                        ；CRC_KEY,来自 3443B2 处的计算结果
0034425E  2BD8                SUB EBX,EAX                             ；
00344260  F7D3                NOT EBX                                 ；
00344262  C1C3 10             ROL EBX,10                              ；
00344265  8958 FC             MOV DWORD PTR DS:[EAX-4],EBX            ；
00344268  66:C740 FA FF25     MOV WORD PTR DS:[EAX-6],25FF            ；恢复为原始的 JMP DWORD PTR [XXXXXXXX]
```

接下来有个`DELPHI`的`MAINFORM`必须要处理一下，因为现在它位于壳所分配的空间内，我们`DUMP`时会丢失掉，脱壳后就会出问题。

```
003442A4  8B85 59324000       MOV EAX,DWORD PTR SS:[EBP+403259]       ；这里是对 DELPHI 的 MAINFORM 的处理
003442AA  0BC0                OR EAX,EAX                              ；其实就是把 MAINFORM 从原代码区搬到了壳里
003442AC  74 3F               JE SHORT 003442ED                       ；只需调整一下首指针就行了
003442AE  8DB5 2E164000       LEA ESI,DWORD PTR SS:[EBP+40162E]
003442B4  03F0                ADD ESI,EAX                             ；
003442B6  8B1E                MOV EBX,DWORD PTR DS:[ESI]              ；MAINFORM 的原始参考 RVA
003442B8  039D 0D324000       ADD EBX,DWORD PTR SS:[EBP+40320D]       ；BASE ADDRESS
003442BE  C706 00000000       MOV DWORD PTR DS:[ESI],0
003442C4  83C6 04             ADD ESI,4                               ；MAINFORM 的现在的地址
003442C7  8933                MOV DWORD PTR DS:[EBX],ESI              ；把 MAINFORM 的现在的地址添到其参考位置
```

当运行到这里时，`ESI=348306`，从`ESI`往后看数据区，容易看出一直到`34921C`都应该是`FORM`的数据，长度为`0F17`字节，

我们再从`EBX=4F96C4`往后看数据区，到`4FAA36`处的有一个`A6`，再往下直到`4FB94C`都是`0`，`A6`与`ESI`处数据的第一字节吻合，

长度也都是`0F17`字节，可以判断出`4FAA36`处就应该是`MAINFORM`的原始位置，因此做如下修改：

```
003442C7  BF 36AA4F00         MOV EDI,4FAA36                          ；FORM 的原始地址
003442CC  893B                MOV DWORD PTR DS:[EBX],EDI              ；添入 FORM 的参考指针
003442CE  B9 170F0000         MOV ECX,0F17                            ；FORM 长度
003442D3  F3:A4               REP MOVSB                               ；移回原始位置
003442D5  EB 16               JMP SHORT 003442ED
003442D7  90                NOP
003442D8  90                NOP
003442D9  90                NOP
003442DA  90                NOP
```

接下来把`ANTI DUMP`处理掉，`2000`下是这里：

```
00344311  64:FF35 30000000    PUSH DWORD PTR FS:[30]
00344318  58                  POP EAX
00344319  85C0                TEST EAX,EAX
0034431B  78 0F               JS SHORT 0034432C
0034431D  8B40 0C             MOV EAX,DWORD PTR DS:[EAX+C]
00344320  8B40 0C             MOV EAX,DWORD PTR DS:[EAX+C]
00344323  C740 20 00100000    MOV DWORD PTR DS:[EAX+20],1000          ；ANTI DUMP，改为 NOP
0034432A  EB 39               JMP SHORT 00344365
```

从下面这里我们`F7`就来到`OEP`了：

```
0034442A  C3                  RETN                                    ；TO OEP
```

看看`OEP`处的代码：

```
0050CB6C  FFD7                CALL EDI
0050CB6E  58                  POP EAX
```

把这两行代码改成：

```
0050CB6C  55                  PUSH EBP
0050CB6D  8BEC                MOV EBP,ESP
```

到此就算大功告成了。

后面工作大家就可以按个自习惯做了，我是用`OD`的`PLUGINS`的`OllyDump`先生成`DUMP.EXE`

然后运行`Import REConstructor v1.6F`，`Attach`到我们的进程上

`OEP`：`0010CB6C`

`RVA`：`0018B1B8` `SIZE`：`000007E0`

去掉`Add new section`选项，输入`RVA`：`0018B998`，这样生成的文件更精练一些，不过要算好空间，有把握才行

点`Get Imports`按钮，一切`OK`

再`Fix Dump`，完全搞定了。

要想做锦上添花，还可以减肥，把壳所在的最后一个`SECTION`去掉，需要手工修复以下`TLS`，还要把被搬到壳中的几个资源

放到合适的位置上，这个我就懒得叙述了。

```
#################################################################################################

后记：

#################################################################################################
```

我还是不要装糊涂的好，脱到最后，我也知道壳是谁做的了，里面有一句信息：`i am xxxxx,do not unpack me,please.`

不好意思了，望兄弟不要介意。

```
#################################################################################################

iPB/heXer

03.09.04  4dd466d0e1e0cc123afbe504da2462ca
```
